# 🔍 Searching Techniques in DSA (Python) — Theory, Code & 15 Problems

> Covers: all commonly used searching algorithms (theory + commented code + output), binary-search templates, "binary search on answer", and 15 standard problems from easy to hard.

---

## 📑 Table of Contents
1. [What is Searching?](#1-what-is-searching)
2. [Linear Search (+ Sentinel Search)](#2-linear-search--sentinel-search)
3. [Binary Search (Iterative & Recursive)](#3-binary-search-iterative--recursive)
4. [Lower Bound / Upper Bound (First & Last Occurrence)](#4-lower-bound--upper-bound)
5. [Jump Search](#5-jump-search)
6. [Interpolation Search](#6-interpolation-search)
7. [Exponential Search](#7-exponential-search)
8. [Ternary Search](#8-ternary-search)
9. [Binary Search on Answer](#9-binary-search-on-answer-the-most-important-advanced-idea)
10. [Hash-based Search](#10-hash-based-search)
11. [Other Searches (BFS / DFS / Fibonacci)](#11-other-searches-you-should-know-by-name)
12. [Comparison Table](#12-comparison-table)
13. [The 3 Binary Search Templates](#13-the-3-binary-search-templates-memorise)
14. [15 Standard Problems](#14-15-standard-problems-easy--hard)
15. [Mistakes, Tips & Practice List](#15-mistakes-tips--practice-list)

---

## 1. What is Searching?

**Searching** = finding the position (or existence) of a target in a collection.

Two big families:
| Family | Needs sorted data? | Examples |
|---|---|---|
| **Sequential** | No | Linear Search, Sentinel Search |
| **Interval / divide-and-conquer** | Yes (or monotonic property) | Binary, Jump, Interpolation, Exponential, Ternary |
| **Direct access** | No (uses extra memory) | Hash Table / Set / Dict |

**Golden rule:** If the data is sorted or has a *yes/no flip point* (monotonic property), think **binary search**. If it is unsorted and queried many times, think **hashing**. Otherwise linear scan.

---

## 2. Linear Search (+ Sentinel Search)

### Theory
Check every element one by one from the start until the target is found or the array ends.

**Steps**
1. Start at index 0.
2. Compare `arr[i]` with target.
3. If equal → return `i`.
4. If the end is reached → return `-1`.

**Complexity:** Best O(1) (target at first place), Worst/Average O(n), Space O(1).
**When to use:** unsorted data, small arrays, or you search only once.

```python
def linear_search(arr, target):
    for i in range(len(arr)):          # visit every index from 0 to n-1
        if arr[i] == target:           # compare current element with the target
            return i                   # found -> return its index immediately
    return -1                          # loop finished without a match -> not present

print(linear_search([5, 3, 8, 1, 9], 8))   # target exists
print(linear_search([5, 3, 8, 1, 9], 7))   # target missing
```
**Output**
```
2
-1
```

### Sentinel Linear Search (small optimisation)
**Idea:** Put the target at the **end** of the array as a guard ("sentinel"), so the loop never needs the `i < n` check. This removes one comparison per iteration. Restore the last element afterwards.

```python
def sentinel_search(arr, target):
    n = len(arr)                       # remember original size
    last = arr[-1]                     # save the last element (we will overwrite it)
    arr[-1] = target                   # place sentinel: the loop is guaranteed to stop
    i = 0
    while arr[i] != target:            # no bounds check needed, sentinel stops us
        i += 1
    arr[-1] = last                     # restore the original last element
    if i < n - 1 or last == target:    # real match found (not just the sentinel)
        return i
    return -1                          # we only hit the sentinel -> not found

print(sentinel_search([5, 3, 8, 1, 9], 1))
print(sentinel_search([5, 3, 8, 1, 9], 9))
print(sentinel_search([5, 3, 8, 1, 9], 4))
```
**Output**
```
3
4
-1
```

---

## 3. Binary Search (Iterative & Recursive)

### Theory
**Precondition:** array must be **sorted**.
**Idea:** Compare the target with the **middle** element. Because the array is sorted, you can throw away half of the array every step.

**Steps**
1. `lo = 0`, `hi = n-1`.
2. `mid = lo + (hi - lo)//2` (this form avoids integer overflow in C++/Java).
3. If `arr[mid] == target` → found.
4. If `arr[mid] < target` → target is in right half → `lo = mid + 1`.
5. Else → target in left half → `hi = mid - 1`.
6. Repeat while `lo <= hi`. If the range becomes empty → not found.

**Dry run:** `arr = [2, 5, 8, 12, 16, 23, 38]`, target = 23
| Step | lo | hi | mid | arr[mid] | Decision |
|---|---|---|---|---|---|
| 1 | 0 | 6 | 3 | 12 | 12 < 23 → lo = 4 |
| 2 | 4 | 6 | 5 | 23 | found at index 5 |

**Complexity:** Time O(log n) (1 billion elements → ~30 steps), Space O(1) iterative / O(log n) recursive.

```python
def binary_search(arr, target):
    lo, hi = 0, len(arr) - 1           # search range is the whole array
    while lo <= hi:                    # range is non-empty while lo <= hi
        mid = lo + (hi - lo) // 2      # middle index (overflow-safe form)
        if arr[mid] == target:         # target found
            return mid
        elif arr[mid] < target:        # middle is too small -> discard left half
            lo = mid + 1
        else:                          # middle is too big -> discard right half
            hi = mid - 1
    return -1                          # range empty -> target not present

def binary_search_recursive(arr, target, lo, hi):
    if lo > hi:                        # base case: empty range
        return -1
    mid = lo + (hi - lo) // 2          # middle index
    if arr[mid] == target:             # found
        return mid
    if arr[mid] < target:              # search right half recursively
        return binary_search_recursive(arr, target, mid + 1, hi)
    return binary_search_recursive(arr, target, lo, mid - 1)  # search left half

data = [2, 5, 8, 12, 16, 23, 38]
print(binary_search(data, 23))                                  # present
print(binary_search(data, 7))                                   # absent
print(binary_search_recursive(data, 38, 0, len(data) - 1))      # recursive version
```
**Output**
```
5
-1
6
```

---

## 4. Lower Bound / Upper Bound

### Theory
Plain binary search returns *any* matching index. With **duplicates** you often need:
- **lower_bound(x):** first index `i` with `arr[i] >= x` (first occurrence / insert position).
- **upper_bound(x):** first index `i` with `arr[i] > x` (one past the last occurrence).

Then: first occurrence = `lower_bound(x)` (if `arr[idx] == x`), last occurrence = `upper_bound(x) - 1`, **count of x = upper_bound(x) - lower_bound(x)**.

**Key trick:** use `hi = n` (half-open range `[lo, hi)`), loop `while lo < hi`, and set `hi = mid` (not `mid - 1`) when the condition holds. This pattern is the most reusable binary search template.

```python
import bisect                            # Python's built-in binary search module

def lower_bound(arr, x):                 # first index with arr[i] >= x
    lo, hi = 0, len(arr)                 # hi = n so that "insert at the end" is possible
    while lo < hi:                       # half-open range [lo, hi)
        mid = (lo + hi) // 2
        if arr[mid] < x:                 # mid is too small -> answer lies to the right
            lo = mid + 1
        else:                            # arr[mid] >= x -> mid could be the answer, keep it
            hi = mid
    return lo                            # lo == hi == first index satisfying arr[i] >= x

def upper_bound(arr, x):                 # first index with arr[i] > x
    lo, hi = 0, len(arr)
    while lo < hi:
        mid = (lo + hi) // 2
        if arr[mid] <= x:                # equal values also go to the left side
            lo = mid + 1
        else:                            # arr[mid] > x -> candidate answer
            hi = mid
    return lo

arr = [1, 2, 2, 2, 3, 5]
print(lower_bound(arr, 2))                       # first 2 is at index 1
print(upper_bound(arr, 2))                       # index after last 2
print(upper_bound(arr, 2) - lower_bound(arr, 2)) # how many times 2 occurs
print(lower_bound(arr, 4))                       # 4 is absent: position where it would be inserted
print(bisect.bisect_left(arr, 2), bisect.bisect_right(arr, 2))  # built-ins give the same results
```
**Output**
```
1
4
3
5
1 4
```
**Python shortcut:** `bisect_left` = lower_bound, `bisect_right` = upper_bound, `bisect.insort(arr, x)` inserts while keeping order.

---

## 5. Jump Search

### Theory
**Precondition:** sorted array.
**Idea:** Jump ahead in fixed blocks of size `√n` until you pass the target, then do a **linear search inside that one block**. Good when going *backwards* is costly (e.g. sequential storage) because it never moves back more than one block.

**Steps**
1. `step = √n`.
2. While the last element of the current block is `< target`, jump to the next block.
3. Linear search inside the block.

**Complexity:** Time O(√n), Space O(1). (Between linear O(n) and binary O(log n).)

```python
import math

def jump_search(arr, target):
    n = len(arr)
    step = math.isqrt(n)                          # optimal block size is sqrt(n)
    prev = 0                                      # start of the current block
    while prev < n and arr[min(prev + step, n) - 1] < target:  # last item of block still smaller?
        prev += step                              # jump to the next block
    for i in range(prev, min(prev + step, n)):    # linear scan only inside the candidate block
        if arr[i] == target:
            return i
    return -1                                     # not found

data = [1, 3, 5, 7, 9, 11, 13, 15, 17, 19, 21, 23]
print(jump_search(data, 15))
print(jump_search(data, 4))
```
**Output**
```
7
-1
```

---

## 6. Interpolation Search

### Theory
**Precondition:** sorted array with **uniformly distributed** values (like searching a name in a phone book: you open "S" near the end, not in the middle).
**Idea:** Instead of always the middle, *estimate* the position using a straight-line formula:

`pos = lo + (target - arr[lo]) * (hi - lo) / (arr[hi] - arr[lo])`

**Complexity:** Average O(log log n) for uniform data, **worst O(n)** for skewed data (e.g. 1, 2, 4, 8, 1000000...). Space O(1).

```python
def interpolation_search(arr, target):
    lo, hi = 0, len(arr) - 1
    while lo <= hi and arr[lo] <= target <= arr[hi]:   # target must lie inside the value range
        if arr[lo] == arr[hi]:                         # avoid division by zero (all same values)
            return lo if arr[lo] == target else -1
        pos = lo + (target - arr[lo]) * (hi - lo) // (arr[hi] - arr[lo])  # estimated index
        if arr[pos] == target:                         # guessed right
            return pos
        elif arr[pos] < target:                        # estimate too small -> search right part
            lo = pos + 1
        else:                                          # estimate too big -> search left part
            hi = pos - 1
    return -1

data = [10, 20, 30, 40, 50, 60, 70, 80, 90]
print(interpolation_search(data, 70))
print(interpolation_search(data, 35))
```
**Output**
```
6
-1
```

---

## 7. Exponential Search

### Theory
**Precondition:** sorted array. Best for **unbounded / infinite** sorted arrays or when the target is likely near the start.
**Idea:** Find a range by doubling the index (1, 2, 4, 8, ...) until `arr[bound] >= target`, then run binary search inside `[bound/2, bound]`.

**Complexity:** O(log i) where `i` is the position of the target. Space O(1).

```python
def exponential_search(arr, target):
    if not arr:                                    # empty array
        return -1
    if arr[0] == target:                           # check the first element directly
        return 0
    bound = 1
    while bound < len(arr) and arr[bound] < target:  # keep doubling until we pass the target
        bound *= 2
    lo = bound // 2                                # target lies between bound/2 ...
    hi = min(bound, len(arr) - 1)                  # ... and bound
    while lo <= hi:                                # ordinary binary search on that range
        mid = (lo + hi) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1

data = [2, 3, 4, 10, 40, 50, 60, 70, 80, 90, 100]
print(exponential_search(data, 10))
print(exponential_search(data, 100))
print(exponential_search(data, 11))
```
**Output**
```
3
10
-1
```

---

## 8. Ternary Search

### Theory
**Use:** find the **maximum (or minimum) of a unimodal function/array** (strictly increasing then strictly decreasing, a "mountain"). Binary search cannot handle it directly with one comparison, so use two midpoints `m1` and `m2`.

**Steps:** if `f(m1) < f(m2)` the peak is right of `m1`; otherwise it is left of `m2`. Remove one third each step.
**Complexity:** O(log₃ n). (For arrays, comparing `arr[mid]` and `arr[mid+1]` with plain binary search is simpler, see Problem 5, but ternary is essential for continuous functions.)

```python
def ternary_peak(arr):                      # arr increases then decreases (distinct values)
    lo, hi = 0, len(arr) - 1
    while hi - lo > 2:                      # stop when only up to 3 candidates remain
        m1 = lo + (hi - lo) // 3            # first third point
        m2 = hi - (hi - lo) // 3            # second third point
        if arr[m1] < arr[m2]:               # still climbing between m1 and m2
            lo = m1 + 1                     # peak is to the right of m1
        else:                               # already falling at m2 side
            hi = m2 - 1                     # peak is to the left of m2
    return max(range(lo, hi + 1), key=lambda i: arr[i])  # check the few remaining indexes

mountain = [1, 3, 7, 12, 9, 4, 2]
idx = ternary_peak(mountain)
print(idx, mountain[idx])                   # index and value of the peak
```
**Output**
```
3 12
```

---

## 9. Binary Search on Answer (the most important advanced idea)

### Theory
Sometimes there is **no sorted array**, but the *answer* lies in a numeric range `[lo, hi]` and a function `feasible(x)` is **monotonic**:

```
x:         1  2  3  4  5  6  7  8
feasible:  F  F  F  F  T  T  T  T      -> find the FIRST True  (minimise answer)
feasible:  T  T  T  T  F  F  F  F      -> find the LAST  True  (maximise answer)
```

**How to spot it:** "minimise the maximum…", "maximise the minimum…", "smallest capacity/speed/time such that…", "largest distance such that…".

**Recipe**
1. Decide the search range (`lo` = smallest possible answer, `hi` = largest possible).
2. Write `feasible(mid)` — usually a greedy O(n) check.
3. Binary search for the boundary.
**Complexity:** O(n × log(range)).

```python
def smallest_true(lo, hi, feasible):        # smallest x in [lo, hi] with feasible(x) == True
    while lo < hi:
        mid = (lo + hi) // 2                # candidate answer
        if feasible(mid):                   # mid works -> maybe something smaller works too
            hi = mid                        # keep mid, shrink the right side
        else:                               # mid fails -> everything smaller also fails
            lo = mid + 1
    return lo

# Example: smallest integer x such that x*x >= 50
print(smallest_true(1, 100, lambda x: x * x >= 50))
```
**Output**
```
8
```

---

## 10. Hash-based Search

### Theory
A **hash table** (Python `dict` / `set`) maps a key to a bucket using a hash function, giving **average O(1)** lookup, insert and delete (worst O(n) with heavy collisions). It does **not need sorting** but uses extra O(n) memory and does not give ordering.

**Use when:** repeated membership queries on unsorted data, two-sum style problems, counting frequencies.

```python
arr = [40, 10, 30, 20, 50]                   # unsorted data
lookup_set = set(arr)                        # build a hash set once: O(n)
print(30 in lookup_set)                      # O(1) average membership test
print(35 in lookup_set)
index_of = {v: i for i, v in enumerate(arr)} # dict: value -> index
print(index_of.get(20, -1))                  # index of 20 (or -1 if missing)
print(index_of.get(99, -1))
```
**Output**
```
True
False
3
-1
```

---

## 11. Other Searches You Should Know By Name
| Search | One-line idea | Used for |
|---|---|---|
| **BFS (Breadth-First Search)** | Explore level by level using a queue | Shortest path in unweighted graph, level order |
| **DFS (Depth-First Search)** | Go deep first using recursion/stack | Connected components, backtracking, tree search |
| **Fibonacci Search** | Like binary search but splits using Fibonacci numbers (only +/- operations) | Sorted arrays when division is costly |
| **Binary Search Tree search** | Go left if smaller, right if larger | Ordered dynamic data O(h) |
| **String search (KMP, Rabin–Karp)** | Find a pattern in text | Covered in the Strings section |

---

## 12. Comparison Table

| Algorithm | Needs sorted? | Best | Average | Worst | Space |
|---|---|---|---|---|---|
| Linear | No | O(1) | O(n) | O(n) | O(1) |
| Binary | **Yes** | O(1) | O(log n) | O(log n) | O(1) |
| Jump | Yes | O(1) | O(√n) | O(√n) | O(1) |
| Interpolation | Yes (uniform) | O(1) | O(log log n) | O(n) | O(1) |
| Exponential | Yes | O(1) | O(log i) | O(log i) | O(1) |
| Ternary | Unimodal | O(1) | O(log₃ n) | O(log₃ n) | O(1) |
| Hash lookup | No | O(1) | O(1) | O(n) | O(n) |

---

## 13. The 3 Binary Search Templates (Memorise)

```py
# TEMPLATE 1 — exact match (closed range [lo, hi])
lo, hi = 0, n - 1
while lo <= hi:
    mid = (lo + hi) // 2
    if a[mid] == t: return mid
    elif a[mid] < t: lo = mid + 1
    else: hi = mid - 1
return -1

# TEMPLATE 2 — first True (minimise)    half-open [lo, hi)  or  [lo, hi]
while lo < hi:
    mid = (lo + hi) // 2
    if ok(mid): hi = mid          # mid may be the answer
    else:       lo = mid + 1
return lo

# TEMPLATE 3 — last True (maximise)     use the upper middle to avoid infinite loop
while lo < hi:
    mid = (lo + hi + 1) // 2
    if ok(mid): lo = mid          # mid may be the answer
    else:       hi = mid - 1
return lo
```
> ⚠️ In Template 3, if you used `(lo + hi) // 2` with `lo = mid`, the loop can run forever when `hi = lo + 1`. That's why we use `+1`.

---

## 14. 15 Standard Problems (Easy → Hard)

---

### 🟢 Problem 1 — Binary Search with Step Trace (Easy)
**Idea:** Classic binary search; we also print each step so you can *see* the halving.

```python
def binary_search_trace(nums, target):
    lo, hi = 0, len(nums) - 1                  # full range
    step = 1                                   # step counter for printing
    while lo <= hi:
        mid = lo + (hi - lo) // 2              # middle index
        print(f"step {step}: lo={lo}, hi={hi}, mid={mid}, nums[mid]={nums[mid]}")  # show state
        if nums[mid] == target:                # found
            return mid
        elif nums[mid] < target:               # go right
            lo = mid + 1
        else:                                  # go left
            hi = mid - 1
        step += 1
    return -1

print(binary_search_trace([-1, 0, 3, 5, 9, 12], 9))
print(binary_search_trace([-1, 0, 3, 5, 9, 12], 2))
```
**Output**
```
step 1: lo=0, hi=5, mid=2, nums[mid]=3
step 2: lo=3, hi=5, mid=4, nums[mid]=9
4
step 1: lo=0, hi=5, mid=2, nums[mid]=3
step 2: lo=0, hi=1, mid=0, nums[mid]=-1
step 3: lo=1, hi=1, mid=1, nums[mid]=0
-1
```
**Complexity:** O(log n) time, O(1) space.

---

### 🟢 Problem 2 — First and Last Position of Element in Sorted Array (Easy-Medium)
**Idea:** `first = lower_bound(x)`, `last = upper_bound(x) - 1`. Verify that `x` actually exists.

```python
def search_range(nums, target):
    def lower_bound(x):                        # first index with nums[i] >= x
        lo, hi = 0, len(nums)
        while lo < hi:
            mid = (lo + hi) // 2
            if nums[mid] < x:
                lo = mid + 1
            else:
                hi = mid
        return lo
    first = lower_bound(target)                # candidate first index
    if first == len(nums) or nums[first] != target:   # target not present
        return [-1, -1]
    last = lower_bound(target + 1) - 1         # first index of (target+1) minus 1 = last index of target
    return [first, last]

print(search_range([5, 7, 7, 8, 8, 10], 8))
print(search_range([5, 7, 7, 8, 8, 10], 6))
print(search_range([], 0))
```
**Output**
```
[3, 4]
[-1, -1]
[-1, -1]
```
**Complexity:** O(log n) time, O(1) space.

---

### 🟢 Problem 3 — Search Insert Position (Easy)
**Idea:** That is exactly `lower_bound`: first index with `nums[i] >= target`.

```python
def search_insert(nums, target):
    lo, hi = 0, len(nums)                      # hi = n allows inserting at the very end
    while lo < hi:
        mid = (lo + hi) // 2
        if nums[mid] < target:                 # target must go to the right of mid
            lo = mid + 1
        else:                                  # mid is a valid spot, try left
            hi = mid
    return lo                                  # index where target is or should be inserted

print(search_insert([1, 3, 5, 6], 5))          # exists
print(search_insert([1, 3, 5, 6], 2))          # between 1 and 3
print(search_insert([1, 3, 5, 6], 7))          # after the end
print(search_insert([1, 3, 5, 6], 0))          # before the start
```
**Output**
```
2
1
4
0
```
**Complexity:** O(log n).

---

### 🟡 Problem 4 — Integer Square Root (Medium) · *Binary search on answer*
**Idea:** Find the largest `m` with `m*m <= x`. The condition `m*m <= x` is True for small m and False for large m (monotonic), so binary search the answer.

```python
def my_sqrt(x):
    lo, hi = 0, x                              # answer lies between 0 and x
    ans = 0                                    # best valid answer so far
    while lo <= hi:
        mid = (lo + hi) // 2
        if mid * mid <= x:                     # mid is valid (not too big)
            ans = mid                          # remember it
            lo = mid + 1                       # try a bigger value
        else:                                  # mid too big
            hi = mid - 1
    return ans

print(my_sqrt(8))                              # floor(2.828) = 2
print(my_sqrt(16))
print(my_sqrt(0))
print(my_sqrt(2147395599))                     # large input, still ~31 iterations
```
**Output**
```
2
4
0
46339
```
**Complexity:** O(log x).

---

### 🟡 Problem 5 — Find Peak Element (Medium) · *Binary search without sorted data*
**Idea:** A peak is greater than both neighbours. If `nums[mid] < nums[mid+1]`, we are on an upward slope → a peak exists on the right. Otherwise a peak exists at `mid` or on the left. (Works because `nums[-1]` and `nums[n]` are treated as -∞.)

```python
def find_peak(nums):
    lo, hi = 0, len(nums) - 1
    while lo < hi:                             # stop when one candidate remains
        mid = (lo + hi) // 2
        if nums[mid] < nums[mid + 1]:          # rising slope -> peak is on the right
            lo = mid + 1
        else:                                  # falling slope -> peak is at mid or left
            hi = mid
    return lo                                  # index of a peak

print(find_peak([1, 2, 3, 1]))
print(find_peak([1, 2, 1, 3, 5, 6, 4]))        # two peaks (idx 1 and 5), either is valid
```
**Output**
```
2
5
```
**Complexity:** O(log n) time, O(1) space.

---

### 🟡 Problem 6 — Search in Rotated Sorted Array (Medium) · *Find pivot, then search*
**Idea:** Step 1: find the index of the minimum (the rotation point). Step 2: decide which sorted half can contain the target and run normal binary search there.

```python
def search_rotated(nums, target):
    n = len(nums)
    lo, hi = 0, n - 1
    while lo < hi:                             # step 1: find index of the smallest element
        mid = (lo + hi) // 2
        if nums[mid] > nums[hi]:               # minimum is in the right part
            lo = mid + 1
        else:                                  # minimum is at mid or left
            hi = mid
    pivot = lo                                 # index of the smallest element
    if nums[pivot] <= target <= nums[n - 1]:   # step 2: target is in the right sorted part
        lo, hi = pivot, n - 1
    else:                                      # otherwise in the left sorted part
        lo, hi = 0, pivot - 1
    while lo <= hi:                            # normal binary search in the chosen part
        mid = (lo + hi) // 2
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1

print(search_rotated([4, 5, 6, 7, 0, 1, 2], 0))
print(search_rotated([4, 5, 6, 7, 0, 1, 2], 3))
print(search_rotated([1], 1))
```
**Output**
```
4
-1
0
```
**Complexity:** O(log n) time, O(1) space.

---

### 🟡 Problem 7 — Find Minimum in Rotated Sorted Array (Medium)
**Idea:** Compare `nums[mid]` with `nums[hi]`. If mid is bigger, the drop (minimum) is to the right; otherwise the minimum is at mid or to the left.

```python
def find_min(nums):
    lo, hi = 0, len(nums) - 1
    while lo < hi:
        mid = (lo + hi) // 2
        if nums[mid] > nums[hi]:               # mid is in the left (larger) part -> min is right of mid
            lo = mid + 1
        else:                                  # mid is in the right part -> min is mid or left of it
            hi = mid
    return nums[lo]

print(find_min([3, 4, 5, 1, 2]))
print(find_min([4, 5, 6, 7, 0, 1, 2]))
print(find_min([11, 13, 15, 17]))              # not rotated
```
**Output**
```
1
0
11
```
**Complexity:** O(log n).

---

### 🟡 Problem 8 — Search a 2D Matrix (Medium) · *Flatten with index math*
**Idea:** Each row is sorted and each row starts after the previous row ends → the matrix behaves like one sorted 1-D array of size `m*n`. Convert `mid` → `(mid // n, mid % n)`.

```python
def search_matrix(matrix, target):
    m, n = len(matrix), len(matrix[0])         # rows and columns
    lo, hi = 0, m * n - 1                      # treat matrix as flat array
    while lo <= hi:
        mid = (lo + hi) // 2
        val = matrix[mid // n][mid % n]        # convert flat index to (row, col)
        if val == target:
            return True
        elif val < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return False

grid = [[1, 3, 5, 7], [10, 11, 16, 20], [23, 30, 34, 60]]
print(search_matrix(grid, 3))
print(search_matrix(grid, 13))
```
**Output**
```
True
False
```
**Complexity:** O(log(m·n)).

---

### 🟡 Problem 9 — Search a 2D Matrix II (Medium) · *Staircase search*
**Idea:** Rows sorted left→right, columns sorted top→bottom (but rows are *not* chained). Start at the **top-right** corner: if value > target move left (eliminates a column); if value < target move down (eliminates a row).

```python
def search_matrix_ii(matrix, target):
    row, col = 0, len(matrix[0]) - 1           # start at top-right corner
    while row < len(matrix) and col >= 0:      # stay inside the matrix
        val = matrix[row][col]
        if val == target:
            return True
        elif val > target:                     # too big: whole column below is also too big -> drop column
            col -= 1
        else:                                  # too small: whole row to the left is too small -> drop row
            row += 1
    return False

grid = [[1, 4, 7, 11, 15], [2, 5, 8, 12, 19], [3, 6, 9, 16, 22],
        [10, 13, 14, 17, 24], [18, 21, 23, 26, 30]]
print(search_matrix_ii(grid, 5))
print(search_matrix_ii(grid, 20))
```
**Output**
```
True
False
```
**Complexity:** O(m + n) time, O(1) space.

---

### 🔴 Problem 10 — Koko Eating Bananas (Medium-Hard) · *Binary search on answer*
**Idea:** Find the **minimum eating speed** `k` such that all piles finish within `h` hours. Time at speed k = `Σ ceil(pile / k)`. Larger speed → fewer hours (monotonic).

```python
def min_eating_speed(piles, h):
    def hours_needed(k):                       # total hours at speed k
        return sum((p + k - 1) // k for p in piles)   # (p + k - 1)//k is ceil(p/k)
    lo, hi = 1, max(piles)                     # slowest = 1 per hour, fastest needed = biggest pile
    while lo < hi:
        mid = (lo + hi) // 2
        if hours_needed(mid) <= h:             # speed mid is fast enough -> try slower
            hi = mid
        else:                                  # too slow -> need faster
            lo = mid + 1
    return lo

print(min_eating_speed([3, 6, 7, 11], 8))
print(min_eating_speed([30, 11, 23, 4, 20], 5))
print(min_eating_speed([30, 11, 23, 4, 20], 6))
```
**Output**
```
4
30
23
```
**Complexity:** O(n log(max pile)).

---

### 🔴 Problem 11 — Capacity to Ship Packages Within D Days (Medium-Hard) · *Minimise the maximum*
**Idea:** Packages must go in order. Find the minimum ship capacity. Capacity must be ≥ heaviest package and ≤ total weight. Greedy check: load packages until capacity is exceeded, then start a new day.

```python
def ship_within_days(weights, days):
    def days_needed(cap):                      # greedy simulation with capacity cap
        d, load = 1, 0                         # start day 1 with an empty ship
        for w in weights:
            if load + w > cap:                 # this package doesn't fit -> next day
                d += 1
                load = 0
            load += w                          # load the package
        return d
    lo, hi = max(weights), sum(weights)        # smallest possible and largest possible capacity
    while lo < hi:
        mid = (lo + hi) // 2
        if days_needed(mid) <= days:           # capacity mid is enough -> try smaller
            hi = mid
        else:                                  # not enough -> need larger capacity
            lo = mid + 1
    return lo

print(ship_within_days([1, 2, 3, 4, 5, 6, 7, 8, 9, 10], 5))
print(ship_within_days([3, 2, 2, 4, 1, 4], 3))
print(ship_within_days([1, 2, 3, 1, 1], 4))
```
**Output**
```
15
6
3
```
**Complexity:** O(n log(sum)).

---

### 🔴 Problem 12 — Aggressive Cows (Hard) · *Maximise the minimum*
**Idea:** Place `c` cows in sorted stalls so the **minimum distance between any two cows is as large as possible**. For a distance `d`, greedily place cows (first stall, then next stall at least `d` away). Feasible for small d, fails for large d → find the **last True**.

```python
def aggressive_cows(stalls, cows):
    stalls.sort()                              # positions must be sorted for greedy placement
    def can_place(d):                          # can we keep every pair at least d apart?
        count, last = 1, stalls[0]             # place first cow in the first stall
        for s in stalls[1:]:
            if s - last >= d:                  # far enough from the previous cow
                count += 1                     # place another cow here
                last = s
        return count >= cows                   # True if we placed all cows
    lo, hi = 1, stalls[-1] - stalls[0]         # smallest and largest possible minimum distance
    ans = 0
    while lo <= hi:
        mid = (lo + hi) // 2
        if can_place(mid):                     # distance mid works -> try larger
            ans = mid
            lo = mid + 1
        else:                                  # too large -> try smaller
            hi = mid - 1
    return ans

print(aggressive_cows([1, 2, 4, 8, 9], 3))     # cows at 1, 4, 8 / 9 -> min gap 3
print(aggressive_cows([1, 2, 8, 4, 9], 2))     # unsorted input is fine, extremes -> 8
```
**Output**
```
3
8
```
**Complexity:** O(n log n + n log(range)).

---

### 🔴 Problem 13 — Split Array Largest Sum (Hard) · *Minimise the maximum subarray sum*
**Idea:** Split the array into `m` contiguous parts so that the **largest part-sum is minimised**. Answer lies in `[max(nums), sum(nums)]`. For a candidate limit, greedily count how many parts you need.

```python
def split_array(nums, m):
    def parts_needed(limit):                   # fewest parts if no part may exceed limit
        parts, cur = 1, 0                      # start the first part
        for x in nums:
            if cur + x > limit:                # adding x would break the limit
                parts += 1                     # start a new part
                cur = 0
            cur += x
        return parts
    lo, hi = max(nums), sum(nums)              # limit can't be below the biggest element
    while lo < hi:
        mid = (lo + hi) // 2
        if parts_needed(mid) <= m:             # limit mid is enough -> try smaller limit
            hi = mid
        else:                                  # needs too many parts -> raise limit
            lo = mid + 1
    return lo

print(split_array([7, 2, 5, 10, 8], 2))        # [7,2,5] | [10,8] -> 18
print(split_array([1, 2, 3, 4, 5], 2))         # [1,2,3] | [4,5] -> largest part is 9
print(split_array([1, 4, 4], 3))
```
**Output**
```
18
9
4
```
**Complexity:** O(n log(sum)). (This same pattern solves "Painter's Partition" and "Allocate Books".)

---

### 🔴 Problem 14 — Kth Smallest Element in a Sorted Matrix (Hard) · *Binary search on value*
**Idea:** Rows and columns are sorted. Binary search over **values** (not indexes) between the smallest and largest element. For a value `mid`, count how many elements are `<= mid` using the staircase walk from the bottom-left. The smallest `mid` with `count >= k` is the answer.

```python
def kth_smallest(matrix, k):
    n = len(matrix)
    def count_le(x):                           # how many elements are <= x
        row, col, cnt = n - 1, 0, 0            # start at bottom-left corner
        while row >= 0 and col < n:
            if matrix[row][col] <= x:          # whole column part above (row+1 items) is <= x
                cnt += row + 1
                col += 1                       # move right
            else:
                row -= 1                       # value too big, move up
        return cnt
    lo, hi = matrix[0][0], matrix[-1][-1]      # smallest and largest value in the matrix
    while lo < hi:
        mid = (lo + hi) // 2
        if count_le(mid) >= k:                 # at least k elements <= mid -> answer <= mid
            hi = mid
        else:                                  # fewer than k -> answer is larger
            lo = mid + 1
    return lo

print(kth_smallest([[1, 5, 9], [10, 11, 13], [12, 13, 15]], 8))
print(kth_smallest([[-5]], 1))
```
**Output**
```
13
-5
```
**Complexity:** O(n log(max − min)) time, O(1) space.

---

### 🔴 Problem 15 — Median of Two Sorted Arrays (Hardest) · *Binary search on partition*
**Idea:** Don't merge. Binary search how many elements `i` to take from the **smaller** array `A` into the left half; the remaining `j = half - i` come from `B`. The partition is correct when
`A[i-1] <= B[j]` **and** `B[j-1] <= A[i]`. Then the median is built from the borders of the partition.

```python
def find_median(A, B):
    if len(A) > len(B):                        # always binary search on the smaller array
        A, B = B, A
    m, n = len(A), len(B)
    half = (m + n + 1) // 2                    # size of the combined left half
    lo, hi = 0, m                              # i can be anything from 0..m
    while lo <= hi:
        i = (lo + hi) // 2                     # elements taken from A into the left half
        j = half - i                           # elements taken from B into the left half
        a_left  = A[i - 1] if i > 0 else float('-inf')   # border values (±inf if empty side)
        a_right = A[i]     if i < m else float('inf')
        b_left  = B[j - 1] if j > 0 else float('-inf')
        b_right = B[j]     if j < n else float('inf')
        if a_left <= b_right and b_left <= a_right:      # perfect partition found
            if (m + n) % 2 == 1:                         # odd total -> max of left half
                return float(max(a_left, b_left))
            return (max(a_left, b_left) + min(a_right, b_right)) / 2  # even -> average of middle two
        elif a_left > b_right:                 # took too many from A -> move i left
            hi = i - 1
        else:                                  # took too few from A -> move i right
            lo = i + 1

print(find_median([1, 3], [2]))
print(find_median([1, 2], [3, 4]))
print(find_median([], [1]))
print(find_median([0, 0], [0, 0]))
```
**Output**
```
2.0
2.5
1.0
0.0
```
**Complexity:** O(log(min(m, n))) time, O(1) space.

---

## 15. Mistakes, Tips & Practice List

### Common mistakes
1. **Infinite loop:** using `lo = mid` with `mid = (lo+hi)//2` (use the upper mid `(lo+hi+1)//2`).
2. **Off-by-one:** mixing `hi = n-1` with `while lo < hi`, or `hi = n` with `while lo <= hi`. Choose ONE template and stick to it.
3. **Binary search on unsorted data** — always confirm sorted/monotonic.
4. **Forgetting duplicates:** use lower/upper bound.
5. **Wrong search range in "answer" problems:** `lo` must be the smallest legal answer (e.g. `max(nums)`), `hi` the largest.
6. **Integer overflow** (C++/Java): `mid = lo + (hi - lo)//2`. Python is safe, but write it this way anyway.
7. **Float binary search:** loop a fixed number of iterations (e.g. 100) instead of `while lo < hi`.

### How to recognise binary search in an interview
- "sorted", "rotated", "find first/last", "O(log n)" → direct.
- "minimum possible maximum", "maximum possible minimum", "least capacity/time/speed" → **binary search on answer**.
- n up to 10^9 or 10^18 with no array → almost surely binary search on answer.


