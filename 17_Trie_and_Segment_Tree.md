# Trie and Segment Tree — Complete Guide (Python) for Coding Rounds

> Two "special" data structures that appear again and again in interviews:
> - **Trie** → fast work with **words and prefixes**
> - **Segment Tree** (and **Fenwick Tree**) → fast **range queries with updates**
>
> Language: Python 3. All code is tested.
> Read Parts A and B once, then solve the 15 problems in Part C before reading the solutions.

---

## Table of Contents

**Part A — Trie**
1. What is a Trie and why do we need it?
2. Building a Trie (insert, search, startsWith, delete)
3. Complexity, memory, dict vs array
4. Trie patterns

**Part B — Segment Tree and Fenwick Tree**
5. The range-query problem
6. Segment Tree: build, query, update (recursive, to understand)
7. Iterative Segment Tree (short, fast, generic)
8. Lazy Propagation (range updates)
9. Fenwick Tree (BIT)
10. Segment tree patterns, coordinate compression, which structure to choose

**Part C — 15 Standard Interview Problems** (8 Trie + 7 Segment/Fenwick)

**Part D — Practice, mistakes, 7-day plan**

---

# PART A — TRIE

## 1. What is a Trie?

A **Trie** (say "try", also called *prefix tree*) is a tree where **each edge is a character**. A path from the root spells out a prefix; some nodes are marked "a word ends here".

Words: `cat`, `car`, `dog`, `do`

```
            (root)
            /    \
           c      d
           |      |
           a      o*          * = a word ends here ("do")
          / \     |
         t*  r*   g*
```

Each path from root to a `*` node is a stored word.

### Why not just use a set of words?

A hash set answers "is `car` in the set?" in O(L). But it cannot quickly answer:
- "Does **any** word start with `ca`?"
- "List all words starting with `ca`" (autocomplete)
- "Longest common prefix of all words"
- "Find words matching `c.t`" (wildcard)

A trie answers all of these in **O(length of the prefix)**, independent of how many words are stored.

---

## 2. Building a Trie

### 2.1 Node structure

Each node needs:
- `children`: mapping from character → child node (dict is easiest in Python)
- `is_end`: does a word finish here?

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False


class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end = True

    def _walk(self, s):
        """Follow the characters of s; return the final node or None."""
        node = self.root
        for ch in s:
            node = node.children.get(ch)
            if node is None:
                return None
        return node

    def search(self, word):
        node = self._walk(word)
        return node is not None and node.is_end

    def starts_with(self, prefix):
        return self._walk(prefix) is not None
```

How it works, for `insert("car")`: start at root, go/create `c`, then `a`, then `r`, mark `r` as end.
`search("ca")` → walk finds node `a`, but `is_end` is False → **not a word** (it is only a prefix).

### 2.2 Delete a word

Un-mark `is_end`, then prune nodes that are no longer useful (no children, not an end).

```python
def trie_delete(trie, word):
    def helper(node, i):
        if i == len(word):
            if not node.is_end:
                return False                 # word not present
            node.is_end = False
            return not node.children         # can this node be removed?
        ch = word[i]
        child = node.children.get(ch)
        if child is None:
            return False
        if helper(child, i + 1):
            del node.children[ch]
            return not node.is_end and not node.children
        return False
    helper(trie.root, 0)
```

### 2.3 Storing counts (very common in interviews)

Keep `pass_count` (how many words go through this node) and `end_count` (how many words end here). Then "how many words start with this prefix" is O(L).

```python
class CountTrie:
    def __init__(self):
        self.root = {}                       # node = dict; special keys below

    def insert(self, word):
        node = self.root
        for ch in word:
            node = node.setdefault(ch, {})
            node['#pass'] = node.get('#pass', 0) + 1
        node['#end'] = node.get('#end', 0) + 1

    def count_prefix(self, prefix):
        node = self.root
        for ch in prefix:
            node = node.get(ch)
            if node is None:
                return 0
        return node.get('#pass', 0)
