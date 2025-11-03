<img width="1000" height="435" alt="image" src="https://github.com/user-attachments/assets/52bbace8-543d-49e3-9b26-45eb5965e765" /># Learning Session: MVVM: Reducing Boilerplate, Shifting Reusable Presentation Logic from controllers into Cross-platform (Stateful & Stateless) ViewModels, and Decoupling Presentation from UI Frameworks with Swift Generics

---

### **Metadata**
- **Date:** `02-Nov-2025`, `03-Nov-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/11188204

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
# Notes: Refactoring to MVVM & The Presentation Layer

This lecture covers refactoring our MVC components into an MVVM (Model-View-ViewModel) pattern. The primary goal is *not* just to use a new pattern, but to **create a platform-agnostic Presentation layer**, moving logic out of our "Massive View Controllers."

---

### 1. What is MVVM (Model-View-ViewModel)?

-   **MVVM** is a UI architectural pattern with three components:
    -   **Model**: The data and business logic (e.g., `FeedLoader`).
    -   **View**: The UI (`UIRefreshControl`, `UITableViewCell`).
    -   **ViewModel**: A new component that holds the **presentation logic** and **state**. It "prepares" data from the Model so it's ready for the View to display.

-   **MVC vs. MVVM**:
    -   In our **MVC** setup, the `FeedRefreshViewController` (Controller) talked *directly* to the `FeedLoader` (Model) to manage state.
    -   In our new **MVVM** setup, the `FeedViewModel` (ViewModel) talks to the `FeedLoader` (Model). The `FeedRefreshViewController` (now a Binder) *only* talks to the `FeedViewModel`.

-   <img width="1000" height="490" alt="image" src="https://github.com/user-attachments/assets/5b2f35be-3a07-4603-bf84-bb00e6a75e68" />

---

### 2. MVVM in UIKit: The "Model-View-Binder" Pattern

-   Unlike some frameworks, UIKit **does not** have automatic, built-in data binding.
-   This means **we must write the "binding" code ourselves**.
-   In our architecture, the `UIViewController`s (`FeedRefreshViewController`, `FeedImageCellController`) take on the new, simpler role of being a **Binder**.
-   Their *only* job is to:
    1.  Pass user events (like a button tap) *to* the ViewModel.
    2.  Observe the ViewModel's state (using closures like `onLoadingStateChange`) and update the View.

-   <img width="1000" height="236" alt="image" src="https://github.com/user-attachments/assets/7472b959-d224-4371-bc96-62531a9926ed" />

---

### 3. The Refactoring: Shifting Responsibilities

We refactored our "Massive View Controller" components by moving their logic into ViewModels.

-   **BEFORE (MVC)**:
    -   `FeedRefreshViewController` (Controller)
        -   Knew about `FeedLoader`.
        -   Managed `isLoading` state.
        -   Called `beginRefreshing()` on `UIRefreshControl`.
-   **AFTER (MVVM)**:
    -   `FeedViewModel` (Presentation)
        -   Knows about `FeedLoader`.
        -   Manages `isLoading` state.
        -   Exposes an `onLoadingStateChange` closure.
    -   `FeedRefreshViewController` (Binder)
        -   Knows *only* about `FeedViewModel`.
        * Binds `onLoadingStateChange` to `UIRefreshControl.begin/endRefreshing()`.

-   <img width="1000" height="259" alt="image" src="https://github.com/user-attachments/assets/b1157079-34c7-4ff5-8b9e-8706a28318ad" />
-   <img width="1000" height="435" alt="image" src="https://github.com/user-attachments/assets/08de6b40-f150-442a-8803-60492a8f8c75" />
-   This refactoring makes our `FeedRefreshViewController` a simple, reusable, "dumb" component. The *logic* now lives in the **platform-agnostic** `FeedViewModel`.

---

### 4. Creating a Platform-Agnostic ViewModel (with Generics)

This is the most powerful concept from the lecture.

-   **The Problem**: The `FeedImageViewModel` needs to provide a displayable image. If it directly referenced `UIImage`, it would be **coupled to UIKit**, making it unusable on macOS (which uses `NSImage`).
-   **The Solution**: Use **Generics** and **Dependency Injection**.
    1.  We define our ViewModel as `FeedImageViewModel<Image>`. The `<Image>` is a generic placeholder.
    2.  We inject a *transformer closure* in the `init`: `imageTransformer: @escaping (Data) -> Image?`.
-   **The Benefit**: The `FeedImageViewModel` is now **completely platform-agnostic**.
    -   In our **iOS** `FeedUIComposer`, we inject the `UIImage.init` transformer.
    -   If we built a **macOS** app, we would inject `NSImage.init`.
-   This design separates our Presentation logic from any specific UI framework.

-   <img width="1000" height="533" alt="image" src="https://github.com/user-attachments/assets/a8e70697-99e9-477d-afe4-6c8d1f394e34" />
-   <img width="1000" height="498" alt="image" src="https://github.com/user-attachments/assets/6a8aa3f1-4424-4ba9-bae2-8ec34145e236" />

---

### 5. Mixing Patterns: MVC + MVVM

-   You don't have to choose 100% MVC or 100% MVVM.
-   We *didn't* create a ViewModel for the main `FeedViewController`. Why? Because after our refactoring, it was *already* simple. Its only job was to manage the list of `FeedImageCellController`s.
-   **Use patterns to solve problems.** We used MVVM to solve the specific problem of stateful presentation logic (`FeedRefreshViewController`) and platform coupling (`FeedImageViewModel`).

---

### Senior & Architect Interview Perspective

**Q: "What's the *real* benefit of MVVM in UIKit, given that it doesn't have automatic data-binding?"**

> "The main benefit isn't *less* code—we still have to write the binding logic, often with closures. The *real* benefit is **Separation of Concerns**.
>
> MVVM allows us to extract all presentation logic—like data formatting, state management (`isLoading`, `error`), and business logic interaction—out of the `UIViewController` and into a **platform-agnostic `ViewModel`**.
>
> This makes our `UIViewController` a simple, "dumb" **Binder**. The `ViewModel` is now a pure Swift object that is completely testable, reusable across platforms, and doesn't import UIKit."

**Q: "How do you make a `ViewModel` that needs to display an image (like a `UIImage`) reusable on macOS (which uses `NSImage`)?"**

> "I'd use **Generics** and **Dependency Injection**.
>
> 1.  I'd define the ViewModel as `FeedImageViewModel<Image>`.
> 2.  The `init` would take a transformer closure: `imageTransformer: @escaping (Data) -> Image?`.
> 3.  The ViewModel's logic would load `Data`, pass it to the `imageTransformer`, and output the generic `Image` type.
>
> This makes the ViewModel platform-agnostic. In my iOS **Composer**, I'd inject `UIImage.init` as the transformer. In my macOS Composer, I'd inject `NSImage.init`."

**Q: "Do you need RxSwift or Combine to implement MVVM?"**

> "No, not at all. While those frameworks are powerful for complex reactive streams, a simple and effective binding mechanism can be built with **standard Swift closures**.
>
> For example, the `ViewModel` can expose properties like `var onLoadingStateChange: ((Bool) -> Void)?`. The `UIViewController` (acting as the Binder) just assigns a closure to this property to update the UI. This is lightweight, easy to debug, and has no external dependencies."

---

## Code Snippets

### 1. The `FeedViewModel` (Presentation Logic)
This class new class holds the presentation logic, talks to the `FeedLoader`, and manages the loading state.

```Swift
final class FeedViewModel {
    typealias Observer<T> = (T) -> Void
    private let feedLoader: FeedLoader
    
