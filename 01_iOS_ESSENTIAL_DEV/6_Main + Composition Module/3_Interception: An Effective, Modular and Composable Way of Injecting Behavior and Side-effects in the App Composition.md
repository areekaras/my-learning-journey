# Learning Session: Interception: An Effective, Modular and Composable Way of Injecting Behavior and Side-effects in the App Composition

---

### **Metadata**
- **Date:** `11-08-2026`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/12372828

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
1. https://github.com/areekaras/EssentialFeed/pull/26

---

#### Notes

### 1. The Challenge of Interception & Injecting Side-Effects
When adding behavior (like saving remote data to a local cache) to an existing composition, we must navigate several architectural principles:
* **Single Responsibility Principle (SRP):** The Feed API module must remain single-purpose (backend communication). It should not know about caching use cases.
* **Command-Query Separation Principle (CQS):** The `RemoteFeedLoader.load` method is a *Query*. It should not perform mutations or side-effects (like saving state).
* **Liskov Substitution Principle (LSP) & Interface Segregation Principle (ISP):** We cannot simply add a `save` method to the `<FeedLoader>` protocol. Not all loaders (e.g., `RemoteFeedLoader`) can implement it (LSP violation), and not all clients need it (ISP violation).
* **Open/Closed Principle (OCP) & Composite Reuse Principle (CRP):** We must inject this side-effect without altering existing components and avoid relying on inheritance.

### 2. The Decorator Design Pattern
* **Definition:** The Decorator pattern dynamically attaches additional responsibilities to an object. It provides a flexible alternative to subclassing for extending functionality.
* **Mechanism:** It wraps one implementation of an abstraction in another implementation of the *same* abstraction. The wrapper delegates operations to the contained object, while injecting behavior before or after the invocation.
* **Dynamic Composition:** Responsibilities can be attached at runtime (unlike subclassing, which is baked in at compile time).
* **Pipelines:** Decorators can wrap other decorators, creating a flexible "pipeline" of interception.

### 3. Implementing Interception
* Interception relies entirely on a polymorphic interface (e.g., `<FeedLoader>`). 
* The `FeedLoaderCacheDecorator` acts as an implementation of `<FeedLoader>`. It forwards the `load` result from its decoratee, but intercepts successful loads to inject a `<FeedCache>.save` side-effect.
* <img width="1427" height="839" alt="image" src="https://github.com/user-attachments/assets/628dfda5-e279-4067-ba12-f50cfec7ca33" />
* <img width="1119" height="329" alt="image" src="https://github.com/user-attachments/assets/89ac4723-9a5d-4234-a934-96f4ad5a339d" />

```Swift
public protocol FeedCache {
    typealias Result = Swift.Result<Void, Error>
    func save(_ feed: [FeedImage], completion: @escaping (Result) -> Void)
}

public protocol FeedImageDataCache {
    typealias Result = Swift.Result<Void, Swift.Error>
    func save(_ data: Data, for url: URL, completion: @escaping (Result) -> Void)
}
```

```Swift
import EssentialFeed

public class FeedLoaderCacheDecorator: FeedLoader {
    private let decoratee: FeedLoader
    private let cache: FeedCache
    
    public init(decoratee: FeedLoader, cache: FeedCache) {
        self.decoratee = decoratee
        self.cache = cache
    }
    
    public func load(completion: @escaping (FeedLoader.Result) -> Void) {
        decoratee.load { [weak self] result in
            completion(result.map({ feed in
                self?.cache.saveIgnoringResult(feed)
                return feed
            }))
        }
    }
}

private extension FeedCache {
    func saveIgnoringResult(_ feed: [FeedImage]) {
        save(feed) { _ in }
    }
}

```

```Swift
import Foundation
import EssentialFeed

public final class FeedImageDataLoaderCacheDecorator: FeedImageDataLoader {
    private let decoratee: FeedImageDataLoader
    private let cache: FeedImageDataCache
    
    public init(decoratee: FeedImageDataLoader, cache: FeedImageDataCache) {
        self.decoratee = decoratee
        self.cache = cache
    }
    
    public func loadImageData(from url: URL, completion: @escaping (FeedImageDataLoader.Result) -> Void) -> any FeedImageDataLoaderTask {
        decoratee.loadImageData(from: url) { [weak self] result in
            completion(result.map({ data in
                self?.cache.saveIgnoringResult(data, for: url)
                return data
            }))
        }
    }
}

private extension FeedImageDataCache {
    func saveIgnoringResult(_ data: Data, for url: URL) {
        save(data, for: url) { _ in }
    }
}
```

---

### Senior & Architect Interview Perspective

**Q: How does the Decorator pattern help enforce the Command-Query Separation (CQS) principle in this architecture?**
> "By strictly separating concerns. The `load` method on our `RemoteFeedLoader` is a Query—it returns data without causing side-effects. However, business requirements dictate that we must cache this data. Instead of polluting the Query with a Command (saving), we wrap the loader in a Decorator. The Decorator handles the interception and executes the Command (`save`) on a separate Cache component, ensuring the original loader remains a pure Query."

**Q: Why choose a Decorator over subclassing `RemoteFeedLoader` to add caching behavior?**
> "Subclassing violates the Composite Reuse Principle and creates rigid, compile-time dependencies. If we subclassed, we would tightly couple caching to the remote loader specifically. By using a Decorator that depends on the `<FeedLoader>` abstraction, we can wrap *any* loader (remote, local, or even a fallback composite). This allows us to compose behaviors dynamically at runtime and adhere strictly to the Open/Closed Principle."

**Q: What are the architectural risks of adding a `save` method directly to the `<FeedLoader>` protocol instead of using a Decorator and a separate Cache protocol?**
> "Adding `save` directly to `<FeedLoader>` violates the Interface Segregation Principle (ISP) because clients that only need to load data are forced to depend on a save method they don't use. It also violates the Liskov Substitution Principle (LSP) because implementations like `RemoteFeedLoader` cannot logically implement a local cache save, forcing them to implement empty methods or throw unexpected errors. Segregating the interfaces keeps modules cohesive and highly decoupled."

---

