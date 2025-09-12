# Learning Session: Are Singletons and Global instances Damaging your system design and Testability

---

### **Metadata**
- **Date:** `13-09-2025`
- **Source:** https://academy.essentialdeveloper.com/ios-lead-essentials/447455/resources/8528509
- **Status:** `In Progress`

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is to understand Singleton and Global instances.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** No previous topics, i am starting today

---

### Phase 2: Active Notes (During the Session)
*My structured notes, guided by deep questions.*

- [x] **Embrace the Struggle:** Attempt problems on your own for 15-20 mins before seeking hints.

### Notes

We are covering two topics in this
1. How to test networking layer?
2. Should we use Singleton

> Singleton Pattern is a way to make sure that a class have single instance and it provides a single point of access to it

```
//Singleton
class ApiClient {
  static let instance = ApiClient()

  private init() {}
}

//singleton
URLSession.shared
URLSession()
```


#### 1. What specific problem does this concept solve?
This concepts covers below topics 
- Singleton
- singleton
- Global mutable state
- Testing via subclassing and property injection
- Extension to separate code for its own space
- Dependency inversion with protocol
- Add adapters for more flexibility

#### 2. How does this connect to what I already know?
- The similar topics I have used and using at my workplace

#### 3. What are the key trade-offs or edge cases?
To work in a team and working without concepts will be harder, but this is needed when working in team

#### 4. How would I explain this to a beginner? (The Simple Analogy)
Dependency inversion topic is all about plug and use, we will have interface, any type of items can be used by just plugging into it with proper socket. only proper socket which confirm to the socket type required

---

### Phase 3: Proof of Learning (After the Session)
*This is where the knowledge becomes permanent. This section is completed from memory.*

#### My "Blank Page" Summary (Feynman Technique)
This topic is about what is Singleton, singlton , Global state along with how to test networking layer and what are the problems using singleton, How to use singleton properly . How to achieve dependency inversion so low level framework will depends on high level code

#### Consolidation Checklist
- [x] **Teach-Back Complete:** I have stood up and explained this concept out loud for 60 seconds.
- [x] **Next-Day Review Scheduled:** I have added a 5-minute task to my plan for tomorrow to review this note.
- [x] **Weekly Review Scheduled:** I have added this note to my list for the upcoming "Weekly Knowledge Review" session.

