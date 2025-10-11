# Learning Session: Union Find

---

### **Metadata**
- **Date:** `11-10-2025`
- **Source:** https://www.algoexpert.io/questions/union-find

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is clear.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)
*My structured notes, guided by deep questions.*

- [x] **Embrace the Struggle:** Attempt problems on your own for 15-20 mins before seeking hints.


#### Notes

---

# Problem: Union-Find (Disjoint Set)

**Goal:** Design a data structure that keeps track of a collection of **disjoint sets**. "Disjoint" means that no element can be in more than one set at the same time. This structure is highly optimized for two main operations: quickly finding which set an element belongs to and merging two sets.

It must support three core methods:
1.  `createSet(value)`: Creates a new set containing only the given `value`.
2.  `find(value)`: Returns the **representative** (or "root") of the set that `value` belongs to. Every set has one unique representative. If you `find` any two elements from the same set, the result must be the same.
3.  `union(valueOne, valueTwo)`: Merges the two sets containing `valueOne` and `valueTwo` into a single set.


### Conceptual Explanation (For Beginners)

Imagine you have a group of people, and you want to keep track of their friend circles.

* Initially, everyone is a stranger, so each person is in their own "circle" of one. This is what `createSet` does.
* `find(person)` is like asking, "Who is the leader of this person's friend circle?" Every circle has a designated leader (the **representative**).
* `union(personA, personB)` is like saying, "Person A's entire friend circle now merges with Person B's entire friend circle." They all become one big group with a single leader.


Under the hood, we represent these sets as a **forest** (a collection of trees). Each tree represents one set. The **root** of the tree is the **representative** of that set.

To implement this, we typically use a hash map (or dictionary) called `parents`. `parents[child] = parent` tells us which node is directly above a child node in the tree. If a node is the root of its tree, it will be its own parent, so `parents[node] = node`.


#### Example
Let's trace a series of operations to see how the sets evolve.

1.  Start by creating sets for numbers 1 through 10.
2.  Perform `union(1, 2)`
3.  Perform `union(3, 4)`
4.  Perform `union(1, 3)`
5.  At this point, `find(4)` should return the same root as `find(2)`. For example, if `1` is the root, both would return `1`.
6.  Perform `union(5, 6)`
7.  Perform `union(6, 7)`
8.  Perform `union(8, 9)`
9.  Now, `find(7)` should return the same root as `find(5)`.
10. `find(1)` and `find(8)` should return different roots, as their sets have not been merged.


## How to Whiteboard the Union-Find Solution

Explaining Union-Find on a whiteboard is highly effective because it's a very visual data structure. Here's a script you can follow.

**(Start by drawing two boxes on the board labeled `parents` and `ranks`.)**

"To solve this, I'll use two hash maps, `parents` and `ranks`. I'll represent the sets as trees, where each node is an element and the root of the tree is the 'representative' of the set."

**1. Explaining `createSet(value)`**
"When we call `createSet(5)`, we are creating a new set with just one member. In our tree representation, this is a root node that points to itself."

**(Draw a circle with '5' in it, and an arrow pointing from the circle back to itself. In the `parents` box, write `5: 5`. In the `ranks` box, write `5: 0`.)**

**2. Explaining `union(valueOne, valueTwo)` with Union by Rank**
"Now, let's say we call `union(5, 6)` after creating a set for `6`. First, we find the root of each. The root of 5 is 5, and the root of 6 is 6. Next, we check their ranks. Both have a rank of 0. When ranks are equal, we can choose one to be the new root. Let's make 6 the root of 5. The key step is that we must **increment the rank of the new root**."

**(Erase the self-pointing arrow from '5'. Draw a new arrow from '5' to '6'. In the `parents` box, update to `5: 6`. In the `ranks` box, update to `6: 1`.)**

"Let's do another one: `union(7, 6)`. The root of 7 is 7 (rank 0). The root of 6 is 6 (rank 1). Since the rank of 7's tree is less than the rank of 6's tree, we **always attach the shorter tree to the taller one**. So, 7's root will point to 6's root."

**(Draw a node '7' pointing to itself. Then erase that arrow and draw a new one from '7' to '6'. The ranks do not change in this case.)**

