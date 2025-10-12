# Learning Session: Binary Tree Diameter

---

### **Metadata**
- **Date:** `12-10-2025`
- **Source:** https://www.algoexpert.io/questions/binary-tree-diameter

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

# Problem: Binary Tree Diameter

**Goal:** Write a function that takes a Binary Tree and returns its **diameter**. The diameter is defined as the length of the **longest path** between any two nodes in the tree. This path does not necessarily have to pass through the root of the tree. The length is the number of edges.

**Example Input:**
A binary tree:
```
          1
         / \
        3   2
       / \
      7   4
     /     \
    8       5
   /         \
  9           6
```

**Example Output:** `6`
(The longest path is `9 -> 8 -> 7 -> 3 -> 4 -> 5 -> 6`, which has 6 edges.)

## Conceptual Overview & Tree Terminology

To understand this problem, we need to know the basic parts of a tree.

* **Node:** A single element in the tree (e.g., the circle with `1`). Each node has a `value`, a `left` child, and a `right` child.
* **Root:** The top-most node of the tree (node `1`).
* **Child:** A node directly connected to another node when moving away from the root. (`3` is a child of `1`).
* **Parent:** The opposite of a child. (`1` is the parent of `3`).
* **Leaf Node:** A node with no children (e.g., `9`, `2`, `6`).
* **Height of a Node:** The number of edges on the longest path from that node down to a leaf node. The height of node `3` is `3` (path `3 -> 7 -> 8 -> 9`).
* **Path:** A sequence of connected nodes.

### The Core Challenge

The key insight is that for any given node, the longest path that **passes through it** is simply the **height of its left subtree** + **the height of its right subtree**.

However, the true diameter of the entire tree might not pass through the root. It could be entirely contained within the left subtree or the right subtree.

For example, in a very large left subtree, the longest path might be between two leaf nodes deep inside it, never involving the root or the right subtree at all.

This means we need a way to get two critical pieces of information from every single node as we traverse the tree:
1.  What is the **height** of the subtree starting at this node?
2.  What is the **diameter** of the longest path found *so far* within this node's subtrees?

A recursive approach that calculates both of these from the bottom up is the perfect solution.


## How to Whiteboard the Binary Tree Diameter

**(Start by drawing the sample tree on the board.)**

"To find the diameter, I'll use a recursive Depth-First Search. The main idea is that for any node, the solution depends on the solutions from its children. So, I'll write a helper function that returns two pieces of information for any given subtree: its **height** and its **diameter**."

**(Circle a small subtree, like the one rooted at node 3.)**

"Let's look at node 3.
1.  The longest path that **passes through node 3** is the longest path down its left side plus the longest path down its right side. The height of the left subtree (rooted at 7) is 2. The height of the right subtree (rooted at 4) is 2. So, the path through node 3 has a length of `2 + 2 = 4`.
2.  But the longest path in the *entire* subtree of node 3 might be hidden entirely inside its children. We need to ask: what was the diameter of the subtree at 7? What about at 4?

So, the diameter at node 3 is the **maximum** of:
* The diameter found in the left child's world (`diameter of subtree 7`).
* The diameter found in the right child's world (`diameter of subtree 4`).
* The path that goes through node 3 itself (`height of left + height of right`).

We can solve this by starting at the leaves and passing this information up to the root."

## Interview Walkthrough Script

**(Interviewer: "How would you find the diameter of this binary tree?")**

"To solve this, I'll use a recursive, post-order traversal approach. The key challenge is that the longest path might not involve the root, so we need to check the diameter at every node.

My strategy is to write a helper function that, for any given node, returns a pair of values: `(diameter, height)`.

Here's the logic for this recursive helper function, let's call it `getTreeInfo`:

