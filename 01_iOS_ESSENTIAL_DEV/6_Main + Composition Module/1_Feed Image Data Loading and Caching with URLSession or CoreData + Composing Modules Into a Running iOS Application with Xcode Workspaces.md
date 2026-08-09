# Learning Session: Feed Image Data Loading and Caching with URLSession/CoreData + Composing Modules Into a Running iOS Application with Xcode Workspaces

---

### **Metadata**
- **Date:** `09-08-2026`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/12046318

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
1. https://github.com/areekaras/EssentialFeed/pull/22
2. https://github.com/areekaras/EssentialFeed/pull/23
3. https://github.com/areekaras/EssentialFeed/pull/24

---

# Notes: Feed Image Data Loading, Caching, and Modular Architecture

This lecture covers implementing remote and local feed image data loading, separating caching responsibilities using protocol segregation, organizing domain models per layer, and composing independent Xcode projects within a unified workspace.

---

### 1. Architectural Outcomes & Workspace Composition

- **Goals**: Combine independent Xcode projects into an Xcode Workspace and embed frameworks to compose modular components into a running iOS application.
- **Value of Specs/Use Cases**: Clarifying intent and specs *before* coding helps:
  - Break down tasks and decrease context-switching overhead.
  - Test-drive implementations (TDD) and improve team communication.
  - Reduce rework and maximize long-term maintainability.

<img width="1497" height="519" alt="image" src="https://github.com/user-attachments/assets/35ab497c-6e08-43c0-9912-36bdec601f40" />

---

### 2. Feed Image Data Loading Use Cases

#### Load Feed Image Data From Remote Use Case
- **Data**: Image `URL`.
- **Primary Course (Happy Path)**: Execute "Load Image Data" command with URL $\rightarrow$ System downloads data $\rightarrow$ System validates downloaded data $\rightarrow$ System delivers image data.
- **Cancel Course**: System cancels network task; does not deliver image data nor error.
- **Error Courses (Sad Paths)**:
  - Invalid Data: System delivers invalid data error.
  - No Connectivity: System delivers connectivity error.

#### Load Feed Image Data From Cache Use Case
- **Data**: Image `URL`.
- **Primary Course (Happy Path)**: Execute "Load Image Data" command with URL $\rightarrow$ System retrieves data from cache $\rightarrow$ System delivers cached image data.
- **Cancel Course**: System does not deliver image data nor error.
- **Error Courses (Sad Paths)**:
  - Retrieval Error: System delivers error.
  - Empty Cache: System delivers "not found" error.

#### Cache Feed Image Data Use Case
- **Data**: Binary image `Data`.
- **Primary Course (Happy Path)**: Execute "Save Image Data" command with data $\rightarrow$ System saves new cache data $\rightarrow$ System delivers success message.
- **Error Course (Sad Path)**: System delivers error.

---

### 3. Interface Segregation Principle (ISP) in Cache Stores

- **The Problem**: `LocalFeedLoader` and `LocalFeedImageDataLoader` both perform caching operations, making it tempting to reuse the existing `<FeedStore>` protocol by adding image data insertion/retrieval methods to it.
- **Why Reusing `<FeedStore>` is Flawed**:
  - `LocalFeedLoader` caches `FeedImage` data models (`[LocalFeedImage]`).
  - `LocalFeedImageDataLoader` caches raw binary image `Data`.
  - Forcing image data operations into `<FeedStore>` violates **ISP**: clients (`LocalFeedLoader`) would depend on methods they do not use or need.
- **The ISP Solution**:
  - Keep `<FeedStore>` strictly for feed models.
  - Create a new, dedicated protocol: `<FeedImageDataStore>` for `LocalFeedImageDataLoader`.
  - Both protocols hide infrastructure details (e.g., CoreData) from their respective clients.
  - `CoreDataFeedStore` implements *both* protocols under the hood as an infrastructure detail, but clients remain decoupled from unused methods.

<img width="1058" height="613" alt="image" src="https://github.com/user-attachments/assets/42118868-f047-4a92-8151-4abc95b958c5" />

---

### 4. Benefits of Separating Models Per Layer (SRP)

