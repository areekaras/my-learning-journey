# Learning Session: Composite Pattern: Implementing a Flexible & Composable Strategy for Loading Data with Fallback Logic

---

### **Metadata**
- **Date:** `11-08-2026`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/12194217

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is clear.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)


#### Notes

PR: 
1. https://github.com/areekaras/EssentialFeed/pull/25

---

### 1. The Stub Test-Double
* Stubs provide predefined responses to method invocations during testing.
* A Stub does not capture values for later inspection (unlike a Spy test double).
* **Trade-offs:** Stubs require less code and help create flexible tests. However, they can result in less code coverage and lead to too many upfront design decisions. Spies are often better for taking smaller steps and gaining more control.

```Swift
private class LoaderStub: FeedLoader {
    private let result: FeedLoader.Result
    
    init(result: FeedLoader.Result) {
        self.result = result
    }
    
    func load(completion: @escaping (FeedLoader.Result) -> Void) {
        completion(result)
    }
}
```

### 2. Composing Remote and Local Fallback Loaders
The goal is to create a modular strategy for loading data with fallback logic without altering the existing services or clients, strictly adhering to the **Open/Closed Principle**.

**Variation 1: Composition with Concrete Types**
* The composite conforms to `<FeedLoader>` and depends directly on the concrete `RemoteFeedLoader` and `LocalFeedLoader` types.
* **Traits:** Enforces compile-time type checks, but results in tight coupling. Testing must be done in integration rather than isolation.
* <img width="566" height="335" alt="image" src="https://github.com/user-attachments/assets/66ef6382-0623-497b-babb-b9e7ed494fd9" />

**Variation 2: Composition with Abstract Types**
* The composite conforms to `<FeedLoader>` and depends purely on the `<FeedLoader>` abstraction for both remote and local strategies.
* **Traits:** Increases flexibility and allows for isolated testing. However, it cannot enforce compile-time checks for passing the dependencies in the correct order.
* <img width="527" height="358" alt="image" src="https://github.com/user-attachments/assets/6124efe1-6c2f-4067-98fe-f7655a8b56be" />

**Variation 3: Composition with Custom Protocol Abstractions**
* The composite depends on two new protocols (`<RemoteFeedLoader>` and `<LocalFeedLoader>`), both inheriting from the base `<FeedLoader>`.
* **Traits:** Enforces compile-time dependency order and supports isolated testing, but increases system complexity by adding more types.
* <img width="538" height="393" alt="image" src="https://github.com/user-attachments/assets/859c1d3a-714d-4f1d-a00a-c9998f2b1d95" />

**Variation 4: Composition Without Remote/Local Notions (Primary/Fallback)**
* The composite (`FeedLoaderWithFallbackComposite`) conforms to `<FeedLoader>` and simply takes a `primary` and `fallback` loader.
* **Traits:** The composite is completely agnostic to the provenance of the data (remote vs. local). It is simple, highly flexible, and allows for infinite composition (e.g., composing composites of composites).
* <img width="525" height="248" alt="image" src="https://github.com/user-attachments/assets/be7b0bd8-f43e-4293-8e82-fbd5fc4ea4a1" />

```Swift
// 1. FeedLoaderWithFallbackComposite
import EssentialFeed

public class FeedLoaderWithFallbackComposite: FeedLoader {
    private let primary: FeedLoader
    private let fallback: FeedLoader
    
    public init(primary: FeedLoader, fallback: FeedLoader) {
        self.primary = primary
        self.fallback = fallback
    }
    
    public func load(completion: @escaping (FeedLoader.Result) -> Void) {
        primary.load { [weak self] result in
            switch result {
            case .success:
                completion(result)
                
            case .failure:
                self?.fallback.load(completion: completion)
            }
        }
    }
}
```

```Swift
// 2. FeedImageDataLoaderWithFallbackComposite (Managing Task State)
import Foundation
import EssentialFeed

public class FeedImageDataLoaderWithFallbackComposite: FeedImageDataLoader {
    private let primary: FeedImageDataLoader
    private let fallback: FeedImageDataLoader
    
    public init(primary: FeedImageDataLoader, fallback: FeedImageDataLoader) {
        self.primary = primary
        self.fallback = fallback
    }
    
    private class TaskWrapper: FeedImageDataLoaderTask {
        var wrapped: FeedImageDataLoaderTask?
        
        func cancel() {
            wrapped?.cancel()
        }
    }
    
    public func loadImageData(from url: URL, completion: @escaping (FeedImageDataLoader.Result) -> Void) -> any FeedImageDataLoaderTask {
        let task = TaskWrapper()
        task.wrapped = primary.loadImageData(from: url) { [weak self] result in
            switch result {
            case .success:
                completion(result)
                
            case .failure:
                task.wrapped = self?.fallback.loadImageData(from: url, completion: completion)
            }
        }
        return task
    }
}
```

### 3. The Composite Design Pattern
* **Definition:** A pattern that allows composing objects into tree structures to represent part-whole hierarchies.
* **Core Benefit:** Clients can treat individual objects (leaves) and compositions of objects (branches) uniformly through a single interface. This eliminates the need for complex tag-and-case statement logic in the client.
* **Structure:** A Composite object can refer to a leaf object (which ends the composition) or another Composite object (which extends the composition further), allowing infinite scaling.
* <img width="553" height="377" alt="image" src="https://github.com/user-attachments/assets/26b4876e-55ad-4c77-8144-245ac247da24" />
* <img width="1502" height="622" alt="image" src="https://github.com/user-attachments/assets/d452a1e2-e9c5-46df-9c4c-da8a8f9014c5" />


### 4. Related Design Patterns
* **Strategy Design Pattern:** Encapsulates a family of related algorithms and makes them interchangeable, allowing the behavior to vary independently from the client using it.
* **Chain of Responsibility Design Pattern:** Chains receiving objects and passes a request along the chain until an object handles it, avoiding coupling the sender of a request to its specific receiver.

---

### Senior & Architect Interview Perspective

**Q: Why use the Composite Pattern for data loading fallback instead of just putting the fallback logic inside a View Controller or a dedicated Manager class?**
> "Using the Composite Pattern strictly adheres to the Open/Closed Principle. If we put this logic in a VC or Manager, adding a third fallback layer (e.g., Remote -> Local -> Default Placeholder Asset) would require modifying existing code. With a Composite, we can simply compose two composites together without touching the client or the underlying loaders. The client remains completely agnostic to where the data comes from; it just speaks to the `<FeedLoader>` interface."

**Q: In Swift, how do you handle cancelling an asynchronous task (like an image download) when the underlying operation switches from a primary loader to a fallback loader?**
> "You use a Wrapper pattern for the task. We create a `TaskWrapper` class that conforms to our task protocol (e.g., `FeedImageDataLoaderTask`) and returns it immediately to the client. Internally, the wrapper holds a reference to the active task. When the primary fails and the fallback initiates, we reassign the wrapper's underlying task. When the client calls `cancel()`, the wrapper safely forwards the cancellation to whichever task is currently active."

**Q: What is the primary tradeoff between injecting concrete types vs. generic abstractions (`primary` and `fallback`) into your Composite?**
> "Concrete types provide strong compile-time safety (you can't accidentally pass a local loader as the remote parameter). However, they tightly couple the composite to those specific implementations, making testing harder and breaking the Open/Closed Principle. Abstracting to `primary` and `fallback` removes compile-time ordering guarantees but achieves total decoupling, allowing us to mock them easily in tests and reuse the composite for any fallback scenario, not just remote-to-local."


---
