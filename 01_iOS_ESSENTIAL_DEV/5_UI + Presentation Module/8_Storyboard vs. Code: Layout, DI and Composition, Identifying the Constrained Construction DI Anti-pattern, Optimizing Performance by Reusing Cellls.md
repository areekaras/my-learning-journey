# Learning Session: Storyboard vs. Code: Layout, DI and Composition, Identifying the constrained Construction DI Anti-pattern, Optimizing Performance by Reusing Cells

---

### **Metadata**
- **Date:** `08-11-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/11390253

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is clear.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)


#### Notes

PR: https://github.com/areekaras/EssentialFeed/pull/17

---
# Notes: Storyboards vs. Code, Cell Reuse & Optimization

This lecture explores the trade-offs of UI implementation strategies and covers important optimizations and bug fixes related to `UITableView` cell reuse.

-   <img width="1000" height="729" alt="image" src="https://github.com/user-attachments/assets/7972533d-e32c-4bf8-858e-aaf6654f6d69" />
-   <img width="1000" height="459" alt="image" src="https://github.com/user-attachments/assets/8d959e4e-8cb6-41b9-9dca-203ab68f75de" />

### 1. Storyboard vs. Code: The Trade-offs

-   **Code (Preferred)**:
    -   **Pros**: Flexible, reusable, efficient, better for version control (no XML merge conflicts), and allows **Constructor Injection** (safe dependencies).
    -   **Cons**: Slower to visualize layout, more verbose for simple screens.
-   **Storyboards**:
    -   **Pros**: Rapid visual feedback, fast for simple layouts, easy to handle multiple screen sizes. Great for prototyping.
    -   **Cons**:
        -   Mixes too many concerns (Layout, Navigation, Composition).
        -   Hard to reuse views.
        -   **Constrained Construction**: Forces you to use `init?(coder:)`, making **Constructor Injection difficult** (you often fall back to Property Injection, which is less safe).
-   **Recommendation**: Use Storyboards *only* for Layout/View Configuration if it helps speed. Avoid using them for navigation or logic. Ideally, keep tests agnostic so you can switch implementation later.

---

### 2. Dependency Injection with Storyboards

-   **The Problem**: Storyboards instantiate ViewControllers for you via `init(coder:)`, preventing you from injecting dependencies in a custom `init`. This forces **Property Injection** (using `var`s), which leads to temporal coupling (dependencies might not be set when the view loads).
-   **The Solution (iOS 13+)**: Use `instantiateInitialViewController(creator:)`. This API lets you intercept creation and call a custom `init` with dependencies, enabling proper **Constructor Injection** even with Storyboards.
    -  <img width="757" height="360" alt="image" src="https://github.com/user-attachments/assets/d8301b3e-b2bb-4088-a149-1ca002b2913c" />
-   **The Strategy**: Move all instantiation logic to the **Composition Root**. This centralizes the "ugly" Storyboard creation code and keeps the rest of the app clean.

---

### 3. Optimizing `UITableView` Performance

-   **Cell Reuse**: Reusing cells (`dequeueReusableCell`) is critical for performance. However, it introduces complexity because a single cell instance is shared by multiple data models over time.
-   **The Bug**: If a `FeedImageCellController` holds a strong reference to a cell after it goes off-screen, and that cell is reused by *another* controller, the old controller might update the cell with old data (e.g., an image download finishes late). This causes the UI to show the wrong image or "flicker."
-   **The Fix**:
    1.  **Release the Cell**: The controller *must* release its reference to the cell when `didEndDisplaying` is called.
    2.  **iOS 15+ Lifecycle Changes**: On iOS 15, `cellForRow` might not be called for cached cells; only `willDisplayCell` is called.
        -   **Fix**: We must re-capture the cell reference and restart requests in `willDisplayCell`.
    3.  **Prefetching Edge Case**: `cellForRow` might be called for prefetching, but the cell might *never* be displayed.
        -   **Fix**: Release the cell reference in `prepareForReuse` to prevent race conditions where a request finishes for a cell that was never shown but is now being reused.

-   **Testing Optimizations**: Before fixing the bug, write a **failing test** (`test_feedImageView_doesNotRenderLoadedImageWhenNotVisibleAnymore`). This proves the bug exists and protects against regressions.

```Swift
final class FeedViewControllerTests: XCTestCase {
...
  func test_feedImageView_doesNotRenderLoadedImageWhenNotVisibleAnymore() {
      let (sut, loader) = makeSUT()
      sut.simulateAppearence()
      loader.completeFeedLoading(with: [makeImage()])
      
      let view = sut.simulateFeedImageViewNotVisible(at: 0)
      loader.completeImageLoading(with: anyImageData())
      
      XCTAssertNil(view?.renderedImage, "Expected no image rendered when an image load finishes after the view is not visible anymore")
  }
}

```
---

### Senior & Architect Interview Perspective

**Q: "Do you prefer Storyboards or programmatic UI, and why?"**

> "I generally prefer **programmatic UI** for large teams. It avoids complex XML merge conflicts and allows for **Constructor Injection**, which guarantees my ViewControllers have all their dependencies at creation time.
>
> Storyboards force **Property Injection** (unless using the newer iOS 13 API), which creates temporal coupling—my controller exists, but its dependencies might be nil. However, Storyboards are excellent for rapid prototyping or small, static screens. I choose the tool based on the project's scale and team size."

**Q: "What are the risks of using `UITableView` cell reuse with asynchronous image loading?"**

> "The main risk is **race conditions** updating the wrong cell.
>
> If a cell is reused for Row B while an image load for Row A is still pending, the completion handler for Row A might fire and update the cell (which now displays Row B). This causes the wrong image to appear.
>
> To fix this, we must:
> 1.  **Cancel** the image load when the cell goes off-screen (`didEndDisplaying`).
> 2.  **Check for identity** before updating the UI (e.g., ensure the cell instance hasn't changed or been reused)."

**Q: "You're optimizing a list performance. What's your process?"**

> "I follow the mantra: **'Make it work. Make it right. Make it fast.'**
>
> 1.  First, ensure the feature works correctly with tests.
> 2.  Then, clean up the design (Make it right).
> 3.  Finally, optimize (Make it fast).
>
> When optimizing (e.g., cell reuse), I treat it as a refactoring. I write a test case to prove any bug I suspect (like a reuse bug), then implement the fix. This ensures my optimization doesn't break existing behavior."

---
