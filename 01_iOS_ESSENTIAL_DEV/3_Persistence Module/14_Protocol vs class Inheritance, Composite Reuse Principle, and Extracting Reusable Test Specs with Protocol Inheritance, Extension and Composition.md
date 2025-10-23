# Learning Session: Protocol vs Class Inheritance, Composite Reuse Principle, and Extracting Reusable Test specs with Protocol Inheritance, Extension and Composition
---

### **Metadata**
- **Date:** `24-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/10184141

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

# Notes: Protocol Composition vs. Class Inheritance (Test Specs)

This lesson covers how to create a reusable, "pluggable" test suite using protocol composition. This is a powerful, Swift-idiomatic pattern that is far superior to class inheritance for code reuse.

---

### 1. The Core Principle: "Prefer Composition Over Inheritance"

This is also known as the **Composite Reuse Principle**.

-   **Class Inheritance (The "Bad" Way)**:
    -   This is a rigid, "is-a" relationship.
    -   Swift only allows **single inheritance** (a class can only inherit from *one* other class).
    -   This makes it impossible to use for composing reusable functionality. We can't make our test class inherit from `XCTestCase` *and* a `FailableSpecs` base class.
    -   <img width="1671" height="874" alt="image" src="https://github.com/user-attachments/assets/eb37e4b0-48be-4a83-83f0-001b1eb295a4" />

-   **Protocol Composition (The "Good" Way)**:
    -   This is a flexible, "behaves-like" relationship.
    -   A class can conform to **multiple protocols**.
    -   We can "compose" different behaviors onto a single class, which is the key to reuse.
    -   <img width="1253" height="685" alt="image" src="https://github.com/user-attachments/assets/00f13eb2-ee5f-4c2f-851e-16f154d14489" />

---

### 2. SOLID Principles: The "Why" Behind the Pattern

This entire pattern is a practical application of the **Interface Segregation Principle (ISP)** and the **Liskov Substitution Principle (LSP)**.

-   **Interface Segregation Principle (ISP)**:
    -   "No client should be forced to depend on methods it does not use."
    -   **The Problem**: We have an `InMemoryFeedStore` that *cannot fail*. If we had one "fat" `FeedStoreSpecs` protocol that included failure tests, our `InMemoryFeedStoreTests` would be *forced* to "implement" (or ignore) failure tests it doesn't use.
    -   **The Fix**: We segregate the interface. We create a small, core `FeedStoreSpecs` for the "happy path" and separate, specialized protocols for failure cases (e.g., `FailableRetrieveFeedStoreSpecs`).
    -   **The Result**: Our `InMemoryFeedStoreTests` can conform *only* to the base `FeedStoreSpecs` and is not "polluted" with irrelevant failable tests.
    -   <img width="1368" height="471" alt="image" src="https://github.com/user-attachments/assets/49d50603-33e1-4b07-bbca-d8f127cbd608" />

-   **Liskov Substitution Principle (LSP)**:
    -   "Subtypes must be substitutable for their base types without altering the program's correctness."
    -   **The Problem**: If `InMemoryFeedStoreTests` conformed to a "fat" protocol with failable tests, it would violate LSP. It would *pretend* to be a failable store, but its "implementation" of `test_retrieve_deliversFailure` would be empty or a `fatalError`, which alters the program's correctness.
    -   **The Fix**: By using ISP to create smaller protocols, we also fix LSP. The `InMemoryFeedStoreTests` is now a *correct* subtype of `FeedStoreSpecs` (the happy path).

---

### 3. Creating Reusable Test Specs

This is the "how-to" for building the contract-based test suite.

1.  **Define a Base Protocol**: Create `FeedStoreSpecs` with all the "happy path" test methods (e.g., `test_retrieve_deliversEmptyOnEmptyCache`).
2.  **Define Specialized Protocols**: Create protocols for specific behaviors, like failure. These protocols **inherit** from the base protocol (e.g., `FailableRetrieveFeedStoreSpecs: FeedStoreSpecs`).
3.  **Provide Default Implementations**: Use **protocol extensions** to provide the *default implementation* for every test method in these protocols.
4.  **Compose as Needed**: Create a `typealias` to easily compose all failable specs: `typealias FailableFeedStoreSpecs = FailableRetrieveFeedStoreSpecs & FailableInsertFeedStoreSpecs ...`
5.  **Conform in the Test Class**: Your test classes now just conform to the specs they need. The protocol extensions provide all the test logic "for free."
    -   `class CodableFeedStoreTests: XCTestCase, FailableFeedStoreSpecs { ... }` (gets all tests)
    -   `class InMemoryFeedStoreTests: XCTestCase, FeedStoreSpecs { ... }` (gets *only* the happy path tests)

---

### 4. Refining Tests: Single Assertion Principle

-   **The Guideline**: Aim for a "single assertion per test" (or, more accurately, a "single *concept* per test").
-   **The Problem**: The process revealed a test named `test_insert_deliversErrorOnInsertionError` which was *also* checking that there were *no side effects*. It was testing two distinct behaviors.
-   **The Risk**: If this test fails, which behavior broke? The error delivery, or the side-effect prevention? You don't know.
-   **The Fix**: Split it into two separate tests, each with an explicit name:
    1.  `test_insert_deliversErrorOnInsertionError`
    2.  `test_insert_hasNoSideEffectsOnInsertionError`
-   **The Benefit**: The test now serves as precise, unambiguous documentation.

---

### Senior & Architect Interview Perspective

**Q: "Why does Swift favor 'Prefer Composition over Inheritance'?"**

> "Because **class inheritance is a rigid, single-dimension tool**, while **protocol composition is flexible and multi-dimensional**.
>
> In Swift, a class can only inherit from *one* superclass, which creates a tight, brittle 'is-a' hierarchy. You can't 'compose' behaviors.
>
> But a class or struct can conform to *many* protocols. This allows us to define small, reusable behaviors (like `Codable`, `Equatable`, or our own `FailableFeedStoreSpecs`) and 'compose' them onto a type. It's how Swift achieves polymorphism and massive code reuse without the problems of multiple inheritance."

**Q: "Explain the relationship between Liskov Substitution (LSP) and Interface Segregation (ISP)."**

> "They are two sides of the same coin, and **ISP is often the solution to an LSP violation**.
>
> An **LSP violation** is when a subtype (like an `InMemoryFeedStore`) can't *actually* be substituted for its base type (a `FeedStore` that can fail) because it would break the program's correctness—for example, by having to `fatalError` an 'unsupported' failable method.
>
> The **ISP solution** is to *segregate* that "fat" interface. We break `FeedStore` into `FeedStoreSpecs` (happy path) and `FailableFeedStoreSpecs`. Now, the `InMemoryFeedStore` can *correctly* conform to *only* the `FeedStoreSpecs`, restoring LSP. It no longer pretends to be something it's not."

**Q: "I see a test `test_login_deliversErrorAndDisablesButton`. What's your opinion?"**

> "That's a code smell. It's testing **two distinct behaviors** in one test. If that test fails, the error message is ambiguous: did the error not get delivered, *or* was the button not disabled?
>
> I would **split this into two separate tests**: `test_login_deliversErrorOnFailure` and `test_login_disablesButtonOnFailure`. This follows the 'single concept per test' principle. The tests now serve as clear, explicit documentation for each individual behavior."

**Q: "You've built this `FeedStoreSpecs` protocol suite. How does this help you as a Lead?"**

> "It becomes a powerful **'Contract as a Test Suite'**. I can now assign a task to a developer to build a new `RealmFeedStore`. Their "Definition of Done" is simple:
>
> 1.  Create a new `RealmFeedStoreTests` class.
> 2.  Make it conform to `XCTestCase` and our `FailableFeedStoreSpecs`.
> 3.  Implement the `RealmFeedStore` (the SUT).
> 4.  Run the tests.
>
> When all the inherited tests pass, I am **guaranteed** that their new implementation correctly fulfills every behavioral contract our application expects (thread-safety, side-effects, error handling) without me having to manually review every single line of implementation logic. It enables high-quality, parallel development."


# Code Snippets: Test Spec Protocols

These snippets illustrate how to build reusable test suites using protocol inheritance and composition.

---

### 1. The Base Protocol (Happy Path)

This protocol defines the core, "happy path" behaviors that *all* `FeedStore` implementations must satisfy.
```swift
protocol FeedStoreSpecs {
    func test_retrieve_deliversEmptyOnEmptyCache()
    func test_retrieve_hasNoSideEffectsOnEmptyCache()
    func test_retrieve_deliversFoundValuesOnNonEmptyCache()
    func test_retrieve_hasNoSideEffectsOnNonEmptyCache()

