# Learning Session: MVP: Create a Reusable and Cross-platform Presentation Layer, Implementing Service Adapters, and Solving Cyclic Dependencies & Memory management Issues with the Proxy Pattern

---

### **Metadata**
- **Date:** `06-Nov-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/11297259

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

# Notes: MVP, Adapters, Proxies, and Cyclic Dependencies

This lecture refactors our MVVM implementation to **MVP (Model-View-Presenter)**. This move helps create an even cleaner separation of concerns, illustrates a different architectural pattern, and introduces powerful solutions for memory management and dependency cycles.

-   <img width="2714" height="1978" alt="image" src="https://github.com/user-attachments/assets/af93baab-69f4-40ae-aeb5-90773a6c7638" />

---

### 1. Understanding MVP (Model-View-Presenter)

-   **What it is**: A UI pattern with three components:
    -   **Model**: The business logic (e.g., `<FeedLoader>`).
    -   **Presenter**: A component responsible for **presentation logic**. It takes data from the Model, transforms it into a "ViewModel" data structure, and tells the View to display it.
    -   **View**: A passive interface (e.g., a protocol like `<FeedLoadingView>`) that the Presenter talks to. The concrete implementation (e.g., `FeedRefreshViewController`) just does what the Presenter tells it.

-   <img width="1000" height="100" alt="image" src="https://github.com/user-attachments/assets/698455d7-fe46-4feb-a4bb-2b46d411b309" />

---

### 2. MVP vs. MVVM vs. MVC (The Key Differences)

This is a critical concept for senior-level interviews.

-   **MVC (Apple's)**: The **Controller** knows about the concrete **View** (`UIRefreshControl`). The View doesn't know about the Controller.
-   **MVVM (Ours)**: The **ViewModel** (a stateful class) has *no reference* to the View. A **Binder** (our `FeedRefreshViewController`) observes the ViewModel and updates the View.
-   **MVP (Ours)**: The **Presenter** holds a reference to an **Abstract View** (a protocol like `<FeedLoadingView>`). The View is "dumb" and just implements this protocol.

-   <img width="1000" height="533" alt="image" src="https://github.com/user-attachments/assets/c71f3e79-bf65-4aa9-bf6f-ee3672e20852" />

---

### 3. The "ViewModel" in MVP (Stateless Data)

-   This is a major point of confusion.
-   In **MVVM**, our `FeedViewModel` was a `class` with dependencies and logic (e.g., `loadFeed()`).
-   In **MVP**, the "ViewModel" (e.g., `FeedLoadingViewModel`) is a **stateless, "dumb" `struct`** that *only* holds formatted data (e.g., `let isLoading: Bool`).
-   The **Presenter** *creates* this `ViewModel` struct and *passes* it to the `View`.

---

### 4. The Problem: Cyclic Dependency (Chicken & Egg)

-   In classic MVP, the View needs a reference to the Presenter (to send events), and the Presenter needs a reference to the View (to send updates).
-   This creates a "chicken or the egg" causality dilemma:
    -   `class Chicken { init(egg: Egg) }`
    -   `class Egg { init(chicken: Chicken) }`
    -   `let egg = Egg(chicken: ???)` // Can't create a Chicken first
    -   `let chicken = Chicken(egg: ???)` // Can't create an Egg first
-   This is a **cyclic dependency**. You can't instantiate one without the other. This also creates a **retain cycle** (`Presenter` holds `View` strongly, `View` holds `Presenter` strongly).

```Swift
class Chicken {
    private let egg: Egg

    init(egg: Egg) { 
        self.egg = egg
    }
}

class Egg {
    private let chicken: Chicken
    
    init(chicken: Chicken) { 
        self.chicken = chicken
    }
}

let egg = Chicken(egg: Egg(chicken: ???))
```

```Swift
// Presentation Layer

protocol View {
    func display(_ viewModel: ViewModel)
}

class Presenter {
    private let view: View

    init(view: View) { 
        self.view = view
    }
}

// View Layer

class ConcreteView: View {
    private let presenter: Presenter
    
    init(presenter: Presenter) { 
        self.presenter = presenter
    }
}

// Composition Layer

