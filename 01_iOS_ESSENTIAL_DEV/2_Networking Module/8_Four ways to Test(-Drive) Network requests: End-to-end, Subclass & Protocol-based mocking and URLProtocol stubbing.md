# Learning Session: Four Ways to Test(-Drive) Network Requests: End-to-end, Subclass & Protocol-based mocking, and URLProtocol stubbing

---

### **Metadata**
- **Date:** `06-10-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/8952465

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is clear.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)
*My structured notes, guided by deep questions.*

#### Notes

# Testing Strategies for Network Requests

## 1. End-to-End Testing

-   **What it is**: Checks the full client-server integration by making **real HTTP requests**.
-   **Pros**:
    -   Guarantees correct communication with a real backend.
-   **Cons**:
    -   Requires a backend to be available.
    -   Needs a network connection.
    -   **Flaky**: Tests can fail due to network issues.
    -   **Slow**: Dependent on network performance.
-   **Verdict**: Better for later-stage integration testing, not for isolated component testing.


## 2. Mocking with Subclasses

-   **What it is**: Subclassing `URLSession` (a type you don't own) and stubbing its methods to avoid real network calls.
-   **Pros**:
    -   Faster and more reliable than end-to-end tests.
-   **Cons**:
    -   **Dangerous**: Subclassing types you don't own can lead to unexpected behavior or crashes (e.g., forgetting to override `resume()`).
    -   **Wrongful Assumptions**: You might incorrectly assume the behavior of the original class.
    -   **Tightly Coupled**: Tests are tied to the implementation (e.g., asserting `dataTask(with:)` and `resume()` were called), making refactoring difficult.


## 3. Mocking with Protocols

-   **What it is**: Creating a protocol that mimics the `URLSession` interface (e.g., `<HTTPSession>`) and using a mock object conforming to it in tests.
-   **Pros**:
    -   Hides unnecessary details of `URLSession` APIs.
    -   Avoids the risks of subclassing a class you don't own.
-   **Cons**:
    -   **Still Tightly Coupled**: The protocol often just mimics the `URLSession` method signatures, so you're still coupled to its design.
    -   **Production Code Noise**: Adds protocols to the production codebase that exist only for testing purposes.


## 4. `URLProtocol` Stubbing (Preferred Approach)

-   **What it is**: Using Apple's built-in **URL Loading System** to intercept outgoing network requests and provide stubbed (fake) responses.
-   **Why it's preferred**:
    -   ✅ **Fast and Reliable**: No real network dependency.
    -   ✅ **Apple Recommended**: A documented, official API for testing network requests, avoiding unexpected mocking behavior.
    -   ✅ **Decoupled**: Tests are not tied to the `URLSession` implementation. You can refactor the network client, and as long as it makes a request, the tests will pass.
    -   ✅ **Clean**: Keeps test-specific code out of the production target.


## Key Concepts

### URL Loading System

-   Apple's framework for interacting with URLs and servers.
-   Provides access to resources using standard protocols (e.g., `https` or custom ones).
-   Loading is performed **asynchronously**.

### `URLProtocol`

-   An abstract class that lets you intercept and handle the loading of URL data.
-   Allows you to define custom loading behavior for any request.
-   **Activation**: `URLProtocol.registerClass(...)`
-   **Deactivation**: `URLProtocol.unregisterClass(...)`
-   **Required `subclass` methods**:
    -   `canInit(with:URLRequest)`
    -   `canonicalRequest(for:URLRequest)`
    -   `startLoading()`
    -   `stopLoading()`


## Media & References

-   **URL Loading System**:
    -   https://github.com/user-attachments/assets/3d356b3e-8e0e-46fe-9215-7828d8fa85a7
-   **Usage in testing**:
    -   <img width="853" height="637" alt="image" src="https://github.com/user-attachments/assets/d0a39c91-71a0-45e2-bce3-75f8ef151341" />
-   **References**:
    -   https://www.loading-systems.com/en(https://developer.apple.com/documentation/foundation/url_loading_system)
    -   [`URLProtocol`](https://developer.apple.com/documentation/foundation/urlprotocol)
    -   [WWDC 2018: Testing Tips & Tricks](https://developer.apple.com/videos/play/wwdc2018/417/)


---

### Phase 3: Proof of Learning (After the Session)
*This is where the knowledge becomes permanent. This section is completed from memory.*

#### My "Blank Page" Summary (Feynman Technique)
*(Write your summary of the topic here without looking at your notes from Phase 2. Draw a diagram if it helps.)*

#### Consolidation Checklist
- [ ] **Teach-Back Complete:** I have stood up and explained this concept out loud for 60 seconds.
- [ ] **Next-Day Review Scheduled:** I have added a 5-minute task to my plan for tomorrow to review this note.
- [ ] **Weekly Review Scheduled:** I have added this note to my list for the upcoming "Weekly Knowledge Review" session.
