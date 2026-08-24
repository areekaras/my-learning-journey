# Learning Session: Validating Acceptance Criteria with High-Level UI Tests, Controlling Network and App State in UI tests with Launch Arguments and Conditional Compilation Directives

---

### **Metadata**
- **Date:** `12-08-2026`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/12730725

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
1. https://github.com/areekaras/EssentialFeed/pull/27

---

### Validating Acceptance Criteria with High-Level UI Tests, Controlling Network and App State
In this lecture, you’ll learn how to leverage XCTest and high-level UI tests to validate your app’s acceptance criteria. Moreover, you’ll learn how to replace and control infrastructure details (e.g., network and database state) during UI tests, and how to use conditional compilation directives to separate debug and test-specific code from production code to increase your app’s security, maintainability, and testability.

### Learning Outcomes
* Validating your app’s acceptance criteria with high-level UI tests.
* Replacing and controlling flow logic and state in black-box UI tests.
* Utilizing conditional compilation directives to safeguard your app from debug- and test-specific details.
* Subclassing and extending components to remove conditional logic resulting in clean, decoupled, maintainable, and testable components.

---

### UI Testing in Apple’s Ecosystem
> “UI testing gives you the ability to find and interact with the UI of your app in order to validate the properties and state of the UI elements.
>
> UI testing includes UI recording, which gives you the ability to generate code that exercises your app’s UI the same way you do, and which you can expand upon to implement UI tests.”—Apple Docs

UI testing allows you to test your app as a “black-box.” You interact with and validate the UI elements of your app using the XCTest APIs that integrate with Accessibility controls. In other words, UI tests are decoupled from production code. UI tests don’t have access to any concrete implementations of your app, as you would typically with unit/isolated or integration tests. In UI tests, you can only interact with your app through the UI.

Thus, unlike unit/isolated and integration tests, UI tests require a running application to execute. Running the application and interacting with UI elements make UI testing a costly testing strategy as it often introduces flakiness in test results and can take a long time to run. That’s why UI tests allocate a very small portion of the testing strategy pyramid (we want only very few UI tests—if any).

<img width="560" height="294" alt="image" src="https://github.com/user-attachments/assets/b2c1de68-53b8-4ce6-864d-3d814c0c581f" />

---

### The Economics of UI Testing
Automated testing is an essential practice to guarantee the correctness and quality of your apps and a productive development process. It can provide you, your team, and the business with the necessary confidence to continue implementing the company’s vision and building great products. But it can also become a liability.

So, let’s look at some numbers to clarify the importance of following the testing pyramid. In the project so far, there are 152 tests:
* 3 UI tests that, in total, take around 27 seconds to run.
* 27 integration tests (performing HTTP requests and reading/writing to disk) that, in total, take around 8 seconds to run.
* 122 unit/isolated tests that, in total, take less than a second to run.

<img width="539" height="273" alt="image" src="https://github.com/user-attachments/assets/9d75b09f-c556-49fe-839d-76c62cee6649" />

The following graph depicts the vast difference in test count vs. duration between the automated testing strategies used in the system:

<img width="1181" height="414" alt="image" src="https://github.com/user-attachments/assets/52382b1e-2d31-4566-935d-10929ab90d21" />

The graph is consistent with the economics of testing strategies presented in the testing pyramid. Observe the difference between the UI vs. Unit/Isolated tests. The UI tests (tallest purple bar) take 27x more time to run than Unit/Isolated tests even though we have 40x more Unit/Isolated tests (peak of the blue line).

As the project grows and more features are developed in the codebase, these numbers will grow as well. Thus, to keep productivity high, we want to rely more on fast unit/isolated tests and keep the number of UI tests low or nonexistent. Remember that a good test suite is fast and reliable. So, you need a solid foundation of unit/isolated tests as your team’s primary testing strategy (the base of the testing strategy pyramid).

So, although UI tests can be one of the ways to check and guarantee the system’s correctness, it should not be the sole or primary testing strategy. Otherwise, as you can see from the graph, the UI tests can quickly grow to a point where it’ll take hours to run, becoming a bottleneck and a liability in the team’s process. Instead, when necessary, use UI tests to validate high-level acceptance criteria or high-level details that you can’t test with unit/isolated or integration tests.

