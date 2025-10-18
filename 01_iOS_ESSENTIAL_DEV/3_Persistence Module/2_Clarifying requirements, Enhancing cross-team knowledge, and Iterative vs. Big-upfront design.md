# Learning Session: Clarifying requriements, Enhancing cross-domain knowledge, and Iterative vs. Big-upfront Design

---

### **Metadata**
- **Date:** `18-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/9230278

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

# Notes: Requirements, Domain Knowledge, & Iterative Design


### 1. Iterative Design vs. Big Up-Front Design (BUFD)

-   **The Problem with BUFD**: Trying to define 100% of requirements at the start is a trap. It leads to **analysis-paralysis** (getting stuck in planning, delivering nothing) and is inflexible, as requirements *always* change.
-   **The Solution (Iterative Design)**:
    -   Take small, incremental steps.
    -   Deliver value quickly in "slices" to gather fast feedback.
    -   Learn, adapt, and refine the requirements continually.
-   **Key Takeaway**: This process is cheaper and more effective. Refining a diagram or use case is low-cost, but fixing a flawed assumption in code is expensive.


### 2. Enhancing Cross-Domain Knowledge

-   **Core Concept**: Software development is a **social activity**, not just a technical one. We must collaborate with **domain experts** (e.g., the business, product, or finance teams).
-   **Why It Matters**: If we *only* talk to other tech people, our understanding of the business need is "naive." We'll build software that *works* but doesn't solve the *real* problem.
-   **Your Role**: Be proactive. Join planning meetings to help non-technical colleagues understand what's possible and to help you understand the *why* behind the *what*.


### 3. Establishing a "Ubiquitous Language"

-   **The Problem**: The business team says "Image Feed," but the backend team's API returns "Feed Items." This creates confusion, slows down communication, and adds cognitive overhead for developers.
-   **The Solution (from Domain-Driven Design)**: Fight for a **Ubiquitous Language**. This means the *same terms* are used by everyone:
    -   In business meetings.
    -   In the UI.
    -   In the code (e.g., class names, variables).
    -   In the API and database.
-   **Example**: If the business calls it a "Customer," the class should be `Customer`, not `UserObject` or `ClientModel`.


### Senior & Architect Interview Perspective

This is how you talk about these "soft skills" to demonstrate seniority.

**Q: "How do you handle vague or changing requirements?"**

> "I avoid the 'big up-front design' trap, as it leads to analysis-paralysis. Instead, I advocate for an **iterative process**. We work with the product team to define a small, valuable slice, build it, get fast feedback, and then refine the next slice. This ensures we're always building the right thing.
>
> To do this effectively, I focus on building a **Ubiquitous Language** with the domain experts, ensuring our code (e.g., `ImageFeedItem`) directly reflects the business terminology. This alignment is critical.
>
> Ultimately, my goal is to build a **modular, decoupled architecture**. This is *how* we welcome changes. When the architecture is flexible, a requirement change is a small, safe adjustment, not a large, risky rewrite."

---
