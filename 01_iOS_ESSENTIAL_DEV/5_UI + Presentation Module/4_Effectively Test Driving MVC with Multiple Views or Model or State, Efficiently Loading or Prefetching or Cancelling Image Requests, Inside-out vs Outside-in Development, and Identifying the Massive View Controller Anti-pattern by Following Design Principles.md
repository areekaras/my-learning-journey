# Learning Session: Efficiently Test driving MVC with Mulitple Views/Models/States, Effectively Loading/Prefetching/Cancelling Image Requests, Inside-out vs. Outside-in Development, and Identifying the Massive View Controller Anti-pattern by Following Design Principles

---

### **Metadata**
- **Date:** `01-Nov-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/10883866

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
# Notes: TDD for UI - Image Loading, Prefetching & The Massive View Controller

This lesson covers the TDD process for the `FeedViewController`'s responsibilities: rendering a list, handling cell visibility, and managing image loading, prefetching, and cancellation.

---

### 1. Development Approach: Inside-Out vs. Outside-In

This module marked a switch in our development approach.

-   **Inside-Out (What we did first)**:
    -   We started with the "Core Domain" (`FeedImage`, `<FeedLoader>`) and moved "out" to "Application Logic" (`LocalFeedLoader`) and "Infrastructure" (`CoreDataFeedStore`).
    -   This is a great approach when you *don't* have a concrete UI design and need to build the core logic.
-   **Outside-In (What we're doing now)**:
    -   We're starting with the "Outside" layer (the `FeedViewController`) and test-driving its needs.
    -   This led us to *discover* the need for a new abstraction: the `<FeedImageDataLoader>`. This protocol is defined by the UI's needs but will be implemented by the "Inner" layers.
    -   This is a valid approach when you have a clear UI design to build against.

-   <img width="1000" height="986" alt="image" src="https://github.com/user-attachments/assets/8a30fa03-1120-4fcc-b398-1b2eef7af7e5" />
-   <img width="750" height="694" alt="image" src="https://github.com/user-attachments/assets/f8e5076a-df16-4be6-9e2c-b7e0c262c7f0" />

---

### 2. The Open-Closed Principle (OCP) in Action

-   **The Problem**: The `FeedViewController` needs to load images. A "naive" approach would be to call `URLSession.shared.dataTask(...)` directly inside `cellForRowAt`.
-   **Why this is bad**: This **tightly couples** the `FeedViewController` to `URLSession`. What if we want to add caching? Logging? Retry logic? We'd have to keep *modifying* the `FeedViewController`, violating the "Closed for Modification" rule.
-   **The Solution**: We apply **Dependency Inversion** by creating a new abstraction, the `<FeedImageDataLoader>` protocol.
    -   The `FeedViewController` *depends on this protocol*.
    -   We are now "Open for Extension" because we can plug in *any* implementation (a `RemoteFeedImageDataLoader`, a `CachedFeedImageDataLoader`, etc.) without *ever* modifying the `FeedViewController`'s code.

-   <img width="1000" height="448" alt="image" src="https://github.com/user-attachments/assets/7b2801a5-5be0-47c5-8e97-9ef705eb6279" />

---

### 3. The Interface Segregation Principle (ISP) in Action

-   **The Problem**: Why create a *new* `<FeedImageDataLoader>` protocol? Why not just add the `loadImageData(from:)` method to the existing `<FeedLoader>` protocol?
-   **Why this is bad**:
    -   It violates **ISP**: "No client should be forced to depend on methods it does not use." Our `RemoteFeedLoader` *has no reason* to know how to load a single image.
    -   It violates **SRP**: The `<FeedLoader>`'s single responsibility is to load the *feed*, not individual images.
    -   It creates a "fat" protocol, which is a common anti-pattern.
-   **The Solution**: We create a new, lean, and specific protocol for this one, distinct responsibility.

-   <img width="1000" height="448" alt="image" src="https://github.com/user-attachments/assets/66e72761-d241-48c9-8afe-13698ad93449" />
-   <img width="1000" height="449" alt="image" src="https://github.com/user-attachments/assets/712eb3d8-b478-4343-9083-efa5798039df" />


---

### 4. Efficient Image Loading in `UITableView`

We TDD'd a robust set of behaviors for handling images in a list:

1.  **Load on Visibility**: The `imageLoader` is only called when `cellForRowAt` is triggered (i.e., the cell becomes visible).
2.  **Cancel on Invisibility**: When a cell scrolls off-screen (`didEndDisplaying`), we **cancel** its `FeedImageDataLoaderTask`. This is a critical optimization to save network and battery.
3.  **Prefetch on Approach**: We use the `UITableViewDataSourcePrefetching` delegate to start loading images *before* they scroll onto the screen, making the UI feel faster.
4.  **Cancel Prefetch**: If the user scrolls back, we also cancel the prefetch requests.

---

### 5. Identifying the "Massive View Controller" (MVC) Anti-Pattern

-   Even with simple, clean code, our `FeedViewController` is already showing signs of the **Massive View Controller** anti-pattern.
-   **The Evidence**: It has too many *responsibilities*:
    -   Managing the `UIRefreshControl` state.
    -   Acting as the `UITableView` data source and delegate.
    -   Managing the "feed is loading" state.
    -   Managing the "image is loading" state *for each cell*.
    -   Handling image prefetching and cancellation.
    -   Keeping a dictionary of `[IndexPath: Task]` to manage cancellations.
-   **The Solution**: This complexity is a "code smell." The solution, which will come later, is to **refactor by moving these responsibilities** into new, focused components, leaving the `FeedViewController` as a simple mediator.

---

### 6. Testing UIKit Without Mocking

-   **The Goal**: We test the *real* `FeedViewController` and assert the state of *real* `FeedImageCell` instances. We **do not mock UIKit components**.
-   **In-Memory Image Representations**:
    -   **The Problem**: Loading test images from disk (`.png` files) makes tests slow and fragile (e.g., if a file is renamed or deleted).
    -   **The Solution**: We create in-memory `UIImage` objects *programmatically* using a simple helper. This is fast, reliable, and keeps the test self-contained.

---

### Senior & Architect Interview Perspective

**Q: "How do you handle loading images in a `UITableView`?"**

> "I'd implement a few key behaviors.
> 1.  I'd have an `ImageLoader` protocol (DIP) to decouple the view controller from the concrete implementation (e.g., `URLSession`).
> 2.  In `cellForRowAt`, I'd trigger the async image load.
> 3.  Crucially, in `didEndDisplaying`, I'd **cancel** the image request for that cell to prevent wasted resources.
> 4.  For a smoother experience, I'd also implement `UITableViewDataSourcePrefetching` to start loading images for cells *before* they become visible, and cancel those prefetch requests in `cancelPrefetchingForRowsAt`."

**Q: "You've identified a 'Massive View Controller.' What's the core problem, and how do you fix it?"**

> "The core problem is that the View Controller has **too many responsibilities**. It's acting as a data source, a layout coordinator, a state manager, and a factory for other components.
>
> The fix is to **refactor and delegate**.
> 1.  **Data Source**: I'd move the `UITableView` data source logic into a separate `FeedDataSource` object.
> 2.  **State Management**: I'd move the loading state logic into a `FeedViewModel` or `Presenter`.
> 3.  **Image Loading**: The complex logic for managing cell image loads, prefetching, and cancellation should be extracted into its *own* component, a `CellController` or `ImageLoaderService`.
>
> The `UIViewController`'s *only* job should be to mediate between these other components and the View."

**Q: "Your `FeedViewController` needs to load the main feed and also load images for each cell. Should you just add the `loadImage` method to your `FeedLoader` protocol?"**

> "No, that would be a **violation of the Interface Segregation Principle (ISP)**.
>
> My `FeedLoader`'s responsibility is *only* to load the array of feed items. The `RemoteFeedLoader` implementation, for example, has no business knowing how to load a single image from a URL.
>
> Forcing it to implement `loadImage` would create a "fat protocol" and violate the Single Responsibility Principle. The correct solution is to create a *new, separate, and focused* protocol, `<FeedImageDataLoader>`, for this distinct responsibility."

**Q: "How do you test that a cell's image, which is loaded from an `in-memory` data blob, is correct?"**

> "We don't need to load PNGs from disk, which is slow and brittle. In my tests, I create a **programmatic `UIImage`** helper (e.g., `UIImage.make(withColor: .red)`), convert that to `Data`, and pass that `Data` to my mock image loader.
>
> Then, in the assertion phase, I just check that the `cell.feedImageView.image?.pngData()` is equal to my original red image `Data`. This is fast, 100% reliable, and keeps the test self-contained."

---

# Code Snippets: UI Testing DSLs and Techniques

These helpers show how to TDD the `FeedViewController` and its cells.

---

### 1. Test-Specific `LoaderSpy`

This spy implements *both* protocols, allowing it to act as the single mock dependency for the `FeedViewController`. This is a powerful pattern for TDD'ing controllers.

```swift
private class LoaderSpy: FeedLoader, FeedImageDataLoader {
    
