# Learning Session: Designing and Testing Thread-safe Componenets with DispatchQueue, Serial vs. Concurrent Queues, Thread-safe Value Types, and Avoiding Race Conditions
---

### **Metadata**
- **Date:** `23-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/10029505

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is clear.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)


#### Notes

---
# Notes: Thread-Safe Design & Concurrency

This lesson covers how to design and test components that are "thread-safe," meaning they can be safely used from multiple threads at the same time without causing crashes or data corruption.

---

### 1. The Core Problem: Race Conditions

-   **What it is**: A **race condition** occurs when multiple threads try to access the same (shared) piece of *mutable* (changeable) *state* (memory) at the same time, and at least one of those accesses is a **write**.
-   **The Result**: Unpredictable behavior, corrupted data, and crashes. You can't trust your data.
-   **Reference Types (`class`)**: Are very vulnerable to this. When you pass a class instance to another thread, you are passing a *reference* (a pointer) to the *same object*. If both threads try to change a property on that object, they "race," and the final value is unpredictable.
-   **Value Types (`struct`)**: Are *safer*. When you pass a struct, Swift creates a **copy**. Each thread gets its own unique copy, so there is no *shared* state to race over.
    -   <img width="3301" height="2231" alt="image" src="https://github.com/user-attachments/assets/ec2de74a-4903-41d3-9897-093ee85ec663" />
    -   **The "Impure" Trap**: A struct is *not* safe if it contains a reference type (like a class or a closure). The struct is copied, but the reference inside it is *also* copied, meaning both copies still point to the *same shared class instance*.

---

### 2. The Tool: `DispatchQueue` (Grand Central Dispatch - GCD)

`DispatchQueue` is the system's tool for managing work. It's how we control *what* code runs *where* and *when*.

-   **Serial Queue**:
    -   Executes tasks **one at a time, in the order they are added** (First-In, First-Out).
    -   This is a simple and powerful way to prevent race conditions. If you wrap all access to a shared resource in a serial queue, you guarantee only one thread can access it at a time.
    -   <img width="3305" height="996" alt="image" src="https://github.com/user-attachments/assets/d2e8d458-cc64-43e5-b0a8-ca06a1c3c610" />

-   **Concurrent Queue**:
    -   Can execute **multiple tasks at the same time** (in parallel). The order of completion is not guaranteed.
    -   This is great for performance, but it *doesn't* protect you from race conditions by default.
    -   <img width="3301" height="1396" alt="image" src="https://github.com/user-attachments/assets/b4a30817-7143-4bac-99df-d5d1c57d38a8" />

---

### 3. The Pattern: Reader-Writer with Barriers

This is the advanced pattern used in the lecture. It gives you the "best of both worlds": the speed of a concurrent queue for "reads" and the safety of a serial queue for "writes."

-   **Concept**: You use one **concurrent queue** to manage access to your resource.
-   **Reads (Queries)**: You dispatch "read" operations as *normal* concurrent tasks. This allows 100 people to read the data at the same time without blocking each other.
-   **Writes (Commands)**: You dispatch "write" operations (like `insert` or `delete`) with a **barrier flag**.
-   **What the Barrier Does**:
    1.  It waits for all currently-running "read" tasks to finish.
    2.  It then runs your "write" task **all by itself**, guaranteeing it's the only thing accessing the resource.
    3.  After the "write" task is done, the queue resumes its normal concurrent behavior.
-   <img width="3304" height="1488" alt="image" src="https://github.com/user-attachments/assets/9a38438f-c86b-4c24-80ad-61083433226e" />
-   **Application**: For `CodableFeedStore`, `retrieve` (a read) can be concurrent, but `insert` and `delete` (writes) *must* be barriers to avoid data corruption.

```Swift
let queue = DispatchQueue(label: "\(CodableFeedStore.self)Queue", qos: .userInitiated, attributes: .concurrent)

