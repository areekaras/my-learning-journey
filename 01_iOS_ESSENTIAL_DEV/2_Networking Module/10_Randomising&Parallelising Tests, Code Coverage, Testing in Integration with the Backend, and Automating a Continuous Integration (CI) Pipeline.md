# Learning Session: Randomising & Parallelising Tests, Code Coverage, Testing in Integraton with the Backend, and Automating a Continuous Integration (CI) Pipeline 

---

### **Metadata**
- **Date:** `08-10-2025`, `09-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/9070355

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

# Notes: CI, E2E Tests, and Test Configurations

### Test Scheme Configurations

-   **Run Tests in Random Order**: **Essential practice.** It exposes order-dependent tests and proves that each test is self-contained and independent.

-   **Parallelize Test Execution**: Speeds up large, slow test suites. It's often unnecessary if you maintain fast unit tests.

-   **Gather Code Coverage**: Use this metric to see which lines of code are **not tested**. 100% coverage **does not** guarantee correct behavior or the absence of bugs.


### End-to-End (E2E) vs. Unit Tests

-   **Economics**: E2E tests are **slow, unreliable, and costly** to run and maintain compared to fast, precise, and reliable unit tests.

-   **When to Use E2E Tests**: Minimize them. Their primary value is to validate integration with external systems where the **contract is not guaranteed** (e.g., a server API that might change without notice). The less you trust the other system, the more E2E tests you need.


### Continuous Integration (CI)

-   **What it is**: The practice of frequently merging all developers' work into the main branch, often called "Trunk Based Development".

-   **The Goal**: An automated CI pipeline runs all tests before a merge, guaranteeing that no change breaks the main build or introduces regressions.

-   **Why You Need It**: It saves time, reduces merge conflicts, and improves codebase integrity. It is valuable even for a solo developer and is the first step towards Continuous Delivery/Deployment.


### CI Pipeline Configuration

-   **Location**: The CI configuration file is located at `.github/workflows/CI.yml` in the project's root directory.

-   **File Content**:
    ```yml
    name: CI

    on:
      push:
        branches: [ "main" ]
      pull_request:
        branches: [ "main" ]

    jobs:
      build-and-test:
        runs-on: macos-latest
        timeout-minutes: 8
        steps:
          - uses: actions/checkout@v4
          - name: Select Xcode
            run: sudo xcode-select -switch /Applications/Xcode_16.0.app
          - name: Build and test
            run: xcodebuild clean build test -project EssentialFeed/EssentialFeed.xcodeproj -scheme "CI" CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO
    ```

-   **YAML File Explanation**: This configuration file defines a CI pipeline named "CI" that **triggers** on any **push** or **pull request** to the **main branch**. It runs a single **job** on the latest macOS runner, which **checks out the code**, selects a specific Xcode version, and then executes a command-line build and test using `xcodebuild`, ensuring that code signing is disabled for the CI environment.


---

### Phase 3: Proof of Learning (After the Session)
*This is where the knowledge becomes permanent. This section is completed from memory.*

#### My "Blank Page" Summary (Feynman Technique)
*(Write your summary of the topic here without looking at your notes from Phase 2. Draw a diagram if it helps.)*

#### Consolidation Checklist
- [ ] **Teach-Back Complete:** I have stood up and explained this concept out loud for 60 seconds.
- [ ] **Next-Day Review Scheduled:** I have added a 5-minute task to my plan for tomorrow to review this note.
- [ ] **Weekly Review Scheduled:** I have added this note to my list for the upcoming "Weekly Knowledge Review" session.
