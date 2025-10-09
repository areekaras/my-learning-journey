# Learning Session: A classicist TDD approach (No mocking) to mapping with Decodable + Domain-specific models

---

### **Metadata**
- **Date:** `30-09-2025`, `05-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/8732933

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is to understand classist TDD approach without mocking to mapping JSON with Decodable plus Domain-specific models.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)

#### Notes

---

# Architect-Level Discussion on Swift's Higher-Order Functions

## The Architect's View on Higher-Order Functions

At its core, a **higher-order function** is a function that takes another function as an argument, returns a function, or both. For an architect, they are not just syntactic sugar; they are fundamental tools for building **declarative, expressive, and composable** data transformation pipelines.

The discussion is a choice between imperative and declarative styles:
* **Imperative (`for` loop):** You tell the computer *how* to do something.
* **Declarative (higher-order functions):** You describe *what* you want to achieve.

The discussion then becomes about the **trade-offs** of that abstraction: readability vs. complexity, performance, and appropriate use cases.


## Core Higher-Order Functions: A Deep Dive

### 1. `map`

* **Core Concept**: Transforms each element in a collection into a new element, returning a **new collection** of the same size. It represents a 1-to-1 transformation.

* **Example**:

#### Declarative Example
```swift
struct UserDTO { let id: Int; let name: String }
struct UserViewModel { let identifier: String }

let dtos = [UserDTO(id: 1, name: "Alice"), UserDTO(id: 2, name: "Bob")]
let viewModels = dtos.map { UserViewModel(identifier: "user_\($0.id)") }
```

#### Custom Generic Implementation
```swift
extension Array {
    func customMap<T>(_ transform: (Element) -> T) -> [T] {
        var result: [T] = []
        result.reserveCapacity(self.count) // Performance optimization
        for item in self {
            result.append(transform(item))
        }
        return result
    }
}
```

* **Architectural Discussion Points**:
    * **Immutability and Predictability**: `map` is a pure transformation. It doesn't mutate the original collection, which is a huge win for concurrency and predictable state management. It's a primary tool for transforming data models between layers (e.g., DTO -> ViewModel).
    * **Performance**: `map` is an O(n) operation that eagerly creates a new array. For very large collections, this can be inefficient if you don't need all the results immediately.

* **Advanced Interview Questions**:
    1.  "How would you implement your own generic `customMap` function on `Array` from first principles?"
        * **Answer**: "To build a custom map, you'd extend `Array` with a generic method. This method needs a generic placeholder `T` for the new type it will return. Inside, you create an empty result array, loop through `self`, apply the transformation closure to each element, and append the result to the new array before returning it."
        *(See above for the implementation).*
    2.  "When would a simple `for-in` loop be a better choice than `map`?"
        * **Answer**: When you need to `break` or `continue`, or if you are performing multiple side-effects where the transformation is secondary.
    3.  "`map` also exists on `Optional`. How does its behavior differ from `map` on `Array`?"
        * **Answer**: `Optional.map` transforms the wrapped value only if it's `.some`, otherwise it returns `.none`. It's a powerful tool for optional chaining without complex `if let` pyramids, treating the optional as a context to operate within.


### 2. `filter`

* **Core Concept**: Examines each element and returns a **new collection** containing only the elements that satisfy a given predicate.

* **Example**:
#### Declarative Example
```swift
struct User { var isActive: Bool }
let allUsers = [User(isActive: true), User(isActive: false), User(isActive: true)]
let activeUsers = allUsers.filter { $0.isActive }
```

#### Custom Generic Implementation
```swift
extension Array {
    func customFilter(_ isIncluded: (Element) -> Bool) -> [Element] {
        var result: [Element] = []
        for item in self {
            if isIncluded(item) {
                result.append(item)
            }
        }
        return result
    }
}
```

#### `filter` implemented with `reduce`
```swift
extension Array {
    func filterWithReduce(_ isIncluded: (Element) -> Bool) -> [Element] {
        return self.reduce(into: []) { result, element in
            if isIncluded(element) {
                result.append(element)
            }
        }
    }
}
```

* **Architectural Discussion Points**:
    * **Separation of Concerns**: `filter` cleanly separates the iteration logic from the business rule (the predicate), making the rule prominent and easy to test.
    * **Composition**: Its power is magnified when chained with other functions like `map`, creating readable, declarative pipelines.

* **Advanced Interview Questions**:
    1.  "How would you implement your own generic `customFilter` function on `Array` from first principles?"
        * **Answer**: "Similar to map, you extend `Array`. The method takes a predicate closure that returns a `Bool`. You create a new empty array, iterate through `self`, and inside the loop, call the predicate on each element. If the predicate returns `true`, you append the element to the new array."
        *(See above for the implementation).*
    2.  "Consider chaining `users.filter { ... }.filter { ... }`. What are the performance implications, and how could you optimize it?"
        * **Answer**: Each `filter` call creates a new intermediate array. It's often more performant to combine the conditions into a single `filter` with a logical AND (`&&`). This is a classic readability vs. performance trade-off that can be mitigated with `.lazy`.
    3.  "How would you implement `filter` yourself using `reduce`?"
        * **Answer**: This tests a deep understanding of `reduce`.
        *(See above for the implementation).*


### 3. `reduce`

* **Core Concept**: The most powerful of the collection functions. It combines all elements in a collection to produce a **single value** of any type.

* **Example**:
#### Declarative Example
```swift
// Simple reduction
let prices = [19.99, 24.50, 9.80]
let totalPrice = prices.reduce(0, +)

