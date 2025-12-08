# Learning Session: Test-driven approach to Deal with Legacy code (code with no tests!) + Extracting cross-platform components from Platform specific code

---

### **Metadata**
- **Date:** `05-12-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/11752012

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is clear.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)


#### Notes

PR: 
1. https://github.com/areekaras/EssentialFeed/pull/20
2. https://github.com/areekaras/EssentialFeed/pull/21

---

# Notes: TDD for Legacy Code & Extracting Cross-Platform Components

This lecture covers two critical senior skills: safely extracting reusable components from a platform-specific module, and using a test-driven approach to add tests to "Legacy Code" (code without tests).

---

### 1. The Goal: Cross-Platform Reusability

-   **The Situation**: We implemented our **Presentation Layer** (`FeedPresenter`, `FeedViewModel`) inside the iOS-specific `EssentialFeediOS` module. These components were `internal` and tested *implicitly* through the iOS Integration Tests.
-   **The Problem**: Presentation logic is often **platform-agnostic**. We want to reuse `FeedPresenter` on macOS, watchOS, or tvOS. It shouldn't be locked inside an iOS framework.
-   **The Solution**: Move the Presentation components into a **cross-platform module** (like `EssentialFeed` or a dedicated `FeedPresentation` module).
-   **The Consequence**: To be usable by other modules, these types must become `public`.

---

### 2. The Legacy Code Challenge

-   **Definition**: "Legacy Code is code without tests." — Michael Feathers.
-   **The specific challenge**: Our Presentation components (`FeedPresenter`) had *no isolated unit tests*. They were only tested via the iOS UI Integration tests.
-   **Why Integration Tests aren't enough**:
    -   They are slow and imprecise. If a test fails, you don't know if it's the UI, the Presenter, or the Loader.
    -   They verify behavior *in the context of iOS*. We need to verify the Presenter works correctly *in isolation* for any platform.
-   **Refactoring Rule**: Before moving or refactoring code without tests, **you must write tests first**.

---

### 3. A "Test-After" TDD Workflow

We can still use TDD principles even when the code already exists.

1.  **Write a Test First**: Write a test for the existing behavior (e.g., `test_init_doesNotSendMessagesToView`).
2.  **Run and See it Pass**: This confirms the current behavior *works* as you expect. (If it fails, you found a bug or misunderstood the code!).
3.  **Break the Code (Mutation Testing)**: Intentionally break the production code (e.g., comment out a line).
4.  **Run and See it Fail**: This confirms your test is **actually testing the code**. If it still passes, your test is worthless.
5.  **Revert and Move On**: Revert the break and write the next test.

This cycle guarantees your new test suite is robust and trustworthy before you start the risky work of moving files between modules.

---

### 4. The Cost of "Test-After"

-   **Development Time**: It is significantly slower. You have to write code, manually test/debug it (slow), and then *later* come back and write tests (double work).
-   **Design Quality**: Code written without tests is often **hard to test**. You typically have to refactor the design just to make it testable, adding even more time.
-   **Confidence**: When writing tests last, developers often skip edge cases because "it works on my machine."
-   **Metric**: A "Test-After" curve in your codebase history shows production code (Blue line) rising while tests (Green line) stay flat, followed by a sudden spike in tests at the end. A healthy TDD curve shows them rising together.

-   <img width="1200" height="472" alt="image" src="https://github.com/user-attachments/assets/96774491-346e-4ecf-bcec-10d35fbfd45e" />

---

### 5. Benefits of the Refactor

By moving `FeedPresenter` and `FeedViewModel` to the cross-platform module:
1.  **Reusability**: We can now ship a macOS app using the exact same presentation logic.
2.  **Test Speed**: We now have fast, isolated unit tests for presentation logic, instead of relying on slow UI tests.
3.  **Architecture**: We have a cleaner separation. The UI module depends on Presentation, which depends on Domain.

---

# Code Snippets: Testing Legacy Code & Cross-Platform Extraction

These snippets illustrate the process of adding isolated tests to existing components.

### 1. The "Test-After" Workflow (Validation)

When writing tests for existing code, we must verify the test actually works.

```swift
func test_init_doesNotSendMessagesToView() {
    let (_, view) = makeSUT()
    
    XCTAssertTrue(view.messages.isEmpty, "Expected no view messages")
}