let presenter = Presenter(view: ConcreteView(presenter: ???))
```

-   <img width="1000" height="390" alt="image" src="https://github.com/user-attachments/assets/9e71d7ae-1a0e-4304-bc9f-2b198ff24da3" />
-   <img width="1000" height="309" alt="image" src="https://github.com/user-attachments/assets/158f271d-b556-459d-b097-540db58905e3" />
-   <img width="1000" height="291" alt="image" src="https://github.com/user-attachments/assets/fe5dfc2a-e351-468e-b504-4bad055338e6" />
-   <img width="1000" height="284" alt="image" src="https://github.com/user-attachments/assets/b4424e7f-1d9a-4ede-95bf-bc8d3b50c4f7" />
-   <img width="1000" height="287" alt="image" src="https://github.com/user-attachments/assets/d1ebfe68-20c4-4741-a0bb-b8fd518eaced" />

---

### 5. Solution Variations for MVP

You don't have to use one "pure" MVP. You can solve the cycle in several ways.

-   **Variation 1: Property Injection (The "Leaky" Fix)**
    -   You can break the `init` cycle by making one of the dependencies an `Optional var` (Property Injection).
    -   `class Presenter { var view: View? }`
    -   `let presenter = Presenter()`
    -   `let view = View(presenter: presenter)`
    -   `presenter.view = view`
    -   **Problem**: This "works," but it's a leaky abstraction. The `Presenter` now has to deal with an `Optional` view, which is a composition detail.

```Swift
// Presentation Layer

protocol View {
    func display(_ viewModel: ViewModel)
}

class Presenter {
    var view: View?
}

// View Layer

class ConcreteView: View {
    private let presenter: Presenter
    
    init(presenter: Presenter) { 
        self.presenter = presenter
    }

    func display(_ viewModel: ViewModel) { … }
}

// Composition Layer

let presenter = Presenter()
presenter.view = ConcreteView(presenter: presenter)
```

```Swift
// Presentation Layer

protocol View {
    func display(_ viewModel: ViewModel)
}

class Presenter {
    private let view: View

    init(view: View) { 
        self.view = view
    }
}

// View Layer

class ConcreteView: View {
    private let presenter: Presenter
    
    init(presenter: Presenter) { 
        self.presenter = presenter
    }

    func display(_ viewModel: ViewModel) { … }
}

// Composition Layer

class LazyViewVirtualProxy: View {
    var view: View?

    func display(_ viewModel: ViewModel)
        view?.display(viewModel) // forwards messages
    }
}

