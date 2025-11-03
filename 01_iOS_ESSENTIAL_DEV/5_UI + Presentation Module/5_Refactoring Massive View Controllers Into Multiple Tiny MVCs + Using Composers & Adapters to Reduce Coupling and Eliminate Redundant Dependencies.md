# Learning Session: Reactoring Massive View Controllers into Multiple Tiny MVCs + Using Composers & Adapters to reduce Coupling and Eliminate Redundant Dependencies 

---

### **Metadata**
- **Date:** `01-Nov-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/10947420

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
# Notes: Refactoring Massive View Controllers (MVC) with Composers & Adapters

This lesson covers the TDD process for the `FeedViewController`'s responsibilities: rendering a list, handling cell visibility, and managing image loading, prefetching, and cancellation.

---

### 1. Identifying the "Massive View Controller" Anti-Pattern

-   **The Problem**: Our `FeedViewController` (MVC) was accumulating too many *responsibilities*.
-   **The Evidence**: It was responsible for:
    1.  Handling refresh logic (communicating with `FeedLoader`, managing `UIRefreshControl`).
    2.  Handling the main feed state (populating the `UITableView`).
    3.  Handling image loading for *each cell* (communicating with `FeedImageDataLoader`).
    4.  Managing the image loading state (loading, failure, retry) for *each cell*.
    5.  Managing `UITableViewDataSourcePrefetching`.
-   This is the classic "Massive View Controller" anti-pattern. The controller is bloated, hard to test, and hard to maintain.

-   <img width="714" height="591" alt="image" src="https://github.com/user-attachments/assets/c62ab2e1-d947-4066-a258-71120ad5d667" />
-   <img width="750" height="425" alt="image" src="https://github.com/user-attachments/assets/dc8e7391-6e27-45eb-bdf4-d6ee27795bf6" />


---

### 2. The Solution: Refactor into Multiple, Tiny MVCs

-   **The Strategy**: We refactor by **distributing responsibilities** into new, small, focused controllers. This is how MVC is *meant* to be used—as many small triads, not one giant one.
-   **New Components**:
    1.  **`FeedRefreshViewController`**: A new controller with *one job*: manage the `UIRefreshControl` and communicate with the `FeedLoader`.
    2.  **`FeedImageCellController`**: A new controller with *one job*: manage the state and logic for a *single* `FeedImageCell` (loading, retry, etc.).
-   **The Result**: The `FeedViewController` is now much simpler. It's just a coordinator that manages the list of `FeedImageCellController`s.

-   <img width="1250" height="533" alt="image" src="https://github.com/user-attachments/assets/b899f13e-05ca-419f-8f61-91c5edc439f2" />

---

### 3. The New Problem: Separating Creation vs. Usage

-   After refactoring, the `FeedViewController` is simpler, but now it *creates* its new child controllers.
-   **The Problem**: This means the `FeedViewController` *still* needs to know about all their dependencies (like the `FeedLoader` and `FeedImageDataLoader`), even if it doesn't use them directly. It's become a "Factory," which still creates unwanted coupling.

---

### 4. The Solution: The Composer Pattern

-   **The Goal**: We must separate **creation** of objects from their **usage**.
-   **The Pattern**: We create a **Composer** (e.g., `FeedUIComposer`).
-   **Its Job**: The Composer is the *only* component that knows how to build and "compose" the final `FeedViewController` with all its dependencies.
-   **The "Composition Root"**: The Composer is only ever used in the app's "Composition Root" (like the `SceneDelegate`). Components like a `LoginViewController` should *never* call the composer directly; they should get the `FeedViewController` via Dependency Injection.

-   <img width="1000" height="572" alt="image" src="https://github.com/user-attachments/assets/971328a1-743d-4099-a2d9-35bfe37b0f7a" />

---

### 5. The Adapter Pattern

-   **The Problem**: We have an **interface mismatch**.
    1.  The `FeedRefreshViewController` finishes loading and produces an `[FeedImage]` (our domain model).
    2.  The `FeedViewController`'s `tableModel` expects an `[FeedImageCellController]` (our new cell controller).
-   **The Solution**: We use the **Adapter Pattern**. We create a simple adapter function that *translates* the output of one component into the input for the next.
-   **The Implementation**: Our `FeedUIComposer` uses a private `adaptFeedToCellControllers` function to "wire" the two components together. This function takes the `[FeedImage]` and maps it into the `[FeedImageCellController]` array that the `FeedViewController` needs.

-   <img width="750" height="82" alt="image" src="https://github.com/user-attachments/assets/a95b8979-62ad-41ac-9795-be65b774b93f" />

---

### Senior & Architect Interview Perspective

**Q: "You've identified a 'Massive View Controller.' What's the core problem, and how do you fix it?"**

> "The core problem is that the View Controller has **too many responsibilities**. It's acting as a data source, a layout coordinator, a state manager, and a factory.
>
> The fix is to **refactor by delegating**. I'd extract responsibilities into new, focused components. For example, I'd create a `FeedRefreshViewController` to handle only the refresh logic and a `FeedImageCellController` to manage the logic for a single cell. This leaves the `FeedViewController` as a simple coordinator."

**Q: "After refactoring, your `FeedViewController` still creates these new controllers. What's the problem with that?"**

> "That's a 'Factory' anti-pattern. It couples the `FeedViewController` to the *dependencies* of its children (like the `FeedLoader`), even if it doesn't use them.
>
> The solution is to use a **Composer** (`FeedUIComposer`). This object exists in the **Composition Root** (like the `SceneDelegate`) and is the *only* thing that knows how to build the `FeedViewController` and its child controllers. This separates 'creation' from 'usage' and keeps my View Controller clean."

**Q: "What is the Adapter pattern, and when have you used it?"**

> "The Adapter pattern is a structural pattern that **connects two incompatible interfaces**. It acts as a translator.
>
> A classic example is in our UI. Our `FeedRefreshViewController` reloads and produces an array of `[FeedImage]` (domain models). But our `FeedViewController`'s `UITableView` needs an array of `[FeedImageCellController]` (UI components).
>
> In our **Composer**, we create a simple `adapt` function. This function is 'wired' to the refresh controller's completion. It receives the `[FeedImage]` and maps it into the `[FeedImageCellController]` array the `FeedViewController` expects, seamlessly connecting the two components."

---

## Code Snippets

### 1. The New, Focused `FeedRefreshViewController`
This controller has one responsibility: manage the `UIRefreshControl` and load the feed.

```Swift
public final class FeedRefreshViewController: NSObject {
    public lazy var view: UIRefreshControl = {
        let view = UIRefreshControl()
        view.addTarget(self, action: #selector(refresh), for: .valueChanged)
        return view
    }()
    
    private let feedLoader: FeedLoader
    
    init(feedLoader: FeedLoader) {
        self.feedLoader = feedLoader
    }
    
    var onRefresh: (([FeedImage]) -> Void)?
    
    @objc func refresh() {
        view.beginRefreshing()
        feedLoader.load { [weak self] result in
            if let feed = try? result.get() {
                self?.onRefresh?(feed)
            }
            self?.view.endRefreshing()
        }
    }
}
```

### 2. The New `FeedImageCellController`
This controller has one responsibility: manage the state and logic for a *single* `FeedImageCell`.

```Swift
final class FeedImageCellController {
    private var task: FeedImageDataLoaderTask?
    private let model: FeedImage
    private let imageLoader: FeedImageDataLoader
    