// Building a dictionary
struct UserWithId { let id: Int; let name: String }
let users = [UserWithId(id: 1, name: "Alice"), UserWithId(id: 2, name: "Bob")]
let userMap = users.reduce(into: [Int: UserWithId]()) { result, user in
    result[user.id] = user
}
```

#### Custom Generic Implementation
```swift
extension Array {
    func customReduce<Result>(_ initialResult: Result, _ nextPartialResult: (Result, Element) -> Result) -> Result {
        var accumulator = initialResult
        for element in self {
            accumulator = nextPartialResult(accumulator, element)
        }
        return accumulator
    }
}
```

* **Architectural Discussion Points**:
    * **The "Swiss Army Knife"**: `reduce` can be used to implement `map`, `filter`, and more. It's the fundamental building block of list transformations.
    * **State Aggregation**: It's the perfect tool for aggregating state, such as calculating statistics or building lookup tables. The `reduce(into:)` variant is highly efficient for this.

* **Advanced Interview Questions**:
    1.  "How would you implement your own generic `customReduce` function on `Array` from first principles?"
        * **Answer**: "The signature is the most complex part. It needs a generic `Result` type. The function takes an initial `Result` value and a combining closure. Inside, you declare a mutable `accumulator` variable initialized with the initial value. Then, you loop through the collection, updating the accumulator in each step by calling the closure with the current accumulator and the element. Finally, you return the accumulator."
        *(See above for the implementation).*
    2.  "Explain the difference between `reduce(_:_:)` and `reduce(into:_:)`."
        * **Answer**: `reduce` returns a *new* value for the accumulator in each step. `reduce(into:)` provides the accumulator as an `inout` parameter, allowing direct mutation, which is far more performant for building collections.
    3.  "In what scenarios could the readability of `reduce` become a liability?"
        * **Answer**: When the combining closure becomes very complex, it can be hard to decipher. In these cases, it's better to extract the logic into a well-named method or use a simpler combination of other higher-order functions if possible.


### 4. `compactMap` and `flatMap`

* **`compactMap`**: Transforms each element and **discards any `nil` results**.
* **`flatMap`**: Transforms a collection of collections and **flattens the result into a single collection**.

* **Examples**:
#### Declarative Example
```swift
// compactMap
let strings = ["1", "two", "3", "four"]
let numbers = strings.compactMap { Int($0) } // Result: [1, 3]

