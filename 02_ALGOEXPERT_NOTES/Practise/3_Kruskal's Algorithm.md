# Learning Session: Kruskal's Algorithm

---

### **Metadata**
- **Date:** `12-10-2025`
- **Source:** https://www.algoexpert.io/questions/kruskals-algorithm

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

# Problem: Kruskal's Algorithm for Minimum Spanning Tree (MST)

**Goal:** You're given a weighted, undirected graph represented by an adjacency list. Your task is to implement Kruskal's Algorithm to find a **Minimum Spanning Tree (MST)** for this graph.

An MST is a special subgraph that connects all the vertices together with the minimum possible total edge weight, without forming any cycles.

**Example Input:**
The input `edges` is an adjacency list where `edges[i]` lists the neighbors of vertex `i`. Each neighbor is an array `[neighbor, weight]`.

```
edges = [
  [[1, 3], [2, 5]],
  [[0, 3], [2, 10], [3, 12]],
  [[0, 5], [1, 10]],
  [[1, 12]]
]
```

**Example Output:**
The output should be the adjacency list representation of the resulting MST.

```
[
  [[1, 3], [2, 5]],
  [[0, 3], [3, 12]],
  [[0, 5]],
  [[1, 12]]
]
```

## Conceptual Overview & Graph Terminology

Before diving into the algorithm, let's clarify some essential graph terms.

* **Graph:** A collection of nodes (vertices) connected by lines (edges). Think of a map of cities and roads.
* **Vertex (or Node):** A point in the graph. (A city)
* **Edge:** A connection between two vertices. (A road)
* **Weight:** A cost associated with an edge. (The length or toll of a road)
* **Undirected Graph:** Edges are two-way. A road from City A to B is also a road from B to A.
* **Cycle:** A path that starts and ends at the same vertex. (A round trip)
* **Tree:** A graph with no cycles.
* **Spanning Tree:** A tree that includes **every single vertex** from the original graph.
* **Minimum Spanning Tree (MST):** A spanning tree with the lowest possible sum of edge weights. It's the cheapest way to connect everything.


### Kruskal's Algorithm: The "Greedy" Strategy

Kruskal's algorithm is a **greedy algorithm**. This means it makes the most optimal choice at each step without worrying about the future. The strategy is brilliantly simple:

1.  **Sort All Edges:** Create a single list of all unique edges in the graph and sort them by weight in ascending order.
2.  **Iterate and Add:** Go through the sorted edges one by one, from cheapest to most expensive.
3.  **Check for Cycles:** For each edge, add it to your MST **only if it does not create a cycle** with the edges you've already added.
4.  **Stop When Done:** The process is complete when you have added `V - 1` edges to your MST, where `V` is the total number of vertices.

### The Million-Dollar Question: How to Detect Cycles?

This is where the **Union-Find** data structure comes in. It is the perfect tool for this job.

1.  **Initialization:** We start by putting every vertex in its own set.
2.  **The Check:** When we consider adding an edge between vertex `A` and vertex `B`:
    * We use `find(A)` and `find(B)` to see which sets they belong to.
    * If `find(A)` is the same as `find(B)`, it means they are already connected in the same component. Adding this edge would create a cycle, so we **discard it**.
    * If `find(A)` is different from `find(B)`, they are in separate components. We **add the edge** to our MST and then **merge** their sets using `union(A, B)`.


## How to Whiteboard Kruskal's Algorithm

**(Start by drawing a few disconnected dots on the board, representing the vertices.)**

"To find the Minimum Spanning Tree, I'm going to use Kruskal's Algorithm. It's a greedy approach that is very efficient and easy to understand."

**1. Step 1: Sort the Edges**
"First, I'll take all the edges from the graph and put them into a single list, sorted by weight from smallest to largest."

**(On the side of the board, list the edges from your example graph, sorted by weight.)**
* `(0, 1)` - weight 3
* `(0, 2)` - weight 5
* `(1, 2)` - weight 10
* `(1, 3)` - weight 12