public func retrieve(completion: @escaping RetrievalCompletion) {
        let storeURL = storeURL
        queue.async {
            guard let data = try? Data(contentsOf: storeURL) else {
                return completion(.empty)
            }
            
            do {
                let decoder = JSONDecoder()
                let cache = try decoder.decode(Cache.self, from: data)
                completion(.found(feed: cache.localFeed, timestamp: cache.timestamp))
            } catch {
                completion(.failure(error))
            }
        }
    }

public func insert(_ feed: [LocalFeedImage], timestamp: Date, completion: @escaping InsertionCompletion) {
        let storeURL = storeURL
        queue.async(flags: .barrier) {
            do {
                let encoder = JSONEncoder()
                let cache = Cache(feed: feed.map(CodableFeedImage.init), timestamp: timestamp)
                let encoded = try encoder.encode(cache)
                try encoded.write(to: storeURL)
                completion(nil)
            } catch {
                completion(error)
            }
        }
    }
    
    public func deleteCachedFeed(completion: @escaping DeletionCompletion) {
        let storeURL = storeURL
        queue.async(flags: .barrier) {
            guard FileManager.default.fileExists(atPath: storeURL.path()) else {
                return completion(nil)
            }
            
            do {
                try FileManager.default.removeItem(at: storeURL)
                completion(nil)
            } catch {
                completion(error)
            }
        }
    }

```


---

### 4. Testing for Thread Safety

-   You can't just *assume* your code is thread-safe; you must **test for it**.
-   **The Strategy**: Use `XCTestExpectation` and dispatch multiple "write" operations from a *concurrent* loop. Even though you fire them all at once, you assert that they complete *in the order you dispatched them*. This proves your component is correctly serializing the side-effects.
-   **Performance**: The lecture notes show this new test adds a tiny **3.9% (0.008s)** overhead. This is a negligible cost for the huge gain in confidence and safety. We must **measure**, not guess, about performance.
-   <img width="1095" height="402" alt="image" src="https://github.com/user-attachments/assets/ec5989ab-d610-4169-b0ae-4106c131eaf3" />

```Swift
func test_storeSideEffects_runSerially() {
        let sut = makeSUT()
        var completedOperationsInOrder = [XCTestExpectation]()
        
        let op1 = expectation(description: "Operation 1")
        sut.insert(uniqueImageFeed().local, timestamp: Date()) { _ in
            completedOperationsInOrder.append(op1)
            op1.fulfill()
        }
        
        let op2 = expectation(description: "Operation 2")
        sut.deleteCachedFeed { _ in
            completedOperationsInOrder.append(op2)
            op2.fulfill()
        }
        
        let op3 = expectation(description: "Operation 3")
        sut.insert(uniqueImageFeed().local, timestamp: Date()) { _ in
            completedOperationsInOrder.append(op3)
            op3.fulfill()
        }
        
        waitForExpectations(timeout: 5.0)
        
        XCTAssertEqual(completedOperationsInOrder, [op1, op2, op3], "Expected side-effects to run serially but operations finished in the wrong order")
    }
