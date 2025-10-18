# Learning Session: Decoupling Use-case business Logic From Framework Details + Controlling Time + Multi-method abstractions Following the Interface Segregation and Single Responsibility Principles

---

### **Metadata**
- **Date:** `18-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/9265138

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

# Notes: Decoupling Use Cases & Discovering Collaborators

This lesson covers the TDD process for implementing business logic (Use Cases) while keeping it fully decoupled from framework details like Core Data or `Codable`.

---

### 1. Decoupling Business Logic from Frameworks

-   **Core Concept**: Your business logic should define *what* the system must do, not *how* it's done. The framework (e.g., Core Data, `URLCache`) is an implementation detail that handles the *how*.
-   **Dependency Inversion**: The framework must conform to the interfaces your business logic *needs*. Your business logic should **never** depend on the framework.
-   **Example**: The `LocalFeedLoader` (business logic) should not know about `Codable` or `NSManagedObject`. It should only talk to an abstraction, like a `FeedStore` protocol.

---

### 2. TDD Strategy: Discovering Interfaces

This is a key TDD refinement to avoid "abstracting too early."

1.  **Start with a Concrete Spy**: In your test, create a concrete helper class (e.g., `FeedStoreSpy`). **Do not create a protocol yet.**
2.  **Test-Drive the Use Case**: Write tests for your Use Case (e.g., `LocalFeedLoader`) and make it talk to the concrete `FeedStoreSpy`. As you write tests, add the methods and properties the Use Case *needs* (e.g., `deleteCache()`, `insert(...)`) to the spy.
3.  **Discover the Interface**: The public interface of the `FeedStoreSpy` *becomes* the interface your Use Case needs.
4.  **Extract Protocol**: Once the tests are passing and the interface is stable, extract a formal `protocol FeedStore` from the spy's public methods.
5.  **Conform**: Make the `LocalFeedLoader` depend on the new protocol, and have the `FeedStoreSpy` conform to it.

-   **Why this is smart**: It gives you the flexibility to refactor and discover the *correct* interface in your tests without the friction of breaking a predefined protocol contract with every change.

---

### 3. Controlling Time in Tests

-   **The Problem**: `Date()` is an impure function. Calling it directly inside business logic makes tests non-deterministic, as the time is always different.
-   **The Solution**: **Inject the "current time"** as a dependency.
-   **How**:
    1.  Add a closure property to your Use Case: `let currentDate: () -> Date`.
    2.  Pass this in via the initializer (Constructor Injection).
    3.  In your production code (at the Composition Root), inject the real clock: `LocalFeedLoader(store: store, currentDate: Date.init)`.
    4.  In your tests, inject a fixed date: `let fixedDate = Date(); LocalFeedLoader(store: spy, currentDate: { fixedDate })`.
-   This gives your tests full, stable control over time, allowing you to assert caching logic (e.g., "cache is 7 days old").

---

### 4. Protocol Design (SRP & ISP)

-   **How many methods should a protocol have?** Ideally, **one**.
-   **When is it okay to have more?** When the methods are all part of a **Single Responsibility (SRP)**.
-   **Example**: The `FeedStore` protocol has `delete` and `insert`. This is acceptable because both methods relate to the single responsibility of "cache management."
-   **Interface Segregation Principle (ISP)**: "Clients shouldn't depend on methods they don't use."
-   **In Practice**: Since the `LocalFeedLoader` (the client) *needs* both `delete` and `insert` to perform its logic, it's not being forced to depend on anything it doesn't use. Therefore, a single protocol is appropriate.

---

### Senior & Architect Interview Perspective

**Q: "How do you ensure your core business logic isn't coupled to frameworks like Core Data or Realm?"**

> "Through **Dependency Inversion**. My business logic (the Use Case) defines a simple, 'persistence-agnostic' protocol it needs—for example, a `FeedStore` with `delete` and `insert` methods. These methods use simple domain models, not framework-specific objects.
>
> Then, I create a separate component, an **Adapter**, that conforms to this protocol. This adapter (e.g., `CoreDataFeedStore`) is the *only* part of my app that knows about Core Data. It acts as a translation layer between the needs of the business logic and the implementation details of the framework. This way, the business logic is totally decoupled, and I could swap Core Data for Realm by just writing a new adapter, with zero changes to the Use Case."

**Q: "You need to test logic that's based on the current time. How do you do that reliably?"**

> "I never let business logic get the time directly by calling `Date()`. That makes it untestable. Instead, I **inject a clock** as a dependency.
>
> In Swift, the simplest way is to inject a closure, `() -> Date`, via the initializer. In production, I pass in the system clock, `Date.init`. In my tests, I create a fixed `Date` and pass in a closure that returns that fixed date, ` { self.fixedTestDate }`. This makes my time-based tests 100% deterministic and reliable."


---