    init(feedLoader: FeedLoader) {
        self.feedLoader = feedLoader
    }
    
    var onLoadingStateChange: Observer<Bool>?
    var onFeedLoad: Observer<[FeedImage]>?
    
    func loadFeed() {
        onLoadingStateChange?(true)
        feedLoader.load { [weak self] result in
            if let feed = try? result.get() {
                self?.onFeedLoad?(feed)
            }
            self?.onLoadingStateChange?(false)
        }
    }
}
```

### 2. The Refactored `FeedRefreshViewController` (The "Binder")
This controller is now a "dumb" binder. It just connects the `ViewModel`'s state to the `UIRefreshControl`.

```Swift
public final class FeedRefreshViewController: NSObject {
    public lazy var view = binded(UIRefreshControl())
    
    private let viewModel: FeedViewModel
    
    init(viewModel: FeedViewModel) {
        self.viewModel = viewModel
    }
        
    @objc func refresh() {
        viewModel.loadFeed()
    }
    
    private func binded(_ view: UIRefreshControl) -> UIRefreshControl {
        viewModel.onLoadingStateChange = { [weak self] isLoading in
            if isLoading {
                self?.view.beginRefreshing()
            } else {
                self?.view.endRefreshing()
            }
        }
        view.addTarget(self, action: #selector(refresh), for: .valueChanged)
        return view
    }
}
```

### 3. The Generic `FeedImageViewModel` (Platform-Agnostic)
This `ViewModel` is decoupled from `UIKit` using the generic `<Image>` type and the `imageTransformer` closure.

```Swift
final class FeedImageViewModel<Image> {
    typealias Observer<T> = (T) -> Void
    
