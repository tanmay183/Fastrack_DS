# 🔁 Recursion & 🗂️ Hashing in DSA (Python) — Concepts, Patterns & 15 Problems

> Part A: Recursion (+ Backtracking). Part B: Hashing (hash tables, sets, dictionaries).
> 15 standard problems in total: **Problems 1–8 = Recursion / Backtracking**, **Problems 9–15 = Hashing**, each from easier to harder, with explanation, commented code and verified output.

---

## 📑 Table of Contents
**Part A — Recursion**
1. [What is Recursion?](#1-what-is-recursion)
2. [How the Call Stack Works](#2-how-the-call-stack-works)
3. [Types of Recursion](#3-types-of-recursion)
4. [Time Complexity of Recursion (Recurrences)](#4-time-complexity-of-recursion)
5. [Memoization (Recursion → DP)](#5-memoization-recursion--dp)
6. [Recursion Limits in Python](#6-recursion-limits-in-python)
7. [Backtracking (Choose → Explore → Un-choose)](#7-backtracking)
8. [Recursion Patterns & Recognition](#8-recursion-patterns--recognition)

**Part B — Hashing**
9. [What is Hashing?](#9-what-is-hashing)
10. [Collisions: Chaining & Open Addressing](#10-collisions-chaining--open-addressing)
11. [Python `dict`, `set`, `Counter`, `defaultdict`](#11-python-dict-set-counter-defaultdict)
12. [Hashing Patterns](#12-hashing-patterns)

**Problems**
13. [15 Standard Problems](#13-15-standard-problems)
14. [Cheat Sheet, Mistakes & Practice List](#14-cheat-sheet-mistakes--practice-list)

---

# PART A — RECURSION

## 1. What is Recursion?

**Recursion** = a function that **calls itself** to solve a smaller version of the same problem.

Every recursive function needs:
1. **Base case** — the smallest input answered directly (stops the recursion). *Without it → infinite recursion → `RecursionError`.*
2. **Recursive case** — break the problem into smaller sub-problem(s) and call the same function.
3. **Progress** — every call must move closer to the base case.

**Think like this (the 3-step "leap of faith"):**
1. *Define* exactly what `f(n)` returns.
2. *Trust* that `f(smaller)` already works.
3. *Combine*: express `f(n)` using `f(smaller)` + a little work.

Example: `factorial(n) = n × factorial(n-1)`, with `factorial(0) = 1`.

```python
def factorial(n, depth=0):
    indent = "  " * depth                          # visual indentation = call-stack depth
    print(f"{indent}factorial({n}) called")        # show the call
    if n == 0 or n == 1:                           # BASE CASE: smallest problem, answer directly
        print(f"{indent}base case -> 1")
        return 1
    result = n * factorial(n - 1, depth + 1)       # RECURSIVE CASE: trust factorial(n-1)
    print(f"{indent}factorial({n}) returns {result}")  # runs while the stack unwinds
    return result

print("Answer:", factorial(4))
```
**Output**
```
factorial(4) called
  factorial(3) called
    factorial(2) called
      factorial(1) called
      base case -> 1
    factorial(2) returns 2
  factorial(3) returns 6
factorial(4) returns 24
Answer: 24
```

---

## 2. How the Call Stack Works

Each function call creates a **stack frame** (parameters, local variables, return address) pushed on the **call stack**. When a call returns, its frame is popped.

```
factorial(4)          <- bottom of the stack (called first)
  factorial(3)
    factorial(2)
      factorial(1)    <- top: base case reached, returns 1 first
```
Then the stack **unwinds**: `1 → 2×1=2 → 3×2=6 → 4×6=24`.

**Space complexity of recursion** = **maximum depth of the stack** (n for factorial, log n for binary-search-style recursion).

---

## 3. Types of Recursion

| Type | Meaning | Example |
|---|---|---|
| **Direct** | function calls itself | `factorial` |
| **Indirect** | A calls B, B calls A | `is_even` ↔ `is_odd` |
| **Linear** | one recursive call per level | factorial, sum of list |
| **Tree / multiple** | two or more calls per level | Fibonacci, subsets |
| **Tail recursion** | recursive call is the **last** action | accumulator-style factorial |
| **Head recursion** | work happens **after** the recursive call | print numbers in reverse |
| **Nested** | argument itself is a recursive call | Ackermann |

> ⚠️ **Python does not optimise tail recursion** (no tail-call elimination), so deep tail recursion still overflows. Convert to a loop when depth is large.

```python
def is_even(n):                                   # indirect recursion: two functions call each other
    return True if n == 0 else is_odd(n - 1)

def is_odd(n):
    return False if n == 0 else is_even(n - 1)

def print_down(n):                                # tail-style: action BEFORE the recursive call
    if n == 0:
        return
    print(n, end=" ")                             # print first ...
    print_down(n - 1)                             # ... then recurse

def print_up(n):                                  # head-style: action AFTER the recursive call
    if n == 0:
        return
    print_up(n - 1)                               # recurse first ...
    print(n, end=" ")                             # ... print while unwinding

print(is_even(10), is_odd(7))
print_down(5); print()
print_up(5); print()
```
**Output**
```
True True
5 4 3 2 1 
1 2 3 4 5
```

---

## 4. Time Complexity of Recursion

Write a **recurrence** `T(n)` and solve it (draw the **recursion tree**: work per level × number of levels).

| Recurrence | Solution | Example |
|---|---|---|
| `T(n) = T(n-1) + O(1)` | O(n) | factorial, sum of list |
| `T(n) = T(n-1) + O(n)` | O(n²) | selection-sort style, quicksort worst case |
| `T(n) = T(n/2) + O(1)` | O(log n) | binary search, fast power |
| `T(n) = 2T(n/2) + O(n)` | O(n log n) | merge sort, quicksort average |
| `T(n) = 2T(n/2) + O(1)` | O(n) | tree traversal, max of array by halving |
| `T(n) = 2T(n-1) + O(1)` | **O(2ⁿ)** | subsets, naive Tower of Hanoi |
| `T(n) = T(n-1) + T(n-2)` | **O(φⁿ) ≈ O(1.6ⁿ)** | naive Fibonacci |
| `T(n) = n·T(n-1)` | **O(n!)** | permutations |

**Master theorem (quick form)** for `T(n) = aT(n/b) + O(nᵈ)`:
- `d > log_b(a)` → O(nᵈ) · `d = log_b(a)` → O(nᵈ log n) · `d < log_b(a)` → O(n^(log_b a)).

---

## 5. Memoization (Recursion → DP)

**Problem:** naive Fibonacci recomputes the same sub-problems again and again (**overlapping sub-problems**).
**Fix:** store each result in a dictionary (hash table!) the first time → **memoization** = *top-down DP*. Time drops from O(2ⁿ) to O(n).

```python
calls_naive = 0                                    # counts how many calls the naive version makes
def fib_naive(n):
    global calls_naive
    calls_naive += 1
    if n <= 1:                                     # base cases: fib(0)=0, fib(1)=1
        return n
    return fib_naive(n - 1) + fib_naive(n - 2)     # two overlapping recursive calls

calls_memo = 0
memo = {}                                          # cache: n -> fib(n)  (a hash table)
def fib_memo(n):
    global calls_memo
    calls_memo += 1
    if n in memo:                                  # already solved -> O(1) lookup
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fib_memo(n - 1) + fib_memo(n - 2)    # solve once, store
    return memo[n]

print(fib_naive(20), "calls:", calls_naive)
print(fib_memo(20), "calls:", calls_memo)

from functools import lru_cache                    # Python's built-in memoization decorator
@lru_cache(maxsize=None)
def fib_cached(n):
    return n if n <= 1 else fib_cached(n - 1) + fib_cached(n - 2)
print(fib_cached(80))                              # instant, naive version would never finish
```
**Output**
```
6765 calls: 21891
6765 calls: 39
23416728348467685
```

---

## 6. Recursion Limits in Python

Python limits the recursion depth (default **1000** frames) to protect memory.

```python
import sys

print(sys.getrecursionlimit())                     # default limit

def depth(n):
    return 0 if n == 0 else 1 + depth(n - 1)       # recursion depth = n

print(depth(500))                                  # safe
try:
    print(depth(5000))                             # too deep for the default limit
except RecursionError:
    print("RecursionError: maximum recursion depth exceeded")
sys.setrecursionlimit(10000)                       # raise the limit (use carefully, can crash the interpreter)
print(depth(5000))                                 # works now
```
**Output**
```
1000
500
RecursionError: maximum recursion depth exceeded
5000
```
**Tips:** For input sizes up to ~10⁵ use an **iterative** solution (explicit stack/loop) or raise the limit; always make sure the base case is reachable.

---

## 7. Backtracking

**Backtracking** = recursion that builds a solution **step by step**, and **undoes** (backtracks) a choice when it leads to a dead end. It explores the *decision tree* of all possibilities, **pruning** branches that cannot work.

**Template (memorise)**
```py
def backtrack(state, choices):
    if is_solution(state):            # goal reached
        save a COPY of state          # res.append(path[:])
        return
    for choice in choices:
        if not valid(choice):         # PRUNE: skip impossible choices early
            continue
        make(choice)                  # 1. CHOOSE   (path.append)
        backtrack(new_state, ...)     # 2. EXPLORE  (recurse)
        undo(choice)                  # 3. UN-CHOOSE (path.pop)
```

**Typical problem types and their shape**
| Problem | Decision at each step | Recursion shape |
|---|---|---|
| Subsets | include / exclude element | `for i in range(start, n)` |
| Permutations | pick any unused element | `used[]` array |
| Combination sum | pick element (reuse allowed) | pass `start` index |
| Parentheses | add `(` or `)` if valid | counters |
| N-Queens | choose a column in this row | sets for column/diagonals |
| Sudoku / Word search | try a value / direction | mark visited, then unmark |

**Complexity:** usually exponential (O(2ⁿ), O(n!)), pruning makes it practical.

---

## 8. Recursion Patterns & Recognition

| Clue in the problem | Technique |
|---|---|
| "generate all …", "list all possible …" | backtracking |
| Sequence defined by itself (`f(n)` from `f(n-1)`) | simple recursion / DP |
| Split array/string in halves, combine | divide & conquer (merge sort, max subarray) |
| Tree / graph traversal | DFS recursion |
| Repeated sub-problems | memoization |
| "number of ways", "min/max cost" with choices | recursion + memo → DP |
| Nested structure (brackets, JSON) | recursion / stack |

---

# PART B — HASHING

## 9. What is Hashing?

**Hashing** maps a **key** to an **index** using a **hash function**, so that data can be stored and found in (average) **O(1)**.

```
key  ──hash function──▶  integer  ──% table_size──▶  bucket index
"cat" ───────────────▶  312      ───── % 5 ───────▶  2
```

**Hash table** = array of buckets + hash function + collision handling.

**A good hash function**
1. **Deterministic** — same key → same hash.
2. **Uniform** — spreads keys evenly (few collisions).
3. **Fast** — O(length of key).

**Load factor** `α = n / table_size`. When α gets high (Python resizes around 2/3 full) the table **rehashes** into a bigger array to keep operations O(1).

| Operation | Average | Worst (many collisions) |
|---|---|---|
| Insert / Search / Delete | **O(1)** | O(n) |
| Space | O(n) | O(n) |

**Hashable in Python:** immutable objects (int, str, tuple of hashables, frozenset). **Not hashable:** list, dict, set (mutable).

```python
print(hash(10))                                    # ints hash to themselves
print(hash("abc") == hash("abc"))                  # same input -> same hash (within one run)
print(hash((1, 2)) == hash((1, 2)))                # tuples of hashables are hashable
try:
    d = {[1, 2]: "value"}                          # a list as a key
except TypeError as e:
    print("TypeError:", e)                         # mutable objects can't be keys
d = {(1, 2): "grid cell"}                          # tuple key works -> great for coordinates
print(d[(1, 2)])
def simple_hash(key, size):                        # toy hash function: sum of ASCII codes mod size
    return sum(ord(c) for c in key) % size
print(simple_hash("cat", 5), simple_hash("dog", 5))
```
**Output**
```
10
True
True
TypeError: unhashable type: 'list'
grid cell
2 4
```

---

## 10. Collisions: Chaining & Open Addressing

Two different keys can land in the same bucket → **collision** (unavoidable by the pigeonhole principle).

### 10.1 Separate Chaining
Each bucket holds a **list** (or linked list) of all entries that hash there. Lookup = hash → scan that small list.

```python
def simple_hash(key, size):
    return sum(ord(c) for c in key) % size         # toy hash (anagrams always collide!)

size = 5
table = [[] for _ in range(size)]                  # 5 buckets, each a list (chain)
for k in ["cat", "act", "tac", "dog", "god"]:
    table[simple_hash(k, size)].append(k)          # collisions are simply appended to the chain
print(table)                                       # same letters -> same sum -> same bucket
```
**Output**
```
[[], [], ['cat', 'act', 'tac'], [], ['dog', 'god']]
```

### 10.2 Open Addressing (all entries inside the array itself)
On collision, **probe** for another free slot:
- **Linear probing:** `i, i+1, i+2, …` (simple, but causes clustering)
- **Quadratic probing:** `i, i+1², i+2², …`
- **Double hashing:** step size from a second hash function

Deletion needs a **tombstone** marker so later probes don't stop early. **Python's dict and set use open addressing** (with a perturbation probe sequence).

```python
def insert_linear_probing(table, key):
    size = len(table)
    idx = key % size                               # home slot
    while table[idx] is not None:                  # slot occupied -> collision
        idx = (idx + 1) % size                     # probe next slot (wrap around)
    table[idx] = key                               # first free slot found
    return idx

table = [None] * 7                                 # table of 7 slots
for k in [10, 20, 15, 7, 14]:                      # 7 and 14 collide at slot 0; 15 sits at slot 1
    insert_linear_probing(table, k)
print(table)
```
**Output**
```
[7, 15, 14, 10, None, None, 20]
```

### 10.3 Comparison
| | Chaining | Open addressing |
|---|---|---|
| Memory | extra pointers/lists | one array, cache-friendly |
| Load factor | can exceed 1 | must stay < 1 |
| Delete | easy | needs tombstones |
| Worst case | O(n) (one long chain) | O(n) (long probe sequences) |

---

## 11. Python `dict`, `set`, `Counter`, `defaultdict`

| Structure | Use | Key operations (avg O(1)) |
|---|---|---|
| `dict` | key → value | `d[k]`, `d.get(k, default)`, `k in d`, `del d[k]`, `d.items()` |
| `set` | unique items, membership | `add`, `remove`, `in`, `a & b`, `a \| b`, `a - b`, `a ^ b` |
| `Counter` | frequency map | `Counter(arr)`, `.most_common(k)` |
| `defaultdict` | auto-create default values | `defaultdict(list)`, `defaultdict(int)` |
| `OrderedDict` | ordered + `move_to_end` (LRU cache) | `popitem(last=False)` |

> Since Python 3.7 a `dict` **preserves insertion order**, but a `set` does **not** guarantee order.

```python
from collections import Counter, defaultdict

d = {"a": 1, "b": 2}
d["c"] = 3                                         # insert / update
print(d.get("z", 0))                               # safe lookup with a default (no KeyError)
print("a" in d, len(d))                            # membership on keys is O(1)
for k, v in d.items():                             # iterate key-value pairs
    print(k, v, end=" | ")
print()
s = {1, 2, 3}
t = {3, 4}
print(s & t, s | t, s - t, s ^ t)                  # intersection, union, difference, symmetric difference
cnt = Counter("mississippi")                       # frequency of each character
print(cnt)
print(cnt.most_common(2))                          # two most frequent
groups = defaultdict(list)                         # missing key -> empty list automatically
for w in ["eat", "tea", "tan"]:
    groups["".join(sorted(w))].append(w)           # group words by sorted-letter key
print(dict(groups))
freq = defaultdict(int)                            # missing key -> 0
for x in [1, 1, 2]:
    freq[x] += 1                                   # no need to check "if x in freq"
print(dict(freq))
```
**Output**
```
0
True 3
a 1 | b 2 | c 3 | 
{3} {1, 2, 3, 4} {1, 2} {1, 2, 4}
Counter({'i': 4, 's': 4, 'p': 2, 'm': 1})
[('i', 4), ('s', 4)]
{'aet': ['eat', 'tea'], 'ant': ['tan']}
{1: 2, 2: 1}
```

---

## 12. Hashing Patterns

| # | Pattern | Idea | Typical problems |
|---|---|---|---|
| 1 | **Seen set / duplicates** | `if x in seen` before adding | contains duplicate, first repeated element, intersection |
| 2 | **Frequency map** | `Counter` / dict of counts | anagrams, majority, top-K frequent |
| 3 | **Complement lookup** | store what you've seen, look for `target - x` | two sum, 4Sum II |
| 4 | **Prefix sum + hash map** | map prefix value → count/first index | subarray sum = k, longest balanced subarray |
| 5 | **Canonical key grouping** | key = sorted word / count tuple | group anagrams |
| 6 | **Bijection (two maps)** | `a→b` and `b→a` consistent | isomorphic strings, word pattern |
| 7 | **Set as O(1) membership** | convert list to set first | longest consecutive sequence |
| 8 | **Cache / memo** | dict stores computed answers | memoized recursion, LRU cache |
| 9 | **Rolling hash** | O(1) window hash updates | Rabin–Karp, repeated DNA sequences |
| 10 | **Sliding window + map** | counts inside window | longest substring with k distinct |

**Rule of thumb:** if the brute force is O(n²) because of "for each element, search for something", replace the inner search with a **hash lookup** → O(n).

---

## 13. 15 Standard Problems

# 🅰️ Recursion & Backtracking (Problems 1–8)

---

### 🟢 Problem 1 — Reverse a String & Palindrome Check Recursively (Easy)
**Idea:** `reverse(s) = reverse(s[1:]) + s[0]`. Palindrome: first and last must match, then check the inside. Base case: length ≤ 1.

```python
def reverse_str(s):
    if len(s) <= 1:                                # base case: empty or one char is its own reverse
        return s
    return reverse_str(s[1:]) + s[0]               # reverse the rest, then put first char at the end

def is_pal(s):
    if len(s) <= 1:                                # base case: nothing left to compare
        return True
    if s[0] != s[-1]:                              # outer characters differ -> not a palindrome
        return False
    return is_pal(s[1:-1])                         # check the inner part

print(reverse_str("hello"))
print(is_pal("racecar"))
print(is_pal("abca"))
```
**Output**
```
olleh
True
False
```
**Complexity:** O(n) calls, but slicing makes it O(n²) total; use index parameters `(l, r)` for O(n). Space O(n) stack.

---

### 🟢 Problem 2 — Fibonacci with Memoization & Bottom-Up (Easy-Medium)
**Idea:** `fib(n) = fib(n-1) + fib(n-2)`. Cache results to avoid recomputation; or build from the bottom with two variables (O(1) space).

```python
def fib_memo(n, memo=None):
    if memo is None:
        memo = {}                                  # fresh cache for the first call
    if n <= 1:                                     # base cases
        return n
    if n in memo:                                  # reuse saved answer
        return memo[n]
    memo[n] = fib_memo(n - 1, memo) + fib_memo(n - 2, memo)  # compute once, save
    return memo[n]

def fib_iter(n):
    a, b = 0, 1                                    # fib(0), fib(1)
    for _ in range(n):
        a, b = b, a + b                            # slide the pair forward
    return a

print(fib_memo(10))
print(fib_memo(50))
print(fib_iter(50))                                # matches the recursive result
```
**Output**
```
55
12586269025
12586269025
```
**Complexity:** memo O(n) time / O(n) space; iterative O(n) time / O(1) space. (Naive: O(2ⁿ).)

---

### 🟡 Problem 3 — Tower of Hanoi (Medium)
**Idea:** To move `n` disks from `src` to `dst`: (1) move top `n-1` to `aux`, (2) move the biggest disk to `dst`, (3) move the `n-1` disks from `aux` to `dst`. `T(n) = 2T(n-1) + 1 = 2ⁿ − 1` moves.

```python
def hanoi(n, src, aux, dst, moves):
    if n == 0:                                     # no disks -> nothing to do
        return
    hanoi(n - 1, src, dst, aux, moves)             # step 1: move n-1 disks out of the way (src -> aux)
    moves.append(f"{src}->{dst}")                  # step 2: move the largest disk
    hanoi(n - 1, aux, src, dst, moves)             # step 3: move n-1 disks on top of it (aux -> dst)

moves = []
hanoi(3, "A", "B", "C", moves)
print(moves)
print(len(moves))                                  # 2^3 - 1
m10 = []
hanoi(10, "A", "B", "C", m10)
print(len(m10))                                    # 2^10 - 1
```
**Output**
```
['A->C', 'A->B', 'C->B', 'A->C', 'B->A', 'B->C', 'A->C']
7
1023
```
**Complexity:** O(2ⁿ) time, O(n) stack.

---

### 🟡 Problem 4 — Generate Parentheses (Medium) · *Backtracking with pruning*
**Idea:** Build the string character by character. Add `(` if we still have openings left (`open < n`); add `)` only if it won't be unmatched (`close < open`). Invalid prefixes are never explored.

```python
def generate_parentheses(n):
    res = []
    def backtrack(cur, open_c, close_c):
        if len(cur) == 2 * n:                      # complete string -> it is valid by construction
            res.append(cur)
            return
        if open_c < n:                             # we may still open a bracket
            backtrack(cur + '(', open_c + 1, close_c)
        if close_c < open_c:                       # we may close only if an open one is waiting
            backtrack(cur + ')', open_c, close_c + 1)
    backtrack('', 0, 0)
    return res

print(generate_parentheses(3))
print(len(generate_parentheses(5)))                # Catalan number C5 = 42
```
**Output**
```
['((()))', '(()())', '(())()', '()(())', '()()()']
42
```
**Complexity:** O(4ⁿ/√n) (Catalan number) outputs.

---

### 🟡 Problem 5 — Subsets (Power Set) (Medium) · *Include/skip via start index*
**Idea:** Every node of the recursion tree is a valid subset. At each call record the current path, then try adding each later element.

```python
def subsets(nums):
    res = []                                       # all subsets
    path = []                                      # current subset being built
    def backtrack(start):
        res.append(path[:])                        # save a COPY of the current subset
        for i in range(start, len(nums)):          # choose the next element from 'start' onward
            path.append(nums[i])                   # CHOOSE
            backtrack(i + 1)                       # EXPLORE (only later elements -> no duplicates)
            path.pop()                             # UN-CHOOSE (backtrack)
    backtrack(0)
    return res

print(subsets([1, 2, 3]))
print(len(subsets([1, 2, 3, 4])))                  # 2^4
```
**Output**
```
[[], [1], [1, 2], [1, 2, 3], [1, 3], [2], [2, 3], [3]]
16
```
**Complexity:** O(n · 2ⁿ) time, O(n) recursion depth.

---

### 🟡 Problem 6 — Permutations (Medium) · *Backtracking with a used array*
**Idea:** At each position try every element not yet used. When the path has length `n`, record it.

```python
def permutations(nums):
    res = []
    path = []
    used = [False] * len(nums)                     # used[i] = element i is already in path
    def backtrack():
        if len(path) == len(nums):                 # complete permutation
            res.append(path[:])
            return
        for i in range(len(nums)):
            if used[i]:                            # skip elements already placed
                continue
            used[i] = True                         # CHOOSE
            path.append(nums[i])
            backtrack()                            # EXPLORE
            path.pop()                             # UN-CHOOSE
            used[i] = False
    backtrack()
    return res

print(permutations([1, 2, 3]))
print(permutations([0, 1]))
```
**Output**
```
[[1, 2, 3], [1, 3, 2], [2, 1, 3], [2, 3, 1], [3, 1, 2], [3, 2, 1]]
[[0, 1], [1, 0]]
```
**Complexity:** O(n · n!) time, O(n) extra space.

---

### 🔴 Problem 7 — Combination Sum (Medium-Hard) · *Reuse allowed, pruning*
**Idea:** Pick numbers (unlimited reuse) that sum to `target`. Sort candidates, pass `start` so combinations aren't repeated in different orders, and break when a candidate exceeds the remaining target (prune).

```python
def combination_sum(candidates, target):
    candidates.sort()                              # sorted so we can stop early
    res, path = [], []
    def backtrack(start, remaining):
        if remaining == 0:                         # exact sum reached
            res.append(path[:])
            return
        for i in range(start, len(candidates)):
            c = candidates[i]
            if c > remaining:                      # PRUNE: this and all later (bigger) numbers are too large
                break
            path.append(c)                         # CHOOSE
            backtrack(i, remaining - c)            # EXPLORE, pass i (not i+1) -> same number can be reused
            path.pop()                             # UN-CHOOSE
    backtrack(0, target)
    return res

print(combination_sum([2, 3, 6, 7], 7))
print(combination_sum([2, 3, 5], 8))
print(combination_sum([2], 1))
```
**Output**
```
[[2, 2, 3], [7]]
[[2, 2, 2, 2], [2, 3, 3], [3, 5]]
[]
```
**Complexity:** exponential, roughly O(2^(target/min)).

---

### 🔴 Problem 8 — N-Queens (Hard) · *Backtracking + hash sets*
**Idea:** Place one queen per row. A square `(r, c)` is attacked if its **column**, **diagonal (r − c)** or **anti-diagonal (r + c)** is already used. Three hash sets give O(1) conflict checks.

```python
def solve_n_queens(n):
    res = []
    cols, diag, anti = set(), set(), set()         # occupied columns / (r-c) diagonals / (r+c) anti-diagonals
    board = []                                     # board[r] = column of the queen in row r
    def backtrack(r):
        if r == n:                                 # all rows filled -> valid arrangement
            res.append(['.' * c + 'Q' + '.' * (n - c - 1) for c in board])
            return
        for c in range(n):
            if c in cols or (r - c) in diag or (r + c) in anti:   # square is attacked
                continue
            cols.add(c); diag.add(r - c); anti.add(r + c)         # CHOOSE
            board.append(c)
            backtrack(r + 1)                                      # EXPLORE next row
            cols.remove(c); diag.remove(r - c); anti.remove(r + c)  # UN-CHOOSE
            board.pop()
    backtrack(0)
    return res

for board in solve_n_queens(4):                    # both solutions for 4x4
    print(board)
print(len(solve_n_queens(6)))
print(len(solve_n_queens(8)))                      # famous 92 solutions
```
**Output**
```
['.Q..', '...Q', 'Q...', '..Q.']
['..Q.', 'Q...', '...Q', '.Q..']
4
92
```
**Complexity:** O(n!) worst case, heavily pruned in practice; O(n) space.

---

# 🅱️ Hashing (Problems 9–15)

---

### 🟢 Problem 9 — Contains Duplicate & Array Intersection (Easy) · *Seen set*
**Idea:** A `set` remembers what we have seen; membership is O(1). Intersection = elements in both sets.

```python
def contains_duplicate(nums):
    seen = set()                                   # values seen so far
    for x in nums:
        if x in seen:                              # O(1) membership test
            return True
        seen.add(x)                                # remember x
    return False

def intersection(a, b):
    return sorted(set(a) & set(b))                 # unique common elements (sorted for stable output)

print(contains_duplicate([1, 2, 3, 1]))
print(contains_duplicate([1, 2, 3, 4]))
print(intersection([1, 2, 2, 1], [2, 2]))
print(intersection([4, 9, 5], [9, 4, 9, 8, 4]))
```
**Output**
```
True
False
[2]
[4, 9]
```
**Complexity:** O(n) time, O(n) space. (Brute force O(n²), sorting O(n log n).)

---

### 🟢 Problem 10 — Word Pattern (Isomorphic Mapping) (Easy-Medium) · *Two maps (bijection)*
**Idea:** Each pattern letter must map to exactly one word **and** each word to exactly one letter. Keep both directions and check consistency.

```python
def word_pattern(pattern, s):
    words = s.split()                              # split sentence into words
    if len(pattern) != len(words):                 # different lengths can't match
        return False
    p2w, w2p = {}, {}                              # letter -> word, word -> letter
    for p, w in zip(pattern, words):
        if p in p2w and p2w[p] != w:               # letter already mapped to a different word
            return False
        if w in w2p and w2p[w] != p:               # word already mapped to a different letter
            return False
        p2w[p] = w                                 # record the mapping in both directions
        w2p[w] = p
    return True

print(word_pattern("abba", "dog cat cat dog"))
print(word_pattern("abba", "dog cat cat fish"))
print(word_pattern("abba", "dog dog dog dog"))     # two letters can't share one word
print(word_pattern("aaaa", "dog dog dog dog"))
```
**Output**
```
True
False
False
True
```
**Complexity:** O(n) time, O(n) space.

---

### 🟡 Problem 11 — Longest Consecutive Sequence (Medium) · *Set + "only start at sequence beginnings"*
**Idea:** Put all numbers in a set. Only start counting from a number `x` where `x-1` is **not** in the set (a sequence start). Then extend `x+1, x+2, …`. Each number is visited at most twice → O(n) instead of sorting O(n log n).

```python
def longest_consecutive(nums):
    s = set(nums)                                  # O(1) membership, also removes duplicates
    best = 0
    for x in s:
        if x - 1 not in s:                         # x is the START of a sequence
            y = x
            while y + 1 in s:                      # extend while the next number exists
                y += 1
            best = max(best, y - x + 1)            # length of this sequence
    return best

print(longest_consecutive([100, 4, 200, 1, 3, 2]))        # 1,2,3,4
print(longest_consecutive([0, 3, 7, 2, 5, 8, 4, 6, 0, 1]))  # 0..8
print(longest_consecutive([]))
```
**Output**
```
4
9
0
```
**Complexity:** O(n) time, O(n) space.

---

### 🟡 Problem 12 — Contiguous Array (Longest Subarray with Equal 0s and 1s) (Medium) · *Prefix balance + first-index map*
**Idea:** Treat `0` as `-1` and `1` as `+1`. A subarray with equal 0s and 1s has balance 0, i.e. the **same prefix balance** appears at two positions. Store the **first index** of each balance; the answer is the max distance between equal balances.

```python
def find_max_length(nums):
    first = {0: -1}                                # balance 0 "occurs" before the array starts (index -1)
    bal = 0                                        # running balance (+1 for 1, -1 for 0)
    best = 0
    for i, x in enumerate(nums):
        bal += 1 if x == 1 else -1                 # update balance
        if bal in first:                           # same balance seen earlier -> zero-sum subarray between
            best = max(best, i - first[bal])       # its length
        else:
            first[bal] = i                         # remember the FIRST position of this balance
    return best

print(find_max_length([0, 1]))
print(find_max_length([0, 1, 0]))
print(find_max_length([0, 0, 1, 0, 0, 0, 1, 1]))
```
**Output**
```
2
2
6
```
**Complexity:** O(n) time, O(n) space.

---

### 🟡 Problem 13 — 4Sum II (Medium) · *Meet in the middle with a hash map*
**Idea:** Count tuples `(i,j,k,l)` with `A[i]+B[j]+C[k]+D[l] = 0`. Brute force is O(n⁴). Store all sums `a+b` with counts, then for every `c+d` look up `-(c+d)` → O(n²).

```python
from collections import Counter

def four_sum_count(A, B, C, D):
    ab = Counter(a + b for a in A for b in B)      # how many ways to get each sum a+b
    total = 0
    for c in C:
        for d in D:
            total += ab[-(c + d)]                  # ways to complete the zero-sum (missing key -> 0)
    return total

print(four_sum_count([1, 2], [-2, -1], [-1, 2], [0, 2]))
print(four_sum_count([0], [0], [0], [0]))
```
**Output**
```
2
1
```
**Complexity:** O(n²) time, O(n²) space.

---

### 🔴 Problem 14 — Design HashMap from Scratch (Medium-Hard) · *Chaining*
**Idea:** Implement `put`, `get`, `remove` with an array of buckets; each bucket is a list of `[key, value]` pairs (separate chaining). This is exactly the theory from Section 10.

```python
class MyHashMap:
    def __init__(self, size=7):
        self.size = size                           # number of buckets (small on purpose to show collisions)
        self.buckets = [[] for _ in range(size)]   # each bucket = chain of [key, value]

    def _index(self, key):
        return hash(key) % self.size               # hash function -> bucket index

    def put(self, key, value):
        bucket = self.buckets[self._index(key)]
        for pair in bucket:                        # key already present -> update its value
            if pair[0] == key:
                pair[1] = value
                return
        bucket.append([key, value])                # otherwise append to the chain

    def get(self, key):
        bucket = self.buckets[self._index(key)]
        for k, v in bucket:                        # scan the (short) chain
            if k == key:
                return v
        return -1                                  # not found

    def remove(self, key):
        bucket = self.buckets[self._index(key)]
        for i, (k, v) in enumerate(bucket):
            if k == key:
                del bucket[i]                      # delete the pair from the chain
                return

m = MyHashMap()
m.put(1, "one")
m.put(8, "eight")                                  # 8 % 7 == 1 -> collides with key 1
m.put(15, "fifteen")                               # 15 % 7 == 1 -> same bucket again
print(m.buckets[1])                                # one bucket holding three colliding keys
print(m.get(1), m.get(8), m.get(99))
m.remove(8)
print(m.get(8))
m.put(1, "uno")                                    # overwrite existing key
print(m.get(1))
```
**Output**
```
[[1, 'one'], [8, 'eight'], [15, 'fifteen']]
one eight -1
-1
uno
```
**Complexity:** average O(1), worst O(n) per operation. Real implementations **resize** (rehash) when the load factor grows.

---

### 🔴 Problem 15 — LRU Cache (Hard) · *Hash map + ordering*
**Idea:** `get` and `put` in O(1) while evicting the **least recently used** item when full. A hash map gives O(1) lookup; an **ordered** structure tracks recency. In Python, `OrderedDict` is a hash map + doubly linked list (in interviews you may be asked to build the doubly linked list manually: `dict[key] → node`, move node to the front on every use, evict from the tail).

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.cap = capacity                        # maximum number of items
        self.cache = OrderedDict()                 # order: least recently used -> most recently used

    def get(self, key):
        if key not in self.cache:                  # miss
            return -1
        self.cache.move_to_end(key)                # mark as most recently used  O(1)
        return self.cache[key]

    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)            # updating also counts as a use
        self.cache[key] = value                    # insert / overwrite
        if len(self.cache) > self.cap:             # over capacity -> evict
            self.cache.popitem(last=False)         # remove the LEAST recently used (front)  O(1)

c = LRUCache(2)
c.put(1, 1)
c.put(2, 2)
print(c.get(1))                                    # 1 (key 1 becomes most recent)
c.put(3, 3)                                        # capacity exceeded -> evicts key 2
print(c.get(2))                                    # -1
c.put(4, 4)                                        # evicts key 1
print(c.get(1))                                    # -1
print(c.get(3))
print(c.get(4))
```
**Output**
```
1
-1
-1
3
4
```
**Complexity:** O(1) per operation, O(capacity) space.

---

## 14. Cheat Sheet, Mistakes & Practice List

### Problem → technique map
| # | Problem | Technique |
|---|---|---|
| 1 | Reverse string / palindrome | simple recursion |
| 2 | Fibonacci memo | recursion + hash memo |
| 3 | Tower of Hanoi | divide & conquer recursion |
| 4 | Generate Parentheses | backtracking + pruning |
| 5 | Subsets | backtracking (start index) |
| 6 | Permutations | backtracking (used array) |
| 7 | Combination Sum | backtracking (reuse + prune) |
| 8 | N-Queens | backtracking + hash sets |
| 9 | Contains Duplicate / Intersection | seen set |
| 10 | Word Pattern | two-way map |
| 11 | Longest Consecutive Sequence | set + sequence starts |
| 12 | Contiguous Array | prefix balance + first index |
| 13 | 4Sum II | meet-in-the-middle hash map |
| 14 | Design HashMap | chaining |
| 15 | LRU Cache | hash map + order |

### Common mistakes — Recursion
1. **Missing / unreachable base case** → infinite recursion.
2. Appending the **same list object** instead of a copy (`res.append(path)` → use `path[:]`).
3. Forgetting to **undo** the choice (`pop`, `remove`, `used[i] = False`).
4. Using **mutable default arguments** (`def f(memo={})`) — shared between calls.
5. Recomputing overlapping sub-problems (add memo).
6. Deep recursion on large `n` → convert to iteration or raise the limit.
7. Duplicates in input → sort and skip `if i > start and nums[i] == nums[i-1]`.

### Common mistakes — Hashing
1. Using a **list as a dictionary key** (unhashable) → use a tuple.
2. `d[key]` on a missing key → `KeyError`; use `.get()` or `defaultdict`.
3. **Modifying a dict/set while iterating** over it → iterate over a copy `list(d)`.
4. Assuming **set order** (not guaranteed).
5. Forgetting the initial `{0: 1}` / `{0: -1}` in prefix-sum maps.
6. Using floats as keys (precision issues).
7. Believing hashing is always O(1): adversarial/colliding keys → O(n).

### Interview talking points
- "Recursion is solving the problem using smaller instances of itself; space = max stack depth."
- "Backtracking = DFS over a decision tree with undo and pruning."
- "Memoization stores results in a hash map, turning exponential recursion into polynomial time."
- "A hash table gives average O(1) insert/search/delete using a hash function, handling collisions with chaining or open addressing."