    // MARK: - FeedLoader
    
    private var feedRequests = [(FeedLoader.Result) -> Void]()
    
    var loadFeedCallCount: Int {
        return feedRequests.count
    }
    
    func load(completion: @escaping (FeedLoader.Result) -> Void) {
        feedRequests.append(completion)
    }
    
    func completeFeedLoading(with feed: [FeedImage] = [], at index: Int = 0) {
        feedRequests[index](.success(feed))
    }

    // MARK: - FeedImageDataLoader
    
    private struct TaskSpy: FeedImageDataLoaderTask {
        let cancelCallBack: () -> Void
        func cancel() {
            cancelCallBack()
        }
    }
    
    private var imageRequests = [(url: URL, completion: (FeedImageDataLoader.Result) -> Void)]()
    
    var loadedImageURLs: [URL] {
        return imageRequests.map { $0.url }
    }
    
    private(set) var cancelledImageURLs = [URL]()
    
    func loadImageData(from url: URL, completion: @escaping (FeedImageDataLoader.Result) -> Void) -> FeedImageDataLoaderTask {
        imageRequests.append((url, completion))
        return TaskSpy { [weak self] in self?.cancelledImageURLs.append(url) }
    }
    
    func completeImageLoading(with imageData: Data = Data(), at index: Int = 0) {
        imageRequests[index].completion(.success(imageData))
    }
    