---

### Acceptance Testing
Acceptance testing is the process of validating the system’s compliance with high-level acceptance criteria or business requirements. Acceptance tests can be expensive to run as they check real scenarios and the whole system running in integration (usually through the UI). 

Acceptance tests are written in a high-level language to describe the scenarios (e.g., GIVEN/WHEN/THEN). So, non-programmers can also author them. In teams following BDD or similar processes, the acceptance criteria and tests are written by business folks (e.g., business analysts), and they are implemented by QA engineers. 

There are many tools like FitNesse and Cucumber for writing high-level acceptance tests, providing an easier collaboration between business folks, QA, and developers. But ultimately, those high-level tests must be implemented by engineers, which is usually done with UI tests.

---

### Using Launch Arguments as Flags in UI Tests
To run UI tests, you must instantiate and launch an `XCUIApplication`, a proxy for an application that can be started, interacted with, and terminated. Interacting with an `XCUIApplication` is limited, as you can only interact with UI elements (like reading the elements on-screen, tapping on buttons, performing gestures…).

So if you want to put your application in a specific state, e.g., logged-in or logged-out, you must perform the login/logout dance by actually logging-in (enter “username” on the “username” field, enter “password” on the “password” field, tap on the “login” button, wait…) or logging-out (tap on “menu” button, then tap on “logout” button).

However, `XCUIApplication` allows you to pass launch arguments by modifying the `launchArguments` instance property. Launch arguments can be used as flags, directing the flow of control of your app. For example, you can pass a `-logged-out` launch argument flag to direct the app to get into the “logged-out” state on launch. You can also use launch arguments for resetting cached data, setting your app on “offline mode,” “no-animations mode,” etc.

You can access the launch arguments passed in an `XCUIApplication` instance in your production code by querying the `CommandLine.arguments` property. You can also access and parse the launch arguments through `UserDefaults`. The standard instance of `UserDefaults` contains all the provided launch arguments and through its APIs it allows the parsing of specific types, e.g. `String`, `URL`, `Bool`, `Int`, etc. for a given key.

---

### Speeding up UI Tests with Stubbed Responses
As mentioned above, launch arguments can be used as flags to direct the flow of control of your app. You can leverage the launch arguments to speed up the execution of your UI tests and make them more reliable by providing canned responses. This way, you can simulate certain aspects in your app (e.g., network requests) instead of testing against real ones.

For example, an acceptance criterion of the app is to make sure we display the cached remote feed when the app has no connectivity. One way of testing this scenario would be to set your device or simulator connectivity to offline (e.g., flight mode) when running tests. But that would require additional manual steps before running the test. In this case, you can instead pass a launch argument to direct the app to simulate the connectivity state to offline.

**Setting offline connectivity in XCUIApplication**
```swift
let offlineApp = XCUIApplication()
offlineApp.launchArguments = ["-connectivity", "offline"]
offlineApp.launch()
```

Then, on the production side, you can query the connectivity state and provide a canned response simulating the behavior of the app when it finds itself without an internet connection.

**makeRemoteClient simulating offline state**
```swift
private func makeRemoteClient() -> HTTPClient {
    switch UserDefaults.standard.string(forKey: "connectivity") {
        case "offline":
            return AlwaysFailingHTTPClient()
        default:
            return URLSessionHTTPClient(session: URLSession(configuration: .ephemeral))
    }
}
```

This way, you can reliably simulate specific scenarios during UI tests.

---

### Safeguarding Launch Arguments with Conditional Compilation Directives
Launch arguments allow you to pass directives to control the flow and state of the app at runtime. However, deploying an app containing and referencing launch argument flags can introduce security risks. Launch arguments introduced for debugging/testing the app may allow malicious users to take control of the app flow in ways they should not be able to.

A simple way to safeguard your app from such risks is to leverage Swift’s conditional compilation directives, preventing debug- and test-specific code from being deployed to production. According to Swift’s reference manual, a conditional compilation block allows code to be conditionally compiled depending on the value of one or more compilation conditions.

Every conditional compilation block begins with the `#if` compilation directive and ends with the `#endif` compilation directive.

