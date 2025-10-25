# Learning Session: Overview

---

### **Metadata**
- **Date:** `25-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/10512500

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
# Notes: Persistence Module - Detailed Learning Outcomes

This is a point-by-point list of all learning outcomes from the Persistence Module.

-   <img width="2153" height="1252" alt="image" src="https://github.com/user-attachments/assets/a0aff645-423b-4d71-ada8-e1000a0e2957" />

---

## Architecture & Design Principles

-   **Dependency Inversion**: Understood the anatomy (High-level, Low-level, Boundary) and its role in decoupling.
-   **Functional Core, Imperative Shell**: Designed side-effect-free (deterministic) domain models (Functional Core) separate from infrastructure (Imperative Shell).
-   **Command-Query Separation (CQS)**: Identified bloated code and separated 'queries' (getters, no side-effects) from 'commands' (mutators, side-effects).
-   **Decoupling Logic**: Separated Application-specific (Use Cases) vs. Application-agnostic (Domain Models) vs. Framework (Infrastructure) logic.
-   **Model Decoupling (DTOs)**: Used data transfer model representations to achieve modularity and solve dependency bottlenecks.
-   **SOLID Principles**:
    -   **Interface Segregation Principle (ISP)**: Formed good abstractions by not forcing clients to depend on methods they don't use.
    -   **Single Responsibility Principle (SRP)**: Applied this principle to protocols and classes.
    -   **Liskov Substitution Principle (LSP)**: Understood its role in forming correct abstractions.
-   **Composition over Inheritance**: Understood the Composite Reuse Principle and why protocol composition is superior to class inheritance in Swift.
-   **Entities vs. Value Objects**: Learned the difference based on whether a model has an intrinsic *identity* or is defined by its *values*.
-   **Single Source of Truth (SSOT)**: Promoted reusability and reduced defects by creating single sources of truth (e.g., for the "7-day" rule).

---

## Requirements, Process & Collaboration

-   **Iterative Design**: Understood the advantages of iterative design over big up-front decision making.
-   **Handling Requirements**: Learned to minimize risk by reviewing, refining, and expanding requirements iteratively.
-   **Domain Knowledge**: Enhanced cross-domain knowledge with clear cross-team communication (e.g., "Ubiquitous Language").
-   **Identifying Bloated Requirements**: Broke down large tasks to uncover hidden information and complexity.
-   **Specs as Contracts**: Used specs (like the `FeedStore` contract) to enable parallel development and ensure implementation correctness.
-   **Documentation**: Documented infrastructure requirements with an "Inbox Checklist."
-   **Codebase Health**: Monitored the impact of refactoring with quantitative analysis (lines of code, indentations, files changed per commit).
-   **Reliable History**: Maintained a reliable codebase history where every commit is in a working state (all tests passing).

---

## Testing Strategies & Techniques

-   **The Testing Pyramid**: Achieved a healthy distribution, focusing on a wide base of Unit/Isolated tests and a small top of Integration tests.
-   **Unit/Isolated vs. Integration**: Understood the pros/cons, performance, and complexity trade-offs for each.
-   **Reusable Test Specs**: Extracted reusable test specs using protocol inheritance, extensions, and composition to validate any protocol implementation.
-   **TDD (Test-Driven Development)**:
    -   Test-drove an API layer implementation.
    -   Test-drove cache invalidation logic.
    -   Test-drove infrastructure components (`CodableFeedStore`) in integration with real frameworks.
-   **Test Refactoring**:
    -   **Async-to-Sync Wrappers**: Turned async code into sync for test readability (hiding `XCTestExpectation`).
    -   **Eliminating "Arrow Code"**: Refactored nested code to improve expressiveness.
    -   **"Tiny DSLs"**: Created simple Domain Specific Languages (e.g., `Date` extensions) to decouple tests from implementation details.
-   **Test Quality**:
    -   **DRY vs. Coincidental Duplication**: Identified when "duplicate" test setup code should *not* be merged.
    -   **Test Triangulation**: Increased coverage by testing boundary conditions (e.g., just before, on, and just after the 7-day limit).
    -   **Explicit Dependencies**: Made tests clearer by making all dependencies (like the `storeURL`) explicit.
    -   **Stateful Tests**: Prevented flaky tests by cleaning up state (`setupEmptyStoreState`) in *both* `setUp()` and `tearDown()`.
-   **Measuring Test Performance**: Used `xcodebuild` to measure test time overheads and make informed decisions about integration testing vs. mocking.

---

## Concurrency

-   **Thread-Safe Design**: Designed and tested thread-safe components using `DispatchQueue`.
-   **Serial vs. Concurrent Queues**: Understood the differences and when to use each.
-   **Race Condition Avoidance**: Proactively avoided bugs (data corruption, crashes) caused by side-effects in multithreaded environments.
-   **Thread-Safe(r) Value Types**: Understood why value types (`struct`) are inherently safer in concurrent environments than reference types (`class`).

---

## Frameworks & Implementations

-   **`URLCache`**: Analyzed its pros and cons as a simple caching alternative.
-   **`Codable` + `FileManager` (`CodableFeedStore`)**:
    -   Encoded and decoded models with `Codable`.
    -   Persisted and retrieved model data from the disk.
    -   Managed the trade-off of `Codable` conformance (domain model vs. DTO).
-   **Core Data (`CoreDataFeedStore`)**:
    -   Understood the Core Data overview, concurrency model (using `context.perform`), trade-offs, and modeling techniques.
    -   Correctly handled calendrical calculations using `Calendar` instead of naive math.
    -   Implemented the reusable `FeedStoreSpecs` contract.
-   **Memory Management**: Checked for expected behavior after deallocation to prevent retain cycles in closures.

---

### Senior & Architect Interview Perspective

This module prepared you to answer senior-level questions on:

-   **System Design**: "How would you design a decoupled persistence layer? Explain your use of DIP and DTOs."
-   **Architecture**: "What's the difference between Application-Agnostic logic and Application-Specific logic? How do you separate them?"
-   **Testing Strategy**: "What is your testing pyramid? Why not just write integration tests?"
-   **Test Design**: "How do you create a reusable test suite for multiple implementations of a protocol?"
-   **Concurrency**: "What is a race condition, and how would you design a thread-safe `FeedStore`? Explain the 'Reader-Writer' pattern."
-   **Framework Trade-offs**: "When would you use Core Data vs. a simple `Codable` + File solution? What are the concurrency implications of Core Data?"
-   **Code Quality**: "What is 'coincidental duplication' and how does it relate to the DRY principle?"
-   **Swift Concurrency**: "How has SwiftData and Actors changed how you would approach designing a thread-safe persistence layer today?"
---
