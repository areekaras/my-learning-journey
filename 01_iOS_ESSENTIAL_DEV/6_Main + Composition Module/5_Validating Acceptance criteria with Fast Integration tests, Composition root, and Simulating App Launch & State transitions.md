# Learning Session: Validating Acceptance Criteria Fast Integration Tests, Composition Root, and Simulating App Launch & State transitions

---

### **Metadata**
- **Date:** `24-08-2026`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/13004189

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
1. https://github.com/areekaras/EssentialFeed/pull/28

---
### Validating Acceptance Criteria with Fast Integration Tests, Composition Root, and Simulating App Launch & State Transitions
In this lecture, you’ll learn how to validate your app’s Acceptance Criteria with fast and thorough Integration Tests and how to create and test components of the Composition Root in iOS apps.

### Learning Outcomes
* Validating acceptance criteria with Integration Tests.
* Replacing UI Tests with significantly faster and thorough Integration Tests.
* What’s a Composition Root.
* Where’s the Composition Root in iOS apps.
* How to test components of the Composition Root.
* Simulating app launch and state transitions during tests.
* Testing methods you cannot invoke.

---

### What’s a Composition Root?
Composition Root is an essential Dependency Injection pattern to achieve a clean and modular design.

> “When you write loosely coupled code, you create many classes to create an application. It can be tempting to compose these classes at many different locations in order to create small subsystems, but that limits your ability to Intercept those systems to modify their behavior. Instead, you should compose classes in one single area of your application.
> 
> When you look at Constructor Injection in isolation, you may wonder, doesn’t it defer the decision about selecting a Dependency to another place? Yes, it does, and that’s a good thing. This means that you get a central place where you can connect collaborating classes.
> 
> The Composition Root acts as a third party that connects consumers with their services. The longer you defer the decision on how to connect classes, the more you keep your options open. Thus, the Composition Root should be placed as close to the application’s entry point as possible.”—*Dependency Injection: Principles, Practices and Patterns* by Mark Seemann, Steven van Deursen

The thing is… When creating independent modules, they should not have any reference to components that belong to another module. 

For example, if you want to decouple a Login module from a Feed module (and vice-versa), they should not reference each other. For instance, a component in the Login module should not instantiate a component from the Feed module. Furthermore, components should require their dependencies through initializer injection.

So, the idea is to create a “Main module” responsible for instantiating and composing all independent modules in a centralized place, aka the **Composition Root**.

Thus, if after login, you want to transition to the feed screen, this transition can be delegated through an abstraction that will be implemented by a component in the Composition Root. So, the modules are independent of each other. You can easily develop, test, maintain, extend, replace, and reuse them in isolation.

For example, you could easily replace the transition to a “Welcome” screen after login or stop forcing customers to login on startup altogether! You can make those changes without affecting the Login or the Feed module by simply changing the composition from the Composition Root. 

Centralizing your app’s instantiation and composition simplifies drastically the development and management of modules, components, and their dependencies. This way, you can design elegant and independent modules, where components communicate through clean abstractions. Moreover, components don’t need to be concerned about concrete dependency implementations or how to instantiate them. The dependencies can be easily injected and replaced without breaking existing code, promoting composability and extensibility (Open/Closed principle).

---

### Where’s the Composition Root?
The Composition Root should live as close as possible to the app’s entry point, or as also known, “Main.”
In iOS apps, the application entry point historically has been the `UIApplicationDelegate.didFinishLaunchingWithOptions`.
More recently, in single-window apps using the new `UIScene` APIs, the `UISceneDelegate.willConnectToSession` can be considered the app entry point.

The Composition Root is the most concrete place in your app as it instantiates the concrete components for the app’s composition. The Composition Root is an application detail. Thus, only applications should have a Composition Root. *(Frameworks shouldn’t need Composition Roots as they shouldn’t hold or depend on application-specific details!)*

The following diagram depicts how the independent modules of our application are referenced only by the Main module (EssentialApp):

