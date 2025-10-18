# Learning Session: From Singletons and Globals to Proper Dependency Injection

---

### **Metadata**
- **Date:** `14-09-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/8516257

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is to understand how dependency injection used for Singletons and Globals.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)
*My structured notes, guided by deep questions.*

- [x] **Embrace the Struggle:** Attempt problems on your own for 15-20 mins before seeking hints.

#### Notes

# Notes: From Singletons to Dependency Injection

This topic covers the architectural trade-offs of the Singleton pattern versus Dependency Injection (DI) and how to test-drive the refactoring of a network layer from a tightly-coupled design to a modular one.

---

## 1. The Singleton Anti-Pattern

A Singleton is a design pattern that ensures a class has only one instance and provides a **single, global point of access** to it.

#### The Singleton (Before)
The `ApiClient` is a Singleton, and `RemoteFeedLoader` is tightly coupled to it.

```swift
// The Singleton
class ApiClient {
    static let instance = ApiClient()
    private init() {}

    func get(from url: URL) {
        // ... makes real network request ...
    }
}

// The Tightly-Coupled Class
class RemoteFeedLoader {
    func load() {
        // ...
        // We are HARD-CODED to the singleton
        ApiClient.instance.get(from: URL(string: "[http://api.com](http://api.com)")!)
    }
}
```

### Advanced Interview Questions (Singletons)

**Q: Singletons are simple and convenient. Why are they often considered an anti-pattern at a senior or architect level?**

**A:** Because "convenient" often leads to "untestable and unmaintainable." The core problems are:
1.  **Implicit Hidden Dependencies**: A class that calls `ApiClient.instance` has a hidden dependency. You can't know its dependencies just by looking at its `init`.
2.  **Tight Coupling**: The class is directly tied to the **concrete implementation** (`ApiClient`), not an abstraction. This violates the Dependency Inversion Principle.
3.  **Global Mutable State**: If the Singleton holds state (e.g., cached data, user tokens), it becomes a global variable. Any part of the app can change this state, leading to unpredictable behavior and bugs that are extremely difficult to trace.
4.  **Untestable**: You cannot mock a Singleton. When you test a class that calls `ApiClient.instance`, you are forced to make a real network request. This makes your unit tests slow, flaky (fail due to network issues), and unable to test specific scenarios, like error handling.

**Q: When, if ever, is a Singleton acceptable?**

**A:** Very rarely. The only potentially acceptable use case is for something that is **truly global, stateless, and immutable**. Examples might include a simple logging utility or a "Theme/Font" provider that is configured once at launch and never changes. Even then, injecting it as a dependency is a cleaner, more testable pattern. For anything with state or side effects (like networking), it should be avoided.

---

## 2. The Solution: Dependency Injection (DI)

Dependency Injection is a pattern where an object's dependencies are "injected" (passed in) from an external source, rather than being created by the object itself.

This moves your code from **Implicit Dependencies** (hidden globals) to **Explicit Dependencies** (clear `init` parameters).

### Advanced Interview Questions (DI)

**Q: What are the main types of Dependency Injection, and as an architect, which do you prefer?**

**A:** There are three main types. My preference depends on the situation, but I default to one.

1.  **Constructor Injection (Preferred)**: Dependencies are passed into the `init`.
    * **Pro**: This is the best approach. It makes dependencies explicit and **guarantees** the object is in a valid, usable state immediately after creation. It also encourages immutability (using `let`).
    * **Con**: Can lead to larger initializers if an object has many dependencies (which might itself be a sign of a design problem).

2.  **Property Injection**: Dependencies are set via a public `var` *after* the object is created.
    * **Pro**: Useful in specific cases, like when using Storyboards (where you can't control the `init`) or for truly optional dependencies.
    * **Con**: The object can exist in an invalid state (before the property is set). It also breaks immutability (`let`) and hides the dependency from the public `init`.

3.  **Method Injection**: The dependency is passed as a parameter into the specific method that needs it (e.g., `func load(with client: HTTPClient)`).
    * **Pro**: Useful if a dependency is only needed for a single operation and doesn't need to be stored.
    * **Con**: Can clutter method signatures.

**Architect's Choice**: **Constructor Injection** is the default for 99% of cases. It creates honest, robust, and testable components.

**Q: How do you test-drive the refactor of a class that depends on a Singleton?**

**A:** This is a classic refactoring process:

1.  **Write a Failing Test**: First, write a unit test for the class that uses the Singleton. You'll quickly find it's impossible to test the error paths, or the test is slow. This test proves *why* the refactor is necessary.
2.  **Introduce an Abstraction (Protocol)**: Create a protocol that defines the *contract* of the service (e.g., `protocol HTTPClient { ... }`).
3.  **Refactor with DI**: Change the class's `init` to accept the new **protocol** as a dependency (Constructor Injection). Update the class to use this injected property instead of `Singleton.instance`.
4.  **Fix the Test**: Create a mock/stub object that conforms to the protocol. You can now inject this mock into your class under test and have full control over its behavior, allowing you to test success, failure, and edge cases instantly.
5.  **Clean Up**: The production code will be broken. To fix it, you make the original Singleton class conform to the new protocol and inject `ApiClient.instance` at the "Composition Root." Later, you can remove the `static instance` entirely.

#### The Abstraction (Protocol)
We define a contract. This is the **Dependency Inversion** Principle.

```Swift
// We define a contract. This is the Dependency Inversion Principle.
// The "what" (our contract)
protocol HTTPClient {
    func get(from url: URL, completion: @escaping (Result<Data, Error>) -> Void)
}
```

#### The Refactor (After)
The `URLSessionHTTPClient` is a concrete implementation. The `RemoteFeedLoader` now depends on the protocol via **Constructor Injection**

```Swift
// The "how" (a concrete implementation)
class URLSessionHTTPClient: HTTPClient {
    func get(from url: URL, completion: @escaping (Result<Data, Error>) -> Void) {
        // ... implementation using URLSession ...
    }
}

