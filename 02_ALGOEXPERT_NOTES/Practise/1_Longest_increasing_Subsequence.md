# Learning Session: Longest increasing Subsequence

---

### **Metadata**
- **Date:** `17-09-2025`, `27-09-2025`
- **Source:** https://www.algoexpert.io/questions/longest-increasing-subsequence

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is Answer and understand the question The longest increasing subsequence.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)
*My structured notes, guided by deep questions.*

- [ ] **Embrace the Struggle:** Attempt problems on your own for 15-20 mins before seeking hints.

#### Notes

---

### DSA Core Concepts for Interviews

#### Dynamic Programming (DP)
- **Core Idea:** A technique for solving complex problems by breaking them into simpler, overlapping subproblems. You solve each subproblem only once and store its solution.
- **Interview Wording:** "This problem has *optimal substructure* and *overlapping subproblems*, which makes it a good candidate for Dynamic Programming. We can use a *bottom-up approach* to build a table of solutions to subproblems."
- **Key Identifiers:** Look for problems that ask for an "**optimal**" (max/min/longest/shortest) solution that can be built up from previous decisions.

#### Big O Notation
- **Core Idea:** Describes how the runtime or memory usage of an algorithm scales with the input size `n`.
- **Interview Wording:** "The brute-force solution is *O(n^2)$*, but we can optimize it to *$O(n log n)* using a more efficient data structure."
- **Key Types:**
    - **O(1) (Constant):** Instant. Accessing an array element by index.
    - **O(log n) (Logarithmic):** Very fast. Binary search.
    - **O(n) (Linear):** Scales directly with input. Looping through an array.
    - **O(n^2) (Quadratic):** Slow. Nested loops over the same input.

#### Core Data Structures
- **Arrays:** Fast, fixed-size, contiguous memory. *O(1)* access.
- **Linked Lists:** Dynamic size, non-contiguous. Efficient insertions/deletions (*O(1)*), but slow lookups (*O(n)*).
- **Hash Tables (Dictionaries/Maps):** Key-value pairs. Amortized *O(1)* for lookups, insertions, and deletions. The go-to for quick data retrieval.
- **Trees:** Hierarchical data. *Binary Search Trees (BSTs)* provide sorted structure and *O(log n)* search on average.
- **Graphs:** Nodes and edges. Represents networks (social, maps). Traversal is key (BFS for shortest path, DFS for pathfinding/exhausting options).

#### Key Algorithms
- **Recursion:** A function that calls itself. Breaks a problem into smaller, self-similar versions. Watch out for the *base case* to avoid infinite loops.
- **Binary Search:** The essential *$O(\log n)$* search algorithm for **sorted** arrays. Involves repeatedly dividing the search space in half.
- **Graph Traversal:**
    - **BFS (Breadth-First Search):** Explores level by level. Uses a **Queue**. Finds the shortest path in an unweighted graph.
    - **DFS (Depth-First Search):** Goes as deep as possible before backtracking. Uses a **Stack** or recursion.

---

### Interview Guide: Longest Increasing Subsequence
This note is structured as a script for how you might explain the LIS problem and your solution in a technical interview.

#### Problem Statement
Given a non-empty array of integers, find the longest strictly-increasing subsequence.
- **Subsequence:** A set of numbers from the array that are not necessarily adjacent but are in the same relative order.
- **Example Input:** `[5, 7, -24, 12, 10, 2, 3, 12, 5, 6, 35]`
- **Example Output:** `[-24, 2, 3, 5, 6, 35]`


#### 1. Clarify the Problem & Confirm Understanding

**(Interviewer: "Please solve the Longest Increasing Subsequence problem.")**

"Absolutely. So, just to make sure I understand correctly, the goal is to find the longest sequence of numbers within an array that are in strictly increasing order. A key point is that this is a **subsequence**, not a subarray, meaning the elements don't have to be adjacent, but they must maintain their original relative order.

For an input like `[5, 7, -24, 12]`, the longest increasing subsequence would be `[5, 7, 12]`. Is that correct?"