**3. Explaining `find(value)` with Path Compression**
"This is the most important optimization. Let's say we call `find(7)`. To find the root, we start at 7 and follow the parent pointers. The path is `7 -> 6`. The root is `6`.

**Path compression** means that after finding the root, we go back along that path and make every node we visited point **directly to the root**."

**(While talking, erase the arrow from `7 -> 6`. Draw a new, direct arrow from `7 -> 6`. In the `parents` box, if you were tracking a deeper tree, you would update all intermediate nodes. For example if the path was `7 -> 8 -> 6`, you would redraw `7 -> 6` and `8 -> 6`.)**

"By doing this, the next time we call `find(7)`, the answer is found in a single step. This dramatically flattens the tree and makes future lookups almost instantaneous."



## Step-by-Step Example Trace (with Optimizations)

Let's trace the example from the course using the fully optimized solution (Union by Rank and Path Compression).

**Initial State:**
`parents = {}`
`ranks = {}`

**1. Operations: `createSet(1)` through `createSet(10)`**
* **Explanation:** Each number is added as its own parent with a rank of 0.
* **`parents` state:** `{1:1, 2:2, 3:3, 4:4, 5:5, 6:6, 7:7, 8:8, 9:9, 10:10}`
* **`ranks` state:** `{1:0, 2:0, 3:0, 4:0, 5:0, 6:0, 7:0, 8:0, 9:0, 10:0}`

**2. Operation: `union(1, 2)`**
* **Explanation:** `root(1)` is 1 (rank 0). `root(2)` is 2 (rank 0). Ranks are equal. Make 2 the parent of 1 and increment rank of 2.
* **`parents` state:** `{1:2, 2:2, ...}`
* **`ranks` state:** `{..., 2:1, ...}`

**3. Operation: `union(3, 4)`**
* **Explanation:** Ranks are equal. Make 4 the parent of 3 and increment rank of 4.
* **`parents` state:** `{..., 3:4, 4:4, ...}`
* **`ranks` state:** `{..., 4:1, ...}`

**4. Operation: `union(1, 3)`**
* **Explanation:** Find `root(1)` which is 2 (rank 1). Find `root(3)` which is 4 (rank 1). Ranks are equal. Make 4 the parent of 2 and increment rank of 4.
* **`parents` state:** `{..., 2:4, 3:4, 4:4, ...}`
* **`ranks` state:** `{..., 4:2, ...}`

**5. Operation: `find(4)`**
* **Explanation:** `root(4)` is 4. Returns `4`. No path compression needed.

**6. Operation: `find(2)`**
* **Explanation:** Path is `2 -> 4`. Root is 4. Path compression is applied, but `2` already points to `4`, so no change. Returns `4`.
* **(Both `find(4)` and `find(2)` correctly return the same root: `4`.)**

**7. Operation: `union(5, 6)`**
* **Explanation:** Ranks are equal. Make 6 the parent of 5 and increment rank of 6.
* **`parents` state:** `{..., 5:6, 6:6, ...}`
* **`ranks` state:** `{..., 6:1, ...}`

**8. Operation: `union(6, 7)`**
* **Explanation:** `root(6)` is 6 (rank 1). `root(7)` is 7 (rank 0). Rank of 7 is smaller. Make 6 the parent of 7. Ranks do not change.
* **`parents` state:** `{..., 5:6, 6:6, 7:6, ...}`
* **`ranks` state:** No change.

**9. Operation: `find(5)`**
* **Explanation:** Path is `5 -> 6`. Root is 6. Returns `6`.

**10. Operation: `find(7)`**
* **Explanation:** Path is `7 -> 6`. Root is 6. Returns `6`.
* **(Both `find(5)` and `find(7)` correctly return the same root: `6`.)**

**11. Final `find(1)` vs `find(8)`**
* **`find(1)`:** Path is `1 -> 2 -> 4`. Root is 4. **Path compression occurs!** After this call, `parents` will be updated: `parents[1] = 4`, `parents[2] = 4`. Returns `4`.
* **`find(8)`:** Path is `8 -> 8`. Root is 8. Returns `8`.
* **(The roots `4` and `8` are different, which is the correct result.)**


## Interview Guide: The Naive Solution

**(Interviewer: "How would you implement a Union-Find data structure?")**