**3. Conditional Compilation Syntax**
```swift
#if [compilation condition]
    statements
#endif
```

Unlike the condition of an `if` statement, the compilation condition is evaluated at compile time. As a result, the statements are compiled and executed only if the compilation condition evaluates to true at compile time.

For example, you can prevent releasing debug- and test-code to your customers by wrapping it in an `#if DEBUG` compilation condition:

**4. File removal guarded by #if DEBUG**
```swift
#if DEBUG
if CommandLine.arguments.contains("-reset") {
    try? FileManager.default.removeItem(at: localStoreURL)
}
#endif
```

By doing so, any code wrapped in the compilation condition will not be deployed when compiling with non-debug versions of your app (e.g., compiled with the Release build configuration). The DEBUG flag is automatically set for the Debug build configuration. But you can also create your own custom build flags.

---

### Extracting Compilation Conditionals from Production Code
So, launch arguments allow you to pass directives to control your app’s flow while debugging or testing your app. And compilation conditionals can make your production code safer by removing debug and test-specific code from release builds.

However, launch arguments combined with compilation conditionals can introduce a lot of noise to your production code, making your code extremely hard to follow and maintain. Compilation conditionals, like any form of conditional, are a procedural way of controlling the app flow. The more conditionals you add, the harder it is to follow and test a component. For example, one conditional requires at least two tests (one for each branch). Two conditionals require four tests. Three conditionals require eight tests. And so on.

As you can see, there can be an exponential growth in complexity when using conditionals everywhere. There’s a better way: **Polymorphism**.

Instead of creating multiple code paths with if/else conditionals everywhere, you can use polymorphic interfaces to remove conditionals from your code. For example, you can eliminate an if/else conditional by creating two different classes: one for the `if` branch and one for the `else` branch. Then, you can move the conditional logic to a higher-level component that will choose which class to instantiate.

In the case of the `SceneDelegate` class, there were `#if DEBUG` conditional compilation directives with debug-specific logic:

**5. SceneDelegate with #if DEBUG**
```swift
class SceneDelegate {
    private func makeRemoteClient() -> HTTPClient {
        #if DEBUG
        if UserDefaults.standard.string(forKey: "connectivity") == "offline" {
            return AlwaysFailingHTTPClient()
        }
        #endif
        return URLSessionHTTPClient(session: URLSession(configuration: .ephemeral))
    }
}
```

We removed the compilation conditionals by extending its behavior in a subclass: the `DebuggingSceneDelegate`:

**6. DebuggingSceneDelegate overriding SceneDelegate**
```swift
class SceneDelegate {
    func makeRemoteClient() -> HTTPClient {
        return URLSessionHTTPClient(session: URLSession(configuration: .ephemeral))
    }
}

class DebuggingSceneDelegate: SceneDelegate {
    override func makeRemoteClient() -> HTTPClient {
        if UserDefaults.standard.string(forKey: "connectivity") == "offline" {
            return AlwaysFailingHTTPClient()
        }
        return super.makeRemoteClient()
    }
}
```

The `SceneDelegate` class has a polymorphic interface as you can subclass and change its behavior while maintaining the same interface. So clients of the `SceneDelegate` can also interact with the `DebuggingSceneDelegate` or any other subclass without even knowing about it.

Such a strategy leaves the `SceneDelegate` clean, isolated, and agnostic of any debug or test-specific compilation conditionals or launch arguments.

You can then push the conditional logic for choosing which scene delegate class to use to a higher-level component: the `AppDelegate`.

**7. AppDelegate injecting DebuggingSceneDelegate**
```swift
#if DEBUG
configuration.delegateClass = DebuggingSceneDelegate.self
#endif
```

By extracting any compilation conditionals from the `SceneDelegate`, we guarantee that production code is not mixed with components created for testing purposes. Such a separation will help you and your team with maintaining, testing, and extending your code in the future.

---

### Should we always write UI tests?
No, you don’t always need to write automated UI tests. In our experience, a good combination of unit/isolated tests and integration tests can be enough. However, some businesses require such high-level acceptance tests going through the UI.

