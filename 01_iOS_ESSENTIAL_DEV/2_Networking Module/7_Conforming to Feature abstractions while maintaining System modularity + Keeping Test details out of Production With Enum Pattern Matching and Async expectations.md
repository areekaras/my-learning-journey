# Learning Session: Conforming to Feature Abstractions while maintaining System Modularity + Keeping Test details out of Production with Enum Pattern Matching and Async Expections

---

### **Metadata**
- **Date:** `05-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/8846462

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is clear.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)
*My structured notes, guided by deep questions.*

#### Notes

---

# Interview Q&A: Swift's Error Handling (`Result`, `Error`)

This note covers key questions about Swift's modern error handling mechanisms, focusing on the `Result` type and the `Error` protocol.

## Q1: What is the `Error` protocol in Swift?

The **`Error` protocol** is a simple, marker protocol that indicates a type can be used for error handling. Any type that conforms to `Error` can be thrown and caught in Swift's `do-catch` system.

Enums are an excellent choice for modeling errors because they allow you to group related error conditions into a distinct, type-safe set of cases.

#### Example:
A custom error enum conforming to the `Error` protocol.
```swift
enum NetworkError: Error {
    case badURL
    case requestFailed(reason: String)
    case decodingFailed
    case unknown
}
```


## Q2: What is the `Result` type and why was it introduced?

**`Result`** is an enum that explicitly represents either a **success** or a **failure** in a single type. It was introduced in Swift 5 to provide a standardized way to handle operations that can fail, especially in asynchronous contexts.

Its definition boils down to:
```Swift
enum Result<Success, Failure: Error> {
    case success(Success)
    case failure(Failure)
}
```

**Why it's important**: Before `Result`, asynchronous APIs often used completion handlers with multiple optional parameters (e.g., `(Data?, Error?)`), which led to ambiguity. Was `(nil, nil)` a valid state? `Result` eliminates this by forcing the outcome to be either a success *or* a failure, never both or neither.

#### Example:
```Swift
// Old, ambiguous way
func fetchData(completion: (Data?, Error?) -> Void) { /* ... */ }

// New, explicit way using Result
func fetchData(completion: (Result<Data, NetworkError>) -> Void) { /* ... */ }
```


## Q3: When would you use `Result` versus `try/catch` (`throws`)?

This is a key architectural question. Both are valid error-handling mechanisms, but they excel in different contexts.

* **Use `throws` for synchronous code**: When an error needs to be handled immediately and locally. The `do-catch` syntax is clean and efficient for code that executes in a single, unbroken sequence.
* **Use `Result` for asynchronous code**: It's perfect for completion handlers in network calls, timers, or any operation where the outcome isn't available immediately. The `Result` value can be stored, passed around, and handled later.

#### Simple Rule of Thumb:
-   If the function runs and returns a value right away, use **`throws`**.
-   If the function starts a task and will return a value in the future (e.g., in a closure), use **`Result`**.


## Q4: How does `Result.flatMap` help improve code quality?

**`flatMap`** on a `Result` is a powerful tool for chaining multiple failable operations together without "pyramids of doom" (deeply nested `if`s or `switch` statements).

It works like this:
-   If the `Result` is a `.success`, `flatMap` unwraps the value and passes it to a closure that returns a *new* `Result`.
-   If the `Result` is a `.failure`, `flatMap` immediately stops and propagates the failure.

This allows you to create a clean, linear chain of dependent asynchronous calls.

#### Example: Chaining Network Calls
```Swift
//Without flatMap (Pyramid of Doom):
func getProfile() {
    fetchUserID { idResult in
        switch idResult {
        case .success(let id):
            fetchUserProfile(for: id) { profileResult in
                switch profileResult {
                case .success(let profile):
                    print("Got profile: \(profile)")
                case .failure(let error):
                    print("Profile fetch failed: \(error)")
                }
            }
        case .failure(let error):
            print("User ID fetch failed: \(error)")
        }
    }
}

// With flatMap (Clean and Linear):
func getProfileCleanly() {
    let finalResult = fetchUserID()
        .flatMap { id in fetchUserProfile(for: id) }
        .flatMap { profile in fetchUserPhotos(for: profile) } // Even more chaining!

    switch finalResult {
    case .success(let photos):
        print("Got photos: \(photos)")
    case .failure(let error):
        print("An operation failed: \(error)")
    }
}
```


## Q5: How can you convert a `Result` type into a thrown error?

The `Result` type has a convenient `get()` method that does exactly this.

-   If the result is `.success(value)`, `get()` returns the `value`.
-   If the result is `.failure(error)`, `get()` **throws** the `error`.

This is useful when you have a function that returns a `Result`, but you want to use it within a synchronous context that uses `do-catch`.

#### Example:
```Swift
// Converting a Result into a thrown error
let userIDResult: Result<Int, NetworkError> = .success(42)

do {
    let id = try userIDResult.get()
    print("Successfully retrieved user ID: \(id)")
} catch {
    print("An error occurred: \(error)")
}

let failedResult: Result<Int, NetworkError> = .failure(.requestFailed(reason: "Server down"))

do {
    let id = try failedResult.get() // This line will throw the error
} catch let error as NetworkError {
    print("A network error occurred: \(error)") // This block will be executed
}
```



---

### Phase 3: Proof of Learning (After the Session)
*This is where the knowledge becomes permanent. This section is completed from memory.*

#### My "Blank Page" Summary (Feynman Technique)
*(Write your summary of the topic here without looking at your notes from Phase 2. Draw a diagram if it helps.)*

#### Consolidation Checklist
- [ ] **Teach-Back Complete:** I have stood up and explained this concept out loud for 60 seconds.
- [ ] **Next-Day Review Scheduled:** I have added a 5-minute task to my plan for tomorrow to review this note.
- [ ] **Weekly Review Scheduled:** I have added this note to my list for the upcoming "Weekly Knowledge Review" session.