<img width="737" height="414" alt="image" src="https://github.com/user-attachments/assets/15e7e83d-05a4-4a97-b1da-867d8184cab9" />

Note how the `SceneDelegate` that belongs to the Main module references components from all other modules, but no module references the `SceneDelegate` or any other component in the Main module.

At the moment, the `SceneDelegate` centralizes the instantiation and composition of our app in a single class. That’s fine for a simple project. But, as the project grows, the Composition Root can include more components for breaking down the app composition and providing better code organization.

> “The Composition Root isn’t a method or a class, it’s a concept. It can be part of the Main method, or it can span multiple classes, as long as they all reside in a single module. Separating it into its own method helps to ensure that the composition is consolidated and not otherwise interspersed with subsequent application logic.” —*Dependency Injection: Principles, Practices and Patterns* by Mark Seemann, Steven van Deursen

#### What if I’m using a DI container?
A DI Container is an application composition detail. Thus, if you’re using a DI Container, only the Composition Root should reference it.

> “Inside the composition root, you can decide to compose your application manually — that’s using Pure DI — or to delegate it to a DI container.” —*Dependency Injection: Principles, Practices and Patterns* by Mark Seemann, Steven van Deursen

Accessing a DI Container outside the Composition Root leads to the **Service Locator Anti-Pattern**.

---

### A Case for Using `@testable import`
So far, we’ve been advocating for testing components only through their public interfaces. Thus, there’s no need to import modules as `@testable` (which would give us access to internal types). By striving to test types only through the public interfaces, the tests will exert great incentives for designing your system with proper encapsulation and access control in mind.

A common question we receive is… *"When is it appropriate to use @testable then?"*

An appropriate case is when you want to test an internal type directly, but you don’t want other modules to have access to it. Thus, making it public is not a good solution.

For example, the Composition Root is the root of the object graph. Thus, the Composition Root and its components shouldn’t be referenced by other modules. You want the inverse dependency: the Composition Root should depend on other modules (never the other way around). Making sure the Composition Root types are not public is a great way to enforce such rules, as the project wouldn’t even compile if another module references the Composition Root.

In this lecture, we tested the initial app’s composition through the `SceneDelegate`, an `internal` class. The `SceneDelegate`, our app’s entry point, belongs to the Composition Root and has no reason to be a public class since it is never referenced from any other modules.

<img width="701" height="366" alt="image" src="https://github.com/user-attachments/assets/e036700d-c825-4be8-98bd-e86afe27ff31" />

In the above graph, you can observe how the `SceneDelegate` can reference public components belonging in the Feed API, Feed Cache, and Feed UI modules. On the other hand, no other module should reference the `SceneDelegate`. So, it doesn’t have to be public. To enforce other modules to not have access to the Composition Root, you can make its components internal and, when necessary, use `@testable import` to test them.

---

### Simulating App Launch & State Transitions
When testing your app with Integration Tests, simulating the app launch and state transitions (e.g., background/foreground) is just a matter of invoking a method.

For example, to test what happens when the app will transition to the background, you can simply invoke the `UISceneDelegate.sceneWillResignActive` method (or `UIApplicationDelegate.applicationWillResignActive` if you’re not using the new UIScene APIs).

### Testing Methods You Cannot Invoke
When testing your logic in integration with 3rd-party frameworks (e.g., UIKit), you may not be able to invoke some methods.

For example, to invoke the `UISceneDelegate.willConnectToSession` method, you need to pass an instance of the `UIScene.ConnectionOptions` class. However, this class doesn’t have a public initializer. Thus, if you cannot instantiate a required parameter of a method, you cannot invoke that method to test it.

But not all is lost. As a trade-off, you can move the logic within the method you cannot invoke into a method you *can* invoke!

### Convenience Initializers
Convenience initializers are secondary initializers for facilitating the creation of a class instance for a specific use case or input.

> “Create convenience initializers whenever a shortcut to a common initialization pattern will save time or make initialization of the class clearer in intent.”—*Swift docs*

You can define a convenience initializer by placing the `convenience` modifier before the `init` keyword, separated by a space:
```swift
convenience init(parameters) {
    statements
}
```

