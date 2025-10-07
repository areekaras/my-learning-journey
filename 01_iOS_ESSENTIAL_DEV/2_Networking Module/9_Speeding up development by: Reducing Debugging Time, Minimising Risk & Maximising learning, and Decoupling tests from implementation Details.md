# Learning Session: Speeding up Development by: Reducing Debugging Time, Minimising Risks & Maximizing Learning, and Decoupling Tests from Implementation Details

---

### **Metadata**
- **Date:** `07-10-2025`, `08-10-2025`
- **Source:** (e.g., "Essential Dev, Networking Module, Video 3")

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [ ] **Goal Set:** My specific goal for this session is clear.
- [ ] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [ ] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)
*My structured notes, guided by deep questions.*

- [ ] **Embrace the Struggle:** Attempt problems on your own for 15-20 mins before seeking hints.

#### Notes

---
- Not much notes on this lesson.
- We covered all cases for the all combinations of (data?, response?, Error?)
- valid scenarios are (anyData, HTTPURLResponse, nil) or (nil, HTTPURLResponse, nil) - The URL Loading System returns with empty data (0 bytes) in the second case
- All other scenarios are invalid and add tests to return as failure

# Maximizing Test Value

### Improving Test Feedback

-   **Goal**: Tests must provide **fast, reliable, and accurate feedback** to avoid time-consuming debugging.

-   **Precise Failures**: A good test failure message immediately reveals the problem's root cause without needing to debug.

-   **Single Responsibility**: A test should check only **one thing**. If a test fails with an unrelated error, split it to assert each scenario in isolation (e.g., one test for the URL, another for the error).

-   **Assertion Helpers**: Pass `file: StaticString = #filePath` and `line: UInt = #line` to custom assertion helpers so Xcode highlights the correct failure location.


### Testing 3rd-Party Frameworks

-   **Manage Risk**: Write tests to validate your assumptions about frameworks you don't own (e.g., `URLSession`).

-   **Catch Breaking Changes**: These tests act as a safety net, notifying you when a framework update (like an OS change) alters behavior.

-   **Example (iOS 14+ `URLSession`)**: `URLSession` started wrapping errors, so direct error equality checks (`XCTAssertEqual`) began to fail. Tests revealed this change, prompting a switch to comparing error `domain` and `code` instead.


---

### Phase 3: Proof of Learning (After the Session)
*This is where the knowledge becomes permanent. This section is completed from memory.*

#### My "Blank Page" Summary (Feynman Technique)
*(Write your summary of the topic here without looking at your notes from Phase 2. Draw a diagram if it helps.)*

#### Consolidation Checklist
- [ ] **Teach-Back Complete:** I have stood up and explained this concept out loud for 60 seconds.
- [ ] **Next-Day Review Scheduled:** I have added a 5-minute task to my plan for tomorrow to review this note.
- [ ] **Weekly Review Scheduled:** I have added this note to my list for the upcoming "Weekly Knowledge Review" session.