```

(The "nested dict" style is the shortest to write in an interview.)

---

## 3. Complexity, Memory, dict vs array

| Operation | Time | Notes |
|---|---|---|
| insert(word) | O(L) | L = length of the word |
| search / startsWith | O(L) | independent of number of words |
| Space | O(total characters) worst case | up to `26 × total chars` pointers with arrays |

**dict vs array of 26:**
- `children = {}` → flexible (any character), memory-friendly. Preferred in Python.
- `children = [None] * 26` → faster index lookup `ord(ch) - ord('a')`, but wastes memory when sparse.

**Trie vs HashMap:** trie wins when you need prefix operations, ordering, or sharing common prefixes. HashMap wins for plain exact lookups (simpler, usually less memory).

---

## 4. Trie Patterns

| # | Pattern | Idea | Problems |
|---|---|---|---|
| 1 | **Basic prefix operations** | insert / search / startsWith / counts | Implement Trie, Map Sum Pairs, Implement Trie II |
| 2 | **Wildcard search** | on `'.'` branch into **all** children with DFS | Add and Search Words |
| 3 | **Trie + Backtracking on a grid** | build trie of dictionary, DFS the board following trie edges; prune | Word Search II |
| 4 | **Shortest/Longest prefix** | walk the trie while condition holds | Replace Words, Longest Common Prefix |
| 5 | **Store extra data on nodes** | e.g. top-3 words, frequency, hot sentences | Search Suggestions, Autocomplete System |
| 6 | **Binary (bitwise) Trie** | characters are bits `0/1`; greedy choose opposite bit | Maximum XOR of Two Numbers |
| 7 | **Reverse Trie** | insert words reversed to match **suffixes** | Stream of Characters, Palindrome Pairs |

---

# PART B — SEGMENT TREE AND FENWICK TREE

## 5. The Range-Query Problem

You have an array and many operations:
- **Query**: sum (or min / max / gcd...) of `arr[l..r]`
- **Update**: change `arr[i]` (later: add to a whole range)

| Approach | Query | Update |
|---|---|---|
| Plain loop | O(n) | O(1) |
| Prefix sums | O(1) | O(n) (rebuild) |
| **Segment Tree** | **O(log n)** | **O(log n)** |
| **Fenwick Tree (BIT)** | **O(log n)** | **O(log n)** |

With `n, q = 10⁵` the plain loop does 10¹⁰ operations (too slow); the tree does ~ 3.4 × 10⁶.
**Use a segment tree when you have BOTH queries and updates, many times.** If the array never changes, prefix sums (sum) or a sparse table (min/max) are simpler.

---

## 6. Segment Tree — the idea

Build a binary tree where **each node stores the answer for a segment** of the array. The root covers the whole array; each node splits its segment in two halves; leaves are single elements.

Array `[2, 1, 5, 3]`, sum tree:

```
                [0..3]=11
               /          \
         [0..1]=3        [2..3]=8
         /     \          /     \
     [0]=2    [1]=1    [2]=5    [3]=3
```

- **Query** `[1..2]`: combine only the few nodes that exactly cover the range (at most ~2 per level) → O(log n).
- **Update** `arr[2] = 10`: change the leaf, then recompute its ancestors up to the root → O(log n).

### 6.1 Recursive implementation (best for understanding)

Tree stored in an array: node `i` has children `2i` and `2i+1`. Allocate `4n` slots.

```python
class SegTreeRec:
    """Range sum, point update. Indices are 0-based, query is inclusive [l, r]."""
    def __init__(self, arr):
        self.n = len(arr)
        self.arr = arr[:]
        self.tree = [0] * (4 * max(1, self.n))
        if self.n:
            self._build(1, 0, self.n - 1)

    def _build(self, node, l, r):
        if l == r:
            self.tree[node] = self.arr[l]
            return
        mid = (l + r) // 2
        self._build(2 * node, l, mid)
        self._build(2 * node + 1, mid + 1, r)
        self.tree[node] = self.tree[2 * node] + self.tree[2 * node + 1]

    def update(self, idx, val):
        self._update(1, 0, self.n - 1, idx, val)

    def _update(self, node, l, r, idx, val):
        if l == r:
            self.tree[node] = val
            return
        mid = (l + r) // 2
        if idx <= mid:
            self._update(2 * node, l, mid, idx, val)
        else:
            self._update(2 * node + 1, mid + 1, r, idx, val)
        self.tree[node] = self.tree[2 * node] + self.tree[2 * node + 1]

    def query(self, ql, qr):
        return self._query(1, 0, self.n - 1, ql, qr)

    def _query(self, node, l, r, ql, qr):
        if qr < l or r < ql:            # completely outside -> neutral value
            return 0
        if ql <= l and r <= qr:         # completely inside -> use stored answer
            return self.tree[node]
        mid = (l + r) // 2              # partial overlap -> ask both children
        return (self._query(2 * node, l, mid, ql, qr) +
                self._query(2 * node + 1, mid + 1, r, ql, qr))
```

**The 3 cases of `query` — memorise them:**
1. **No overlap** → return the *identity* (0 for sum, +∞ for min, −∞ for max).
2. **Total overlap** → return the node's stored value.
3. **Partial overlap** → recurse into both children and combine.

Time: build `O(n)`, query/update `O(log n)`. Space `O(n)`.

---

## 7. Iterative Segment Tree (short and generic) — use this in interviews

A bottom-up version: array of size `2n`, leaves at `[n, 2n)`. Works for **any** `n` and any associative operation (sum, min, max, gcd, xor...). Query uses a **half-open** range `[l, r)`.

```python
class SegmentTree:
    def __init__(self, data, func, identity):
        """func: associative combine function. identity: neutral element."""
        self.n = len(data)
        self.func = func
        self.identity = identity
        self.t = [identity] * self.n + list(data)
        for i in range(self.n - 1, 0, -1):
            self.t[i] = func(self.t[2 * i], self.t[2 * i + 1])

    def update(self, i, value):                 # set data[i] = value
        i += self.n
        self.t[i] = value
        while i > 1:
            i >>= 1
            self.t[i] = self.func(self.t[2 * i], self.t[2 * i + 1])

    def query(self, l, r):                      # combine data[l .. r-1]
        left = right = self.identity
        l += self.n
        r += self.n
        while l < r:
            if l & 1:
                left = self.func(left, self.t[l])
                l += 1
            if r & 1:
                r -= 1
                right = self.func(self.t[r], right)
            l >>= 1
            r >>= 1
        return self.func(left, right)
