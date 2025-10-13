# Learning Session: Radix Sort

---

### **Metadata**
- **Date:** `14-10-2025`
- **Source:** https://www.algoexpert.io/questions/radix-sort

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

# Problem: Radix Sort

**Goal:** Write a function that takes an array of non-negative integers and returns a sorted version of that array using the Radix Sort algorithm.

**Example Input:**
```
array = [8762, 654, 3008, 345, 87, 65, 234, 12, 2]
```

**Example Output:**
```
[2, 12, 65, 87, 234, 345, 654, 3008, 8762]
```

## Conceptual Overview

Radix Sort is a clever sorting algorithm that completely avoids comparing elements to each other. Instead, it sorts numbers by processing their individual digits.

The main idea is to sort the entire list of numbers multiple times, once for each digit position, starting from the **least significant digit (the ones place)** and moving to the **most significant digit (the leftmost place)**.


**The Secret Ingredient: A Stable Sort**
For this digit-by-digit process to work, the sorting algorithm used for each pass must be **stable**. A stable sort preserves the original relative order of elements that have equal sorting keys.

* **Example:** After sorting by the ones digit, the list might contain `[... 8762, 12, 2, ...]`. When we then sort by the tens digit, `12` needs to come before `8762` because `1` is less than `6`. The stability ensures that if the tens digits were the same, their order from the ones-digit-sort would be maintained.

The most common stable sorting algorithm used as a subroutine for Radix Sort is **Counting Sort**. Counting Sort is very efficient for sorting integers within a small, known range (like digits 0-9).


## How to Whiteboard Radix Sort

**(Start by writing a small array on the board, e.g., `[170, 45, 75, 90, 802]`.)**

"To solve this, I'll use Radix Sort. This is a non-comparison sort, which means it can be faster than O(n log n). It works by sorting the numbers digit by digit, from the ones place up to the largest place value. The key is to use a stable sorting algorithm like Counting Sort for each pass."

**1. Determine Number of Passes**
"First, I find the largest number, which is `802`. It has 3 digits, so we'll need 3 passes."

**2. Pass 1: Sort by Ones Digit**
"We'll sort the array based only on the ones digit (0, 5, 5, 0, 2)."
* **(Draw 10 buckets, labeled 0-9.)**
* "Place `170` in bucket 0, `45` in bucket 5, `75` in bucket 5, `90` in bucket 0, `802` in bucket 2."
* "Now, we reconstruct the array by pulling from the buckets in order. Because our subroutine is stable, `170` comes before `90`, and `45` comes before `75`."
* **Resulting Array:** `[170, 90, 802, 45, 75]`

**3. Pass 2: Sort by Tens Digit**
"Now we sort the *new* array based on the tens digit (7, 9, 0, 4, 7)."
* **Resulting Array:** `[802, 45, 170, 75, 90]` (Notice `170` comes before `75` because of stability from the previous pass).

**4. Pass 3: Sort by Hundreds Digit**
"Finally, we sort the latest array by the hundreds digit (1, 0, 0, 0, 8)."
* **Resulting Array:** `[45, 75, 90, 170, 802]` (Numbers with no hundreds digit are treated as 0).

"After the final pass, the array is fully sorted."

## Interview Walkthrough Script

"For this problem, I'll implement Radix Sort. It's an excellent choice for sorting integers and can achieve linear time complexity under the right conditions.

My strategy will be as follows:
1.  First, I'll determine the maximum number in the array. The number of digits in this max number will tell me how many sorting passes I need to make.
2.  I'll then loop from the least significant digit (ones place) up to the most significant digit.
3.  In each iteration of the loop, I will sort the entire array based on the current digit place. The crucial part here is that I must use a **stable sorting algorithm**. I will implement **Counting Sort** as a helper function for this purpose.
4.  Counting sort works by counting the occurrences of each digit (0-9), and then uses those counts to calculate the final sorted position of each number.
5.  After the final pass (for the most significant digit), the array will be completely sorted."

### Step-by-Step Example Trace

Let's trace the sample input: `[8762, 654, 3008, 345, 87, 65, 234, 12, 2]`
The maximum number is **8762**, which has 4 digits. We need 4 passes.

**Pass 1: Sort by Ones Digit (digit 0)**
* Sorting key is the last digit of each number: `2, 4, 8, 5, 7, 5, 4, 2, 2`.
* After stable sort: `[8762, 12, 2, 654, 234, 345, 65, 87, 3008]`

