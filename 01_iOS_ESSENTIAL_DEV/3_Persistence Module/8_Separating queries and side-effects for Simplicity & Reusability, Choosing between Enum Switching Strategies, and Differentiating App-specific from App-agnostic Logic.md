# Learning Session: Separating Queries and Side-effects for Simplicity & Reusability, Choosing between Enum Switching Strategies, and Differentiating App-specific from App-agnostic Logic

---

### **Metadata**
- **Date:** `20-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/9677292

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

# Notes: Separating Queries & Side-Effects (CQS)

This lesson covers the practical application of the Command-Query Separation (CQS) principle, separating the `load` (Query) and `validateCache` (Command) logic into two distinct use cases.

---

### 1. Applying Command-Query Separation (CQS)

-   **The Problem**: The `load()` method was doing two things: retrieving a feed (**Query**) and deleting the cache if invalid (**Command**). This is a CQS violation.
-   **The Solution**: We refactored this into two separate methods, representing two distinct Use Cases:
    1.  `load()`: Is now a pure **Query**. It *only* retrieves the cache. It has **no side-effects**.
    2.  `validateCache()`: Is now a pure **Command**. It *only* checks the cache and deletes it if invalid. It **returns `Void`**.
-   **The Benefit**: This separation makes the code simpler and more **reusable**. For example, we can now run `validateCache()` on an app background-fetch timer, without being forced to also perform a `load()`.
-   **Testing**: After refactoring, we add new tests to the `load()` method to *assert that it has no side-effects* (i.e., it *doesn't* delete the cache). This serves as documentation and prevents future regressions.

-   <img width="823" height="862" alt="image" src="https://github.com/user-attachments/assets/8805c45e-470f-45fb-b9de-dc6f89dc300e" />
-
```Swift
extension LocalFeedLoader: FeedLoader {
    public typealias LoadResult = LoadFeedResult

    public func load(completion: @escaping (LoadResult) -> Void) {
        store.retrieve { [weak self] result in
            guard let self = self else { return }
            switch result {
            case let .failure(error):
                completion(.failure(error))
                
            case let .found(feed, timestamp) where self.validate(timestamp):
                completion(.success(feed.toModels()))
                
            case .found, .empty:
                completion(.success([]))
            }
        }
    }
}

extension LocalFeedLoader{
    public func validateCache() {
        store.retrieve { [weak self] result in
            guard let self = self else { return }
            
            switch result {
            case .failure:
                self.store.deleteCachedFeed { _ in }
                
            case let .found(_, timestamp) where !self.validate(timestamp):
                self.store.deleteCachedFeed { _ in }
                
            case .found, .empty: break
            }
        }
        
    }
}
```

---

### 2. `switch` vs. `default` vs. `@unknown default`

When switching on an `enum`, you have three choices.

1.  **Explicit Cases (Preferred for your own enums)**:
    -   `case .a: ...`
    -   `case .b: ...`
    -   **Pro**: If a new case (`.c`) is added to the `enum`, the compiler generates a **build error**. This is good, as it forces you to consciously handle the new case.
    -   **Con**: Violates the Open-Closed principle.

2.  **`default` Case (Avoid)**:
    -   `case .a: ...`
    -   `default: ...`
    -   **Pro**: Code doesn't break if a new case is added.
    -   **Con**: You get **no notification** (no error, no warning) that a new case was added, which can lead to silent bugs.

3.  **`@unknown default` (Best for 3rd-party enums)**:
    -   `case .a: ...`
    -   `@unknown default: ...`
    -   **Pro**: The best of both worlds. If a new case is added (especially from an external framework you don't control), you get a **compiler warning** instead of an error. This reminds you to handle the new case without breaking your build.

---

### 3. Application-Specific vs. Application-Agnostic Logic

This is a key architectural concept for reusability.

-   **Application-Specific Logic (The "Use Case")**:
    -   This is the "controller" or "interactor" logic that orchestrates a specific feature.
    -   Example: `LocalFeedLoader` *uses* other objects to achieve a goal. It is specific to *this* application's needs.

-   **Application-Agnostic Logic (The "Domain Model")**:
    -   This is the core business logic or "rules" that are true regardless of the application.
    -   Example: The *rule* "a cache is valid if it's less than 7 days old" is a pure, reusable piece of logic. It's application-agnostic.
    -   **The Goal**: We should extract this "Agnostic" logic (the validation policy) *out* of the `LocalFeedLoader` (the "Specific" logic) and into its own reusable Domain Model.

---

### Senior & Architect Interview Perspective

**Q: "How do you decide whether to use an explicit `switch` or a `default` case for an enum?"**

> "My choice depends on who *owns* the enum.
>
> 1.  For **enums I own** (defined within my module), I always use **explicit cases** and avoid `default`. This is a defensive practice. If my team adds a new case, I *want* the compiler to produce a build error, as it forces us to consciously decide how to handle that new state.
> 2.  For **enums from external frameworks** (like Apple's or other 3rd-party SDKs), I *never* use a hard `default`. I use **`@unknown default`**. This gives us resilience against the SDK adding new cases—our code will still compile, but we'll get a *warning* to revisit our logic."

**Q: "What's the difference between Application-Specific and Application-Agnostic logic?"**

> "This is a key concept for building a reusable architecture.
>
> **Application-Agnostic** logic is the pure, core "Domain Rules" of the business. For example, 'a password must be 8 characters' or 'a cache expires in 7 days.' This logic is universal and should live in a "Domain Model" that has zero dependencies on the app.
>
> **Application-Specific** logic is the "Use Case" or "Interactor" that *uses* those rules to perform a feature. For example, a `LoginUseCase` *uses* the password rule, and a `FeedLoader` *uses* the cache expiry rule.
>
> By separating them, I can reuse my core Domain Models across multiple apps or features, and my Use Cases become simple coordinators that are easy to test."

---