// flatMap
let nestedArrays = [[1, 2], [3, 4], [5]]
let flattened = nestedArrays.flatMap { $0 } // Result: [1, 2, 3, 4, 5]
```

#### Custom Generic Implementation
```swift
extension Array {
    // Custom compactMap
    func customCompactMap<T>(_ transform: (Element) -> T?) -> [T] {
        var result: [T] = []
        for item in self {
            if let transformedItem = transform(item) {
                result.append(transformedItem)
            }
        }
        return result
    }
}

// Custom flatMap requires a more specific extension
extension Array where Element: Collection {
    func customFlatMap() -> [Element.Element] {
        var result: [Element.Element] = []
        for subCollection in self {
            result.append(contentsOf: subCollection)
        }
        return result
    }
}
```

#### `Result.flatMap` Example
```swift
struct UserProfile {}
// Using the same User struct from the filter example
enum NetworkError: Error { case badURL }

func fetchUser(id: Int) -> Result<User, Error> { return .success(User(isActive: true)) }
func fetchProfile(for user: User) -> Result<UserProfile, Error> { return .success(UserProfile()) }

let userId = 123
let profileResult = fetchUser(id: userId).flatMap { user in
    return fetchProfile(for: user)
}
```

* **Architectural Discussion Points**:
    * **Data Integrity (`compactMap`)**: A crucial tool for data sanitization, especially when parsing data from an external source (e.g., JSON).
    * **Monadic Operations (`flatMap`)**: `flatMap` is fundamental for chaining computations within a context (like `Optional`, `Result`, or a Combine `Publisher`).

* **Advanced Interview Questions**:
    1.  "How would you implement your own generic `customCompactMap` and `customFlatMap` functions from first principles?"
        * **Answer**: "For `customCompactMap`, the logic is a mix of map and filter: you loop, apply the failable transformation, and use an `if let` to append only the non-nil results to your result array. For `customFlatMap`, you need a nested loop. The outer loop iterates through the collection of collections, and the inner loop appends elements from each sub-collection to the final flat array."
        *(See above for the implementations).*
    2.  "Explain the evolution of `flatMap` in Swift and why `compactMap` was introduced."
        * **Answer**: `flatMap` used to have two confusing overloads. SE-0187 separated them: the optional-unwrapping behavior was moved to the more descriptive `compactMap`, leaving `flatMap` to solely handle the flattening of sequences. This made the code's intent clearer.
    3.  "How would you use a higher-order function to chain two functions that each return a `Result<T, Error>`?"
        * **Answer**: This is a direct application of `Result`'s `flatMap` to cleanly chain the operations and propagate errors.
        *(See above for the implementation).*


## The Broader Architectural Picture 🗺️

### Laziness and Performance

Chaining multiple higher-order functions can be inefficient because each step creates an intermediate collection.

* **Solution**: The `.lazy` property on sequences.

#### `Result.flatMap` Example
```swift
let result = (1...1_000_000)
    .lazy
    .filter { $0 % 2 == 0 }
    .map { $0 * $0 }
    .first // Triggers computation for only the first valid element
```

Here, no intermediate arrays are created. The operations are fused and executed only as needed until the terminal operation (`.first`) is satisfied.

* **Interview Question**: "When should you use `.lazy`? What are its potential downsides?"
    * **Answer**: Use `.lazy` on large sequences where you chain multiple operations and/or don't need all the results at once. The downside is the result is a `LazySequence`, not an `Array`, and you must convert it if you need an array. For small collections, the overhead of the lazy wrappers can sometimes make it slightly slower.


---

### Phase 3: Proof of Learning (After the Session)
*This is where the knowledge becomes permanent. This section is completed from memory.*

#### My "Blank Page" Summary (Feynman Technique)
*(Write your summary of the topic here without looking at your notes from Phase 2. Draw a diagram if it helps.)*

#### Consolidation Checklist
- [ ] **Teach-Back Complete:** I have stood up and explained this concept out loud for 60 seconds.
- [ ] **Next-Day Review Scheduled:** I have added a 5-minute task to my plan for tomorrow to review this note.
- [ ] **Weekly Review Scheduled:** I have added this note to my list for the upcoming "Weekly Knowledge Review" session.