```

Usage:

```python
import operator
INF = float('inf')

sum_tree = SegmentTree([2, 1, 5, 3], operator.add, 0)
min_tree = SegmentTree([2, 1, 5, 3], min, INF)
max_tree = SegmentTree([2, 1, 5, 3], max, -INF)

sum_tree.query(1, 3)      # 1 + 5 = 6
sum_tree.update(2, 10)
sum_tree.query(1, 3)      # 1 + 10 = 11
```

> Changing the problem from "sum" to "min" or "gcd" is just changing `func` and `identity`. That is why one template solves many problems.

---

## 8. Lazy Propagation (range updates)

**Problem:** update a **whole range** ("add 5 to every element in `l..r`") and query range sums. Updating each leaf is O(n).

**Idea:** be **lazy**. If a node's segment is fully inside the update range, update only that node and write a "pending" note (`lazy`) saying *"my children still owe this update"*. Push the note down only when we actually need to visit the children.

```python
class LazySegTree:
    """Range add, range sum. Inclusive indices [l, r]."""
    def __init__(self, arr):
        self.n = len(arr)
        self.sums = [0] * (4 * self.n)
        self.lazy = [0] * (4 * self.n)
        self._build(1, 0, self.n - 1, arr)

    def _build(self, node, l, r, arr):
        if l == r:
            self.sums[node] = arr[l]
            return
        mid = (l + r) // 2
        self._build(2 * node, l, mid, arr)
        self._build(2 * node + 1, mid + 1, r, arr)
        self.sums[node] = self.sums[2 * node] + self.sums[2 * node + 1]

    def _apply(self, node, l, r, v):            # apply "+v to everything" on this node
        self.sums[node] += v * (r - l + 1)
        self.lazy[node] += v

    def _push(self, node, l, r):                # give the pending note to the children
        if self.lazy[node]:
            mid = (l + r) // 2
            self._apply(2 * node, l, mid, self.lazy[node])
            self._apply(2 * node + 1, mid + 1, r, self.lazy[node])
            self.lazy[node] = 0

    def update(self, ql, qr, v):
        self._update(1, 0, self.n - 1, ql, qr, v)

    def _update(self, node, l, r, ql, qr, v):
        if qr < l or r < ql:
            return
        if ql <= l and r <= qr:
            self._apply(node, l, r, v)
            return
        self._push(node, l, r)
        mid = (l + r) // 2
        self._update(2 * node, l, mid, ql, qr, v)
        self._update(2 * node + 1, mid + 1, r, ql, qr, v)
        self.sums[node] = self.sums[2 * node] + self.sums[2 * node + 1]

    def query(self, ql, qr):
        return self._query(1, 0, self.n - 1, ql, qr)

    def _query(self, node, l, r, ql, qr):
        if qr < l or r < ql:
            return 0
        if ql <= l and r <= qr:
            return self.sums[node]
        self._push(node, l, r)
        mid = (l + r) // 2
        return (self._query(2 * node, l, mid, ql, qr) +
                self._query(2 * node + 1, mid + 1, r, ql, qr))
```

Both update and query are `O(log n)`.

**Rule for lazy trees:** always `_push` **before** going into children (in both update and query), and recompute the parent **after** the children return (in update).

Other lazy operations: range **assign** (set all to `x`), range add + range **max**, range flip. The only things that change are `_apply` and how the node value is recomputed.

---

## 9. Fenwick Tree (Binary Indexed Tree, BIT)

A much **shorter** structure for **prefix sums with point updates** (and anything that can be written as a prefix: counts, frequencies). About 10 lines, very fast.

Idea: `tree[i]` stores the sum of a block of elements ending at `i`; the block length is the lowest set bit of `i` (`i & -i`).

```python
class Fenwick:
    """1-indexed. add(i, delta) point update; prefix(i) = sum of elements 1..i."""
    def __init__(self, n):
        self.n = n
        self.tree = [0] * (n + 1)

    def add(self, i, delta):
        while i <= self.n:
            self.tree[i] += delta
            i += i & -i              # jump to the next block that includes i

    def prefix(self, i):
        s = 0
        while i > 0:
            s += self.tree[i]
            i -= i & -i              # remove lowest set bit
        return s

    def range_sum(self, l, r):       # inclusive, 1-indexed
        return self.prefix(r) - self.prefix(l - 1)