    func completeImageLoadingWithError(at index: Int = 0) {
        let error = NSError(domain: "an error", code: 0)
        imageRequests[index].completion(.failure(error))
    }
}
```

### 2. Simulating Cell Lifecycle Events (Tiny DSL)

These helpers on the `FeedViewController` test extension allow us to simulate table view behavior in a readable, decoupled way.
```Swift
private extension FeedViewController {
    // Simulates a cell becoming visible
    @discardableResult
    func simulateFeedImageViewVisible(at index: Int) -> FeedImageCell? {
        return feedImageView(at: index) as? FeedImageCell
    }
    
    // Simulates a cell scrolling off-screen
    func simulateFeedImageViewNotVisible(at index: Int) {
        let view = simulateFeedImageViewVisible(at: index)
        
        let delegate = tableView.delegate
        let index = IndexPath(row: index, section: feedImagesSection)
        // Manually trigger the delegate method
        delegate?.tableView?(tableView, didEndDisplaying: view!, forRowAt: index)
    }
    
    // Simulates a cell starting to prefetch
    func simulateFeedImageViewNearVisible(at row: Int) {
        let ds = tableView.prefetchDataSource
        let index = IndexPath(row: row, section: feedImagesSection)
        ds?.tableView(tableView, prefetchRowsAt: [index])
    }
    
    // Simulates a cell prefetch being cancelled
    func simulateFeedImageViewNotNearVisible(at row: Int) {
        simulateFeedImageViewNearVisible(at: row)
        
        let ds = tableView.prefetchDataSource
        let index = IndexPath(row: row, section: feedImagesSection)
        ds?.tableView?(tableView, cancelPrefetchingForRowsAt: [index])
    }
}
```

### 3. Simulating Button Taps (Tiny DSL)

This helper hides the `UIControl` target-action logic.

```Swift
private extension UIButton {
    func simulateTap() {
        allTargets.forEach { target in
            actions(forTarget: target, forControlEvent: .touchUpInside)?.forEach {
                (target as NSObject).perform(Selector($0))
            }
        }
    }
}

// And the cell-specific helper:
private extension FeedImageCell {
    func simulateRetryAction() {
        feedImageRetryButton.simulateTap()
    }
}
```

### 4. In-Memory Image Creation (Test Helper)

This helper creates a test `UIImage` programmatically, avoiding the need to bundle image assets with the tests.

```Swift
private extension UIImage {
    static func make(withColor color: UIColor) -> UIImage {
        let rect = CGRect(x: 0, y: 0, width: 1, height: 1)
        let format = UIGraphicsImageRendererFormat()
        format.scale = 1 // Fix for iOS 16.1+ comparison issues
        
        return UIGraphicsImageRenderer(size: rect.size, format: format).image { rendererContext in
            color.setFill()
            rendererContext.fill(rect)
        }
    }
}
```