// VALIDATION STEP:
// Go to `FeedPresenter.init` and add a print statement or call a view method.
// Run the test.
// DOES IT FAIL?
// If YES: The test is valid. Revert the change.
// If NO: The test is broken or not connected. Fix it.
```

### 2. The Cross-Platform FeedPresenter

This presenter is now in the EssentialFeed module (or a dedicated Presentation module) and imports Foundation, not UIKit.

```Swift

import Foundation // No UIKit!

public final class FeedPresenter {
    private let feedView: FeedView
    private let loadingView: FeedLoadingView
    
    public init(feedView: FeedView, loadingView: FeedLoadingView) {
        self.feedView = feedView
        self.loadingView = loadingView
    }
    
    public static var title: String {
        return NSLocalizedString("FEED_VIEW_TITLE",
            tableName: "Feed",
            bundle: Bundle(for: FeedPresenter.self),
            comment: "Title for the feed view")
    }
    
    public func didStartLoadingFeed() {
        // ... logic ...
    }
}
```

### 3. The Isolated Unit Test

This test replaces the slow Integration Test. It mocks the Views to verify logic quickly.

```Swift
class FeedPresenterTests: XCTestCase {
    
    func test_title_isLocalized() {
        XCTAssertEqual(FeedPresenter.title, localized("FEED_VIEW_TITLE"))
    }

    func test_didStartLoadingFeed_displaysLoading() {
        let (sut, view) = makeSUT()
        
        sut.didStartLoadingFeed()
        
        XCTAssertEqual(view.messages, [.display(isLoading: true)])
    }
    
    // MARK: - Helpers
    
    private class ViewSpy: FeedLoadingView, FeedView {
        enum Message: Equatable {
            case display(isLoading: Bool)
            case display(feed: [FeedImage])
        }
        
        private(set) var messages = [Message]()
        
        func display(_ viewModel: FeedLoadingViewModel) {
            messages.append(.display(isLoading: viewModel.isLoading))
        }
        
        func display(_ viewModel: FeedViewModel) {
            messages.append(.display(feed: viewModel.feed))
        }
    }
}
```

---

### Senior & Architect Interview Perspective

**Q: "You have a large class with no tests, and you need to refactor it. How do you approach this?"**

> "I treat it as **Legacy Code**. I never refactor without a safety net.
>
> 1.  **Characterization Tests**: I write tests to 'pin down' the current behavior. Since I can't trust the code, I verify my tests by intentionally breaking the production code to ensure the tests fail (Mutation Testing).
> 2.  **Refactor**: Only once I have a solid suite of passing tests do I start refactoring.
> 3.  **Iterate**: I make small changes, running tests constantly to ensure I haven't broken existing functionality."

**Q: "Why move Presentation logic out of the UI module?"**

> "To enable **Cross-Platform Reuse**.
>
> Presentation logic (formatting dates, localized strings, state management) is usually platform-agnostic. By moving it into its own module (or the core module), I can share it between an iOS app (UIKit) and a macOS app (AppKit/SwiftUI).
>
> It also enforces better architecture. The UI becomes a 'dumb' consumer of the Presentation layer, which makes the UI easier to change and the Presentation logic easier to test in isolation."

**Q: "What are the downsides of writing tests after the code is done?"**

> "It's the most expensive way to develop.
> 1.  **Feedback Latency**: You lose the benefit of instant feedback during development, leading to more time spent in the debugger.
> 2.  **Testability**: Code written without tests in mind is often tightly coupled and hard to test, forcing you to refactor (re-do work) just to add tests.
> 3.  **Coverage Gaps**: When the feature is 'done,' there is psychological pressure to skip testing edge cases. TDD forces you to handle them."


---
