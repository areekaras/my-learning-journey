# Learning Session: Creating, Localizing, and Testing Customer Facing Strings in the Presentation Layer + NSLocalizedString Best Practices 

---

### **Metadata**
- **Date:** `02-12-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/11463761

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is clear.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)


#### Notes

PR: https://github.com/areekaras/EssentialFeed/pull/18

---

# Notes: Creating, Localizing, and Testing Presentation Strings

This lecture focuses on managing user-facing strings (localization) within a clean architecture. We treat localization as a **Presentation** concern, separating it from business logic, and implement automated tests to guarantee translation integrity.

-   <img width="813" height="116" alt="image" src="https://github.com/user-attachments/assets/44ceae8c-e8cc-4b53-b33b-5480c7c9aae0" />

---

### 1. Localization as a Presentation Concern

-   **The Rule**: Business Logic (Models) should never know about localization.
-   **Why?**: The same business logic (e.g., "loading failed") might be presented differently depending on the context (alert, toast, log) or language. Coupling logic to strings hurts reusability.
-   **Where to put Strings**:
    -   **MVC**: In the Controller or View.
    -   **MVVM**: In the ViewModel.
    -   **MVP**: In the Presenter.
-   **Implementation**: We create a localized string accessor (e.g., `FeedPresenter.title`) in the Presentation layer. This keeps our core logic pure and allows us to reuse the localized strings across platforms (iOS, macOS, etc.) if we put the Presentation layer in a cross-platform module.

---

### 2. Best Practices for Naming Keys

-   **Don't reuse keys**: Even if two strings are "Cancel" in English, they might be different words in another language depending on context (e.g., "Cancel Transaction" vs. "Cancel Upload").
-   **Use Contextual Keys**: Use specific keys like `CANCEL_TRANSACTION_ACTION` instead of just `CANCEL`. This gives translators context and flexibility.
-   **Never use the key as the default value**: Always rely on a separate strings file. Using the key as the default value (e.g., `NSLocalizedString("Cancel", comment: "")`) hides missing translations and leads to bad UX.

---

### 3. Testing Localized Strings

We implemented two types of tests for localization:

1.  **Integration Tests (UI)**:
    -   We verify that the UI displays the *correct string* for the current locale.
    -   **Crucial**: We do **not** assert the literal string "My Feed". We assert `sut.title == localized("FEED_VIEW_TITLE")`.
    -   **Why?**: Literal strings make tests fragile. If a translator changes "My Feed" to "Your Feed," the test breaks. By asserting against the *lookup*, the test passes as long as the key is correct, regardless of the value.

2.  **Iterative Architecture Tests (Safety Net)**:
    -   We wrote a powerful test `FeedLocalizationTests` that automatically iterates through *all* localization bundles (`en`, `pt-BR`, etc.) and *all* keys.
    -   It asserts that **every key has a value** in every supported language.
    -   **The Benefit**: This catches missing translations automatically. If you add a key to English but forget Portuguese, this test fails instantly. It's a massive safety net for quality assurance.

---

## Code snippets

### 1. `FeedPresenter` with Localized Title
The Presenter is responsible for defining the localized title key and bundle lookup.

```Swift
final class FeedPresenter {
    // ...
    static var title: String {
        return NSLocalizedString("FEED_VIEW_TITLE",
            tableName: "Feed",
            bundle: Bundle(for: FeedPresenter.self),
            comment: "Title for the feed view")
    }
}
```

### 2. UI Integration Test (Comparing Key vs. Key)
This test helper ensures we compare the looked-up value, not a hardcoded string.

```Swift
extension FeedUIIntegrationTests {
    func localized(_ key: String, file: StaticString = #filePath, line: UInt = #line) -> String {
        let table = "Feed"
        let bundle = Bundle(for: FeedViewController.self)
        let value = bundle.localizedString(forKey: key, value: nil, table: table)
        
        // Safety check: if lookup fails, it returns the key. We fail the test if that happens.
        if key == value {
            XCTFail("Missing localized string for key: \(key)", file: file, line: line)
        }
        return value
    }
    
    func test_feedView_hasTitle() {
        let (sut, _) = makeSUT()
        sut.simulateAppearence()
        // Assertion is robust against copy changes
        XCTAssertEqual(sut.title, localized("FEED_VIEW_TITLE"))
    }
}
```

### 3. Automated Localization Safety Check
This powerful test iterates all bundles and keys to ensure 100% translation coverage.

```Swift
final class FeedLocalizationTests: XCTestCase {

    func test_localizedStrings_haveKeysAndValuesForAllSupportedLocalizations() {
        let table = "Feed"
        let presentationBundle = Bundle(for: FeedPresenter.self)
        let localizationBundles = allLocalizationBundles(in: presentationBundle)
        let localizedStringKeys = allLocalizedStringKeys(in: localizationBundles, table: table)

        localizationBundles.forEach { (bundle, localization) in
            localizedStringKeys.forEach { key in
                let localizedString = bundle.localizedString(forKey: key, value: nil, table: table)

                if localizedString == key {
                    let language = Locale.current.localizedString(forLanguageCode: localization) ?? ""
                    XCTFail("Missing \(language) (\(localization)) localized string for key: '\(key)'")
                }
            }
        }
    }
    
    // ... helpers to find bundles and keys ...
}
```


---

### Senior & Architect Interview Perspective

**Q: "Where should localization logic live in your architecture?"**

> "Localization is a **Presentation concern**. It should never live in the Business Logic (Domain) layer.
>
> In an MVP or MVVM architecture, the **Presenter** or **ViewModel** is responsible for retrieving the localized string using the correct key and passing it to the View. This keeps the View 'dumb' and the Model pure. It also allows us to share the Presentation layer (and its localized strings) across different platforms (iOS, macOS, watchOS)."

**Q: "How do you test localized strings without making your tests fragile?"**

> "I avoid testing literal string values (e.g., `XCTAssertEqual(label.text, "Hello")`). This breaks whenever copy changes.
>
> Instead, I test the **integration**: I assert that `label.text` matches the result of looking up the expected key in the resource bundle.
>
> Furthermore, I implement an **architectural test** that iterates through every supported localization bundle (`.lproj`) and verifies that every key defined in the main language exists in all other languages. This guarantees we never ship with missing translations."

**Q: "Why shouldn't I just reuse the string 'Cancel' everywhere in my app?"**

> "Because context matters in translation. 'Cancel' might be a verb in one context ('Cancel the request') and a noun in another ('Cancel button'). In English they are the same, but in languages like German or Portuguese, they might be completely different words.
>
> Using specific keys like `FEED_VIEW_TITLE` or `ERROR_VIEW_DISMISS_BUTTON` gives translators the context they need to choose the correct word, preventing embarrassing UI mistakes in other languages."

---
