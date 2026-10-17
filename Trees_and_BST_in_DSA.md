# 🌳 Trees & Binary Search Trees (BST) in DSA (Python) — From Scratch to Advanced Interview Problems

> Easy-language guide: what a tree is → terminology → binary trees → construction → all traversals (recursive + iterative + BFS) → BST (search, insert, delete) → patterns → **15 standard interview problems** with commented code and verified output.

---

## 📑 Table of Contents
1. [What is a Tree? (Terminology)](#1-what-is-a-tree)
2. [Types of Binary Trees](#2-types-of-binary-trees)
3. [Node, Construction & Helper Functions](#3-node-construction--helper-functions)
4. [Tree Traversals (DFS & BFS)](#4-tree-traversals)
5. [Measuring a Tree & Important Properties](#5-measuring-a-tree--important-properties)
6. [Binary Search Tree (BST)](#6-binary-search-tree-bst)
7. [Balanced BSTs & Complexity](#7-balanced-bsts--complexity)
8. [Tree Patterns (Techniques)](#8-tree-patterns-techniques)
9. [15 Standard Interview Problems](#9-15-standard-interview-problems)
10. [Cheat Sheet, Mistakes & Practice List](#10-cheat-sheet-mistakes--practice-list)

---

## 1. What is a Tree?

A **tree** is a **non-linear, hierarchical** data structure made of **nodes** connected by **edges**, with **no cycles**. Think of a family tree or a folder structure on your computer.

```
            A          <- root (level 0)
          /   \
         B     C       <- level 1   (B and C are siblings, children of A)
        / \     \
       D   E     F     <- level 2   (D, E, F are leaves)
```

### Terminology (learn these exactly, interviewers use them)
| Term | Meaning | In the picture |
|---|---|---|
| **Node** | one element holding data | A, B, C, ... |
| **Edge** | link between parent and child | A–B |
| **Root** | topmost node (no parent) | A |
| **Parent / Child** | node directly above / below | A is parent of B |
| **Siblings** | same parent | B and C |
| **Leaf** | node with **no children** | D, E, F |
| **Internal node** | node with at least one child | A, B, C |
| **Ancestor / Descendant** | any node on the path up / down | A is an ancestor of D |
| **Subtree** | a node together with all its descendants | subtree of B = {B, D, E} |
| **Degree** | number of children of a node | degree(B) = 2 |
| **Level / Depth** | distance (edges) from the **root** | depth(D) = 2 |
| **Height of a node** | longest distance (edges) **down to a leaf** | height(A) = 2 |
| **Height of the tree** | height of the root | 2 |

> ⚠️ Some books count height in **nodes** (a single node has height 1), others in **edges** (height 0). LeetCode "maximum depth" counts **nodes**. Always check the problem statement.

**Facts to remember**
- A tree with **n nodes has exactly n − 1 edges**.
- There is **exactly one path** between any two nodes.
- A tree is a special kind of **graph** (connected, acyclic).

**Why trees?** Fast search/insert/delete (BST: O(log n)), hierarchical data (file systems, HTML DOM), priority queues (heaps), prefix search (Trie), compilers (expression trees), databases (B-trees), routing.

---

## 2. Types of Binary Trees

A **binary tree** = every node has **at most 2 children**: `left` and `right`.

| Type | Definition | Example idea |
|---|---|---|
| **Full (strict)** | every node has 0 or 2 children | no node with exactly one child |
| **Complete** | all levels full except possibly the last, which is filled **left to right** | binary heap |
| **Perfect** | all internal nodes have 2 children and all leaves are on the same level | `2^(h+1) − 1` nodes |
| **Balanced (height-balanced)** | for every node the heights of left and right subtrees differ by **at most 1** | AVL tree |
| **Degenerate / Skewed** | every node has only one child (behaves like a linked list) | sorted insert in a plain BST |
| **Binary Search Tree** | `left subtree < node < right subtree` for **every** node | search tree |

```
Perfect          Complete          Skewed (degenerate)
    1                1               1
   / \              / \               \
  2   3            2   3               2
 / \ / \          / \ /                 \
4  5 6  7        4  5 6                  3
```

**Key formulas (height h counted in edges)**
- Max nodes at level `i` = `2^i`
- Max nodes in a tree of height `h` = `2^(h+1) − 1`
- Minimum height of a tree with `n` nodes ≈ `log₂(n)`
- Maximum height (skewed) = `n − 1`

---

## 3. Node, Construction & Helper Functions

### 3.1 The node
```
        +---------+
        |   val   |
        +----+----+
        |left|right|
        +----+----+
```

### 3.2 Helpers used by all problems (paste these above any problem to run it)
`build_tree` builds a tree from a **level-order list** (the same format LeetCode uses; `None` = missing child) and `tree_to_list` converts it back for printing.

```python
# HELPERS ------------------------------------------------------------
from collections import deque

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val                             # data stored in the node
        self.left = left                           # left child (or None)
        self.right = right                         # right child (or None)

def build_tree(arr):                               # [1,2,3,None,4] -> tree (level-order input)
    if not arr or arr[0] is None:
        return None
    root = TreeNode(arr[0])
    q = deque([root])                              # nodes waiting for their children
    i = 1
    while q and i < len(arr):
        node = q.popleft()
        if i < len(arr) and arr[i] is not None:    # next value is the left child
            node.left = TreeNode(arr[i])
            q.append(node.left)
        i += 1
        if i < len(arr) and arr[i] is not None:    # following value is the right child
            node.right = TreeNode(arr[i])
            q.append(node.right)
        i += 1
    return root

def tree_to_list(root):                            # tree -> level-order list with None for gaps
    if not root:
        return []
    res, q = [], deque([root])
    while q:
        node = q.popleft()
        if node:
            res.append(node.val)
            q.append(node.left)
            q.append(node.right)
        else:
            res.append(None)
    while res and res[-1] is None:                 # remove trailing Nones
        res.pop()
    return res

print(tree_to_list(build_tree([1, 2, 3, None, 4])))
```
**Output**
```
[1, 2, 3, None, 4]
```

### 3.3 Building a tree manually (what `build_tree` does internally)
```python
# uses TreeNode, tree_to_list from Section 3.2
root = TreeNode(1)                                 # root node
root.left = TreeNode(2)                            # attach children one by one
root.right = TreeNode(3)
root.left.left = TreeNode(4)
root.left.right = TreeNode(5)
root.right.right = TreeNode(6)
#         1
#        / \
#       2   3
#      / \   \
#     4   5   6
print(tree_to_list(root))

# Array representation (used by heaps): node at index i has
#   left child = 2*i + 1, right child = 2*i + 2, parent = (i - 1) // 2
arr = [1, 2, 3, 4, 5, None, 6]
i = 1                                              # node with value 2
print(arr[2 * i + 1], arr[2 * i + 2], arr[(i - 1) // 2])   # its children 4,5 and parent 1
```
**Output**
```
[1, 2, 3, 4, 5, None, 6]
4 5 1
```

The tree above is used in the next section.

---

## 4. Tree Traversals

**Traversal** = visiting every node exactly once. There are two families:

| Family | Orders | Data structure |
|---|---|---|
| **DFS** (Depth-First) | Preorder, Inorder, Postorder | recursion / stack |
| **BFS** (Breadth-First) | Level order | queue |

All traversals take **O(n)** time. Space = **O(h)** for DFS (recursion stack, h = height) and **O(w)** for BFS (w = maximum width).

### 4.1 DFS orders — when is the node visited?
| Order | Sequence | Typical use |
|---|---|---|
| **Preorder** | **Root** → Left → Right | copy/serialize a tree, prefix expressions |
| **Inorder** | Left → **Root** → Right | **BST gives sorted order** |
| **Postorder** | Left → Right → **Root** | delete a tree, evaluate expression trees, bottom-up answers |

For the tree `1 / (2 / 4,5) / (3 \ 6)`:
- Preorder: `1 2 4 5 3 6`
- Inorder: `4 2 5 1 3 6`
- Postorder: `4 5 2 6 3 1`
- Level order: `[1] [2 3] [4 5 6]`

### 4.2 Recursive DFS (3 lines each, very important)
```python
# uses build_tree from Section 3.2
def preorder(node, out):
    if not node:                                   # base case: empty subtree
        return
    out.append(node.val)                           # visit ROOT first
    preorder(node.left, out)                       # then the whole left subtree
    preorder(node.right, out)                      # then the whole right subtree

def inorder(node, out):
    if not node:
        return
    inorder(node.left, out)                        # left subtree first
    out.append(node.val)                           # visit ROOT in the middle
    inorder(node.right, out)                       # right subtree last

def postorder(node, out):
    if not node:
        return
    postorder(node.left, out)                      # left subtree
    postorder(node.right, out)                     # right subtree
    out.append(node.val)                           # ROOT last

root = build_tree([1, 2, 3, 4, 5, None, 6])
for fn in (preorder, inorder, postorder):
    out = []
    fn(root, out)
    print(fn.__name__, out)
```
**Output**
```
preorder [1, 2, 4, 5, 3, 6]
inorder [4, 2, 5, 1, 3, 6]
postorder [4, 5, 2, 6, 3, 1]
```

### 4.3 Iterative DFS (explicit stack) — often asked in interviews
```python
# uses build_tree from Section 3.2
def preorder_iter(root):
    res, stack = [], [root] if root else []
    while stack:
        node = stack.pop()                         # take the top node
        res.append(node.val)                       # visit it immediately (root first)
        if node.right:
            stack.append(node.right)               # push RIGHT first so that LEFT is popped first
        if node.left:
            stack.append(node.left)
    return res

def inorder_iter(root):
    res, stack, cur = [], [], root
    while cur or stack:
        while cur:                                 # go as far left as possible
            stack.append(cur)
            cur = cur.left
        cur = stack.pop()                          # leftmost unvisited node
        res.append(cur.val)                        # visit it (root of its own subtree)
        cur = cur.right                            # then handle its right subtree
    return res

def postorder_iter(root):
    res, stack = [], [root] if root else []
    while stack:                                   # produce Root -> Right -> Left ...
        node = stack.pop()
        res.append(node.val)
        if node.left:
            stack.append(node.left)
        if node.right:
            stack.append(node.right)
    return res[::-1]                               # ... then reverse it to get Left -> Right -> Root

root = build_tree([1, 2, 3, 4, 5, None, 6])
print(preorder_iter(root))
print(inorder_iter(root))
print(postorder_iter(root))
```
**Output**
```
[1, 2, 4, 5, 3, 6]
[4, 2, 5, 1, 3, 6]
[4, 5, 2, 6, 3, 1]
```
> **Morris Traversal** does inorder in O(1) extra space by temporarily threading the tree (advanced, know the name).

### 4.4 BFS — Level Order Traversal
Use a **queue**. Process the tree level by level: each loop handles exactly the nodes currently in the queue (`len(q)` of them).

```python
from collections import deque
# uses build_tree from Section 3.2
def level_order(root):
    if not root:
        return []
    res, q = [], deque([root])                     # queue starts with the root
    while q:
        level = []
        for _ in range(len(q)):                    # exactly the nodes of the current level
            node = q.popleft()
            level.append(node.val)
            if node.left:
                q.append(node.left)                # children form the next level
            if node.right:
                q.append(node.right)
        res.append(level)
    return res

print(level_order(build_tree([1, 2, 3, 4, 5, None, 6])))
```
**Output**
```
[[1], [2, 3], [4, 5, 6]]
```

---

## 5. Measuring a Tree & Important Properties

Almost every tree problem uses a **recursive formula** based on the left and right subtrees:

```python
# uses build_tree from Section 3.2
def height(node):                                  # height in NODES (empty tree = 0)
    if not node:
        return 0
    return 1 + max(height(node.left), height(node.right))   # 1 for this node + taller subtree

def count_nodes(node):
    if not node:
        return 0
    return 1 + count_nodes(node.left) + count_nodes(node.right)

def count_leaves(node):
    if not node:
        return 0
    if not node.left and not node.right:           # a leaf has no children
        return 1
    return count_leaves(node.left) + count_leaves(node.right)

def tree_sum(node):
    if not node:
        return 0
    return node.val + tree_sum(node.left) + tree_sum(node.right)

root = build_tree([1, 2, 3, 4, 5, None, 6])
print(height(root), count_nodes(root), count_leaves(root), tree_sum(root))
```
**Output**
```
3 6 3 21
```

**The recursive mindset (most important idea for trees):**
1. **Base case:** `if not node: return <something for an empty tree>`.
2. **Trust** that the function already works for `node.left` and `node.right`.
3. **Combine** the two answers with the current node's value.

---

## 6. Binary Search Tree (BST)

### 6.1 Definition
For **every** node:
- all values in the **left subtree** are **smaller**,
- all values in the **right subtree** are **larger**.
(Usually no duplicates; if allowed, decide one side and stay consistent.)

```
          50
        /    \
      30      70
     /  \    /  \
    20  40  60  80
```
**Superpower:** the **inorder traversal of a BST is sorted** (`20 30 40 50 60 70 80`).

### 6.2 Search — go left or right (like binary search)
- `target == node.val` → found
- `target < node.val` → search left
- `target > node.val` → search right

### 6.3 Insert — search for the spot where the value *would* be, attach a new leaf there.

### 6.4 Delete — the 3 cases (most asked!)
| Case | What to do |
|---|---|
| **1. Leaf** (no children) | simply remove it |
| **2. One child** | replace the node with its child |
| **3. Two children** | replace the node's value with its **inorder successor** (smallest value in the right subtree), then delete that successor from the right subtree |

*(You can use the inorder predecessor = largest in the left subtree instead.)*

### 6.5 Full BST class

```python
# uses TreeNode, tree_to_list from Section 3.2
class BST:
    def __init__(self):
        self.root = None

    def insert(self, val):                         # O(h)
        if not self.root:
            self.root = TreeNode(val)
            return
        cur = self.root
        while True:
            if val < cur.val:                      # belongs in the left subtree
                if cur.left is None:
                    cur.left = TreeNode(val)       # found the empty spot
                    return
                cur = cur.left
            elif val > cur.val:                    # belongs in the right subtree
                if cur.right is None:
                    cur.right = TreeNode(val)
                    return
                cur = cur.right
            else:
                return                             # duplicate: ignore

    def search(self, val):                         # O(h)
        cur = self.root
        while cur:
            if val == cur.val:
                return True
            cur = cur.left if val < cur.val else cur.right   # discard one whole half
        return False

    def find_min(self, node=None):                 # leftmost node
        cur = node or self.root
        while cur.left:
            cur = cur.left
        return cur.val

    def find_max(self):                            # rightmost node
        cur = self.root
        while cur.right:
            cur = cur.right
        return cur.val

    def delete(self, val):                         # O(h)
        self.root = self._delete(self.root, val)

    def _delete(self, node, val):
        if not node:                               # value not found
            return None
        if val < node.val:
            node.left = self._delete(node.left, val)    # delete in the left subtree
        elif val > node.val:
            node.right = self._delete(node.right, val)  # delete in the right subtree
        else:                                      # this is the node to delete
            if not node.left:                      # case 1 and 2: no left child -> right child takes its place
                return node.right
            if not node.right:                     # only a left child -> it takes its place
                return node.left
            succ = node.right                      # case 3: two children -> find the inorder successor
            while succ.left:
                succ = succ.left
            node.val = succ.val                    # copy successor's value here
            node.right = self._delete(node.right, succ.val)   # remove the successor node
        return node

    def inorder(self):
        res, stack, cur = [], [], self.root
        while cur or stack:
            while cur:
                stack.append(cur)
                cur = cur.left
            cur = stack.pop()
            res.append(cur.val)
            cur = cur.right
        return res

bst = BST()
for v in [50, 30, 70, 20, 40, 60, 80]:
    bst.insert(v)
print(tree_to_list(bst.root))                      # level-order view of the tree
print(bst.inorder())                               # sorted output
print(bst.search(60), bst.search(65))
print(bst.find_min(), bst.find_max())
bst.delete(20)                                     # case 1: leaf
print(tree_to_list(bst.root))
bst.delete(30)                                     # case 2: one child (40)
print(tree_to_list(bst.root))
bst.delete(50)                                     # case 3: two children, successor 60 replaces it
print(tree_to_list(bst.root), bst.inorder())
```
**Output**
```
[50, 30, 70, 20, 40, 60, 80]
[20, 30, 40, 50, 60, 70, 80]
True False
20 80
[50, 30, 70, None, 40, 60, 80]
[50, 40, 70, None, None, 60, 80]
[60, 40, 70, None, None, None, 80] [40, 60, 70, 80]
```

### 6.6 Why BST shape matters: degenerate tree
```python
# uses TreeNode from Section 3.2
def insert(root, val):                             # plain recursive BST insert
    if not root:
        return TreeNode(val)
    if val < root.val:
        root.left = insert(root.left, val)
    else:
        root.right = insert(root.right, val)
    return root

def height(node):
    return 0 if not node else 1 + max(height(node.left), height(node.right))

sorted_root = None
for v in [1, 2, 3, 4, 5, 6, 7]:                    # inserting sorted data
    sorted_root = insert(sorted_root, v)
balanced_root = None
for v in [4, 2, 6, 1, 3, 5, 7]:                    # same values, balanced insertion order
    balanced_root = insert(balanced_root, v)
print(height(sorted_root), height(balanced_root))  # 7 (like a linked list) vs 3
```
**Output**
```
7 3
```
Same 7 values, but one tree is height 7 (operations O(n)) and the other is height 3 (O(log n)). This is why **self-balancing trees** exist.

### 6.7 Inorder successor / predecessor (useful facts)
- **Successor** of a node = the next larger value: the **leftmost node of its right subtree**, or (if no right subtree) the lowest ancestor for which the node is in the left subtree.
- **Predecessor** = the **rightmost node of its left subtree**, or the lowest ancestor for which the node is in the right subtree.

---

## 7. Balanced BSTs & Complexity

| Structure | Search | Insert | Delete | Notes |
|---|---|---|---|---|
| BST (balanced) | O(log n) | O(log n) | O(log n) | h ≈ log n |
| BST (skewed) | **O(n)** | **O(n)** | **O(n)** | h ≈ n |
| **AVL tree** | O(log n) | O(log n) | O(log n) | strictly balanced (|balance factor| ≤ 1), uses **rotations** |
| **Red-Black tree** | O(log n) | O(log n) | O(log n) | looser balance, fewer rotations (used in C++ `std::map`, Java `TreeMap`) |
| **B-Tree / B+ Tree** | O(log n) | O(log n) | O(log n) | wide nodes, used in **databases** |
| **Heap** | O(n) | O(log n) | O(log n) | complete tree stored in an array |
| **Trie** | O(L) | O(L) | O(L) | prefix tree for strings |

Traversal complexity: **time O(n), space O(h)**. Python has no built-in balanced BST; in practice use `sortedcontainers.SortedList` or `bisect` on a sorted list, but interviews expect you to know the ideas.

---

## 8. Tree Patterns (Techniques)

### Pattern 1 — Recursive DFS: Top-Down vs Bottom-Up
| Style | Idea | Examples |
|---|---|---|
| **Top-down** | pass information **down** as parameters (range limits, remaining sum, depth) | Validate BST, Path Sum II |
| **Bottom-up** | children **return** information, parent combines it | Height, Diameter, Max Path Sum, Balanced tree |

For bottom-up problems with a "best answer anywhere in the tree", keep a **global/nonlocal** variable, update it at each node, but **return** the value the parent needs (these two are often different!).

### Pattern 2 — BFS / Level Order
Queue + "process `len(q)` nodes per level". For: level order, zigzag, right side view, minimum depth, level averages, connect next pointers.

### Pattern 3 — Backtracking on Paths
Append node to `path`, recurse, **pop** on the way back. For: root-to-leaf paths, path sum II.

### Pattern 4 — BST Inorder Property
Inorder = sorted. For: kth smallest, validate BST, recover BST, closest value, two-sum in BST, BST iterator.

### Pattern 5 — Divide & Conquer Construction
Pick the root, recursively build left and right parts. For: sorted array → BST, preorder + inorder → tree.

### Pattern 6 — Lowest Common Ancestor (LCA)
BST: walk down using values. General tree: recursion returns the node if found; the node where **both** sides report something is the LCA.

### Pattern 7 — Serialize / Deserialize
Preorder with null markers uniquely identifies a tree.

### Pattern 8 — Parent pointers / hash maps
When you need to walk **up** (nodes at distance K, all paths to the root), build a `child → parent` map.

### Recognition table
| Clue | Technique |
|---|---|
| "depth", "height", "balanced", "diameter" | bottom-up DFS |
| "level by level", "right side view", "minimum depth" | BFS |
| "root to leaf path", "path sum" | DFS + backtracking |
| "BST", "sorted order", "kth" | inorder |
| "valid BST" | range limits (top-down) |
| "common ancestor" | LCA recursion |
| "build tree from traversals / array" | divide & conquer + hash map of inorder indexes |
| "save / transfer a tree as a string" | preorder with `#` |
| "mirror", "symmetric", "same tree" | recurse on two nodes at once |

---

## 9. 15 Standard Interview Problems

> Problems use `TreeNode`, `build_tree`, `tree_to_list` from **Section 3.2**.

---

### 🟢 Problem 1 — Maximum Depth of Binary Tree (Easy) · *Bottom-up DFS (and BFS)*
**Idea:** depth of a tree = 1 + the deeper of its two subtrees. Empty tree = 0. BFS alternative: count the number of levels.

```python
from collections import deque
# uses TreeNode, build_tree from Section 3.2
def max_depth(root):
    if not root:                                   # empty tree has depth 0
        return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))   # this node + deeper child

def max_depth_bfs(root):
    if not root:
        return 0
    depth, q = 0, deque([root])
    while q:
        depth += 1                                 # one more level
        for _ in range(len(q)):                    # consume the whole level
            node = q.popleft()
            if node.left:
                q.append(node.left)
            if node.right:
                q.append(node.right)
    return depth

print(max_depth(build_tree([3, 9, 20, None, None, 15, 7])))
print(max_depth(build_tree([1, None, 2])))
print(max_depth(build_tree([])))
print(max_depth_bfs(build_tree([3, 9, 20, None, None, 15, 7])))
```
**Output**
```
3
2
0
3
```
**Complexity:** O(n) time, O(h) space.

---

### 🟢 Problem 2 — Invert (Mirror) Binary Tree (Easy) · *Recursion, swap children*
**Idea:** swap the left and right children of every node. Recursively invert both subtrees.

```python
# uses TreeNode, build_tree, tree_to_list from Section 3.2
def invert_tree(root):
    if not root:
        return None
    root.left, root.right = invert_tree(root.right), invert_tree(root.left)   # swap and invert subtrees
    return root

print(tree_to_list(invert_tree(build_tree([4, 2, 7, 1, 3, 6, 9]))))
print(tree_to_list(invert_tree(build_tree([2, 1, 3]))))
print(tree_to_list(invert_tree(build_tree([]))))
```
**Output**
```
[4, 7, 2, 9, 6, 3, 1]
[2, 3, 1]
[]
```
**Complexity:** O(n) time, O(h) space.

---

### 🟢 Problem 3 — Symmetric Tree (Easy) · *Recurse on two nodes at once*
**Idea:** A tree is symmetric if its left subtree is the **mirror** of its right subtree. Two nodes mirror each other if their values match, and `a.left` mirrors `b.right` and `a.right` mirrors `b.left`.

```python
# uses TreeNode, build_tree from Section 3.2
def is_symmetric(root):
    def mirror(a, b):
        if not a and not b:                        # both empty -> symmetric
            return True
        if not a or not b or a.val != b.val:       # one missing or values differ -> not symmetric
            return False
        return mirror(a.left, b.right) and mirror(a.right, b.left)   # outer pair and inner pair
    return mirror(root.left, root.right) if root else True

print(is_symmetric(build_tree([1, 2, 2, 3, 4, 4, 3])))
print(is_symmetric(build_tree([1, 2, 2, None, 3, None, 3])))
```
**Output**
```
True
False
```
**Complexity:** O(n) time, O(h) space.

---

### 🟡 Problem 4 — Level Order Traversal & Right Side View (Medium) · *BFS*
**Idea:** Process the tree level by level. The **right side view** is simply the last node of every level.

```python
from collections import deque
# uses TreeNode, build_tree from Section 3.2
def level_order(root):
    if not root:
        return []
    res, q = [], deque([root])
    while q:
        level = []
        for _ in range(len(q)):                    # snapshot of the current level size
            node = q.popleft()
            level.append(node.val)
            if node.left:
                q.append(node.left)
            if node.right:
                q.append(node.right)
        res.append(level)
    return res

def right_side_view(root):
    return [level[-1] for level in level_order(root)]   # rightmost node of each level

print(level_order(build_tree([3, 9, 20, None, None, 15, 7])))
print(right_side_view(build_tree([1, 2, 3, None, 5, None, 4])))
print(level_order(build_tree([])))
```
**Output**
```
[[3], [9, 20], [15, 7]]
[1, 3, 4]
[]
```
**Complexity:** O(n) time, O(w) space (w = maximum width).

---

### 🟡 Problem 5 — Diameter of Binary Tree & Balanced Tree (Medium) · *Bottom-up with a global answer*
**Idea (diameter):** The longest path (in edges) through a node = `height(left) + height(right)`. Compute heights bottom-up and track the best sum seen at any node.
**Idea (balanced):** return `-1` as a "this subtree is unbalanced" signal while computing heights.

```python
# uses TreeNode, build_tree from Section 3.2
def diameter(root):
    best = 0                                       # longest path found so far (in edges)
    def height(node):
        nonlocal best
        if not node:
            return 0
        l = height(node.left)
        r = height(node.right)
        best = max(best, l + r)                    # path that bends at this node
        return 1 + max(l, r)                       # what the PARENT needs: the height
    height(root)
    return best

def is_balanced(root):
    def check(node):                               # returns height, or -1 if unbalanced
        if not node:
            return 0
        l = check(node.left)
        if l == -1:
            return -1                              # left subtree already unbalanced
        r = check(node.right)
        if r == -1:
            return -1
        if abs(l - r) > 1:                         # height difference too large here
            return -1
        return 1 + max(l, r)
    return check(root) != -1

print(diameter(build_tree([1, 2, 3, 4, 5])))       # path 4-2-1-3 or 5-2-1-3 -> 3 edges
print(diameter(build_tree([1, 2])))
print(is_balanced(build_tree([3, 9, 20, None, None, 15, 7])))
print(is_balanced(build_tree([1, 2, 2, 3, 3, None, None, 4, 4])))
```
**Output**
```
3
1
True
False
```
**Complexity:** O(n) time, O(h) space. (Calling `height()` separately at every node would be O(n²).)

---

### 🟡 Problem 6 — Path Sum II (All Root-to-Leaf Paths with a Given Sum) (Medium) · *DFS + backtracking*
**Idea:** Walk down keeping the current `path` and the `remaining` sum. At a **leaf** with `remaining == 0` save a **copy** of the path. Pop the node when returning.

```python
# uses TreeNode, build_tree from Section 3.2
def path_sum(root, target):
    res, path = [], []
    def dfs(node, remaining):
        if not node:
            return
        path.append(node.val)                      # CHOOSE: step into this node
        remaining -= node.val
        if not node.left and not node.right and remaining == 0:   # leaf with the exact sum
            res.append(path[:])                    # save a COPY of the path
        dfs(node.left, remaining)                  # EXPLORE
        dfs(node.right, remaining)
        path.pop()                                 # UN-CHOOSE: backtrack
    dfs(root, target)
    return res

tree = build_tree([5, 4, 8, 11, None, 13, 4, 7, 2, None, None, 5, 1])
print(path_sum(tree, 22))
print(path_sum(build_tree([1, 2, 3]), 5))
```
**Output**
```
[[5, 4, 11, 2], [5, 8, 4, 5]]
[]
```
**Complexity:** O(n) time (plus copying paths), O(h) space.

---

### 🟡 Problem 7 — Validate Binary Search Tree (Medium) · *Top-down with value range*
**Idea:** It is **not enough** to compare a node with its direct children. Every node must lie inside an allowed range `(low, high)`; going left tightens `high`, going right tightens `low`.

```python
# uses TreeNode, build_tree from Section 3.2
def is_valid_bst(root):
    def check(node, low, high):
        if not node:                               # empty subtree is valid
            return True
        if not (low < node.val < high):            # value outside the allowed range -> invalid
            return False
        return (check(node.left, low, node.val) and      # left values must be < node.val
                check(node.right, node.val, high))       # right values must be > node.val
    return check(root, float('-inf'), float('inf'))

print(is_valid_bst(build_tree([2, 1, 3])))
print(is_valid_bst(build_tree([5, 1, 4, None, None, 3, 6])))     # 4 < 5 on the right side
print(is_valid_bst(build_tree([5, 4, 6, None, None, 3, 7])))     # 3 is in the right subtree of 5 -> invalid
```
**Output**
```
True
False
False
```
**Complexity:** O(n) time, O(h) space. (Alternative: inorder must be strictly increasing.)

---

### 🟡 Problem 8 — Convert Sorted Array to Height-Balanced BST (Easy-Medium) · *Divide & conquer*
**Idea:** Take the **middle** element as the root so both sides have (almost) equal size; recursively build the left half and the right half.

```python
# uses TreeNode, tree_to_list from Section 3.2
def sorted_array_to_bst(nums):
    def build(lo, hi):
        if lo > hi:                                # empty range -> no node
            return None
        mid = (lo + hi) // 2                       # middle becomes the root
        return TreeNode(nums[mid], build(lo, mid - 1), build(mid + 1, hi))
    return build(0, len(nums) - 1)

def height(n):
    return 0 if not n else 1 + max(height(n.left), height(n.right))

root = sorted_array_to_bst([-10, -3, 0, 5, 9])
print(tree_to_list(root))
print(height(root))
print(tree_to_list(sorted_array_to_bst([1, 3])))
```
**Output**
```
[0, -10, 5, None, -3, None, 9]
3
[1, None, 3]
```
**Complexity:** O(n) time, O(log n) recursion depth.

---

### 🟡 Problem 9 — Kth Smallest Element in a BST (Medium) · *Iterative inorder*
**Idea:** Inorder of a BST visits values in increasing order, so the k-th visited node is the answer. Stop early (no need to traverse everything).

```python
# uses TreeNode, build_tree from Section 3.2
def kth_smallest(root, k):
    stack, cur = [], root
    while cur or stack:
        while cur:                                 # go to the smallest unvisited value
            stack.append(cur)
            cur = cur.left
        cur = stack.pop()                          # next smallest value
        k -= 1                                     # visited one more node
        if k == 0:                                 # this is the k-th node in sorted order
            return cur.val
        cur = cur.right
    return None

print(kth_smallest(build_tree([3, 1, 4, None, 2]), 1))
print(kth_smallest(build_tree([5, 3, 6, 2, 4, None, None, 1]), 3))
```
**Output**
```
1
3
```
**Complexity:** O(h + k) time, O(h) space. (If the tree is modified often, store subtree sizes in nodes for O(h).)

---

### 🔴 Problem 10 — Lowest Common Ancestor (BST and General Tree) (Medium) · *LCA recursion*
**Idea (BST):** Walk down: if both values are smaller go left; both larger go right; otherwise the current node is the split point = LCA.
**Idea (general tree):** If the current node is `p` or `q` return it. Search both subtrees; if **both** return something, the current node is the LCA; otherwise return whichever side is non-empty.

```python
from collections import deque
# uses TreeNode, build_tree from Section 3.2
def lca_bst(root, p, q):
    while root:
        if p.val < root.val and q.val < root.val:
            root = root.left                       # both targets are in the left subtree
        elif p.val > root.val and q.val > root.val:
            root = root.right                      # both targets are in the right subtree
        else:
            return root                            # paths split here (or one equals root)
    return None

def lca(root, p, q):
    if not root or root is p or root is q:         # found a target (or hit the end)
        return root
    left = lca(root.left, p, q)                    # search the left subtree
    right = lca(root.right, p, q)                  # search the right subtree
    if left and right:                             # one target on each side -> this node is the LCA
        return root
    return left or right                           # both targets are on one side (or not found)

def find(root, val):                               # helper: locate a node object by value
    q = deque([root])
    while q:
        node = q.popleft()
        if node.val == val:
            return node
        for ch in (node.left, node.right):
            if ch:
                q.append(ch)

bst = build_tree([6, 2, 8, 0, 4, 7, 9, None, None, 3, 5])
print(lca_bst(bst, find(bst, 2), find(bst, 8)).val)
print(lca_bst(bst, find(bst, 2), find(bst, 4)).val)
tree = build_tree([3, 5, 1, 6, 2, 0, 8, None, None, 7, 4])
print(lca(tree, find(tree, 5), find(tree, 1)).val)
print(lca(tree, find(tree, 5), find(tree, 4)).val)
```
**Output**
```
6
2
3
5
```
**Complexity:** BST O(h); general tree O(n); space O(h).

---

### 🔴 Problem 11 — Delete Node in a BST (Medium-Hard) · *3 deletion cases*
**Idea:** Find the node, then apply the three cases from Section 6.4 (leaf, one child, two children with inorder successor).

```python
# uses TreeNode, build_tree, tree_to_list from Section 3.2
def delete_node(root, key):
    if not root:
        return None                                # key not found
    if key < root.val:
        root.left = delete_node(root.left, key)    # keep searching left
    elif key > root.val:
        root.right = delete_node(root.right, key)  # keep searching right
    else:                                          # found the node to delete
        if not root.left:                          # no left child (leaf or only right child)
            return root.right
        if not root.right:                         # only a left child
            return root.left
        succ = root.right                          # two children: smallest node of the right subtree
        while succ.left:
            succ = succ.left
        root.val = succ.val                        # take the successor's value
        root.right = delete_node(root.right, succ.val)   # delete the successor from the right subtree
    return root

print(tree_to_list(delete_node(build_tree([5, 3, 6, 2, 4, None, 7]), 3)))    # node with two children
print(tree_to_list(delete_node(build_tree([5, 3, 6, 2, 4, None, 7]), 5)))    # root with two children
print(tree_to_list(delete_node(build_tree([5, 3, 6, 2, 4, None, 7]), 0)))    # key not present
```
**Output**
```
[5, 4, 6, 2, None, None, 7]
[6, 3, 7, 2, 4]
[5, 3, 6, 2, 4, None, 7]
```
**Complexity:** O(h) time, O(h) space.

---

### 🔴 Problem 12 — Construct Binary Tree from Preorder and Inorder (Medium-Hard) · *Divide & conquer + hash map*
**Idea:** The **first** element of preorder is the root. Its position in inorder splits the tree into the left part and the right part. Build left first (it comes next in preorder), then right. A dictionary `value → inorder index` makes finding the split O(1).

```python
# uses TreeNode, tree_to_list from Section 3.2
def build_tree_pre_in(preorder, inorder):
    pos = {v: i for i, v in enumerate(inorder)}    # value -> index in inorder (O(1) lookup)
    idx = 0                                        # next root to take from preorder
    def build(lo, hi):                             # build the subtree for inorder[lo..hi]
        nonlocal idx
        if lo > hi:                                # no elements -> empty subtree
            return None
        val = preorder[idx]                        # next preorder value is this subtree's root
        idx += 1
        node = TreeNode(val)
        mid = pos[val]                             # root's index splits inorder into left / right
        node.left = build(lo, mid - 1)             # left subtree must be built FIRST (preorder order)
        node.right = build(mid + 1, hi)
        return node
    return build(0, len(inorder) - 1)

root = build_tree_pre_in([3, 9, 20, 15, 7], [9, 3, 15, 20, 7])
print(tree_to_list(root))
print(tree_to_list(build_tree_pre_in([1], [1])))
```
**Output**
```
[3, 9, 20, None, None, 15, 7]
[1]
```
**Complexity:** O(n) time, O(n) space. (Postorder + inorder works the same way, taking roots from the **end** of postorder and building **right first**.)

---

### 🔴 Problem 13 — Serialize and Deserialize Binary Tree (Hard) · *Preorder with null markers*
**Idea:** Convert the tree to a string using preorder and write `#` for every missing child. This uniquely identifies the tree. To rebuild, read tokens in the same order: a token is a root, followed by its left subtree, then its right subtree.

```python
# uses TreeNode, build_tree, tree_to_list from Section 3.2
def serialize(root):
    out = []
    def dfs(node):
        if not node:
            out.append('#')                        # marker for an empty child
            return
        out.append(str(node.val))                  # root
        dfs(node.left)                             # left subtree
        dfs(node.right)                            # right subtree
    dfs(root)
    return ','.join(out)

def deserialize(data):
    vals = iter(data.split(','))                   # tokens consumed one by one, in preorder
    def build():
        v = next(vals)
        if v == '#':                               # empty child
            return None
        node = TreeNode(int(v))
        node.left = build()                        # next tokens describe the left subtree
        node.right = build()                       # then the right subtree
        return node
    return build()

root = build_tree([1, 2, 3, None, None, 4, 5])
s = serialize(root)
print(s)
print(tree_to_list(deserialize(s)))                # round trip gives the original tree
print(repr(serialize(None)), deserialize(serialize(None)))
```
**Output**
```
1,2,#,#,3,4,#,#,5,#,#
[1, 2, 3, None, None, 4, 5]
'#' None
```
**Complexity:** O(n) time and space.

---

### 🔴 Problem 14 — Binary Tree Maximum Path Sum (Hard) · *Bottom-up, return ≠ answer*
**Idea:** A path may start and end at **any** nodes. At each node the best path that **bends** here is `node.val + best_left_gain + best_right_gain` (ignore a negative gain by using `max(gain, 0)`). But the value **returned to the parent** can use only **one** side (a path cannot split upward): `node.val + max(left_gain, right_gain)`.

```python
# uses TreeNode, build_tree from Section 3.2
def max_path_sum(root):
    best = float('-inf')                           # best path sum anywhere in the tree
    def gain(node):                                # best downward path starting at node
        nonlocal best
        if not node:
            return 0
        l = max(gain(node.left), 0)                # drop negative branches
        r = max(gain(node.right), 0)
        best = max(best, node.val + l + r)         # path bending at this node (uses both sides)
        return node.val + max(l, r)                # parent can continue with only ONE side
    gain(root)
    return best

print(max_path_sum(build_tree([1, 2, 3])))
print(max_path_sum(build_tree([-10, 9, 20, None, None, 15, 7])))   # 15 + 20 + 7
print(max_path_sum(build_tree([-3])))
```
**Output**
```
6
42
-3
```
**Complexity:** O(n) time, O(h) space.

---

### 🔴 Problem 15 — Recover Binary Search Tree (Hard) · *Inorder property*
**Idea:** Exactly two nodes of a BST were swapped. In the inorder sequence, a swap creates **one or two "descents"** (`prev > cur`). The **first** descent gives `first = prev`; the **last** descent gives `second = cur`. Swap their values.
Example inorder `1 3 2 4` → one descent (3 > 2): swap 3 and 2. Inorder `3 2 1` → two descents: first = 3, second = 1.

```python
# uses TreeNode, build_tree, tree_to_list from Section 3.2
def recover_tree(root):
    first = second = prev = None                   # the two wrong nodes and the previous inorder node
    def inorder(node):
        nonlocal first, second, prev
        if not node:
            return
        inorder(node.left)
        if prev and prev.val > node.val:           # a "descent": order is broken here
            if not first:
                first = prev                       # first descent: left element is wrong
            second = node                          # keep updating: last descent's right element is wrong
        prev = node
        inorder(node.right)
    inorder(root)
    first.val, second.val = second.val, first.val  # swap values to fix the BST
    return root

print(tree_to_list(recover_tree(build_tree([1, 3, None, None, 2]))))
print(tree_to_list(recover_tree(build_tree([3, 1, 4, None, None, 2]))))
```
**Output**
```
[3, 1, None, None, 2]
[2, 1, 4, None, None, 3]
```
**Complexity:** O(n) time, O(h) space (Morris traversal gives O(1) space).

---

## 10. Cheat Sheet, Mistakes & Practice List

### Problem → technique map
| # | Problem | Technique |
|---|---|---|
| 1 | Max Depth | bottom-up DFS / BFS |
| 2 | Invert Tree | recursion (swap) |
| 3 | Symmetric Tree | two-node recursion |
| 4 | Level Order / Right Side View | BFS |
| 5 | Diameter / Balanced | bottom-up + global answer |
| 6 | Path Sum II | DFS + backtracking |
| 7 | Validate BST | top-down range |
| 8 | Sorted Array → BST | divide & conquer |
| 9 | Kth Smallest in BST | iterative inorder |
| 10 | Lowest Common Ancestor | LCA recursion / BST walk |
| 11 | Delete Node in BST | 3 cases + successor |
| 12 | Build from Preorder + Inorder | divide & conquer + hash map |
| 13 | Serialize / Deserialize | preorder with `#` |
| 14 | Max Path Sum | bottom-up, return ≠ answer |
| 15 | Recover BST | inorder descents |

### Templates to memorise
```py
# 1) Bottom-up DFS with global answer
def solve(root):
    best = 0
    def dfs(node):
        nonlocal best
        if not node: return 0
        l, r = dfs(node.left), dfs(node.right)
        best = max(best, <combine l, r, node>)   # answer at this node
        return <what the parent needs>           # may differ from the answer!
    dfs(root)
    return best

# 2) BFS by levels
q = deque([root])
while q:
    for _ in range(len(q)):
        node = q.popleft()
        ... push children ...

# 3) Iterative inorder
stack, cur = [], root
while cur or stack:
    while cur: stack.append(cur); cur = cur.left
    cur = stack.pop(); visit(cur); cur = cur.right
```

### Common mistakes
1. Forgetting the **base case** (`if not node`).
2. **Validating a BST** by comparing only with direct children (use ranges).
3. Using `>=` vs `>` with **duplicate** values without checking the problem rules.
4. Confusing **height in nodes vs edges** and **depth vs height**.
5. In backtracking, appending `path` instead of a **copy** `path[:]`, or forgetting `path.pop()`.
6. In bottom-up problems, **returning the answer instead of what the parent needs** (diameter, max path sum).
7. BFS: not capturing `len(q)` **before** the inner loop.
8. Deleting a BST node with two children but not removing the successor afterwards.
9. Recursion depth on skewed trees of 10⁵ nodes → use iteration or raise the recursion limit.
10. Comparing node **values** when the problem needs node **identity** (`is`).

### Interview talking points
- "Inorder traversal of a BST is sorted; that gives kth smallest, validation and recovery."
- "BST operations are O(h): O(log n) when balanced, O(n) when skewed; AVL/Red-Black trees guarantee balance."
- "Tree DP: children return information, the parent combines it; I keep a global variable for the best answer."
- "BFS uses a queue and visits level by level; DFS uses recursion/stack and is O(h) space."

### Practice list
| Level | Problems |
|---|---|
| Easy | Maximum Depth (104), Same Tree (100), Invert Binary Tree (226), Symmetric Tree (101), Path Sum (112), Diameter of Binary Tree (543), Balanced Binary Tree (110), Subtree of Another Tree (572), Minimum Depth (111), Convert Sorted Array to BST (108), Merge Two Binary Trees (617), Lowest Common Ancestor of a BST (235) |
| Medium | Level Order Traversal (102), Zigzag Level Order (103), Right Side View (199), Validate BST (98), Kth Smallest in BST (230), LCA of a Binary Tree (236), Construct from Preorder & Inorder (105), Path Sum II (113), Count Good Nodes (1448), Delete Node in a BST (450), House Robber III (337), Flatten Binary Tree to Linked List (114), Populating Next Right Pointers (116), BST Iterator (173), All Nodes Distance K (863) |
| Hard | Binary Tree Maximum Path Sum (124), Serialize and Deserialize Binary Tree (297), Recover BST (99), Vertical Order Traversal (987), Binary Tree Cameras (968), Boundary of Binary Tree (545), Longest Increasing Path in a Matrix (329, tree/graph DFS idea) |

### ✅ Final advice for 12 November 2026
- For every tree problem ask first: **"What do I need from the left and right subtrees?"** and **"What do I return to my parent?"**
- Master the **three templates** above: they cover most tree questions.
- Always dry-run on a tiny tree with the edge cases: empty tree, single node, skewed tree.
- Topics finished: **Math → Arrays → Searching → Sorting → Strings → Recursion → Hashing → Stack & Queue → Linked List → Trees & BST**. Suggested next: **Heap / Priority Queue, Graphs, Dynamic Programming, Trie**. 🚀