**2. Step 2: Iterate and Use Union-Find**
"Next, I'll iterate through this sorted list. To prevent cycles, I'll use a Union-Find data structure. Initially, every vertex is in its own set."

**(Point to the disconnected dots.)**

"Let's process the edges:"

* **Edge (0, 1) - weight 3:** "Are 0 and 1 in the same set? No. So, I'll add this edge to our MST and `union` them." **(Draw the edge between 0 and 1. Circle them to show they are now one set.)**
* **Edge (0, 2) - weight 5:** "Are 0 and 2 in the same set? No. The root of 0 is now 1 (or vice-versa), which is different from 2's root. So, I'll add this edge and `union` them." **(Draw the edge. Expand your circle to include 0, 1, and 2.)**
* **Edge (1, 2) - weight 10:** "Are 1 and 2 in the same set? Yes, they are. `find(1)` and `find(2)` will return the same root. Adding this edge would create a cycle (0-1-2-0). So, we **skip** it." **(Put an 'X' over this edge in your list.)**
* **Edge (1, 3) - weight 12:** "Are 1 and 3 in the same set? No. So, I'll add it and `union` them." **(Draw the edge and expand the circle to include all four nodes.)**

**3. Step 3: Conclusion**
"I have now added `V-1` (which is 3) edges. All vertices are connected, and I have a Minimum Spanning Tree with a total weight of 3 + 5 + 12 = 20."

### Interview Walkthrough Script

**(Interviewer: "How would you find the minimum cost to connect all these nodes?")**

"This sounds like a classic Minimum Spanning Tree problem. My preferred method for this is **Kruskal's Algorithm** because it's very effective, especially for sparse graphs.

My approach will be a **greedy** one. The main steps are:

1.  First, I'll transform the input adjacency list into a simple list of all unique edges, represented as tuples like `(vertex1, vertex2, weight)`.
2.  Then, I'll sort this list of edges in non-decreasing order of their weights.
3.  The core of the algorithm is to iterate through this sorted list and pick edges to build our MST. The main challenge is to avoid creating cycles. To solve this, I'll use a **Union-Find data structure** with both **path compression** and **union by rank** optimizations for near-constant time performance.
4.  I'll initialize the Union-Find structure with each vertex in its own disjoint set.
5.  As I iterate through the sorted edges `(u, v)`, I'll check if `u` and `v` are already in the same set by comparing `find(u)` and `find(v)`.
    * If they are different, it's safe to add this edge to my result. I'll then perform a `union(u, v)` operation.
    * If they are the same, adding the edge would form a cycle, so I will discard it.
6.  I'll continue this until I have added `V-1` edges, where `V` is the number of vertices. The result will be the Minimum Spanning Tree."

