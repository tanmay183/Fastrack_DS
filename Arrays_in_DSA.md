# 📚 Arrays in DSA (Python) — From Basics to Advanced Patterns

> What you get in this file: (1) Array fundamentals + all methods, (2) every important array pattern with templates and "when to use", (3) 15 standard problems (basic → advanced) with line-by-line comments and verified output.

---

## 📑 Table of Contents
1. [What is an Array?](#1-what-is-an-array)
2. [Array in Python (list) — Operations & Methods](#2-array-in-python-list--operations--methods)
3. [Time Complexity Table](#3-time-complexity-table)
4. [Useful Tools for Array Problems](#4-useful-tools-for-array-problems)
5. [How to Attack ANY Array Problem](#5-how-to-attack-any-array-problem)
6. [Array Patterns (Techniques)](#6-array-patterns-techniques)
7. [15 Standard Problems (Easy → Hard)](#7-15-standard-problems-easy--hard)
8. [Pattern Cheat Sheet](#8-pattern-cheat-sheet)
9. [Mistakes & Interview Tips](#9-mistakes--interview-tips)
10. [Practice List](#10-practice-list)

---

## 1. What is an Array?

An **array** is a collection of elements stored in **contiguous memory locations**, accessed by an **index** (starting from 0).

```
Index:    0     1     2     3     4
Value:  [10]  [20]  [30]  [40]  [50]
Address: 1000  1004  1008  1012  1016   (each int = 4 bytes in C-like arrays)
```

**Why is access O(1)?** Because the address is calculated directly:
`address(i) = base_address + i * element_size`. No searching needed.

**Key properties**
- Fixed order, index based, fast random access.
- Insert/delete in the middle is slow (elements must be shifted) → O(n).
- Cache friendly (neighbouring elements are close in memory) → real-world fast.

**Types**
| Type | Meaning | Example |
|---|---|---|
| 1-D array | single row of items | `[1, 2, 3]` |
| 2-D array (matrix) | array of arrays | `[[1,2],[3,4]]` |
| Static array | fixed size (C, Java `int[]`) | — |
| Dynamic array | grows automatically | Python `list`, C++ `vector`, Java `ArrayList` |

**Python note:** Python's `list` is a *dynamic array*. It over-allocates memory, so `append` is **amortized O(1)** (occasionally it resizes and copies everything, but on average it is constant). Lists can hold mixed types, but in DSA we use same-type numbers/strings.

---

## 2. Array in Python (list) — Operations & Methods

### 2.1 Creating, indexing, slicing
```python
arr = [10, 20, 30, 40, 50]   # create a list (our array)
print(arr[0])                # first element, index starts at 0
print(arr[-1])               # negative index counts from the end -> last element
print(arr[1:4])              # slice [start:stop) -> indexes 1,2,3
print(arr[::-1])             # slice with step -1 -> reversed copy
print(arr[::2])              # every 2nd element
print(len(arr))              # number of elements
arr[2] = 99                  # update element at index 2 (O(1))
print(arr)                   # show the updated list
zeros = [0] * 5              # create array of five zeros
print(zeros)
```
**Output**
```
10
50
[20, 30, 40]
[50, 40, 30, 20, 10]
[10, 30, 50]
5
[10, 20, 99, 40, 50]
[0, 0, 0, 0, 0]
```

### 2.2 All important list methods
```python
a = [3, 1, 2]            # start list
a.append(4)              # add ONE element at the end            -> O(1) amortized
a.extend([5, 6])         # add MANY elements at the end          -> O(k)
a.insert(1, 100)         # insert 100 at index 1 (shifts right)  -> O(n)
print(a)                 # [3, 100, 1, 2, 4, 5, 6]
print(a.pop())           # remove & return LAST element          -> O(1)
print(a.pop(0))          # remove & return element at index 0    -> O(n)
a.remove(100)            # remove FIRST occurrence of value 100  -> O(n)
print(a)                 # current list
print(a.index(2))        # index of first occurrence of 2        -> O(n)
print(a.count(1))        # how many times 1 appears              -> O(n)
a.sort()                 # sort in place ascending               -> O(n log n)
print(a)
a.sort(reverse=True)     # sort in place descending
print(a)
a.reverse()              # reverse in place                      -> O(n)
print(a)
b = a.copy()             # shallow copy (new list object)
a.clear()                # remove all elements from a
print(a, b)              # a is empty, b still has data
```
**Output**
```
[3, 100, 1, 2, 4, 5, 6]
6
3
[1, 2, 4, 5]
1
1
[1, 2, 4, 5]
[5, 4, 2, 1]
[1, 2, 4, 5]
[] [1, 2, 4, 5]
```

### 2.3 Built-in functions you will use daily
```python
nums = [5, 2, 9, 1]                     # sample array
print(len(nums), min(nums), max(nums), sum(nums))  # size, smallest, largest, total
print(sorted(nums))                     # returns NEW sorted list (original unchanged)
print(sorted(nums, key=lambda x: -x))   # custom sort key -> descending
for i, v in enumerate(nums):            # get index and value together
    print(i, v, end=" | ")
print()
for x, y in zip([1, 2, 3], ['a', 'b', 'c']):  # walk two lists together
    print(x, y, end=" | ")
print()
squares = [x * x for x in nums]         # list comprehension: build new list in one line
print(squares)
evens = [x for x in range(10) if x % 2 == 0]  # comprehension with a filter
print(evens)
print(any(x > 8 for x in nums), all(x > 0 for x in nums))  # any / all checks
```
**Output**
```
4 1 9 17
[1, 2, 5, 9]
[9, 5, 2, 1]
0 5 | 1 2 | 2 9 | 3 1 | 
1 a | 2 b | 3 c | 
[25, 4, 81, 1]
[0, 2, 4, 6, 8]
True True
```

### 2.4 Two-dimensional arrays (and the classic bug)
```python
rows, cols = 3, 4
good = [[0] * cols for _ in range(rows)]   # CORRECT: each row is a separate list
bad = [[0] * cols] * rows                  # WRONG: same row object repeated 3 times
good[0][0] = 1                             # change only first cell
bad[0][0] = 1                              # changes that cell in EVERY row (same object)
print(good)
print(bad)
matrix = [[1, 2, 3], [4, 5, 6]]
print([list(r) for r in zip(*matrix)])     # transpose using zip(*matrix)
```
**Output**
```
[[1, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0]]
[[1, 0, 0, 0], [1, 0, 0, 0], [1, 0, 0, 0]]
[[1, 4], [2, 5], [3, 6]]
```

### 2.5 Copy vs reference
```python
import copy
a = [1, [2, 3]]
ref = a                    # same object, not a copy
shallow = a.copy()         # new outer list, inner list still shared
deep = copy.deepcopy(a)    # fully independent copy
a[1][0] = 99               # modify inner list
print(ref)                 # changed (same object)
print(shallow)             # changed (inner list shared)
print(deep)                # unchanged (independent)
```
**Output**
```
[1, [99, 3]]
[1, [99, 3]]
[1, [2, 3]]
```

---

## 3. Time Complexity Table

| Operation | Python | Time |
|---|---|---|
| Access `arr[i]` | read/write | O(1) |
| Append at end | `append` | O(1) amortized |
| Pop from end | `pop()` | O(1) |
| Insert / delete at index | `insert`, `pop(i)` | O(n) |
| Search (unsorted) | `x in arr`, `index` | O(n) |
| Search (sorted) | binary search | O(log n) |
| Sort | `sort()` | O(n log n) |
| Slice `arr[a:b]` | creates copy | O(b-a) |
| Reverse | `reverse()` | O(n) |
| Min / Max / Sum | built-ins | O(n) |

> 💡 Need fast pop/push at the **front**? Use `collections.deque` (O(1) both ends).

---

## 4. Useful Tools for Array Problems

```python
from collections import Counter, defaultdict, deque   # frequency map, default dict, double-ended queue
import bisect                                         # binary search on sorted lists
import heapq                                          # min-heap / priority queue

nums = [1, 2, 2, 3, 3, 3]
print(Counter(nums))                     # frequency of each value
d = defaultdict(list)                    # dict that auto-creates empty list for new keys
for i, x in enumerate(nums):
    d[x].append(i)                       # group indexes by value
print(dict(d))
q = deque([1, 2, 3])                     # deque supports fast operations at both ends
q.appendleft(0)                          # add at front O(1)
q.pop()                                  # remove from back O(1)
print(list(q))
sorted_arr = [1, 3, 5, 7]
print(bisect.bisect_left(sorted_arr, 5)) # index where 5 is / should be inserted (left)
print(bisect.bisect_right(sorted_arr, 5))# index just after the last 5
h = [5, 1, 4]
heapq.heapify(h)                         # turn list into min-heap in O(n)
print(heapq.heappop(h))                  # smallest element
```
**Output**
```
Counter({3: 3, 2: 2, 1: 1})
{1: [0], 2: [1, 2], 3: [3, 4, 5]}
[0, 1, 2]
2
3
1
```

---

## 5. How to Attack ANY Array Problem

1. **Read constraints.** `n ≤ 10^5` → need O(n) or O(n log n). `n ≤ 1000` → O(n²) is ok. `n ≤ 20` → backtracking/bitmask.
2. **Write the brute force first** (usually two nested loops, O(n²)). Say it out loud.
3. **Find the waste.** Which work is repeated? Re-computed sums? Re-searched values? Re-checked pairs?
4. **Pick a pattern** that removes the waste (see table).
5. **Dry run** on a tiny example + edge cases (empty, one element, all same, negative numbers, already sorted).
6. State **time and space complexity**.

### Pattern recognition table
| Clue in the problem | Pattern |
|---|---|
| Sorted array, pair/triplet with target | Two Pointers (opposite ends) |
| Remove/move/partition elements in place | Two Pointers (read/write) |
| Contiguous subarray of size k | Fixed Sliding Window |
| Longest/shortest subarray with a condition (positives) | Variable Sliding Window |
| "Sum of subarray = k", negatives allowed | Prefix Sum + HashMap |
| Many range-sum queries | Prefix Sum |
| Many range updates | Difference Array |
| "Find pair / seen before / count frequency" | Hashing |
| Max subarray sum | Kadane |
| Values in range 1..n, find missing/duplicate | Cyclic Sort / Floyd cycle / XOR |
| Cycle in index→value jumps | Fast & Slow Pointers |
| Sorted / rotated sorted / "minimise the max" | Binary Search |
| Next greater/smaller element | Monotonic Stack |
| Max/min of every window | Monotonic Deque |
| Overlapping ranges | Sort + Merge Intervals |
| Only 0/1/2 values | Dutch National Flag |
| Majority element > n/2 | Boyer–Moore Voting |
| Everything appears twice except one | XOR |
| Need left info and right info per index | Prefix + Suffix arrays |

---

## 6. Array Patterns (Techniques)

### Pattern 1 — Two Pointers
**Idea:** Use two indexes to scan the array instead of nested loops. Reduces O(n²) → O(n).

**Variants**
- **Opposite ends** (`left=0`, `right=n-1`, move inward): sorted pair sum, container with most water, palindrome check, 3Sum.
- **Same direction / read-write** (`slow` writes, `fast` reads): remove duplicates, move zeroes, partition.

**Why it works for sorted arrays:** if the sum is too small, only moving `left` right can increase it; if too big, only moving `right` left can decrease it. So you never need to check discarded pairs.

```python
def pair_with_sum(nums, target):           # nums must be SORTED
    left, right = 0, len(nums) - 1         # one pointer at each end
    while left < right:                    # stop when pointers meet
        s = nums[left] + nums[right]       # current pair sum
        if s == target:                    # found the pair
            return (left, right)
        elif s < target:                   # sum too small -> need bigger value
            left += 1                      # move left pointer to a bigger number
        else:                              # sum too big -> need smaller value
            right -= 1                     # move right pointer to a smaller number
    return None                            # no such pair

print(pair_with_sum([1, 2, 3, 4, 6, 8], 10))  # 2 + 8
print(pair_with_sum([1, 2, 3], 100))          # impossible
```
**Output**
```
(1, 5)
None
```
**Complexity:** O(n) time, O(1) space. **Problems:** 2, 4, 9, 12.

---

### Pattern 2 — Sliding Window (Fixed Size)
**Idea:** For every window of size `k`, don't recompute the sum. Slide: add the new element, remove the old one.

```python
def max_sum_k(nums, k):
    window = sum(nums[:k])                 # sum of the first window
    best = window                          # best sum so far
    for i in range(k, len(nums)):          # slide window one step at a time
        window += nums[i] - nums[i - k]    # add entering element, remove leaving element
        best = max(best, window)           # update the answer
    return best

print(max_sum_k([2, 1, 5, 1, 3, 2], 3))    # windows: 8, 7, 9, 6 -> 9
```
**Output**
```
9
```
**Complexity:** O(n) time, O(1) space.

---

### Pattern 3 — Sliding Window (Variable Size)
**Idea:** Expand `right` always; shrink `left` while the window is invalid (or, for "shortest", while it is valid). Each element enters and leaves once → O(n). **Works when all numbers are positive** (so shrinking always reduces the sum). With negatives, use Prefix Sum + HashMap.

**Template**
```
left = 0
for right in range(n):
    add nums[right] to window
    while window is invalid:      # (or "while window is valid" for shortest)
        remove nums[left]; left += 1
    update answer using (right - left + 1)
```
```python
def longest_sum_at_most_k(nums, k):        # positive numbers only
    left = 0                               # window start
    cur = 0                                # current window sum
    best = 0                               # longest valid window length
    for right, x in enumerate(nums):       # expand window by one element each loop
        cur += x                           # include nums[right]
        while cur > k:                     # window invalid -> shrink from left
            cur -= nums[left]              # remove leftmost element
            left += 1                      # move left boundary
        best = max(best, right - left + 1) # window is valid here -> update length
    return best

print(longest_sum_at_most_k([1, 2, 1, 0, 1, 1, 0], 4))
```
**Output**
```
5
```
**Complexity:** O(n) time, O(1) space. **Problem:** 11.

---

### Pattern 4 — Fast & Slow Pointers (Floyd's Cycle Detection)
**Idea:** Two pointers move at different speeds (slow 1 step, fast 2 steps). If a cycle exists they must meet inside it. After meeting, reset one pointer to the start and move both 1 step: they meet at the **cycle entry**.

**Array use:** when `nums[i]` is treated as "next index", a duplicate value creates a cycle (see Problem 14). Also used for: linked list cycle, middle of list, happy number.
**Complexity:** O(n) time, O(1) space (no extra set, no modifying the array).

---

### Pattern 5 — Prefix Sum
**Idea:** `prefix[i]` = sum of the first `i` elements. Then sum of range `[l, r]` = `prefix[r+1] - prefix[l]` in O(1).
**Use:** range-sum queries, equilibrium index, subarray sums, counting subarrays (with HashMap).

```python
nums = [2, 4, 6, 8, 10]
prefix = [0]                               # prefix[0] = 0 (sum of zero elements)
for x in nums:
    prefix.append(prefix[-1] + x)          # each prefix = previous prefix + current element

def range_sum(l, r):                       # inclusive range [l, r]
    return prefix[r + 1] - prefix[l]       # difference of two prefixes

print(prefix)
print(range_sum(1, 3))                     # 4 + 6 + 8
```
**Output**
```
[0, 2, 6, 12, 20, 30]
18
```
**Complexity:** O(n) build, O(1) per query. **Problems:** 10, 13.

---

### Pattern 6 — Difference Array (Range Updates)
**Idea:** Instead of adding `v` to every element in `[l, r]` (O(n) each), do `diff[l] += v` and `diff[r+1] -= v`. A single prefix sum at the end rebuilds the array. Many updates → O(1) each.

```python
n = 6
diff = [0] * (n + 1)                       # extra slot so r+1 never goes out of range

def add(l, r, v):                          # add v to every index in [l, r]
    diff[l] += v                           # start adding from l
    diff[r + 1] -= v                       # stop adding after r

add(1, 3, 5)                               # indexes 1..3 get +5
add(2, 5, 2)                               # indexes 2..5 get +2
res, run = [], 0
for i in range(n):
    run += diff[i]                         # running sum rebuilds actual values
    res.append(run)
print(res)
```
**Output**
```
[0, 5, 7, 7, 2, 2]
```
**Use:** flight bookings, car pooling, range increment queries.

---

### Pattern 7 — Hashing (HashMap / HashSet)
**Idea:** Trade memory for speed. Store what you've seen so lookup is O(1) instead of scanning.
- "Have I seen `target - x` before?" → Two Sum (Problem 3).
- Frequency count → `Counter`.
- Duplicates → `set`.
- Prefix-sum remainder/count → Problem 10.

---

### Pattern 8 — Kadane's Algorithm (Maximum Subarray)
**Idea:** At each element decide: *extend the previous subarray* or *start fresh here*. `cur = max(x, cur + x)`. If the running sum becomes negative it only hurts, so drop it. **Problem:** 7.

---

### Pattern 9 — Dutch National Flag (3-way partition)
**Idea:** Three pointers `low, mid, high` keep regions `[0..low)=0`, `[low..mid)=1`, `(high..end]=2`. One pass, in place. **Problem:** 8.

---

### Pattern 10 — Binary Search on Arrays
**Idea:** Whenever the search space has a **monotonic property** (sorted, rotated sorted, or "answer is yes/no and flips once"), halve it each step → O(log n).

```python
def search_rotated(nums, target):          # sorted array rotated at some pivot, distinct values
    lo, hi = 0, len(nums) - 1              # search range
    while lo <= hi:
        mid = (lo + hi) // 2               # middle index
        if nums[mid] == target:            # found
            return mid
        if nums[lo] <= nums[mid]:          # left half [lo..mid] is sorted
            if nums[lo] <= target < nums[mid]:  # target lies inside the sorted left half
                hi = mid - 1               # go left
            else:
                lo = mid + 1               # otherwise go right
        else:                              # right half [mid..hi] is sorted
            if nums[mid] < target <= nums[hi]:  # target lies inside the sorted right half
                lo = mid + 1               # go right
            else:
                hi = mid - 1               # otherwise go left
    return -1                              # not found

print(search_rotated([4, 5, 6, 7, 0, 1, 2], 0))
print(search_rotated([4, 5, 6, 7, 0, 1, 2], 3))
```
**Output**
```
4
-1
```
**Also:** "Binary search on the answer" (capacity to ship packages, Koko eating bananas, split array largest sum).

---

### Pattern 11 — Monotonic Stack
**Idea:** Keep a stack of indexes whose values are in sorted order. When a new element breaks the order, pop and resolve the popped ones. Each element pushed/popped once → O(n).
**Use:** next greater element, daily temperatures, largest rectangle in histogram, stock span.

```python
def next_greater(nums):
    res = [-1] * len(nums)                 # default: no greater element
    stack = []                             # holds INDEXES, values decreasing from bottom to top
    for i, x in enumerate(nums):
        while stack and nums[stack[-1]] < x:   # x is the next greater for those on stack
            res[stack.pop()] = x           # resolve them
        stack.append(i)                    # push current index to wait for its greater element
    return res

print(next_greater([2, 1, 2, 4, 3]))
```
**Output**
```
[4, 2, 4, -1, -1]
```

---

### Pattern 12 — Monotonic Deque (Sliding Window Maximum)
**Idea:** Deque keeps indexes of useful candidates with decreasing values; the front is always the window maximum.

```python
from collections import deque

def max_sliding_window(nums, k):
    dq = deque()                           # stores indexes, values decreasing
    res = []                               # answers
    for i, x in enumerate(nums):
        while dq and nums[dq[-1]] <= x:    # smaller elements behind x can never be max again
            dq.pop()
        dq.append(i)                       # add current index
        if dq[0] <= i - k:                 # front index slid out of window
            dq.popleft()
        if i >= k - 1:                     # first full window formed
            res.append(nums[dq[0]])        # front is the maximum
    return res

print(max_sliding_window([1, 3, -1, -3, 5, 3, 6, 7], 3))
```
**Output**
```
[3, 3, 5, 5, 6, 7]
```
**Complexity:** O(n) time, O(k) space.

---

### Pattern 13 — Cyclic Sort (Numbers in range 1..n)
**Idea:** Put every value `x` at index `x-1` by swapping. After that, any index with the wrong value reveals the missing/duplicate number. O(n) time, O(1) space.

```python
def first_missing_positive(nums):
    n = len(nums)
    i = 0
    while i < n:
        j = nums[i] - 1                    # correct index for value nums[i]
        if 0 <= j < n and nums[j] != nums[i]:  # value in range and target slot not already correct
            nums[i], nums[j] = nums[j], nums[i]  # place it where it belongs
        else:
            i += 1                         # ignore out of range / duplicates, move on
    for i in range(n):
        if nums[i] != i + 1:               # first slot with wrong value
            return i + 1                   # that number is missing
    return n + 1                           # all 1..n present

print(first_missing_positive([3, 4, -1, 1]))
print(first_missing_positive([1, 2, 0]))
```
**Output**
```
2
3
```

---

### Pattern 14 — Boyer–Moore Voting (Majority Element)
**Idea:** Keep a candidate and a counter. Same value → `+1`, different → `-1`. When counter hits 0 pick a new candidate. A majority element (> n/2) cannot be cancelled out completely.

```python
def majority(nums):
    cand, cnt = None, 0                    # current candidate and its vote count
    for x in nums:
        if cnt == 0:                       # no candidate -> choose current number
            cand = x
        cnt += 1 if x == cand else -1      # vote for / against candidate
    return cand                            # guaranteed majority (if one exists)

print(majority([2, 2, 1, 1, 1, 2, 2]))
```
**Output**
```
2
```
**Complexity:** O(n) time, O(1) space.

---

### Pattern 15 — Sort + Merge Intervals / Greedy
**Idea:** Sorting brings related items next to each other, then one linear sweep solves it.

```python
def merge(intervals):
    intervals.sort(key=lambda x: x[0])     # sort by start time
    res = [intervals[0]]                   # start with the first interval
    for s, e in intervals[1:]:
        if s <= res[-1][1]:                # overlaps with the last merged interval
            res[-1][1] = max(res[-1][1], e)  # extend the end
        else:
            res.append([s, e])             # no overlap -> new interval
    return res

print(merge([[1, 3], [2, 6], [8, 10], [15, 18]]))
```
**Output**
```
[[1, 6], [8, 10], [15, 18]]
```
**Use:** meeting rooms, insert interval, non-overlapping intervals, assign cookies.

---

### Pattern 16 — In-place Reversal / Rotation
**Idea:** Reversing parts of an array rearranges it with O(1) extra space. Rotate right by k = reverse all → reverse first k → reverse the rest. **Problem:** 6.

### Pattern 17 — XOR / Bit Trick
**Idea:** `a ^ a = 0` and `a ^ 0 = a`, XOR is commutative → pairs cancel out.

```python
from functools import reduce

nums = [4, 1, 2, 1, 2]                     # every number appears twice except 4
print(reduce(lambda a, b: a ^ b, nums))    # XOR everything: pairs vanish, single remains
```
**Output**
```
4
```

### Pattern 18 — Prefix + Suffix Arrays
**Idea:** When the answer at index `i` depends on everything left of `i` and everything right of `i`, precompute both sides. **Problems:** 13 (Product Except Self), 15 (Trapping Rain Water).

### Pattern 19 — Matrix (2D array) traversal
Row/column loops, diagonals (`i - j` constant for the same diagonal, `i + j` for anti-diagonal), spiral traversal with four boundaries (`top, bottom, left, right`), and transposing then reversing rows to rotate 90°.

---

## 7. 15 Standard Problems (Easy → Hard)

---

### 🟢 Problem 1 — Second Largest Element (Easy) · *Single pass*
**Idea:** Keep `first` and `second` largest while scanning once. Ignore duplicates of the maximum.

```python
def second_largest(nums):
    first = second = float('-inf')         # both start at "minus infinity"
    for x in nums:                         # single scan of the array
        if x > first:                      # new overall maximum
            second = first                 # old maximum becomes second
            first = x                      # update maximum
        elif first > x > second:           # strictly between -> new second largest
            second = x
    return second if second != float('-inf') else None  # None if no distinct second exists

print(second_largest([12, 35, 1, 10, 34, 1]))
print(second_largest([10, 10, 10]))
```
**Output**
```
34
None
```
**Complexity:** O(n) time, O(1) space.

---

### 🟢 Problem 2 — Move Zeroes (Easy) · *Two pointers (read/write)*
**Idea:** `write` marks where the next non-zero should go. Swap every non-zero to that spot.

```python
def move_zeroes(nums):
    write = 0                              # position for the next non-zero element
    for read in range(len(nums)):          # scan every element
        if nums[read] != 0:                # found a non-zero element
            nums[write], nums[read] = nums[read], nums[write]  # bring it forward
            write += 1                     # advance the write position
    return nums                            # zeros are now at the end, order preserved

print(move_zeroes([0, 1, 0, 3, 12]))
print(move_zeroes([0, 0, 1]))
```
**Output**
```
[1, 3, 12, 0, 0]
[1, 0, 0]
```
**Complexity:** O(n) time, O(1) space.

---

### 🟢 Problem 3 — Two Sum (Easy) · *Hashing*
**Idea:** For each `x`, check whether `target - x` was already seen. Store `value → index`.

```python
def two_sum(nums, target):
    seen = {}                              # value -> index
    for i, x in enumerate(nums):
        need = target - x                  # value required to complete the pair
        if need in seen:                   # we saw it earlier
            return [seen[need], i]         # return both indexes
        seen[x] = i                        # remember current value
    return []                              # no answer

print(two_sum([2, 7, 11, 15], 9))
print(two_sum([3, 2, 4], 6))
```
**Output**
```
[0, 1]
[1, 2]
```
**Complexity:** O(n) time, O(n) space. (Brute force was O(n²).)

---

### 🟢 Problem 4 — Remove Duplicates from Sorted Array (Easy) · *Two pointers (slow/fast)*
**Idea:** `slow` is the last unique element position, `fast` looks for a new value.

```python
def remove_duplicates(nums):
    if not nums:                           # empty array has 0 unique elements
        return 0
    slow = 0                               # index of last unique element
    for fast in range(1, len(nums)):       # scan from second element
        if nums[fast] != nums[slow]:       # found a new value
            slow += 1                      # move to next slot
            nums[slow] = nums[fast]        # write the new value there
    return slow + 1                        # count of unique elements

arr = [0, 0, 1, 1, 1, 2, 2, 3, 3, 4]
k = remove_duplicates(arr)                 # modifies arr in place
print(k, arr[:k])
```
**Output**
```
5 [0, 1, 2, 3, 4]
```
**Complexity:** O(n) time, O(1) space.

---

### 🟢 Problem 5 — Best Time to Buy and Sell Stock (Easy) · *Running minimum*
**Idea:** Profit if selling today = `price - cheapest price so far`. Track the best.

```python
def max_profit(prices):
    min_price = float('inf')               # cheapest price seen so far
    best = 0                               # best profit so far (0 = no trade)
    for p in prices:
        min_price = min(min_price, p)      # update cheapest buy price
        best = max(best, p - min_price)    # profit if we sell today
    return best

print(max_profit([7, 1, 5, 3, 6, 4]))      # buy 1, sell 6
print(max_profit([7, 6, 4, 3, 1]))         # always falling -> 0
```
**Output**
```
5
0
```
**Complexity:** O(n) time, O(1) space.

---

### 🟡 Problem 6 — Rotate Array by K (Medium) · *In-place reversal*
**Idea:** Rotate right by k: reverse whole array, then reverse first k, then reverse remaining n-k.

```python
def rotate(nums, k):
    n = len(nums)
    k %= n                                 # rotating n times gives the same array
    def rev(l, r):                         # helper: reverse nums[l..r] in place
        while l < r:
            nums[l], nums[r] = nums[r], nums[l]  # swap ends
            l += 1                         # move inward
            r -= 1
    rev(0, n - 1)                          # step 1: reverse everything
    rev(0, k - 1)                          # step 2: reverse first k elements
    rev(k, n - 1)                          # step 3: reverse the rest
    return nums

print(rotate([1, 2, 3, 4, 5, 6, 7], 3))
print(rotate([-1, -100, 3, 99], 2))
```
**Output**
```
[5, 6, 7, 1, 2, 3, 4]
[3, 99, -1, -100]
```
**Trace (k=3):** `1234567 → 7654321 → 5674321 → 5671234`.
**Complexity:** O(n) time, O(1) space.

---

### 🟡 Problem 7 — Maximum Subarray (Kadane's Algorithm) (Medium) · *Dynamic idea*
**Idea:** `cur` = best sum of a subarray ending at this index. Either extend (`cur + x`) or restart (`x`).

```python
def max_subarray(nums):
    cur = best = nums[0]                   # start with the first element (handles all-negative arrays)
    for x in nums[1:]:
        cur = max(x, cur + x)              # extend previous subarray or start a new one at x
        best = max(best, cur)              # remember the best seen anywhere
    return best

print(max_subarray([-2, 1, -3, 4, -1, 2, 1, -5, 4]))  # subarray [4,-1,2,1]
print(max_subarray([-3, -1, -2]))                     # best is the single element -1
```
**Output**
```
6
-1
```
**Complexity:** O(n) time, O(1) space.

---

### 🟡 Problem 8 — Sort Colors (Medium) · *Dutch National Flag*
**Idea:** Three pointers. 0 goes to the front, 2 goes to the back, 1 stays in the middle.

```python
def sort_colors(nums):
    low, mid, high = 0, 0, len(nums) - 1   # low: next 0 slot, mid: scanner, high: next 2 slot
    while mid <= high:                     # until scanner crosses the 2-region
        if nums[mid] == 0:
            nums[low], nums[mid] = nums[mid], nums[low]  # send 0 to front region
            low += 1
            mid += 1                       # swapped-in value is a known 1 -> safe to advance
        elif nums[mid] == 1:
            mid += 1                       # 1 already in the middle region
        else:
            nums[mid], nums[high] = nums[high], nums[mid]  # send 2 to back region
            high -= 1                      # do NOT move mid: the swapped value is unchecked
    return nums

print(sort_colors([2, 0, 2, 1, 1, 0]))
print(sort_colors([2, 0, 1]))
```
**Output**
```
[0, 0, 1, 1, 2, 2]
[0, 1, 2]
```
**Complexity:** O(n) time, single pass, O(1) space.

---

### 🟡 Problem 9 — Container With Most Water (Medium) · *Two pointers (opposite)*
**Idea:** Area = `min(h[l], h[r]) * (r - l)`. Start with the widest container; move the **shorter** wall inward (moving the taller one can never help).

```python
def max_area(height):
    left, right = 0, len(height) - 1       # widest possible container
    best = 0
    while left < right:
        width = right - left               # distance between walls
        h = min(height[left], height[right])  # water level limited by the shorter wall
        best = max(best, width * h)        # update the answer
        if height[left] < height[right]:   # left wall is shorter
            left += 1                      # only a taller left wall can improve things
        else:
            right -= 1                     # otherwise move the right wall
    return best

print(max_area([1, 8, 6, 2, 5, 4, 8, 3, 7]))
print(max_area([1, 1]))
```
**Output**
```
49
1
```
**Complexity:** O(n) time, O(1) space.

---

### 🟡 Problem 10 — Subarray Sum Equals K (Medium) · *Prefix sum + HashMap*
**Idea:** If `prefix[j] - prefix[i] = k`, then subarray `(i, j]` sums to k. For each prefix, count how many earlier prefixes equal `prefix - k`. Works with **negatives and zeros**.

```python
from collections import defaultdict

def subarray_sum(nums, k):
    count = 0                              # number of valid subarrays
    prefix = 0                             # running prefix sum
    freq = defaultdict(int)                # how many times each prefix sum has occurred
    freq[0] = 1                            # empty prefix: lets subarrays starting at index 0 count
    for x in nums:
        prefix += x                        # extend prefix sum
        count += freq[prefix - k]          # earlier prefixes that make a subarray sum of k
        freq[prefix] += 1                  # record this prefix
    return count

print(subarray_sum([1, 1, 1], 2))
print(subarray_sum([1, 2, 3], 3))          # [1,2] and [3]
print(subarray_sum([1, -1, 0], 0))         # works with negatives and zeros
```
**Output**
```
2
2
3
```
**Complexity:** O(n) time, O(n) space.

---

### 🟡 Problem 11 — Minimum Size Subarray Sum (Medium) · *Variable sliding window*
**Idea:** Expand until sum ≥ target, then shrink from the left as long as it stays valid; record the smallest length.

```python
def min_subarray_len(target, nums):
    left = 0                               # window start
    cur = 0                                # window sum
    best = float('inf')                    # smallest valid length found
    for right, x in enumerate(nums):       # expand the window
        cur += x
        while cur >= target:               # window is valid -> try to shrink it
            best = min(best, right - left + 1)  # record length
            cur -= nums[left]              # drop leftmost element
            left += 1
    return 0 if best == float('inf') else best  # 0 if no valid window

print(min_subarray_len(7, [2, 3, 1, 2, 4, 3]))   # [4,3]
print(min_subarray_len(4, [1, 4, 4]))
print(min_subarray_len(11, [1, 1, 1, 1]))
```
**Output**
```
2
1
0
```
**Complexity:** O(n) time, O(1) space.

---

### 🔴 Problem 12 — 3Sum (Medium-Hard) · *Sort + two pointers + duplicate skipping*
**Idea:** Sort. Fix one number `a`, then find pairs with sum `-a` using opposite-end two pointers. Skip duplicates so results are unique.

```python
def three_sum(nums):
    nums.sort()                            # sorting enables two pointers + easy duplicate skip
    res = []
    n = len(nums)
    for i in range(n - 2):                 # fix the first number
        if nums[i] > 0:                    # smallest is positive -> no triplet can sum to 0
            break
        if i > 0 and nums[i] == nums[i - 1]:  # skip duplicate first numbers
            continue
        left, right = i + 1, n - 1         # two pointers over the remaining part
        while left < right:
            s = nums[i] + nums[left] + nums[right]  # current triplet sum
            if s == 0:
                res.append([nums[i], nums[left], nums[right]])  # valid triplet
                left += 1
                right -= 1
                while left < right and nums[left] == nums[left - 1]:    # skip duplicate lefts
                    left += 1
                while left < right and nums[right] == nums[right + 1]:  # skip duplicate rights
                    right -= 1
            elif s < 0:
                left += 1                  # need larger sum
            else:
                right -= 1                 # need smaller sum
    return res

print(three_sum([-1, 0, 1, 2, -1, -4]))
print(three_sum([0, 0, 0, 0]))
```
**Output**
```
[[-1, -1, 2], [-1, 0, 1]]
[[0, 0, 0]]
```
**Complexity:** O(n²) time, O(1) extra space (ignoring output). Brute force is O(n³).

---

### 🔴 Problem 13 — Product of Array Except Self (Medium-Hard) · *Prefix + suffix, no division*
**Idea:** `answer[i] = (product of everything left of i) × (product of everything right of i)`. Build left products first, then multiply by running right product.

```python
def product_except_self(nums):
    n = len(nums)
    res = [1] * n                          # res[i] will hold the left product first
    left = 1                               # product of elements to the left
    for i in range(n):
        res[i] = left                      # store product of all elements before i
        left *= nums[i]                    # include nums[i] for the next index
    right = 1                              # product of elements to the right
    for i in range(n - 1, -1, -1):         # walk backwards
        res[i] *= right                    # multiply by product of all elements after i
        right *= nums[i]                   # include nums[i] for the previous index
    return res

print(product_except_self([1, 2, 3, 4]))
print(product_except_self([-1, 1, 0, -3, 3]))
```
**Output**
```
[24, 12, 8, 6]
[0, 0, 9, 0, 0]
```
**Complexity:** O(n) time, O(1) extra space (output array excluded).

---

### 🔴 Problem 14 — Find the Duplicate Number (Hard) · *Fast & slow pointers (Floyd)*
**Idea:** Array has `n+1` numbers in range `1..n`, one duplicate. Treat `i → nums[i]` as a linked list; the duplicate is the **entry point of the cycle**. No extra space, array not modified.

```python
def find_duplicate(nums):
    slow = fast = nums[0]                  # both start at the first "node"
    while True:                            # phase 1: find a meeting point inside the cycle
        slow = nums[slow]                  # slow moves 1 step
        fast = nums[nums[fast]]            # fast moves 2 steps
        if slow == fast:                   # they met inside the cycle
            break
    slow = nums[0]                         # phase 2: restart slow from the beginning
    while slow != fast:                    # move both 1 step at a time
        slow = nums[slow]
        fast = nums[fast]
    return slow                            # they meet at the cycle entry = duplicate number

print(find_duplicate([1, 3, 4, 2, 2]))
print(find_duplicate([3, 1, 3, 4, 2]))
```
**Output**
```
2
3
```
**Complexity:** O(n) time, O(1) space. (Sorting = O(n log n), set = O(n) space.)

---

### 🔴 Problem 15 — Trapping Rain Water (Hard) · *Two pointers with left-max / right-max*
**Idea:** Water above index `i` = `min(maxLeft, maxRight) - height[i]`. Instead of storing two arrays, move the pointer on the **smaller** side: its water level is decided by that side's max, because the other side is at least as tall.

```python
def trap(height):
    left, right = 0, len(height) - 1       # two pointers at the ends
    left_max = right_max = 0               # tallest wall seen from each side
    water = 0
    while left < right:
        if height[left] < height[right]:   # left side is the limiting side
            if height[left] >= left_max:
                left_max = height[left]    # new tallest on the left, no water here
            else:
                water += left_max - height[left]  # water trapped above this bar
            left += 1
        else:                              # right side is the limiting side
            if height[right] >= right_max:
                right_max = height[right]  # new tallest on the right
            else:
                water += right_max - height[right]  # water trapped above this bar
            right -= 1
    return water

print(trap([0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]))
print(trap([4, 2, 0, 3, 2, 5]))
```
**Output**
```
6
9
```
**Complexity:** O(n) time, O(1) space.

---

## 8. Pattern Cheat Sheet

| Problem # | Name | Pattern |
|---|---|---|
| 1 | Second Largest | single pass tracking |
| 2 | Move Zeroes | two pointers (read/write) |
| 3 | Two Sum | hashing |
| 4 | Remove Duplicates | two pointers (slow/fast) |
| 5 | Stock Buy/Sell | running minimum |
| 6 | Rotate Array | in-place reversal |
| 7 | Maximum Subarray | Kadane |
| 8 | Sort Colors | Dutch National Flag |
| 9 | Container With Most Water | two pointers (opposite) |
| 10 | Subarray Sum = K | prefix sum + hashmap |
| 11 | Min Size Subarray Sum | variable sliding window |
| 12 | 3Sum | sort + two pointers |
| 13 | Product Except Self | prefix / suffix |
| 14 | Find Duplicate | fast & slow pointers |
| 15 | Trapping Rain Water | two pointers + max tracking |

**Template to remember**
```python
# Two pointers (opposite)      # Sliding window (variable)         # Prefix + HashMap
l, r = 0, n-1                  # left = 0                          # pre = 0; seen = {0: 1}
while l < r:                   # for right in range(n):            # for x in nums:
    ...                        #     add nums[right]               #     pre += x
                               #     while invalid: shrink left    #     ans += seen.get(pre-k, 0)
                               #     update answer                 #     seen[pre] = seen.get(pre,0)+1
```

---

## 9. Mistakes & Interview Tips
1. **Off-by-one errors:** `range(n)` vs `range(n-1)`, `<` vs `<=` in binary search, `r+1` in prefix sums.
2. **Modifying a list while looping over it** (use a second pointer or build a new list).
3. **`[[0]*m]*n` bug** in 2-D arrays (use a comprehension).
4. **Sliding window with negatives** breaks the shrink logic → use prefix sum + hashmap.
5. **`list.pop(0)` / `insert(0, x)` inside loops** is O(n²) overall → use `deque`.
6. **Slicing inside loops** (`nums[i:]`) copies the list each time → hidden O(n²).
7. **Empty array / single element / all negatives / duplicates** — always test these.
8. Say complexity out loud and mention the brute force → optimization story. Interviewers love it.
9. When the array is sorted (or you may sort it), think **two pointers / binary search** immediately.

---

## 10. Practice List (after the 15 above)
| Level | Problems |
|---|---|
| Easy | Contains Duplicate (217), Single Number (136), Majority Element (169), Merge Sorted Array (88), Missing Number (268), Plus One (66) |
| Medium | Top K Frequent Elements (347), Group Anagrams (49), Longest Consecutive Sequence (128), Next Permutation (31), Set Matrix Zeroes (73), Spiral Matrix (54), Rotate Image (48), Merge Intervals (56), Search in Rotated Sorted Array (33), Find Min in Rotated Array (153), Daily Temperatures (739), 4Sum (18), Maximum Product Subarray (152) |
| Hard | Trapping Rain Water (42), Sliding Window Maximum (239), First Missing Positive (41), Largest Rectangle in Histogram (84), Median of Two Sorted Arrays (4), Count of Smaller Numbers After Self (315), Subarrays with K Different Integers (992) |

### ✅ Final advice for 12 November 2026
- Re-type Problems 3, 7, 9, 10, 11, 12, 15 from memory — they cover most array interview patterns.
- Every day: 1 easy + 1 medium, and write down **which pattern** solved it.
- Next topic: tell me **Strings**, **Hashing**, **Recursion**, **Sorting & Searching** or any other, and I'll prepare it in the same format. 🚀