let lazyView = LazyViewVirtualProxy()
let presenter = Presenter(view: lazyView)
lazyView.view = ConcreteView(presenter: presenter)
```

-   **Variation 2: Adapter Pattern (Our Solution)**
    -   We break the cycle by introducing an **Adapter** (`FeedLoaderPresentationAdapter`). This adapter sits between the View and the Presenter.
    -   The `FeedRefreshViewController` (View) *doesn't* know about the Presenter. It just has a `delegate` (our Adapter).
    -   The `Adapter` (the delegate) receives the refresh event, calls the `FeedLoader` (Model), and on completion, tells the `FeedPresenter` to format the data.
    -   The `FeedPresenter` sends the formatted data back to the `FeedLoadingView` (the View).
    -   This breaks the direct cycle: the View no longer depends on the Presenter.

    -   <img width="1000" height="322" alt="image" src="https://github.com/user-attachments/assets/574a326a-e9d3-49c3-84a2-968189317188" />

-   **Variation 3: Delegate Pattern**
    -   A variation on the Adapter, where the View just delegates messages to a protocol (`<ViewDelegate>`) that the Adapter conforms to.
    -   <img width="1000" height="315" alt="image" src="https://github.com/user-attachments/assets/e2938199-9d17-4371-8958-6ad125473799" />

---

### 6. The Problem: Retain Cycles & Leaky Abstractions

-   Our `FeedPresenter` still needs to update the `FeedLoadingView` (which is our `FeedRefreshViewController`). This creates a new cycle: `Presenter -> View -> Adapter -> Presenter`.
-   A common solution is to make the `view` property `weak`:
    `private weak var loadingView: FeedLoadingView?`
-   **This is a leaky abstraction.** We are forcing the `Presenter` (a platform-agnostic component) to know about UIKit's memory management rules (that the View is a `class` and must be held `weak`ly). This pollutes our Presenter with platform-specific details.

---

### 7. The Solution: The Proxy Pattern (`WeakRefVirtualProxy`)

We can move this memory management logic to the **Composition Root** using a **Proxy**.

1.  We create a generic `WeakRefVirtualProxy<T>` class.
2.  This class *wraps* a `weak` reference to an object (`T`).
3.  It conforms to the *same protocols* as the object it's wrapping (e.g., `FeedLoadingView`).
4.  When a method (like `display(...)`) is called on the Proxy, it just *forwards* the call to the `weak` object *if it's still in memory*.

In the **Composition Root** (`FeedUIComposer`):
-   **Before**: `let presenter = FeedPresenter(view: refreshController)` (creates a retain cycle)
-   **After**:
    1.  `let proxy = WeakRefVirtualProxy(refreshController)` (wraps the view in a weak proxy)
    2.  `let presenter = FeedPresenter(view: proxy)` (Presenter holds the *proxy* strongly, but the proxy holds the *view* weakly)
-   **The Result**: The retain cycle is broken, and the `FeedPresenter` remains clean and platform-agnostic, with no `weak` properties.

---

### Senior & Architect Interview Perspective

**Q: "What are the key differences between MVC, MVVM, and MVP?"**

> "They're all patterns for separating UI concerns.
> * In **MVC**, the Controller talks directly to a concrete View.
> * In **MVVM**, the ViewModel has *no reference* to the View; they are connected by a 'Binder' (like closures, Combine, or RxSwift).
> * In **MVP**, the Presenter holds an *abstract reference* (a protocol) to the View. The View is 'dumb' and just implements that protocol."

**Q: "How is a 'ViewModel' in MVP different from a 'ViewModel' in MVVM?"**

> "This is a key distinction.
> * In **MVVM**, the ViewModel is a stateful `class` with logic. It *has* dependencies and *performs* actions (e.g., `viewModel.loadFeed()`).
> * In **MVP**, the ViewModel is a **stateless `struct`** of formatted data (e.g., `FeedLoadingViewModel`). The **Presenter** *creates* this `struct` and *passes* it to the View. It has no behavior."

**Q: "In MVP, the Presenter needs to talk to the View, and the View needs to talk to the Presenter. How do you solve the retain cycle?"**

> "This is a classic cyclic dependency. You have two main options:
> 1.  **`weak var`**: The Presenter can hold a `weak var view: ...?`. This is the simplest solution, but it leaks a UIKit memory management concept (that the View is a `class`) into the platform-agnostic Presenter.
> 2.  **Proxy Pattern**: A cleaner, more advanced solution is to use a `WeakRefVirtualProxy` in the **Composition Root**. The Presenter holds a *strong* reference to the Proxy (which conforms to the View protocol), and the Proxy holds a *weak* reference to the real View. This breaks the cycle and keeps the Presenter free of `weak` or `optional` properties."

**Q: "What is the Adapter pattern?"**

> "An Adapter is an object that **translates one interface into another**. We used it to break a cyclic dependency. Our `FeedRefreshViewController` (View) needed to tell the `FeedPresenter` to load, but that created a strong cycle.
>
> We introduced a `FeedLoaderPresentationAdapter` that acts as the `FeedRefreshViewControllerDelegate`. The View just tells its delegate 'I was refreshed.' The Adapter, which *is* the delegate, then translates that call into a `feedLoader.load()` call and passes the result to the Presenter. It 'adapts' the simple View delegate protocol to the more complex Model/Presenter interaction."

---

## Code Snippets

### 1. The `FeedPresenter` and its `ViewModel` Structs (MVP)
This shows the core MVP logic. The `FeedPresenter` is the logic, and the `ViewModels` are just "dumb" data structs.

```Swift
final class FeedPresenter {
    let feedView: FeedView
    let loadingView: FeedLoadingView
    
    init(feedView: FeedView, loadingView: FeedLoadingView) {
        self.feedView = feedView
        self.loadingView = loadingView
    }
    
    func didStartLoadingFeed() {
        loadingView.display(FeedLoadingViewModel(isLoading: true))
    }
    
    func didFinishLoadingFeed(with feed: [FeedImage]) {
        feedView.display(FeedViewModel(feed: feed))
        loadingView.display(FeedLoadingViewModel(isLoading: false))
    }
    
    func didFinishLoadingFeed(with error: Error) {
        loadingView.display(FeedLoadingViewModel(isLoading: false))
    }
}

// A "ViewModel" in MVP is a stateless struct of data
struct FeedLoadingViewModel {
    let isLoading: Bool
}

struct FeedViewModel {
    let feed: [FeedImage]
}
```

### 2. The `View` Protocols (The MVP Boundary)
The Presenter communicates with these abstract protocols, not with concrete UIKit views.

```Swift
protocol FeedLoadingView {
    func display(_ viewModel: FeedLoadingViewModel)
}