    func test_insert_overridesPreviouslyInsertedCacheValues()
    // ... other happy path insert/delete tests
    
    func test_storeSideEffects_runSerially()
}
```

### 2. The Specialized Protocol (Failure Case)
This protocol inherits from the base FeedStoreSpecs and adds new, specialized requirements for "failable" stores.

```swift
protocol FailableRetrieveFeedStoreSpecs: FeedStoreSpecs {
    func test_retrieve_deliversFailureOnRetrievalError()
    func test_retrieve_hasNoSideEffectsOnFailure()
}

protocol FailableInsertFeedStoreSpecs: FeedStoreSpecs {
    func test_insert_deliversErrorOnInsertionError()
    func test_insert_hasNoSideEffectsOnInsertionError()
}

// ... and so on for Delete
```

### 3. Composition with a typealias
We use a typealias to compose all the granular protocols into one convenient "failable" spec.

```swift
typealias FailableFeedStoreSpecs = FailableRetrieveFeedStoreSpecs & FailableDeleteFeedStoreSpecs & FailableInsertFeedStoreSpecs
```

### 4. Usage in Test Classes
Now, our test classes simply conform to the specs they need.

```swift
// This test class gets ALL specs (happy path + all failure paths)
class CodableFeedStoreTests: XCTestCase, FailableFeedStoreSpecs {
    // Test implementations are inherited from protocol extensions
}

// This test class gets ONLY the happy path specs
class InMemoryFeedStoreTests: XCTestCase, FeedStoreSpecs {
    // It is not forced to depend on failure tests it doesn't use (ISP)
}
```

---
