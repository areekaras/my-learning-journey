# Learning Session: Persisting/Retrieving Models with Codable+FileSystem, Test-driving in Integration with Real Frameworks Instead of Mocks & Measuring Test times overhead with `xcodebuild`
---

### **Metadata**
- **Date:** `21-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/9929059

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
# Notes: TDD for Infrastructure (`CodableFeedStore`)

This lesson covers test-driving a concrete infrastructure component (`CodableFeedStore`) that interacts with real frameworks (`Codable` and `FileManager`), along with the trade-offs of this approach.

---

### Key Framework Concepts: `Codable` & `FileManager`

Before diving into the notes, here's a quick refresher on the frameworks used.

-   **`Codable`**: A protocol in Swift (a `typealias` for `Encodable & Decodable`) that lets you easily serialize and deserialize your custom data types to and from formats like JSON or Property Lists.
-   **`JSONEncoder` / `JSONDecoder`**: These are the workhorses for `Codable`. You tell an encoder to `encode(myCodableObject)` to get `Data`, and a decoder to `decode(MyType.self, from: data)` to get your object back.
-   **`FileManager`**: The
    API you use to interact with the file system—creating, reading, writing, and deleting files and directories.
-   **`PropertyListEncoder` / `PropertyListDecoder`**: These work just like the JSON ones, but they use Apple's "Property List" (`.plist`) format. It's an older, Apple-specific XML or binary format. JSON is now the standard for almost all web and cross-platform communication.

---

### 1. The Dilemma: Framework-Agnostic vs. Convenience

This is a core architectural trade-off.

-   **The Goal**: We want our `Feed Cache` module to be **framework-agnostic**. The core model, `LocalFeedImage`, should not know *how* it's being saved.
-   **The Temptation**: Swift makes it "easy" by just adding `: Codable` to `LocalFeedImage`.
-   **The Problem with Convenience**: Adding `: Codable` **leaks infrastructure details** into our domain model. Our `LocalFeedImage` is now coupled to the *implementation detail* that it will be saved using `Codable`. This creates rigidity and sends the wrong message (that it's okay to add framework details to domain models).

-   <img width="1533" height="748" alt="image" src="https://github.com/user-attachments/assets/17745c69-58ee-4e85-94af-b4b1050fc101" />

-   **The Solution: The Adapter / DTO Pattern**
    1.  Keep `LocalFeedImage` (the domain model) **clean and framework-agnostic**.
    2.  Create a **new, `private struct CodableFeedImage`** *inside* the `CodableFeedStore`.
    3.  This new `CodableFeedImage` (a DTO) conforms to `Codable` and mirrors the domain model.
    4.  Create simple mappers to translate:
        -   `[LocalFeedImage]` -> `[CodableFeedImage]` before saving.
        -   `[CodableFeedImage]` -> `[LocalFeedImage]` after loading.
-   **The Benefit**: This small extra step creates a strong **Anti-Corruption Layer**. The `Codable` requirement is now a private implementation detail of the `CodableFeedStore`, and our domain models remain pure.

-   <img width="1533" height="748" alt="image" src="https://github.com/user-attachments/assets/0d6a2304-5177-41cf-a892-506a9cab40db" />

```Swift
class CodableFeedStore {
    
    private struct Cache: Codable {
        let feed: [CodableFeedImage]
        let timestamp: Date
        
        var localFeed: [LocalFeedImage] {
            return feed.map { $0.local }
        }
    }
    
    private struct CodableFeedImage: Codable {
        private let id: UUID
        private let description: String?
        private let location: String?
        private let url: URL
        
        init( _ image: LocalFeedImage) {
            self.id = image.id
            self.description = image.description
            self.location = image.location
            self.url = image.url
        }

        var local: LocalFeedImage {
            return LocalFeedImage(id: id, description: description, location: location, url: url)
        }
    }

    private let storeURL: URL
    
    init(storeURL: URL) {
        self.storeURL = storeURL
    }
    
    func retrieve(completion: @escaping FeedStore.RetrievalCompletion) {
        guard let data = try? Data(contentsOf: storeURL) else {
            return completion(.empty)
        }
        
        let decoder = JSONDecoder()
        let cache = try! decoder.decode(Cache.self, from: data)
        completion(.found(feed: cache.localFeed, timestamp: cache.timestamp))
    }
    
    func insert(_ feed: [LocalFeedImage], timestamp: Date, completion: @escaping FeedStore.InsertionCompletion) {
        let encoder = JSONEncoder()
        let cache = Cache(feed: feed.map(CodableFeedImage.init), timestamp: timestamp)
        let encoded = try! encoder.encode(cache)
        try! encoded.write(to: storeURL)
        completion(nil)
    }

}
```

---

### 2. Testing Stateful Components (with Side-Effects)

-   **The Challenge**: We're testing *in integration* with the real `FileManager`, which has side-effects (it writes files to disk). One test can leave "dirty" state on the disk that causes *other* tests to fail.
-   **The Wrong Fix**: Only cleaning up in `tearDown()`. If a test crashes or is stopped at a breakpoint, `tearDown()` is never called, and the disk remains dirty.
-   **The Correct Fix: `setUp()` AND `tearDown()`**:
    -   **`setUp()`**: *Before* each test, delete any artifacts to guarantee a clean state.
    -   **`tearDown()`**: *After* each test, delete artifacts to be a good citizen.
-   This "before and after" approach makes your stateful tests robust and reliable.

```Swift
final class CodableFeedStoreTests: XCTestCase {
    
