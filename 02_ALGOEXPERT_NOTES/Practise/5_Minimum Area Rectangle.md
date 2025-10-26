# Learning Session: Minimum Area Rectangle

---

### **Metadata**
- **Date:** `12-10-2025`
- **Source:** https://www.algoexpert.io/questions/minimum-area-rectangle

---

### Phase 1: Preparation Checklist (Before the Session)
*The setup for effective deep work.*

- [x] **Goal Set:** My specific goal for this session is clear.
- [x] **Environment Ready:** Phone is off/away, and my workspace is distraction-free.
- [x] **Brain Primed:** I have completed a 2-minute review of the previous related topic.

---

### Phase 2: Active Notes (During the Session)
*My structured notes, guided by deep questions.*

- [] **Embrace the Struggle:** Attempt problems on your own for 15-20 mins before seeking hints.


#### Notes

---

# Problem: Minimum Area Rectangle

**Goal:** Given an array of points `[x, y]` plotted on a 2D plane, find the minimum area of a rectangle that can be formed using any four of these points. The rectangle's sides **must be parallel to the x and y axes**. If no rectangle can be formed, return 0.

**Example Input:**
```
points = [
  [1, 5], [5, 1], [4, 2], [2, 4],
  [2, 2], [1, 2], [4, 5], [2, 5],
  [-1, -2]
]
```

**Example Output:** `3`
(The rectangle with corners `[1, 5]`, `[2, 5]`, `[1, 2]`, and `[2, 2]` has an area of 3.)


## Conceptual Overview

The core of this problem is geometric. A rectangle with sides parallel to the axes is defined by four points that share x and y coordinates in a specific way.

If you pick any two points that are **diagonally opposite**, say `p1 = (x1, y1)` and `p2 = (x2, y2)`, the other two corners of the rectangle are uniquely determined. They *must* be `p3 = (x1, y2)` and `p4 = (x2, y1)`.


The entire problem boils down to this: how can we efficiently check if these other two required points, `p3` and `p4`, actually exist in our input list?

The area of such a rectangle is simply `abs(x2 - x1) * abs(y2 - y1)`. Our goal is to find the combination of four points that form such a rectangle with the smallest possible area.


## How to Whiteboard the Solutions

**(Start by plotting a few points from the example on the board.)**

"My goal is to find four points that form a rectangle and have the minimum area. I'll walk through three approaches, from brute-force to optimal."

**1. The Brute-Force Approach (O(n^4))**
"The most straightforward way is to try every possible combination of four points from the input list. For each combination, I'd check if they form a rectangle with sides parallel to the axes. This would involve four nested loops, making it an O(n^4) solution. This is far too slow for a typical interview constraint, so we must do better."

**2. A Better Approach: Checking Diagonals (O(n^2))**
"We can improve this significantly. The key idea is that any two points can be considered potential diagonal corners of a rectangle.

Let's pick two points, `p1 = (x1, y1)` and `p2 = (x2, y2)`. If these form a diagonal, the other two corners must be `p3 = (x1, y2)` and `p4 = (x2, y1)`.

So, my algorithm will be:
1.  First, I'll put all the input points into a **Hash Set** for O(1) average time lookups.
2.  Then, I'll use two nested loops to iterate through every unique pair of points `p1` and `p2`.
3.  For each pair, I'll check if the other two required corner points, `p3` and `p4`, exist in my hash set.
4.  If they both exist, I've found a rectangle. I'll calculate its area and keep track of the minimum area found so far.

This approach has a time complexity of O(n^2) because of the nested loops, and a space complexity of O(n) for the hash set. This is a very solid solution."

**3. The Optimal Approach: Column-Based (O(n^2))**
"There's another clever O(n^2) solution that can be faster in practice for certain data distributions. Instead of iterating through all pairs of points, we can process the points column by column.

The logic is:
1.  First, I'll group all points by their x-coordinate into a dictionary, where the keys are x-values and the values are lists of y-values.
2.  I'll then iterate through this dictionary, column by column. For each column, I'll iterate through all pairs of y-values (`y1`, `y2`). These two points `(x, y1)` and `(x, y2)` form a vertical edge.
3.  I'll store this vertical edge (represented by its y-coordinates) in another dictionary, mapping it to the `x` coordinate where I found it.
4.  If I'm processing a new column and I find a vertical edge that I've seen before in a previous column, I have found a rectangle! The four corners are `(previous_x, y1)`, `(previous_x, y2)`, `(current_x, y1)`, and `(current_x, y2)`. I can then calculate the area and update my minimum."

