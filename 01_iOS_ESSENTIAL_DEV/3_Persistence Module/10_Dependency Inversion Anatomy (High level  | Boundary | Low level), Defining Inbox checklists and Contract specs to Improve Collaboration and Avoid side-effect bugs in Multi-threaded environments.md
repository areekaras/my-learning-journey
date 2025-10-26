# Learning Session: Dependency Inversion Anatomy (High level | Boundary | Low level), Defining Inbox Checklists and Contract Specs to Improve Collaboration and Avoid side-effect bugs in Multi-threaded Environments

---

### **Metadata**
- **Date:** `21-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/9858190

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
# Notes: Dependency Inversion Anatomy & Specs as Contracts

This lesson covers the three core components of the Dependency Inversion Principle (DIP) and how to use technical specifications as a "contract" to ensure any implementation will work as expected.

---

### 1. The Anatomy of Dependency Inversion (DIP)

DIP is the foundation for modular, decoupled systems. It's built on three roles:

1.  **High-Level Component (The "Why")**:
    * The core business logic or use case (e.g., `LocalFeedLoader`).
    * It should have **zero references** to low-level details.

2.  **Low-Level Component (The "How")**:
    * The infrastructure implementation or "delivery mechanism" (e.g., `CoreDataFeedStore`, `URLSessionHTTPClient`).
    * This is the "dirty" code that talks to frameworks.

3.  **Boundary (The "Abstraction")**:
    * The protocol or closure that connects the two (e.g., `<FeedStore>`, `<HTTPClient>`).
    * This abstraction is **owned by the High-Level component**.

**How it works**: The High-Level component depends *only* on the Boundary. The Low-Level component *also* depends on the Boundary (by conforming to it). This **inverts** the traditional dependency: the infrastructure details now depend on the business logic's requirements, not the other way around.

-   <img width="3806" height="2510" alt="image" src="https://github.com/user-attachments/assets/bae4e1fa-8715-41d6-9147-5478b189ac68" />
-   <img width="5944" height="3637" alt="image" src="https://github.com/user-attachments/assets/9d7542a1-2018-48b4-b77a-e6a14fe5dc47" />
-   <img width="2000" height="1008" alt="image" src="https://github.com/user-attachments/assets/08370269-3874-4804-8bdc-4e725abadd59" />

---

### 2. Specs as Contracts

"Specs" (specifications) are a contract that defines *what* a component must do, without defining *how*. This enables parallel development.

-   **Use Cases** are contracts for your internal team.
-   **API Payloads** are contracts between your team and the backend team.
-   **`FeedStore` Specs** are a technical contract for *any* future persistence implementation (Core Data, Realm, File System, etc.).

This contract ensures that any developer building a new `FeedStore` implementation will create one that is 100% compatible with your application's business logic.

---

### 3. `FeedStore` Contract Specs (The Inbox Checklist)

This is the technical contract for any `FeedStore` implementation.

```
- Insert
    - To empty cache works
    - To non-empty cache overrides previous value
    - Error (if possible to simulate, e.g., no write permission)

- Retrieve
    - Empty cache works (before something is inserted)
    - Non-empty cache returns data
    - Non-empty cache twice returns same data (retrieve should have no side-effects)
    - Error (if possible to simulate, e.g., invalid data)

- Delete
    - Empty cache does nothing (cache stays empty and does not fail)
    - Inserted data leaves cache empty
    - Error (if possible to simulate, e.g., no write permission)

- Side-effects must run serially to avoid race-conditions (deleting the wrong cache... overriding the latest data...)
```

---

### Senior & Architect Interview Perspective

**Q: "Can you explain the Dependency Inversion Principle? What are its components?"**

> "Absolutely. DIP is what stops your business logic from being coupled to infrastructure. It has three parts:
>
> 1.  The **High-Level Component**: This is your business logic, like a `LocalFeedLoader`.
> 2.  The **Low-Level Component**: This is the infrastructure detail, like a `CoreDataStore`.
> 3.  The **Boundary**: This is an abstraction, like a `<FeedStore>` protocol, which is *owned* by the High-Level component.
>
> The 'inversion' is that the `CoreDataStore` (Low-Level) is forced to depend on and conform to the `<FeedStore>` protocol (Boundary) defined by the business logic. This decouples the 'what' from the 'how' and lets me swap `CoreDataStore` for a `RealmStore` without the business logic ever knowing."

**Q: "Your team needs to build a new persistence layer. How do you ensure any implementation they build (Core Data, Realm, etc.) will work correctly with your app?"**

> "I'd start by writing **"Specs as a Contract."** I would create a suite of abstract tests (we call it an "Inbox Checklist") that define the *behavioral contract* of the persistence layer. This includes things like: 'retrieve must have no side-effects,' 'insert must override previous data,' and critically, 'all side-effects must be serial to prevent race conditions.'
>
> Any developer building a new implementation (Core Data, Realm, etc.) must run and pass this shared test suite. This guarantees their implementation conforms to the contract our business logic expects."

**Q: "The `FeedStore` contract says 'Side-effects must run serially.' Why is this so important, and what bug does it prevent?"**

> "That's one of the most critical parts of the contract. It's there to prevent **race conditions**.
>
> Imagine you have two operations happening at nearly the same time: 1) `deleteCache()` and 2) `saveCache(...)`. Without a serial queue, they could interleave. The `saveCache` might write its data, and *then* the `deleteCache` runs, incorrectly wiping out the new data.
>
> By enforcing that all side-effects are serial, we guarantee that one operation finishes completely before the next one begins, which ensures a deterministic and safe cache."

---