    private var task: FeedImageDataLoaderTask?
    private let model: FeedImage
    private let imageLoader: FeedImageDataLoader
    private let imageTransformer: (Data) -> Image?
    
    init(model: FeedImage, imageLoader: FeedImageDataLoader, imageTransformer: @escaping (Data) -> Image?) {
        self.model = model
        self.imageLoader = imageLoader
        self.imageTransformer = imageTransformer
    }
    
    var description: String? { /* ... */ }
    var location: String? { /* ... */ }
    var hasLocation: Bool { /* ... */ }
        
    var onImageLoad: Observer<Image>?
    var onImageLoadingStateChange: Observer<Bool>?
    var OnShouldRetryImageLoadStateChange: Observer<Bool>?
    
    func loadImageData() {
        onImageLoadingStateChange?(true)
        OnShouldRetryImageLoadStateChange?(false)
        task = imageLoader.loadImageData(from: model.url) { [weak self] result in
            self?.handle(result)
        }
    }
    
    private func handle(_ result: FeedImageDataLoader.Result) {
        if let image = (try? result.get()).flatMap(imageTransformer) {
            onImageLoad?(image)
        } else {
            OnShouldRetryImageLoadStateChange?(true)
        }
        onImageLoadingStateChange?(false)
    }
        
    func cancelImageDataLoad() {
        task?.cancel()
        task = nil
    }
}
```

### 4. The `FeedImageCellController` (The "Binder")
This controller is also now a "binder," connecting the `FeedImageViewModel` to the `FeedImageCell`.

```Swift
final class FeedImageCellController {
    private let viewModel: FeedImageViewModel<UIImage>
    
    init(viewModel: FeedImageViewModel<UIImage>) {
        self.viewModel = viewModel
    }
    
    func view() -> UITableViewCell {
        let cell = binded(FeedImageCell())
        viewModel.loadImageData()
        return cell
    }
    
    func preload() {
        viewModel.loadImageData()
    }
    
    func cancelLoad() {
        viewModel.cancelImageDataLoad()
    }
    
    private func binded(_ cell: FeedImageCell) -> FeedImageCell {
        cell.locationContainer.isHidden = !viewModel.hasLocation
        cell.locationLabel.text = viewModel.location
        cell.descriptionLabel.text = viewModel.description
        cell.onRetry = viewModel.loadImageData
        
        viewModel.onImageLoad = { [weak cell] image in
            cell?.feedImageView.image = image
        }
        
        viewModel.onImageLoadingStateChange = { [weak cell] isLoading in
            cell?.feedImageContainer.isShimmering = isLoading
        }
        
        viewModel.OnShouldRetryImageLoadStateChange = { [weak cell] shouldRetry in
            cell?.feedImageRetryButton.isHidden = !shouldRetry
        }
                
        return cell
    }
}
```

### 5. The `FeedUIComposer` (Injecting the Transformer)
This is where the platform-agnostic `ViewModel` is connected to the platform-specific `UIImage`

```Swift
public final class FeedUIComposer {
    private init() {}
    
    public static func feedComposedWith(feedLoader: FeedLoader, imageLoader: FeedImageDataLoader) -> FeedViewController {
        let feedViewModel = FeedViewModel(feedLoader: feedLoader)
        let refreshController = FeedRefreshViewController(viewModel: feedViewModel)
        let feedController = FeedViewController(refreshController: refreshController)
        feedViewModel.onFeedLoad = adaptFeedToCellControllers(forwardingTo: feedController, loader: imageLoader)
        return feedController
    }
    
    private static func adaptFeedToCellControllers(forwardingTo controller: FeedViewController, loader: FeedImageDataLoader) -> ([FeedImage]) -> Void {
        return { [weak controller] feed in
            controller?.tableModel = feed.map { model in
                FeedImageCellController(viewModel: FeedImageViewModel(
                    model: model,
                    imageLoader: loader,
                    imageTransformer: UIImage.init // <-- The UIKit dependency is injected here
                ))
            }
        }
    }
}
```

