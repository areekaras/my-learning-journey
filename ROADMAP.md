# 3-Month Learning Roadmap (Sep 10 - Dec 10, 2025)

> This document contains the detailed, week-by-week syllabus and checklists for my focused 3-month learning journey.

---

### Pillar 1: Deepen the Craft (The Skill Pillar)

#### **Objective 1.1: Deepen Mobile Engineering Craft (Essential Developer)**
- [x] **Analyze Essential Developer iOS Lead Program**
  - [x] Review the remaining modules and projects.
  - [x] Break down the course into a checklist of videos and assignments.
- [ ] **Essential Dev Course Checklist**
  - [x] Analyse github with course material and understand previous learning stage
  - [x] **Planning, Initial System design**
    - [x] Singleton
    - [x] Intro to dependency diagram and composition
    - [x] Into to modular design
    - [x] BDD, TDD, Use cases, Architectures and Modular design
    - [x] Effective developing Swift apps before backend/design ready
  - [x] **Networking**
    - [x] Set up start up project and automated testing set up guide
    - [x] From Singleton and Global to proper Dependency injection
    - [x] Asserting a captured value is not enought + cross module access control
    - [x] Handling errors + Stubbing vs Spying + Handling invalid paths
    - [x] A classicist TDD approach (No mocking) to mapping JSON with Decodable protocols + Domain Specific Models
    - [x] Automating memory leak detection + Preventing a (Way too common) async bug
    - [x] Conforming to Feature Abstractions while maintaining System Modularity + Keeping Test details out of production with enum pattern matching and async expectations
    - [x] Four approaches to Test(-Drive) Network requests; End-to-end, Subclassing and Protocol-based mocking, and URL Protocol stubbing
    - [x] Speeding up Development by Reducing debugging time, Minimising risks, Maximising learning and Decoupling tests from implementation details
    - [x] Randomising & Parallelising Tests, Integration tests with the backend and Automating a continuous integration(CI) pipeline
    - [x] Identifying, Debugging and Solving Data Races with Thread the sanitizer
    - [x] How many Apps get wrong about Reachability and How to get it right
    - [x] The Feed API Challenge
  - [x] **Networking Recap**
    - [x] Overview
    - [x] Key stats and insights to Maximise you Development output
  - [x] **Persistance**
    - [x] URLCache as a persistence alternative and Solving the infamous "But it woks on my machine" caching problem
    - [x] Clarifying requirements, Enhancing cross-team domain knowledge and Iterative vs Big upfront design
    - [x] Decoupling the Use-case business logic from frameworks + Controlling time + Multi method abstractions by following Interface segregation and Single responsibility principles
    - [x] Proper memory management of Captured references in Deeply nested modules + Identifying Highly coupled modules
    - [x] Visualising and solving high coupling issues by Decentralising components using Data Transfer Model Representations
    - [x] Performing Calendrical calculations correctly, Dealing with Coincidental duplication while respecting DRY principles, Decoupling the tests from implementation with tiny DSLs, and test triangulation to increase code coverage and confidence.
    - [x] Test-driving the cache invalidation + Identifying the complex (Bloated) functionality with Command Query Separation Principles
    - [x] Separating Queries and side-effects for Simplicity and Reusability, Choosing between Enum switching strategies, and Differentiating App-specific from App-agnostic logic  
    - [x] Separating App-Specific and App-agnostic and Framework logic, Entities vs Values Objects, Establishing single source of truth, and Designing side-effects free (Deterministic) Domain models with Functional Core, Imperative Shell Principles
    - [x] Dependency Inversion Anatomy (High level | Boundary | Low level), Defining inbox checklist and Contract Specs to Improve Collaboration and avoid Side effect bugs in Multithreaded Environments
    - [x] Persisitng/Retrieving Models with Codable + FileSystem, Test-driving in Integration with Real Frameworks Instead of Mocks & Measuring test times overhead with `xcodebuild`
    - [x] Deleting Models and Handling Errors with Codable+FileSystem, Making Async code look Sync in Tests to Eliminate Arrow Anti-pattern, and More Essential Test Guidelines to Improve Code Quality and Team communication
    - [x] Designing and Testing Thread-safe Components with DispatchQueue, Serial vs. Concurrent Queues, Thread-safe Value Types, and Avoiding Race Conditions
    - [x] Protocol vs Class Inheritance, Composite Reuse Principle, and Extracting Reusable Test Specs with Protocol Inheritance, Extensions and Composition
    - [x] Core Data Overview, Implementation, Concurrency Model, Trade-offs, Modeling & Testing Techniques, and Implementing Reusable Protocol Specs
    - [x] Finishing the Cache Implementation with Business Logic + Core Data Integration Tests—Unit vs. Integration: Pros/Cons, Performance, Complexity & How to Achieve The ideal Testing Pyramid Distribution
    - [x] The Feed Store Challenge
  - [x] **Persistance Recap**
    - [x] Overview
    - [x] Key Stats and Insight to Maximise Your Development Output
  - [x] **Bonus**
    - [x] Improving Model Composability With Swift's Standard Result and Optional Types, Map, Functors, and Powerful Refactorings Backed by Tests and Types (Compiler!)
  - [ ] **UI + Presentation Module**
    - [x] Gathering Fast Feedback and Validating UI Design and Dev Decisions Through Realistic App Prototypes
    - [x] Supporting Multiple Platforms with Swift Frameworks While Separating Platform-specific Components to Facilitate Loose Coupling and Speed up Development
    - [x] Apple MVC, Test-driving UIViewControllers, Dealing with UIKit's Inversion of Control & Temporal Coupling, and Decoupling Tests from UI Implementation Details
    - [x] Effectively Test-driving MVC UI with Multiple Views/Models/State, Efficiently Loading/Prefetching/Cancelling Image Requests, Inside-Out vs. Outside-In Development, and Identifying the Massive View Controller Anti-pattern by Following Design Principles
    - [x] Refactoring Massive View Controllers Into Multiple Tiny MVCs + Using Composers & Adapters to Reduce Coupling and Eliminate Redundant Dependencies
    - [x] MVVM: Reducing Boilerplate, Shifting Reusable Presentation Logic from Controllers into Cross-Platform (Stateful & Stateless) ViewModels, and Decoupling Presentation from UI Frameworks with Swift Generics
    - [ ] MVP: Creating a Reusable and Cross-Platform Presentation Layer, Implementing Service Adapters, and Solving Cyclic Dependencies & Memory Management issues with the Proxy Pattern
    - [ ] Storyboard vs. Code: Layout, DI and Composition, Identifying the Constrained Construction DI Anti-pattern, and Optimizing Performance by Reusing Cells
    - [ ] Creating, Localizing, and Testing Customer Facing Strings in the Presentation Layer + NSLocalizedString Best Practices
    - [ ] Decorator Pattern: Decoupling UIKit Components From Threading Details, Removing Duplication, and Implementing Cross-Cutting Concerns In a Clean & SOLID Way
    - [ ] The UI Design Patterns Challenge
    - [ ] Test-driven Approach to Effectively Dealing with Legacy Code (Code With No Tests!) + Extracting Cross-platform Components From a Platform-specific Module
  - [ ] **UI + Presentation Module Recap**
    - [ ] Overview
    - [ ] Key Stats and Insight to Maximise Your Development Output
  - [ ] **Main + Composition Module**
    - [ ] Feed Image Data Loading and Caching with URLSession/CoreData + Composing Modules Into a Running iOS Application with Xcode Workspaces
    - [ ] Composite Pattern: Implementing a Flexible & Composable Strategy for Loading Data with Fallback Logic
    - [ ] Interception: An Effective, Modular and Composable Way of Injecting Behavior and Side-effects in the App Composition
    - [ ] Validating Acceptance Criteria with High-Level UI Tests, Controlling Network and App State in UI tests with Launch Arguments and Conditional Compilation Directives
    - [ ] Validating Acceptance Criteria with Fast Integration Tests, Composition Root, and Simulating App Launch & State Transitions
    - [ ] Validating the UI with Snapshot Tests + Dark Mode Support
    - [ ] Preventing a Crash when Invalidating Work Based on UITableViewDelegate events
    - [ ] Organizing Modular Codebases with Horizontal and Vertical Slicing
    - [ ] Continuous Delivery and Deployment: Automating the App Deploy to App Store Connect
    - [ ] From Design Patterns to Universal Abstractions Using the Combine Framework
  - [ ] **Main + Composition Module Recap**
    - [ ] Overview
    - [ ] Key Stats and Insight to Maximise Your Development Output
  - [ ] [Bonus] Navigation + Advanced Patterns Module
    - [ ] #001 - [Image Comments API] From Dependency Injection to Dependency Rejection (2 hr 20 min)
    - [ ] #002 - [Image Comments Presentation] Reusable Presentation Logic (2 hr 32 min)
    - [ ] #003 - [Image Comments UI] Reusable UI Components, Diffable Data Sources, Dynamic Type, Snapshot Testing (2 hr 45 min)
    - [ ] #004 - [Image Comments Composition] Navigation and Feature Composition (2 hr 5 min)
    - [ ] 🥇 The Image Comments Challenge
    - [ ] #005 - Keyset Pagination with Caching Strategy (2 hr 49 min)
    - [ ] #006 - Logging, Profiling, and Optimizing Infrastructure Services (1 hr 42 min)
    - [ ] #007 - Async Injection: Decoupling the Domain from Infra Details (2 hr 7 min)
  - [ ] [Bonus] Migrating to Swift Concurrency
    - [ ] #001 - Complete Swift Concurrency Checking (56 min)
  - [ ] Certification
    - [ ] Certificate of Completion and Blue belt 🔵
    - [ ] Blue to Purple Belt Journey