For example, the `SceneDelegate`’s designated initializer is the default `init()` without params used by UIKit to instantiate the delegate instance. To facilitate the injection of infrastructure details in the `SceneDelegate`, such as the `HTTPClient`, we created a convenience initializer.

This new way of instantiating the `SceneDelegate` allowed us to pass stubbed infrastructure implementations during tests instead of making real calls to the network and database. Thus, saving us valuable time and eliminating potential flakiness while maintaining the default `init()` used by UIKit.

### `lazy` Properties in Swift
“Lazy” or “lazy stored properties” in Swift are properties whose initial value is not calculated until the first time they are used. You can specify a lazy stored property by placing the `lazy` modifier before its declaration.

Lazy properties must always be declared as a variable (with the `var` keyword) because their initial value might be set after the instance initialization completes. Oppositely, constant properties (`let`) must always have a value before the initialization completes, and therefore cannot be declared as lazy.

Lazy properties come in handy when the initial or default value for a property can be deferred until it is required by the system (e.g., if it requires complex or computationally expensive setup).

---

### UI vs. Integration Tests Comparison
The following graph depicts the performance difference in testing the app’s composition when using integration vs. high-level UI tests.

<img width="528" height="323" alt="image" src="https://github.com/user-attachments/assets/184c6dfc-2c94-4773-953b-9e90c6cf685d" />

Even with almost double the test count, validating the app’s composition with Integration Tests has proved to be over 100x faster than testing the composition as a black-box with UI Tests.

As the project grows and more features are being developed, finding and adopting strategies that yield dramatically better results than others will be key for maintaining a sustainable development and release pace. Ignoring performance overheads and flakiness can turn your app’s testing strategy into a bottleneck slowing down the whole product development. And eventually forcing your team to stop running the tests altogether.

---

### Code Snippets

**1. AppDelegate (Removed debugging setup)**
```swift
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {}
```

**2. SceneDelegate (Updated with lazy loading and convenience init)**
```swift
import UIKit
import CoreData
import EssentialFeed

class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?
    
    private lazy var httpClient: HTTPClient = {
        return URLSessionHTTPClient(session: URLSession(configuration: .ephemeral))
    }()
    
    private lazy var store: FeedStore & FeedImageDataStore = {
        return try! CoreDataFeedStore(
            storeURL: NSPersistentContainer
                .defaultDirectoryURL()
                .appendingPathComponent("feed-store.sqlite"))
    }()
    
    private lazy var localFeedLoader: LocalFeedLoader = {
        return LocalFeedLoader(store: store, currentDate: Date.init)
    }()
    
    convenience init(httpClient: HTTPClient, store: FeedStore & FeedImageDataStore) {
        self.init()
        self.httpClient = httpClient
        self.store = store
    }

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        guard let _ = (scene as? UIWindowScene) else { return }
        
        configureWindow()
    }
    
    func configureWindow() {
        let remoteURL = URL(string: "[https://static1.squarespace.com/static/5891c5b8d1758ec68ef5dbc2/t/5db4155a4fbade21d17ecd28/1572083034355/essential_app_feed.json](https://static1.squarespace.com/static/5891c5b8d1758ec68ef5dbc2/t/5db4155a4fbade21d17ecd28/1572083034355/essential_app_feed.json)")!
        
        // let remoteURL = URL(string: "[https://ile-api.essentialdeveloper.com/essential-feed/v1/feed](https://ile-api.essentialdeveloper.com/essential-feed/v1/feed)")!
        
        let remoteFeedLoader = RemoteFeedLoader(url: remoteURL, client: httpClient)
        let remoteImageLoader = RemoteFeedImageDataLoader(client: httpClient)
        let localImageLoader = LocalFeedImageDataLoader(store: store)
        
        window?.rootViewController = UINavigationController(
            rootViewController: FeedUIComposer.feedComposedWith(
                feedLoader: FeedLoaderWithFallbackComposite(
                    primary: FeedLoaderCacheDecorator(
                        decoratee: remoteFeedLoader,
                        cache: localFeedLoader
                    ),
                    fallback: localFeedLoader
                ),
                imageLoader: FeedImageDataLoaderWithFallbackComposite(
                    primary: localImageLoader,
                    fallback: FeedImageDataLoaderCacheDecorator(
                        decoratee: remoteImageLoader,
                        cache: localImageLoader
                    )
                )
            ))
    }
    
    func sceneWillResignActive(_ scene: UIScene) {
        localFeedLoader.validateCache { _ in }
    }
}
```

