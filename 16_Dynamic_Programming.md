# Dynamic Programming (DP) — Complete Guide (Python) for Coding Rounds

> Goal: understand DP so well that you can **derive** a solution instead of memorising it.
> Language: Python 3. All code is tested.
> Read Parts 1–5 slowly, then solve Part 7 (15 problems) on your own before reading the solutions.

---

## Table of Contents

1. What is DP? (the idea in plain words)
2. When can I use DP? (the 2 conditions)
3. Two ways to write DP: Memoization vs Tabulation (+ space optimisation)
4. The 5-step recipe to solve ANY DP problem
5. The DP Patterns (the real secret)
6. Pattern recognition cheat sheet
7. **15 Standard Interview Problems** (state → transition → code → complexity)
8. Extra practice, common mistakes, 7-day plan

---

# Part 1 — What is DP?

**DP = recursion + remembering answers.**

Dynamic programming is a technique to solve a big problem by:
1. breaking it into smaller **sub-problems**,
2. solving each sub-problem **only once**,
3. **storing** the answer and reusing it.

## The classic example: Fibonacci

`fib(n) = fib(n-1) + fib(n-2)`

Plain recursion:

```python
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)
```

Look at what it computes for `fib(5)`:

```
                    fib(5)
                  /        \
             fib(4)         fib(3)
            /     \         /    \
        fib(3)   fib(2)  fib(2)  fib(1)
        /   \
    fib(2) fib(1) ...
```

`fib(3)` is computed **twice**, `fib(2)` **three times**... This is wasted work.
Plain recursion takes **O(2ⁿ)** time. `fib(50)` would take years.

DP idea: *"I already solved fib(3) once. Write it in a notebook. Next time just read the notebook."*
Time becomes **O(n)**. That's the whole magic.

---

# Part 2 — When can I use DP?

A problem is a DP problem when it has BOTH:

### 1. Overlapping sub-problems
The same smaller problem appears again and again (like `fib(3)` above).
*(If sub-problems never repeat — like merge sort — it's plain divide and conquer, not DP.)*

### 2. Optimal substructure
The best answer of the big problem can be built from best answers of smaller problems.
*(Shortest path from A to C through B = shortest A→B + shortest B→C.)*

### Words in a problem that hint DP
- "**minimum / maximum** cost / ways / length / profit"
- "**count the number of ways**"
- "is it **possible** to ...?" (true/false)
- "**longest / shortest** subsequence"
- You see choices at each step: *take or skip*, *go left or right*, *buy or sell*
- Brute force would be exponential (try all subsets / all paths)

> If the problem asks you to **print all** solutions (all subsets, all paths), it is usually **backtracking**, not DP. DP gives the **count / best value**.

---

# Part 3 — Memoization vs Tabulation

## 3.1 Top-Down = Recursion + Memo ("Memoization")

Write the recursion first. Before computing, check the notebook (dict / array / `lru_cache`).

```python
from functools import lru_cache

def fib_memo(n):
    @lru_cache(maxsize=None)
    def f(i):
        if i <= 1:
            return i
        return f(i - 1) + f(i - 2)
    return f(n)

# Manual memo (same thing)
def fib_memo2(n, memo=None):
    if memo is None:
        memo = {}
    if n <= 1:
        return n
    if n in memo:
        return memo[n]
    memo[n] = fib_memo2(n - 1, memo) + fib_memo2(n - 2, memo)
    return memo[n]
```

Pros: easy to write, only computes needed states.
Cons: recursion depth (Python limit ~1000), function-call overhead.

## 3.2 Bottom-Up = Table filling ("Tabulation")

Start from the smallest sub-problems and fill a table (array) to the answer. No recursion.

```python
def fib_tab(n):
    if n <= 1:
        return n
    dp = [0] * (n + 1)
    dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]
    return dp[n]
```

## 3.3 Space Optimisation

`dp[i]` only needs the previous 2 values, so keep just 2 variables.

```python
def fib_opt(n):
    if n <= 1:
        return n
    prev2, prev1 = 0, 1
    for _ in range(2, n + 1):
        prev2, prev1 = prev1, prev2 + prev1
    return prev1
```

Time `O(n)`, Space `O(1)`.

## 3.4 Which one to use in an interview?

| | Top-Down | Bottom-Up |
|---|---|---|
| Easy to think | **Yes** (just add memo to recursion) | Needs ordering of states |
| Speed in Python | Slower | **Faster** |
| Recursion limit risk | Yes | No |
| Space optimisation | Hard | **Easy** |

**Strategy:** think in recursion → get the formula → write bottom-up (mention space optimisation at the end).

---

# Part 4 — The 5-Step Recipe

For every DP problem answer these five questions **in order**:

1. **State** — "What information do I need to describe a sub-problem?"
   Example: `dp[i]` = best answer using the first `i` items. Or `dp[i][j]`.
2. **Transition** — "How does `dp[state]` depend on smaller states?"
   This is the formula (the hardest part). Think about the **last choice** you make: take / skip, which previous element, etc.
3. **Base case** — smallest states whose answer you know directly (`dp[0] = ...`).
4. **Order** — fill so that dependencies are ready (increasing `i`; for 2D, rows then columns).
5. **Answer** — where is the final answer? (`dp[n]`, `dp[n][m]`, or `max(dp)`.)

Then: **state the complexity** = (number of states) × (work per state).

> Trick for finding the state: look at the **input variables that change** as you recurse. Two strings → `dp[i][j]`. One array → `dp[i]`. Array + capacity → `dp[i][w]`.

---

# Part 5 — The DP Patterns

Almost every interview DP problem belongs to one of these families. Learn the family, and new problems become variations.

## Pattern 1 — Linear DP (1D): "decide for each position"

State `dp[i]` depends on the previous few `dp`'s.
```
dp[i] = f(dp[i-1], dp[i-2], ...)
```
Problems: Climbing Stairs, House Robber, Decode Ways, Maximum Subarray, Min Cost Climbing Stairs.

## Pattern 2 — Grid DP (2D paths)

`dp[r][c]` depends on the cell above and the cell to the left.
```
dp[r][c] = dp[r-1][c] + dp[r][c-1]          # count paths
dp[r][c] = grid[r][c] + min(dp[r-1][c], dp[r][c-1])   # min cost path
```
Problems: Unique Paths (I, II), Minimum Path Sum, Triangle, Maximal Square, Dungeon Game.

## Pattern 3 — Knapsack family ("take it or leave it")

For each item: **take** it (use up capacity) or **skip** it.
- **0/1 Knapsack** — each item at most once → loop capacity **backwards**.
- **Unbounded Knapsack** — unlimited use of each item → loop capacity **forwards**.
- Variants: Subset Sum, Partition Equal Subset Sum, Target Sum, Coin Change (min coins), Coin Change II (count ways), Rod Cutting, Perfect Squares, Last Stone Weight II.

```
# 0/1: iterate capacity high -> low (so an item is not reused)
for item in items:
    for w in range(W, item.weight - 1, -1):
        dp[w] = max(dp[w], dp[w - item.weight] + item.value)

# Unbounded: iterate capacity low -> high (item can be reused)
for w in range(item.weight, W + 1):
    dp[w] = max(dp[w], dp[w - item.weight] + item.value)
```

## Pattern 4 — Subsequence DP (LIS family)

`dp[i]` = best answer for a subsequence **ending at index `i`**.
```
dp[i] = 1 + max(dp[j] for j < i if nums[j] < nums[i])
```
Problems: Longest Increasing Subsequence, Number of LIS, Russian Doll Envelopes, Largest Divisible Subset, Longest Bitonic Subsequence.

## Pattern 5 — Two-string DP (LCS family)

`dp[i][j]` = answer for the first `i` chars of string A and the first `j` chars of string B.
```
if A[i-1] == B[j-1]:  dp[i][j] = dp[i-1][j-1] + 1     (match: both move)
else:                 dp[i][j] = max(dp[i-1][j], dp[i][j-1])  (skip one char)
```
Problems: LCS, Edit Distance, Longest Palindromic Subsequence (LCS of `s` and `reverse(s)`), Shortest Common Supersequence, Distinct Subsequences, Interleaving String, Regex / Wildcard Matching.

## Pattern 6 — String / Palindrome DP

`dp[i][j]` = is substring `s[i..j]` a palindrome / can it be formed?
Problems: Longest Palindromic Substring, Palindromic Substrings, Word Break, Palindrome Partitioning II.

## Pattern 7 — State-machine DP (buy/sell stocks)

At each step you are in one of a few **states** (holding, sold, cooldown). Write a transition for each state.
Problems: Best Time to Buy and Sell Stock I–IV, with Cooldown, with Transaction Fee.

## Pattern 8 — Interval DP

`dp[l][r]` = best answer for the segment `l..r`; try every split point `k` between `l` and `r`.
```
dp[l][r] = best over k in (l, r) of  dp[l][k] + dp[k][r] + cost(l, k, r)
```
Problems: Burst Balloons, Matrix Chain Multiplication, Minimum Cost to Merge Stones, Palindrome Partitioning, Strange Printer.

## Pattern 9 — (Know they exist) Bitmask DP, DP on Trees, Digit DP

- **Bitmask DP**: `dp[mask]` where mask = which items are used. For `n ≤ 20` (Travelling Salesman, Partition to K Equal Sum Subsets).
- **DP on Trees**: `dp[node]` computed from children via DFS (House Robber III, Binary Tree Maximum Path Sum, Diameter).
- **Digit DP**: count numbers ≤ N with a digit property.

---

# Part 6 — Pattern Recognition Cheat Sheet

| The problem says... | Pattern | Typical state |
|---|---|---|
| "ways to reach step n", "can't take adjacent" | Linear | `dp[i]` |
| Grid, move right/down | Grid | `dp[r][c]` |
| Items with weight/cost, target sum / capacity | Knapsack | `dp[w]` or `dp[i][w]` |
| "Min coins / count ways to make amount" | Unbounded knapsack | `dp[amount]` |
| Longest increasing / divisible subsequence | LIS | `dp[i]` ends at i |
| Two strings: common, transform, match | LCS family | `dp[i][j]` |
| Palindromes, substring checks | String DP | `dp[i][j]` for `s[i..j]` |
| Buy / sell / cooldown / at most k transactions | State machine | `hold, sold, rest` |
| Burst / merge / split a range optimally | Interval | `dp[l][r]` |
| Subsets of ≤ 20 items | Bitmask | `dp[mask]` |

**Complexity rule:** `time = (#states) × (transitions per state)`.

---

# Part 7 — 15 Standard Interview Problems

For each problem: **Problem → State → Transition → Code → Complexity → Variants.**
(LeetCode numbers are given so you can practise online.)

```python
from functools import lru_cache
import bisect
```

---

## Problem 1 — Climbing Stairs (LC 70) · Easy · Linear

**Problem:** You can climb 1 or 2 steps at a time. How many distinct ways to reach step `n`?

**State:** `dp[i]` = ways to reach step `i`.
**Transition:** the last move was either 1 step (from `i-1`) or 2 steps (from `i-2`):
`dp[i] = dp[i-1] + dp[i-2]` (it is Fibonacci).
**Base:** `dp[1] = 1`, `dp[2] = 2`.

```python
def climb_stairs(n):
    if n <= 2:
        return n
    prev2, prev1 = 1, 2
    for _ in range(3, n + 1):
        prev2, prev1 = prev1, prev1 + prev2
    return prev1
```

**Complexity:** Time `O(n)`, Space `O(1)`.
**Variants:** Min Cost Climbing Stairs (746), steps allowed `{1,3,5}` (loop over allowed steps), Tribonacci.

---

## Problem 2 — House Robber (LC 198) · Easy/Medium · Linear

**Problem:** Houses in a row with money `nums[i]`. You cannot rob two adjacent houses. Maximise money.

**State:** `dp[i]` = best money using houses `0..i`.
**Transition:** for house `i` you either **skip** it (`dp[i-1]`) or **rob** it (`dp[i-2] + nums[i]`):
`dp[i] = max(dp[i-1], dp[i-2] + nums[i])`.

```python
def rob(nums):
    prev2, prev1 = 0, 0               # dp[i-2], dp[i-1]
    for x in nums:
        prev2, prev1 = prev1, max(prev1, prev2 + x)
    return prev1
```

**Complexity:** Time `O(n)`, Space `O(1)`.
**Variants:** House Robber II (circular: answer = `max(rob(nums[1:]), rob(nums[:-1]))`, edge case single house), House Robber III (tree DP), Delete and Earn (740).

---

## Problem 3 — Coin Change (LC 322) · Medium · Unbounded Knapsack

**Problem:** Given coin denominations (unlimited supply) and an `amount`, return the **fewest coins** to make it, or `-1`.

**State:** `dp[a]` = min coins to make amount `a`.
**Transition:** try each coin `c`: the last coin used is `c`, so `dp[a] = min(dp[a - c] + 1)` over coins `c ≤ a`.
**Base:** `dp[0] = 0`; everything else starts at infinity.

```python
def coin_change(coins, amount):
    INF = float('inf')
    dp = [INF] * (amount + 1)
    dp[0] = 0
    for a in range(1, amount + 1):
        for c in coins:
            if c <= a and dp[a - c] + 1 < dp[a]:
                dp[a] = dp[a - c] + 1
    return -1 if dp[amount] == INF else dp[amount]
```

**Complexity:** Time `O(amount × #coins)`, Space `O(amount)`.
**Why greedy fails:** coins `[1, 3, 4]`, amount `6` → greedy gives 4+1+1 (3 coins), best is 3+3 (2 coins).
**Variants:** Coin Change II (518: count ways → loop coins **outer**, amount inner, `dp[a] += dp[a-c]`), Perfect Squares (279), Minimum Cost For Tickets (983).

---

## Problem 4 — Longest Increasing Subsequence (LC 300) · Medium · Subsequence

**Problem:** Length of the longest strictly increasing subsequence (not necessarily contiguous).

**State:** `dp[i]` = LIS length that **ends** at index `i`.
**Transition:** `dp[i] = 1 + max(dp[j])` over all `j < i` with `nums[j] < nums[i]` (or 1 if none).
**Answer:** `max(dp)`.

```python
def length_of_lis(nums):          # O(n^2) — the DP idea
    n = len(nums)
    dp = [1] * n
    for i in range(n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp) if nums else 0

def length_of_lis_fast(nums):     # O(n log n) — "patience sorting"
    tails = []                    # tails[k] = smallest tail of an increasing subseq of length k+1
    for x in nums:
        pos = bisect.bisect_left(tails, x)
        if pos == len(tails):
            tails.append(x)
        else:
            tails[pos] = x
    return len(tails)
```

**Complexity:** `O(n²)` and `O(n log n)`; space `O(n)`.
**Note:** `tails` is NOT the actual subsequence, only its length is meaningful.
**Variants:** Number of LIS (673), Russian Doll Envelopes (354: sort by width asc, height desc, then LIS on height), Largest Divisible Subset (368).

---

## Problem 5 — Unique Paths (LC 62) · Medium · Grid

**Problem:** A robot starts at top-left of an `m × n` grid and can only move right or down. Count paths to bottom-right.

**State:** `dp[r][c]` = number of ways to reach cell `(r, c)`.
**Transition:** came from above or from left: `dp[r][c] = dp[r-1][c] + dp[r][c-1]`.
**Base:** first row and first column are all `1`.

Space trick: only the previous row is needed, so keep one row.

```python
def unique_paths(m, n):
    row = [1] * n
    for _ in range(1, m):
        for c in range(1, n):
            row[c] += row[c - 1]       # row[c] (old) = from above, row[c-1] = from left
    return row[-1]

def unique_paths_with_obstacles(grid):      # LC 63
    R, C = len(grid), len(grid[0])
    row = [0] * C
    row[0] = 1 if grid[0][0] == 0 else 0
    for r in range(R):
        for c in range(C):
            if grid[r][c] == 1:
                row[c] = 0                  # blocked cell: 0 ways
            elif c > 0:
                row[c] += row[c - 1]
    return row[-1]
```

**Complexity:** Time `O(m·n)`, Space `O(n)`.
**Math shortcut:** answer = C(m+n-2, m-1).
**Variants:** Minimum Path Sum (64: `grid[r][c] + min(up, left)`), Triangle (120), Maximal Square (221), Dungeon Game (174, fill backwards).

---

## Problem 6 — Maximum Subarray (LC 53) · Easy/Medium · Kadane's Algorithm

**Problem:** Find the contiguous subarray with the largest sum.

**State:** `cur` = best sum of a subarray **ending at** the current index.
**Transition:** either extend the previous subarray or start fresh here: `cur = max(x, cur + x)`.
**Answer:** the maximum `cur` seen.

```python
def max_subarray(nums):
    cur = best = nums[0]
    for x in nums[1:]:
        cur = max(x, cur + x)
        best = max(best, cur)
    return best
```

**Complexity:** Time `O(n)`, Space `O(1)`.
**Variants:** Maximum Product Subarray (152: track both `max_so_far` and `min_so_far` because a negative can flip), Maximum Sum Circular Subarray (918), Best Time to Buy and Sell Stock (121).

---

## Problem 7 — Longest Common Subsequence (LC 1143) · Medium · Two-string DP

**Problem:** Length of the longest subsequence common to both strings.

**State:** `dp[i][j]` = LCS of `A[:i]` and `B[:j]`.
**Transition:**
- chars match → `dp[i][j] = dp[i-1][j-1] + 1`
- else → `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`
**Base:** `dp[0][*] = dp[*][0] = 0`. **Answer:** `dp[n][m]`.

```python
def longest_common_subsequence(a, b):
    n, m = len(a), len(b)
    dp = [[0] * (m + 1) for _ in range(n + 1)]
    for i in range(1, n + 1):
        for j in range(1, m + 1):
            if a[i - 1] == b[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    return dp[n][m]
```

**Complexity:** Time `O(n·m)`, Space `O(n·m)` (reducible to `O(min(n,m))` with two rows).
**Variants:** Longest Palindromic Subsequence (516) = LCS(s, reversed s), Min Deletions to Make Strings Equal (583) = `n + m - 2·LCS`, Shortest Common Supersequence (1092), Distinct Subsequences (115).

---

## Problem 8 — Edit Distance (LC 72) · Hard-ish · Two-string DP

**Problem:** Minimum operations (insert, delete, replace a character) to convert `word1` into `word2`.

**State:** `dp[i][j]` = min operations to convert `word1[:i]` into `word2[:j]`.
**Transition:**
- chars equal → `dp[i][j] = dp[i-1][j-1]` (no cost)
- else `1 + min(`
  - `dp[i-1][j]` → **delete** from word1,
  - `dp[i][j-1]` → **insert** into word1,
  - `dp[i-1][j-1]` → **replace** `)`
**Base:** `dp[i][0] = i` (delete everything), `dp[0][j] = j` (insert everything).

```python
def min_distance(word1, word2):
    n, m = len(word1), len(word2)
    dp = [[0] * (m + 1) for _ in range(n + 1)]
    for i in range(n + 1):
        dp[i][0] = i
    for j in range(m + 1):
        dp[0][j] = j
    for i in range(1, n + 1):
        for j in range(1, m + 1):
            if word1[i - 1] == word2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1]
            else:
                dp[i][j] = 1 + min(dp[i - 1][j],      # delete
                                   dp[i][j - 1],      # insert
                                   dp[i - 1][j - 1])  # replace
    return dp[n][m]
```

**Complexity:** Time `O(n·m)`, Space `O(n·m)` (or `O(m)` with rolling rows).
**Variants:** One Edit Distance (161), Delete Operation for Two Strings (583), Wildcard Matching (44), Regular Expression Matching (10).

---

## Problem 9 — 0/1 Knapsack (classic) · Medium · Knapsack

**Problem:** `n` items with `weights[i]` and `values[i]`. A bag of capacity `W`. Each item can be taken **at most once**. Maximise total value.

**State:** `dp[w]` = best value with capacity `w` (after considering some items).
**Transition:** for each item, either skip it or take it: `dp[w] = max(dp[w], dp[w - wt] + val)`.
**Important:** loop `w` from **high to low**, so the same item is not used twice in one pass.

```python
def knapsack_01(weights, values, W):
    dp = [0] * (W + 1)
    for wt, val in zip(weights, values):
        for w in range(W, wt - 1, -1):          # backwards!
            dp[w] = max(dp[w], dp[w - wt] + val)
    return dp[W]
```

**Complexity:** Time `O(n·W)`, Space `O(W)`.
**Unbounded version:** loop `w` forwards from `wt` to `W` (item can repeat).
**Variants:** Target Sum (494), Last Stone Weight II (1049), Ones and Zeroes (474, 2D capacity).

---

## Problem 10 — Partition Equal Subset Sum (LC 416) · Medium · Knapsack (Subset Sum)

**Problem:** Can the array be split into two subsets with equal sum?

**Idea:** If total is odd → no. Otherwise ask: "is there a subset with sum = `total/2`?" — a true/false 0/1 knapsack.

**State:** `dp[s]` = True if some subset sums to `s`.
**Transition:** `dp[s] = dp[s] or dp[s - x]` for each number `x`, `s` from high to low.
**Base:** `dp[0] = True`.

```python
def can_partition(nums):
    total = sum(nums)
    if total % 2:
        return False
    target = total // 2
    dp = [False] * (target + 1)
    dp[0] = True
    for x in nums:
        for s in range(target, x - 1, -1):
            if dp[s - x]:
                dp[s] = True
    return dp[target]
```

**Complexity:** Time `O(n · sum/2)`, Space `O(sum)`.
**Python bonus (bitset trick):** `bits |= bits << x` over a big integer, then check bit `target`.
**Variants:** Subset Sum, Partition to K Equal Sum Subsets (698, bitmask/backtracking), Target Sum (494: `(sum + target) / 2` subset count).

---

## Problem 11 — Word Break (LC 139) · Medium · Linear DP over a string

**Problem:** Can string `s` be split into a sequence of words from a dictionary?

**State:** `dp[i]` = True if `s[:i]` can be segmented.
**Transition:** `dp[i] = True` if there is a `j < i` with `dp[j]` True and `s[j:i]` in the dictionary.
**Base:** `dp[0] = True` (empty string).

```python
def word_break(s, word_dict):
    words = set(word_dict)
    max_len = max(map(len, words)) if words else 0
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True
    for i in range(1, n + 1):
        for j in range(max(0, i - max_len), i):    # only try plausible lengths
            if dp[j] and s[j:i] in words:
                dp[i] = True
                break
    return dp[n]
```

**Complexity:** Time `O(n · L)` substring checks (each costs up to `O(L)`), Space `O(n)`.
**Variants:** Word Break II (140: return all sentences → memoised backtracking), Concatenated Words (472), Palindrome Partitioning (131 / 132).

---

## Problem 12 — Longest Palindromic Substring (LC 5) · Medium · String DP

**Problem:** Return the longest palindromic substring of `s`.

**State:** `dp[i][j]` = True if `s[i..j]` is a palindrome.
**Transition:** `dp[i][j] = (s[i] == s[j]) and (j - i < 3 or dp[i+1][j-1])` — the ends match and the inside is a palindrome (or very short).
**Order:** `i` from the end down to 0 (we need `dp[i+1][...]` ready), `j` from `i` up.

```python
def longest_palindrome(s):
    n = len(s)
    dp = [[False] * n for _ in range(n)]
    start, best = 0, 1
    for i in range(n - 1, -1, -1):
        for j in range(i, n):
            if s[i] == s[j] and (j - i < 3 or dp[i + 1][j - 1]):
                dp[i][j] = True
                if j - i + 1 > best:
                    start, best = i, j - i + 1
    return s[start:start + best]
```

**Complexity:** Time `O(n²)`, Space `O(n²)`.
**Better in interviews (`O(1)` space):** "expand around centre" — for each of the `2n-1` centres, expand while the characters match. Mention both.
**Variants:** Palindromic Substrings (647: count all `True`), Longest Palindromic Subsequence (516), Palindrome Partitioning II (132).

---

## Problem 13 — Best Time to Buy and Sell Stock with Cooldown (LC 309) · Medium · State machine

**Problem:** You may buy/sell many times (hold one share at a time). After you sell you must wait one day (cooldown) before buying. Maximise profit.

**States at the end of each day:**
- `hold` = max profit while **holding** a share,
- `sold` = max profit having **just sold** today,
- `rest` = max profit **not holding**, free to buy (cooldown over).

**Transitions (for price `p`):**
- `hold = max(hold, rest - p)` — keep holding, or buy today (only from `rest`)
- `sold = hold_prev + p` — sell the share we held
- `rest = max(rest, sold_prev)` — stay idle, or cooldown finished after yesterday's sell

```python
def max_profit_cooldown(prices):
    hold, sold, rest = float('-inf'), 0, 0
    for p in prices:
        prev_sold = sold
        sold = hold + p
        hold = max(hold, rest - p)
        rest = max(rest, prev_sold)
    return max(sold, rest)
```

**Complexity:** Time `O(n)`, Space `O(1)`.
**Related template:** Stock I (121: one transaction), II (122: unlimited), III/IV (123/188: `dp[k][hold/not]`), With Fee (714).

---

## Problem 14 — Decode Ways (LC 91) · Medium · Linear DP over a string

**Problem:** `'A'→1, 'B'→2, ..., 'Z'→26`. Given a digit string, count how many ways it can be decoded.

**State:** `dp[i]` = ways to decode the first `i` characters.
**Transition:**
- one digit: if `s[i-1] != '0'` → add `dp[i-1]`
- two digits: if `10 ≤ int(s[i-2:i]) ≤ 26` → add `dp[i-2]`
**Base:** `dp[0] = 1` (empty string has one way).

```python
def num_decodings(s):
    if not s or s[0] == '0':
        return 0
    prev2, prev1 = 1, 1                # dp[i-2], dp[i-1]
    for i in range(2, len(s) + 1):
        cur = 0
        if s[i - 1] != '0':
            cur += prev1
        if 10 <= int(s[i - 2:i]) <= 26:
            cur += prev2
        prev2, prev1 = prev1, cur
    return prev1
```

**Complexity:** Time `O(n)`, Space `O(1)`.
**Tricky cases:** `"0"` → 0, `"06"` → 0, `"10"` → 1, `"226"` → 3.
**Variants:** Decode Ways II (639, with `'*'`), Climbing Stairs is the same shape.

---

## Problem 15 — Burst Balloons (LC 312) · Hard · Interval DP

**Problem:** Balloons with values `nums`. Bursting balloon `i` gives `nums[i-1] * nums[i] * nums[i+1]` coins (outside the array counts as 1). Maximise coins.

**Key insight:** Thinking "which balloon do I burst **first**?" breaks the problem badly (neighbours change). Think instead: **which balloon is burst LAST** in the range?

**State:** `dp[l][r]` = max coins from bursting **all balloons strictly between** `l` and `r` (balloons `l` and `r` stay).
**Transition:** pick `k` (`l < k < r`) as the last balloon burst inside. At that moment its neighbours are `l` and `r`:
`dp[l][r] = max over k of ( dp[l][k] + nums[l]*nums[k]*nums[r] + dp[k][r] )`.
**Setup:** pad `nums` with `1` on both ends. **Answer:** `dp[0][n-1]`.

```python
def max_coins(nums):
    nums = [1] + nums + [1]
    n = len(nums)
    dp = [[0] * n for _ in range(n)]
    for length in range(2, n):                    # distance between l and r
        for l in range(n - length):
            r = l + length
            for k in range(l + 1, r):
                coins = nums[l] * nums[k] * nums[r] + dp[l][k] + dp[k][r]
                if coins > dp[l][r]:
                    dp[l][r] = coins
    return dp[0][n - 1]
```

**Complexity:** Time `O(n³)`, Space `O(n²)`.
**Variants:** Matrix Chain Multiplication, Minimum Cost to Cut a Stick (1547), Minimum Score Triangulation of Polygon (1039), Strange Printer (664), Guess Number Higher or Lower II (375).

---

# Part 8 — Extra Practice, Mistakes, Plan

## 8.1 Extra problems (after the 15)

| Level | Problems |
|---|---|
| Easy | Min Cost Climbing Stairs (746), N-th Tribonacci (1137), Pascal's Triangle (118), Divisor Game (1025), Counting Bits (338) |
| Medium | Coin Change II (518), Target Sum (494), Perfect Squares (279), House Robber II (213), Minimum Path Sum (64), Triangle (120), Maximal Square (221), Maximum Product Subarray (152), Longest Palindromic Subsequence (516), Palindromic Substrings (647), Jump Game (55 / 45), Unique Binary Search Trees (96), Ones and Zeroes (474), Partition Labels, Best Time to Buy and Sell Stock II / with Fee (122 / 714), Minimum Cost for Tickets (983), Number of LIS (673), Russian Doll Envelopes (354), Longest String Chain (1048) |
| Hard | Regular Expression Matching (10), Wildcard Matching (44), Distinct Subsequences (115), Interleaving String (97), Best Time to Buy and Sell Stock III / IV (123 / 188), Dungeon Game (174), Palindrome Partitioning II (132), Partition to K Equal Sum Subsets (698), Largest Rectangle in Histogram / Maximal Rectangle (84 / 85), Egg Drop (887), Cherry Pickup (741) |

## 8.2 Common mistakes

1. **Wrong state** — if you cannot write the transition, the state is missing information (add an index, a flag, remaining capacity).
2. **Wrong loop direction in knapsack** — 0/1 goes backwards, unbounded goes forwards.
3. **Coin Change II order** — to count *combinations* put coins in the outer loop; amount outer + coins inner counts *permutations*.
4. **Bad base cases / off-by-one** — use size `n+1` tables and make `dp[0]` mean "empty prefix".
5. **Using `0` as "impossible"** for min problems — use `float('inf')`, and for max problems with impossible states use `-inf`.
6. **Copying rows by reference** — `[[0]*m]*n` makes all rows the same list. Use `[[0]*m for _ in range(n)]`.
7. **Space-optimising too early** — get the 2D solution correct first, then reduce.
8. **Recursion depth** in Python top-down — for `n > 1000` use bottom-up or `sys.setrecursionlimit`.
9. Mutable default arguments (`memo={}`) shared across calls.
10. Assuming greedy works (Coin Change) — test with a counter-example.

## 8.3 What to say in the interview

1. "This has overlapping sub-problems and optimal substructure, so DP."
2. "Brute force is the recursion ___, it is exponential because ___."
3. "State is `dp[...]` meaning ___. Transition is ___. Base case ___."
4. "Time is states × transitions = ___, space is ___, and I can reduce it to ___ because only the previous row is needed."
5. Dry-run on a tiny example.