Even if your current project requires UI tests, we recommend you not to rely on it as your sole or primary strategy. Instead, focus on creating a solid foundation of fast and reliable unit/isolated tests. Then, a couple of integration tests. And finally, if needed, very few black-box UI tests validating the high-level acceptance criteria.

---

### Who should write UI tests?
Usually, QA engineers and developers. Since it’s possible to write UI tests by recording interactions with the app, anyone with a basic understanding of Xcode and Swift can write and maintain them. Thus, it’s common to see QA engineers that are not full-time Swift developers writing UI tests.

But working with QA engineers that write high-level tests doesn’t mean you don’t need to test your code. As a developer, you still have the responsibility of creating a solid foundation of fast and reliable unit/isolated tests to continuously deliver high-quality apps and relentlessly improve the system’s design and the team's process.

---

### Senior & Architect Interview Perspective

**Q: How do launch arguments passed to XCUIApplication alter state during black-box UI testing?**
> "In black-box UI tests, test code cannot directly access production objects or memory. `XCUIApplication.launchArguments` allows us to pass flag strings (like `-reset` or `-connectivity offline`) to the target application process. Inside the app, we read these flags via `CommandLine.arguments` or `UserDefaults.standard` at launch to conditionally configure infrastructure components like network clients or local database storage."

**Q: What is the architectural benefit of using subclassing (Polymorphism) over inline `#if DEBUG` statements in SceneDelegate?**
> "Adding `#if DEBUG` and `CommandLine` logic directly inside production methods pollutes production code with test/debug concerns and increases cyclomatic complexity. By subclassing `SceneDelegate` into a `DebuggingSceneDelegate`, production components remain clean, isolated, and focused solely on production logic. The debug/test overrides live entirely in the subclass, and the high-level composition root (`AppDelegate`) chooses which class to instantiate at compile-time."

**Q: Why shouldn't an iOS app rely solely or primarily on UI tests for software verification?**
> "UI tests operate through accessibility layers and require launching the full application context, making them orders of magnitude slower and significantly more prone to flakiness than unit or integration tests. Relying heavily on UI tests creates a brittle, slow CI build process that severely lowers developer velocity. A balanced strategy uses fast, deterministic unit/isolated tests at the base, fewer integration tests in the middle, and minimal end-to-end UI tests at the top for acceptance criteria verification."

---

### Code Snippets



**8. Final Code: AppDelegate**
```swift
@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    func application(_ application: UIApplication, configurationForConnecting connectingSceneSession: UISceneSession, options: UIScene.ConnectionOptions) -> UISceneConfiguration {
        let configuration = UISceneConfiguration(name: "Default configuration", sessionRole: connectingSceneSession.role)
        
        #if DEBUG
        configuration.delegateClass = DebuggingSceneDelegate.self
        #endif
        
        return configuration
    }
}
```

**9. Final Code: DebuggingSceneDelegate & DebuggingHTTPClient**
```swift
import UIKit
import EssentialFeed

#if DEBUG
class DebuggingSceneDelegate: SceneDelegate {
    
    override func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        guard let _ = (scene as? UIWindowScene) else { return }
        
        if CommandLine.arguments.contains("-reset") {
            try? FileManager.default.removeItem(at: localStoreURL)
        }
        
        super.scene(scene, willConnectTo: session, options: connectionOptions)
    }
    
    override func makeRemoteClient() -> HTTPClient {
        if let connectivity = UserDefaults.standard.string(forKey: "connectivity") {
            return DebuggingHTTPClient(connectivity: connectivity)
        }
        return super.makeRemoteClient()
    }
}

private class DebuggingHTTPClient: HTTPClient {
    private class Task: HTTPClientTask {
        func cancel() {}
    }
    
    private let connectivity: String
    
    init(connectivity: String) {
        self.connectivity = connectivity
    }
    
    func get(from url: URL, completion: @escaping (HTTPClient.Result) -> Void) -> HTTPClientTask {
        switch connectivity {
        case "online":
            completion(.success(makeSuccessfulResponse(for: url)))
            
        default:
            completion(.failure(NSError(domain: "offline", code: 0)))
        }
        
        return Task()
    }
    
    private func makeSuccessfulResponse(for url: URL) -> (Data, HTTPURLResponse) {
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
        let rect = CGRect(x: 0, y: 0, width: 1, height: 1)
        UIGraphicsBeginImageContext(rect.size)
        let context = UIGraphicsGetCurrentContext()!
        context.setFillColor(UIColor.red.cgColor)
        context.fill(rect)
        let img = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        return img!.pngData()!
    }
    
    private func makeFeedData() -> Data {
        return try! JSONSerialization.data(withJSONObject: ["items": [
            ["id": UUID().uuidString, "image": "[http://image.com](http://image.com)"],
            ["id": UUID().uuidString, "image": "[http://image.com](http://image.com)"]
        ]])
    }
}
#endif
```