- **Single Responsibility Principle**: A component or model representation should have only *one reason to change*.
- **Domain vs. Infrastructure Models**:
  - `FeedImage` (Domain Model) changes due to business requirements.
  - `LocalFeedImage` (Cache Model) changes due to caching requirements.
  - `ManagedFeedImage` (CoreData Managed Object) changes due to database/performance optimizations.
- **Decoupling Impact**: Adding a `data` property to `ManagedFeedImage` in CoreData only affects the CoreData module. Because models are segregated per layer, domain modules and other infrastructure implementations do not require recompilation or redeployment.

---

### 5. Storing Binary Data in Core Data

- **SQLite Mechanics**: SQLite efficiently handles binary data directly within the database file up to ~100 KB.
- **Allows External Storage**:
  - Xcode Core Data Data Model option for `Binary Data` attributes.
  - Enabling **"Allows External Storage"** allows Core Data to automatically decide whether to store data inline inside SQLite or as external files on disk based on binary size.

<img width="689" height="180" alt="image" src="https://github.com/user-attachments/assets/53f953b2-0e61-4a48-aae3-269f9601d8b8" />


---

### 6. Code Placeholders

**Remote Feed Image Data Loader:**
```Swift
import Foundation

public final class RemoteFeedImageDataLoader: FeedImageDataLoader {
    private let client: HTTPClient
    
    public init(client: HTTPClient) {
        self.client = client
    }
    
    public enum Error: Swift.Error {
        case connectivity
        case invalidData
    }
    
    private final class HTTPTaskWrapper: FeedImageDataLoaderTask {
        private var completion: ((FeedImageDataLoader.Result) -> Void)?
        
        var wrapped: HTTPClientTask?
        
        init(_ completion: @escaping (FeedImageDataLoader.Result) -> Void) {
            self.completion = completion
        }
        
        func complete(with result: FeedImageDataLoader.Result) {
            completion?(result)
        }
        
        func cancel() {
            preventFurtherCompletion()
            wrapped?.cancel()
        }
        
        private func preventFurtherCompletion() {
            completion = nil
        }
    }
    
    public func loadImageData(from url: URL, completion: @escaping (FeedImageDataLoader.Result) -> Void) -> FeedImageDataLoaderTask {
        let task = HTTPTaskWrapper(completion)
        task.wrapped = client.get(from: url) { [weak self] result in
            guard self != nil else { return }
            
            task.complete(with: result
                .mapError { _ in Error.connectivity }
                .flatMap { (data, response) in
                    let isInvalidResponse = response.isOK && !data.isEmpty
                    return isInvalidResponse ? .success(data) : .failure(Error.invalidData)
                })
        }
        return task
    }
}

```

**Local Feed Image Data Loader & FeedImageDataStore Protocol:**
```Swift
import Foundation

public protocol FeedImageDataStore {
    typealias RetrievalResult = Swift.Result<Data?, Error>
    typealias InsertionResult = Swift.Result<Void, Error>
    
    func insert(_ data: Data, for url: URL, completion: @escaping (InsertionResult) -> Void)
    func retrieve(dataForURL url: URL, completion: @escaping (RetrievalResult) -> Void)
}

public final class LocalFeedImageDataLoader {
    private let store: FeedImageDataStore
    
    public init(store: FeedImageDataStore) {
        self.store = store
    }
}

extension LocalFeedImageDataLoader {
    public typealias SaveResult = Result<Void, Swift.Error>
    
    public enum SaveError: Swift.Error {
        case failed
    }
    
    public func save(_ data: Data, for url: URL, completion: @escaping (SaveResult) -> Void) {
        store.insert(data, for: url) { [weak self] result in
            guard self != nil else { return }
            
            completion(result.mapError { _ in SaveError.failed })
        }
    }
}

extension LocalFeedImageDataLoader: FeedImageDataLoader {
    public typealias LoadResult = FeedImageDataLoader.Result
    
    public enum LoadError: Swift.Error {
        case failed
        case notFound
    }
    
    private final class LoadImageDataTask: FeedImageDataLoaderTask {
        private var completion: ((FeedImageDataLoader.Result) -> Void)?
        
        init(_ completion: @escaping (FeedImageDataLoader.Result) -> Void) {
            self.completion = completion
        }
        
        func complete(with result: FeedImageDataLoader.Result) {
            completion?(result)
        }
        
        func cancel() {
            preventFurtherCompletions()
        }
        
        private func preventFurtherCompletions() {
            completion = nil
        }
    }
    
    public func loadImageData(from url: URL, completion: @escaping (LoadResult) -> Void) -> FeedImageDataLoaderTask {
        let task = LoadImageDataTask(completion)
        store.retrieve(dataForURL: url) { [weak self] result in
            guard self != nil else { return }
            
            task.complete(with: result
                .mapError { _ in LoadError.failed }
                .flatMap { data in
                    data.map { .success($0) } ?? .failure(LoadError.notFound)
                })
        }
        return task
    }
}
```

