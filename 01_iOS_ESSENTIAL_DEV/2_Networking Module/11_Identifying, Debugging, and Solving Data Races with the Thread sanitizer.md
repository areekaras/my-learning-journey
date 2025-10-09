# Learning Session: Identifying, Debugging, and Solving Data Races with the Thread Sanitizer

---

### **Metadata**
- **Date:** `09-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/11047340

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

# Data Races & Thread Sanitizer

### Key Concepts

-   **Data Race**: Occurs when multiple threads access the same memory without synchronization, and at least one access is a **write**. This can cause unpredictable behavior and memory corruption.

-   **Thread Sanitizer**: An LLVM tool that detects data races at runtime.
    -   **Performance Cost**: It's resource-intensive (2-20x CPU slowdown, 5-10x memory increase).
    -   **When to Use**: Enable it periodically for local development and **keep it enabled on the CI scheme**.


### Finding & Fixing Data Races

-   **Common Cause**: A combination of **global shared state** (like a static variable) and **background threading** (like `URLSession`'s completion handlers).

-   **The Problem Scenario**: A test finishes, but a background network request from that test continues running. A new test starts and writes to the same global variable that the background task is still reading.

-   <img width="985" height="403" alt="image" src="https://github.com/user-attachments/assets/eb3362f8-1536-40ed-8acd-b0eba5193005" />

-   **The Solution**: Ensure all operations finish before a test method returns. Avoid mutable global state whenever possible.

-   <img width="1183" height="385" alt="image" src="https://github.com/user-attachments/assets/3317dece-67b4-4486-85ad-ca74ba01cbae" />


### Best Practices

-   Run the Thread Sanitizer locally before pushing to CI to catch issues faster.
-   Be aware of the performance impact on CI servers.
-   Use the **Dispatch** framework (e.g., queues, locks) to synchronize memory access when shared state between threads is unavoidable.


---

### Phase 3: Proof of Learning (After the Session)
*This is where the knowledge becomes permanent. This section is completed from memory.*

#### My "Blank Page" Summary (Feynman Technique)
*(Write your summary of the topic here without looking at your notes from Phase 2. Draw a diagram if it helps.)*

#### Consolidation Checklist
- [ ] **Teach-Back Complete:** I have stood up and explained this concept out loud for 60 seconds.
- [ ] **Next-Day Review Scheduled:** I have added a 5-minute task to my plan for tomorrow to review this note.
- [ ] **Weekly Review Scheduled:** I have added this note to my list for the upcoming "Weekly Knowledge Review" session.