```

---

### How to Understand This Topic Completely (A Learning Path)

You mentioned a knowledge gap, which is very common. Here's a structured way to get confident.

1.  **Understand the Problem (The "Why")**: First, fully internalize what a **race condition** is. Write a simple app with a `class` that has a `counter` variable. Create two threads and have both of them increment the counter 1,000 times. Print the final result. You'll see it's *never* 2,000. This is the "Aha!" moment.
2.  **Learn the "Old" Solution (GCD)**:
    -   **Serial Queues**: Take your broken app and add a `DispatchQueue(label: "serial.queue")`. Make *every* read and write to the `counter` happen inside `queue.sync { ... }`. Rerun the app. It will now correctly print 2,000.
    -   **Concurrent Queues + Barriers**: This is the **Reader-Writer Pattern** from this lecture. It's the most common *manual* solution. Study the code snippet in File 2 until it makes perfect sense.
3.  **Learn the "New" Solution (Swift Concurrency)**:
    -   **`async/await`**: Understand this first. It's the syntax that makes asynchronous code look synchronous.
    -   **`Actors`**: This is the *modern* solution to this entire problem. An `actor` is like a `class` that **automatically protects its own state**. The compiler *guarantees* that all access to its properties is thread-safe, eliminating race conditions by default.
    -   For a senior role, **you must know Actors**. They replace the manual barrier pattern in 99% of new code.

---

### Senior & Architect Interview Perspective

**Q: "What is a race condition, and how do you prevent one?"**

> "A race condition is when multiple threads access a *shared, mutable state* simultaneously, with at least one access being a *write*. This leads to unpredictable behavior and data corruption.
>
> You prevent it by **enforcing synchronization**. My toolbox for this includes:
> 1.  **Serial `DispatchQueue`**: The simplest way. Wrap all access to the shared resource in `queue.sync` or `queue.async`, which guarantees only one operation can happen at a time.
> 2.  **Concurrent `DispatchQueue` with Barriers**: This is the high-performance 'Reader-Writer' pattern. Reads happen concurrently for speed, while writes (`insert`, `delete`) use a `barrier` flag to run exclusively.
> 3.  **`Actors` (Modern Swift)**: This is now the preferred solution. By declaring my state-holding object as an `actor`, the Swift compiler *automatically* enforces mutually exclusive access, making race conditions impossible by default."

**Q: "Explain the Reader-Writer pattern. When would you use it?"**

> "The Reader-Writer pattern is for when you have a resource that is **read often** but **written to infrequently**, like a cache.
>
> You use a `concurrent` queue. 'Reads' are dispatched as normal `sync` blocks, allowing many threads to read at once. 'Writes' are dispatched with a `barrier` flag. The barrier waits for all active reads to finish, runs the write task exclusively, and then re-opens the queue to concurrent reads. This gives you the safety of a serial queue for writes, with the high performance of a concurrent queue for reads."

**Q: "Why is a `struct` 'safer' in a multi-threaded environment than a `class`?"**

> "Because `struct`s are **value types**. When you pass a `struct` to another thread, it gets a **copy** of the data. Each thread has its own unique instance, so there is no *shared state* to cause a race condition.
>
> A `class` is a **reference type**. When you pass it, you're just passing a pointer to the *same* object in memory. If both threads try to mutate that one object, they will race."

**Q: "Can a `struct` still be part of a race condition?"**

> "Yes, if it's an **'impure' value type**. This is a `struct` that contains a `class` (a reference type) as one of its properties. When you copy the `struct`, the `class` property inside it is *not* cloned; only the *reference* is copied. The result is two `struct`s that look separate, but which both point to the *exact same class instance*, re-introducing shared mutable state and the risk of a race."

**Q: "How would you *test* that your `FeedStore`'s `insert` operation is thread-safe and runs serially?"**

> "I would write a test that *proves* serial behavior.
> 1.  I'd use `XCTestExpectation` to track the completion of several `insert` operations.
> 2.  I would use `DispatchQueue.concurrentPerform` to fire off (for example) 10 `insert` operations all at once from different threads.
> 3.  Even though they were *fired* concurrently, I would assert that the completion expectations are fulfilled *in the exact order* they were dispatched. This proves that the `FeedStore` is correctly lining them up and executing them one by one."

**Q: "With `async/await` and `Actors`, is `DispatchQueue` still relevant?"**

> "Absolutely, but its role has changed. For managing *state protection*, **Actors** are the clear modern replacement for `DispatchQueue` barriers.
>
> However, `DispatchQueue` is still fundamental. The `async/await` runtime is built on top of it. We still use `DispatchQueue.main` to dispatch to the main thread, and we still use global queues for scheduling work. We just no longer use it for *manual* synchronization as our first choice."

**Q: "Your `FeedStore` runs its completion block on a background thread. Is this a good design?"**

> "It's a common and reasonable default, *as long as it's documented*. The component can't guess what the client will do next. By returning on a background thread, it avoids blocking the main thread and gives the client the *choice* to map data, chain another async call, or dispatch to the main thread if they need to update UI.
>
> An alternative pattern is to inject the `completionQueue` as a dependency in the `init`, so the client can *tell* the store which queue to use, e.g., `init(store: myStore, completionQueue: .main)`."


### Code Snippets (Illustrating Concurrency Concepts)

```Swift
// MARK: 1. The Problem: A Race Condition