protocol FeedView {
    func display(_ viewModel: FeedViewModel)
}
```

### 3. The `FeedLoaderPresentationAdapter` (The Adapter)
This class acts as the `FeedRefreshViewControllerDelegate` and adapts its simple `didRequestFeedRefresh` call into the `feedLoader.load` and `presenter` callbacks.

```Swift
private final class FeedLoaderPresentationAdapter: FeedRefreshViewControllerDelegate {
    private let feedLoader: FeedLoader
    var presenter: FeedPresenter?
    
    init(feedLoader: FeedLoader) {
        self.feedLoader = feedLoader
    }
    
    func didRequestFeedRefresh() {
        presenter?.didStartLoadingFeed()
        
        feedLoader.load { [weak self] result in
            switch result {
            case let .success(feed):
                self?.presenter?.didFinishLoadingFeed(with: feed)
                
            case let .failure(error):
                self?.presenter?.didFinishLoadingFeed(with: error)
            }
        }
    }
}
```

### 4. The `WeakRefVirtualProxy` (The Proxy)
A generic class that holds a `weak` reference to an object and forwards messages to it, breaking retain cycles.

```Swift
private class WeakRefVirtualProxy<T: AnyObject> {
    private weak var object: T?
    
    init(_ object: T) {
        self.object = object
    }
}

// The proxy conditionally conforms to the protocols of its wrapped object.
extension WeakRefVirtualProxy: FeedLoadingView where T: FeedLoadingView {
    func display(_ viewModel: FeedLoadingViewModel) {
        object?.display(viewModel)
    }
}

extension WeakRefVirtualProxy: FeedImageView where T: FeedImageView, T.Image == UIImage {
    func display(_ viewModel: FeedImageViewModel<UIImage>) {
        object?.display(viewModel)
    }
}
```

### 5. The `FeedUIComposer` (Wiring with Proxies and Adapters)
This shows the **Composition Root** in action, connecting all the pieces.

```Swift
public final class FeedUIComposer {
    private init() {}
    
    public static func feedComposedWith(feedLoader: FeedLoader, imageLoader: FeedImageDataLoader) -> FeedViewController {
        // 1. Create the Adapter
        let presentationAdapter = FeedLoaderPresentationAdapter(feedLoader: feedLoader)

        // 2. Create the View (Controller), injecting the Adapter as its delegate
        let refreshController = FeedRefreshViewController(delegate: presentationAdapter)
        let feedController = FeedViewController(refreshController: refreshController)

        // 3. Create the Presenter, wrapping the view in a WeakRefProxy to break the retain cycle
        presentationAdapter.presenter = FeedPresenter(
            feedView: FeedViewAdapter(controller: feedController, imageLoader: imageLoader),
            loadingView: WeakRefVirtualProxy(refreshController))
            
        return feedController
    }
}
```

### 6. The `FeedRefreshViewController` (The "View")
The refactored controller is now just a simple View component. It has no logic, only a `delegate`.

```Swift
protocol FeedRefreshViewControllerDelegate {
    func didRequestFeedRefresh()
}

public final class FeedRefreshViewController: NSObject, FeedLoadingView {
    public lazy var view = loadView()
    
    private let delegate: FeedRefreshViewControllerDelegate
    
    init(delegate: FeedRefreshViewControllerDelegate) {
        self.delegate = delegate
    }
        
    @objc func refresh() {
        delegate.didRequestFeedRefresh()
    }
    
    func display(_ viewModel: FeedLoadingViewModel) {
        if viewModel.isLoading {
            view.beginRefreshing()
        } else {
            view.endRefreshing()
        }
    }
    
    private func loadView() -> UIRefreshControl {
        let view = UIRefreshControl()
        view.addTarget(self, action: #selector(refresh), for: .valueChanged)
        return view
    }
}
```

### 7. The `FeedImageCellController` (The "View")
This controller is also now just a View component with a delegate.

```Swift
protocol FeedImageCellControllerDelegate {
    func didRequestImage()
    func didCancelImageRequest()
}

final class FeedImageCellController: FeedImageView {
    private let delegate: FeedImageCellControllerDelegate
    private lazy var cell = FeedImageCell()
    
    init(delegate: FeedImageCellControllerDelegate) {
        self.delegate = delegate
    }
    
    func view() -> UITableViewCell {
        delegate.didRequestImage()
        return cell
    }
    
    func preload() {
        delegate.didRequestImage()
    }
    
    func cancelLoad() {
        delegate.didCancelImageRequest()
    }
    
    func display(_ viewModel: FeedImageViewModel<UIImage>) {
        // ... configures the cell from the ViewModel ...
        cell.onRetry = delegate.didRequestImage
    }
}
```

---