```

Time `O(log n)` for both. Space `O(n)`.

**Limits:** BIT is natural for sum/count. For min/max with updates or for lazy range updates use a segment tree.

---

## 10. Segment Tree Patterns and Choosing the Right Tool

| # | Pattern | Idea | Problems |
|---|---|---|---|
| 1 | **Range query + point update** | iterative generic tree | Range Sum Query – Mutable, RMQ with updates |
| 2 | **Tree over VALUES, not indices** (counting) | index = (compressed) value; count how many values are smaller/larger so far | Count of Smaller Numbers After Self, Reverse Pairs, LIS in O(n log n) |
| 3 | **Coordinate compression** | map huge values to ranks `0..k-1` using `sorted(set(...))` + `bisect` | Count of Range Sum, Falling Squares |
| 4 | **Lazy propagation** | range update + range query | Range Addition, Range Update Range Sum |
| 5 | **Dynamic (sparse) segment tree** | huge range (10⁹) → create nodes only when needed (dict) | My Calendar III |
| 6 | **Range assign + range max** | lazy "assign" tag | Falling Squares |
| 7 | **Merge-able nodes** | node stores a tuple (e.g. max prefix sum, total, ...) | Longest Consecutive 1s in range, Maximum Subarray in range |

### Choosing the tool

| Need | Use |
|---|---|
| Static array, many range sums | Prefix sums |
| Static array, many range min/max | Sparse table |
| Range sum / count + point updates | **Fenwick** (shortest) |
| Range min/max/gcd + point updates | **Segment tree** |
| Range update + range query | **Lazy segment tree** |
| Values up to 10⁹, few operations | Dynamic segment tree or compression |

---

# PART C — 15 STANDARD INTERVIEW PROBLEMS

Format: **Problem → Idea → Code → Complexity → Variants.**
Problems 1–8 are Trie. Problems 9–15 are Segment Tree / Fenwick.
Reuse classes `SegmentTree`, `LazySegTree`, `Fenwick` from Part B.

```python
import operator
from bisect import bisect_left, bisect_right
from collections import defaultdict, deque
```

---

## Problem 1 — Implement Trie (Prefix Tree) (LC 208) + counts (LC 1804) · Medium

**Problem:** Implement `insert`, `search`, `startsWith`. (Bonus from Trie II: `countWordsEqualTo`, `countWordsStartingWith`, `erase`.)

**Idea:** Section 2 — walk character by character; store `pass_count` and `end_count` on every node.

```python
class Trie:
    def __init__(self):
        self.root = {}

    def insert(self, word):
        node = self.root
        for ch in word:
            node = node.setdefault(ch, {})
            node['#p'] = node.get('#p', 0) + 1     # words passing through
        node['#e'] = node.get('#e', 0) + 1         # words ending here

    def _find(self, s):
        node = self.root
        for ch in s:
            node = node.get(ch)
            if node is None:
                return None
        return node

    def search(self, word):
        node = self._find(word)
        return node is not None and node.get('#e', 0) > 0

    def startsWith(self, prefix):
        return self._find(prefix) is not None

    def countWordsEqualTo(self, word):
        node = self._find(word)
        return node.get('#e', 0) if node else 0

    def countWordsStartingWith(self, prefix):
        node = self._find(prefix)
        return node.get('#p', 0) if node else 0

    def erase(self, word):                          # assumes the word exists
        node = self.root
        for ch in word:
            node = node[ch]
            node['#p'] -= 1
        node['#e'] -= 1
```

**Complexity:** every operation `O(L)`, space `O(total characters)`.
**Variants:** Map Sum Pairs (677), Design Underground System-like prefix problems.

---

## Problem 2 — Design Add and Search Words Data Structure (LC 211) · Medium · Wildcard

**Problem:** `addWord(word)`, and `search(pattern)` where `'.'` matches any single letter.

**Idea:** Normal trie. When the pattern character is `'.'`, **try every child** (DFS); otherwise follow the single matching child.

```python
class WordDictionary:
    def __init__(self):
        self.root = {}

    def addWord(self, word):
        node = self.root
        for ch in word:
            node = node.setdefault(ch, {})
        node['$'] = True                     # end marker

    def search(self, pattern):
        def dfs(node, i):
            if i == len(pattern):
                return '$' in node
            ch = pattern[i]
            if ch == '.':
                for key, child in node.items():
                    if key != '$' and dfs(child, i + 1):
                        return True
                return False
            if ch not in node:
                return False
            return dfs(node[ch], i + 1)
        return dfs(self.root, 0)
