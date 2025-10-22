# Learning Session: Deleting Models and Handling error with Codable+FileSystem, Making Async code Look sync in Tests to Eliminate Arrow Anti-pattern, and More Essential Test Guidelines to Improve Code Quality and Team Communication  

---

### **Metadata**
- **Date:** `22-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/10006695

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

# Notes: Refactoring Tests for Readability and Communication

This lesson focuses on refactoring test code to be more expressive, less complex, and easier to maintain. The key techniques are turning asynchronous code into synchronous helpers and making all test dependencies explicit.

---

### 1. Turning Async Tests into Sync Helpers

-   **The Problem**: Asynchronous tests using `XCTestExpectation` and completion closures are verbose, hard to read, and lead to "arrow-shaped code" (deep nesting).

-   **The Solution**: Create **synchronous helper methods** in your test target. These helpers *wrap* the asynchronous SUT (System Under Test) methods.
    -   **Signature Change**:
        -   *Before (Async)*: `func asyncFunction(completion: @escaping (Result) -> Void)`
        -   *After (Sync Wrapper)*: `func syncFunction() -> Result`
    -   **Mechanism**: The helper method creates an `XCTestExpectation`, calls the async function, `wait(for:)` the expectation, and then `return`s the captured result.

-   **The Benefits**:
    1.  **Readability**: Tests become linear, clean, and simple (e.g., `let result = syncFunction()`).
    2.  **Maintainability**: The complexity of handling asynchronicity is hidden in one place (the helper) instead of being repeated in every test.
    3.  **Decoupling**: The tests are now decoupled from the *implementation detail* of asynchronicity.
 
```Swift
    func test_insert_overridesPreviouslyInsertedCacheValues() {
        let sut = makeSUT()
        
        let firstInsertionError = insert((uniqueImageFeed().local, Date()), to: sut)
        XCTAssertNil(firstInsertionError, "Expected to insert cache successfully")
        
        let latestFeed = uniqueImageFeed().local
        let latestTimestamp = Date()
        let latestInsertionError = insert((latestFeed, latestTimestamp), to: sut)
        
        XCTAssertNil(latestInsertionError, "Expected to override cache successfully")
        
        expect(sut, toRetrieve: .found(feed: latestFeed, timestamp: latestTimestamp))
    }

// Helpers which makes the async code to above sync usage

    @discardableResult
    private func insert(_ cache: (feed: [LocalFeedImage], timestamp: Date), to sut: FeedStore) -> Error? {
        let exp = expectation(description: "Wait of cache insertion")
        var insertionError: Error?
        sut.insert(cache.feed, timestamp: cache.timestamp) { receivedInsertionError in
            insertionError = receivedInsertionError
            exp.fulfill()
        }
        wait(for: [exp], timeout: 1.0)
        return insertionError
    }

```

---

### 2. Eliminating "Arrow Code" (The Pyramid of Doom)

-   **The Problem**: Deeply nested code (closures, `if`s, `switch`es) is complex, hard to read, and difficult to maintain. This is the "arrow-shaped code" anti-pattern.
-   **The Refactoring**: By extracting test logic into the synchronous helpers, we dramatically reduced this complexity.
-   **The Measured Impact**:
    -   **Total Lines of Test Code**: Reduced from 85 to 20 (a 4x improvement).
    -   **Total Indentation Levels**: Reduced from 16 to 4 (a 4x improvement).
-   **The "Broken Windows" Effect**: Messy code encourages more messy code. Keeping tests tidy encourages the whole team to maintain a high-quality standard.

-   <img width="1500" height="659" alt="image" src="https://github.com/user-attachments/assets/92030602-3248-422b-818f-6185f119a8b8" />
-   <img width="1500" height="584" alt="image" src="https://github.com/user-attachments/assets/8248a9db-0388-4dcc-80e1-df7907782bbb" />

---

### 3. Making Test Dependencies Explicit

-   **The Principle**: Good tests are clear and transparent. They follow a **Given / When / Then** structure where all important values are explicit.
-   **The Problem ("Before")**: The `makeSUT()` helper *implicitly* used a `testSpecificStoreURL()`. The test's "When" block *explicitly* used the same `testSpecificStoreURL()` to corrupt the file. The connection between the SUT's file and the corrupted file was *implicit* and unclear.

-   <img width="1500" height="534" alt="image" src="https://github.com/user-attachments/assets/2677af30-74ba-4975-a670-b4143f4e4d25" />

-   **The Solution ("After")**:
    1.  **Given**: Explicitly create `let storeURL = testSpecificStoreURL()`.
    2.  **Given**: Explicitly **inject** that URL: `let sut = makeSUT(storeURL: storeURL)`.
    3.  **When**: Explicitly use the *same* `storeURL` variable to perform the action (e.g., corrupt the file).

-   **The Benefit**: The test is now transparent. It's obvious to any reader that the file being acted upon is the *exact same file* the SUT is using.

-   <img width="1500" height="372" alt="image" src="https://github.com/user-attachments/assets/95247551-6af1-4e42-a827-5652a3dcbb06" />

---

### Senior & Architect Interview Perspective

**Q: "Your project's tests are full of `XCTestExpectation` and `wait(for:)`. Isn't this verbose and hard to read?"**

> "Yes, it is. That's a common testing anti-pattern that leads to 'arrow code'—deeply nested and hard-to-maintain tests.
>
> My approach is to refactor this by creating **synchronous test helpers** in the test target. For an async `sut.insert(...)` function, I'll create a *sync* helper `func insert(...) -> Error?`. This helper is the *only* place that manages the `expectation` and `wait` logic.
>
> My tests then become clean, linear, and readable, like `let error = insert(...)` followed by `XCTAssertNil(error)`. It decouples the test's *intent* from the async *implementation*."

**Q: "What makes a test 'easy to read' for you, and why does it matter?"**

> "A test is readable if it's transparent and follows a strict **Given / When / Then** structure. A common anti-pattern I fix is *implicit setup*.
>
> For example, if a test needs to corrupt a file on disk, the test *must* explicitly create the `storeURL` in the 'Given' block, explicitly **inject** that *same URL* into the SUT (System Under Test), and then use that *same URL* in the 'When' block to perform the corruption.
>
> This matters because **tests are documentation**. An explicit test clearly communicates the cause-and-effect relationship to any developer, making failures easy to diagnose and the component's behavior easy to understand."

**Q: "Why spend time refactoring tests? If they pass, who cares if they are messy?"**

> "I care deeply, because **tests are the primary communication tool** for a component's behavior. Messy tests create a 'broken windows' effect—they signal that quality isn't valued and encourage other developers to be messy.
>
> By refactoring tests to be simple, clean, and fast, I lower the cognitive load for the whole team. On this project, this exact refactoring reduced our test code lines and indentation levels by over 4x. This isn't just 'tidying up'; it's a direct investment in the team's long-term speed and the codebase's maintainability."

---