**CoreData Feed Store Image Data Extension:**
```Swift
import Foundation

extension CoreDataFeedStore: FeedImageDataStore {
    public func insert(_ data: Data, for url: URL, completion: @escaping (InsertionResult) -> Void) {
        perform { context in
            completion(Result {
                try ManagedFeedImage.first(with: url, in: context)
                    .map { $0.data = data }
                    .map(context.save)
            })
        }
    }
    
    public func retrieve(dataForURL url: URL, completion: @escaping (RetrievalResult) -> Void) {
        perform { context in
            completion(Result {
                try ManagedFeedImage.first(with: url, in: context)?.data
            })
        }
    }
}
```

**CoreData Feed Store & Managed Entities:**
```Swift
import CoreData

@objc(ManagedFeedImage)
class ManagedFeedImage: NSManagedObject {
    @NSManaged var id: UUID
    @NSManaged var imageDescription: String?
    @NSManaged var location: String?
    @NSManaged var url: URL
    @NSManaged var data: Data?
    @NSManaged var cache: ManagedCache
}

extension ManagedFeedImage {
    static func first(with url: URL, in context: NSManagedObjectContext) throws -> ManagedFeedImage? {
        let request = NSFetchRequest<ManagedFeedImage>(entityName: entity().name!)
        request.predicate = NSPredicate(format: "%K = %@", argumentArray: [#keyPath(ManagedFeedImage.url), url])
        request.returnsObjectsAsFaults = false
        request.fetchLimit = 1
        return try context.fetch(request).first
    }
    
    static func images(from localFeed: [LocalFeedImage], in context: NSManagedObjectContext) -> NSOrderedSet {
        return NSOrderedSet(array: localFeed.map({ local in
            let managed = ManagedFeedImage(context: context)
            managed.id = local.id
            managed.imageDescription = local.description
            managed.location = local.location
            managed.url = local.url
            return managed
        }))
    }
    
    var local: LocalFeedImage {
        return LocalFeedImage(id: id, description: imageDescription, location: location, url: url)
    }
}
```

---

### Senior & Architect Interview Perspective

**Q: Why create two separate protocols (`FeedStore` and `FeedImageDataStore`) if `CoreDataFeedStore` implements both anyway?**

> "This enforces the Interface Segregation Principle (ISP). Clients should not depend on methods they don't use. `LocalFeedLoader` only manages feed metadata and shouldn't know about binary image data blobs. By segregating the store interfaces, we keep the consumers tightly scoped, easier to mock in unit tests, and capable of using completely different storage backends (e.g., `FeedStore` backed by CoreData, while `FeedImageDataStore` is backed by a simple Disk/Keychain store)."

**Q: What are the architectural tradeoffs of sharing domain models vs. creating separate model representations per layer?**

> "Sharing domain models across all layers causes high coupling and cascading changes across the codebase. A schema update in database entities would force recompilation of domain logic and API mapping layers. Segregating models per layer respects the Single Responsibility Principle, isolates infrastructure details, and allows independent module compilation, deployment, and testing."

---

### Phase 3: Proof of Learning (After the Session)
*This is where the knowledge becomes permanent. This section is completed from memory.*

#### My "Blank Page" Summary (Feynman Technique)
*(Write your summary of the topic here without looking at your notes from Phase 2. Draw a diagram if it helps.)*

#### Consolidation Checklist
- [ ] **Teach-Back Complete:** I have stood up and explained this concept out loud for 60 seconds.
- [ ] **Next-Day Review Scheduled:** I have added a 5-minute task to my plan for tomorrow to review this note.
- [ ] **Weekly Review Scheduled:** I have added this note to my list for the upcoming "Weekly Knowledge Review" session.

---