// The Decoupled Class
class RemoteFeedLoader {
    private let client: HTTPClient // Depends on the abstraction!

    // The dependency is "injected"
    init(client: HTTPClient) {
        self.client = client
    }

    func load(from url: URL) {
        client.get(from: url) { result in
            // ... handle result ...
        }
    }
}
```

#### The Test (Mocking)
Our test now uses a mock, giving us full control

```Swift
class RemoteFeedLoaderTests: XCTestCase {

    func test_load_deliversErrorOnNetworkError() {
        // ARRANGE
        let (sut, client) = makeSUT()
        let expectedError = NSError(domain: "Test", code: 1)

        // ACT
        var capturedError: Error?
        sut.load(from: anyURL()) { result in
            if case let .failure(error) = result {
                capturedError = error
            }
        }
        
        // Assert on the *mock*
        client.complete(with: expectedError) 

        // ASSERT
        XCTAssertNotNil(capturedError)
    }

    // MARK: - Helpers
    
    private func makeSUT() -> (sut: RemoteFeedLoader, client: HTTPClientMock) {
        let client = HTTPClientMock()
        let sut = RemoteFeedLoader(client: client)
        return (sut, client)
    }

    private class HTTPClientMock: HTTPClient {
        private var messages = [(completion: (Result<Data, Error>) -> Void)]()

        func get(from url: URL, completion: @escaping (Result<Data, Error>) -> Void) {
            messages.append(completion)
        }
        
        // Helper to "complete" the request in tests
        func complete(with error: Error, at index: Int = 0) {
            messages[index](.failure(error))
        }
    }
}
```

**Q: What is a "Composition Root" and how does it relate to DI?**

**A:** The **Composition Root** is the *single, centralized place* in your application where objects are "composed" or "wired up." This is typically at the very start of the app, like the `AppDelegate` or `SceneDelegate`.

Instead of `ServiceA` creating `ServiceB`, and `ServiceB` creating `ServiceC`, the Composition Root creates all three: `ServiceC`, `ServiceB(client: ServiceC)`, and `ServiceA(client: ServiceB)`.

This is crucial because it keeps your components decoupled. The `RemoteFeedLoader` has no idea what concrete `HTTPClient` it's using; it only knows about the protocol. This centralization of creation logic is the key to a modular and maintainable architecture.

#### The Composition Root (Example)
In your `SceneDelegate` (or similar), you "compose" the app

```Swift
// In the SceneDelegate...
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {

        // 1. Create the concrete, low-level dependencies
        let httpClient = URLSessionHTTPClient()

        // 2. Create the high-level module and INJECT the dependency
        let feedLoader = RemoteFeedLoader(client: httpClient)

        // 3. Pass the loader to the View Controller
        let feedViewController = FeedViewController(loader: feedLoader)
        
        // ...
    }
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