**3. Test Helper: HTTPClientStub**
```swift
import Foundation
import EssentialFeed

class HTTPClientStub: HTTPClient {
    private class Task: HTTPClientTask {
        func cancel() {}
    }
    
    private let stub: (URL) -> HTTPClient.Result
    
    init(stub: @escaping (URL) -> HTTPClient.Result) {
        self.stub = stub
    }
    
    func get(from url: URL, completion: @escaping (HTTPClient.Result) -> Void) -> HTTPClientTask {
        completion(stub(url))
        return Task()
    }
}

extension HTTPClientStub {
    static var offline: HTTPClientStub {
        return .init { _ in .failure(NSError(domain: "offline", code: 0)) }
    }
    
    static func online(_ stub: @escaping (URL) -> (Data, HTTPURLResponse)) -> HTTPClientStub {
        return HTTPClientStub { url in .success(stub(url)) }
    }
}
```

**4. Test Helper: InMemoryFeedStore**
```swift
import Foundation
import EssentialFeed

class InMemoryFeedStore: FeedStore, FeedImageDataStore {
    private(set) var feedCache: CachedFeed?
    private var feedImageDataCache: [URL: Data] = [:]
    
    private init(feedCache: CachedFeed? = nil) {
        self.feedCache = feedCache
    }
    
    func deleteCachedFeed(completion: @escaping FeedStore.DeletionCompletion) {
        feedCache = nil
        completion(.success(()))
    }
    
    func insert(_ feed: [LocalFeedImage], timestamp: Date, completion: @escaping FeedStore.InsertionCompletion) {
        feedCache = CachedFeed(feed: feed, timestamp: timestamp)
        completion(.success(()))
    }
    
    func retrieve(completion: @escaping RetrievalCompletion) {
        completion(.success(feedCache))
    }
    
    func insert(_ data: Data, for url: URL, completion: @escaping (FeedImageDataStore.InsertionResult) -> Void) {
        feedImageDataCache[url] = data
        completion(.success(()))
    }
    
    func retrieve(dataForURL url: URL, completion: @escaping (FeedImageDataStore.RetrievalResult) -> Void) {
        completion(.success(feedImageDataCache[url]))
    }
}

extension InMemoryFeedStore {
    static var empty: InMemoryFeedStore {
        return InMemoryFeedStore()
    }
    
    static var withExpiredFeedCache: InMemoryFeedStore {
        return InMemoryFeedStore(feedCache: CachedFeed(feed: [], timestamp: Date.distantPast))
    }
    
    static var withNonExpiredFeedCache: InMemoryFeedStore {
        return InMemoryFeedStore(feedCache: CachedFeed(feed: [], timestamp: Date()))
    }
}
```

