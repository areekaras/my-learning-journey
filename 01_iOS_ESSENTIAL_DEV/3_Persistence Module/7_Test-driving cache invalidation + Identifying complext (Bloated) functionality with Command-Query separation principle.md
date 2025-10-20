# Learning Session: Test-driving cache invalidation + Identifying Complex (Bloated) Functionality with the Command-Query Separation Principle

---

### **Metadata**
- **Date:** `20-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/9613903

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
# Notes: Command-Query Separation (CQS) & Bloated Requirements

This lesson covers how to identify hidden complexity in requirements and code, using the Command-Query Separation principle to write simpler, more maintainable functions.

---

### 1. Identifying Bloated Requirements

-   **The Problem**: Large, complex tasks often hide smaller, "hidden" requirements. If you don't find these before coding, you risk making bad design decisions or missing deadlines.
-   **The "Load" Use Case Example**: The "Load Feed From Cache" use case (see image) contains *two* separate responsibilities, not one:
    1.  **A Query**: Retrieving the feed.
    2.  **A Command**: Deleting the cache if it's expired or corrupt.
-   <img width="624" height="551" alt="image" src="https://github.com/user-attachments/assets/3fac10da-6e2a-4865-a550-f1e7102d0879" />
-   **The Solution**: Break down problems into the smallest possible tasks *before* you start coding.
    -   Refining requirements is a core developer skill, not just a product manager's job.
    -   This uncovers hidden work, leads to better estimates, and helps you negotiate scope.

---

### 2. Command-Query Separation (CQS) Principle

CQS is a design principle that helps you identify and fix "bloated" methods that do too much.

-   **The Principle**: A method should be *either* a Command *or* a Query, but **never both**.

-   **Query (Getter)**:
    -   **Returns a result** (e.g., gets data).
    -   **Has NO side-effects**. It does not change the state of the system.
    -   *Asking a question shouldn't change the answer.*

-   **Command (Modifier / Setter)**:
    -   **Performs an action** that changes the state of the system (a "side-effect").
    -   **Does NOT return a value** (`Void`).

-   **How it applies to the Use Case**:
    -   The `load()` function was doing **both**: it was a **query** (returning a feed) and a **command** (deleting the cache).
    -   This is a violation of CQS. It makes the code complex, hard to test, and surprising (a "getter" shouldn't delete things).
    -   **The Fix**: Break it into two separate functions:
        1.  `load()` (a **Query**) that *only* retrieves the feed and returns a result.
        2.  `validateCache()` (a **Command**) that *only* checks the cache and deletes it if invalid, returning `Void`.

---

### 3. CQS is a Guideline, Not a Silver Bullet

-   CQS is a powerful tool for simplifying code, but it's not a strict law.
-   It's common to break CQS if it leads to simpler, more idiomatic code.
-   **Example**: Swift's `Array.remove(at:)` is a **command** (mutates the array) but it also **returns a value** (the removed element). This is a pragmatic and useful design, even though it violates CQS.

---

### Senior & Architect Interview Perspective

**Q: "Your `loadFeed` function retrieves a feed, but *also* deletes the cache if it's expired. What's your opinion of this design?"**

> "That's a **violation of the Command-Query Separation (CQS) principle**. The function is acting as both a **query** (returning data) and a **command** (mutating state by deleting the cache).
>
> This is a poor design because it's **surprising**—a function that sounds like a simple "getter" has a hidden, destructive side-effect. It makes the code harder to reason about and test.
>
> I would refactor this by splitting the responsibilities. I'd have a `retrieve()` function that is a pure **query** and *only* returns the cached data. Then, I'd have a separate `validateCache()` function that is a pure **command** and *only* handles the deletion logic."

**Q: "Is CQS an absolute rule? For example, Swift's `Array.remove(at:)` returns the removed element."**

> "No, it's a powerful **guideline**, not an absolute law. The goal of CQS is to reduce complexity. In the case of `Array.remove(at:)`, breaking CQS is a pragmatic choice that makes the API more useful and idiomatic.
>
> The key is to be intentional. In our `loadFeed` example, the side-effect was hidden and created complexity. With `Array.remove(at:)`, the behavior is explicit and expected. The principle is a tool to help us identify and simplify 'bloated' or 'surprising' functions, not a dogma we follow blindly."

---
