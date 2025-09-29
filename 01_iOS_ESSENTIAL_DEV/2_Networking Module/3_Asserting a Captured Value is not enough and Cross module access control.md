# Learning Session: Asserting a Captured value is not enough + Cross module Access Control

---

### **Metadata**
- **Date:** `24-09-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/8575843
---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is clear.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)
*My structured notes, guided by deep questions.*

#### Notes

- Internal types are not visible to external modules
- @testable makes the internal types visible to the test target
- A better approach, when possible to test the module through public interfaces, So we can test the behavious as a client of the module
- Benefit: We are free to change internal and private implementation details without breaking the tests
- When testing objects collaborating, asserting the values passed in not enough
- We also need to ask "how many times was the method invoked?"

---

### Phase 3: Proof of Learning (After the Session)
*This is where the knowledge becomes permanent. This section is completed from memory.*

#### My "Blank Page" Summary (Feynman Technique)
*(Write your summary of the topic here without looking at your notes from Phase 2. Draw a diagram if it helps.)*

#### Consolidation Checklist
- [ ] **Teach-Back Complete:** I have stood up and explained this concept out loud for 60 seconds.
- [ ] **Next-Day Review Scheduled:** I have added a 5-minute task to my plan for tomorrow to review this note.
- [ ] **Weekly Review Scheduled:** I have added this note to my list for the upcoming "Weekly Knowledge Review" session.