```Swift
// Kruskal's Algorithm using Union-Find
// The Union-Find logic is implemented with helper functions
// operating on 'parents' and 'ranks' arrays.

class Program {
    // O(e * log(e) + e * α(v)) time | O(e + v) space
    // which simplifies to O(e * log(e))
    // e = number of edges, v = number of vertices
    func kruskalsAlgorithm(edges: [[[Int]]]) -> [[[Int]]] {
        let numberOfVertices = edges.count
        var sortedEdges: [(Int, Int, Int)] = []

        // 1. Create a flat list of unique edges
        for (sourceIndex, vertex) in edges.enumerated() {
            for edge in vertex {
                let destinationIndex = edge[0]
                let weight = edge[1]
                // Avoid adding duplicate edges (e.g., [0,1] and [1,0])
                if destinationIndex > sourceIndex {
                    sortedEdges.append((sourceIndex, destinationIndex, weight))
                }
            }
        }
        
        // 2. Sort the edges by weight
        sortedEdges.sort { $0.2 < $1.2 }
        
        var parents = Array(0..<numberOfVertices)
        var ranks = Array(repeating: 0, count: numberOfVertices)
        var mst: [[[Int]]] = Array(repeating: [], count: numberOfVertices)
        var edgesAdded = 0
        
        // 3. Iterate through sorted edges and apply Union-Find
        for edge in sortedEdges {
            // Stop when we have V-1 edges in our MST
            if edgesAdded == numberOfVertices - 1 {
                break
            }
            
            let vertex1Root = find(edge.0, &parents)
            let vertex2Root = find(edge.1, &parents)
            
            // 4. If they are not in the same set, add edge to MST and union them
            if vertex1Root != vertex2Root {
                mst[edge.0].append([edge.1, edge.2])
                mst[edge.1].append([edge.0, edge.2])
                edgesAdded += 1
                union(vertex1Root, vertex2Root, &parents, &ranks)
            }
        }
        
        return mst
    }

    // Find operation with path compression
    func find(_ vertex: Int, _ parents: inout [Int]) -> Int {
        if vertex != parents[vertex] {
            // Path compression happens here
            parents[vertex] = find(parents[vertex], &parents)
        }
        return parents[vertex]
    }
    
    // Union operation with union by rank
    func union(_ vertex1Root: Int, _ vertex2Root: Int, _ parents: inout [Int], _ ranks: inout [Int]) {
        let rank1 = ranks[vertex1Root]
        let rank2 = ranks[vertex2Root]
        
        if rank1 < rank2 {
            parents[vertex1Root] = vertex2Root
        } else if rank1 > rank2 {
            parents[vertex2Root] = vertex1Root
        } else {
            // Ranks are equal, make one the parent and increment its rank
            parents[vertex2Root] = vertex1Root
            ranks[vertex1Root] += 1
        }
    }
}
```

## Complexity Analysis

* **Time Complexity: O(E log E)**
    * Extracting the edges into a list takes O(V + E) time.
    * Sorting the list of `E` edges is the most expensive step, taking **O(E log E)** time.
    * Iterating through the sorted edges involves `E` iterations. In each iteration, we perform two `find` operations and at most one `union`. With optimizations, these are nearly constant time (O(α(V))). This part takes roughly O(E * α(V)).
    * The dominant factor is sorting, so the overall time complexity is **O(E log E)**.

* **Space Complexity: O(V + E)**
    * We need O(E) space to store the list of sorted edges.
    * The Union-Find data structure requires `parents` and `ranks` arrays, which take O(V) space.
    * The resulting `mst` also takes O(V) space since it will have `V-1` edges.
    * Therefore, the total space complexity is O(V + E).

## Insight Notes: Kruskal's in the Wild

#### Real-World Use Cases

Kruskal's algorithm is fundamental in network design and optimization.
* **Network Cabling:** Finding the cheapest way to lay fiber optic or ethernet cables to connect a set of locations.
* **Electrical Grids:** Designing a power grid that connects all substations with the minimum amount of wire.
* **Circuit Design:** Minimizing the amount of metal needed to connect all the pins on a printed circuit board (PCB).
* **Clustering Algorithms:** In data science, it can be used to group similar data points together.

#### How It Appears in Coding Interviews

Directly asking for "Kruskal's Algorithm" is possible, but it's more common for it to be the hidden solution.

* **Keywords:** Look for problems asking to **"connect all points"** with **"minimum cost,"** **"cheapest way,"** or **"least amount."**
* **The "Connecting Islands" Problem:** You are given a set of islands and the costs to build bridges between them. Find the minimum cost to connect all the islands. This is a direct application of MST.
* **Distinguishing from Shortest Path:** A common trap is confusing MST with shortest path algorithms (like Dijkstra's).
    * **MST (Kruskal's):** Finds the cheapest way to connect **ALL** nodes in a graph. The total cost is minimized.
    * **Shortest Path (Dijkstra's):** Finds the cheapest path from a **single source node** to **another specific node**.

If the goal is a global connection at minimum cost, think MST. If the goal is a specific A-to-B route, think shortest path.

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
