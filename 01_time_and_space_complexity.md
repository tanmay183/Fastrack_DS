# Topic 1: Time and Space Complexity (in Python)

> **Goal of this note:** after reading it, you should be able to look at any piece of code and say, within seconds, "this is O(n log n) time and O(n) space", and know whether it will pass or give TLE (Time Limit Exceeded) in your coding round.

---

## Table of Contents

1. [Why complexity matters](#1-why-complexity-matters)
2. [What is time complexity?](#2-what-is-time-complexity)
3. [Big-O, Omega, Theta in simple words](#3-big-o-omega-theta-in-simple-words)
4. [The 3 rules for finding Big-O](#4-the-3-rules-for-finding-big-o)
5. [Common complexities with examples](#5-common-complexities-with-examples)
6. [How fast do they grow?](#6-how-fast-do-they-grow)
7. [How to analyse loops](#7-how-to-analyse-loops-step-by-step)
8. [Analysing recursion](#8-analysing-recursion)
9. [Space complexity](#9-space-complexity)
10. [Best, average, worst case](#10-best-average-and-worst-case)
11. [Amortized complexity](#11-amortized-complexity)
12. [Python built-in operation costs (very important)](#12-python-built-in-operation-costs)
13. [Constraints to complexity cheat sheet](#13-constraints-to-complexity-cheat-sheet)
14. [Time vs space trade-off (Two Sum example)](#14-time-vs-space-trade-off-two-sum)
15. [Common mistakes](#15-common-mistakes)
16. [Practice questions with answers](#16-practice-questions-with-answers)
17. [Quick revision sheet](#17-quick-revision-sheet)

---

## 1. Why complexity matters

In a coding round, your code is run against many test cases, and some of them are **huge** (for example, an array of 100,000 or 1,000,000 elements).

- A correct but **slow** solution gives **TLE** and you lose marks.
- A correct but **memory-hungry** solution gives **MLE** (Memory Limit Exceeded).

Complexity is how we **predict** this before running the code.

**Real-life analogy:** You need to find a name in a phone book.

| Method | What you do | Effort |
|---|---|---|
| Linear | Read name by name from page 1 | Grows directly with the size of the book |
| Binary | Open the middle, decide left or right, repeat | Barely grows even if the book becomes 1000x bigger |

Both give the right answer. One survives big inputs. That is the whole point of complexity.

---

## 2. What is time complexity?

**Time complexity does NOT measure seconds.**

Seconds depend on your laptop, the language, and what else is running. Instead, we measure:

> **How does the number of steps grow when the input size `n` grows?**

Example:

```python
def print_all(arr):
    for x in arr:        # runs n times
        print(x)
```

- If `n = 10`, about 10 steps.
- If `n = 1000`, about 1000 steps.
- Steps grow in a straight line with `n`, so this is **O(n)**.

We care about the **growth rate**, not the exact count.

---

## 3. Big-O, Omega, Theta in simple words

| Symbol | Meaning | Think of it as |
|---|---|---|
| **O (Big-O)** | Upper bound | "It will take **at most** this much" |
| **Ω (Omega)** | Lower bound | "It will take **at least** this much" |
| **Θ (Theta)** | Tight bound | "It takes **exactly about** this much" |

**Example:** Linear search in an array of n elements.

- Best case: element is at index 0, so Ω(1)
- Worst case: element is at the end or missing, so O(n)

**In interviews and coding rounds, people almost always say "Big-O" and mean the worst case.** Focus mainly on Big-O.

---

## 4. The 3 rules for finding Big-O

### Rule 1: Drop constants

```python
for i in range(n):
    print(i)
    print(i * 2)
```

This does 2n operations, but we write **O(n)**, not O(2n).

Why? When n is huge, the "2" does not change the shape of growth.

### Rule 2: Keep only the dominant (biggest) term

```python
for i in range(n):        # O(n)
    print(i)

for i in range(n):        # O(n^2)
    for j in range(n):
        print(i, j)
```

Total = n + n². For large n, n² is far bigger, so the answer is **O(n²)**.

Order of growth (small to big):

```
1  <  log n  <  n  <  n log n  <  n²  <  n³  <  2ⁿ  <  n!
```

### Rule 3: Different inputs need different variables

```python
def f(a, b):
    for x in a:      # a has n elements
        print(x)
    for y in b:      # b has m elements
        print(y)
```

This is **O(n + m)**, not O(n). And if the loops were nested, it would be **O(n × m)**.

> **Remember:** Sequential steps are **added**. Nested steps are **multiplied**.

---

## 5. Common complexities with examples

### O(1): Constant time

The work does not depend on input size.

```python
def get_first(arr):
    return arr[0]

x = arr[5]              # index access
d["key"] = 10           # dictionary insert (average)
```

### O(log n): Logarithmic time

The input is **cut down by a factor** (usually half) every step.

```python
def binary_search(arr, target):
    lo, hi = 0, len(arr) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1
```

**Dry run:** `arr = [1, 3, 5, 7, 9, 11]`, `target = 7`

| Step | lo | hi | mid | arr[mid] | Decision |
|---|---|---|---|---|---|
| 1 | 0 | 5 | 2 | 5 | 5 < 7, go right, lo = 3 |
| 2 | 3 | 5 | 4 | 9 | 9 > 7, go left, hi = 3 |
| 3 | 3 | 3 | 3 | 7 | Found! |

For 1,000,000 elements, it needs only about **20 steps** (because 2²⁰ ≈ 1,000,000).

**Clue:** if a loop does `i *= 2` or `n //= 2`, it is **log n**.

```python
i = 1
while i < n:
    i *= 2      # 1, 2, 4, 8, 16 ... so log2(n) iterations
```

### O(n): Linear time

You touch each element a constant number of times.

```python
def find_max(arr):
    best = arr[0]
    for x in arr:
        if x > best:
            best = x
    return best
```

### O(n log n): Linearithmic time

Typical of **efficient sorting** and "divide and conquer with linear merging".

```python
arr.sort()              # Python's Timsort: O(n log n)
sorted(arr)             # also O(n log n)
```

Also: merge sort, heap sort, and "loop of n with a log n step inside".

### O(n²): Quadratic time

Usually **a loop inside a loop**, both going over n.

```python
def has_duplicate_bruteforce(arr):
    n = len(arr)
    for i in range(n):
        for j in range(i + 1, n):
            if arr[i] == arr[j]:
                return True
    return False
```

Examples: bubble sort, selection sort, insertion sort (worst), checking all pairs.

### O(2ⁿ): Exponential time

Typical of **trying all subsets** or naive recursion that branches twice.

```python
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)    # two calls each time
```

This becomes unusable around n = 40 or so. (Dynamic Programming fixes this later.)

### O(n!): Factorial time

Trying **all permutations** of n items.

```python
from itertools import permutations
for p in permutations(arr):     # n! permutations
    pass
```

Only feasible for n up to about 10.

---

## 6. How fast do they grow?

Approximate number of operations:

| Complexity | n = 10 | n = 100 | n = 1,000 | n = 100,000 |
|---|---|---|---|---|
| O(1) | 1 | 1 | 1 | 1 |
| O(log n) | ~3 | ~7 | ~10 | ~17 |
| O(n) | 10 | 100 | 1,000 | 100,000 |
| O(n log n) | ~33 | ~664 | ~10,000 | ~1.7 million |
| O(n²) | 100 | 10,000 | 1,000,000 | 10,000,000,000 |
| O(2ⁿ) | 1,024 | ~10³⁰ | impossible | impossible |
| O(n!) | 3.6 million | impossible | impossible | impossible |

**Key lesson:** at n = 100,000, an O(n²) solution does about 10 billion operations and will definitely TLE, while O(n log n) does only about 1.7 million and finishes instantly.

---

## 7. How to analyse loops (step by step)

**Method:** For every loop, ask two things.
1. How many times does it run?
2. How much work is done per run?

Then **multiply** nested ones and **add** sequential ones.

### Pattern A: Single loop

```python
for i in range(n):
    # O(1) work
```
**O(n)**

### Pattern B: Nested loops, both full

```python
for i in range(n):
    for j in range(n):
        # O(1) work
```
**O(n²)**

### Pattern C: Inner loop depends on outer loop (triangle)

```python
for i in range(n):
    for j in range(i):
        # O(1) work
```

Inner runs: 0 + 1 + 2 + ... + (n-1) = n(n-1)/2, which is about n²/2. Drop the constant, so **O(n²)**.

### Pattern D: Loop with a constant inner loop

```python
for i in range(n):
    for j in range(100):     # 100 is a constant
        pass
```
Total = 100n, so **O(n)**. A constant bound does not count as "n".

### Pattern E: Loop that doubles or halves

```python
i = 1
while i < n:
    i *= 2
```
**O(log n)**

### Pattern F: Linear outer loop with log inner loop

```python
for i in range(n):
    j = 1
    while j < n:
        j *= 2
```
n times × log n = **O(n log n)**

### Pattern G: Sequential loops

```python
for i in range(n):        # O(n)
    pass
for j in range(n):        # O(n)
    pass
```
n + n = 2n, so **O(n)**

### Pattern H: Square-root loop

```python
i = 1
while i * i <= n:
    i += 1
```
Runs about √n times, so **O(√n)**. (Common in prime checking.)

### Pattern I: Hidden loops

Many lines **look** like O(1) but are actually loops inside.

```python
for i in range(n):
    if x in my_list:       # "in" on a list is O(n)!
        pass
```
This is **O(n × len(my_list))**, not O(n). (See Section 12.)

---

## 8. Analysing recursion

For recursion, ask three things:

1. How many calls are made in total?
2. How much work is done in each call (excluding the recursive calls)?
3. How deep does the recursion go? (This decides the space.)

### Example 1: Factorial (one call per level)

```python
def fact(n):
    if n <= 1:
        return 1
    return n * fact(n - 1)
```

- Calls: n (a chain)
- Work per call: O(1)
- **Time O(n)**, **Space O(n)** (call stack depth is n)

### Example 2: Binary search (recursive)

```python
def bs(arr, lo, hi, t):
    if lo > hi:
        return -1
    mid = (lo + hi) // 2
    if arr[mid] == t:
        return mid
    if arr[mid] < t:
        return bs(arr, mid + 1, hi, t)
    return bs(arr, lo, mid - 1, t)
```

- Problem halves each time, so **Time O(log n)**, **Space O(log n)** (stack depth).
- The iterative version has **O(1) space**. This is a classic follow-up question.

### Example 3: Naive Fibonacci (two calls per level)

```python
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)
```

Think of it as a tree where each node makes 2 children and the height is n.

```
              fib(4)
             /      \
        fib(3)      fib(2)
        /    \       /   \
    fib(2) fib(1) fib(1) fib(0)
    /   \
fib(1) fib(0)
```

- Total calls is about 2ⁿ, so **Time O(2ⁿ)**.
- Depth is n, so **Space O(n)**. (The stack only holds one root-to-leaf path at a time.)

### Example 4: Merge sort (divide and conquer)

```
T(n) = 2 * T(n/2) + O(n)
```

- Two half-sized problems, plus O(n) work to merge.
- The recursion tree has **log n levels**, and each level does **n work in total**.
- So **Time = O(n log n)**.
- The merge step needs a temporary array, so **Space O(n)**.

### Example 5: Halving with linear work

```python
def f(n):
    if n <= 1:
        return
    for i in range(n):      # O(n) work
        pass
    f(n // 2)
```

Work = n + n/2 + n/4 + ... which is about 2n, so **Time O(n)**. Stack depth is log n, so **Space O(log n)**.

### Quick recursion cheat sheet

| Recurrence | Example | Time |
|---|---|---|
| T(n) = T(n-1) + O(1) | factorial, linear recursion | O(n) |
| T(n) = T(n/2) + O(1) | binary search | O(log n) |
| T(n) = T(n/2) + O(n) | halving with linear work | O(n) |
| T(n) = 2T(n/2) + O(n) | merge sort | O(n log n) |
| T(n) = 2T(n-1) + O(1) | naive fibonacci, subsets | O(2ⁿ) |
| T(n) = T(n-1) + O(n) | selection sort style | O(n²) |

---

## 9. Space complexity

> **Space complexity = extra memory your algorithm uses as the input size grows.**

We usually count **auxiliary space**, which is the extra space beyond the input itself. (Some people include the input too. In a coding round, state clearly which one you mean.)

### What takes space?

| Thing | Space |
|---|---|
| A few variables (`i`, `total`, `max_val`) | O(1) |
| A list / set / dict with up to n items | O(n) |
| A 2D grid of n × m | O(n × m) |
| Recursion call stack of depth d | O(d) |
| A new string or list made by slicing / copying | O(size of copy) |

### Example 1: O(1) space

```python
def total(arr):
    s = 0
    for x in arr:
        s += x
    return s
```
Only the variable `s` is used. **Space O(1)**.

### Example 2: O(n) space

```python
def squares(arr):
    result = []
    for x in arr:
        result.append(x * x)     # result grows to n items
    return result
```
**Space O(n)**.

### Example 3: Recursion uses hidden space

```python
def sum_to(n):
    if n == 0:
        return 0
    return n + sum_to(n - 1)
```
No list is created, but there are n stacked calls, so **Space O(n)**.

**Python warning:** the default recursion limit is about **1000**. For a depth like 100,000, you get a `RecursionError`. Either convert to iteration or use `sys.setrecursionlimit(...)`, and be careful with it.

### Example 4: Slicing secretly costs space and time

```python
def f(arr):
    if not arr:
        return 0
    return arr[0] + f(arr[1:])     # arr[1:] copies the list every call!
```

Each slice copies up to n items, so the time becomes **O(n²)** and the space also becomes **O(n²)** in the worst case, because the copies are alive on the stack at once. Pass **indices** instead:

```python
def f(arr, i=0):
    if i == len(arr):
        return 0
    return arr[i] + f(arr, i + 1)   # O(n) time, O(n) stack space
```

### In-place vs not in-place

- **In-place** means O(1) extra space, modifying the input itself (for example, reversing an array with two pointers).
- **Not in-place** means creating a new structure (for example, `arr[::-1]` creates a new list).

```python
# In-place reverse: O(n) time, O(1) space
def reverse(arr):
    l, r = 0, len(arr) - 1
    while l < r:
        arr[l], arr[r] = arr[r], arr[l]
        l += 1
        r -= 1
```

---

## 10. Best, average, and worst case

The same algorithm can behave differently depending on the input.

| Algorithm | Best | Average | Worst |
|---|---|---|---|
| Linear search | O(1) | O(n) | O(n) |
| Binary search | O(1) | O(log n) | O(log n) |
| Bubble sort (with early stop) | O(n) | O(n²) | O(n²) |
| Insertion sort | O(n) | O(n²) | O(n²) |
| Quick sort | O(n log n) | O(n log n) | O(n²) |
| Merge sort | O(n log n) | O(n log n) | O(n log n) |
| Hash table lookup | O(1) | O(1) | O(n) |

**In coding rounds, always design for the worst case** because test cases are often built to hit it.

---

## 11. Amortized complexity

Some operations are usually cheap but **occasionally expensive**. Averaged over many operations, the cost per operation is small. This is called **amortized** cost.

### The classic example: `list.append()`

A Python list is a **dynamic array**. It keeps spare capacity.

- Most appends: just place the element, which is O(1).
- When full: allocate a bigger array and copy everything, which is O(n). This happens rarely, because the size grows by a factor each time.

Spread over many appends, the cost per append is **O(1) amortized**.

**Analogy:** You pay a big electricity bill once a year, but averaged per day it is small.

---

## 12. Python built-in operation costs

This is where many people lose marks. **A single innocent-looking line can hide an O(n) cost.**

### List

| Operation | Cost | Note |
|---|---|---|
| `lst[i]`, `lst[i] = x` | O(1) | |
| `lst.append(x)` | O(1) amortized | |
| `lst.pop()` | O(1) | removes from the end |
| `lst.pop(0)` | **O(n)** | everything shifts left |
| `lst.insert(0, x)` | **O(n)** | everything shifts right |
| `lst.insert(i, x)` / `lst.pop(i)` | O(n) | |
| `x in lst` | **O(n)** | linear scan |
| `lst.index(x)`, `lst.count(x)` | O(n) | |
| `lst.remove(x)` | O(n) | |
| `lst[a:b]` | O(b - a) | creates a copy |
| `lst.copy()`, `lst[:]` | O(n) | |
| `len(lst)` | O(1) | |
| `lst.sort()`, `sorted(lst)` | O(n log n) | Timsort |
| `min(lst)`, `max(lst)`, `sum(lst)` | O(n) | |
| `lst.reverse()` | O(n) | |
| `lst + other` | O(n + m) | new list |

### Dictionary and set (hash based)

| Operation | Average | Worst |
|---|---|---|
| `d[key]`, `d[key] = v` | O(1) | O(n) |
| `key in d`, `x in s` | O(1) | O(n) |
| `s.add(x)`, `s.remove(x)` | O(1) | O(n) |
| iterate all items | O(n) | O(n) |
| `len(d)` | O(1) | O(1) |

The worst case happens only with many hash collisions, which is rare. Treat these as **O(1)** in coding rounds.

> **Big takeaway:** `x in list` is O(n), but `x in set` is O(1). Converting a list to a set is one of the most common optimizations.

### Deque (`from collections import deque`)

| Operation | Cost |
|---|---|
| `append`, `appendleft` | O(1) |
| `pop`, `popleft` | O(1) |
| `dq[i]` (middle access) | O(n) |

Use `deque` for queues (BFS). **Never use `list.pop(0)` as a queue.**

### Heap (`import heapq`)

| Operation | Cost |
|---|---|
| `heappush(h, x)` | O(log n) |
| `heappop(h)` | O(log n) |
| `h[0]` (peek min) | O(1) |
| `heapify(lst)` | **O(n)** |

### String

Strings in Python are **immutable** (cannot be changed in place).

| Operation | Cost | Note |
|---|---|---|
| `s[i]` | O(1) | |
| `len(s)` | O(1) | |
| `s[a:b]` | O(b - a) | copy |
| `s + t` | O(len(s) + len(t)) | new string |
| `s += ch` inside a loop | up to **O(n²)** in total | each step may copy the whole string |
| `"".join(list_of_strings)` | O(total length) | the right way to build strings |
| `s[::-1]` | O(n) | |
| `sub in s` | O(n × m) worst case | substring search |
| `s.split()` | O(n) | |

**Wrong way (slow):**

```python
s = ""
for ch in chars:
    s += ch            # may be O(n^2) overall
```

**Right way:**

```python
parts = []
for ch in chars:
    parts.append(ch)
s = "".join(parts)     # O(n) overall
```

(CPython sometimes optimizes `s += ch`, but you should not rely on it in a coding round.)

---

## 13. Constraints to complexity cheat sheet

Most coding platforms give limits like `1 ≤ n ≤ 10⁵`. **Use the limit to guess the intended complexity.**

A safe assumption for Python is roughly **10⁷ simple operations per second** (Python is slower than C++ or Java). Time limits are usually 1 to 2 seconds per test, but check the platform.

| Constraint on n | Target complexity | Typical techniques |
|---|---|---|
| n ≤ 10 | O(n!) | permutations, brute force |
| n ≤ 20 | O(2ⁿ) | subsets, bitmask, backtracking |
| n ≤ 500 | O(n³) | triple loops, Floyd-Warshall, some DP |
| n ≤ 5,000 | O(n²) | double loops, 2D DP |
| n ≤ 10⁵ | **O(n log n)** | sorting, heap, binary search, segment tree |
| n ≤ 10⁶ | **O(n)** | hashing, two pointers, sliding window, prefix sums |
| n ≤ 10⁹ or more | O(log n) or O(1) | binary search on answer, math formula |

**Pro tip:** If n = 10⁵ and your idea is O(n²), stop and look for a better idea. The constraint is a hint.

---

## 14. Time vs space trade-off (Two Sum)

**Problem:** Given an array and a target, return indices of two numbers that add up to the target.

### Approach 1: Brute force

```python
def two_sum_brute(nums, target):
    n = len(nums)
    for i in range(n):
        for j in range(i + 1, n):
            if nums[i] + nums[j] == target:
                return [i, j]
    return []
```

**Time O(n²)**, **Space O(1)**

### Approach 2: Hash map (spend memory to save time)

```python
def two_sum_hash(nums, target):
    seen = {}                          # value -> index
    for i, x in enumerate(nums):
        need = target - x
        if need in seen:               # O(1) lookup
            return [seen[need], i]
        seen[x] = i
    return []
```

**Time O(n)**, **Space O(n)**

We used **extra space to buy speed**. This trade-off appears in many problems. In a coding round, the faster solution is almost always preferred unless memory is restricted.

---

## 15. Common mistakes

1. **Counting the constant:** saying O(2n) or O(n + 5). Write O(n).
2. **Forgetting hidden loops:** `in` on a list, `sum()`, `min()`, `max()`, slicing, `sorted()`, `.count()`, `.index()`.
3. **Using `list.pop(0)` or `insert(0, x)` in a loop:** this makes the code O(n²). Use `deque`.
4. **Building a string with `+=` in a loop:** use `"".join(...)`.
5. **Ignoring recursion stack space:** recursive DFS on a tree of height h uses O(h) space.
6. **Using one variable for two inputs:** write O(n × m), not O(n²), when the two sizes differ.
7. **Saying nested loops are always O(n²):** check the bounds. A constant inner loop gives O(n). An `i *= 2` inner loop gives O(n log n).
8. **Saying "sorting is free":** sorting adds O(n log n) time. If you sort first, your total is at least O(n log n).
9. **Assuming dict/set operations are always O(1):** they are average O(1). Mention this if asked.
10. **Not stating space complexity:** interviewers and graders often expect both time **and** space.

---

## 16. Practice questions with answers

Try each yourself first, then check.

**Q1.**
```python
for i in range(n):
    for j in range(n):
        print(i, j)
```
<details><summary>Answer</summary>

**O(n²)** time, O(1) space.
</details>

**Q2.**
```python
i = 1
while i < n:
    i *= 3
```
<details><summary>Answer</summary>

**O(log n)**. The base of the log (3) is a constant, so we drop it.
</details>

**Q3.**
```python
for i in range(n):
    for j in range(i, n):
        pass
```
<details><summary>Answer</summary>

Inner runs n, n-1, ..., 1, which is n(n+1)/2, so **O(n²)**.
</details>

**Q4.**
```python
for i in range(n):
    for j in range(100):
        pass
```
<details><summary>Answer</summary>

**O(n)**. The inner loop is a constant 100.
</details>

**Q5.**
```python
def f(n):
    if n <= 1:
        return 1
    return f(n - 1) + f(n - 1)
```
<details><summary>Answer</summary>

Time **O(2ⁿ)** (two calls per level, depth n). Space **O(n)** (stack depth).
</details>

**Q6.**
```python
for i in range(n):
    j = i
    while j > 0:
        j //= 2
```
<details><summary>Answer</summary>

The inner loop runs about log(i) times, so total is **O(n log n)**.
</details>

**Q7.**
```python
def f(n):
    if n <= 1:
        return
    for i in range(n):
        pass
    f(n // 2)
```
<details><summary>Answer</summary>

n + n/2 + n/4 + ... ≈ 2n, so Time **O(n)**, Space **O(log n)** (stack).
</details>

**Q8.**
```python
for ch in arr:
    result = result + ch      # result is a string
```
<details><summary>Answer</summary>

Each concatenation can copy the whole string, so up to **O(n²)** overall. Use `"".join(arr)` for **O(n)**.
</details>

**Q9.** `a` has n items and `b` has m items.
```python
for x in a:
    if x in b:        # b is a list
        count += 1
```
<details><summary>Answer</summary>

**O(n × m)**. If `b` is converted to a set first, it becomes **O(n + m)** time with O(m) space.
</details>

**Q10.**
```python
def is_prime(n):
    if n < 2:
        return False
    i = 2
    while i * i <= n:
        if n % i == 0:
            return False
        i += 1
    return True
```
<details><summary>Answer</summary>

**O(√n)** time, O(1) space.
</details>

**Q11.** What is the time and space complexity of this?
```python
def f(arr):
    arr.sort()
    return arr[len(arr) // 2]
```
<details><summary>Answer</summary>

Time **O(n log n)** because of the sort. Space is O(1) extra in terms of your own variables, although Timsort can use up to O(n) auxiliary space internally. State "O(1) extra, ignoring the sort's internal buffer" or just say O(n) to be safe.
</details>

**Q12.** A friend says: "My solution loops over the array, and for each element it calls `lst.pop(0)`. It's O(n)." Is that right?

<details><summary>Answer</summary>

No. `pop(0)` is O(n) itself, so the total is **O(n²)**. Use `collections.deque` and `popleft()` for O(1).
</details>

---

## 17. Quick revision sheet

### Rules
- Drop constants. Keep the biggest term.
- Sequential, so **add**. Nested, so **multiply**.
- Halving or doubling, so **log n**.
- Different inputs, so different variables (n, m).

### Order of growth
```
O(1) < O(log n) < O(√n) < O(n) < O(n log n) < O(n²) < O(n³) < O(2ⁿ) < O(n!)
```

### Common algorithms

| Algorithm | Time | Space |
|---|---|---|
| Linear search | O(n) | O(1) |
| Binary search (iterative) | O(log n) | O(1) |
| Binary search (recursive) | O(log n) | O(log n) |
| Bubble / Selection / Insertion sort | O(n²) | O(1) |
| Merge sort | O(n log n) | O(n) |
| Quick sort | O(n log n) avg, O(n²) worst | O(log n) avg |
| Heap sort | O(n log n) | O(1) |
| Python `sort()` | O(n log n) | O(n) worst |
| BFS / DFS on a graph | O(V + E) | O(V) |
| Hash map lookup / insert | O(1) avg | O(n) |
| Heap push / pop | O(log n) | O(1) |

### Before you submit, ask yourself
1. What is `n`? (Is there more than one variable?)
2. What is the worst case?
3. Does any line hide a loop? (`in list`, slicing, `sum`, `+=` on strings)
4. How deep is my recursion?
5. Does my complexity fit the constraints? (Use the table in Section 13.)

---

**Next topic suggestion:** Arrays and Strings. They are the base for almost every other pattern, and every concept above will now start showing up in real problems.