    override func setUp() {
        super.setUp()
        
        setupEmptyStoreState()
    }
    
    override func tearDown() {
        super.tearDown()
        
        undoStoreSideEffects()
    }

  // Rest of the code .....

    // MARK: - Helpers
    
    private func setupEmptyStoreState() {
        deleteStoreArtifacts()
    }
    
    private func undoStoreSideEffects() {
        deleteStoreArtifacts()
    }
    
    private func deleteStoreArtifacts() {
        try? FileManager.default.removeItem(at: testSpecificStoreURL())
    }
}
```

---

### 3. Avoiding Shared State in Tests

-   **The Problem**: Hardcoding a file URL (e.g., `"image-feed.store"`) is bad. If multiple tests run in parallel, they will all try to write to the same file, causing race conditions and flaky tests.
-   **The Solution: Explicit URL Injection**:
    1.  **Inject the `storeURL`** into the `CodableFeedStore`'s `init`.
    2.  In your tests, create a **test-specific URL** (e.g., one that uses the test class's name).
    3.  Use the `.cachesDirectory`, which is a "discardable" directory the OS can clean up.
-   **The Benefit**: This makes tests run in isolation and even enables parallel test execution. It also makes the production code more reusable (e.g., for supporting multiple user accounts, each with their own store URL).

```Swift
private func testSpecificStoreURL() -> URL {
  return FileManager.default.urls(for: .cachesDirectory, in: .userDomainMask).first!.appendingPathComponent("\(type(of: self)).store")
}
```

---

### 4. Measuring Test Performance

-   **The Fear**: Testing with real I/O (disk/network) is "slow."
-   **The Reality**: We must **measure, don't guess**.
-   **How**: Run `xcodebuild` from the command line to get precise timings.
-   **The Result**: The new tests (with real file I/O) added only **0.018 seconds** to the total test run. This is an 11% increase, but the *absolute* time is unnoticeable.
-   **The Takeaway**: For a *few* integration tests, the overhead is tiny and worth the confidence. A slow test suite is usually a sign of *too many* integration tests, which means your business logic is probably mixed in with your framework code.

-   <img width="545" height="800" alt="image" src="https://github.com/user-attachments/assets/85238145-c0b9-4140-8174-408b832eea9d" />
-   <img width="1694" height="625" alt="image" src="https://github.com/user-attachments/assets/4a9205c8-0dc3-4a81-9978-e8d8a7665db5" />
-   <img width="1694" height="625" alt="image" src="https://github.com/user-attachments/assets/b8f948b4-857c-4714-ad1f-ff7fc660db92" />

---

### Senior & Architect Interview Perspective

**Q: "What's the difference between `JSONEncoder` and `PropertyListEncoder`?"**

> "`JSONEncoder` serializes `Codable` objects to **JSON**, which is the universal, text-based standard for web APIs. `PropertyListEncoder` serializes to Apple's proprietary **Property List** (`.plist`) format, which can be binary or XML. For any modern development involving network communication, you'll use `JSONEncoder`. `PropertyListEncoder` is mostly for Apple-internal app preferences or configurations."

**Q: "Your domain model `LocalFeedImage` needs to be saved. Should you just make it conform to `Codable`?"**

> "No, I would avoid that. Making the domain model `Codable` is a **"conformist" approach** that leaks infrastructure details into my core logic. It couples my domain model to a specific persistence framework.
>
> The better solution is to create a **private DTO** (e.g., `CodableFeedImage`) *inside* my persistence layer (the `CodableFeedStore`). This DTO conforms to `Codable`. My store is then responsible for mapping the domain model to this DTO before saving. This creates an **Anti-Corruption Layer**, keeps my domain model pure, and makes my architecture framework-agnostic."

**Q: "You're writing tests for a class that writes to the disk. How do you manage side-effects to prevent flaky tests?"**

> "I manage side-effects in two ways:
>
> 1.  **Isolation**: I use **Dependency Injection** to pass a unique `storeURL` to my component, so each test writes to a different, test-specific file. This prevents tests from colliding and enables parallel execution.
> 2.  **Lifecycle**: I implement *both* `setUp()` and `tearDown()`. `setUp()` deletes any old artifacts *before* the test runs, guaranteeing a clean state. `tearDown()` cleans up *after* the test, just in case. This is crucial because `tearDown` isn't called if a test crashes, but `setUp` will always run for the *next* test, ensuring the suite remains stable."

---
