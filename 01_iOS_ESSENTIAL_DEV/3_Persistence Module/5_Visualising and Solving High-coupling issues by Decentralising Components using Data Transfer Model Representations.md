# Learning Session: Visualizing and Solving High-coupling issues by Decentralizing Components using Data Transfer Model Representations

---

### **Metadata**
- **Date:** `19-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/9433913

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
# Notes: Solving High Coupling by Decentralizing Models

This lesson focuses on fixing the architectural bottleneck of a "unified model" by creating separate data models (DTOs) for each module.

---

### 1. The Problem: Unified vs. Segregated Models

-   **The Problem**: Having one "unified" model (`FeedItem`) that is shared across all modules (API, Cache, UI) creates **high coupling**.
-   **Why it's a Bottleneck**:
    -   A simple change to this model (e.g., adding an API-specific property) forces **all** modules to recompile and change.
    -   Models become "bloated" with properties and methods that only one module needs, leading to confusion and inconsistency.
    -   This is a **"conformist approach,"** where external needs (like the backend's) dictate your app's internal design, making it hard to maintain.

-   <img width="866" height="443" alt="image" src="https://github.com/user-attachments/assets/e73829f9-28a0-468c-9285-cc6441905f8d" />


---

### 2. The Solution: Data Transfer Representations (DTOs)

-   **The Concept**: Each module defines **its own, separate model** that represents *only* what it needs.
    -   **API Module**: `RemoteFeedItem` (reflects the JSON from the backend).
    -   **Cache Module**: `LocalFeedImage` (reflects how data is stored in the database).
    -   **Feature/UI Module**: `FeedImage` (reflects what the UI needs to display).
-   **How it Works**: A **translation layer** (or "Anti-Corruption Layer") is created to map between these models at the boundaries.
    -   When `RemoteFeedLoader` (API) finishes, it maps its `[RemoteFeedItem]` to `[FeedImage]` before returning.
    -   When `LocalFeedLoader` (Cache) saves, it receives `[FeedImage]` and maps them to `[LocalFeedImage]` for storage.
-   **The Benefit**: This **decentralizes** the models. Now, the backend team can change their API (and `RemoteFeedItem`) without breaking the UI module. The modules are **loosely coupled** and can be developed in parallel.

---

### 3. Implementation: Swifty Model Translation

-   A clean, "Swifty" way to implement the translation layer is with a **private extension** on `Array` constrained to a specific model.
-   This keeps the mapping logic private to the module that owns it and neatly co-locates the translation logic.

-   <img width="1081" height="152" alt="image" src="https://github.com/user-attachments/assets/63656b9a-caab-4097-b49f-58ec71d69fef" />

---

### Senior & Architect Interview Perspective

**Q: "Your team's API, Cache, and UI layers all share one `Codable` model. What are the risks, and how would you fix it?"**

> "That's a **"conformist" design** and a major architectural risk. The single model creates high coupling; a change in the API's JSON response will break the UI and Cache.
>
> **The Fix** is to **decentralize the models** by introducing Data Transfer Objects (DTOs) and an **Anti-Corruption Layer**.
>
> 1.  The **API module** has its own `RemoteFeedItem` that matches the JSON.
> 2.  The **Cache module** has its own `LocalFeedItem` that matches the database schema.
> 3.  The **Feature (UI) module** defines its *own* clean `FeedItem` model, representing *its* needs.
>
> The API and Cache modules are then responsible for **translating** their DTOs to the `FeedItem` model. This decouples all modules. The API can now change completely without ever affecting the UI."

**Q: "I see you're using `private extensions` for model mapping. Why not make them `public` for reusability?"**

> "Because that would **leak implementation details**. That specific mapping logic (e.g., `RemoteFeedItem` -> `FeedImage`) is a private responsibility of the API module. No other module should *ever* know about `RemoteFeedItem`.
>
> By keeping it private, we enforce our modular boundaries. It also forces us to **test through the public interface**. We refactored the entire API module's internals (from `FeedItem` to `RemoteFeedItem`) without changing a single test, because our tests only validated the *public output* (`[FeedImage]`), not the private implementation."

**Q: "What do you do when the backend team calls a model 'FeedItem' but the business team calls it 'FeedImage'?"**

> "We establish a **Ubiquitous Language** *per context*.
>
> 1.  Inside the **API module** (the "Anti-Corruption Layer"), we use the backend's term: `RemoteFeedItem`. This makes communication with the backend team clear.
> 2.  Everywhere *outside* that boundary (in our business logic, UI, and cache modules), we use the business's term: `FeedImage`.
>
> This creates clear contexts and prevents technical jargon from leaking into the business domain, which keeps our domain models clean and easy to understand."

---