"I'll start with a straightforward implementation. The core of this data structure will be a dictionary called `parents` to store the parent of each element.

1.  **`createSet(value)`**: This is simple. When we create a new set, the element is its own parent. So I'll just set `parents[value] = value`.

2.  **`find(value)`**: To find the representative of a set, I'll start at the given `value` and traverse up the tree by repeatedly looking up its parent until I find an element that is its own parent. That element is the root, or representative, of the set.

3.  **`union(valueOne, valueTwo)`**: To union two sets, I first need to find the representative of each set by calling the `find` method on both values. If they have different representatives, it means they are in different sets. I can then merge them by making one root the parent of the other. For instance, I'll set the parent of `valueOne`'s root to be `valueTwo`'s root."

```Swift
// Solution 1: Naive Implementation
// Time Complexity: find -> O(n), union -> O(n)
// Space Complexity: O(n)
class UnionFind {
    var parents: [Int: Int] = [:]

    // O(1) time | O(1) space
    func createSet(value: Int) {
        parents[value] = value
    }

    // O(n) time | O(1) space
    // n is the number of elements in the set.
    // In the worst case, we might have a linked-list-like tree.
    func find(_ value: Int) -> Int? {
        guard parents[value] != nil else {
            return nil
        }
        
        var currentParent = value
        while currentParent != parents[currentParent]! {
            currentParent = parents[currentParent]!
        }
        return currentParent
    }

    // O(n) time | O(1) space
    // Dominated by the two calls to find().
    func union(valueOne: Int, valueTwo: Int) {
        guard parents[valueOne] != nil, parents[valueTwo] != nil else {
            return
        }
        
        if let valueOneRoot = find(valueOne), let valueTwoRoot = find(valueTwo) {
            // Only merge if they are in different sets
            if valueOneRoot != valueTwoRoot {
                parents[valueOneRoot] = valueTwoRoot
            }
        }
    }
}

```

## Interview Guide: Optimization 1 - Union by Rank

**(Interviewer: "What is the weakness of this naive approach?")**

"The main weakness is that the `find` operation can become very slow, taking O(n) time. This happens if our `union` operations create tall, unbalanced trees that look like long chains. Traversing from a leaf to the root would then require visiting every single node.


To fix this, I can introduce an optimization called **Union by Rank**. The 'rank' of a set can be thought of as its height or depth.

When I perform a `union`, instead of arbitrarily connecting one root to another, I'll compare their ranks. I will always attach the root of the **shorter tree** to the root of the **taller tree**. This strategy helps keep the trees flat and balanced, preventing them from becoming long chains."

**(Interviewer: "How do you implement that?")**

"I'll need another dictionary called `ranks` to store the rank of each set's root.

1.  **`createSet(value)`**: When a new set is created, its rank is `0`.
2.  **`union(valueOne, valueTwo)`**: After finding the roots, I'll compare `ranks[rootOne]` and `ranks[rootTwo]`.
    * If `rank(A) < rank(B)`, I'll make B the parent of A. The ranks don't change.
    * If `rank(A) > rank(B)`, I'll make A the parent of B. The ranks don't change.
    * If `rank(A) == rank(B)`, I can pick either one as the new parent, but I must then increment the rank of the new parent by 1.

This optimization improves the time complexity of `find` and `union` to **O(log n)**, which is a significant improvement."

```Swift
// Solution 2: Union by Rank Optimization
// Time Complexity: find -> O(log n), union -> O(log n)
// Space Complexity: O(n)
class UnionFind {
    var parents: [Int: Int] = [:]
    var ranks: [Int: Int] = [:]

    // O(1) time | O(1) space
    func createSet(value: Int) {
        parents[value] = value
        ranks[value] = 0
    }

    // O(log n) time | O(1) space
    func find(_ value: Int) -> Int? {
        guard parents[value] != nil else {
            return nil
        }
        
        var currentParent = value
        while currentParent != parents[currentParent]! {
            currentParent = parents[currentParent]!
        }
        return currentParent
    }

    // O(log n) time | O(1) space
    func union(valueOne: Int, valueTwo: Int) {
        guard parents[valueOne] != nil, parents[valueTwo] != nil else {
            return
        }
        
        guard let valueOneRoot = find(valueOne), let valueTwoRoot = find(valueTwo) else {
            return
        }
        
        if valueOneRoot != valueTwoRoot {
            let rankOne = ranks[valueOneRoot]!
            let rankTwo = ranks[valueTwoRoot]!
            
            if rankOne < rankTwo {
                parents[valueOneRoot] = valueTwoRoot
            } else if rankOne > rankTwo {
                parents[valueTwoRoot] = valueOneRoot
            } else {
                // Ranks are equal, make one the parent and increment its rank
                parents[valueOneRoot] = valueTwoRoot
                ranks[valueTwoRoot]! += 1
            }
        }
    }
}

```