**Pass 2: Sort by Tens Digit (digit 1)**
* We sort the *new* array. Sorting key is the tens digit: `6, 1, 0, 5, 3, 4, 6, 8, 0`.
* After stable sort: `[2, 3008, 12, 234, 345, 654, 8762, 65, 87]`

**Pass 3: Sort by Hundreds Digit (digit 2)**
* Sorting key is the hundreds digit: `0, 0, 0, 2, 3, 6, 7, 0, 0`.
* After stable sort: `[2, 12, 65, 87, 3008, 234, 345, 654, 8762]`

**Pass 4: Sort by Thousands Digit (digit 3)**
* Sorting key is the thousands digit: `0, 0, 0, 0, 3, 0, 0, 0, 8`.
* After stable sort: `[2, 12, 65, 87, 234, 345, 654, 3008, 8762]`

The loop finishes, and the array is now fully sorted.

```Swift
class Program {
    // O(d * (n + b)) time | O(n + b) space
    // d = max number of digits, n = length of array, b = base of numbering system (10)
    func radixSort(_ array: inout [Int]) {
        if array.isEmpty {
            return
        }

        let maxNumber = array.max()!
        
        var digit = 0
        // Loop while there are still digits to process
        while (maxNumber / Int(pow(10.0, Double(digit)))) > 0 {
            countingSort(&array, digit)
            digit += 1
        }
    }

    func countingSort(_ array: inout [Int], _ digit: Int) {
        var sortedArray = Array(repeating: 0, count: array.count)
        var countArray = Array(repeating: 0, count: 10) // Digits 0-9
        
        let digitColumn = Int(pow(10.0, Double(digit)))

        // 1. Populate countArray with frequencies of digits
        for num in array {
            let countIndex = (num / digitColumn) % 10
            countArray[countIndex] += 1
        }

        // 2. Create cumulative counts
        for idx in 1..<10 {
            countArray[idx] += countArray[idx - 1]
        }
        
        // 3. Build sortedArray by iterating backwards (for stability)
        for idx in stride(from: array.count - 1, through: 0, by: -1) {
            let num = array[idx]
            let countIndex = (num / digitColumn) % 10
            countArray[countIndex] -= 1
            let sortedIndex = countArray[countIndex]
            sortedArray[sortedIndex] = num
        }
        
        // 4. Copy back to original array
        for idx in 0..<array.count {
            array[idx] = sortedArray[idx]
        }
    }
}

```

---

## Complexity Analysis

* **Time Complexity: O(d * (n + b))**
    * `d` is the number of digits in the largest number.
    * `n` is the number of elements in the input array.
    * `b` is the base of the numbering system (which is 10 for decimal numbers).
    * **Explanation:** The main `radixSort` function has a loop that runs `d` times. Inside this loop, we call `countingSort`. The `countingSort` function itself has several loops that run `n` times or `b` times, but not nested. Its complexity is O(n + b). Therefore, the total time complexity is `d` passes times the O(n + b) work done in each pass.
    * When `n` is much larger than `b`, this is often simplified to **O(d * n)**. If `d` is also considered a constant (e.g., we're sorting 32-bit integers, where `d` is fixed), the complexity can be seen as linear, **O(n)**.

* **Space Complexity: O(n + b)**
    * This space is required by the `countingSort` helper function. We need a `sortedArray` of size `n` to build the sorted output and a `countArray` of size `b` (in this case, 10) to store the digit counts.

## Insight Notes

#### Real-World Use Cases
* **Sorting Large Datasets of Integers:** Radix sort is highly efficient for sorting data like user IDs, transaction IDs, or any large integer keys, especially when the number of digits is relatively small.
* **Suffix Array Construction:** It's a key component in advanced algorithms for string processing, like building suffix arrays used in full-text search.
* **Sorting Strings:** Radix sort can be adapted to sort strings alphabetically, treating each character as a "digit" in a larger base (e.g., base 26 for lowercase letters).

#### How It Appears in Coding Interviews
* **Non-Comparison Sort Knowledge:** This question is often used to check if you know sorting algorithms beyond the standard comparison-based ones (Merge Sort, Quick Sort). The main point to make is that Radix Sort can **beat the O(n log n) lower bound** for comparison sorts.
* **The Stability Requirement:** A common follow-up question is "Why must the subroutine be stable?". You need to be able to explain that without stability, the work done in previous passes (e.g., sorting by the ones digit) would be undone in subsequent passes, leading to an incorrect result.
* **Implementation Detail:** You are expected to know how to implement the `countingSort` subroutine. The trickiest part is correctly using the cumulative counts to place elements into the new sorted array. The backward loop is key to maintaining stability.

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