**5. Acceptance Tests: FeedAcceptanceTests**
```swift
import XCTest
import EssentialFeed
import EssentialFeediOS
@testable import EssentialApp

final class FeedAcceptanceTests: XCTestCase {
    func test_onLaunch_displaysRemoteFeedWhenCustomerHasConnectivity() {
        let feed = launch(httpClient: .online(response), store: .empty)
        
        XCTAssertEqual(feed.numberOfRenderedFeedImageViews(), 2)
        XCTAssertEqual(feed.simulateFeedImageViewVisible(at: 0)?.renderedImage, makeImageData())
        XCTAssertEqual(feed.simulateFeedImageViewVisible(at: 1)?.renderedImage, makeImageData())
    }
    
    func test_onLaunch_displaysCachedRemoteFeedWhenCustomerHasNoConnectivity() {
        let sharedStore = InMemoryFeedStore.empty
        let onlineFeed = launch(httpClient: .online(response), store: sharedStore)
        onlineFeed.simulateFeedImageViewVisible(at: 0)
        onlineFeed.simulateFeedImageViewVisible(at: 1)
        
        let offlineFeed = launch(httpClient: .offline, store: sharedStore)
        
        XCTAssertEqual(offlineFeed.numberOfRenderedFeedImageViews(), 2)
        XCTAssertEqual(offlineFeed.simulateFeedImageViewVisible(at: 0)?.renderedImage, makeImageData())
        XCTAssertEqual(offlineFeed.simulateFeedImageViewVisible(at: 1)?.renderedImage, makeImageData())
    }
    
    func test_onLaunch_displaysEmptyFeedWhenCustomerHasNoConnectivityAndNoCache() {
        let feed = launch(httpClient: .offline, store: .empty)
        
        XCTAssertEqual(feed.numberOfRenderedFeedImageViews(), 0)
    }
    
    func test_onEnteringBackground_deletesExpiredFeedCache() {
        let store = InMemoryFeedStore.withExpiredFeedCache
        
        enterBackground(with: store)
        
        XCTAssertNil(store.feedCache, "Expected to delete expired cache")
    }
    
    func test_onEnteringBackground_keepsNonExpiredFeedCache() {
        let store = InMemoryFeedStore.withNonExpiredFeedCache
        
        enterBackground(with: store)
        
        XCTAssertNotNil(store.feedCache, "Expected to keep non-expired cache")
    }
    
    //MARK: - Helpers
    
    private func launch(
        httpClient: HTTPClientStub = .offline,
        store: InMemoryFeedStore = .empty
    ) -> FeedViewController {
        let sut = SceneDelegate(httpClient: httpClient, store: store)
        sut.window = UIWindow()
        sut.configureWindow()
        
        let nav = sut.window?.rootViewController as? UINavigationController
        let feed = nav?.topViewController as! FeedViewController
        feed.simulateAppearence()
        return feed
    }
    
    private func enterBackground(with store: InMemoryFeedStore) {
        let sut = SceneDelegate(httpClient: HTTPClientStub.offline, store: store)
        sut.sceneWillResignActive(UIApplication.shared.connectedScenes.first!)
    }
    
    private func response(for url: URL) -> (Data, HTTPURLResponse) {
        let response = HTTPURLResponse(url: url, statusCode: 200, httpVersion: nil, headerFields: nil)!
        return (makeData(for: url), response)
    }
    
    private func makeData(for url: URL) -> Data {
        switch url.absoluteString {
        case "[http://image.com](http://image.com)":
            return makeImageData()
            
        default:
            return makeFeedData()
        }
    }
    
    private func makeImageData() -> Data {
        return UIImage.make(withColor: .red).pngData()!
    }
    
    private func makeFeedData() -> Data {
        return try! JSONSerialization.data(withJSONObject: ["items": [
            ["id": UUID().uuidString, "image": "[http://image.com](http://image.com)"],
            ["id": UUID().uuidString, "image": "[http://image.com](http://image.com)"]
        ]])
    }
}
```

**6. Acceptance Tests: SceneDelegateTests**
```swift
import XCTest
import EssentialFeediOS
@testable import EssentialApp

final class SceneDelegateTests: XCTestCase {
    
    func test_sceneWillConnectToSession_configuresRootViewController() {
        let sut = SceneDelegate()
        sut.window = UIWindow()
        
        sut.configureWindow()
        
        let root = sut.window?.rootViewController
        let rootNavigation = root as? UINavigationController
        let topController = rootNavigation?.topViewController
        
        XCTAssertNotNil(rootNavigation, "Expected a navigation controller as root, got \(String(describing: root)) instead")
        XCTAssertTrue(topController is FeedViewController, "Expected a feed controller as top view controller, got \(String(describing: topController)) instead")
    }
}
```


---


