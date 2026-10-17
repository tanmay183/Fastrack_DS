# 📚 Stack & Queue in DSA (Python) — From Construction to Advanced Interview Problems

> Easy-language guide: what they are → how to build them (list, linked list, circular array, deque) → special variants (Deque, Priority Queue, Monotonic Stack/Deque, Min Stack) → patterns → **15 standard interview problems** with commented code and verified output.

---

## 📑 Table of Contents
1. [Big Picture: Stack vs Queue](#1-big-picture-stack-vs-queue)
2. [Stack — Concept](#2-stack--concept)
3. [Building a Stack (3 ways)](#3-building-a-stack-3-ways)
4. [Queue — Concept](#4-queue--concept)
5. [Building a Queue (list problem, deque, circular, linked list)](#5-building-a-queue)
6. [Deque & Priority Queue](#6-deque--priority-queue)
7. [Advanced Ideas: Monotonic Stack, Monotonic Deque, Min Stack, Expressions](#7-advanced-ideas)
8. [Patterns & Recognition Table](#8-patterns--recognition-table)
9. [15 Standard Interview Problems](#9-15-standard-interview-problems)
10. [Cheat Sheet, Mistakes & Practice List](#10-cheat-sheet-mistakes--practice-list)

---

## 1. Big Picture: Stack vs Queue

Both are **linear data structures** where you can only touch the ends. The difference is *which end you remove from*.

| | **Stack** | **Queue** |
|---|---|---|
| Rule | **LIFO** — Last In, First Out | **FIFO** — First In, First Out |
| Real life | pile of plates, undo button, browser back | line at a ticket counter, printer queue |
| Add | `push` (top) | `enqueue` (rear/back) |
| Remove | `pop` (top) | `dequeue` (front) |
| Look | `peek/top` | `front` |
| Where used | recursion, DFS, parentheses, undo, expression evaluation | BFS, scheduling, buffering, level-order, sliding window |

```
STACK (one open end)         QUEUE (two ends)
   push ↓  ↑ pop              enqueue →  [ 1 | 2 | 3 | 4 ]  → dequeue
       | 4 |  <- top                      rear            front
       | 3 |
       | 2 |
       | 1 |
```

---

## 2. Stack — Concept

**Operations (all O(1))**
| Operation | Meaning |
|---|---|
| `push(x)` | put `x` on top |
| `pop()` | remove and return the top element |
| `peek()` / `top()` | look at the top without removing |
| `is_empty()` | is the stack empty? |
| `size()` | number of elements |

**Errors you must know**
- **Underflow:** `pop()`/`peek()` on an empty stack.
- **Overflow:** `push()` on a full stack (only for fixed-size stacks).

**Where the stack is hidden in real life**
- The **call stack** of recursion (every function call is pushed, returns pop it).
- **Undo/Redo**, browser history.
- **DFS** (depth-first search), backtracking.
- Matching **brackets**, evaluating **expressions**, converting infix → postfix.
- **Monotonic stack** problems (next greater element, histogram).

---

## 3. Building a Stack (3 ways)

### 3.1 Using a Python list (most common in interviews)
`list.append` = push, `list.pop()` = pop → both **O(1)** at the end of the list.

```python
class Stack:
    def __init__(self):
        self._data = []                            # internal list; the END of the list is the TOP

    def push(self, x):
        self._data.append(x)                       # add on top, O(1) amortized

    def pop(self):
        if self.is_empty():                        # nothing to remove -> underflow
            raise IndexError("pop from empty stack (underflow)")
        return self._data.pop()                    # remove and return the top, O(1)

    def peek(self):
        if self.is_empty():
            raise IndexError("peek from empty stack")
        return self._data[-1]                      # last element is the top

    def is_empty(self):
        return len(self._data) == 0

    def size(self):
        return len(self._data)

s = Stack()
for x in [10, 20, 30]:
    s.push(x)                                      # stack (bottom -> top): 10 20 30
print(s.peek(), s.size())                          # top is 30
print(s.pop(), s.pop())                            # 30 then 20 come out (LIFO)
print(s.size(), s.is_empty())
s.pop()                                            # removes 10, now empty
try:
    s.pop()                                        # pop from empty stack
except IndexError as e:
    print("Error:", e)
```
**Output**
```
30 3
30 20
1 False
Error: pop from empty stack (underflow)
```

### 3.2 Using a Linked List (no resizing, true O(1) always)
Push/pop at the **head** of a singly linked list.

```python
class Node:
    def __init__(self, val, next=None):
        self.val = val                             # stored value
        self.next = next                           # pointer to the node below

class LinkedStack:
    def __init__(self):
        self.head = None                           # head = top of the stack
        self._size = 0

    def push(self, x):
        self.head = Node(x, self.head)             # new node points to old top, becomes new top
        self._size += 1

    def pop(self):
        if self.head is None:                      # empty -> underflow
            raise IndexError("pop from empty stack")
        val = self.head.val
        self.head = self.head.next                 # move top pointer down
        self._size -= 1
        return val

    def peek(self):
        if self.head is None:
            raise IndexError("peek from empty stack")
        return self.head.val

    def is_empty(self):
        return self.head is None

ls = LinkedStack()
ls.push(1); ls.push(2); ls.push(3)
print(ls.peek())                                   # 3 is on top
print(ls.pop(), ls.pop(), ls.pop())                # 3 2 1
print(ls.is_empty())
```
**Output**
```
3
3 2 1
True
```

### 3.3 Fixed-size array stack (shows OVERFLOW)
Useful to understand how stacks work in C/Java; in Python we simulate it.

```python
class ArrayStack:
    def __init__(self, capacity):
        self.arr = [None] * capacity               # pre-allocated fixed array
        self.top = -1                              # index of top element (-1 = empty)
        self.cap = capacity

    def push(self, x):
        if self.top == self.cap - 1:               # array full -> overflow
            raise OverflowError("stack overflow")
        self.top += 1                              # move top up
        self.arr[self.top] = x                     # store element

    def pop(self):
        if self.top == -1:                         # empty -> underflow
            raise IndexError("stack underflow")
        x = self.arr[self.top]                     # read top
        self.top -= 1                              # logically remove it
        return x

st = ArrayStack(2)
st.push(1)
st.push(2)
try:
    st.push(3)                                     # capacity is only 2
except OverflowError as e:
    print("Error:", e)
print(st.pop(), st.pop())
```
**Output**
```
Error: stack overflow
2 1
```

**Complexity (all versions):** push, pop, peek = **O(1)**; space O(n).

---

## 4. Queue — Concept

**Operations**
| Operation | Meaning |
|---|---|
| `enqueue(x)` | add `x` at the **rear** |
| `dequeue()` | remove and return the **front** element |
| `front()` | look at the front |
| `is_empty()`, `size()` | as usual |

**Types of queues**
| Type | Idea |
|---|---|
| **Simple (linear) queue** | FIFO |
| **Circular queue** | array whose end wraps to the start (reuses freed space) |
| **Deque** | insert/remove at **both** ends |
| **Priority queue** | remove the element with the **highest priority**, not the oldest (heap) |

**Where used:** BFS, level-order tree traversal, task scheduling, CPU scheduling (round robin), printer spooling, buffering streams, sliding-window problems, simulation (hot potato).

---

## 5. Building a Queue

### 5.1 Why NOT `list.pop(0)`
`list.pop(0)` removes the first element and **shifts all others** left → **O(n)** per dequeue. A loop of n dequeues becomes O(n²). Use `collections.deque` instead.

### 5.2 Using `collections.deque` (best in Python)
`append` = enqueue (O(1)), `popleft` = dequeue (O(1)).

```python
from collections import deque

q = deque()                                        # empty queue
q.append(1)                                        # enqueue 1
q.append(2)                                        # enqueue 2
q.append(3)                                        # enqueue 3
print(q[0])                                        # front element (peek) without removing
print(q.popleft())                                 # dequeue -> 1 (first in, first out)
print(list(q), len(q))                             # remaining queue and size
print(bool(q))                                     # True if not empty (empty deque is falsy)
```
**Output**
```
1
1
[2, 3] 2
True
```

### 5.3 Circular Queue (array with wrap-around)
**Problem with a plain array queue:** after many dequeues the front moves right and the left space is wasted. **Fix:** treat the array as a circle using `% capacity`.

- `front` = index of the first element
- `rear = (front + size) % capacity` = where the next element goes

```python
class CircularQueue:
    def __init__(self, k):
        self.q = [None] * k                        # fixed-size storage
        self.cap = k                               # capacity
        self.front = 0                             # index of the first element
        self.size = 0                              # number of stored elements

    def enqueue(self, x):
        if self.size == self.cap:                  # full
            return False
        rear = (self.front + self.size) % self.cap # next free slot, wraps around the end
        self.q[rear] = x
        self.size += 1
        return True

    def dequeue(self):
        if self.size == 0:                         # empty
            return False
        self.front = (self.front + 1) % self.cap   # advance front (wraps around)
        self.size -= 1
        return True

    def front_value(self):
        return -1 if self.size == 0 else self.q[self.front]

cq = CircularQueue(3)
print(cq.enqueue(1), cq.enqueue(2), cq.enqueue(3))   # fills the queue
print(cq.enqueue(4))                                 # full -> False
print(cq.dequeue())                                  # frees one slot at the front
print(cq.enqueue(4))                                 # 4 reuses the freed slot (wrap-around)
print(cq.q, cq.front_value())                        # raw array shows the wrap, front is 2
```
**Output**
```
True True True
False
True
True
[4, 2, 3] 2
```

### 5.4 Queue using a Linked List
Keep both a `head` (front, for dequeue) and a `tail` (rear, for enqueue) → both O(1).

```python
class Node:
    def __init__(self, val):
        self.val = val
        self.next = None

class LinkedQueue:
    def __init__(self):
        self.head = None                           # front of the queue
        self.tail = None                           # rear of the queue

    def enqueue(self, x):
        node = Node(x)
        if self.tail is None:                      # queue is empty: node is both head and tail
            self.head = self.tail = node
        else:
            self.tail.next = node                  # link after the current rear
            self.tail = node                       # new rear

    def dequeue(self):
        if self.head is None:
            raise IndexError("dequeue from empty queue")
        val = self.head.val
        self.head = self.head.next                 # move front pointer forward
        if self.head is None:                      # queue became empty -> reset tail too
            self.tail = None
        return val

lq = LinkedQueue()
for x in [5, 6, 7]:
    lq.enqueue(x)
print(lq.dequeue(), lq.dequeue())                  # FIFO: 5 then 6
lq.enqueue(8)
print(lq.dequeue(), lq.dequeue())                  # 7 then 8
```
**Output**
```
5 6
7 8
```

**Complexity:** enqueue, dequeue, front = **O(1)**.

---

## 6. Deque & Priority Queue

### 6.1 Deque (double-ended queue)
Insert/remove at **both** ends in O(1). Use it as a stack, a queue, or a sliding-window helper.

```python
from collections import deque

d = deque([2, 3])
d.appendleft(1)                                    # add at the front
d.append(4)                                        # add at the back
print(list(d))
print(d.popleft(), d.pop())                        # remove from front and back
d.rotate(1)                                        # rotate right by 1 step
print(list(d))
dq = deque(maxlen=3)                               # fixed-size window: old items fall off automatically
for x in [1, 2, 3, 4, 5]:
    dq.append(x)
print(list(dq))
```
**Output**
```
[1, 2, 3, 4]
1 4
[3, 2]
[3, 4, 5]
```

### 6.2 Priority Queue (heap)
Each element has a priority; `pop` returns the **smallest** priority first (min-heap). Python: `heapq`. Push/pop = **O(log n)**, peek = O(1).
For a **max-heap** push negative values.

```python
import heapq

pq = []                                            # a plain list used as a heap
heapq.heappush(pq, (3, "write code"))              # (priority, task): lower number = more urgent
heapq.heappush(pq, (1, "fix bug"))
heapq.heappush(pq, (2, "review PR"))
print(pq[0])                                       # peek the most urgent task, O(1)
while pq:
    print(heapq.heappop(pq))                       # tasks come out in priority order, not insertion order
maxh = []
for x in [4, 9, 2]:
    heapq.heappush(maxh, -x)                       # negate to simulate a max-heap
print(-heapq.heappop(maxh))                        # largest value
```
**Output**
```
(1, 'fix bug')
(1, 'fix bug')
(2, 'review PR')
(3, 'write code')
9
```
**Use:** Dijkstra, top-K, merge K sorted lists, task scheduling, median of a stream.

### 6.3 Which Python structure to use?
| Need | Use |
|---|---|
| Stack | `list` (`append`, `pop`) |
| Queue / BFS | `collections.deque` (`append`, `popleft`) |
| Deque / sliding window | `collections.deque` |
| Priority queue | `heapq` |
| Thread-safe queue | `queue.Queue` |
| Never | `list.pop(0)` / `list.insert(0, x)` for queues |

---

## 7. Advanced Ideas

### 7.1 Monotonic Stack — "next greater / smaller element" in O(n)
Keep the stack **sorted** (e.g. decreasing from bottom to top). When a new element breaks the order, **pop** the elements it beats; those popped elements have found their answer. Each element is pushed and popped once → **O(n)**.

**Trace** of "next greater element" for `[2, 1, 2, 4, 3]` (stack holds indexes, shown as values):
| i | x | stack before | action | answer filled |
|---|---|---|---|---|
| 0 | 2 | [] | push | – |
| 1 | 1 | [2] | 1 < 2, push | – |
| 2 | 2 | [2,1] | pop 1 (2 > 1) → ans[1]=2, push | ans[1]=2 |
| 3 | 4 | [2,2] | pop 2, pop 2 → ans=4,4, push | ans[0]=4, ans[2]=4 |
| 4 | 3 | [4] | push | – |
Leftover (4, 3) have no greater element → `-1`.

**Template**
```py
stack = []                                    # stores indexes
for i, x in enumerate(nums):
    while stack and nums[stack[-1]] < x:      # x is the "next greater" for everything it pops
        j = stack.pop()
        answer[j] = x                         # (or i - j for distances)
    stack.append(i)
```
**Problems:** 7, 8, 14 (Daily Temperatures, Next Greater II, Largest Rectangle).

### 7.2 Monotonic Deque — max/min of every window
Same idea with a deque so the **front** always holds the current window's maximum, and old indexes are removed from the front. **Problem:** 15 (Sliding Window Maximum).

### 7.3 Min Stack — O(1) `get_min`
Store with every value the **minimum so far** (as a pair). **Problem:** 4.

### 7.4 Expressions: Infix, Postfix, Prefix
| Form | Example |
|---|---|
| Infix (normal) | `(2 + 1) * 3` |
| Postfix / RPN | `2 1 + 3 *` |
| Prefix | `* + 2 1 3` |

**Evaluating postfix with a stack:** read tokens left→right; numbers are pushed; an operator pops **two** numbers (`b` first, then `a`), computes `a op b`, and pushes the result. **Problem:** 6. Infix with precedence (`*` and `/` before `+` and `-`) uses a stack of pending terms. **Problem:** 11.

### 7.5 Stack ↔ Queue conversions (classic interview questions)
- **Queue using two stacks:** `in_stack` for pushes, `out_stack` for pops; refill `out_stack` only when empty → amortized O(1). **Problem:** 2.
- **Stack using queue(s):** after each push, rotate the queue so the newest is at the front. **Problem:** 3.

---

## 8. Patterns & Recognition Table

| Clue in the problem | Technique |
|---|---|
| matching brackets / nested structure | stack |
| "undo", "backspace", "most recent first" | stack |
| "next greater / smaller", "how many days until warmer" | monotonic stack |
| "largest rectangle", "span", "visible buildings" | monotonic stack |
| evaluate / parse expression | stack (RPN, calculator) |
| remove adjacent duplicates / k digits / collisions | stack simulation |
| "get min in O(1)", design problems | augmented stack |
| level by level, shortest path in unweighted grid/graph | queue (BFS) |
| process in arrival order, streams | queue |
| max/min in every sliding window | monotonic deque |
| top-K, always take the best next | priority queue |

---

## 9. 15 Standard Interview Problems

---

### 🟢 Problem 1 — Valid Parentheses & Minimum Remove to Make Valid (Easy) · *Stack*
**Idea:** Push openers, match closers with the top. For "minimum remove", push the **indexes** of `(` and delete the unmatched ones (unmatched `)` immediately, leftover `(` at the end).

```python
def is_valid(s):
    pairs = {')': '(', ']': '[', '}': '{'}           # closing -> matching opening
    stack = []
    for ch in s:
        if ch in pairs:                              # closing bracket
            if not stack or stack[-1] != pairs[ch]:  # nothing to match or wrong type
                return False
            stack.pop()                              # matched pair removed
        else:
            stack.append(ch)                         # opening bracket waits for its partner
    return not stack                                 # leftover openers -> invalid

def min_remove_to_make_valid(s):
    chars = list(s)                                  # strings are immutable -> work on a list
    stack = []                                       # indexes of unmatched '('
    for i, ch in enumerate(chars):
        if ch == '(':
            stack.append(i)
        elif ch == ')':
            if stack:
                stack.pop()                          # matches the latest '('
            else:
                chars[i] = ''                        # unmatched ')' -> delete
    for i in stack:
        chars[i] = ''                                # unmatched '(' -> delete
    return ''.join(chars)

print(is_valid("()[]{}"), is_valid("(]"), is_valid("([{}])"))
print(min_remove_to_make_valid("lee(t(c)o)de)"))
print(min_remove_to_make_valid("a)b(c)d"))
print(repr(min_remove_to_make_valid("))((")))
```
**Output**
```
True False True
lee(t(c)o)de
ab(c)d
''
```
**Complexity:** O(n) time, O(n) space.

---

### 🟢 Problem 2 — Implement Queue using Two Stacks (Easy-Medium)
**Idea:** `inp` receives pushes. When we need the front, if `out` is empty move **everything** from `inp` to `out` (this reverses the order, giving FIFO). Each element moves at most once → **amortized O(1)**.

```python
class MyQueue:
    def __init__(self):
        self.inp = []                                # stack for incoming elements
        self.out = []                                # stack for outgoing elements (reversed order)

    def push(self, x):
        self.inp.append(x)                           # enqueue = push on the input stack

    def _shift(self):
        if not self.out:                             # only refill when out is empty (keeps order correct)
            while self.inp:
                self.out.append(self.inp.pop())      # moving reverses the order

    def pop(self):
        self._shift()
        return self.out.pop()                        # top of out = oldest element

    def peek(self):
        self._shift()
        return self.out[-1]

    def empty(self):
        return not self.inp and not self.out

q = MyQueue()
q.push(1)
q.push(2)
print(q.peek())                                      # front is 1
print(q.pop())                                       # 1 (FIFO)
q.push(3)
print(q.pop(), q.pop())                              # 2 then 3
print(q.empty())
```
**Output**
```
1
1
2 3
True
```
**Complexity:** amortized O(1) per operation.

---

### 🟢 Problem 3 — Implement Stack using Queue (Easy-Medium)
**Idea:** After pushing a new element at the back, rotate the other `n-1` elements behind it so the newest element sits at the **front**. Then `pop` is a plain `popleft`.

```python
from collections import deque

class MyStack:
    def __init__(self):
        self.q = deque()                             # single queue

    def push(self, x):
        self.q.append(x)                             # new element goes to the back
        for _ in range(len(self.q) - 1):             # rotate all older elements behind it
            self.q.append(self.q.popleft())          # now x is at the front = stack top

    def pop(self):
        return self.q.popleft()                      # front is the most recent push

    def top(self):
        return self.q[0]

    def empty(self):
        return not self.q

s = MyStack()
s.push(1); s.push(2); s.push(3)
print(s.top())                                       # 3 (last pushed)
print(s.pop(), s.pop())                              # 3 then 2 (LIFO)
print(s.top(), s.empty())
```
**Output**
```
3
3 2
1 False
```
**Complexity:** push O(n), pop/top O(1). (Alternative: make pop O(n) and push O(1).)

---

### 🟡 Problem 4 — Min Stack (Medium) · *Augmented stack*
**Idea:** With each pushed value also store the minimum of the stack **at that moment**. The top pair always knows the current minimum.

```python
class MinStack:
    def __init__(self):
        self.st = []                                 # stores pairs (value, min_so_far)

    def push(self, val):
        cur_min = min(val, self.st[-1][1]) if self.st else val   # min including the new value
        self.st.append((val, cur_min))

    def pop(self):
        self.st.pop()                                # min info disappears together with the value

    def top(self):
        return self.st[-1][0]

    def get_min(self):
        return self.st[-1][1]                        # O(1): stored with the top element

ms = MinStack()
ms.push(-2); ms.push(0); ms.push(-3)
print(ms.get_min())                                  # -3
ms.pop()
print(ms.top(), ms.get_min())                        # 0 and -2
```
**Output**
```
-3
0 -2
```
**Complexity:** all operations O(1), O(n) space.

---

### 🟡 Problem 5 — Backspace String Compare (Easy-Medium) · *Stack simulation*
**Idea:** `#` means backspace. Build each final string with a stack (push letters, pop on `#`) and compare.

```python
def backspace_compare(s, t):
    def build(x):
        st = []
        for ch in x:
            if ch == '#':                            # backspace deletes the previous character
                if st:
                    st.pop()
            else:
                st.append(ch)                        # normal character
        return st
    return build(s) == build(t)

print(backspace_compare("ab#c", "ad#c"))             # both become "ac"
print(backspace_compare("ab##", "c#d#"))             # both become ""
print(backspace_compare("a#c", "b"))
```
**Output**
```
True
True
False
```
**Complexity:** O(n) time and space. (Follow-up: two pointers scanning from the back give O(1) space.)

---

### 🟡 Problem 6 — Evaluate Reverse Polish Notation (Medium) · *Stack*
**Idea:** Numbers are pushed; each operator pops `b` then `a`, computes `a op b`, pushes the result. Division truncates toward zero (`int(a / b)`).

```python
def eval_rpn(tokens):
    st = []
    for t in tokens:
        if t in {'+', '-', '*', '/'}:
            b = st.pop()                             # second operand (popped first!)
            a = st.pop()                             # first operand
            if t == '+':
                st.append(a + b)
            elif t == '-':
                st.append(a - b)
            elif t == '*':
                st.append(a * b)
            else:
                st.append(int(a / b))                # truncate toward zero (not floor) as required
        else:
            st.append(int(t))                        # operand: convert and push
    return st[0]

print(eval_rpn(["2", "1", "+", "3", "*"]))           # (2 + 1) * 3
print(eval_rpn(["4", "13", "5", "/", "+"]))          # 4 + (13 / 5)
print(eval_rpn(["10", "6", "9", "3", "+", "-11", "*", "/", "*", "17", "+", "5", "+"]))
```
**Output**
```
9
6
22
```
**Complexity:** O(n) time, O(n) space.

---

### 🟡 Problem 7 — Daily Temperatures (Medium) · *Monotonic stack*
**Idea:** For each day, find how many days until a **warmer** day. Keep a stack of indexes with decreasing temperatures. A warmer day pops colder days and fills `i - j`.

```python
def daily_temperatures(temps):
    res = [0] * len(temps)                           # 0 = no warmer day ahead
    st = []                                          # indexes of days still waiting for a warmer day
    for i, t in enumerate(temps):
        while st and temps[st[-1]] < t:              # today is warmer than the waiting day
            j = st.pop()
            res[j] = i - j                           # number of days waited
        st.append(i)                                 # today now waits for its own warmer day
    return res

print(daily_temperatures([73, 74, 75, 71, 69, 72, 76, 73]))
print(daily_temperatures([30, 40, 50, 60]))
print(daily_temperatures([30, 60, 90]))
```
**Output**
```
[1, 1, 4, 2, 1, 1, 0, 0]
[1, 1, 1, 0]
[1, 1, 0]
```
**Complexity:** O(n) time, O(n) space (brute force O(n²)).

---

### 🟡 Problem 8 — Next Greater Element II (Circular Array) (Medium) · *Monotonic stack, two passes*
**Idea:** The array is circular, so simulate going around twice (`2n` iterations, index `i % n`). Push indexes only during the first pass.

```python
def next_greater_circular(nums):
    n = len(nums)
    res = [-1] * n                                   # default: no greater element
    st = []                                          # indexes waiting for their next greater element
    for i in range(2 * n):                           # go around the circle twice
        x = nums[i % n]                              # circular access
        while st and nums[st[-1]] < x:               # x is greater than the waiting element
            res[st.pop()] = x
        if i < n:                                    # only first pass pushes (each index once)
            st.append(i)
    return res

print(next_greater_circular([1, 2, 1]))
print(next_greater_circular([1, 2, 3, 4, 3]))
```
**Output**
```
[2, -1, 2]
[2, 3, 4, -1, 4]
```
**Complexity:** O(n) time, O(n) space.

---

### 🟡 Problem 9 — Asteroid Collision (Medium) · *Stack simulation*
**Idea:** Positive = moving right, negative = moving left. A collision happens only when the **top is positive** and the **new one is negative**. Bigger one survives, equal sizes destroy both.

```python
def asteroid_collision(asts):
    st = []
    for a in asts:
        alive = True                                 # does the current asteroid survive?
        while alive and a < 0 and st and st[-1] > 0: # moving left meets one moving right
            if st[-1] < -a:                          # top is smaller -> top explodes, keep checking
                st.pop()
            elif st[-1] == -a:                       # same size -> both explode
                st.pop()
                alive = False
            else:                                    # top is bigger -> current explodes
                alive = False
        if alive:
            st.append(a)
    return st

print(asteroid_collision([5, 10, -5]))
print(asteroid_collision([8, -8]))
print(asteroid_collision([10, 2, -5]))
print(asteroid_collision([-2, -1, 1, 2]))            # never meet: all survive
```
**Output**
```
[5, 10]
[]
[10]
[-2, -1, 1, 2]
```
**Complexity:** O(n) time, O(n) space.

---

### 🟡 Problem 10 — Remove K Digits (Medium) · *Greedy + monotonic stack*
**Idea:** To get the smallest number, remove a digit whenever it is **bigger than the next digit** (a "peak"). Keep a stack with non-decreasing digits; pop while the top is larger than the new digit and removals remain.

```python
def remove_k_digits(num, k):
    st = []                                          # kept digits, non-decreasing
    for d in num:
        while k and st and st[-1] > d:               # previous digit is larger -> removing it makes number smaller
            st.pop()
            k -= 1
        st.append(d)
    if k:                                            # still need removals -> drop from the end (largest digits)
        st = st[:-k]
    return ''.join(st).lstrip('0') or '0'            # remove leading zeros; empty means 0

print(remove_k_digits("1432219", 3))
print(remove_k_digits("10200", 1))
print(remove_k_digits("10", 2))
```
**Output**
```
1219
200
0
```
**Complexity:** O(n) time, O(n) space.

---

### 🔴 Problem 11 — Basic Calculator II (Medium-Hard) · *Stack of terms*
**Idea:** Evaluate `+ - * /` without brackets. Keep a stack of terms: `+n` pushes `n`, `-n` pushes `-n`, `*` and `/` combine **immediately** with the top (higher precedence). Final answer = sum of the stack. We apply the **previous operator** when we meet the next operator (or the end).

```python
def calculate(s):
    st = []                                          # signed terms to be summed at the end
    num = 0                                          # number currently being read
    op = '+'                                         # operator that came BEFORE num
    for i, ch in enumerate(s):
        if ch.isdigit():
            num = num * 10 + int(ch)                 # build multi-digit numbers
        if (not ch.isdigit() and ch != ' ') or i == len(s) - 1:   # operator found, or end of string
            if op == '+':
                st.append(num)
            elif op == '-':
                st.append(-num)
            elif op == '*':
                st.append(st.pop() * num)            # multiply with the previous term right away
            elif op == '/':
                st.append(int(st.pop() / num))       # integer division truncating toward zero
            op = ch                                  # this operator applies to the NEXT number
            num = 0
    return sum(st)

print(calculate("3+2*2"))
print(calculate(" 3/2 "))
print(calculate(" 3+5 / 2 "))
print(calculate("14-3/2"))
```
**Output**
```
7
1
5
13
```
**Complexity:** O(n) time, O(n) space.

---

### 🔴 Problem 12 — First Non-Repeating Character in a Stream (Medium) · *Queue + counts*
**Idea:** Characters arrive one by one; after each arrival report the first character that has appeared only once (or `#`). Keep a **queue** of candidates in arrival order and a **count** map. Pop from the front while the front has become repeated.

```python
from collections import deque

def first_non_repeating(stream):
    q = deque()                                      # candidates in arrival order
    count = {}                                       # character -> times seen
    res = []
    for ch in stream:
        count[ch] = count.get(ch, 0) + 1             # update frequency
        if count[ch] == 1:                           # first time seen -> becomes a candidate
            q.append(ch)
        while q and count[q[0]] > 1:                 # front is no longer unique -> discard
            q.popleft()
        res.append(q[0] if q else '#')               # front of queue is the answer ('#' if none)
    return ''.join(res)

print(first_non_repeating("aabc"))
print(first_non_repeating("abadbc"))
```
**Output**
```
a#bb
aabbdd
```
**Complexity:** O(n) time, O(alphabet) space.

---

### 🔴 Problem 13 — Rotting Oranges (Medium-Hard) · *BFS with a queue (multi-source)*
**Idea:** All rotten oranges spread rot **simultaneously** each minute → **BFS by levels** starting from *all* rotten cells at once. Each BFS level = 1 minute. If fresh oranges remain at the end → `-1`.

```python
from collections import deque

def oranges_rotting(grid):
    rows, cols = len(grid), len(grid[0])
    q = deque()                                      # BFS queue of rotten cells
    fresh = 0                                        # number of fresh oranges
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 2:
                q.append((r, c))                     # all rotten oranges are sources at time 0
            elif grid[r][c] == 1:
                fresh += 1
    minutes = 0
    while q and fresh:                               # continue while rot can still spread
        for _ in range(len(q)):                      # process exactly one level = one minute
            r, c = q.popleft()
            for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):   # 4 neighbours
                nr, nc = r + dr, c + dc
                if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == 1:
                    grid[nr][nc] = 2                 # fresh orange rots (also marks it visited)
                    fresh -= 1
                    q.append((nr, nc))               # it will spread rot in the next minute
        minutes += 1
    return minutes if fresh == 0 else -1             # unreachable fresh oranges -> -1

print(oranges_rotting([[2, 1, 1], [1, 1, 0], [0, 1, 1]]))
print(oranges_rotting([[2, 1, 1], [0, 1, 1], [1, 0, 1]]))
print(oranges_rotting([[0, 2]]))
```
**Output**
```
4
-1
0
```
**Complexity:** O(rows × cols) time and space.

---

### 🔴 Problem 14 — Sliding Window Maximum (Hard) · *Monotonic deque*
**Idea:** Keep a deque of **indexes** whose values are **decreasing**. New element removes all smaller elements from the back (they can never be a maximum again). Remove the front if it left the window. The front is the window maximum.

```python
from collections import deque

def max_sliding_window(nums, k):
    dq = deque()                                     # indexes; values decreasing from front to back
    res = []
    for i, x in enumerate(nums):
        while dq and nums[dq[-1]] <= x:              # smaller elements behind x are useless now
            dq.pop()
        dq.append(i)
        if dq[0] <= i - k:                           # front index is outside the window [i-k+1, i]
            dq.popleft()
        if i >= k - 1:                               # window is fully formed
            res.append(nums[dq[0]])                  # front = maximum of the window
    return res

print(max_sliding_window([1, 3, -1, -3, 5, 3, 6, 7], 3))
print(max_sliding_window([1], 1))
print(max_sliding_window([9, 11], 2))
```
**Output**
```
[3, 3, 5, 5, 6, 7]
[1]
[11]
```
**Complexity:** O(n) time (each index enters and leaves once), O(k) space. Brute force: O(n·k).

---

### 🔴 Problem 15 — Largest Rectangle in Histogram (Hard) · *Monotonic stack*
**Idea:** For each bar, the widest rectangle using it as the **shortest** bar extends left and right until a **shorter** bar. Keep a stack of indexes with **increasing** heights. When a shorter bar arrives, pop taller bars and compute their rectangles: height = popped bar, width = distance between the new top of the stack and the current index. A sentinel `0` at the end flushes the stack.

```python
def largest_rectangle(heights):
    st = []                                          # indexes with increasing bar heights
    best = 0
    heights = heights + [0]                          # sentinel bar of height 0 forces all pops at the end
    for i, h in enumerate(heights):
        while st and heights[st[-1]] > h:            # current bar is shorter -> popped bar's rectangle ends here
            height = heights[st.pop()]               # height of the rectangle
            width = i if not st else i - st[-1] - 1  # left boundary = new stack top (or the array start)
            best = max(best, height * width)
        st.append(i)
    return best

print(largest_rectangle([2, 1, 5, 6, 2, 3]))         # bars 5 and 6 -> 5 * 2 = 10
print(largest_rectangle([2, 4]))
print(largest_rectangle([1, 1, 1, 1]))
```
**Output**
```
10
4
4
```
**Complexity:** O(n) time, O(n) space. Related: Maximal Rectangle (85), Trapping Rain Water (42).

---

## 10. Cheat Sheet, Mistakes & Practice List

### Problem → technique map
| # | Problem | Technique |
|---|---|---|
| 1 | Valid Parentheses / Min Remove | stack (indexes) |
| 2 | Queue using Stacks | two stacks, amortized O(1) |
| 3 | Stack using Queue | queue rotation |
| 4 | Min Stack | augmented stack |
| 5 | Backspace String Compare | stack simulation |
| 6 | Evaluate RPN | stack evaluation |
| 7 | Daily Temperatures | monotonic stack |
| 8 | Next Greater Element II | monotonic stack, 2 passes |
| 9 | Asteroid Collision | stack simulation |
| 10 | Remove K Digits | greedy + monotonic stack |
| 11 | Basic Calculator II | stack of terms |
| 12 | First Non-Repeating in Stream | queue + count map |
| 13 | Rotting Oranges | BFS queue |
| 14 | Sliding Window Maximum | monotonic deque |
| 15 | Largest Rectangle in Histogram | monotonic stack |

### Operation complexity (Python)
| Structure | Insert | Remove | Peek |
|---|---|---|---|
| Stack (`list`) | O(1) `append` | O(1) `pop()` | O(1) `[-1]` |
| Queue (`deque`) | O(1) `append` | O(1) `popleft()` | O(1) `[0]` |
| Queue (`list`) ❌ | O(1) | **O(n)** `pop(0)` | O(1) |
| Priority queue (`heapq`) | O(log n) | O(log n) | O(1) |

### Common mistakes
1. **Popping from an empty stack/queue** → always check `if st` first.
2. Using `list.pop(0)` as a queue (O(n)).
3. Pushing **values** when you need **indexes** (you lose position info for distances/widths).
4. Wrong operand order in RPN: pop `b` first, then `a`, compute `a op b`.
5. In Python, `-7 // 2 = -4` (floor); problems asking for truncation toward zero need `int(a / b)`.
6. Forgetting **leftover elements** in the stack after the loop (histogram sentinel, unmatched brackets).
7. In BFS, forgetting to mark visited **when pushing** (causes duplicates / infinite loops).
8. In monotonic stacks, mixing up strict (`<`) vs non-strict (`<=`) comparisons with duplicates.

### How to decide stack vs queue in an interview
- Need the **most recent** unresolved item? → **Stack**.
- Need to process in **arrival order / by levels**? → **Queue (BFS)**.
- Need the **best** next item by priority? → **Heap**.
- Need max/min of a **moving window**? → **Monotonic deque**.