**(Wait for the interviewer's confirmation.)**


#### 2. Discuss Initial Thoughts & Brute Force

"My initial thought is to consider the brute-force approach. We could generate every possible subsequence, check if each one is strictly increasing, and then keep track of the longest one we find.

However, an array of size `n` has $2^n$ subsequences. This approach would have an exponential time complexity, likely O($2^n$), which is not feasible for anything but very small inputs. So, we need a more efficient approach."



#### 3. Propose the Dynamic Programming Solution

"This problem has characteristics that make it a great candidate for **Dynamic Programming**, specifically *optimal substructure* and *overlapping subproblems*. We can build the solution for the entire array by using solutions to smaller, previously solved subproblems.

My approach will be to iterate through the array and, for each element, answer the core question: **'What is the longest increasing subsequence that *ends* at this specific element?'**"



#### 4. Detail the Implementation Plan & Share Code

"To implement this, I'll use two auxiliary arrays, both the same size as the input array.

1.  The first, let's call it `lengths`, will store the length of the LIS ending at each index `i`. I'll initialize all its values to `1`, since any single element is, by itself, an LIS of length one.

2.  The second array, `sequences`, is crucial for reconstructing the final result. It will store the index of the *previous* element in the LIS ending at index `i`. This creates a trail of breadcrumbs we can follow backward later.

My algorithm will be to iterate through the input array with an index `i`. Inside this loop, I'll have a nested loop that looks backward from the beginning of the array up to `i`, let's call that index `j`.

In the inner loop, I'll check if `array[j]` is less than `array[i]`. If it is, it's a candidate for extending a subsequence. I'll then check if using this predecessor creates a longer subsequence—that is, if `lengths[j] + 1` is greater than the current `lengths[i]`. If it is, I'll update `lengths[i]` and set `sequences[i] = j` to record this new best path."

Here is the Swift code that implements this logic:

```swift
class Program {
  func longestIncreasingSubsequence(_ array: [Int]) -> [Int] {

    var lengthArray = Array(repeating: 1, count: array.count)
    var sequenceArray = Array<Int?>(repeating: nil, count:array.count)
    var biggestLengthIndex = 0

    for (mainIndex, mainItem) in array.enumerated() {
      for (subIndex, subItem) in array[0 ..< mainIndex].enumerated() {
        if subItem < mainItem {
          let lengthAtSubIndexPlusOne = lengthArray[subIndex] + 1
          if lengthAtSubIndexPlusOne >= lengthArray[mainIndex] {
            lengthArray[mainIndex] = lengthAtSubIndexPlusOne
            sequenceArray[mainIndex] = subIndex
          }
        }
      }
      if lengthArray[mainIndex] >= lengthArray[biggestLengthIndex] {
        biggestLengthIndex = mainIndex
      }
    }

    // Reconstructing the Result
    var result = [Int]()
    var currentIndex: Int? = biggestLengthIndex

    while currentIndex != nil {
        result.append(array[currentIndex!])
        currentIndex = sequenceArray[currentIndex!]
    }
    
    return result.reversed()
  }
}
```

#### 5. Explain Reconstructing the Result

"While iterating, I'll also keep track of the index that holds the overall maximum length found so far (`biggestLengthIndex`).

After the loops complete, I'll have the `lengths` and `sequences` arrays fully populated. Starting from `biggestLengthIndex`, I'll use the `sequences` array to backtrack, jumping from one index to the previous one and collecting the numbers along the way. Since this builds the sequence in reverse, I'll append the numbers to a list and then reverse it once at the very end to get the final answer in the correct order. This is a very efficient way to build the final array."


#### 6. Analyze the Complexity

"Now, for the complexity analysis of this approach:

* The **Time Complexity** will be **O(n^2)**. This is because of the two nested loops, where for each element, we are potentially looking at all the elements before it.
* The **Space Complexity** will be **O(n)**. This is because we are creating two additional arrays (`lengths` and `sequences`), each with a size proportional to the input array `n`."


#### 7. Mention Further Optimizations

"This O(n^2) solution is a solid and very common approach. It's worth mentioning that this problem can be further optimized to run in **O(n log n)** time. That more advanced algorithm involves a clever use of binary search to maintain an array of the smallest ending elements for subsequences of various lengths. If we have time, I'd be happy to elaborate on that as well."

---


### Interview Guide: LIS (O(n log n) Approach)

#### 1. High-Level Strategy

"My approach will be an optimized O(n log n) solution using binary search. The core idea is to maintain a helper array, let's call it `indices`, that helps us build the longest possible subsequence at every step.

This `indices` array is a bit abstract. You can think of `indices[k]` as storing the index of the best candidate to end an increasing subsequence of length `k`. The 'best' candidate is the one with the smallest value, because it's the easiest to add to in the future."

#### 2. The Role of the Helper Arrays

"I'll use two helper arrays:

1.  **`indices`**: This is the brain of the algorithm. `indices[k]` will hold the **index** from the original array that ends the best possible increasing subsequence of length `k`.
2.  **`sequences`**: This is just for memory. It will store the predecessor of each number, allowing us to rebuild the final sequence once we're done."

#### 3. The Process in Plain English

"I will iterate through the input array, number by number. For each number, I will:

1.  Use a **binary search** on my `indices` array. The goal of this search is to find the length of the longest subsequence that can be formed by including the current number.
2.  Once I have this new length, I'll update my `sequences` (predecessor) array to create a 'breadcrumb trail', linking the current number's index back to the end of the subsequence that came before it.
3.  Finally, I'll update the `indices` array itself, placing the current number's index as the new 'best' ending point for a subsequence of this new length."

```Swift
// The main function orchestrating the LIS algorithm
func longestIncreasingSubsequence(_ array: [Int]) -> [Int] {
    // sequences[i] stores the index of the predecessor of array[i]
    var sequences = Array<Int?>(repeating: nil, count: array.count)
    // indices[k] stores the index of the smallest end element of an LIS of length k
    var indices = Array<Int?>(repeating: nil, count: array.count + 1)
    // Stores the length of the longest increasing subsequence found so far
    var length = 0

    for (i, num) in array.enumerated() {
        // Find the length of the LIS ending with the current number
        let newLength = binarySearch(1, length, indices, array, num)
        
        // Set the predecessor for the current index
        sequences[i] = indices[newLength - 1]
        
        // Update the indices array with the current index for the new length
        indices[newLength] = i
        
        // Update the overall longest length found
        length = max(length, newLength)
    }
    
    // Get the index of the last element of the LIS to start the build process
    let lastElementIndex = indices[length]
    
    return buildSequence(array, sequences, lastElementIndex)
}

// Iterative binary search to find the length of the LIS
// ending with 'num'.
func binarySearch(_ startIdx: Int, _ endIdx: Int, _ indices: [Int?], _ array: [Int], _ num: Int) -> Int {
    var low = startIdx
    var high = endIdx
    
    while low <= high {
        let middleIdx = low + (high - low) / 2
        
        // Use 'middleIdx' to get an index from 'indices', then use that
        // index to get the value from the main 'array' for comparison.
        if let indexAtMiddle = indices[middleIdx], array[indexAtMiddle] < num {
            low = middleIdx + 1
        } else {
            high = middleIdx - 1
        }
    }
    // 'low' now represents the new length
    return low
}

// Backtracks through the sequences array to build the final result
func buildSequence(_ array: [Int], _ sequences: [Int?], _ currentIdx: Int?) -> [Int] {
    var sequence: [Int] = []
    var currentIndex = currentIdx
    
    while currentIndex != nil {
        sequence.append(array[currentIndex!])
        currentIndex = sequences[currentIndex!]
    }
    
    return sequence.reversed()
}

```

#### Complexity Analysis

* **Time Complexity: O(n log n)**
    * We iterate through the main array once, which is O(n).
    * Inside the loop, we perform a binary search, which takes O(log n) time.
    * This gives a total of O(n * log n).

* **Space Complexity: O(n)**
    * We create two extra arrays (`sequences` and `indices`), both of which are proportional to the size of the input array.


#### Key Takeaway for Future You

When you come back to this, remember this one thing:

The `indices` array is the most important part. It **does not** store the LIS. It stores the **location** of the best (smallest) numbers that can **end** an increasing subsequence of a certain length. This is what makes the binary search possible.

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