1.  **Base Case:** If the current node is `null` (we've gone past a leaf), it contributes nothing. I'll return a `(diameter: 0, height: 0)` object.

2.  **Recursive Step:** I will make two recursive calls: one for the left child and one for the right child. This will give me the `(diameter, height)` information for both subtrees.
    * `leftTreeInfo = getTreeInfo(tree.left)`
    * `rightTreeInfo = getTreeInfo(tree.right)`

3.  **Calculations at the Current Node:**
    * First, I'll calculate the longest path that could possibly pass **through the current node**. This is simply `leftTreeInfo.height + rightTreeInfo.height`.
    * Next, I need to find the maximum diameter found *so far*. This will be the maximum of the diameters passed up from my children (`leftTreeInfo.diameter` and `rightTreeInfo.diameter`).
    * The `currentDiameter` is the maximum of the path through the current node and the max diameter found so far.
    * The `currentHeight` is `1 + max(leftTreeInfo.height, rightTreeInfo.height)`.

4.  **Return Value:** Finally, I'll return a new info object containing `(currentDiameter, currentHeight)` to the node's parent.

The main `binaryTreeDiameter` function will simply kick off this process by calling the helper on the root and then returning the diameter from the final result."

```Swift
// Main BinaryTree class definition
class BinaryTree {
    var value: Int
    var left: BinaryTree?
    var right: BinaryTree?

    init(value: Int) {
        self.value = value
    }
}

// Helper struct to pass information up the recursion
struct TreeInfo {
    var diameter: Int
    var height: Int
}

class Program {
    // O(n) time | O(h) space
    // n = number of nodes, h = height of the tree
    func binaryTreeDiameter(_ tree: BinaryTree) -> Int {
        // Kick off the recursive helper and return the diameter from the final result
        return getTreeInfo(tree).diameter
    }
    
    func getTreeInfo(_ tree: BinaryTree?) -> TreeInfo {
        // Base case: a null node has no height or diameter
        guard let tree = tree else {
            return TreeInfo(diameter: 0, height: 0)
        }
        
        // Recursive step: get info from children
        let leftTreeInfo = getTreeInfo(tree.left)
        let rightTreeInfo = getTreeInfo(tree.right)
        
        // --- Calculations at the current node ---
        
        // 1. Longest path through the current node
        let longestPathThroughRoot = leftTreeInfo.height + rightTreeInfo.height
        
        // 2. The max diameter seen so far (from children's subtrees)
        let maxDiameterSoFar = max(leftTreeInfo.diameter, rightTreeInfo.diameter)
        
        // 3. The current diameter is the max of the above two
        let currentDiameter = max(longestPathThroughRoot, maxDiameterSoFar)
        
        // 4. The current height to pass up to the parent
        let currentHeight = 1 + max(leftTreeInfo.height, rightTreeInfo.height)
        
        return TreeInfo(diameter: currentDiameter, height: currentHeight)
    }
}
```

## Complexity Analysis

* **Time Complexity: O(N)**
    * Where N is the number of nodes in the tree. We visit each node exactly once to compute its height and the diameter of the subtree rooted at it.

* **Space Complexity: O(H)**
    * Where H is the height of the tree. This space is used by the recursion call stack.
    * In the best case (a perfectly balanced tree), H is approximately log(N), making the space complexity O(log N).
    * In the worst case (a skewed tree that looks like a linked list), H is N, making the space complexity O(N).

## Insight Notes: Binary Tree Diameter

#### Real-World Use Cases

The concept of finding the "longest path" in a tree-like structure is useful in various domains:

* **Network Routing:** Calculating the maximum propagation delay in a communication network, where the diameter represents the worst-case travel time for a data packet.
* **Bioinformatics:** Analyzing the structure of molecules or evolutionary trees, where the diameter can represent the greatest degree of divergence between two species.
* **Social Networks:** Finding the "six degrees of separation" concept in a hierarchical subgroup.

#### How It Appears in Coding Interviews

This is a very popular tree question because it tests several key concepts:

* **Mastery of Recursion:** It's a non-trivial recursive problem that requires a clear understanding of base cases and recursive steps.
* **Depth-First Search (DFS):** The algorithm is a classic example of a post-order DFS traversal (we process children before the parent).
* **The "Return Multiple Values" Pattern:** The key insight is that a single recursive pass can solve the problem if the function returns multiple pieces of information (height *and* diameter). This pattern is crucial for many optimal tree algorithms. If you try to solve it by calculating height first and then diameter in separate functions, you'll end up with a less efficient O(N^2) solution.
* **Problem Decomposition:** It shows you can break down a complex global property (the tree's diameter) into smaller, manageable local calculations at each node.

Variations on this problem include "Maximum Path Sum" (where nodes have positive or negative values and you want the path with the largest sum) and "Longest Univalue Path." The core recursive strategy remains the same.

---
---

### Phase 3: Proof of Learning (After the Session)
*This is where the knowledge becomes permanent. This section is completed from memory.*

#### My "Blank Page" Summary (Feynman Technique)
*(Write your summary of the topic here without looking at your notes from Phase 2. Draw a diagram if it helps.)*

#### Consolidation Checklist
- [ ] **Teach-Back Complete:** I have stood up and explained this concept out loud for 60 seconds.
- [ ] **Next-Day Review Scheduled:** I have added a 5-minute task to my plan for tomorrow to review this note.
- [ ] **Weekly Review Scheduled:** I have added this note to my list for the upcoming "Weekly Knowledge Review" session.
