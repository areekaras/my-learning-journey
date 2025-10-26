# Learning Session: Are Singletons and Global instances Damaging your system design and Testability

---

### **Metadata**
- **Date:** `13-09-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/8528509

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is to understand Singleton and Global instances.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** No previous topics, i am starting today

---

### Phase 2: Active Notes (During the Session)
*My structured notes, guided by deep questions.*

- [x] **Embrace the Struggle:** Attempt problems on your own for 15-20 mins before seeking hints.

### Notes

---

# Refactoring Singletons & Testing the Network Layer

## The Singleton Pattern
>A Singleton is a design pattern that ensures a class has **only one instance** and provides a **single, global point of access** to it.


### Singleton Example

A typical Singleton implementation in Swift and its usage.
```swift
// A typical Singleton implementation in Swift
class ApiClient {
    static let instance = ApiClient()

    // The private initializer prevents creating other instances
    private init() {}

    func fetchData() { /* ... */ }
}

// Usage:
ApiClient.instance.fetchData()

// Apple's frameworks often use this pattern, e.g., URLSession.shared
```

## Senior/Architect-Level Discussion

#### What's the Problem with Singletons? 🧐

While simple, the Singleton pattern is often considered an **anti-pattern** in modern development for several key reasons:

1.  **Global Mutable State**: The single instance is a global variable. If its state can be changed from anywhere in the app, it becomes very difficult to reason about the application's state, leading to unpredictable behavior and bugs.

2.  **Implicit Dependencies & Tight Coupling**: When a class uses `ApiClient.instance` directly, it creates a **hidden, tight coupling** to that concrete `ApiClient`. This makes the class difficult to:
    * **Test**: You cannot easily replace the real `ApiClient` with a mock or stub for unit tests.
    * **Reuse**: The class is not reusable in another context without also bringing along the `ApiClient` Singleton.
    * **Maintain**: If you need to change `ApiClient`, you have to hunt down and change every place it's used directly.

3.  **Breaks SOLID Principles**:
    * **Single Responsibility**: The Singleton is responsible for both its own logic *and* for managing its single instance.
    * **Dependency Inversion**: High-level modules become dependent on a low-level, concrete implementation, violating this principle.


## Refactoring Away from Singletons

The goal is to move from **implicit dependencies** (hidden globals) to **explicit dependencies** (passed in). This is achieved through **Dependency Injection (DI)**.

### The Refactoring Steps

1.  **Create an Abstraction (Protocol)**: Define a protocol that describes *what* the service does, not *how*. This inverts the dependency.

    ```Swift
    protocol HTTPClient {
        func get(from url: URL, completion: @escaping (Result<Data, Error>) -> Void)
    }
    ```

2.  **Conform the Concrete Type**: Make your existing networking class conform to this protocol. It is no longer a Singleton.

    ```Swift
    class URLSessionHTTPClient: HTTPClient {
      private let session: URLSession
  
      // The dependency (URLSession) is now injected
      init(session: URLSession = .shared) {
          self.session = session
      }
  
      func get(from url: URL, completion: @escaping (Result<Data, Error>) -> Void) {
          // ... implementation ...
      }
    }
    ```

3.  **Inject the Dependency**: The classes that need the networking service should accept the **protocol**, not the concrete type, in their initializer.

    ```Swift
    class RemoteFeedLoader {
      private let client: HTTPClient // Depends on the protocol!
  
      init(client: HTTPClient) {
          self.client = client
      }
  
      func load() {
          // ... uses client ...
      }
    }
    ```

This process breaks the tight coupling, makes your code testable, and aligns with SOLID principles. The creation of objects (the "composition root") is now handled at a single, high-level point in your app.

### Advanced Interview Questions

1.  **"When, if ever, is a Singleton an acceptable choice?"**
    * **Answer**: In very rare cases where you need a truly global, stateless, and immutable service. A good example might be a logging utility or a theme provider where the configuration is loaded once and never changes. Even then, injecting it as a dependency is often a better, more testable pattern.

2.  **"You've refactored a Singleton to use DI. Where does the responsibility of creating the concrete instance now lie?"**
    * **Answer**: The responsibility moves to a "Composition Root." This is a single place in the application (often in the `AppDelegate` or `SceneDelegate`) where all the main objects are created and wired together.

3.  **"How does this refactoring from Singleton to DI improve your ability to test the networking layer?"**
    * **Answer**: It's a game-changer. By depending on a protocol (`HTTPClient`), our unit tests can now create a mock implementation. We can inject this mock into the class we're testing and have complete control over the networking "responses." This allows us to test all possible scenarios—success, specific errors, empty data—without making any real network calls, resulting in fast, reliable, and deterministic tests.

---

### Phase 3: Proof of Learning (After the Session)
*This is where the knowledge becomes permanent. This section is completed from memory.*

#### My "Blank Page" Summary (Feynman Technique)
This topic is about what is Singleton, singlton , Global state along with how to test networking layer and what are the problems using singleton, How to use singleton properly . How to achieve dependency inversion so low level framework will depends on high level code

#### Consolidation Checklist
- [x] **Teach-Back Complete:** I have stood up and explained this concept out loud for 60 seconds.
- [x] **Next-Day Review Scheduled:** I have added a 5-minute task to my plan for tomorrow to review this note.
- [x] **Weekly Review Scheduled:** I have added this note to my list for the upcoming "Weekly Knowledge Review" session.

