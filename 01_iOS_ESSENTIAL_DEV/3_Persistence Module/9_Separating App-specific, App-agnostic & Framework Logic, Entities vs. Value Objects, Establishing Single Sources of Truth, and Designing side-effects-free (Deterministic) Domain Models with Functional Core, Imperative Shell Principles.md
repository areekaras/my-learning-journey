# Learning Session: Separating App-sepcific, App-agnostic & Framework Logic, Entities vs. Value Types, Establishing Single sources of Truth, and Designing side-effects-free (Deterministic) Domain Models with Functional Core, Imperative Shell Principles

---

### **Metadata**
- **Date:** `20-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/9763057

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
# Notes: Functional Core, Logic Separation, & Single Source of Truth

This lesson covers separating logic into distinct layers (Functional Core, Imperative Shell) and the difference between Entities and Value Objects.

---

### 1. The Three Layers of Logic

A common mistake is mixing business rules with framework code (e.g., putting validation logic inside a `URLSession` client). A clean architecture strictly separates them.

1.  **Framework (Infrastructure) Logic (The "Imperative Shell")**
    * **What it is**: The "dirty" outer layer. It performs all side-effects (I/O, UI, network, database).
    * **Job**: It should be **DUMB**. It takes commands from the Use Case and executes them. It contains **no business logic**.
    * **Examples**: `URLSessionHTTPClient`, `CoreDataFeedStore`.

2.  **Application-Specific Logic (The "Use Case / Controller")**
    * **What it is**: The "coordinator." It describes *what* a specific feature does (e.g., "load the feed, then validate the cache").
    * **Job**: It orchestrates the flow. It can be stateful and asynchronous, but it delegates all "dirty" work to the Framework layer via protocols.
    * **Example**: `LocalFeedLoader`, `RemoteFeedLoader`.

3.  **Application-Agnostic Logic (The "Domain Model / Functional Core")**
    * **What it is**: The "pure" inner core. It contains the **core business rules** that are true for any application.
    * **Job**: It must be **SIDE-EFFECT FREE** (deterministic). It takes data in, applies a rule, and returns data out. It has no dependencies on frameworks or async code.
    * **Example**: `FeedCachePolicy` (which contains the rule: "cache is invalid after 7 days").

-   <img width="2206" height="1127" alt="image" src="https://github.com/user-attachments/assets/c30f986d-85f0-4550-92e1-9d7bc9ad751a" />
-   <img width="1500" height="1475" alt="image" src="https://github.com/user-attachments/assets/f714049d-1e1e-4da9-8531-6e6553473bc7" />

---

### 2. Functional Core: Shifting from Impure to Pure

We refactored the `FeedCachePolicy` to make it a pure "Functional Core."

-   **Before (Impure)**:
    -   `init(currentDate: () -> Date)`
    -   It had a side-effect (getting the current time), making it non-deterministic and harder to test.

-   **After (Pure & Deterministic)**:
    -   `func validate(_ timestamp: Date, against date: Date) -> Bool`
    -   It's now a pure function. It has no side-effects.
    -   The `LocalFeedLoader` (the "Imperative Shell") is now responsible for the side-effect of getting `Date()` and passing it *into* the pure function.

---

### 3. Entities vs. Value Objects

-   **Value Object (or Value Type)**: A model defined by its *data*, not its *identity*.
    -   **Example**: `Money { let amount: Decimal, let currency: String }`. Two `Money` objects are identical if their `amount` and `currency` are the same.
-   **Entity**: A model defined by its unique *identity*.
    -   **Example**: `User { let id: UUID, var name: String }`. Two `User` objects are identical if their `id` is the same, even if their `name` is different.
-   The `FeedCachePolicy` is a **Value Object** (or just a set of static functions) because it has no state or identity; it just encapsulates rules.

---

### 4. Single Source of Truth (SSOT)

-   **The Problem**: The cache age (`7`) was hardcoded in the `FeedCachePolicy` *and* in multiple tests (e.g., `test_load_failsOn7DaysOldCache`). If this business rule changed to `8` days, we'd have to find and replace it in many places, which is error-prone.
-   **The Solution**: Create a **Single Source of Truth** for this rule. We created a "Tiny DSL" helper:
    -   `Date.minusFeedCacheMaxAge()`
-   **The Benefit**: This helper is now the *only* place the 7-day logic exists. Both the production code (`FeedCachePolicy`) and the tests (`XCTAssert...`) use this helper. If the rule changes, we update this one function, and the entire system (prod + tests) updates automatically.

```Swift
extension Date {
    func minusFeedCacheMaxAge() -> Date {
        return adding(days: -maxFeedCacheAgeInDays)
    }
    
    private var maxFeedCacheAgeInDays: Int {
        return 7
    }
    
    private func adding(days: Int) -> Date {
        return Calendar(identifier: .gregorian).date(byAdding: .day, value: days, to: self)!
    }
}
 
extension Date {
    func adding(seconds: TimeInterval) -> Date {
        return self + seconds
    }
}
```

---

### Senior & Architect Interview Perspective

**Q: "Explain the 'Functional Core, Imperative Shell' pattern."**

> "It's a design pattern that separates pure logic from side-effects.
>
> 1.  The **Functional Core** is the center of your app. It's made of pure, deterministic functions and models that contain your core business rules (e.g., a validation policy). It has no network, database, or UI code. It's easy to test because it's just 'data in, data out.'
> 2.  The **Imperative Shell** is the outer layer. It handles all the 'dirty' work: UI, networking, disk I/O, etc. This layer is *allowed* to have side-effects.
>
> The 'Shell' calls the 'Core' to make decisions. For example, the `LocalFeedLoader` (Shell) gets the current `Date()` (a side-effect) and the `timestamp` from the DB, then passes both *into* the `FeedCachePolicy` (Core) to ask, 'Is this valid?'"

**Q: "I see the number '7' for a 7-day cache in your tests and in your logic. What's the problem with that?"**

> "That's a **"magic number"** and it violates the **Single Source of Truth (SSOT)** principle. That '7' is a business rule, and it's now duplicated. If that rule ever changes to '8', I'm guaranteed to miss one of the locations, and my tests and production code will be out of sync.
>
> The fix is to create a "Tiny DSL" helper, like a `private extension on Date`, that encapsulates this rule (e.g., `Date.feedCacheMaxAge()`). Both the tests and the production logic will then call this one function. This makes the code more readable and ensures that a future change only needs to happen in one single, reliable place."


---