#### Solution 1: Brute-Force Solution

```Swift
// O(n^4) time | O(1) space
func minimumAreaRectangle_bruteForce(_ points: [[Int]]) -> Int {
    let n = points.count
    var minArea = Int.max
    
    for i in 0..<n {
        for j in 0..<n {
            for k in 0..<n {
                for l in 0..<n {
                    // Ensure 4 distinct points
                    if i == j || i == k || i == l || j == k || j == l || k == l {
                        continue
                    }
                    
                    let p1 = points[i]
                    let p2 = points[j]
                    let p3 = points[k]
                    let p4 = points[l]
                    
                    // Check if they form a rectangle
                    // This is complex, a better way is to create a set of points
                    // and check for coordinates, but a pure 4-loop check is possible.
                    // For simplicity, we'll just acknowledge this is the brute-force idea.
                    // A full implementation is overly complex and not practical.
                }
            }
        }
    }
    // This is conceptually O(n^4), but the practical solutions are far better.
    return minArea == Int.max ? 0 : minArea
}
```

#### Solution 2: Better Swift Code - Checking Diagonals

```Swift
// O(n^2) time | O(n) space
class Program {
    func minimumAreaRectangle(_ points: [[Int]]) -> Int {
        // 1. Create a set of points for fast lookups
        var pointSet = createPointSet(points)
        var minimumAreaFound = Int.max

        // 2. Iterate through all pairs of points
        for (currentIdx, p2) in points.enumerated() {
            let p2x = p2[0]
            let p2y = p2[1]
            
            for previousIdx in 0..<currentIdx {
                let p1 = points[previousIdx]
                let p1x = p1[0]
                let p1y = p1[1]

                // Check if points form a diagonal (not aligned horizontally or vertically)
                let pointsShareValue = (p1x == p2x || p1y == p2y)
                if pointsShareValue {
                    continue
                }

                // 3. Check if the other two corners exist
                let point3Exists = pointSet.contains("\(p1x):\(p2y)")
                let point4Exists = pointSet.contains("\(p2x):\(p1y)")

                if point3Exists && point4Exists {
                    // 4. Calculate area and update minimum
                    let currentArea = abs(p2x - p1x) * abs(p2y - p1y)
                    minimumAreaFound = min(minimumAreaFound, currentArea)
                }
            }
        }
        
        return minimumAreaFound == Int.max ? 0 : minimumAreaFound
    }

    // Helper to create a set of strings for O(1) lookups
    func createPointSet(_ points: [[Int]]) -> Set<String> {
        var pointSet: Set<String> = []
        for point in points {
            let x = point[0]
            let y = point[1]
            pointSet.insert("\(x):\(y)")
        }
        return pointSet
    }
}
```

#### Solution 3. Optimal Swift Code - Column-Based

```Swift
// O(n^2) time | O(n) space (can be O(n^2) in worst case for edges)
class Program {
    func minimumAreaRectangle(_ points: [[Int]]) -> Int {
        // 1. Group points by x-coordinate
        let columns = initializeColumns(points)
        var minimumAreaFound = Int.max
        // Maps a y-pair string "y1:y2" to the x-coordinate where it was seen
        var edgesParallelToYAxis: [String: Int] = [:]
        
        let sortedColumns = columns.keys.sorted()
        
        // 2. Iterate through columns
        for x in sortedColumns {
            var yValuesInCurrentColumn = columns[x]!
            yValuesInCurrentColumn.sort()
            
            // 3. For each column, find all vertical edges
            for (currentIdx, y2) in yValuesInCurrentColumn.enumerated() {
                for previousIdx in 0..<currentIdx {
                    let y1 = yValuesInCurrentColumn[previousIdx]
                    let pointString = "\(y1):\(y2)"
                    
                    // 4. If this edge has been seen before, we found a rectangle
                    if let previousX = edgesParallelToYAxis[pointString] {
                        let currentArea = (x - previousX) * (y2 - y1)
                        minimumAreaFound = min(minimumAreaFound, currentArea)
                    }
                    
                    // Store the current x-coordinate for this edge
                    edgesParallelToYAxis[pointString] = x
                }
            }
        }
        
        return minimumAreaFound == Int.max ? 0 : minimumAreaFound
    }

    // Helper to group points into columns
    func initializeColumns(_ points: [[Int]]) -> [Int: [Int]] {
        var columns: [Int: [Int]] = [:]
        for point in points {
            let x = point[0]
            let y = point[1]
            columns[x, default: []].append(y)
        }
        return columns
    }
}
```