#### **Objective 1.2: Master Data Structures & Algorithms (AlgoExpert)**
- [x] **Analyze AlgoExpert Course Structure**
  - [x] Review the learning path and list of topics.
  - [x] Break down the course into logical sections.
- [ ] **Course Checklist** (Finish random 10 problems -> Crash course -> Make plan again)
  - [ ] **Random 10 Problems**
    - [x] 1. https://www.algoexpert.io/questions/longest-increasing-subsequence
    - [x] 2. https://www.algoexpert.io/questions/kruskals-algorithm
    - [x] 3. https://www.algoexpert.io/questions/binary-tree-diameter
    - [x] 4. https://www.algoexpert.io/questions/minimum-area-rectangle
    - [x] 5. https://www.algoexpert.io/questions/radix-sort
    - [ ] 6. https://www.algoexpert.io/questions/invert-binary-tree
    - [ ] 7. https://www.algoexpert.io/questions/sweet-and-savory
    - [ ] 8. https://www.algoexpert.io/questions/union-find
    - [ ] 9. https://www.algoexpert.io/questions/pattern-matcher
    - [ ] 10. https://www.algoexpert.io/questions/longest-common-subsequence
  - [ ] **Crash course**
    - [ ] 1 - Introduction
    - [ ] 2 - What Are Data Structures?
    - [ ] 3 - Complexity Analysis
    - [ ] 4 - Memory
    - [ ] 5 - Big O Notation
    - [ ] 6 - Logarithm
    - [ ] 7 - Arrays
    - [ ] 8 - Linked Lists
    - [ ] 9 - Hash Tables
    - [ ] 10 - Stacks And Queues
    - [ ] 11 - Strings
    - [ ] 12 - Graphs
    - [ ] 13 - Trees
  - [ ] **Make plan after finishing above**
    - [ ] Check on Coding Interview Assessments
    - [ ] Check on Mock coding interviews
    - [ ] Check on Coding interview Tips

