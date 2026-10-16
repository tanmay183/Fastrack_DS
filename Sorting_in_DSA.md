# 🔢 Sorting Techniques in DSA (Python) — Theory, Code & 15 Problems

> Covers: sorting terminology, Python's built-in sort, all commonly used sorting algorithms (theory + commented code + output), a comparison table, how to choose a sort, and 15 standard problems from easy to hard.

---

## 📑 Table of Contents
1. [What is Sorting & Why It Matters](#1-what-is-sorting--why-it-matters)
2. [Important Terminology](#2-important-terminology)
3. [Python Built-in Sorting (`sort`, `sorted`, `key`)](#3-python-built-in-sorting)
4. [Bubble Sort](#4-bubble-sort)
5. [Selection Sort](#5-selection-sort)
6. [Insertion Sort](#6-insertion-sort)
7. [Merge Sort](#7-merge-sort)
8. [Quick Sort](#8-quick-sort)
9. [Heap Sort](#9-heap-sort)
10. [Counting Sort](#10-counting-sort)
11. [Radix Sort](#11-radix-sort)
12. [Bucket Sort](#12-bucket-sort)
13. [Shell Sort](#13-shell-sort)
14. [Comparison Table & How to Choose](#14-comparison-table--how-to-choose)
15. [Sorting Patterns in Problems](#15-sorting-patterns-in-problems)
16. [15 Standard Problems](#16-15-standard-problems-easy--hard)
17. [Mistakes, Tips & Practice List](#17-mistakes-tips--practice-list)

---

## 1. What is Sorting & Why It Matters

**Sorting** = arranging elements in a defined order (ascending / descending / custom).

**Why it is everywhere in DSA**
- It makes **binary search** possible (O(log n) search).
- It brings equal / close elements **next to each other** (duplicates, closest pair, anagrams, intervals).
- It enables **two pointers** (3Sum, merge, pair sum).
- It is the first step of many **greedy** algorithms (activity selection, meeting rooms, assign cookies).

> 💡 Interview rule of thumb: *"If brute force is O(n²) and the order of elements does not matter, try sorting first → O(n log n)."*

---

## 2. Important Terminology

| Term | Meaning |
|---|---|
| **Stable sort** | Equal elements keep their original relative order. (Important when sorting records by multiple keys.) |
| **In-place sort** | Uses O(1) (or O(log n)) extra memory, sorts inside the same array. |
| **Adaptive sort** | Runs faster if the input is already (nearly) sorted. |
| **Comparison-based** | Decides order only by comparing two elements. **Lower bound = O(n log n)** for any such algorithm. |
| **Non-comparison** | Uses the value itself as an index/digit (Counting, Radix, Bucket). Can reach **O(n)** under constraints. |
| **Internal vs External** | Internal = data fits in RAM. External = data on disk (external merge sort). |

**Stable example:** records `(Ravi,85), (Meena,85)` sorted by marks only. A stable sort keeps *Ravi before Meena* because Ravi came first in the input.

---

## 3. Python Built-in Sorting

Python uses **Timsort** (a hybrid of merge sort + insertion sort): **O(n log n) worst case, O(n) best case (already sorted), stable**, O(n) extra space. In coding rounds you will use the built-in 90% of the time, and write your own sort only when asked.

```python
from functools import cmp_to_key                  # converts an old-style compare function into a key

nums = [5, 2, 9, 1, 5, 6]
nums.sort()                                       # in-place ascending, returns None
print(nums)
nums.sort(reverse=True)                           # in-place descending
print(nums)
print(sorted([3, 1, 2]))                          # sorted() returns a NEW list, original unchanged
words = ["banana", "kiwi", "apple", "fig"]
print(sorted(words))                              # strings sort alphabetically
print(sorted(words, key=len))                     # key = function giving the sort value (length)
print(sorted(words, key=len, reverse=True))       # longest first
students = [("Ravi", 85), ("Anu", 92), ("Meena", 85), ("Karan", 70)]
print(sorted(students, key=lambda s: s[1]))       # by marks; Ravi stays before Meena (stability)
print(sorted(students, key=lambda s: (-s[1], s[0])))  # marks DESC, then name ASC using a tuple key
def compare(a, b):                                # old-style comparator: negative => a first
    return -1 if a + b > b + a else 1             # "which concatenation is bigger"
print(sorted(["3", "30", "34"], key=cmp_to_key(compare)))  # custom comparison logic
```
**Output**
```
[1, 2, 5, 5, 6, 9]
[9, 6, 5, 5, 2, 1]
[1, 2, 3]
['apple', 'banana', 'fig', 'kiwi']
['fig', 'kiwi', 'apple', 'banana']
['banana', 'apple', 'kiwi', 'fig']
[('Karan', 70), ('Ravi', 85), ('Meena', 85), ('Anu', 92)]
[('Anu', 92), ('Meena', 85), ('Ravi', 85), ('Karan', 70)]
['34', '3', '30']
```

**Cheat-sheet**
- `key=` is evaluated **once per element** (fast). Prefer it over `cmp_to_key`.
- To sort descending numerically with a tuple key use `-value`; for strings use `reverse=True` or two sorts (stable!).
- `sort()` is in-place → returns `None` (common bug: `a = a.sort()`).

---

## 4. Bubble Sort

### Theory
Repeatedly **compare adjacent elements and swap** them if they are in the wrong order. After pass 1 the largest element "bubbles" to the end; after pass 2 the second largest settles, and so on.

**Steps**
1. For `i` in `0..n-2` (passes):
2. Compare each adjacent pair `(arr[j], arr[j+1])` for `j` in `0..n-2-i`.
3. Swap if `arr[j] > arr[j+1]`.
4. If a full pass makes **no swap**, the array is sorted → stop early (optimisation).

**Dry run** `[5, 3, 4, 1]`
| Pass | Array after pass |
|---|---|
| start | 5 3 4 1 |
| 1 | 3 4 1 **5** |
| 2 | 3 1 **4 5** |
| 3 | 1 **3 4 5** |

**Complexity:** Best O(n) (with early exit), Average/Worst O(n²), Space O(1). **Stable ✔, in-place ✔, adaptive ✔.**
**Use:** teaching, tiny arrays. Never for large data.

```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n - 1):                        # each pass fixes one more element at the end
        swapped = False                           # tracks whether this pass changed anything
        for j in range(n - 1 - i):                # last i elements are already in place
            if arr[j] > arr[j + 1]:               # adjacent pair in wrong order
                arr[j], arr[j + 1] = arr[j + 1], arr[j]  # swap them
                swapped = True                    # remember that we swapped
        if not swapped:                           # no swaps => already sorted => stop early
            break
    return arr

print(bubble_sort([64, 34, 25, 12, 22, 11, 90]))
print(bubble_sort([1, 2, 3, 4, 5]))               # sorted input finishes in one pass
```
**Output**
```
[11, 12, 22, 25, 34, 64, 90]
[1, 2, 3, 4, 5]
```

---

## 5. Selection Sort

### Theory
Repeatedly **select the minimum** of the unsorted part and **place it at the front** of the unsorted part.

**Steps**
1. For `i` in `0..n-2`: assume `arr[i]` is the minimum.
2. Scan `i+1..n-1` to find the real minimum index.
3. Swap it with `arr[i]`.

**Dry run** `[29, 10, 14, 37, 13]`: min=10 → `[10, 29, 14, 37, 13]` → min=13 → `[10, 13, 14, 37, 29]` → 14 in place → min=29 → `[10, 13, 14, 29, 37]`.

**Complexity:** Always O(n²) comparisons, only **O(n) swaps** (at most n-1), Space O(1). **Not stable** (long-distance swap), in-place ✔.
**Use:** when writes/swaps are expensive (flash memory).

```python
def selection_sort(arr):
    n = len(arr)
    for i in range(n - 1):                        # i = start of the unsorted part
        min_idx = i                               # assume the first unsorted element is smallest
        for j in range(i + 1, n):                 # scan the rest of the unsorted part
            if arr[j] < arr[min_idx]:             # found a smaller element
                min_idx = j                       # remember its index
        arr[i], arr[min_idx] = arr[min_idx], arr[i]  # place the minimum at position i
    return arr

print(selection_sort([29, 10, 14, 37, 13]))
print(selection_sort([5, 5, 2, 8, 2]))            # duplicates are fine
```
**Output**
```
[10, 13, 14, 29, 37]
[2, 2, 5, 5, 8]
```

---

## 6. Insertion Sort

### Theory
Works like **sorting playing cards in your hand**: take the next card and **insert** it in the correct position among the already sorted cards on the left, shifting bigger ones right.

**Steps**
1. Left part `arr[0..i-1]` is sorted. Take `key = arr[i]`.
2. Shift every element of the sorted part that is `> key` one step right.
3. Drop `key` into the gap.

**Dry run** `[12, 11, 13, 5, 6]`: insert 11 → `[11,12,13,5,6]`; 13 stays; insert 5 → `[5,11,12,13,6]`; insert 6 → `[5,6,11,12,13]`.

**Complexity:** Best O(n) (already sorted), Average/Worst O(n²), Space O(1). **Stable ✔, in-place ✔, adaptive ✔, online ✔** (can sort as data arrives).
**Use:** small or nearly sorted arrays. Timsort uses it for tiny runs.

```python
def insertion_sort(arr):
    for i in range(1, len(arr)):                  # arr[0] alone is already "sorted"
        key = arr[i]                              # element to insert into the sorted part
        j = i - 1                                 # start comparing from the end of sorted part
        while j >= 0 and arr[j] > key:            # shift larger elements one place right
            arr[j + 1] = arr[j]
            j -= 1
        arr[j + 1] = key                          # place key in its correct position
    return arr

print(insertion_sort([12, 11, 13, 5, 6]))
print(insertion_sort([1, 2, 3, 4]))               # nearly O(n) for sorted input
```
**Output**
```
[5, 6, 11, 12, 13]
[1, 2, 3, 4]
```

---

## 7. Merge Sort

### Theory
**Divide and conquer.**
1. **Divide:** split the array into two halves.
2. **Conquer:** recursively sort each half.
3. **Combine (merge):** merge two sorted halves into one sorted array using two pointers.

**Recursion tree** for `[38, 27, 43, 3]`:
```
[38,27,43,3]
 ├─[38,27] ─ [38] [27] → merge → [27,38]
 └─[43,3]  ─ [43] [3]  → merge → [3,43]
 merge([27,38],[3,43]) → [3,27,38,43]
```
**Complexity:** **Always O(n log n)** (best/avg/worst), Space O(n). **Stable ✔** (if `<=` used when merging), not in-place.
**Use:** guaranteed O(n log n), linked lists, external sorting, counting inversions.

```python
def merge_sort(arr):
    if len(arr) <= 1:                             # base case: 0 or 1 element is already sorted
        return arr
    mid = len(arr) // 2                           # split point
    left = merge_sort(arr[:mid])                  # recursively sort the left half
    right = merge_sort(arr[mid:])                 # recursively sort the right half
    return merge(left, right)                     # combine the two sorted halves

def merge(left, right):
    result = []                                   # merged output
    i = j = 0                                     # pointers into left and right
    while i < len(left) and j < len(right):       # while both halves have elements
        if left[i] <= right[j]:                   # '<=' keeps equal elements in original order (stable)
            result.append(left[i])                # take the smaller front element
            i += 1
        else:
            result.append(right[j])
            j += 1
    result.extend(left[i:])                       # append leftovers of left (if any)
    result.extend(right[j:])                      # append leftovers of right (if any)
    return result

print(merge_sort([38, 27, 43, 3, 9, 82, 10]))
print(merge_sort([5, 1, 1, 2, 0, 0]))
```
**Output**
```
[3, 9, 10, 27, 38, 43, 82]
[0, 0, 1, 1, 2, 5]
```

---

## 8. Quick Sort

### Theory
**Divide and conquer using a pivot.**
1. Choose a **pivot**.
2. **Partition:** rearrange so everything `<= pivot` is on the left, everything `> pivot` on the right. The pivot lands in its **final sorted position**.
3. Recursively sort the left and right parts.

**Lomuto partition (used below):** pivot = last element; pointer `i` marks the end of the "smaller" zone; scan `j` and swap smaller elements into the zone; finally swap pivot into `i+1`.

**Complexity:** Average **O(n log n)**, Worst O(n²) (already sorted + bad pivot like always-last), Space O(log n) recursion. **Not stable**, in-place ✔.
**Fix worst case:** pick a **random pivot** or median-of-three.
**Use:** fastest in practice for arrays (cache-friendly). Its partition idea also gives **Quickselect** (kth smallest in O(n) average).

```python
import random

def partition(arr, low, high):
    r = random.randint(low, high)                 # random pivot avoids the O(n^2) worst case
    arr[r], arr[high] = arr[high], arr[r]         # move the chosen pivot to the end
    pivot = arr[high]                             # pivot value
    i = low - 1                                   # end of the "<= pivot" zone (empty at first)
    for j in range(low, high):                    # scan every element except the pivot
        if arr[j] <= pivot:                       # belongs to the smaller zone
            i += 1                                # grow the zone
            arr[i], arr[j] = arr[j], arr[i]       # swap element into the zone
    arr[i + 1], arr[high] = arr[high], arr[i + 1] # put pivot right after the smaller zone
    return i + 1                                  # final index of the pivot

def quick_sort(arr, low=0, high=None):
    if high is None:                              # first call: sort the whole array
        high = len(arr) - 1
    if low < high:                                # at least 2 elements
        p = partition(arr, low, high)             # pivot is now at its final place p
        quick_sort(arr, low, p - 1)               # sort elements left of the pivot
        quick_sort(arr, p + 1, high)              # sort elements right of the pivot
    return arr

print(quick_sort([10, 7, 8, 9, 1, 5]))
print(quick_sort([3, 3, 3, 1, 2]))                # duplicates handled by '<='
```
**Output**
```
[1, 5, 7, 8, 9, 10]
[1, 2, 3, 3, 3]
```

---

## 9. Heap Sort

### Theory
Uses a **binary max-heap** stored inside the array (children of index `i` are `2i+1` and `2i+2`).
1. **Build a max-heap** from the array (heapify from the last parent down to 0) → O(n).
2. Repeat: swap the root (maximum) with the last element, shrink the heap by 1, **heapify** the root down to restore the heap.

**Complexity:** Always O(n log n), Space O(1). **In-place ✔, not stable.**
**Use:** guaranteed O(n log n) with O(1) memory. The heap idea is also used for top-K, priority queues, merging K lists.

```python
def heapify(arr, n, i):                           # sink arr[i] down inside heap of size n
    largest = i                                   # assume the parent is the largest
    left, right = 2 * i + 1, 2 * i + 2            # children indexes
    if left < n and arr[left] > arr[largest]:     # left child bigger than current largest
        largest = left
    if right < n and arr[right] > arr[largest]:   # right child bigger than current largest
        largest = right
    if largest != i:                              # heap property violated
        arr[i], arr[largest] = arr[largest], arr[i]  # swap parent with the bigger child
        heapify(arr, n, largest)                  # continue fixing the affected subtree

def heap_sort(arr):
    n = len(arr)
    for i in range(n // 2 - 1, -1, -1):           # build max-heap starting from the last parent
        heapify(arr, n, i)
    for end in range(n - 1, 0, -1):               # repeatedly extract the maximum
        arr[0], arr[end] = arr[end], arr[0]       # move current max to its final place
        heapify(arr, end, 0)                      # restore heap on the reduced array
    return arr

print(heap_sort([12, 11, 13, 5, 6, 7]))
```
**Output**
```
[5, 6, 7, 11, 12, 13]
```

---

## 10. Counting Sort

### Theory
**Non-comparison sort** for integers in a **small range** `[0, k]`.
1. Count the frequency of each value.
2. Take **prefix sums** of counts → `count[v]` tells the position *after* the last `v` in the output.
3. Walk the input **backwards**, place each element at `count[v]-1` and decrement (this keeps the sort **stable**).

**Complexity:** O(n + k) time, O(n + k) space. **Stable ✔** (backward fill).
**Use:** small value range (ages, marks, digits, characters), as the subroutine of Radix sort. Bad when `k` is huge (e.g. values up to 10^9).

```python
def counting_sort(arr):
    if not arr:                                   # nothing to sort
        return arr
    mx = max(arr)                                 # range is 0..mx (non-negative integers only)
    count = [0] * (mx + 1)                        # count[v] = number of times v appears
    for x in arr:
        count[x] += 1                             # frequency counting
    for i in range(1, len(count)):
        count[i] += count[i - 1]                  # prefix sum: count[v] = number of elements <= v
    out = [0] * len(arr)                          # output array
    for x in reversed(arr):                       # go backwards to keep stability
        count[x] -= 1                             # next free slot for value x
        out[count[x]] = x                         # place x there
    return out

print(counting_sort([4, 2, 2, 8, 3, 3, 1]))
print(counting_sort([0, 5, 0, 5]))
```
**Output**
```
[1, 2, 2, 3, 3, 4, 8]
[0, 0, 5, 5]
```

---

## 11. Radix Sort

### Theory
Sort integers **digit by digit** from the **least significant digit (LSD)** to the most significant, using a **stable** sort (counting sort) for each digit.

**Why stable matters:** when sorting by the tens digit, numbers with the same tens digit must stay ordered by the units digit sorted earlier.

**Dry run** `[170, 45, 75, 90, 802, 24, 2, 66]`: by units → `[170, 90, 802, 2, 24, 45, 75, 66]`; by tens → `[802, 2, 24, 45, 66, 170, 75, 90]`; by hundreds → `[2, 24, 45, 66, 75, 90, 170, 802]`.

**Complexity:** O(d × (n + b)) where `d` = number of digits, `b` = base (10). Space O(n + b). **Stable ✔.**
**Use:** large lists of integers with a limited number of digits (phone numbers, IDs).

```python
def counting_by_digit(arr, exp):                  # stable counting sort on digit (x // exp) % 10
    count = [0] * 10                              # digits 0..9
    out = [0] * len(arr)
    for x in arr:
        count[(x // exp) % 10] += 1               # count each digit
    for i in range(1, 10):
        count[i] += count[i - 1]                  # prefix sums -> final positions
    for x in reversed(arr):                       # backwards for stability
        d = (x // exp) % 10                       # digit at current place value
        count[d] -= 1
        out[count[d]] = x
    return out

def radix_sort(arr):
    if not arr:
        return arr
    exp = 1                                       # 1 = units, 10 = tens, 100 = hundreds ...
    mx = max(arr)                                 # number of passes depends on the largest number
    while mx // exp > 0:                          # until all digits of the biggest number are processed
        arr = counting_by_digit(arr, exp)         # sort by the current digit
        exp *= 10                                 # move to the next digit
    return arr

print(radix_sort([170, 45, 75, 90, 802, 24, 2, 66]))
```
**Output**
```
[2, 24, 45, 66, 75, 90, 170, 802]
```

---

## 12. Bucket Sort

### Theory
Distribute elements into **buckets** (ranges), sort each bucket (usually insertion sort or built-in), then concatenate.
Best when input is **uniformly distributed**, e.g. floats in `[0, 1)`.

**Steps**
1. Create `n` empty buckets.
2. Put each value `x` in bucket `int(x * n)`.
3. Sort every bucket.
4. Concatenate buckets in order.

**Complexity:** Average O(n + k) (uniform data), Worst O(n²) (everything in one bucket). Space O(n + k). **Stable** if the bucket sort is stable.

```python
def bucket_sort(arr):                             # values must be in [0, 1)
    n = len(arr)
    buckets = [[] for _ in range(n)]              # n empty buckets
    for x in arr:
        buckets[int(x * n)].append(x)             # bucket index from the value range
    for b in buckets:
        b.sort()                                  # sort each small bucket
    return [x for b in buckets for x in b]        # concatenate all buckets in order

print(bucket_sort([0.78, 0.17, 0.39, 0.26, 0.72, 0.94, 0.21, 0.12, 0.23, 0.68]))
```
**Output**
```
[0.12, 0.17, 0.21, 0.23, 0.26, 0.39, 0.68, 0.72, 0.78, 0.94]
```

---

## 13. Shell Sort

### Theory
An improved **insertion sort**: first sort elements that are **far apart** (gap `g`), then reduce the gap until it is 1 (the final pass is a normal insertion sort on a nearly sorted array, which is fast).

**Complexity:** depends on the gap sequence: ~O(n²) for `n/2, n/4, ...`, O(n^1.3)–O(n log² n) for better sequences. Space O(1). **Not stable.**

```python
def shell_sort(arr):
    n = len(arr)
    gap = n // 2                                  # start with a big gap
    while gap > 0:
        for i in range(gap, n):                   # gapped insertion sort
            temp = arr[i]                         # element to insert
            j = i
            while j >= gap and arr[j - gap] > temp:  # shift elements that are gap apart
                arr[j] = arr[j - gap]
                j -= gap
            arr[j] = temp                         # place temp
        gap //= 2                                 # shrink the gap (ends with gap = 1)
    return arr

print(shell_sort([12, 34, 54, 2, 3, 100, 1]))
```
**Output**
```
[1, 2, 3, 12, 34, 54, 100]
```

---

## 14. Comparison Table & How to Choose

| Algorithm | Best | Average | Worst | Space | Stable | In-place |
|---|---|---|---|---|---|---|
| Bubble | O(n) | O(n²) | O(n²) | O(1) | ✔ | ✔ |
| Selection | O(n²) | O(n²) | O(n²) | O(1) | ✘ | ✔ |
| Insertion | O(n) | O(n²) | O(n²) | O(1) | ✔ | ✔ |
| Merge | O(n log n) | O(n log n) | O(n log n) | O(n) | ✔ | ✘ |
| Quick | O(n log n) | O(n log n) | O(n²) | O(log n) | ✘ | ✔ |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | ✘ | ✔ |
| Counting | O(n+k) | O(n+k) | O(n+k) | O(n+k) | ✔ | ✘ |
| Radix | O(d(n+b)) | O(d(n+b)) | O(d(n+b)) | O(n+b) | ✔ | ✘ |
| Bucket | O(n+k) | O(n+k) | O(n²) | O(n+k) | ✔* | ✘ |
| Shell | O(n log n) | ~O(n^1.3) | O(n²) | O(1) | ✘ | ✔ |
| Timsort (Python) | O(n) | O(n log n) | O(n log n) | O(n) | ✔ | ✘ |

**Which to use?**
| Situation | Choice |
|---|---|
| Normal coding problem | built-in `sort()` / `sorted()` |
| Need guaranteed O(n log n), stable | Merge sort |
| Need in-place and fastest on average | Quick sort |
| Need in-place + guaranteed O(n log n) | Heap sort |
| Nearly sorted / tiny array | Insertion sort |
| Small integer range | Counting sort |
| Large integers with few digits | Radix sort |
| Uniform floats | Bucket sort |
| Only top K needed | Heap / Quickselect (no need to sort everything) |
| Count inversions, smaller elements to the right | Merge sort with counting |

---

## 15. Sorting Patterns in Problems

| Clue | Technique |
|---|---|
| "closest pair", "minimum difference" | sort, then check **adjacent** elements |
| duplicates / anagram grouping | sort the element (or its characters) as a key |
| "pair/triplet with sum" | sort + two pointers |
| overlapping intervals / meetings | sort by start (or end) + sweep |
| frequency-based ordering | counting / bucket sort |
| "form the largest/smallest number" | custom comparator |
| k-th largest / top K | heap or quickselect |
| "minimum swaps to sort" | cycle detection on sorted positions |
| inversions, "count smaller/greater to the right" | merge sort with counting |
| range of values is small | counting sort |

---

## 16. 15 Standard Problems (Easy → Hard)

---

### 🟢 Problem 1 — Check if Array is Sorted (and Sorted + Rotated) (Easy)
**Idea:** Sorted ⇔ no adjacent pair with `nums[i] < nums[i-1]`. Sorted-and-rotated ⇔ **at most one drop** when the array is viewed circularly.

```python
def is_sorted(nums):
    for i in range(1, len(nums)):                 # compare each element with its predecessor
        if nums[i] < nums[i - 1]:                 # found a descent -> not sorted
            return False
    return True

def is_sorted_and_rotated(nums):
    n = len(nums)
    drops = 0                                     # number of places where order breaks
    for i in range(n):
        if nums[i] > nums[(i + 1) % n]:           # circular comparison (last vs first)
            drops += 1
    return drops <= 1                             # sorted array rotated has at most one drop

print(is_sorted([1, 2, 2, 5]))
print(is_sorted([1, 3, 2]))
print(is_sorted_and_rotated([3, 4, 5, 1, 2]))
print(is_sorted_and_rotated([2, 1, 3, 4]))
```
**Output**
```
True
False
True
False
```
**Complexity:** O(n) time, O(1) space.

---

### 🟢 Problem 2 — Squares of a Sorted Array (Easy) · *Two pointers*
**Idea:** Input sorted with negatives; the largest square is at one of the two **ends**. Fill the result from the back, comparing absolute values at both ends.

```python
def sorted_squares(nums):
    n = len(nums)
    res = [0] * n                                 # result array
    l, r = 0, n - 1                               # pointers at both ends
    for pos in range(n - 1, -1, -1):              # fill from the largest square to the smallest
        if abs(nums[l]) > abs(nums[r]):           # left end has larger magnitude
            res[pos] = nums[l] ** 2
            l += 1
        else:                                     # right end has larger (or equal) magnitude
            res[pos] = nums[r] ** 2
            r -= 1
    return res

print(sorted_squares([-4, -1, 0, 3, 10]))
print(sorted_squares([-7, -3, 2, 3, 11]))
```
**Output**
```
[0, 1, 9, 16, 100]
[4, 9, 9, 49, 121]
```
**Complexity:** O(n) (better than squaring + sorting = O(n log n)), O(n) space.

---

### 🟢 Problem 3 — Merge Sorted Array in Place (Easy) · *Merge from the back*
**Idea:** `nums1` has extra empty space at its end. If you merge from the **front** you overwrite data, so merge from the **back** placing the larger element first.

```python
def merge_sorted(nums1, m, nums2, n):
    i, j, k = m - 1, n - 1, m + n - 1             # i: last real of nums1, j: last of nums2, k: write position
    while j >= 0:                                 # once nums2 is exhausted, nums1's rest is already in place
        if i >= 0 and nums1[i] > nums2[j]:        # nums1's element is larger
            nums1[k] = nums1[i]
            i -= 1
        else:                                     # nums2's element is larger (or nums1 exhausted)
            nums1[k] = nums2[j]
            j -= 1
        k -= 1                                    # move write position left
    return nums1

print(merge_sorted([1, 2, 3, 0, 0, 0], 3, [2, 5, 6], 3))
print(merge_sorted([0], 0, [1], 1))
```
**Output**
```
[1, 2, 2, 3, 5, 6]
[1]
```
**Complexity:** O(m + n) time, O(1) space.

---

### 🟢 Problem 4 — Minimum Absolute Difference (Easy) · *Sort, then adjacent*
**Idea:** The closest two numbers must be neighbours after sorting. Find the smallest adjacent gap, then collect all pairs with that gap.

```python
def min_abs_diff(arr):
    arr.sort()                                    # neighbours are now the closest candidates
    best = min(arr[i + 1] - arr[i] for i in range(len(arr) - 1))  # smallest adjacent difference
    return [[arr[i], arr[i + 1]]                  # collect every pair that achieves it
            for i in range(len(arr) - 1)
            if arr[i + 1] - arr[i] == best]

print(min_abs_diff([4, 2, 1, 3]))
print(min_abs_diff([1, 3, 6, 10, 15]))
```
**Output**
```
[[1, 2], [2, 3], [3, 4]]
[[1, 3]]
```
**Complexity:** O(n log n) time.

---

### 🟡 Problem 5 — Sort Characters by Frequency (Medium) · *Bucket sort*
**Idea:** Count characters, put each character into the bucket of its frequency (index = frequency), then read buckets from the highest frequency down. Ties may appear in any order.

```python
from collections import Counter

def frequency_sort(s):
    freq = Counter(s)                             # character -> count
    buckets = [[] for _ in range(len(s) + 1)]     # buckets[c] = characters that appear c times
    for ch, c in freq.items():
        buckets[c].append(ch)                     # drop char into its frequency bucket
    res = []
    for c in range(len(s), 0, -1):                # highest frequency first
        for ch in buckets[c]:
            res.append(ch * c)                    # write the character c times
    return ''.join(res)

print(frequency_sort("tree"))                     # 'eetr' (e twice first; 'eert' is also valid)
print(frequency_sort("cccaaa"))
print(frequency_sort("Aabb"))
```
**Output**
```
eetr
cccaaa
bbAa
```
**Complexity:** O(n) time, O(n) space.

---

### 🟡 Problem 6 — Largest Number (Medium) · *Custom comparator*
**Idea:** Order numbers by which concatenation is bigger: `a` goes before `b` if `a+b > b+a` (as strings). Example: `"9"` vs `"34"` → `"934" > "349"`.

```python
from functools import cmp_to_key

def largest_number(nums):
    strs = list(map(str, nums))                   # work with strings for concatenation
    def cmp(a, b):
        if a + b > b + a:                         # a should come first
            return -1
        elif a + b < b + a:                       # b should come first
            return 1
        return 0                                  # equal
    strs.sort(key=cmp_to_key(cmp))                # sort with the custom order
    res = ''.join(strs)                           # glue together
    return '0' if res[0] == '0' else res          # all zeros -> single "0"

print(largest_number([10, 2]))
print(largest_number([3, 30, 34, 5, 9]))
print(largest_number([0, 0]))
```
**Output**
```
210
9534330
0
```
**Complexity:** O(n log n · L) where L = digit length.

---

### 🟡 Problem 7 — Meeting Rooms II (Medium) · *Sort starts and ends separately*
**Idea:** The number of rooms = maximum number of overlapping meetings. Sort start times and end times independently. For each start: if it is `>=` the earliest finishing meeting, reuse that room; otherwise open a new room.

```python
def min_meeting_rooms(intervals):
    starts = sorted(i[0] for i in intervals)      # all start times in order
    ends = sorted(i[1] for i in intervals)        # all end times in order
    rooms = 0                                     # rooms currently allocated
    e = 0                                         # pointer to the earliest ending meeting
    for s in starts:                              # process meetings by start time
        if s >= ends[e]:                          # earliest meeting is over -> reuse its room
            e += 1
        else:                                     # all rooms busy -> need a new room
            rooms += 1
    return rooms

print(min_meeting_rooms([[0, 30], [5, 10], [15, 20]]))
print(min_meeting_rooms([[7, 10], [2, 4]]))
```
**Output**
```
2
1
```
**Complexity:** O(n log n) time, O(n) space.

---

### 🟡 Problem 8 — Top K Frequent Elements (Medium) · *Bucket sort*
**Idea:** Frequency can be at most `n`, so use buckets indexed by frequency and scan from the top → O(n), better than sorting by frequency (O(n log n)).

```python
from collections import Counter

def top_k_frequent(nums, k):
    freq = Counter(nums)                          # value -> frequency
    buckets = [[] for _ in range(len(nums) + 1)]  # index = frequency
    for num, c in freq.items():
        buckets[c].append(num)
    res = []
    for c in range(len(nums), 0, -1):             # most frequent first
        for num in buckets[c]:
            res.append(num)
            if len(res) == k:                     # collected k elements -> done
                return res
    return res

print(top_k_frequent([1, 1, 1, 2, 2, 3], 2))
print(top_k_frequent([1], 1))
```
**Output**
```
[1, 2]
[1]
```
**Complexity:** O(n) time, O(n) space.

---

### 🟡 Problem 9 — Kth Largest Element (Medium-Hard) · *Quickselect*
**Idea:** Use quick sort's partition but recurse into **only the side that contains the target index**. Kth largest = element at index `n - k` in sorted order. Average O(n).

```python
import random

def kth_largest(nums, k):
    target = len(nums) - k                        # index of the answer in ascending order
    lo, hi = 0, len(nums) - 1
    while True:
        r = random.randint(lo, hi)                # random pivot for expected O(n)
        nums[r], nums[hi] = nums[hi], nums[r]     # move pivot to the end
        pivot = nums[hi]
        i = lo                                    # boundary of elements < pivot
        for j in range(lo, hi):
            if nums[j] < pivot:                   # smaller element goes to the left zone
                nums[i], nums[j] = nums[j], nums[i]
                i += 1
        nums[i], nums[hi] = nums[hi], nums[i]     # pivot into its final sorted position i
        if i == target:                           # pivot is exactly the answer
            return nums[i]
        elif i < target:                          # answer lies to the right of pivot
            lo = i + 1
        else:                                     # answer lies to the left of pivot
            hi = i - 1

print(kth_largest([3, 2, 1, 5, 6, 4], 2))
print(kth_largest([3, 2, 3, 1, 2, 4, 5, 5, 6], 4))
```
**Output**
```
5
4
```
**Complexity:** Average O(n), worst O(n²) (rare with random pivot), O(1) space. Alternative: min-heap of size k → O(n log k).

---

### 🔴 Problem 10 — Minimum Swaps to Sort an Array (Hard-ish) · *Cycle decomposition*
**Idea (distinct elements):** Find where each element must go after sorting. This forms **cycles**; a cycle of length `L` needs `L - 1` swaps.

```python
def min_swaps(arr):
    n = len(arr)
    pos = sorted(range(n), key=lambda i: arr[i])  # pos[k] = original index of the k-th smallest value
    visited = [False] * n
    swaps = 0
    for i in range(n):
        if visited[i] or pos[i] == i:             # already visited or already in the right place
            continue
        j, size = i, 0                            # walk along the cycle starting at i
        while not visited[j]:
            visited[j] = True                     # mark node as visited
            j = pos[j]                            # jump to where this element must go
            size += 1                             # count cycle length
        swaps += size - 1                         # a cycle of length L needs L-1 swaps
    return swaps

print(min_swaps([4, 3, 2, 1]))
print(min_swaps([1, 5, 4, 3, 2]))
print(min_swaps([1, 2, 3]))
```
**Output**
```
2
2
0
```
**Complexity:** O(n log n) time, O(n) space.

---

### 🔴 Problem 11 — Count Inversions (Hard) · *Merge sort with counting*
**Idea:** An inversion is a pair `i < j` with `arr[i] > arr[j]`. During the merge step, if `right[j]` is taken before `left[i]`, then **all remaining** elements in `left` (from `i` on) are larger than it → add `len(left) - i`.

```python
def count_inversions(arr):
    def sort_count(a):                            # returns (sorted list, inversion count)
        if len(a) <= 1:
            return a, 0
        mid = len(a) // 2
        left, lc = sort_count(a[:mid])            # inversions inside the left half
        right, rc = sort_count(a[mid:])           # inversions inside the right half
        merged, i, j = [], 0, 0
        inv = lc + rc                             # start with inversions found in the halves
        while i < len(left) and j < len(right):
            if left[i] <= right[j]:               # no inversion, take from left
                merged.append(left[i])
                i += 1
            else:                                 # right[j] jumps ahead of everything left in 'left'
                merged.append(right[j])
                j += 1
                inv += len(left) - i              # all remaining left elements are bigger -> inversions
        merged.extend(left[i:])                   # append leftovers
        merged.extend(right[j:])
        return merged, inv
    return sort_count(arr)[1]

print(count_inversions([2, 4, 1, 3, 5]))          # (2,1) (4,1) (4,3)
print(count_inversions([5, 4, 3, 2, 1]))          # fully reversed -> n(n-1)/2
print(count_inversions([1, 2, 3]))
```
**Output**
```
3
10
0
```
**Complexity:** O(n log n) time, O(n) space (brute force is O(n²)).

---

### 🔴 Problem 12 — Maximum Gap (Hard) · *Bucket / pigeonhole principle*
**Idea:** Find the maximum difference between successive elements in sorted order in **O(n)** without sorting. With `n` numbers between `lo` and `hi`, the answer is at least `(hi - lo)/(n - 1)`. So use buckets of that width: the max gap can **never be inside one bucket**, only between the max of one bucket and the min of the next non-empty bucket.

```python
def maximum_gap(nums):
    n = len(nums)
    if n < 2:                                     # need at least two numbers
        return 0
    lo, hi = min(nums), max(nums)
    if lo == hi:                                  # all numbers equal
        return 0
    size = max(1, (hi - lo) // (n - 1))           # bucket width (pigeonhole lower bound on the gap)
    count = (hi - lo) // size + 1                 # number of buckets
    bmin = [None] * count                         # smallest value in each bucket
    bmax = [None] * count                         # largest value in each bucket
    for x in nums:
        b = (x - lo) // size                      # bucket index of x
        bmin[b] = x if bmin[b] is None else min(bmin[b], x)
        bmax[b] = x if bmax[b] is None else max(bmax[b], x)
    best, prev = 0, lo                            # prev = max of the previous non-empty bucket
    for b in range(count):
        if bmin[b] is None:                       # skip empty buckets
            continue
        best = max(best, bmin[b] - prev)          # gap between consecutive non-empty buckets
        prev = bmax[b]
    return best

print(maximum_gap([3, 6, 9, 1]))
print(maximum_gap([10]))
print(maximum_gap([100, 3, 2, 1]))
```
**Output**
```
3
0
97
```
**Complexity:** O(n) time, O(n) space.

---

### 🔴 Problem 13 — Merge K Sorted Arrays (Hard) · *Heap*
**Idea:** Keep a min-heap with the current front element of each array. Pop the smallest, push the next element from the same array.

```python
import heapq

def merge_k_sorted(lists):
    heap = []                                     # entries: (value, which list, index in that list)
    res = []
    for i, l in enumerate(lists):
        if l:                                     # push the first element of every non-empty list
            heapq.heappush(heap, (l[0], i, 0))
    while heap:
        val, i, j = heapq.heappop(heap)           # smallest value among all current fronts
        res.append(val)
        if j + 1 < len(lists[i]):                 # that list still has elements
            heapq.heappush(heap, (lists[i][j + 1], i, j + 1))  # push its next element
    return res

print(merge_k_sorted([[1, 4, 5], [1, 3, 4], [2, 6]]))
print(merge_k_sorted([[], [1], []]))
```
**Output**
```
[1, 1, 2, 3, 4, 4, 5, 6]
[1]
```
**Complexity:** O(N log k) where N = total elements, k = number of lists.

---

### 🔴 Problem 14 — Count of Smaller Numbers After Self (Hard) · *Merge sort on indexes*
**Idea:** For every `i`, count `j > i` with `nums[j] < nums[i]`. Merge sort the **indexes** by value. While merging, when an element from the **left** half is placed, the number of right-half elements already placed (`j - mid`) are exactly the smaller elements that were to its right.

```python
def count_smaller(nums):
    n = len(nums)
    counts = [0] * n                              # answer for every original index
    idx = list(range(n))                          # we sort indexes, not values
    def sort(lo, hi):                             # sorts idx[lo:hi] by nums value
        if hi - lo <= 1:
            return
        mid = (lo + hi) // 2
        sort(lo, mid)                             # sort left half
        sort(mid, hi)                             # sort right half
        merged = []
        i, j = lo, mid
        while i < mid or j < hi:
            if j == hi or (i < mid and nums[idx[i]] <= nums[idx[j]]):
                counts[idx[i]] += j - mid         # right elements already placed are strictly smaller
                merged.append(idx[i])
                i += 1
            else:
                merged.append(idx[j])             # take from right half
                j += 1
        idx[lo:hi] = merged                       # write merged order back
    sort(0, n)
    return counts

print(count_smaller([5, 2, 6, 1]))
print(count_smaller([-1]))
print(count_smaller([-1, -1]))
```
**Output**
```
[2, 1, 1, 0]
[0]
[0, 0]
```
**Complexity:** O(n log n) time, O(n) space (brute force O(n²)).

---

### 🔴 Problem 15 — Reverse Pairs (Hard) · *Merge sort with a custom counting step*
**Idea:** Count pairs `i < j` with `nums[i] > 2 * nums[j]`. After sorting both halves, count **before** merging using a moving pointer `j` on the right half (as `x` grows in sorted `left`, `j` never moves back), then merge normally.

```python
def reverse_pairs(nums):
    def sort_count(a):                            # returns (sorted list, number of reverse pairs)
        if len(a) <= 1:
            return a, 0
        mid = len(a) // 2
        left, lc = sort_count(a[:mid])
        right, rc = sort_count(a[mid:])
        cnt = lc + rc                             # pairs inside each half
        j = 0
        for x in left:                            # left is sorted ascending
            while j < len(right) and x > 2 * right[j]:  # right[j] is "small enough" for x
                j += 1
            cnt += j                              # all right[0..j-1] pair with x (and with larger x's)
        return sorted(left + right), cnt          # merge (Timsort merges two sorted runs in O(n))
    return sort_count(nums)[1]

print(reverse_pairs([1, 3, 2, 3, 1]))
print(reverse_pairs([2, 4, 3, 5, 1]))
```
**Output**
```
2
3
```
**Complexity:** O(n log n) time, O(n) space.

---

## 17. Mistakes, Tips & Practice List

### Common mistakes
1. `arr = arr.sort()` → `sort()` returns `None`. Use `arr.sort()` or `arr = sorted(arr)`.
2. Sorting inside a loop (O(n² log n) hidden cost).
3. Forgetting **stability** when sorting by multiple keys: sort by the secondary key first, then by the primary (or use a tuple key).
4. Quick sort with a fixed pivot on sorted input → O(n²). Randomize.
5. Counting sort with huge values (e.g. up to 10^9) → memory blow-up.
6. Comparator mistakes: `cmp_to_key` must return negative / zero / positive, not True/False.
7. Modifying the input when the problem says "do not modify" → copy first.
8. In merge sort use `<=` to keep it stable.

### Interview talking points
- "Python's `sort` is Timsort: stable, O(n log n), O(n) on already sorted data."
- "Any comparison sort needs Ω(n log n) comparisons."
- "Quick sort is faster in practice, merge sort gives a worst-case guarantee and stability."
- "If I only need the top K, I don't sort fully: heap O(n log k) or quickselect O(n)."

### Practice list
| Level | Problems |
|---|---|
| Easy | Sort Array By Parity (905), Relative Sort Array (1122), Height Checker (1051), Assign Cookies (455), Intersection of Two Arrays II (350), Valid Anagram (242) |
| Medium | Sort Colors (75), Merge Intervals (56), Non-overlapping Intervals (435), Group Anagrams (49), Sort List (148), Wiggle Sort II (324), K Closest Points to Origin (973), Car Fleet (853), Insert Interval (57) |
| Hard | Count of Range Sum (327), Reverse Pairs (493), Count Smaller Numbers After Self (315), Maximum Gap (164), Median of Two Sorted Arrays (4), Merge k Sorted Lists (23) |

### ✅ Final advice for 12 November 2026
- Be able to **write from memory**: Merge sort, Quick sort (partition), Insertion sort, Heap sort. Understand Counting sort.
- Remember the **patterns** in Section 15: most "sorting problems" are just *sort first, then scan*.
- Next topic: tell me **Strings**, **Hashing**, **Recursion & Backtracking**, **Linked List**, **Stack & Queue**... and I'll prepare it in the same format. 🚀