```

**Complexity:** add `O(L)`; search worst case `O(26^dots · L)` but usually tiny.
**Variants:** Wildcard matching with `*` needs DP instead.

---

## Problem 3 — Word Search II (LC 212) · Hard · Trie + Backtracking

**Problem:** Given a grid of letters and a list of words, find all words that can be formed by adjacent cells (no cell reused in one word).

**Idea:** Running Word Search (LC 79) once per word is too slow. Put **all words in a trie**, then DFS from each cell, moving only along edges that exist in the trie. A branch with no trie edge is cut immediately. Store the full word at its end node, remove it when found (no duplicates), and **prune empty branches** so the search gets faster over time.

```python
def find_words(board, words):
    root = {}
    for w in words:
        node = root
        for ch in w:
            node = node.setdefault(ch, {})
        node['$'] = w                       # store the word itself at its end

    R, C = len(board), len(board[0])
    res = []

    def dfs(r, c, parent):
        ch = board[r][c]
        node = parent[ch]
        if '$' in node:
            res.append(node.pop('$'))       # found; pop so it is not found twice
        board[r][c] = '#'                   # mark visited
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < R and 0 <= nc < C and board[nr][nc] in node:
                dfs(nr, nc, node)
        board[r][c] = ch                    # backtrack
        if not node:                        # prune exhausted branch
            del parent[ch]

    for r in range(R):
        for c in range(C):
            if board[r][c] in root:
                dfs(r, c, root)
    return res
```

**Complexity:** Time ≈ `O(R·C·4·3^(L-1))` worst case, but pruning makes it fast; Space `O(total chars)`.
**Variants:** Word Search (79), Boggle.

---

## Problem 4 — Longest Common Prefix (LC 14) · Easy · Trie walk

**Problem:** Longest common prefix of an array of strings.

**Idea:** Insert all strings; then walk from root while the current node has **exactly one child** and is **not the end of a word**. (Simpler alternative: compare the smallest and largest string — mention it.)

```python
def longest_common_prefix(strs):
    if not strs or "" in strs:
        return ""
    root = {}
    for s in strs:
        node = root
        for ch in s:
            node = node.setdefault(ch, {})
        node['$'] = True
    prefix = []
    node = root
    while len(node) == 1 and '$' not in node:
        ch, node = next(iter(node.items()))
        prefix.append(ch)
    return "".join(prefix)

def longest_common_prefix_simple(strs):          # the shortcut, O(total length)
    lo, hi = min(strs), max(strs)
    i = 0
    while i < len(lo) and lo[i] == hi[i]:
        i += 1
    return lo[:i]
```

**Complexity:** `O(total characters)`.

---

## Problem 5 — Replace Words (LC 648) · Medium · Shortest prefix

**Problem:** Given a dictionary of "roots" and a sentence, replace every word with the **shortest root** that is its prefix.

**Idea:** Trie of roots. For each word, walk down; the first time you hit a node marked as end, stop and return that prefix. If you fall off the trie first, keep the original word.

```python
def replace_words(dictionary, sentence):
    root = {}
    for w in dictionary:
        node = root
        for ch in w:
            node = node.setdefault(ch, {})
        node['$'] = True

    def shortest_root(word):
        node = root
        for i, ch in enumerate(word):
            if ch not in node:
                return word
            node = node[ch]
            if '$' in node:
                return word[:i + 1]
        return word

    return " ".join(shortest_root(w) for w in sentence.split())
```

**Complexity:** `O(total chars)`.
**Variants:** Index Pairs of a String (1065), Longest Word in Dictionary (720).

---

## Problem 6 — Maximum XOR of Two Numbers in an Array (LC 421) · Medium · Binary Trie

**Problem:** Return the maximum `nums[i] XOR nums[j]`.

**Idea:** To maximise XOR you want, at every bit from the **highest to lowest**, the **opposite** bit. Store all numbers in a **binary trie** (each node has children `0` and `1`). For each number walk down; at each bit prefer the child with the opposite bit (that bit becomes `1` in the XOR); if it does not exist take the same bit.

```python
def find_maximum_xor(nums):
    L = max(max(nums).bit_length(), 1)
    root = [None, None]

    def insert(x):
        node = root
        for b in range(L - 1, -1, -1):
            bit = (x >> b) & 1
            if node[bit] is None:
                node[bit] = [None, None]
            node = node[bit]

    def best_xor(x):
        node, res = root, 0
        for b in range(L - 1, -1, -1):
            bit = (x >> b) & 1
            want = bit ^ 1
            if node[want] is not None:
                res |= (1 << b)
                node = node[want]
            else:
                node = node[bit]
        return res

    best = 0
    insert(nums[0])
    for x in nums[1:]:
        best = max(best, best_xor(x))
        insert(x)
    return best
