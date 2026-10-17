# 🧠 Greedy Method in DSA (Python) — Concepts, Proofs, Techniques & 15 Interview Problems

> Easy-language guide: what greedy means → when it works and when it fails → how to prove it → all common greedy techniques → **15 standard interview problems** with explanation, commented code and verified output.

---

## 📑 Table of Contents
1. [What is the Greedy Method?](#1-what-is-the-greedy-method)
2. [The Two Properties Greedy Needs](#2-the-two-properties-greedy-needs)
3. [Greedy vs DP vs Brute Force / Backtracking](#3-greedy-vs-dp-vs-brute-force--backtracking)
4. [When Greedy Fails (with proof by example)](#4-when-greedy-fails)
5. [How to Prove a Greedy Algorithm](#5-how-to-prove-a-greedy-algorithm)
6. [A 5-Step Recipe to Solve Greedy Problems](#6-a-5-step-recipe-to-solve-greedy-problems)
7. [Greedy Techniques & Patterns](#7-greedy-techniques--patterns)
8. [Famous Greedy Algorithms](#8-famous-greedy-algorithms)
9. [15 Standard Interview Problems](#9-15-standard-interview-problems)
10. [Cheat Sheet, Mistakes & Practice List](#10-cheat-sheet-mistakes--practice-list)

---

## 1. What is the Greedy Method?

A **greedy algorithm** builds a solution **step by step**, and at **every step picks the option that looks best right now** (the *locally optimal* choice), **never reconsidering** it later. The hope (and for correct greedy problems, the proof) is that these local choices add up to a **globally optimal** solution.

**Real-life examples**
- **Giving change** with the biggest coin first (₹41 → 25 + 10 + 5 + 1).
- **Packing a bag**: grab the items with the best value per kg first.
- **Choosing meetings**: always pick the meeting that finishes earliest so more meetings fit.

```
Brute force:  try ALL combinations  →  slow (exponential)
Greedy:       make ONE best-looking choice per step, move on  →  fast (usually O(n log n) or O(n))
```

**Why interviewers love it:** greedy solutions are short and fast, but you must **justify why the local choice is safe**. That reasoning is what they test.

---

## 2. The Two Properties Greedy Needs

| Property | Meaning |
|---|---|
| **Greedy-choice property** | There is an optimal solution that **starts with the greedy choice**. So taking the greedy choice first never ruins optimality. |
| **Optimal substructure** | After making the greedy choice, the remaining problem is a **smaller instance of the same problem**, and its optimal solution combined with the choice gives the optimal solution of the whole. |

If both hold, greedy works. If the greedy choice can "block" a better overall solution (see Section 4), you need **Dynamic Programming**.

---

## 3. Greedy vs DP vs Brute Force / Backtracking

| | Brute force / Backtracking | Dynamic Programming | Greedy |
|---|---|---|---|
| Idea | try every possibility | try all choices but **reuse** sub-results | make **one** locally best choice |
| Reconsiders choices? | yes (backtracks) | yes (compares options) | **no** |
| Guaranteed optimal? | yes | yes | **only if proven** |
| Speed | exponential | polynomial | fastest |
| Example | subsets, N-Queens | knapsack 0/1, LCS | activity selection, Huffman |

> 💡 If you suspect greedy but can find a **counter-example**, switch to DP.

---

## 4. When Greedy Fails

### 4.1 Coin Change (greedy works only for "nice" coin systems)
With coins `{1, 5, 10, 25}` greedy is optimal. With `{1, 3, 4}` and amount `6` it is **not**.

```python
def greedy_coins(coins, amount):
    coins = sorted(coins, reverse=True)            # always try the biggest coin first
    used = []
    for c in coins:
        while amount >= c:                         # take this coin as many times as possible
            amount -= c
            used.append(c)
    return (len(used), used) if amount == 0 else (-1, used)

def dp_coins(coins, amount):                       # correct answer using dynamic programming
    INF = float('inf')
    dp = [0] + [INF] * amount                      # dp[a] = fewest coins to make amount a
    for a in range(1, amount + 1):
        for c in coins:
            if c <= a and dp[a - c] + 1 < dp[a]:
                dp[a] = dp[a - c] + 1
    return dp[amount] if dp[amount] != INF else -1

print(greedy_coins([1, 5, 10, 25], 41))            # greedy is optimal here
print(greedy_coins([1, 3, 4], 6), dp_coins([1, 3, 4], 6))   # greedy: 4+1+1 (3 coins), optimal: 3+3 (2 coins)
```
**Output**
```
(4, [25, 10, 5, 1])
(3, [4, 1, 1]) 2
```

### 4.2 0/1 Knapsack vs Fractional Knapsack
Taking the item with the best **value/weight ratio** first is optimal **only when items can be split** (fractional). For 0/1 items (take whole or nothing) it can fail.

```python
def greedy_01(items, cap):                         # items = [(value, weight)]
    items = sorted(items, key=lambda x: x[0] / x[1], reverse=True)   # best ratio first
    total = 0
    for v, w in items:
        if w <= cap:                               # take the whole item if it fits
            total += v
            cap -= w
    return total

def dp_01(items, cap):                             # correct answer for 0/1 knapsack
    dp = [0] * (cap + 1)
    for v, w in items:
        for c in range(cap, w - 1, -1):            # go backwards so each item is used once
            dp[c] = max(dp[c], dp[c - w] + v)
    return dp[cap]

items = [(60, 10), (100, 20), (120, 30)]
print(greedy_01(items, 50), dp_01(items, 50))      # greedy picks 60+100=160; best is 100+120=220
```
**Output**
```
160 220
```
**Lesson:** greedy needs a *proof*. A plausible-looking rule is not enough.

---

## 5. How to Prove a Greedy Algorithm

Three standard arguments (you do not need formal proofs in interviews, but you should be able to *explain the idea*):

### 5.1 Exchange Argument
Take any optimal solution and show you can **swap** one of its choices for the greedy choice **without making it worse**. Repeating this turns an optimal solution into the greedy one.
*Example (Activity Selection):* let the greedy pick the activity `g` that ends **first**. Any optimal schedule's first activity `x` ends no earlier than `g`. Replace `x` by `g`: it still doesn't overlap the rest and the count is unchanged. So an optimal solution exists that starts with `g`.

### 5.2 "Greedy Stays Ahead"
Show that after each step the greedy solution is **at least as good** as any other solution's partial result.
*Example (Jump Game):* after processing index `i`, the greedy `farthest` reach is ≥ the reach of any other strategy.

### 5.3 Proof by Contradiction / Counter-example
Assume a better solution exists and derive a contradiction; or **try to break your rule** with small tricky inputs (empty, ties, equal values, extremes).

> ✅ **Interview habit:** after coding, say *"This is greedy because choosing X first never prevents an optimal answer: swapping any other first choice for X doesn't make the result worse."*

---

## 6. A 5-Step Recipe to Solve Greedy Problems

1. **Understand the goal:** maximise/minimise what? (count, profit, length, cost)
2. **Find a candidate greedy rule:** "pick the earliest end", "pick the largest ratio", "pick the cheapest", "extend the farthest".
3. **Decide the ordering:** almost always **sort** by the rule's key, or use a **heap** when the "best" changes dynamically.
4. **Scan once**, make the choice if it is feasible, update the state.
5. **Test the rule** against small counter-examples; if it breaks, think DP.

---

## 7. Greedy Techniques & Patterns

| # | Technique | Idea | Examples |
|---|---|---|---|
| 1 | **Sort + scan** | sort by a key, then take whatever is feasible | Assign Cookies, Fractional Knapsack |
| 2 | **Interval scheduling** (sort by **end** time) | earliest finishing interval leaves the most room | Activity Selection, Non-overlapping Intervals, Burst Balloons |
| 3 | **Interval merging / covering** (sort by **start**) | extend current end while overlapping | Merge Intervals, Min Taps to Water Garden |
| 4 | **Running reach / farthest** | track the farthest position reachable | Jump Game I & II |
| 5 | **Running sum with reset** | if the running total goes negative, restart | Gas Station, Kadane's Algorithm |
| 6 | **Heap-based greedy** | the best choice changes, so keep a **priority queue** | Huffman, Job Sequencing, Refueling Stops, Task Scheduler (alt.), Meeting Rooms II |
| 7 | **Two-pass greedy** | satisfy left-neighbour rule, then right-neighbour rule | Candy, Trapping Rain Water |
| 8 | **Counting / frequency greedy** | decide using counts of items | Lemonade Change, Task Scheduler, Reorganize String |
| 9 | **Local gain accumulation** | add every positive local improvement | Stock Buy/Sell II |
| 10 | **Last-occurrence / boundary extension** | extend the current block to cover required ends | Partition Labels |
| 11 | **Monotonic stack greedy** | remove a larger digit before a smaller one | Remove K Digits, Remove Duplicate Letters |
| 12 | **Custom comparator sort** | sort with a pairwise rule | Largest Number, Queue Reconstruction by Height |

### Which sort key? (the most common decision)
| Goal | Sort by |
|---|---|
| Max number of non-overlapping intervals | **end time** ascending |
| Min number of points/arrows to hit all intervals | **end time** ascending |
| Merge overlapping intervals | **start time** ascending |
| Max profit with deadlines | **profit** descending, then slot by deadline |
| Max value with fractional items | **value/weight** descending |
| Minimise total waiting/completion time | **shortest job first** (burst time ascending) |
| Match smallest-sufficient to need | sort both, two pointers |

---

## 8. Famous Greedy Algorithms

| Algorithm | Greedy rule | Time |
|---|---|---|
| **Activity / Interval Selection** | earliest finish time | O(n log n) |
| **Fractional Knapsack** | highest value/weight ratio | O(n log n) |
| **Huffman Coding** | merge the two least frequent symbols | O(n log n) |
| **Job Sequencing with Deadlines** | highest profit, latest free slot | O(n log n) |
| **Dijkstra's Shortest Path** (non-negative weights) | settle the closest unvisited vertex | O((V+E) log V) |
| **Prim's MST** | cheapest edge leaving the current tree | O(E log V) |
| **Kruskal's MST** | cheapest edge that doesn't form a cycle (Union-Find) | O(E log E) |
| **Optimal Merge Pattern / Connect Ropes** | merge the two smallest | O(n log n) |
| **Shortest Job First scheduling** | smallest burst time first | O(n log n) |
| **Minimum Platforms** | sort arrivals & departures, sweep | O(n log n) |

*(Graph algorithms Dijkstra/Prim/Kruskal are covered in the Graph topic.)*

---

## 9. 15 Standard Interview Problems

---

### 🟢 Problem 1 — Assign Cookies (Easy) · *Sort + two pointers*
**Problem:** child `i` needs greed `g[i]`; cookie `j` has size `s[j]`; a child is content if `s[j] >= g[i]`. Each child gets at most one cookie. Maximise content children.
**Greedy rule:** give each cookie to the **least greedy child it can satisfy**. Sort both lists; for each cookie (smallest first) satisfy the next unsatisfied child if possible.

```python
def find_content_children(g, s):
    g.sort()                                       # children from least to most greedy
    s.sort()                                       # cookies from smallest to largest
    child = 0                                      # next child to satisfy
    for cookie in s:                               # try cookies from the smallest
        if child < len(g) and cookie >= g[child]:  # this cookie satisfies the current child
            child += 1                             # move to the next child
    return child                                   # number of satisfied children

print(find_content_children([1, 2, 3], [1, 1]))    # only the child with greed 1 can be satisfied
print(find_content_children([1, 2], [1, 2, 3]))    # both can be satisfied
```
**Output**
```
1
2
```
**Why it works:** using a big cookie on a small greed wastes it (exchange argument). **Complexity:** O(n log n + m log m) time, O(1) extra space.

---

### 🟢 Problem 2 — Lemonade Change (Easy) · *Counting greedy*
**Problem:** each lemonade costs ₹5; customers pay with 5, 10 or 20. Start with no change. Can you give correct change to everyone?
**Greedy rule:** when giving change for 20, **prefer 10 + 5** (keep the more flexible 5s for later), else 5+5+5.

```python
def lemonade_change(bills):
    five = ten = 0                                 # number of 5 and 10 bills in hand
    for b in bills:
        if b == 5:
            five += 1                              # no change needed
        elif b == 10:
            if not five:                           # need one 5 as change
                return False
            five -= 1
            ten += 1
        else:                                      # b == 20, change = 15
            if ten and five:                       # greedy: use 10 + 5 first (5s are more valuable)
                ten -= 1
                five -= 1
            elif five >= 3:                        # otherwise three 5s
                five -= 3
            else:
                return False
    return True

print(lemonade_change([5, 5, 5, 10, 20]))
print(lemonade_change([5, 5, 10, 10, 20]))
```
**Output**
```
True
False
```
**Why:** 5s can be used for both 10 and 20, 10s only for 20, so spend the less flexible bill (10) first. **Complexity:** O(n), O(1).

---

### 🟢 Problem 3 — Best Time to Buy and Sell Stock II (Easy) · *Local gain accumulation*
**Problem:** unlimited transactions (hold at most one share). Maximise profit.
**Greedy rule:** collect **every positive day-to-day increase**. A long climb equals the sum of its daily steps.

```python
def max_profit(prices):
    profit = 0
    for i in range(1, len(prices)):
        if prices[i] > prices[i - 1]:              # price went up today
            profit += prices[i] - prices[i - 1]    # "buy yesterday, sell today"
    return profit

print(max_profit([7, 1, 5, 3, 6, 4]))              # (5-1) + (6-3) = 7
print(max_profit([1, 2, 3, 4, 5]))                 # one long climb = 4
print(max_profit([7, 6, 4, 3, 1]))                 # only falls -> 0
```
**Output**
```
7
4
0
```
**Complexity:** O(n), O(1).

---

### 🟡 Problem 4 — Jump Game (Medium) · *Farthest reach*
**Problem:** `nums[i]` is the maximum jump length from index `i`. Can you reach the last index?
**Greedy rule:** keep the **farthest index reachable** so far. If you stand at an index beyond it, you are stuck.

```python
def can_jump(nums):
    farthest = 0                                   # farthest index we can reach so far
    for i, x in enumerate(nums):
        if i > farthest:                           # current index is unreachable -> stuck
            return False
        farthest = max(farthest, i + x)            # extend the reach using this position
    return True                                    # we never got stuck

print(can_jump([2, 3, 1, 1, 4]))
print(can_jump([3, 2, 1, 0, 4]))                   # always lands on the 0 at index 3
```
**Output**
```
True
False
```
**Complexity:** O(n), O(1).

---

### 🟡 Problem 5 — Jump Game II (Medium) · *BFS-like greedy windows*
**Problem:** minimum number of jumps to reach the last index (always reachable).
**Greedy rule:** think in **windows**. `cur_end` = how far we can get with the current number of jumps; `farthest` = how far we could get with one more jump. When we reach `cur_end`, we **must** jump, and the next window ends at `farthest`.

```python
def jump(nums):
    jumps = 0                                      # jumps used so far
    cur_end = 0                                    # end of the range reachable with `jumps` jumps
    farthest = 0                                   # farthest reachable with one more jump
    for i in range(len(nums) - 1):                 # no need to jump from the last index
        farthest = max(farthest, i + nums[i])      # best reach among all positions in the current window
        if i == cur_end:                           # reached the edge of the window: must take a jump
            jumps += 1
            cur_end = farthest                     # new window
    return jumps

print(jump([2, 3, 1, 1, 4]))                       # 0 -> 1 -> 4
print(jump([2, 3, 0, 1, 4]))
print(jump([1]))                                   # already at the end
```
**Output**
```
2
2
0
```
**Complexity:** O(n), O(1). (DP would be O(n²).)

---

### 🟡 Problem 6 — Gas Station (Medium) · *Running sum with reset*
**Problem:** circular route; `gas[i]` available at station `i`, `cost[i]` to reach `i+1`. Find the starting station that completes the circuit (unique if it exists), else `-1`.
**Greedy facts:**
1. If `sum(gas) < sum(cost)`, impossible.
2. If the tank goes negative at station `i` starting from `start`, **no station between `start` and `i` can be a valid start**, so try `i + 1`.

```python
def can_complete_circuit(gas, cost):
    if sum(gas) < sum(cost):                       # total fuel is not enough for any start
        return -1
    tank = 0                                       # fuel balance since the candidate start
    start = 0                                      # candidate starting station
    for i in range(len(gas)):
        tank += gas[i] - cost[i]                   # net fuel after traveling station i -> i+1
        if tank < 0:                               # cannot reach i+1 from `start`
            start = i + 1                          # none of start..i can work; next candidate
            tank = 0                               # reset the tank
    return start                                   # total >= 0 guarantees this candidate works

print(can_complete_circuit([1, 2, 3, 4, 5], [3, 4, 5, 1, 2]))
print(can_complete_circuit([2, 3, 4], [3, 4, 3]))
```
**Output**
```
3
-1
```
**Complexity:** O(n), O(1).

---

### 🟡 Problem 7 — Activity Selection / Non-overlapping Intervals (Medium) · *Sort by end time*
**Problem:** choose the maximum number of non-overlapping intervals (equivalently, the minimum number to **remove**).
**Greedy rule:** sort by **end time**; always take the interval that ends first and doesn't overlap the last chosen one.

```python
def max_activities(intervals):
    intervals = sorted(intervals, key=lambda x: x[1])   # earliest finishing first
    chosen = []
    end = float('-inf')                            # end time of the last chosen interval
    for s, e in intervals:
        if s >= end:                               # does not overlap the last chosen one
            chosen.append([s, e])                  # take it
            end = e
    return chosen

def erase_overlap_intervals(intervals):
    return len(intervals) - len(max_activities(intervals))   # removals = total - kept

print(max_activities([[1, 3], [2, 4], [3, 5], [0, 6], [5, 7], [8, 9], [5, 9]]))
print(erase_overlap_intervals([[1, 2], [2, 3], [3, 4], [1, 3]]))    # remove [1,3]
print(erase_overlap_intervals([[1, 2], [1, 2], [1, 2]]))
print(erase_overlap_intervals([[1, 2], [2, 3]]))                    # touching is allowed
```
**Output**
```
[[1, 3], [3, 5], [5, 7], [8, 9]]
1
2
0
```
**Why sort by end (not start or length)?** Earliest finish leaves the largest free time for the rest (exchange argument, Section 5.1). **Complexity:** O(n log n).

---

### 🟡 Problem 8 — Minimum Number of Arrows to Burst Balloons (Medium) · *Interval stabbing*
**Problem:** balloons are intervals `[start, end]` on the x-axis. One arrow at `x` bursts every balloon with `start <= x <= end`. Minimum arrows?
**Greedy rule:** sort by **end**; shoot at the end of the first balloon (it hits as many as possible); skip all balloons that start before or at that point.

```python
def find_min_arrow_shots(points):
    points.sort(key=lambda x: x[1])                # sort by right end
    arrows = 0
    end = float('-inf')                            # position of the last arrow
    for s, e in points:
        if s > end:                                # this balloon is NOT hit by the last arrow
            arrows += 1                            # shoot a new arrow at its right end
            end = e
    return arrows

print(find_min_arrow_shots([[10, 16], [2, 8], [1, 6], [7, 12]]))
print(find_min_arrow_shots([[1, 2], [3, 4], [5, 6], [7, 8]]))
print(find_min_arrow_shots([[1, 2], [2, 3], [3, 4], [4, 5]]))
```
**Output**
```
2
4
2
```
**Complexity:** O(n log n).

---

### 🟡 Problem 9 — Partition Labels (Medium) · *Last occurrence boundary*
**Problem:** split the string into as many parts as possible so that each letter appears in **at most one part**. Return part sizes.
**Greedy rule:** record each letter's **last index**. Extend the current part's end to the last occurrence of every letter seen; when the scan index reaches that end, the part can be closed.

```python
def partition_labels(s):
    last = {ch: i for i, ch in enumerate(s)}       # last position of every character
    res = []
    start = end = 0                                # current part is s[start..end]
    for i, ch in enumerate(s):
        end = max(end, last[ch])                   # this part must extend to ch's last occurrence
        if i == end:                               # all letters in the part end here -> close it
            res.append(end - start + 1)
            start = i + 1                          # next part begins
    return res

print(partition_labels("ababcbacadefegdehijhklij"))
print(partition_labels("eccbbbbdec"))
```
**Output**
```
[9, 7, 8]
[10]
```
**Complexity:** O(n) time, O(1) space (26 letters).

---

### 🟡 Problem 10 — Fractional Knapsack (Medium) · *Best value-to-weight ratio*
**Problem:** items have `(value, weight)`; items **can be split**. Maximise value in a bag of capacity `W`.
**Greedy rule:** sort by `value / weight` descending; take whole items while they fit, then take a **fraction** of the next.

```python
def fractional_knapsack(items, cap):
    items = sorted(items, key=lambda x: x[0] / x[1], reverse=True)   # best ratio first
    total = 0.0
    for value, weight in items:
        if cap >= weight:                          # the whole item fits
            total += value
            cap -= weight
        else:                                      # take only the fraction that fits, then stop
            total += value * cap / weight
            break
    return total

print(fractional_knapsack([(60, 10), (100, 20), (120, 30)], 50))   # 60 + 100 + (2/3)*120 = 240
print(fractional_knapsack([(500, 30)], 10))
```
**Output**
```
240.0
166.66666666666666
```
**Compare with Section 4.2:** the same items give 160 (greedy) vs 220 (DP) for the **0/1** version; but fractional greedy is optimal. **Complexity:** O(n log n).

---

### 🟡 Problem 11 — Task Scheduler (Medium) · *Frequency greedy (formula)*
**Problem:** tasks are letters; the same task must be separated by at least `n` intervals (idle allowed). Minimum total time?
**Greedy idea:** the most frequent task decides the frame. Place it first and last in `max_freq - 1` blocks of size `n + 1`; other tasks fill the gaps. If many tasks tie for the max frequency, they occupy the final row.
`time = max( len(tasks), (max_freq - 1) * (n + 1) + count_of_max_freq_tasks )`

```python
from collections import Counter

def least_interval(tasks, n):
    freq = Counter(tasks)                          # how many times each task appears
    mx = max(freq.values())                        # highest frequency
    cnt = sum(1 for v in freq.values() if v == mx) # how many tasks have that highest frequency
    frame = (mx - 1) * (n + 1) + cnt               # slots needed with idle time included
    return max(len(tasks), frame)                  # never less than the number of tasks (no idles needed)

print(least_interval(["A", "A", "A", "B", "B", "B"], 2))   # A B _ A B _ A B
print(least_interval(["A", "A", "A", "B", "B", "B"], 0))
print(least_interval(["A", "A", "A", "A", "A", "A", "B", "C", "D", "E", "F", "G"], 2))
```
**Output**
```
8
6
16
```
**Complexity:** O(n). (Heap + cooldown queue simulation gives the same answer in O(n log 26).)

---

### 🔴 Problem 12 — Candy (Hard) · *Two-pass greedy*
**Problem:** children in a line with ratings; each gets ≥ 1 candy; a child with a **higher rating than a neighbour** gets **more** candy than that neighbour. Minimum total candies?
**Greedy rule:** handle the two constraints **separately**: left→right (compare with left neighbour), then right→left (compare with right neighbour, keep the max).

```python
def candy(ratings):
    n = len(ratings)
    c = [1] * n                                    # everyone starts with one candy
    for i in range(1, n):                          # pass 1: satisfy "higher than LEFT neighbour"
        if ratings[i] > ratings[i - 1]:
            c[i] = c[i - 1] + 1
    for i in range(n - 2, -1, -1):                 # pass 2: satisfy "higher than RIGHT neighbour"
        if ratings[i] > ratings[i + 1]:
            c[i] = max(c[i], c[i + 1] + 1)         # max keeps the left-pass requirement too
    return sum(c)

print(candy([1, 0, 2]))                            # candies 2,1,2
print(candy([1, 2, 2]))                            # candies 1,2,1
print(candy([1, 3, 2, 2, 1]))
```
**Output**
```
5
4
7
```
**Complexity:** O(n) time, O(n) space (an O(1)-space slope-counting version exists).

---

### 🔴 Problem 13 — Job Sequencing with Deadlines (Medium-Hard) · *Max profit + latest free slot*
**Problem:** each job takes 1 unit of time, has a deadline and a profit; one job at a time. Maximise profit.
**Greedy rule:** consider jobs in **decreasing profit**; put each job in the **latest free time slot on or before its deadline** (keeping earlier slots free for other jobs). If no slot is free, skip the job.

```python
def job_sequencing(jobs):                          # jobs = [(id, deadline, profit)]
    jobs = sorted(jobs, key=lambda j: j[2], reverse=True)   # most profitable first
    max_d = max(j[1] for j in jobs)                # latest deadline = number of slots
    slot = [None] * (max_d + 1)                    # slot[t] = job done at time t (1-indexed)
    profit = count = 0
    for jid, d, p in jobs:
        for t in range(min(d, max_d), 0, -1):      # try the latest possible free slot first
            if slot[t] is None:
                slot[t] = jid                      # schedule the job here
                profit += p
                count += 1
                break
    return count, profit

print(job_sequencing([(1, 4, 20), (2, 1, 10), (3, 1, 40), (4, 1, 30)]))   # best: job 3 + job 1
print(job_sequencing([(1, 2, 100), (2, 1, 19), (3, 2, 27), (4, 1, 25), (5, 3, 15)]))
```
**Output**
```
(2, 60)
(3, 142)
```
**Complexity:** O(n²) with the slot scan (O(n log n) with a DSU/heap version).

---

### 🔴 Problem 14 — Huffman Coding (Hard) · *Min-heap, merge two smallest*
**Problem:** build a **prefix-free binary code** that minimises the total encoded length, given character frequencies.
**Greedy rule:** repeatedly **merge the two least frequent** nodes into a parent whose frequency is their sum, until one tree remains. Left edge = `0`, right edge = `1`. Frequent characters end up near the root (short codes).

```python
import heapq

def huffman_codes(freq):
    heap, order = [], 0                            # order = tie-breaker so tuples never compare nodes
    for ch, f in freq.items():
        heapq.heappush(heap, (f, order, ch))       # leaf node: (frequency, order, character)
        order += 1
    while len(heap) > 1:
        f1, _, left = heapq.heappop(heap)          # least frequent
        f2, _, right = heapq.heappop(heap)         # second least frequent
        heapq.heappush(heap, (f1 + f2, order, (left, right)))   # merged parent node
        order += 1
    codes = {}
    def walk(node, code):
        if isinstance(node, str):                  # leaf: store the code (at least '0' for a single symbol)
            codes[node] = code or '0'
            return
        walk(node[0], code + '0')                  # left edge = 0
        walk(node[1], code + '1')                  # right edge = 1
    walk(heap[0][2], '')
    return codes

freq = {'a': 5, 'b': 9, 'c': 12, 'd': 13, 'e': 16, 'f': 45}
codes = huffman_codes(freq)
for ch in sorted(codes):
    print(ch, codes[ch])                           # note: frequent 'f' gets the shortest code
print("total bits:", sum(freq[ch] * len(codes[ch]) for ch in freq))   # vs 3*100 = 300 with fixed 3-bit codes
```
**Output**
```
a 1100
b 1101
c 100
d 101
e 111
f 0
total bits: 224
```
**Complexity:** O(n log n). Used in compression (ZIP, JPEG, MP3 stages).

---

### 🔴 Problem 15 — Minimum Number of Refueling Stops (Hard) · *Max-heap of passed stations*
**Problem:** a car starts with `start_fuel` and must travel `target` miles; stations `[position, fuel]` sorted by position; 1 liter = 1 mile. Minimum refuels (or `-1`).
**Greedy rule:** drive as far as possible. When you **cannot** reach the next place, retroactively refuel at the **passed station with the most fuel** (max-heap). This delays each stop until it is truly needed and always picks the best one.

```python
import heapq

def min_refuel_stops(target, start_fuel, stations):
    heap = []                                      # max-heap (store negatives) of fuel amounts at passed stations
    reach = start_fuel                             # farthest distance reachable with the fuel used so far
    stops = 0
    for pos, gas in stations + [[target, 0]]:      # treat the destination as a final "station"
        while reach < pos and heap:                # cannot reach this point yet
            reach += -heapq.heappop(heap)          # refuel at the best station we already passed
            stops += 1
        if reach < pos:                            # no more stations to use -> impossible
            return -1
        heapq.heappush(heap, -gas)                 # this station becomes available for later refueling
    return stops

print(min_refuel_stops(100, 10, [[10, 60], [20, 30], [30, 30], [60, 40]]))
print(min_refuel_stops(1, 1, []))
print(min_refuel_stops(100, 1, [[10, 100]]))
```
**Output**
```
2
0
-1
```
**Complexity:** O(n log n) time, O(n) space.

---

## 10. Cheat Sheet, Mistakes & Practice List

### Problem → technique map
| # | Problem | Greedy rule / technique |
|---|---|---|
| 1 | Assign Cookies | sort + smallest sufficient cookie |
| 2 | Lemonade Change | spend the less flexible bill first |
| 3 | Stock II | sum all positive differences |
| 4 | Jump Game | track farthest reach |
| 5 | Jump Game II | window / BFS-levels of reach |
| 6 | Gas Station | running sum, reset on negative |
| 7 | Activity Selection | sort by end time |
| 8 | Burst Balloons | sort by end, shoot at the end |
| 9 | Partition Labels | extend to last occurrence |
| 10 | Fractional Knapsack | best value/weight ratio |
| 11 | Task Scheduler | most frequent task decides frame |
| 12 | Candy | two passes (left, right) |
| 13 | Job Sequencing | max profit, latest free slot |
| 14 | Huffman Coding | merge two smallest (heap) |
| 15 | Refueling Stops | max-heap of passed stations |

### Greedy template
```py
# Sort-and-scan template
items.sort(key=<greedy key>)
state = initial
for item in items:
    if feasible(item, state):      # can I take it without breaking constraints?
        take(item)                 # commit (never undone)
        update(state)
return answer

# Heap template (best choice changes dynamically)
heap = []
for x in stream:
    push candidate into heap
    while not feasible_now and heap:
        use best candidate from heap
```

### Common mistakes
1. **Assuming greedy works without checking** — try counter-examples (Section 4).
2. **Wrong sort key** (start vs end time, ratio vs value, ascending vs descending).
3. Forgetting **ties** and **equal values** (touching intervals `s >= end` vs `s > end`).
4. Greedy on **0/1 knapsack, general coin change, longest path** problems (need DP).
5. Not handling **empty input / single element**.
6. Mixing up **"minimum removed"** and **"maximum kept"** in interval problems.
7. In heap problems, forgetting Python's `heapq` is a **min-heap** (use negatives for max).
8. Comparing floats for ratios (prefer cross-multiplication or `fractions` when exactness matters).

### Interview talking points
- "Greedy makes the locally best choice and never reconsiders; it's correct when the greedy-choice property and optimal substructure hold."
- "I justify with an exchange argument: swapping any other first choice for the greedy choice doesn't hurt."
- "If a counter-example exists, e.g. coins {1,3,4}, I switch to DP."
- "Sort-based greedy is O(n log n); the greedy scan itself is O(n)."

### Practice list
| Level | Problems |
|---|---|
| Easy | Assign Cookies (455), Lemonade Change (860), Best Time to Buy and Sell Stock II (122), Maximum Units on a Truck (1710), Minimum Absolute Difference (1200), Split a String in Balanced Strings (1221), Array Partition I (561) |
| Medium | Jump Game (55), Jump Game II (45), Gas Station (134), Non-overlapping Intervals (435), Minimum Arrows to Burst Balloons (452), Partition Labels (763), Task Scheduler (621), Merge Intervals (56), Queue Reconstruction by Height (406), Two City Scheduling (1029), Remove K Digits (402), Reorganize String (767), Hand of Straights (846), Boats to Save People (881), Largest Number (179), Meeting Rooms II (253), Minimum Number of Taps to Open (1326), Video Stitching (1024) |
| Hard | Candy (135), Minimum Number of Refueling Stops (871), IPO (502), Course Schedule III (630), Remove Duplicate Letters (316), Patching Array (330), Create Maximum Number (321), Minimum Cost to Hire K Workers (857) |

### ✅ Final advice for 12 November 2026
- Learn to **recognise the sort key**: intervals → end time; profit → descending; ratio problems → value/weight.
- For each greedy problem, say out loud *why the greedy choice is safe* (exchange argument). Interviewers weigh this as much as the code.
- Greedy and DP are siblings: if the greedy rule breaks on a small example, move to DP.
- Topics finished: **Math → Arrays → Searching → Sorting → Strings → Recursion → Hashing → Stack & Queue → Linked List → Trees & BST → Greedy**. Suggested next: **Heap / Priority Queue, Graphs, Dynamic Programming, Trie**. 🚀