#### **Objective 1.3: Master System Design**
- [x] **Analyze System Design Learning Path**
  - [x] Select a primary resource ("Grokking the System Design Interview").
  - [x] List the key topics to master.
- [ ] **Course Checklist** (Finish AlgoExpert Fundamentals -> Make plan again)
  - [ ] **Systems Design Fundamentals**
    - [ ] 1 - Introduction
    - [ ] 2 - What Are Design Fundamentals?
    - [ ] 3 - Client—Server Model
    - [ ] 4 - Network Protocols
    - [ ] 5 - Storage
    - [ ] 6 - Latency And Throughput
    - [ ] 7 - Availability
    - [ ] 8 - Caching
    - [ ] 9 - Proxies
    - [ ] 10 - Load Balancers
    - [ ] 11 - Hashing
    - [ ] 12 - Relational Databases
    - [ ] 13 - Key-Value Stores
    - [ ] 14 - Specialized Storage Paradigms
    - [ ] 15 - Replication And Sharding
    - [ ] 16 - Leader Election
    - [ ] 17 - Peer-To-Peer Networks
    - [ ] 18 - Polling And Streaming
    - [ ] 19 - Configuration
    - [ ] 20 - Rate Limiting
    - [ ] 21 - Logging And Monitoring
    - [ ] 22 - Publish/Subscribe Pattern
    - [ ] 23 - MapReduce
    - [ ] 24 - Security And HTTPS
    - [ ] 25 - API Design
  - [ ] **Make plan after finishing above**
    - [ ] Check on other sections once finish the above and make plan again

---

### Pillar 2: Master Communication

#### **Objective 2.1: Launch YouTube Channel & Improve Speaking**
- [x] **Analyze "The Art of Speaking" Course**
  - [x] Review the course structure and exercises.
  - [x] Create a checklist of modules to complete.
- [ ] **Course Checklist**
  - [x] Introduction
  - [x] Module 1. Speak with Impact: Master non verbal power
  - [x] Module 2. Keep them hooked: Build irresistible speeches
  - [ ] Module 3. Win any audience: Command attention
  - [ ] Module 4. Unlock Charisma and Magnetism
  - [ ] Module 5. Think fast: Improvisation techniques
  - [ ] Module 6. The art of persuasion
  - [ ] Module 7. Be unforgettable: Self presentation and personal brand
  - [ ] Module 8. Own the screen: Speak with power online

---

### Pillar 3: Master the Market

#### **Objective 3.1: Actively Apply & Interview (Starting November 2025)**
- [ ] **Apply to 20 high-quality, targeted roles (iOS & Hybrid).**
- [ ] **Complete at least 5 full interview loops.**