## Complexity Analysis

* **Brute-Force:**
    * **Time: O(n^4)** - Four nested loops to select four points.
    * **Space: O(1)** - No extra space needed.

* **Better Solution (Checking Diagonals):**
    * **Time: O(n^2)** - Two nested loops to iterate through all pairs of points. Hash set lookups are O(1) on average.
    * **Space: O(n)** - To store `n` points in the hash set.

* **Optimal Solution (Column-Based):**
    * **Time: O(n^2)** - The dominant part is the nested loop over y-values within each column. In the worst case, all `n` points could share the same few x-coordinates, leading to a loop that runs roughly `(n/c)^2` times for `c` columns. Across all columns, this sums up to O(n^2). Sorting the y-values contributes at most O(n log n).
    * **Space: O(n)** - The `columns` dictionary stores `n` points. However, the `edgesParallelToYAxis` dictionary could, in the worst case, store O(n^2) edges if all points form unique vertical pairs. But typically, it's closer to O(n).


## Insight Notes

#### Real-World Use Cases
This type of geometric problem is common in:
* **Computer Graphics & UI Design:** Detecting bounding boxes, collisions, or alignment of UI elements.
* **Geographic Information Systems (GIS):** Finding rectangular land parcels or features in a map dataset.
* **Image Processing:** Identifying rectangular objects or features within an image.

#### How It Appears in Coding Interviews
This question is a great test of problem-solving and optimization skills.
* **The Brute-Force Trap:** Interviewers expect you to identify the brute-force solution but immediately recognize its inefficiency and move on.
* **Use of Data Structures:** The key to solving this efficiently is choosing the right data structure. Using a **Hash Set** for O(1) lookups is the most critical insight for the diagonal-checking method.
* **Preprocessing:** The column-based solution demonstrates the power of preprocessing the input (grouping points by x-coordinate) to enable a more structured and sometimes more efficient search.
* **Thinking about Edges:** The optimal solution shifts the thinking from "pairs of points" to "pairs of vertical edges". This change in perspective is a common theme in harder algorithmic problems.


## Step-by-Step Example Trace (Column-Based Solution)

Let's trace with a simpler input: `points = [[1, 5], [1, 2], [2, 5], [2, 2]]`

**1. `initializeColumns(points)`**
* **Result:** `columns = [1: [5, 2], 2: [5, 2]]`

**2. Main Loop Setup**
* `minimumAreaFound = Int.max`
* `edgesParallelToYAxis = [:]`
* `sortedColumns = [1, 2]`

**3. Processing Column `x = 1`**
* `yValuesInCurrentColumn` becomes `[2, 5]` after sorting.
* **Inner Loops (y1, y2):**
    * The only pair is `(y1, y2) = (2, 5)`.
    * `pointString` becomes `"2:5"`.
    * Is `"2:5"` in `edgesParallelToYAxis`? No.
    * **Update:** `edgesParallelToYAxis["2:5"] = 1`.
* **State after column 1:** `edgesParallelToYAxis = ["2:5": 1]`

**4. Processing Column `x = 2`**
* `yValuesInCurrentColumn` becomes `[2, 5]` after sorting.
* **Inner Loops (y1, y2):**
    * The only pair is `(y1, y2) = (2, 5)`.
    * `pointString` becomes `"2:5"`.
    * Is `"2:5"` in `edgesParallelToYAxis`? **Yes!** Its value is `previousX = 1`.
    * **A rectangle is found!**
    * `currentArea = (x - previousX) * (y2 - y1)`
    * `currentArea = (2 - 1) * (5 - 2) = 1 * 3 = 3`.
    * `minimumAreaFound` is updated from `Int.max` to `3`.
    * **Update:** `edgesParallelToYAxis["2:5"] = 2`.
* **State after column 2:** `edgesParallelToYAxis = ["2:5": 2]`

**5. End of Loop**
* The loop finishes.
* `minimumAreaFound` is `3`.
* The function returns `3`.

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