**10. Final Code: SceneDelegate**
```swift
import UIKit
import CoreData
import EssentialFeed
import EssentialFeediOS

class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?
    
    let localStoreURL = NSPersistentContainer
        .defaultDirectoryURL()
        .appendingPathComponent("feed-store.sqlite")

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        guard let _ = (scene as? UIWindowScene) else { return }
        
        let remoteURL = URL(string: "[https://static1.squarespace.com/static/5891c5b8d1758ec68ef5dbc2/t/5db4155a4fbade21d17ecd28/1572083034355/essential_app_feed.json](https://static1.squarespace.com/static/5891c5b8d1758ec68ef5dbc2/t/5db4155a4fbade21d17ecd28/1572083034355/essential_app_feed.json)")!
        
        // let remoteURL = URL(string: "[https://ile-api.essentialdeveloper.com/essential-feed/v1/feed](https://ile-api.essentialdeveloper.com/essential-feed/v1/feed)")!
        
        let remoteClient = makeRemoteClient()
        let remoteFeedLoader = RemoteFeedLoader(url: remoteURL, client: remoteClient)
        let remoteImageLoader = RemoteFeedImageDataLoader(client: remoteClient)
        
        let localStore = try! CoreDataFeedStore(storeURL: localStoreURL)
        let localFeedLoader = LocalFeedLoader(store: localStore, currentDate: Date.init)
        let localImageLoader = LocalFeedImageDataLoader(store: localStore)
        
        window?.rootViewController = FeedUIComposer.feedComposedWith(
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
        )
    }
    
    func makeRemoteClient() -> HTTPClient {
        return URLSessionHTTPClient(session: URLSession(configuration: .ephemeral))
    }
}
```

**11. Final Code: EssentialAppUIAcceptanceTests**
```swift
final class EssentialAppUIAcceptanceTests: XCTestCase {
    
    func test_onLaunch_displaysRemoteFeedWhenCustomerHasConnectivity() {
        let app = XCUIApplication()
        app.launchArguments = ["-reset", "-connectivity", "online"]
        app.launch()
        
        let feedCells = app.cells.matching(identifier: "feed-image-cell")
        XCTAssertEqual(feedCells.count, 2)
        
        let firstImage = app.images.matching(identifier: "feed-image-view").firstMatch
        XCTAssertTrue(firstImage.exists)
    }
    
    func test_onLaunch_displaysCachedRemoteFeedWhenCustomerHasNoConnectivity() {
        let onlineApp = XCUIApplication()
        onlineApp.launchArguments = ["-reset", "-connectivity", "online"]
        onlineApp.launch()
        
        let offlineApp = XCUIApplication()
        offlineApp.launchArguments = ["-connectivity", "offline"]
        offlineApp.launch()
        
        let feedCells = offlineApp.cells.matching(identifier: "feed-image-cell")
        XCTAssertEqual(feedCells.count, 2)
        
        let firstImage = offlineApp.images.matching(identifier: "feed-image-view").firstMatch
        XCTAssertTrue(firstImage.exists)
    }
    
    func test_onLaunch_displaysEmptyFeedWhenCustomerHasNoConnectivityAndNoCache() {
        let app = XCUIApplication()
        app.launchArguments = ["-reset", "-connectivity", "offline"]
        app.launch()
        
        let feedCells = app.cells.matching(identifier: "feed-image-cell")
        XCTAssertEqual(feedCells.count, 0)
    }
}
```


---