```

**Complexity:** `O(n · 32)`, Space `O(n · 32)`.
**Variants:** Maximum XOR With an Element From Array (1707, offline queries), Count Pairs With XOR in a Range (1803).

---

## Problem 7 — Search Suggestions System (LC 1268) · Medium · Data stored in nodes

**Problem:** After typing each character of `searchWord`, return up to 3 lexicographically smallest products that start with the typed prefix.

**Idea:** Sort products first. Insert each product into a trie, and at **every node on its path** append the product to that node's `top3` list if it has fewer than 3. Typing a prefix = walking the trie and reading `top3`.

```python
def suggested_products(products, search_word):
    products.sort()
    root = {}
    for p in products:
        node = root
        for ch in p:
            node = node.setdefault(ch, {})
            lst = node.setdefault('#top', [])
            if len(lst) < 3:
                lst.append(p)

    res = []
    node = root
    for ch in search_word:
        if node is not None:
            node = node.get(ch)
        res.append(node['#top'] if node else [])
    return res
```

**Complexity:** `O(total chars)` build, `O(len(searchWord))` queries.
**Variants:** Design Search Autocomplete System (642: store counts, sort by frequency then lexicographic order).

---

## Problem 8 — Stream of Characters (LC 1032) · Hard · Reverse Trie

**Problem:** Given words, process a stream of letters; after each letter return `True` if **some suffix of the stream so far** equals a word.

**Idea:** We match **suffixes**, so insert every word **reversed**. Keep the stream; on each new letter walk the trie starting from the **latest** letter backwards. If you reach a word-end → `True`. Stop early when the path breaks, or after the longest word length.

```python
class StreamChecker:
    def __init__(self, words):
        self.root = {}
        self.max_len = 0
        for w in words:
            node = self.root
            for ch in reversed(w):
                node = node.setdefault(ch, {})
            node['$'] = True
            self.max_len = max(self.max_len, len(w))
        self.stream = []

    def query(self, letter):
        self.stream.append(letter)
        node = self.root
        for i in range(len(self.stream) - 1, max(-1, len(self.stream) - 1 - self.max_len), -1):
            node = node.get(self.stream[i])
            if node is None:
                return False
            if '$' in node:
                return True
        return False
```

**Complexity:** per query `O(max word length)`.
**Variants:** Palindrome Pairs (336 — insert reversed words and check palindromic remainders).

---

## Problem 9 — Range Sum Query – Mutable (LC 307) · Medium · Segment tree basic

**Problem:** `update(index, val)` and `sumRange(left, right)` (inclusive) on an array.

**Idea:** Exactly the iterative sum segment tree. Convert inclusive `[left, right]` to half-open `[left, right+1)`.

```python
class NumArray:
    def __init__(self, nums):
        self.tree = SegmentTree(nums, operator.add, 0)

    def update(self, index, val):
        self.tree.update(index, val)

    def sumRange(self, left, right):
        return self.tree.query(left, right + 1)
```

**Complexity:** build `O(n)`, each operation `O(log n)`.
**Alternative:** Fenwick tree (store differences when updating: `add(i, val - old)`).
**Variants:** Range Sum Query 2D – Mutable (308, 2D BIT).

---

## Problem 10 — Range Minimum Query with Updates · Medium · Generic tree (min)

**Problem:** Array with two operations: `update(i, x)` — set `a[i] = x`; `query(l, r)` — the **minimum** in `a[l..r]`. (A classic company-test problem; also the engine behind many "nearest smaller" tasks.)

**Idea:** Same tree, different combine function: `func = min`, `identity = +∞`.

```python
class RangeMin:
    def __init__(self, arr):
        self.tree = SegmentTree(arr, min, float('inf'))

    def update(self, i, x):
        self.tree.update(i, x)

    def query(self, l, r):              # inclusive
        return self.tree.query(l, r + 1)
```

**Complexity:** `O(log n)` per operation.
**Why it matters:** shows you understand the tree is generic. Change `min` to `max`, `math.gcd` (identity 0), `operator.xor` (identity 0) with no other change.
**Variants:** Range GCD queries, Range Maximum with updates, Range Minimum Query (static → Sparse Table).

---

## Problem 11 — Count of Smaller Numbers After Self (LC 315) · Hard · Fenwick over values

**Problem:** For every `nums[i]`, count how many elements to its **right** are smaller.

**Idea:** Process the array **right to left**. Keep a Fenwick tree indexed by the **rank** of each value (coordinate compression). For `x`: answer = number of already-inserted values with rank `< rank(x)` = `prefix(rank - 1)`; then insert `x` (add 1 at its rank).

```python
def count_smaller(nums):
    ranks = {v: i + 1 for i, v in enumerate(sorted(set(nums)))}   # 1-indexed ranks
    bit = Fenwick(len(ranks))
    res = [0] * len(nums)
    for i in range(len(nums) - 1, -1, -1):
        r = ranks[nums[i]]
        res[i] = bit.prefix(r - 1)
        bit.add(r, 1)
    return res
