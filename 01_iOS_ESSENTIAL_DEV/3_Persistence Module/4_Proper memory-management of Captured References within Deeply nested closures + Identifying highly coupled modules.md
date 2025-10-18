# Learning Session: Proper Memory-management of Captured references within Deeply nested Closures + Identifying Highly-coupled Modules

---

### **Metadata**
- **Date:** `18-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/9291531

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
# Memory Management & Module Coupling


### 1. Memory Management: Testing for Deallocation

-   **Core Concept**: When a class uses closures, especially for asynchronous work (like `RemoteFeedLoader`), you must ensure it doesn't create a **retain cycle**.
-   **The "Why"**: A retain cycle (e.g., a class holds a closure strongly, and the closure captures the class strongly via `self`) causes a memory leak. The object is never deallocated.
-   **Test Strategy**: You must write a specific test to **check for expected behavior after deallocation**.
    1.  Create your object (the "SUT" - System Under Test) within a test.
    2.  Store a `weak` reference to it.
    3.  Trigger the code path (e.g., perform the load).
    4.  Set the main strong reference to `nil` (or let it fall out of scope).
    5.  **Assert that the `weak` reference is now `nil`**. If it's not, you have a memory leak.

---

### 2. Modular Design: Cohesion vs. Coupling

-   **High Cohesion (Good)**: Everything *inside* a single module is closely related and works together for one clear purpose (e.g., the `FeedCache` module only handles caching logic).
-   **Low Coupling (Good)**: Modules do not have deep, concrete knowledge of each other. They communicate through simple, abstract interfaces.
-   **The Litmus Test**: Ask yourself, "Can I refactor or add a feature to this module **without touching any files in another module**?" If the answer is `no`, your modules are highly coupled.

---

### 3. Identifying an Architectural Bottleneck

-   **The Problem**: As seen in the diagram, many modules (`Feed API`, `Feed Cache`, `Feed Feature`) all have a direct, source-code dependency on the **concrete `FeedItem` data model**.
-   **Why This is a Bottleneck**:
    -   `FeedItem` is a central point of coupling.
    -   A simple change to the `FeedItem` model (e.g., adding one property) will **force a cascade of changes** and recompilation in every module that depends on it.
    -   This is a "design that fights change" and makes development slow, risky, and expensive.

-   <img width="861" height="377" alt="image" src="https://github.com/user-attachments/assets/267bec56-f2c2-45f5-ae56-7121d8683228" />


---

### Senior & Architect Interview Perspective

**Q: "In this diagram, `FeedItem` is a major bottleneck. What does that mean, and how would you fix it?"**

> "This diagram shows a **highly-coupled architecture**. The `FeedItem` model is a concrete detail that all modules depend on, which violates the **Dependency Inversion Principle**.
>
> **The Fix**: The lower-level modules (API, Cache) should not dictate the model to the higher-level module (Feature). Instead, the **`Feed Feature` module should own its own abstract representation** of what it needs (e.g., a `FeedItemViewModel` or a simple `FeedItem` *struct* defined *within* the Feature module).
>
> Then, the `Feed API` and `Feed Cache` modules would be responsible for mapping *their* internal, concrete models (e.g., `RemoteFeedItem`, `LocalFeedItem`) to the model defined by the Feature module. This inverts the dependency. A change to the API's model no longer affects the Feature module at all."

**Q: "Your notes mention 'high-quality software is the result of high-quality small contributions.' What does that mean to you as a Lead?"**

> "It means you can't achieve a high-quality system with a low-quality process. Quality is a byproduct of discipline. As a lead, I enforce this by ensuring every small contribution is high-quality:
>
> 1.  **Test-First**: We write tests first to define the behavior.
> 2.  **Refactor**: We *always* clean up the code after the tests pass.
> 3.  **Process**: We use pairing and peer reviews to catch issues (like memory leaks or coupling) early.
>
> Focusing on the quality of every single commit, test, and review is the only way to build a high-quality, sustainable system. The end result is just a mirror of those small, daily actions."

---
