# Learning Session: Download the Starter Project (+ Automated Testing Setup Guide)

---

### **Metadata**
- **Date:** `14-09-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/9683102

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is setting up starter project and understand Automated Testing set up guide
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)
*My structured notes, guided by deep questions.*

- [x] **Embrace the Struggle:** Attempt problems on your own for 15-20 mins before seeking hints.

#### Notes
- If selected 'Include Unit tests' while creating a project, Xcode will automatically include Test target and Test folder with the product name suffixed with "Tests"
- Scheme is needed to run tests within Xcode
- A Scheme is a collection of settings which specify the targets to build for a project, build configuration to use, and the executable enviroment to use when the product is launched
- Think of scheme as instructions (recipe steps) for building, running, testing, profiling, analysing and archiving for your project
- When you open a project (or create a new one), Xcode automatically create a scheme for each target
- The default scheme is named after your project and will already include your test target if you checked the Include Unit Test checkbox when creating your project.
- If you’re adding a test target to an existing project, you may need to configure your default scheme to run the tests.
- To run a single test, click on the gutter button next to the function declaration corresponding to the test you’d like to run.
- To run all tests in a test case class, click on the gutter button next to the test case class declaration.
- CMD+U for run all tests with in the targets in the selected scheme
- For successful tests run, Xcode will notify you by displaying a Test Succeeded HUD and turning the gutter buttons to a green checkmark
- For failed tests run, Xcode will notify you by displaying a Test Failed HUD and turning the gutter buttons to a red x
- Run build with xcodebuild (CLI) - `xcodebuild test -project [ProjectName].xcodeproj -scheme "[Scheme name]"


---

### Phase 3: Proof of Learning (After the Session)
*This is where the knowledge becomes permanent. This section is completed from memory.*

#### My "Blank Page" Summary (Feynman Technique)
*(Write your summary of the topic here without looking at your notes from Phase 2. Draw a diagram if it helps.)*

#### Consolidation Checklist
- [ ] **Teach-Back Complete:** I have stood up and explained this concept out loud for 60 seconds.
- [ ] **Next-Day Review Scheduled:** I have added a 5-minute task to my plan for tomorrow to review this note.
- [ ] **Weekly Review Scheduled:** I have added this note to my list for the upcoming "Weekly Knowledge Review" session.