```

**Complexity:** Time `O(n log n)`, Space `O(n)`.
**Variants:** Reverse Pairs (493), Count of Range Sum (327, Problem 15), Number of Longest Increasing Subsequence (673), Count Good Triplets, Create Sorted Array through Instructions (1649). Merge sort also solves these.

---

## Problem 12 — Range Update, Range Sum (Lazy Propagation) · Hard · Lazy segment tree

**Problem:** Array of `n` zeros (or given values). Operations: `add(l, r, v)` — add `v` to every element in `l..r`; `sum(l, r)` — sum of `l..r`.
(Easier LeetCode relative: **Range Addition**, LC 370, solvable with a difference array when all updates come before queries.)

**Idea:** The `LazySegTree` of Part B 8. A full-cover update touches one node and leaves a lazy tag.

```python
def range_add_range_sum(n, operations):
    """operations: ('add', l, r, v) or ('sum', l, r). Returns the answers of 'sum' ops."""
    tree = LazySegTree([0] * n)
    out = []
    for op in operations:
        if op[0] == 'add':
            _, l, r, v = op
            tree.update(l, r, v)
        else:
            _, l, r = op
            out.append(tree.query(l, r))
    return out
```

**Complexity:** `O(log n)` per operation.
**Check by hand:** `n=5`; `add(0,2,3)` → `[3,3,3,0,0]`; `sum(1,3)` → `3+3+0 = 6`.
**Variants:** Range Addition II (598), Corporate Flight Bookings (1109, difference array), Booking/Calendar problems.

---

## Problem 13 — My Calendar III (LC 732) · Hard · Dynamic segment tree

**Problem:** `book(start, end)` adds the half-open interval `[start, end)`. After each booking return the **maximum number of simultaneous bookings (k-booking)** so far. Times go up to 10⁹.

**Idea:** A segment tree over `[0, 10⁹]` is too large to allocate, so create nodes **lazily in a dict** (dynamic segment tree). Each node stores `mx` = the max overlap inside its segment, plus a permanent `lazy` count (no push-down needed: a node's lazy applies to its whole segment, and the parent re-reads `lazy + max(children)`).

```python
class MyCalendarThree:
    def __init__(self):
        self.mx = defaultdict(int)
        self.lz = defaultdict(int)

    def _update(self, node, s, e, l, r):
        if r < s or e < l:
            return
        if l <= s and e <= r:
            self.mx[node] += 1
            self.lz[node] += 1
            return
        mid = (s + e) // 2
        self._update(2 * node, s, mid, l, r)
        self._update(2 * node + 1, mid + 1, e, l, r)
        self.mx[node] = self.lz[node] + max(self.mx[2 * node], self.mx[2 * node + 1])

    def book(self, start, end):
        self._update(1, 0, 10**9, start, end - 1)
        return self.mx[1]
```

**Complexity:** `O(log 10⁹) ≈ 30` levels × a few nodes per booking → `O(log C)` per booking.
**Alternative:** store a sorted map of `+1/-1` events and sweep — `O(n)` per query (My Calendar I/II can use plain lists).
**Variants:** My Calendar I (729), II (731), Range Module (715), Data Stream as Disjoint Intervals.

---

## Problem 14 — Falling Squares (LC 699) · Hard · Coordinate compression + range-assign/range-max

**Problem:** Squares fall one by one: `positions[i] = [left, sideLength]`. Each lands on top of whatever is below (or the ground) and sticks. After each drop, return the **current tallest height** of any stack.

**Idea:** When a square covering `[left, left+side)` lands, its top height = `(max height currently in that range) + side`, and the whole range then has **exactly that height** (assign). So we need: **range max query** and **range assign** — a lazy segment tree with an *assign* tag. Compress coordinates to the sorted set of all `left` and `left+side` values; each tree position is the elementary interval between two consecutive coordinates.

```python
class AssignMaxTree:
    """Range assign (to a positive value), range max. Inclusive indices."""
    def __init__(self, n):
        self.n = n
        self.mx = [0] * (4 * n)
        self.lz = [0] * (4 * n)          # 0 = no pending assign (heights are always > 0)

    def _push(self, node):
        if self.lz[node]:
            for ch in (2 * node, 2 * node + 1):
                self.mx[ch] = self.lz[ch] = self.lz[node]
            self.lz[node] = 0

    def assign(self, l, r, h, node=1, s=0, e=None):
        if e is None:
            e = self.n - 1
        if r < s or e < l:
            return
        if l <= s and e <= r:
            self.mx[node] = self.lz[node] = h
            return
        self._push(node)
        mid = (s + e) // 2
        self.assign(l, r, h, 2 * node, s, mid)
        self.assign(l, r, h, 2 * node + 1, mid + 1, e)
        self.mx[node] = max(self.mx[2 * node], self.mx[2 * node + 1])

    def query(self, l, r, node=1, s=0, e=None):
        if e is None:
            e = self.n - 1
        if r < s or e < l:
            return 0
        if l <= s and e <= r:
            return self.mx[node]
        self._push(node)
        mid = (s + e) // 2
        return max(self.query(l, r, 2 * node, s, mid),
                   self.query(l, r, 2 * node + 1, mid + 1, e))


