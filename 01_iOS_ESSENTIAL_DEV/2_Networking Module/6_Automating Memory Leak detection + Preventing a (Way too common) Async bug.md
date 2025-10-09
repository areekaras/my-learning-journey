# Learning Session: Automating Memory Leak Detection + Preventing a (Way too common) Async bug

---

### **Metadata**
- **Date:** `05-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/8767959

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is clear.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)
*My structured notes, guided by deep questions.*

- [ ] **Embrace the Struggle:** Attempt problems on your own for 15-20 mins before seeking hints.

#### Notes

- Add memory leak detection test inside `tearDownBlock` to ensure the instance is deallocated after the test is completed
  <img width="911" height="405" alt="image" src="https://github.com/user-attachments/assets/bf90d7f5-2c68-4fa6-829a-90b8a3e8790e" />
- Add specific test case to prevent a way too common async bug to ensure the invokation should not happen after the instance has been deallocated
  <img width="1566" height="320" alt="image" src="https://github.com/user-attachments/assets/e281580b-2b03-4935-9081-0da71119d1b5" />


---

### Phase 3: Proof of Learning (After the Session)
*This is where the knowledge becomes permanent. This section is completed from memory.*

#### My "Blank Page" Summary (Feynman Technique)
*(Write your summary of the topic here without looking at your notes from Phase 2. Draw a diagram if it helps.)*

#### Consolidation Checklist
- [ ] **Teach-Back Complete:** I have stood up and explained this concept out loud for 60 seconds.
- [ ] **Next-Day Review Scheduled:** I have added a 5-minute task to my plan for tomorrow to review this note.
- [ ] **Weekly Review Scheduled:** I have added this note to my list for the upcoming "Weekly Knowledge Review" session.