    init(model: FeedImage, imageLoader: FeedImageDataLoader) {
        self.model = model
        self.imageLoader = imageLoader
    }
    
    func view() -> FeedImageCell {
        let cell = FeedImageCell()
        // ... configure cell from model ...
        
        let loadImage = { [weak self, weak cell] in
            // ... load image logic ...
        }
        
        cell.onRetry = loadImage
        loadImage()
        
        return cell
    }
    
    func preload() {
        task = imageLoader.loadImageData(from: model.url) { _ in }
    }
    
    func cancelLoad() {
        task?.cancel()
    }
}
```

### 3. The Refactored `FeedViewController` (Now a "Coordinator")
This controller is now much lighter. It just manages a list of `FeedImageCellController`s.

```Swift
public final class FeedViewController: UITableViewController, UITableViewDataSourcePrefetching {
    public var refreshController: FeedRefreshViewController?
    var tableModel = [FeedImageCellController]() {
        didSet { tableView.reloadData() }
    }
    
    convenience init(refreshController: FeedRefreshViewController) {
        self.init()
        self.refreshController = refreshController
    }
    
    public override func viewDidLoad() {
        super.viewDidLoad()
        
        tableView.prefetchDataSource = self
        refreshControl = refreshController?.view
        refreshController?.refresh()
    }

    public override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return tableModel.count
    }
    
    public override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        return cellController(for: indexPath).view()
    }
    
    public override func tableView(_ tableView: UITableView, didEndDisplaying cell: UITableViewCell, forRowAt indexPath: IndexPath) {
        cancelCellControllerLoad(forRowAt: indexPath)
    }
    
    public func tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath]) {
        indexPaths.forEach { indexPath in
            cellController(for: indexPath).preload()
        }
    }
    
    public func tableView(_ tableView: UITableView, cancelPrefetchingForRowsAt indexPaths: [IndexPath]) {
        indexPaths.forEach(cancelCellControllerLoad)
    }
    
    private func cellController(for indexPath: IndexPath) -> FeedImageCellController {
        return tableModel[indexPath.row]
    }

    private func cancelCellControllerLoad(forRowAt indexPath: IndexPath) {
        cellController(for: indexPath).cancelLoad()
    }
}
```

### 4. The `FeedUIComposer` (The "Factory")
This is the **Composition Root**. It knows how to *create* and *wire up* all the components.

```Swift
public final class FeedUIComposer {
    private init() {}
    
    public static func feedComposedWith(feedLoader: FeedLoader, imageLoader: FeedImageDataLoader) -> FeedViewController {
        let refreshController = FeedRefreshViewController(feedLoader: feedLoader)
        let feedController = FeedViewController(refreshController: refreshController)
        
        // Wiring components together using the Adapter
        refreshController.onRefresh = adaptFeedToCellControllers(forwardingTo: feedController, loader: imageLoader)
        
        return feedController
    }
    
    // ... adapter function is next ...
}
```

### 5. The Adapter Function
This private function *adapts* the `[FeedImage]` output to the `[FeedImageCellController]` input.

```Swift
private extension FeedUIComposer {
    private static func adaptFeedToCellControllers(forwardingTo controller: FeedViewController, loader: FeedImageDataLoader) -> ([FeedImage]) -> Void {
        return { [weak controller] feed in
            controller?.tableModel = feed.map { model in
                FeedImageCellController(model: model, imageLoader: loader)
            }
        }
    }
}
```
