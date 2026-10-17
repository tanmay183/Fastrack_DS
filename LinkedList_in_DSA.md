# 🔗 Linked List in DSA (Python) — From Construction to Advanced Interview Problems

> Easy-language guide: what a linked list is → how to build one (singly, doubly, circular) → all core operations → patterns (dummy node, fast & slow pointers, reversal, merge) → **15 standard interview problems** with commented code and verified output.

---

## 📑 Table of Contents
1. [What is a Linked List?](#1-what-is-a-linked-list)
2. [Array vs Linked List](#2-array-vs-linked-list)
3. [Types of Linked Lists](#3-types-of-linked-lists)
4. [Node + Helper Functions (used in all problems)](#4-node--helper-functions)
5. [Singly Linked List — Construction & Operations](#5-singly-linked-list--construction--operations)
6. [Doubly Linked List](#6-doubly-linked-list)
7. [Circular Linked List](#7-circular-linked-list)
8. [Linked List Patterns (Techniques)](#8-linked-list-patterns-techniques)
9. [15 Standard Interview Problems](#9-15-standard-interview-problems)
10. [Cheat Sheet, Mistakes & Practice List](#10-cheat-sheet-mistakes--practice-list)

---

## 1. What is a Linked List?

A **linked list** is a chain of **nodes**. Each node stores:
1. **data** (`val`)
2. a **pointer/reference** to the **next** node (`next`)

The nodes are **not stored next to each other in memory**; they are connected only by pointers. The first node is the **head**, the last node points to `None`.

```
head
 │
 ▼
[10 | •]──▶[20 | •]──▶[30 | •]──▶[40 | None]
 node 1     node 2     node 3     node 4
```

**Key facts**
- You can only reach a node by **walking from the head** (no direct indexing).
- Insert/delete is cheap **if you already have the pointer** (just re-link, no shifting).
- It grows and shrinks one node at a time (no resizing).

---

## 2. Array vs Linked List

| Feature | Array / Python list | Linked List |
|---|---|---|
| Memory layout | contiguous | scattered, connected by pointers |
| Access `a[i]` | **O(1)** | **O(n)** (walk i steps) |
| Search | O(n) (O(log n) if sorted) | O(n) |
| Insert/delete at **front** | O(n) (shift) | **O(1)** |
| Insert/delete at **end** | O(1) amortized | O(1) with tail pointer (else O(n)) |
| Insert/delete in **middle** (node known) | O(n) | **O(1)** |
| Extra memory | none | 1 pointer per node |
| Cache friendly | ✔ | ✘ |
| Size | resize by copying | grows node by node |

> 💡 Interview reminder: linked lists are mostly about **pointer manipulation**; draw boxes and arrows before coding.

---

## 3. Types of Linked Lists

| Type | Node | Shape |
|---|---|---|
| **Singly** | `val, next` | `1 → 2 → 3 → None` |
| **Doubly** | `val, prev, next` | `None ⇄ 1 ⇄ 2 ⇄ 3 ⇄ None` (can walk both ways, O(1) delete given the node) |
| **Circular (singly)** | `val, next` | last node's `next` = head |
| **Circular doubly** | `val, prev, next` | both ends linked |
| **Skip list / multi-level** | extra pointers | fast search (advanced) |

---

## 4. Node + Helper Functions

All problems below use these three helpers (paste them above any problem to run it).

```python
# HELPERS ------------------------------------------------------------
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val                             # data stored in the node
        self.next = next                           # reference to the next node (None at the end)

def build_list(arr):                               # [1,2,3] -> 1 -> 2 -> 3
    dummy = ListNode()                             # dummy node avoids special-casing the first node
    cur = dummy
    for x in arr:
        cur.next = ListNode(x)                     # attach a new node after the current last node
        cur = cur.next                             # move to it
    return dummy.next                              # real head is after the dummy

def to_list(head):                                 # 1 -> 2 -> 3 -> [1,2,3] (for printing)
    res = []
    while head:                                    # walk until the end
        res.append(head.val)
        head = head.next
    return res

print(to_list(build_list([1, 2, 3])))
print(to_list(build_list([])))                     # empty list -> head is None
```
**Output**
```
[1, 2, 3]
[]
```

---

## 5. Singly Linked List — Construction & Operations

### 5.1 Full class: insert, delete, search, reverse

```python
class Node:
    def __init__(self, val, next=None):
        self.val = val
        self.next = next

class SinglyLinkedList:
    def __init__(self):
        self.head = None                           # empty list
        self.size = 0                              # number of nodes

    def insert_head(self, val):                    # O(1)
        self.head = Node(val, self.head)           # new node points to the old head, becomes the head
        self.size += 1

    def insert_tail(self, val):                    # O(n) because we must walk to the end
        new = Node(val)
        if not self.head:                          # empty list: new node is the head
            self.head = new
        else:
            cur = self.head
            while cur.next:                        # stop on the LAST node
                cur = cur.next
            cur.next = new                         # link last node to the new node
        self.size += 1

    def insert_at(self, index, val):               # O(n)
        if index < 0 or index > self.size:
            raise IndexError("index out of range")
        if index == 0:                             # inserting at the front
            self.insert_head(val)
            return
        cur = self.head
        for _ in range(index - 1):                 # stop at the node BEFORE the insert position
            cur = cur.next
        cur.next = Node(val, cur.next)             # new node points to the old next; previous points to new
        self.size += 1

    def delete_value(self, val):                   # O(n): delete first node with this value
        if not self.head:
            return False
        if self.head.val == val:                   # deleting the head
            self.head = self.head.next
            self.size -= 1
            return True
        cur = self.head
        while cur.next and cur.next.val != val:    # find the node BEFORE the one to delete
            cur = cur.next
        if cur.next:
            cur.next = cur.next.next               # skip the target node (it gets garbage collected)
            self.size -= 1
            return True
        return False                               # value not found

    def search(self, val):                         # O(n): index of first match or -1
        cur, i = self.head, 0
        while cur:
            if cur.val == val:
                return i
            cur = cur.next
            i += 1
        return -1

    def reverse(self):                             # O(n), in place
        prev, cur = None, self.head
        while cur:
            nxt = cur.next                         # 1. remember the next node
            cur.next = prev                        # 2. reverse the arrow
            prev = cur                             # 3. move prev forward
            cur = nxt                              # 4. move cur forward
        self.head = prev                           # prev is the new head

    def __str__(self):
        parts, cur = [], self.head
        while cur:
            parts.append(str(cur.val))
            cur = cur.next
        return " -> ".join(parts) + " -> None"

ll = SinglyLinkedList()
ll.insert_head(10)                                 # 10
ll.insert_tail(20)                                 # 10 20
ll.insert_tail(30)                                 # 10 20 30
ll.insert_at(1, 15)                                # 10 15 20 30
print(ll)
ll.insert_head(5)
print(ll, "size =", ll.size)
print(ll.delete_value(20), ll.delete_value(99))    # delete existing / missing value
print(ll)
print(ll.search(30), ll.search(99))                # index of 30, and -1 for missing
ll.reverse()
print(ll)
```
**Output**
```
10 -> 15 -> 20 -> 30 -> None
5 -> 10 -> 15 -> 20 -> 30 -> None size = 5
True False
5 -> 10 -> 15 -> 30 -> None
3 -1
30 -> 15 -> 10 -> 5 -> None
```

### 5.2 Complexity of singly linked list operations
| Operation | Time |
|---|---|
| Insert at head | O(1) |
| Insert at tail | O(n) (O(1) if you keep a `tail` pointer) |
| Insert after a given node | O(1) |
| Delete head | O(1) |
| Delete a given value | O(n) |
| Search / access by index | O(n) |
| Reverse | O(n) time, O(1) space |
| Length | O(n) (or O(1) if you store `size`) |

### 5.3 Pointer-reversal trace (the most important skill)
Reverse `1 → 2 → 3`:
| Step | prev | cur | action | list so far |
|---|---|---|---|---|
| start | None | 1 | – | 1→2→3 |
| 1 | 1 | 2 | `1.next = None` | 1→None, 2→3 |
| 2 | 2 | 3 | `2.next = 1` | 2→1→None, 3 |
| 3 | 3 | None | `3.next = 2` | 3→2→1→None |
Return `prev` = node 3.

---

## 6. Doubly Linked List

Each node has `prev` **and** `next`. Benefits: walk backwards, and delete a node in **O(1)** when you have the node itself (no need to find its predecessor).

```python
class DNode:
    def __init__(self, val):
        self.val = val
        self.prev = None                           # link to the previous node
        self.next = None                           # link to the next node

class DoublyLinkedList:
    def __init__(self):
        self.head = None
        self.tail = None

    def insert_head(self, val):
        node = DNode(val)
        if not self.head:                          # empty list: node is both head and tail
            self.head = self.tail = node
        else:
            node.next = self.head                  # new node -> old head
            self.head.prev = node                  # old head <- new node
            self.head = node
        return node

    def insert_tail(self, val):
        node = DNode(val)
        if not self.tail:
            self.head = self.tail = node
        else:
            node.prev = self.tail                  # old tail <- new node
            self.tail.next = node                  # old tail -> new node
            self.tail = node
        return node

    def delete_node(self, node):                   # O(1): we already hold the node
        if node.prev:
            node.prev.next = node.next             # predecessor skips the node
        else:
            self.head = node.next                  # node was the head
        if node.next:
            node.next.prev = node.prev             # successor points back to the predecessor
        else:
            self.tail = node.prev                  # node was the tail

    def forward(self):
        out, cur = [], self.head
        while cur:
            out.append(cur.val)
            cur = cur.next
        return out

    def backward(self):
        out, cur = [], self.tail
        while cur:
            out.append(cur.val)
            cur = cur.prev                         # walk using prev pointers
        return out

d = DoublyLinkedList()
d.insert_tail(2)
d.insert_tail(3)
d.insert_head(1)
mid = d.insert_tail(4)
print(d.forward())
print(d.backward())
d.delete_node(mid)                                 # delete the last node in O(1)
print(d.forward(), d.tail.val)
```
**Output**
```
[1, 2, 3, 4]
[4, 3, 2, 1]
[1, 2, 3] 3
```
**Used in:** LRU cache, browser history, undo/redo, text editors.

---

## 7. Circular Linked List

The last node points back to the **head** instead of `None`. Used for round-robin scheduling and repeated cycling (e.g. Josephus problem). **Danger:** a plain `while cur:` loop never ends, so stop when you return to the start.

```python
class Node:
    def __init__(self, val):
        self.val = val
        self.next = None

a, b, c, d = Node(1), Node(2), Node(3), Node(4)
a.next, b.next, c.next, d.next = b, c, d, a        # last node points back to the head -> circle
cur, out = a, []
for _ in range(6):                                 # walk more steps than the list has nodes
    out.append(cur.val)
    cur = cur.next                                 # never hits None, it just wraps around
print(out)
cur, count = a.next, 1                             # safe traversal: stop when we come back to the start
while cur is not a:
    count += 1
    cur = cur.next
print("length =", count)
```
**Output**
```
[1, 2, 3, 4, 1, 2]
length = 4
```

---

## 8. Linked List Patterns (Techniques)

### Pattern 1 — Dummy Node (Sentinel)
Create `dummy = ListNode(0, head)` and build/modify the list after it. This removes special cases for "the head changes" (deleting the head, merging, adding numbers). Return `dummy.next`.
**Problems:** 3, 6, 9, 12, 14.

### Pattern 2 — Fast & Slow Pointers (Tortoise and Hare)
`slow` moves 1 step, `fast` moves 2 steps.
- When `fast` reaches the end, `slow` is at the **middle**. (Problems 2, 8, 10, 11)
- If there is a **cycle**, `fast` eventually meets `slow`. (Problem 5)

**Why they meet:** inside a loop `fast` gains 1 node per step on `slow`, so the gap shrinks by 1 each step until it reaches 0.

**Finding the cycle start (Floyd):** after they meet, put one pointer back at `head`; move both **1 step at a time**; they meet at the cycle's first node. (Distance from head to start = distance from meeting point to start, going around the loop.)

### Pattern 3 — In-place Reversal
Three pointers `prev, cur, nxt`. Reverse whole list, a sublist, or groups of k.
**Problems:** 1, 8, 10, 12.

### Pattern 4 — Two Pointers with a Gap
Move `first` n steps ahead, then move both; when `first` hits the end, `second` is n nodes from the end. **Problem:** 6.

### Pattern 5 — Merge (two-pointer on two lists)
Compare heads and append the smaller. Foundation of merge sort on lists and merging K lists.
**Problems:** 3, 11, 14.

### Pattern 6 — Hash Map / Hash Set
Store nodes you've seen (cycle detection with O(n) memory) or map original node → copy node (copy list with random pointer). **Problem:** 13.

### Pattern 7 — Pointer Switching (length equalising trick)
Two pointers traverse `A+B` and `B+A`, so they meet at the intersection after the same number of steps. **Problem:** 7.

### Pattern 8 — Combine Doubly Linked List + Hash Map
O(1) lookup *and* O(1) reorder → LRU Cache. **Problem:** 15.

### Recognition table
| Clue | Technique |
|---|---|
| "reverse" | 3-pointer reversal |
| "middle", "palindrome", "reorder" | fast & slow + reverse half |
| "cycle", "loop" | Floyd fast & slow |
| "nth from the end" | gap pointers + dummy |
| "merge", "sorted lists" | dummy + two pointers / heap |
| "sort a linked list" | merge sort (O(1) extra space) |
| "add numbers digit by digit" | dummy + carry |
| "intersection of two lists" | pointer switching |
| "copy / clone with extra pointers" | hash map old→new |
| "O(1) cache with eviction" | DLL + hash map |

---

## 9. 15 Standard Interview Problems

> Problems use `ListNode`, `build_list`, `to_list` from **Section 4**.

---

### 🟢 Problem 1 — Reverse a Linked List (Easy) · *In-place reversal*
**Idea:** For every node flip its `next` arrow to point backward. Use `prev`, `cur`, `nxt`. A recursive version reverses the rest first, then links the head at the end.

```python
# uses ListNode, build_list, to_list from Section 4
def reverse_list(head):
    prev, cur = None, head                         # prev = already-reversed part
    while cur:
        nxt = cur.next                             # save the rest of the list
        cur.next = prev                            # flip the arrow
        prev = cur                                 # move prev one step forward
        cur = nxt                                  # move cur one step forward
    return prev                                    # prev is the new head

def reverse_recursive(head):
    if not head or not head.next:                  # base case: empty or single node
        return head
    new_head = reverse_recursive(head.next)        # reverse everything after head; new_head = old tail
    head.next.next = head                          # make the next node point back to head
    head.next = None                               # head becomes the tail
    return new_head

print(to_list(reverse_list(build_list([1, 2, 3, 4, 5]))))
print(to_list(reverse_recursive(build_list([1, 2, 3]))))
print(to_list(reverse_list(build_list([]))))
```
**Output**
```
[5, 4, 3, 2, 1]
[3, 2, 1]
[]
```
**Complexity:** O(n) time; O(1) space (iterative), O(n) stack (recursive).

---

### 🟢 Problem 2 — Middle of the Linked List (Easy) · *Fast & slow pointers*
**Idea:** `fast` moves two steps for every one step of `slow`. When `fast` reaches the end, `slow` is at the middle. For an even length, return the **second** middle.

```python
# uses ListNode, build_list, to_list from Section 4
def middle_node(head):
    slow = fast = head                             # both start at the head
    while fast and fast.next:                      # fast can still jump two nodes
        slow = slow.next                           # 1 step
        fast = fast.next.next                      # 2 steps
    return slow                                    # slow is at the middle

print(to_list(middle_node(build_list([1, 2, 3, 4, 5]))))      # middle is 3
print(to_list(middle_node(build_list([1, 2, 3, 4, 5, 6]))))   # two middles (3,4): returns the second
```
**Output**
```
[3, 4, 5]
[4, 5, 6]
```
**Complexity:** O(n) time, O(1) space (one pass, no length counting).

---

### 🟢 Problem 3 — Merge Two Sorted Lists (Easy) · *Dummy node + two pointers*
**Idea:** Keep a `tail` of the merged list. Compare the heads of both lists, attach the smaller, advance. When one list ends, attach the rest of the other.

```python
# uses ListNode, build_list, to_list from Section 4
def merge_two_lists(l1, l2):
    dummy = ListNode()                             # dummy head simplifies the first insertion
    tail = dummy                                   # last node of the merged list
    while l1 and l2:
        if l1.val <= l2.val:                       # l1's node is smaller (<= keeps it stable)
            tail.next = l1
            l1 = l1.next
        else:
            tail.next = l2
            l2 = l2.next
        tail = tail.next                           # advance the merged tail
    tail.next = l1 or l2                           # append whichever list still has nodes
    return dummy.next

print(to_list(merge_two_lists(build_list([1, 2, 4]), build_list([1, 3, 4]))))
print(to_list(merge_two_lists(build_list([]), build_list([]))))
print(to_list(merge_two_lists(build_list([]), build_list([0]))))
```
**Output**
```
[1, 1, 2, 3, 4, 4]
[]
[0]
```
**Complexity:** O(m + n) time, O(1) extra space (nodes are re-linked, not copied).

---

### 🟢 Problem 4 — Remove Duplicates from Sorted List (Easy) · *Single pass*
**Idea:** Duplicates are adjacent. If `cur.val == cur.next.val` skip the next node; otherwise advance.

```python
# uses ListNode, build_list, to_list from Section 4
def delete_duplicates(head):
    cur = head
    while cur and cur.next:
        if cur.val == cur.next.val:                # next node is a duplicate
            cur.next = cur.next.next               # unlink it (do not advance: there may be more)
        else:
            cur = cur.next                         # no duplicate -> move on
    return head

print(to_list(delete_duplicates(build_list([1, 1, 2, 3, 3]))))
print(to_list(delete_duplicates(build_list([1, 1, 2, 2, 2]))))
```
**Output**
```
[1, 2, 3]
[1, 2]
```
**Complexity:** O(n) time, O(1) space.

---

### 🟡 Problem 5 — Linked List Cycle & Cycle Start (Medium) · *Floyd's algorithm*
**Idea:** Phase 1: slow/fast pointers meet inside a cycle (if any). Phase 2: reset `slow` to `head`, move both one step at a time; they meet at the **first node of the cycle**.

```python
# uses ListNode, build_list, to_list from Section 4
def make_cycle(head, pos):                         # helper for testing: tail.next = node at index pos
    if pos < 0:
        return head
    tail, target, i, cur = head, None, 0, head
    while cur:
        if i == pos:
            target = cur                           # node where the cycle will start
        tail = cur
        cur = cur.next
        i += 1
    tail.next = target                             # close the loop
    return head

def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next                           # tortoise
        fast = fast.next.next                      # hare
        if slow is fast:                           # they met -> cycle exists
            return True
    return False                                   # fast fell off the end -> no cycle

def detect_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:                           # meeting point inside the cycle
            slow = head                            # restart one pointer from the head
            while slow is not fast:                # both move 1 step now
                slow = slow.next
                fast = fast.next
            return slow                            # first node of the cycle
    return None

print(has_cycle(make_cycle(build_list([3, 2, 0, -4]), 1)))
print(detect_cycle(make_cycle(build_list([3, 2, 0, -4]), 1)).val)   # cycle begins at value 2
print(detect_cycle(make_cycle(build_list([1, 2]), 0)).val)
print(has_cycle(build_list([1, 2, 3])))
print(detect_cycle(build_list([1])))
```
**Output**
```
True
2
1
False
None
```
**Complexity:** O(n) time, **O(1) space** (a hash set would use O(n)).

---

### 🟡 Problem 6 — Remove N-th Node From End (Medium) · *Gap pointers + dummy*
**Idea:** Move `first` n+1 steps ahead of `second` (starting both at the dummy). Then move both until `first` falls off the end. `second` now stands **just before** the node to delete.

```python
# uses ListNode, build_list, to_list from Section 4
def remove_nth_from_end(head, n):
    dummy = ListNode(0, head)                      # handles deleting the head itself
    first = second = dummy
    for _ in range(n + 1):                         # create a gap of n nodes between the pointers
        first = first.next
    while first:                                   # move both until first passes the last node
        first = first.next
        second = second.next
    second.next = second.next.next                 # unlink the target node
    return dummy.next

print(to_list(remove_nth_from_end(build_list([1, 2, 3, 4, 5]), 2)))   # removes 4
print(to_list(remove_nth_from_end(build_list([1]), 1)))               # removes the only node
print(to_list(remove_nth_from_end(build_list([1, 2]), 1)))
```
**Output**
```
[1, 2, 3, 5]
[]
[1]
```
**Complexity:** O(n) time, one pass, O(1) space.

---

### 🟡 Problem 7 — Intersection of Two Linked Lists (Medium) · *Pointer switching*
**Idea:** Two lists may **merge** into a shared tail (same node objects, not just equal values). Walk `pA` over `A` then `B`; walk `pB` over `B` then `A`. Both walk `lenA + lenB` nodes in total, so they meet at the intersection (or both become `None`).

```python
# uses ListNode, build_list, to_list from Section 4
def get_intersection(headA, headB):
    pA, pB = headA, headB
    while pA is not pB:                            # compare node identity, not values
        pA = pA.next if pA else headB              # after finishing A, continue on B
        pB = pB.next if pB else headA              # after finishing B, continue on A
    return pA                                      # intersection node or None

shared = build_list([8, 4, 5])                     # common tail
a = build_list([4, 1])
b = build_list([5, 6, 1])
a.next.next = shared                               # A: 4 -> 1 -> 8 -> 4 -> 5
b.next.next.next = shared                          # B: 5 -> 6 -> 1 -> 8 -> 4 -> 5
node = get_intersection(a, b)
print(node.val, node is shared)                    # they meet at the node with value 8
print(get_intersection(build_list([1, 2]), build_list([3])))   # no intersection
```
**Output**
```
8 True
None
```
**Complexity:** O(m + n) time, O(1) space.

---

### 🟡 Problem 8 — Palindrome Linked List (Medium) · *Middle + reverse half*
**Idea:** Find the middle (fast/slow), reverse the **second half**, compare it with the first half node by node.

```python
# uses ListNode, build_list, to_list from Section 4
def is_palindrome(head):
    slow = fast = head
    while fast and fast.next:                      # step 1: find the middle
        slow = slow.next
        fast = fast.next.next
    prev = None
    while slow:                                    # step 2: reverse the second half starting at slow
        nxt = slow.next
        slow.next = prev
        prev = slow
        slow = nxt
    left, right = head, prev                       # step 3: compare first half with reversed second half
    while right:                                   # second half is the shorter (or equal) one
        if left.val != right.val:
            return False
        left = left.next
        right = right.next
    return True

print(is_palindrome(build_list([1, 2, 2, 1])))
print(is_palindrome(build_list([1, 2])))
print(is_palindrome(build_list([1, 2, 3, 2, 1])))
```
**Output**
```
True
False
True
```
**Complexity:** O(n) time, O(1) space. (Converting to an array is O(n) space; mention the restore-the-list follow-up.)

---

### 🟡 Problem 9 — Add Two Numbers (Medium) · *Dummy + carry*
**Idea:** Digits are stored in **reverse order** (units digit first) so we can add from the head like column addition. Keep a `carry`.

```python
# uses ListNode, build_list, to_list from Section 4
def add_two_numbers(l1, l2):
    dummy = ListNode()
    cur = dummy
    carry = 0
    while l1 or l2 or carry:                       # continue while any digit or carry remains
        total = carry                              # start with the carry from the previous column
        if l1:
            total += l1.val
            l1 = l1.next
        if l2:
            total += l2.val
            l2 = l2.next
        carry, digit = divmod(total, 10)           # new carry and the digit to store
        cur.next = ListNode(digit)
        cur = cur.next
    return dummy.next

print(to_list(add_two_numbers(build_list([2, 4, 3]), build_list([5, 6, 4]))))      # 342 + 465 = 807
print(to_list(add_two_numbers(build_list([0]), build_list([0]))))
print(to_list(add_two_numbers(build_list([9, 9, 9, 9, 9, 9, 9]), build_list([9, 9, 9, 9]))))
```
**Output**
```
[7, 0, 8]
[0]
[8, 9, 9, 9, 0, 0, 0, 1]
```
**Complexity:** O(max(m, n)) time and space.

---

### 🟡 Problem 10 — Reorder List (Medium) · *Middle + reverse + interleave*
**Idea:** Turn `L0→L1→…→Ln` into `L0→Ln→L1→Ln-1→…`. (1) find the middle, (2) reverse the second half, (3) merge the two halves alternately.

```python
# uses ListNode, build_list, to_list from Section 4
def reorder_list(head):
    if not head or not head.next:
        return head
    slow = fast = head
    while fast.next and fast.next.next:            # find the end of the first half
        slow = slow.next
        fast = fast.next.next
    second = slow.next                             # second half starts after slow
    slow.next = None                               # cut the list into two parts
    prev = None
    while second:                                  # reverse the second half
        nxt = second.next
        second.next = prev
        prev = second
        second = nxt
    first, second = head, prev                     # interleave: first, second, first, second ...
    while second:
        n1, n2 = first.next, second.next           # save both next pointers
        first.next = second                        # first -> second
        second.next = n1                           # second -> next of first
        first, second = n1, n2                     # advance both
    return head

print(to_list(reorder_list(build_list([1, 2, 3, 4]))))
print(to_list(reorder_list(build_list([1, 2, 3, 4, 5]))))
```
**Output**
```
[1, 4, 2, 3]
[1, 5, 2, 4, 3]
```
**Complexity:** O(n) time, O(1) space.

---

### 🔴 Problem 11 — Sort List (Medium-Hard) · *Merge sort on a linked list*
**Idea:** Arrays are sorted with quick/merge sort; for linked lists **merge sort** is ideal: split at the middle (fast/slow), sort both halves recursively, merge (Problem 3). No random access needed, O(n log n), and no extra arrays.

```python
# uses ListNode, build_list, to_list from Section 4
def merge_two(l1, l2):
    dummy = ListNode()
    tail = dummy
    while l1 and l2:
        if l1.val <= l2.val:
            tail.next, l1 = l1, l1.next            # take from l1
        else:
            tail.next, l2 = l2, l2.next            # take from l2
        tail = tail.next
    tail.next = l1 or l2                           # append leftovers
    return dummy.next

def sort_list(head):
    if not head or not head.next:                  # base case: 0 or 1 node is already sorted
        return head
    slow, fast = head, head.next                   # fast starts one ahead so slow ends at the END of the first half
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    mid = slow.next                                # head of the second half
    slow.next = None                               # cut the list in two
    left = sort_list(head)                         # sort first half
    right = sort_list(mid)                         # sort second half
    return merge_two(left, right)                  # combine

print(to_list(sort_list(build_list([4, 2, 1, 3]))))
print(to_list(sort_list(build_list([-1, 5, 3, 4, 0]))))
```
**Output**
```
[1, 2, 3, 4]
[-1, 0, 3, 4, 5]
```
**Complexity:** O(n log n) time, O(log n) recursion stack.

---

### 🔴 Problem 12 — Reverse Nodes in k-Group (Hard) · *Group reversal with dummy*
**Idea:** For each block of `k` nodes: (1) check that `k` nodes exist (find the k-th node), (2) reverse the block while connecting its new end to the next block, (3) reconnect the previous block's tail. Leftover nodes (< k) stay unchanged.

```python
# uses ListNode, build_list, to_list from Section 4
def reverse_k_group(head, k):
    dummy = ListNode(0, head)
    group_prev = dummy                             # node just before the current group
    while True:
        kth = group_prev
        for _ in range(k):                         # find the k-th node of this group
            kth = kth.next
            if not kth:                            # fewer than k nodes left -> done
                return dummy.next
        group_next = kth.next                      # first node after the group
        prev, cur = group_next, group_prev.next    # reverse the group; the group's tail will point to group_next
        while cur is not group_next:
            nxt = cur.next
            cur.next = prev
            prev = cur
            cur = nxt
        tmp = group_prev.next                      # old first node of the group is now its tail
        group_prev.next = kth                      # connect previous part to the new group head (old kth)
        group_prev = tmp                           # move to the end of the reversed group

print(to_list(reverse_k_group(build_list([1, 2, 3, 4, 5]), 2)))
print(to_list(reverse_k_group(build_list([1, 2, 3, 4, 5]), 3)))
```
**Output**
```
[2, 1, 4, 3, 5]
[3, 2, 1, 4, 5]
```
**Complexity:** O(n) time, O(1) space.

---

### 🔴 Problem 13 — Copy List with Random Pointer (Medium-Hard) · *Hash map old → new*
**Idea:** Every node has `next` and an extra `random` pointer. A **deep copy** needs new nodes with the same structure. Pass 1: create a copy of every node and store `mp[old] = new`. Pass 2: wire `next` and `random` using the map.

```python
class RNode:
    def __init__(self, val, next=None, random=None):
        self.val = val
        self.next = next
        self.random = random                       # may point to any node or None

def copy_random_list(head):
    if not head:
        return None
    mp = {}                                        # original node -> its copy
    cur = head
    while cur:                                     # pass 1: create all copies (no links yet)
        mp[cur] = RNode(cur.val)
        cur = cur.next
    cur = head
    while cur:                                     # pass 2: set next and random using the map
        mp[cur].next = mp.get(cur.next)            # .get(None) -> None keeps the end of the list
        mp[cur].random = mp.get(cur.random)
        cur = cur.next
    return mp[head]

def build_random(pairs):                           # pairs = [[val, random_index or None], ...]
    nodes = [RNode(v) for v, _ in pairs]
    for i, (_, r) in enumerate(pairs):
        if i + 1 < len(nodes):
            nodes[i].next = nodes[i + 1]
        if r is not None:
            nodes[i].random = nodes[r]
    return nodes[0], nodes

def serialize(head):                               # back to [[val, random_index or None], ...]
    order, cur = [], head
    while cur:
        order.append(cur)
        cur = cur.next
    index = {id(n): i for i, n in enumerate(order)}
    return [[n.val, index[id(n.random)] if n.random else None] for n in order]

head, originals = build_random([[7, None], [13, 0], [11, 4], [10, 2], [1, 0]])
copy = copy_random_list(head)
print(serialize(copy))                             # same structure as the original
copies = []
cur = copy
while cur:
    copies.append(cur)
    cur = cur.next
print(all(c is not o for c, o in zip(copies, originals)))   # no node is shared -> real deep copy
```
**Output**
```
[[7, None], [13, 0], [11, 4], [10, 2], [1, 0]]
True
```
**Complexity:** O(n) time, O(n) space. (An O(1)-space version interleaves copies between original nodes.)

---

### 🔴 Problem 14 — Merge K Sorted Lists (Hard) · *Min-heap*
**Idea:** Keep a min-heap with the current head of each list. Pop the smallest, append it, push its `next`. The index `i` in the tuple breaks ties so Python never compares two `ListNode` objects.

```python
import heapq
# uses ListNode, build_list, to_list from Section 4
def merge_k_lists(lists):
    heap = []
    for i, node in enumerate(lists):
        if node:
            heapq.heappush(heap, (node.val, i, node))   # (value, tie-breaker, node)
    dummy = ListNode()
    cur = dummy
    while heap:
        val, i, node = heapq.heappop(heap)         # smallest current value among all lists
        cur.next = node                            # append it to the result
        cur = cur.next
        if node.next:                              # push the next node of the same list
            heapq.heappush(heap, (node.next.val, i, node.next))
    return dummy.next

lists = [build_list([1, 4, 5]), build_list([1, 3, 4]), build_list([2, 6])]
print(to_list(merge_k_lists(lists)))
print(to_list(merge_k_lists([])))
```
**Output**
```
[1, 1, 2, 3, 4, 4, 5, 6]
[]
```
**Complexity:** O(N log k) time (N total nodes, k lists), O(k) space. (Alternative: divide & conquer merging pairs, O(N log k), O(1) extra.)

---

### 🔴 Problem 15 — LRU Cache (Hard) · *Doubly linked list + hash map*
**Idea:** Need `get` and `put` in **O(1)** and evict the **least recently used** key. The hash map gives O(1) access to a node; the **doubly linked list** keeps usage order (most recent next to the head, least recent next to the tail) and lets us move/remove a node in O(1). Dummy `head`/`tail` sentinels remove edge cases.

```python
class DNode:
    def __init__(self, key=0, val=0):
        self.key = key                             # stored so we can delete it from the map on eviction
        self.val = val
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity):
        self.cap = capacity
        self.map = {}                              # key -> node
        self.head = DNode()                        # sentinel: most recent side
        self.tail = DNode()                        # sentinel: least recent side
        self.head.next = self.tail
        self.tail.prev = self.head

    def _remove(self, node):                       # unlink a node in O(1)
        node.prev.next = node.next
        node.next.prev = node.prev

    def _add_front(self, node):                    # insert right after the head sentinel in O(1)
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node

    def get(self, key):
        if key not in self.map:
            return -1
        node = self.map[key]
        self._remove(node)                         # mark as most recently used:
        self._add_front(node)                      # move it to the front
        return node.val

    def put(self, key, value):
        if key in self.map:                        # update existing key
            node = self.map[key]
            node.val = value
            self._remove(node)
            self._add_front(node)
        else:
            if len(self.map) == self.cap:          # full -> evict the least recently used (before tail)
                lru = self.tail.prev
                self._remove(lru)
                del self.map[lru.key]
            node = DNode(key, value)
            self.map[key] = node
            self._add_front(node)

c = LRUCache(2)
c.put(1, 1)
c.put(2, 2)
print(c.get(1))                                    # 1, key 1 becomes most recent
c.put(3, 3)                                        # evicts key 2
print(c.get(2))                                    # -1
c.put(4, 4)                                        # evicts key 1
print(c.get(1))                                    # -1
print(c.get(3), c.get(4))
```
**Output**
```
1
-1
-1
3 4
```
**Complexity:** O(1) per operation, O(capacity) space.

---

## 10. Cheat Sheet, Mistakes & Practice List

### Problem → technique map
| # | Problem | Technique |
|---|---|---|
| 1 | Reverse Linked List | 3-pointer reversal |
| 2 | Middle of the List | fast & slow |
| 3 | Merge Two Sorted Lists | dummy + two pointers |
| 4 | Remove Duplicates (sorted) | single pass |
| 5 | Cycle Detection + Start | Floyd |
| 6 | Remove N-th From End | gap pointers + dummy |
| 7 | Intersection of Two Lists | pointer switching |
| 8 | Palindrome List | middle + reverse half |
| 9 | Add Two Numbers | dummy + carry |
| 10 | Reorder List | middle + reverse + interleave |
| 11 | Sort List | merge sort |
| 12 | Reverse in k-Group | group reversal |
| 13 | Copy with Random Pointer | hash map old→new |
| 14 | Merge K Sorted Lists | min-heap |
| 15 | LRU Cache | doubly linked list + hash map |

### Templates to memorise
```py
# 1) Reverse
prev, cur = None, head
while cur:
    nxt = cur.next; cur.next = prev; prev = cur; cur = nxt
return prev

# 2) Fast & slow (middle / cycle)
slow = fast = head
while fast and fast.next:
    slow = slow.next; fast = fast.next.next

# 3) Dummy node
dummy = ListNode(0, head); cur = dummy
...
return dummy.next
```

### Common mistakes
1. **Losing the rest of the list:** always save `nxt = cur.next` *before* changing `cur.next`.
2. **Null pointer errors:** check `if cur and cur.next` before `cur.next.next`.
3. Forgetting the **dummy node** when the head can change (delete/merge).
4. **Not cutting the list** (`slow.next = None`) before reversing/sorting a half → creates cycles.
5. Comparing **values** instead of **node identity** (`is`) for intersection/cycle problems.
6. Off-by-one in the "gap" technique (`n` vs `n+1` steps).
7. Infinite loops on circular/cyclic lists — stop condition needed.
8. Returning the wrong head after reversing (return `prev`, not `cur`).
9. Recursion depth on long lists (Python limit ≈ 1000) → prefer iteration.

### Interview talking points
- "Linked lists trade O(1) insert/delete for O(n) access."
- "I use a dummy node so the head is never a special case."
- "Fast/slow pointers solve middle, cycle and palindrome problems in O(1) space."
- "A doubly linked list + hash map gives an O(1) LRU cache."

### Practice list
| Level | Problems |
|---|---|
| Easy | Reverse Linked List (206), Merge Two Sorted Lists (21), Middle of the Linked List (876), Linked List Cycle (141), Remove Linked List Elements (203), Palindrome Linked List (234), Intersection of Two Linked Lists (160), Delete Node in a Linked List (237), Remove Duplicates from Sorted List (83) |
| Medium | Add Two Numbers (2), Remove Nth Node From End (19), Linked List Cycle II (142), Odd Even Linked List (328), Swap Nodes in Pairs (24), Rotate List (61), Reorder List (143), Sort List (148), Copy List with Random Pointer (138), Reverse Linked List II (92), Partition List (86), Remove Duplicates from Sorted List II (82), LRU Cache (146), Flatten a Multilevel Doubly Linked List (430) |
| Hard | Reverse Nodes in k-Group (25), Merge k Sorted Lists (23), LFU Cache (460), All O`one Data Structure (432) |

### ✅ Final advice for 12 November 2026
- Draw **boxes and arrows** for every pointer problem and dry-run with 3–4 nodes plus the edge cases (empty, one node, two nodes).
- Be able to write **reverse**, **middle**, **merge two lists** and **Floyd cycle** from memory — they are building blocks of most other problems.
- Topics finished: **Math → Arrays → Searching → Sorting → Strings → Recursion → Hashing → Stack & Queue → Linked List**. Suggested next: **Trees (Binary Tree, BST), Graphs, Dynamic Programming**. 🚀
