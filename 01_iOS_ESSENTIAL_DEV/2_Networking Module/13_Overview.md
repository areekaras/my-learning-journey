# Learning Session: Overview

---

### **Metadata**
- **Date:** `10-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/10538540

---

#### Notes
- <img width="1491" height="762" alt="image" src="https://github.com/user-attachments/assets/3145ee84-3906-4e2b-9d08-b93dee24bdd2" />

# iOS Lead Essential Academy: Learning Outcomes To-Date

## Architectural Principles & Design

-   Depicting dependencies, abstractions, and concrete types in diagrams.
-   Translating diagrams into code and vice-versa.
-   Applying **SOLID principles** and **composition**.
-   Understanding the trade-offs between starting with abstractions vs. concrete components.
-   Representing component and module relationships in a diagram.
-   Assessing system modularity from its diagram.
-   Understanding that systems must be tailored, not fit into a predefined template.
-   Applying modularization incrementally.
-   Protecting architecture by using domain-specific models instead of data-transfer models (e.g., `Decodable` structs).
-   Maintaining modularity by protecting high-level abstractions from low-level implementation details.
-   Designing with enums to make invalid states unrepresentable.


## Team Processes & Requirements

-   Understanding that good architecture is a byproduct of good team processes.
-   The importance of good communication and well-defined requirements.
-   How to handle vague or incomplete requirements.
-   Representing and communicating requirements in different formats.
-   Using contracts (e.g., API specifications) to enable parallel, independent development.
-   Establishing processes that promote detailed documentation.


## Dependency Management

-   Understanding and refactoring **Singletons**: when to use them, their better alternatives, and steps to remove them.
-   Controlling dependencies: Implicit (global shared instances) vs. Explicit (**Dependency Injection**).
-   Understanding the trade-offs of access control (e.g., `internal` vs. `public`) for testing purposes.


## Testing Strategies & Techniques

-   Test-driving an API layer implementation.
-   Automating memory leak detection with tests.
-   Using **Test Spies** to expand behavior checking.
-   Understanding the differences between **Stubbing** and **Spying**.
-   Leveraging factory methods and test helper functions to simplify tests.
-   Protecting production code from test-specific details.
-   Asserting asynchronous behavior with `XCTestExpectation`.
-   **Testing Network Requests**:
    -   Understanding various strategies and their trade-offs (End-to-end, Subclass Mocking, Protocol Mocking, `URLProtocol` Stubbing).
    -   Using the **URL Loading System** and `URLProtocol` to intercept requests.
-   **Test Economics**:
    -   Understanding the value of fast, reliable, and accurate test feedback.
    -   Comparing the high cost of End-to-End tests vs. the efficiency of Unit tests.
-   Testing all scenarios (happy, sad, and edge cases) to minimize risk.
-   Using extra Xcode configurations: running tests in **random order**, **parallelizing execution**, and gathering **code coverage**.
-   Testing client-server integration (End-to-End) even when the backend is not ready.


## Swift Language & Frameworks

-   Differences and similarities between closures and protocols as abstractions.
-   Mapping JSON to models using the **`Decodable`** protocol.
-   Handling potential issues when using the `Swift.Error` protocol.
-   Using pattern matching with enums.
-   Preventing common asynchronous bugs.
-   Using macOS frameworks to speed up development.


### Extra Context from Our Discussions

-   **`@escaping` vs. Non-Escaping Closures**: We discussed that `@escaping` is required for closures used in asynchronous operations (like network completions) because the closure must "escape" the function's scope to be called later.
-   **Higher-Order Functions (`map`, `filter`, `reduce`, etc.)**: We covered how to explain these from a senior/architect perspective, focusing on their role in creating declarative data pipelines and the trade-offs (performance, readability) compared to imperative loops.
-   **`Result` Type vs. `throws`**: We established that `Result` is ideal for asynchronous operations (completion handlers), while `throws` is suited for synchronous code where errors must be handled immediately.
-   **Network Reachability**: We confirmed Apple's best practice: **always attempt the connection** first instead of performing a pre-flight reachability check. Use `URLSessionConfiguration` properties like `waitsForConnectivity` to manage behavior.
-   **Data Races & Thread Sanitizer**: We noted that data races occur from unsynchronized access to shared memory. The **Thread Sanitizer** is the tool to detect these, and it's best enabled on a separate CI scheme due to its performance overhead.




---