## Interview Guide: Optimization 2 - Path Compression

**(Interviewer: "Can we do even better?")**

"Yes, there's one more powerful optimization called **Path Compression**. It complements Union by Rank perfectly.

The idea is that when we call `find(value)`, we traverse up the tree to find the root. After we find the root, all the nodes we visited on that path can be updated to point **directly** to the root.

This dramatically flattens the tree structure during the `find` operation itself. The next time we call `find` on any of those nodes, the lookup will be almost instant because they will be directly connected to the root."

**(Interviewer: "How would you modify your `find` method?")**

"I'll modify the `find` method to be recursive.

1.  The base case for the recursion is when a node is its own parent (it's the root).
2.  If it's not the root, I'll recursively call `find` on its parent.
3.  When the recursion returns the root of the set, I'll update the current node's parent to be that root before returning it.

When both Union by Rank and Path Compression are used together, the time complexity for `find` and `union` becomes nearly constant time, specifically **O(α(n))**, where α is the Inverse Ackermann function. For all practical purposes in an interview, we can describe this as being **amortized O(1)**."

```Swift
// Solution 3: Path Compression & Union by Rank
// Time Complexity: find -> O(α(n)), union -> O(α(n))
// (Amortized O(1) for practical purposes)
// Space Complexity: O(n)
class UnionFind {
    var parents: [Int: Int] = [:]
    var ranks: [Int: Int] = [:]

    // O(1) time | O(1) space
    func createSet(value: Int) {
        parents[value] = value
        ranks[value] = 0
    }

    // O(α(n)) time | O(α(n)) space (due to recursion stack)
    // α is the Inverse Ackermann function, so it's nearly constant
    func find(_ value: Int) -> Int? {
        guard let parent = parents[value] else {
            return nil
        }

        // Base case: if the node is its own parent, it's the root.
        if value == parent {
            return value
        }
        
        // Recursively find the root of the parent.
        if let root = find(parent) {
            // Path Compression: set our direct parent to be the root.
            parents[value] = root
            return root
        }
        
        return nil // Should not be reached if value exists
    }

    // O(α(n)) time | O(α(n)) space
    func union(valueOne: Int, valueTwo: Int) {
        guard parents[valueOne] != nil, parents[valueTwo] != nil else {
            return
        }
        
        guard let valueOneRoot = find(valueOne), let valueTwoRoot = find(valueTwo) else {
            return
        }
        
        if valueOneRoot != valueTwoRoot {
            let rankOne = ranks[valueOneRoot]!
            let rankTwo = ranks[valueTwoRoot]!
            
            if rankOne < rankTwo {
                parents[valueOneRoot] = valueTwoRoot
            } else if rankOne > rankTwo {
                parents[valueTwoRoot] = valueOneRoot
            } else {
                parents[valueOneRoot] = valueTwoRoot
                ranks[valueTwoRoot]! += 1
            }
        }
    }
}
```

---

### Phase 3: Proof of Learning (After the Session)
*This is where the knowledge becomes permanent. This section is completed from memory.*

#### My "Blank Page" Summary (Feynman Technique)
*(Write your summary of the topic here without looking at your notes from Phase 2. Draw a diagram if it helps.)*

#### Consolidation Checklist
- [ ] **Teach-Back Complete:** I have stood up and explained this concept out loud for 60 seconds.
- [ ] **Next-Day Review Scheduled:** I have added a 5-minute task to my plan for tomorrow to review this note.
- [ ] **Weekly Review Scheduled:** I have added this note to my list for the upcoming "Weekly Knowledge Review" session.