def falling_squares(positions):
    xs = sorted({x for left, side in positions for x in (left, left + side)})
    idx = {x: i for i, x in enumerate(xs)}
    tree = AssignMaxTree(len(xs) - 1)    # one slot per elementary interval
    res, tallest = [], 0
    for left, side in positions:
        a, b = idx[left], idx[left + side] - 1
        h = tree.query(a, b) + side
        tree.assign(a, b, h)
        tallest = max(tallest, h)
        res.append(tallest)
    return res
```

**Complexity:** `O(n log n)`.
**Check:** `[[1,2],[2,3],[6,1]] → [2, 5, 5]`.
**Variants:** The Skyline Problem (218), Rectangle Area II (850), Range Module (715). (A simple `O(n²)` solution also exists — compare with all earlier squares — mention it first, then optimise.)

---

## Problem 15 — Count of Range Sum (LC 327) · Hard · Prefix sums + Fenwick + compression

**Problem:** Count subarrays whose sum lies in `[lower, upper]`.

**Idea:** Let `P[k]` be prefix sums (`P[0] = 0`). Subarray `i+1..j` has sum `P[j] - P[i]`. For each `j` we need the number of earlier `i` with
`lower ≤ P[j] - P[i] ≤ upper` ⟺ `P[j] - upper ≤ P[i] ≤ P[j] - lower`.
Compress all prefix sums, keep a Fenwick tree of counts of earlier prefixes, and for each `j` do a range-count query, then insert `P[j]`.

```python
def count_range_sum(nums, lower, upper):
    prefix = [0]
    for x in nums:
        prefix.append(prefix[-1] + x)
    vals = sorted(set(prefix))
    bit = Fenwick(len(vals))
    count = 0
    for p in prefix:
        lo = bisect_left(vals, p - upper)           # first value >= p - upper
        hi = bisect_right(vals, p - lower)          # first value >  p - lower
        count += bit.prefix(hi) - bit.prefix(lo)    # counts values in [p-upper, p-lower]
        bit.add(bisect_left(vals, p) + 1, 1)        # insert p (1-indexed)
    return count
```

**Complexity:** Time `O(n log n)`, Space `O(n)`.
**Check:** `nums=[-2,5,-1], lower=-2, upper=2 → 3`.
**Variants:** Reverse Pairs (493), Subarray Sum Equals K (560 → hashmap), Number of Subarrays With Sum in Range, Merge-sort solution to the same problem.

---

# PART D — Practice, Mistakes, Plan

## Extra practice

| Topic | Easy | Medium | Hard |
|---|---|---|---|
| **Trie** | Longest Common Prefix (14), Longest Word in Dictionary (720) | Map Sum Pairs (677), Implement Trie II (1804), Design Search Autocomplete (642), Prefix and Suffix Search (745), Count Prefix Scores (2416), Maximum XOR With Element (1707) | Word Search II (212), Palindrome Pairs (336), Concatenated Words (472), Stream of Characters (1032) |
| **Seg tree / BIT** | Range Sum Query – Immutable (303) | Range Sum Query – Mutable (307), Range Addition (370), My Calendar II (731), Count of Smaller Numbers (315) | Reverse Pairs (493), Count of Range Sum (327), Falling Squares (699), The Skyline Problem (218), Rectangle Area II (850), Range Module (715) |

## Common mistakes

**Trie**
1. Confusing **prefix exists** with **word exists** — always check the end flag in `search`.
2. Using a plain `set` for `children` and forgetting what the dict stores (mixing char keys and special keys like `'$'` — skip special keys when iterating all children).
3. Not pruning in Word Search II → TLE.
4. Forgetting that deleting a word must keep the shared prefixes of other words.
5. Binary trie: using too few bits (use `max(nums).bit_length()`).

**Segment tree**
1. Mixing **inclusive `[l, r]`** and **half-open `[l, r)`** conventions → off-by-one. Pick one and write it on top of your code.
2. Using the wrong **identity** (`0` for min would give wrong answers; use `inf`).
3. Array size: recursive version needs `4n`.
4. Lazy: forgetting to **push down** before recursing, or forgetting to **recompute the parent** after.
5. Fenwick tree is **1-indexed** — index `0` creates an infinite loop (`0 & -0 = 0`).
6. Not compressing coordinates when values are huge or negative.
7. Python recursion depth is fine for segment trees (depth ~ log n), but Python is slow: for `n, q = 2·10⁵` prefer the iterative tree or a Fenwick tree.

## How to explain in the interview

- **Trie:** "A trie shares common prefixes, so insert/search/prefix queries cost O(length of the word), not O(number of words). I will store an end marker and, if needed, counts."
- **Segment tree:** "Each node stores the answer for a segment. Queries combine O(log n) nodes; an update recomputes O(log n) ancestors. For range updates I add lazy propagation. If it is only prefix sums with point updates, a Fenwick tree is simpler."
- Always say the **complexity** and **why a simpler approach is too slow** (n·q).


