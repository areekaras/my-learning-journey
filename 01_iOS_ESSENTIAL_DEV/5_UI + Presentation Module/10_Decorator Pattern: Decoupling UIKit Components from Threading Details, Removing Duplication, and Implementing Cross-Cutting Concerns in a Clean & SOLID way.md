# Learning Session: Decorator Pattern: Decoupling UIKit Components from Threading Details, Removing Duplication, and Implementing Cross-cutting Concerns in a clean & SOLID way 

---

### **Metadata**
- **Date:** `04-12-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/11570507

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is clear.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)


#### Notes

PR: https://github.com/areekaras/EssentialFeed/pull/19

---
# Notes: The Decorator Pattern & Threading

This lecture introduces the **Decorator Pattern** as a solution to a common architectural problem: polluting UI code with threading logic. We use it to decouple our View Controllers from the details of background vs. main thread execution.

---

### 1. The Problem: Leaking Threading Details

-   **The Requirement**: UIKit components (like `UITableView`, `UILabel`) **must** be updated on the **Main Thread**.
-   **The Conflict**: Our `<FeedLoader>` implementations (like `RemoteFeedLoader`) often run on background threads.
-   **The Anti-Pattern**: If we put `DispatchQueue.main.async` inside our `FeedViewController` (or `Presenter`), we are **leaking implementation details**.
    -   The View Controller shouldn't know *where* the data comes from or *what thread* it arrives on. It should just receive data and display it.
    -   If we assume background threads and defensively dispatch to main everywhere, we create massive **duplication** and clutter.

-   **[Image Placeholder: Diagram showing "leaky" threading logic in VC]**

---

### 2. The Solution: The Decorator Pattern

The **Decorator Pattern** allows us to dynamically add behavior (like threading) to an individual object without changing its class.

-   **How it works**:
    1.  Create a class (the **Decorator**) that conforms to the *same protocol* as the object it wraps (the **Decoratee**).
    2.  The Decorator receives the message (e.g., `load()`).
    3.  It adds extra behavior (e.g., "dispatch to main queue").
    4.  It forwards the message to the underlying Decoratee.

-   **Why it's SOLID**:
    -   **Single Responsibility Principle (SRP)**: The `FeedLoader` focuses on loading data. The `MainQueueDispatchDecorator` focuses on threading. The `ViewController` focuses on UI. Concerns are separated.
    -   **Open/Closed Principle (OCP)**: We extended the behavior of the system (added thread safety) without modifying existing classes.
    -   **Liskov Substitution Principle (LSP)**: The Decorator conforms to `<FeedLoader>`, so it can be swapped in anywhere a `FeedLoader` is expected.

---

### 3. Cross-Cutting Concerns

Threading is a "Cross-Cutting Concern"—a requirement that affects many parts of the system (like Logging, Analytics, Security).

-   Instead of scattering logging or threading code inside every single class, we use Decorators in the **Composition Root**.
-   This keeps our core business logic pure and platform-agnostic.

---

### 4. Implementation: `MainQueueDispatchDecorator`

We created a generic `MainQueueDispatchDecorator<T>` that wraps *any* object `T`.

-   **Safety Check**: It checks `Thread.isMainThread`. If we are already on the main thread, it executes immediately. If not, it dispatches async to the main queue.
-   **Conformance**: We extend the Decorator to conform to specific protocols (like `<FeedLoader>` or `<FeedImageDataLoader>`) so it can be injected transparently.

---

### Senior & Architect Interview Perspective

**Q: "How do you handle threading in your architecture? Do you put `DispatchQueue.main.async` in your View Controllers?"**

> "I strive to keep my View Controllers and Presenters threading-agnostic. They shouldn't care if data comes from a background thread or memory.
>
> Instead, I use the **Decorator Pattern** in the Composition Root. I wrap my `FeedLoader` in a `MainQueueDispatchDecorator`. This decorator intercepts the completion block and ensures it runs on the main thread. This keeps my UI code clean, readable, and free of duplicated dispatch logic."

**Q: "What are Cross-Cutting Concerns, and how do you handle them?"**

> "Cross-cutting concerns are functionalities like Logging, Analytics, Authentication, or Threading that apply to multiple modules across the system.
>
> If we put this logic inside our business classes, we violate the **Single Responsibility Principle**. Instead, I use **Decorators**. For example, I can wrap a `FeedLoader` in a `LoggingFeedLoaderDecorator` that logs requests, and then wrap *that* in a `MainQueueDispatchDecorator` for the UI. This lets me compose behavior dynamically without modifying the core logic."

**Q: "Why use a Decorator instead of just subclassing?"**

> "Subclassing is rigid and compile-time bound. You can't easily combine multiple behaviors (like Logging + Threading + Caching) via inheritance without creating a massive hierarchy.
>
> The Decorator pattern relies on **Composition**. It allows me to wrap *specific instances* at runtime. I can have one instance of `RemoteFeedLoader` that is logged, and another that isn't, simply by how I compose them in the Composition Root."

---
