# Learning Session: Perform Calendrical Calculations Correctly, Dealing with Coincidental Duplication while respecting the DRY Principle, Decoupling Tests from Implementation with Tiny DSLs, and Test Triangulation to increase coverage and confidence

---

### **Metadata**
- **Date:** `19-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/9500765

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
# Notes: Calendrical Calculations, DRY, and Test Triangulation

This lesson covers how to correctly handle date logic and write more robust, decoupled tests by identifying different types of duplication and "triangulating" test values.

---

### 1. DRY vs. Coincidental Duplication

-   **DRY (Don't Repeat Yourself)**: A principle to avoid duplicating the same logic in multiple places.
-   **Coincidental Duplication (A Trap)**: Code that *looks* the same but represents completely different concepts or use cases.
-   **The Problem**: If you merge "coincidental duplication," you tightly couple two different concepts. When one concept needs to change, you accidentally break the other.
-   **Example**: The setup code for `test_save()` and `test_load()` might look similar, but they are testing two separate use cases (Save vs. Load). It's correct to keep them separate, as it makes the tests clearer and easier to refactor if the Use Cases are ever split into different types.

---

### 2. Performing Calendrical Calculations Correctly

-   **The Problem**: Naive date math (e.g., `60 * 60 * 24 * 7`) is unreliable. It fails to account for edge cases like **daylight saving time**, where a "day" is not exactly 24 hours.
-   **The Risk**: This can cause critical, hard-to-debug bugs (e.g., cache expiring at the wrong time, users missing flights) and makes tests flaky, as they can fail when run in different time zones or locales (like on a CI server).
-   **The Solution**: **Always use Apple's `Calendar` API** for calendrical calculations (e.g., `Calendar.current.date(byAdding:)`). It correctly handles all time zone and locale complexities.

---

### 3. Decoupling Tests with a "Tiny DSL"

-   **DSL (Domain Specific Language)**: Creating simple, readable helper methods for your tests.
-   **The Goal**: To make tests more expressive and **decouple them from implementation details**.
-   **Example**: Instead of writing complex `Calendar.current.date(byAdding: ...)` logic *inside* every test, you create a "Tiny DSL" by extending `Date`:

    -   <img width="1047" height="213" alt="image" src="https://github.com/user-attachments/assets/fbda7dd6-a669-4852-977b-950b37d78f63" />

-   **The Benefit**: The tests become highly readable (e.F., `timestamp.adding(days: -7)`). More importantly, if you ever need to change *how* you calculate dates, you only change the `Date` extension; **none of the tests need to be modified**.

---

### 4. Test Triangulation

-   **The Concept**: To increase confidence and cover edge cases, you "triangulate" your tests around a specific requirement.
-   **Example Requirement**: "Cache must be *less than* 7 days old."
-   **The Data Points to Test (The Triangulation)**:
    1.  **On the boundary**: `7 days old` (should be invalid).
    2.  **Just inside the boundary**: `less than 7 days old` (e.g., 6 days and 23:59:59) (should be valid).
    3.  **Just outside the boundary**: `more than 7 days old` (e.g., 7 days and 00:00:01) (should be invalid).
-   **The Benefit**: This process forces you to test the hard limits and adjacent values, which is where "off-by-one" errors commonly occur. As a senior developer, this is part of your role as a "risk manager"—finding and testing the edge cases the business might not have specified.

---

### Senior & Architect Interview Perspective

**Q: "Your test for a 7-day cache expiry fails, but only on the CI server in a different time zone. What's your first guess for the cause?"**

> "My first guess is **naive date math**. The code is almost certainly doing manual calculations, like `60*60*24*7`, instead of using Apple's `Calendar` API. `Calendar` correctly handles all time zone and daylight saving time complexities, while manual math doesn't. This is a classic bug that only appears when code is run in different locales."

**Q: "I see two test setup methods in your test suite that look identical. Should you merge them to follow the DRY principle?"**

> "Not necessarily. I'd first investigate if it's **coincidental duplication**. If `TestA` and `TestB` are for two completely different use cases, like 'Save' and 'Load', then that setup code *represents* two different contexts, even if it *looks* the same. Merging them would tightly couple two unrelated tests. I'd rather have a little duplication if it means my tests remain decoupled, clear, and focused on a single responsibility."

**Q: "How do you ensure your tests are robust when testing against a boundary, like a 7-day limit?"**

> "I use **test triangulation**. For a 'less than 7 days' rule, I don't just test one 'valid' and one 'invalid' case. I test three points:
>
> 1.  **At 7 days** (the boundary, should be invalid)
> 2.  **At 6 days, 23 hours, 59 mins** (just inside the boundary, should be valid)
> 3.  **At 7 days, 1 second** (just outside the boundary, should be invalid)
>
> This process finds 'off-by-one' errors and proves the boundary logic is precise."

**Q: "What's a 'Tiny DSL' in the context of testing, and why is it valuable?"**

> "A 'Tiny DSL' is just a set of helper methods, often as `private` extensions, that make test code more readable. For example, `date.adding(days: -7)` is a DSL. Its value is **decoupling**: the test now describes its *intent* and is no longer coupled to the *implementation* (the `Calendar` API). If we need to change that calculation, we change the helper in one place, and no tests are broken."

---