// A simple class that is NOT thread-safe.
class Counter {
    var count = 0
    
    func increment() {
        count += 1
    }
}

let counter = Counter()

// Firing from two different threads
DispatchQueue.global().async {
    counter.increment() // Thread 1 writes
}

DispatchQueue.global().async {
    counter.increment() // Thread 2 writes
}
// The final value of counter.count is unpredictable. It could be 1 or 2.

// MARK: 2. Solution: Serial Queue

// This class IS thread-safe.
class SafeCounter {
    private var count = 0
    private let queue = DispatchQueue(label: "com.example.serial.counter")
    
    func increment() {
        // We synchronize all access (reads and writes) on our serial queue.
        // This guarantees only one thread can be in this block at a time.
        queue.async {
            self.count += 1
        }
    }
    
    func getCount() -> Int {
        var currentCount = 0
        queue.sync { // .sync blocks the current thread until the work is done
            currentCount = self.count
        }
        return currentCount
    }
}

// MARK: 3. Solution: Reader-Writer Pattern (from the lecture)

// This class is thread-safe and optimized for frequent reads.
class ThreadSafeStore<T> {
    private var item: T
    
    // 1. We use a CONCURRENT queue
    private let queue = DispatchQueue(label: "com.example.concurrent.store", attributes: .concurrent)
    
    init(_ item: T) {
        self.item = item
    }

    // A "Read" operation (Query)
    func getItem() -> T {
        var currentItem: T!
        // We use a normal .sync block.
        // Multiple threads can be in here at the same time.
        queue.sync {
            currentItem = self.item
        }
        return currentItem
    }
    
    // A "Write" operation (Command)
    func setItem(_ newItem: T) {
        // We use a .barrier flag.
        // This waits for all reads to finish, runs this block ALONE,
        // and then resumes the queue.
        queue.async(flags: .barrier) {
            self.item = newItem
        }
    }
}

// MARK: 4. The "Impure" Struct Trap

class MyData {
    var value = 0 // This is mutable shared state
}

// This struct is NOT thread-safe
struct ImpureStruct {
    var data: MyData // It holds a reference type
}

let myData = MyData()
var struct1 = ImpureStruct(data: myData)
var struct2 = struct1 // The struct is copied...

struct2.data.value = 100 // ...but the reference inside it was also copied.

// Both structs point to the SAME MyData instance
print(struct1.data.value) // Prints 100
print(struct2.data.value) // Prints 100

// MARK: 5. The Modern Solution: Actor

// This actor IS thread-safe. The compiler handles all the synchronization.
// This replaces the Reader-Writer pattern in most new code.
actor SafeStore<T> {
    private var item: T
    
    init(_ item: T) {
        self.item = item
    }
    
    // A "Read" operation (Query)
    func getItem() -> T {
        return self.item // Safe to access
    }
    
    // A "Write" operation (Command)
    func setItem(_ newItem: T) {
        self.item = newItem // Safe to access
    }
}

// To use it, you must be in an async context and use 'await'
Task {
    let store = SafeStore("Hello")
    
    await store.setItem("World") // 'await' handles the thread-safety
    let value = await store.getItem()
    print(value) // Prints "World"
}
```


---
