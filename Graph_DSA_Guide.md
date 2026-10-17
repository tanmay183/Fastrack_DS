# Graphs in DSA — Complete Guide (Python) for Coding Rounds

> Goal: take you from "what is a graph?" to solving medium/hard interview problems.
> Language: Python 3. All code is tested.
> Read Parts 1–9 once, then solve Part 11 (15 problems) **without looking** at the solutions first.

---

## Table of Contents

1. What is a Graph? (vocabulary)
2. Types of Graphs
3. How to Store a Graph (construction)
4. Traversals: BFS and DFS
5. Graphs Hidden in Grids (matrix problems)
6. Connected Components and Cycle Detection
7. Bipartite Graph Check
8. Topological Sort (DAG)
9. Shortest Path Algorithms (BFS, 0-1 BFS, Dijkstra, Bellman-Ford, Floyd-Warshall)
10. Union-Find (DSU) and Minimum Spanning Tree (Kruskal, Prim)
11. Advanced: Bridges, Articulation Points, SCC (Kosaraju)
12. Pattern Recognition Cheat Sheet ("which algorithm for which problem?")
13. **15 Standard Interview Problems** (with approach + code + complexity)
14. Extra Practice List, Common Mistakes, Final Checklist

---

# Part 1 — What is a Graph?

A graph is just **things (nodes) connected by relationships (edges)**.

- Cities connected by roads
- People connected by friendships
- Courses connected by "prerequisite of"
- Web pages connected by links

```
   0 --- 1
   |     |
   2 --- 3 --- 4
```

Formally `G = (V, E)`: `V` = set of vertices, `E` = set of edges.

## Vocabulary you MUST know

| Term | Meaning |
|---|---|
| Vertex / Node | A point in the graph |
| Edge | A connection between two vertices |
| Adjacent / Neighbour | Two nodes joined by an edge |
| Degree | Number of edges on a node. Directed graph: **in-degree** (incoming) and **out-degree** (outgoing) |
| Path | Sequence of nodes where consecutive nodes are connected |
| Cycle | A path that starts and ends at the same node |
| Connected graph | Every node can reach every other node (undirected) |
| Component | A maximal connected piece of the graph |
| Weighted edge | Edge with a cost/distance/time |
| Self-loop | Edge from a node to itself |
| Parallel edges | More than one edge between the same pair |
| Sparse / Dense | Few edges (E ≈ V) / many edges (E ≈ V²) |

Useful facts:
- Undirected graph with `V` nodes has at most `V(V-1)/2` edges.
- A **tree** = connected graph with **no cycle**, has exactly `V-1` edges.
- Sum of degrees = `2E` (undirected).

---

# Part 2 — Types of Graphs

| Type | Idea | Example |
|---|---|---|
| Undirected | Edge works both ways | Facebook friends |
| Directed (Digraph) | Edge has direction `u -> v` | Twitter follow, prerequisites |
| Weighted | Edges carry cost | Road distances |
| Unweighted | All edges cost 1 | Social network hops |
| Cyclic / Acyclic | Has / has no cycle | |
| **DAG** | Directed Acyclic Graph | Task scheduling, build order |
| Connected / Disconnected | One piece / many pieces | |
| Bipartite | Nodes split in 2 groups, edges only between groups | Job–applicant matching |
| Tree | Connected + acyclic | File system |
| Complete | Every pair connected | |

> **Interview habit:** always ask — directed or undirected? weighted? can have cycles? disconnected? negative weights? These 5 questions pick the algorithm.

---

# Part 3 — How to Store a Graph (Construction)

Take this example (undirected, 5 nodes, 0 to 4):

```
edges = [(0,1), (0,2), (1,3), (2,3), (3,4)]
```

## 3.1 Adjacency List (USE THIS 95% OF THE TIME)

For each node store a list of its neighbours.

```python
from collections import defaultdict, deque

n = 5
edges = [(0, 1), (0, 2), (1, 3), (2, 3), (3, 4)]

# Method 1: list of lists (nodes are 0..n-1)
graph = [[] for _ in range(n)]
for u, v in edges:
    graph[u].append(v)
    graph[v].append(u)        # remove this line for a DIRECTED graph

# Method 2: dict (nodes can be strings / any labels)
g = defaultdict(list)
for u, v in edges:
    g[u].append(v)
    g[v].append(u)

print(graph)
# [[1, 2], [0, 3], [0, 3], [1, 2, 4], [3]]
```

**Weighted** graph: store `(neighbour, weight)`.

```python
wedges = [(0, 1, 4), (0, 2, 1), (2, 1, 2)]   # (u, v, w)
wgraph = [[] for _ in range(3)]
for u, v, w in wedges:
    wgraph[u].append((v, w))
    wgraph[v].append((u, w))   # only if undirected
```

- Space: `O(V + E)`
- Iterate all neighbours of `u`: `O(deg(u))`
- Check if edge `(u,v)` exists: `O(deg(u))`

## 3.2 Adjacency Matrix

`matrix[u][v] = 1` if edge exists (or the weight).

```python
matrix = [[0] * n for _ in range(n)]
for u, v in edges:
    matrix[u][v] = 1
    matrix[v][u] = 1
```

- Space: `O(V²)` — wasteful for sparse graphs
- Edge check: `O(1)`
- Good for: dense graphs, Floyd-Warshall, small `V` (≤ 400)

## 3.3 Edge List

Just the list of edges `[(u, v, w), ...]`.
Used by **Kruskal** and **Bellman-Ford**.

## 3.4 Comparison

| | Adjacency List | Adjacency Matrix | Edge List |
|---|---|---|---|
| Space | O(V+E) | O(V²) | O(E) |
| Edge exists? | O(deg) | **O(1)** | O(E) |
| All neighbours | **O(deg)** | O(V) | O(E) |
| Best for | Most problems | Dense / small V | Kruskal, Bellman-Ford |

## 3.5 Typical input conversions

```python
# 1-indexed nodes -> convert to 0-indexed or allocate n+1
graph = [[] for _ in range(n + 1)]

# "parent array" / "prerequisites [a, b] means b -> a"
for a, b in prerequisites:
    graph[b].append(a)

# Words, strings as nodes -> dict
g = defaultdict(list)
```

---

# Part 4 — Traversals: BFS and DFS

Visiting every node of a graph. You need a `visited` set, otherwise cycles cause infinite loops.

## 4.1 BFS (Breadth First Search) — level by level

Think of **ripples in water**. Uses a **queue**.

Use BFS when you need: **shortest path in an unweighted graph**, minimum number of steps/moves, level order, nearest-something.

```python
from collections import deque

def bfs(graph, start):
    n = len(graph)
    visited = [False] * n
    visited[start] = True
    q = deque([start])
    order = []
    while q:
        node = q.popleft()
        order.append(node)
        for nei in graph[node]:
            if not visited[nei]:
                visited[nei] = True      # mark when PUSHING, not popping
                q.append(nei)
    return order
```

> **Golden rule:** mark visited when you **push** into the queue. Marking on pop makes nodes enter the queue many times (slow / TLE).

**BFS shortest distance** (unweighted):

```python
def bfs_dist(graph, src):
    dist = [-1] * len(graph)
    dist[src] = 0
    q = deque([src])
    while q:
        u = q.popleft()
        for v in graph[u]:
            if dist[v] == -1:
                dist[v] = dist[u] + 1
                q.append(v)
    return dist
```

Time `O(V + E)`, space `O(V)`.

## 4.2 DFS (Depth First Search) — go deep, then backtrack

Think of **exploring a maze**: keep walking until stuck, then come back. Uses recursion (call stack) or an explicit **stack**.

Use DFS for: connected components, cycle detection, path existence, topological sort, bridges, SCC, backtracking on graphs.

```python
# Recursive
def dfs(graph, node, visited):
    visited[node] = True
    for nei in graph[node]:
        if not visited[nei]:
            dfs(graph, nei, visited)

# Iterative (safe from Python's recursion limit ~1000)
def dfs_iter(graph, start):
    visited = [False] * len(graph)
    stack = [start]
    order = []
    while stack:
        node = stack.pop()
        if visited[node]:
            continue
        visited[node] = True
        order.append(node)
        for nei in graph[node]:
            if not visited[nei]:
                stack.append(nei)
    return order
```

> **Python warning:** default recursion limit is 1000. For `n` up to 10^5 either use iterative DFS or `sys.setrecursionlimit(10**6)` (and still risk stack overflow in some judges). Prefer iterative for big inputs.

Time `O(V + E)`, space `O(V)`.

## 4.3 BFS vs DFS

| | BFS | DFS |
|---|---|---|
| Data structure | Queue | Stack / recursion |
| Explores | Level by level | Branch by branch |
| Shortest path (unweighted) | **Yes** | No |
| Memory | Wide graphs use more | Deep graphs use more |
| Typical | Min steps, levels | Components, cycles, topo, backtracking |

## 4.4 Disconnected graph: loop over all start nodes

```python
def traverse_all(graph):
    n = len(graph)
    visited = [False] * n
    for s in range(n):
        if not visited[s]:
            # new component starts here
            dfs(graph, s, visited)
```

---

# Part 5 — Graphs Hidden in Grids (Matrix Problems)

A 2D grid **is** a graph: each cell is a node, neighbours are up/down/left/right (sometimes 8 directions).

```python
DIRS = [(1, 0), (-1, 0), (0, 1), (0, -1)]              # 4-directional
DIRS8 = DIRS + [(1, 1), (1, -1), (-1, 1), (-1, -1)]    # 8-directional

def neighbours(r, c, R, C):
    for dr, dc in DIRS:
        nr, nc = r + dr, c + dc
        if 0 <= nr < R and 0 <= nc < C:
            yield nr, nc
```

**Flood fill (DFS on grid)** — count cells of an island, iterative:

```python
def island_size(grid, r, c):
    R, C = len(grid), len(grid[0])
    stack = [(r, c)]
    grid[r][c] = 0               # mark visited by modifying the grid
    size = 0
    while stack:
        x, y = stack.pop()
        size += 1
        for dx, dy in DIRS:
            nx, ny = x + dx, y + dy
            if 0 <= nx < R and 0 <= ny < C and grid[nx][ny] == 1:
                grid[nx][ny] = 0
                stack.append((nx, ny))
    return size
```

Trick: encode cell `(r, c)` as one number `r * C + c` when using DSU or arrays.

**Multi-source BFS**: push *all* starting cells into the queue at time 0 (rotten oranges, nearest 0 in matrix, fire spread). Covered in Problem 3.

---

# Part 6 — Connected Components and Cycle Detection

## 6.1 Count connected components (undirected)

```python
def count_components(n, edges):
    graph = [[] for _ in range(n)]
    for u, v in edges:
        graph[u].append(v)
        graph[v].append(u)
    visited = [False] * n
    count = 0
    for s in range(n):
        if visited[s]:
            continue
        count += 1
        stack = [s]
        visited[s] = True
        while stack:
            u = stack.pop()
            for v in graph[u]:
                if not visited[v]:
                    visited[v] = True
                    stack.append(v)
    return count
```

## 6.2 Cycle in an UNDIRECTED graph

Idea: during DFS, if you meet a visited node that is **not your parent**, there is a cycle.

```python
def has_cycle_undirected(n, edges):
    graph = [[] for _ in range(n)]
    for u, v in edges:
        graph[u].append(v)
        graph[v].append(u)
    visited = [False] * n
    for s in range(n):
        if visited[s]:
            continue
        visited[s] = True
        stack = [(s, -1)]                 # (node, parent)
        while stack:
            u, parent = stack.pop()
            for v in graph[u]:
                if not visited[v]:
                    visited[v] = True
                    stack.append((v, u))
                elif v != parent:
                    return True
    return False
```

(Alternative: DSU — if an edge joins two nodes already in the same set, it makes a cycle. See Part 10.)

## 6.3 Cycle in a DIRECTED graph

"Visited" is not enough. A node can be reached twice without a cycle (diamond shape). Use **3 states**:

- `0` = unvisited
- `1` = currently in the recursion stack (being explored)
- `2` = fully done

If you reach a node with state `1`, you found a **back edge → cycle**.

```python
def has_cycle_directed(n, edges):
    graph = [[] for _ in range(n)]
    for u, v in edges:
        graph[u].append(v)
    state = [0] * n

    def dfs(u):
        state[u] = 1
        for v in graph[u]:
            if state[v] == 1:
                return True
            if state[v] == 0 and dfs(v):
                return True
        state[u] = 2
        return False

    return any(state[i] == 0 and dfs(i) for i in range(n))
```

Another way: Kahn's topological sort — if you cannot process all nodes, there is a cycle (Part 8).

---

# Part 7 — Bipartite Graph Check

A graph is bipartite if you can **colour nodes with 2 colours** so every edge connects different colours. Equivalent: **no odd-length cycle**.

Method: BFS/DFS colouring. Give a neighbour the opposite colour; if a neighbour already has the same colour → not bipartite. Remember to loop over every component.

```python
def is_bipartite(graph):
    n = len(graph)
    color = [-1] * n
    for s in range(n):
        if color[s] != -1:
            continue
        color[s] = 0
        q = deque([s])
        while q:
            u = q.popleft()
            for v in graph[u]:
                if color[v] == -1:
                    color[v] = 1 - color[u]
                    q.append(v)
                elif color[v] == color[u]:
                    return False
    return True
```

Used in: "possible bipartition", "divide into two teams who don't dislike each other".

---

# Part 8 — Topological Sort (DAG)

**Topological order** = a linear ordering of nodes such that for every edge `u -> v`, `u` comes **before** `v`. Only possible in a **DAG** (no cycle).

Real life: course prerequisites, build systems, task dependencies.

## 8.1 Kahn's Algorithm (BFS + in-degree) — preferred

1. Compute in-degree of every node.
2. Put all nodes with in-degree 0 in a queue.
3. Pop a node, add to answer, reduce in-degree of its neighbours; if a neighbour becomes 0, push it.
4. If answer has fewer than `n` nodes → **cycle exists**.

```python
def topo_sort_kahn(n, edges):
    graph = [[] for _ in range(n)]
    indeg = [0] * n
    for u, v in edges:          # u -> v
        graph[u].append(v)
        indeg[v] += 1
    q = deque(i for i in range(n) if indeg[i] == 0)
    order = []
    while q:
        u = q.popleft()
        order.append(u)
        for v in graph[u]:
            indeg[v] -= 1
            if indeg[v] == 0:
                q.append(v)
    return order if len(order) == n else []     # [] means cycle
```

## 8.2 DFS-based topological sort

Do DFS; add a node to a list **after** all its descendants are done; reverse the list at the end.

```python
def topo_sort_dfs(n, edges):
    graph = [[] for _ in range(n)]
    for u, v in edges:
        graph[u].append(v)
    state = [0] * n
    out = []
    cyc = False

    def dfs(u):
        nonlocal cyc
        state[u] = 1
        for v in graph[u]:
            if state[v] == 1:
                cyc = True
                return
            if state[v] == 0:
                dfs(v)
                if cyc:
                    return
        state[u] = 2
        out.append(u)

    for i in range(n):
        if state[i] == 0:
            dfs(i)
            if cyc:
                return []
    return out[::-1]
```

Time `O(V + E)`.

Also: Kahn's order with a **min-heap** gives the lexicographically smallest topological order.

---

# Part 9 — Shortest Path Algorithms

## 9.0 Which one to use?

| Situation | Algorithm | Time |
|---|---|---|
| Unweighted | **BFS** | O(V+E) |
| Weights only 0 or 1 | **0-1 BFS** (deque) | O(V+E) |
| Non-negative weights | **Dijkstra** (heap) | O((V+E) log V) |
| Negative weights, or "at most K edges" | **Bellman-Ford** | O(V·E) |
| All pairs, small V (≤ 400) | **Floyd-Warshall** | O(V³) |
| DAG | DP in topological order | O(V+E) |

## 9.1 Dijkstra

Idea: always expand the **closest unvisited node** using a min-heap. Greedy works because weights are non-negative.

```python
import heapq

def dijkstra(graph, src):
    """graph[u] = list of (v, w). Returns list of distances."""
    n = len(graph)
    INF = float('inf')
    dist = [INF] * n
    dist[src] = 0
    heap = [(0, src)]                    # (distance, node)
    while heap:
        d, u = heapq.heappop(heap)
        if d > dist[u]:                  # stale entry, skip
            continue
        for v, w in graph[u]:
            nd = d + w
            if nd < dist[v]:
                dist[v] = nd
                heapq.heappush(heap, (nd, v))
    return dist
```

- Time `O((V + E) log V)`.
- **Fails with negative edges.**
- To get the actual path, keep a `parent[v] = u` array when you relax.

## 9.2 0-1 BFS

When edge weights are only 0 or 1: push 0-weight neighbours to the **front** of a deque, 1-weight to the **back**.

```python
def zero_one_bfs(graph, src):
    n = len(graph)
    INF = float('inf')
    dist = [INF] * n
    dist[src] = 0
    dq = deque([src])
    while dq:
        u = dq.popleft()
        for v, w in graph[u]:          # w is 0 or 1
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                if w == 0:
                    dq.appendleft(v)
                else:
                    dq.append(v)
    return dist
```

## 9.3 Bellman-Ford

Relax **every edge** `V-1` times. Handles negative weights. One more round that still improves → **negative cycle**.

```python
def bellman_ford(n, edges, src):
    """edges = [(u, v, w)]. Returns (dist, has_negative_cycle)."""
    INF = float('inf')
    dist = [INF] * n
    dist[src] = 0
    for _ in range(n - 1):
        changed = False
        for u, v, w in edges:
            if dist[u] != INF and dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                changed = True
        if not changed:
            break
    for u, v, w in edges:
        if dist[u] != INF and dist[u] + w < dist[v]:
            return dist, True
    return dist, False
```

"At most K edges": run exactly `K` rounds and relax using a **copy** of the previous round (Problem 9).

## 9.4 Floyd-Warshall (all pairs)

For every middle node `k`, try going `i -> k -> j`.

```python
def floyd_warshall(n, edges):
    INF = float('inf')
    d = [[INF] * n for _ in range(n)]
    for i in range(n):
        d[i][i] = 0
    for u, v, w in edges:
        d[u][v] = min(d[u][v], w)
    for k in range(n):                   # k MUST be the outer loop
        for i in range(n):
            for j in range(n):
                if d[i][k] + d[k][j] < d[i][j]:
                    d[i][j] = d[i][k] + d[k][j]
    return d
```

---

# Part 10 — Union-Find (DSU) and Minimum Spanning Tree

## 10.1 Disjoint Set Union

Maintains groups of nodes. Two operations:
- `find(x)` — which group leader does `x` belong to?
- `union(a, b)` — merge the two groups.

With **path compression + union by size** each op is almost `O(1)` (inverse Ackermann).

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.size = [1] * n
        self.components = n

    def find(self, x):
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]   # path compression
            x = self.parent[x]
        return x

    def union(self, a, b):
        ra, rb = self.find(a), self.find(b)
        if ra == rb:
            return False                 # already connected -> would form a cycle
        if self.size[ra] < self.size[rb]:
            ra, rb = rb, ra
        self.parent[rb] = ra
        self.size[ra] += self.size[rb]
        self.components -= 1
        return True
```

Use DSU for: number of components, cycle detection in undirected graphs, "are x and y connected?", Kruskal, dynamic connectivity (edges added over time).

## 10.2 Minimum Spanning Tree (MST)

A **spanning tree** connects all `V` nodes using exactly `V-1` edges. The **MST** is the one with the smallest total weight.

### Kruskal (sort edges + DSU)

Sort edges by weight; add an edge if it connects two different components.

```python
def kruskal(n, edges):
    """edges = [(u, v, w)]. Returns (total_weight, used_edges) or None if not connected."""
    dsu = DSU(n)
    total, used = 0, []
    for u, v, w in sorted(edges, key=lambda e: e[2]):
        if dsu.union(u, v):
            total += w
            used.append((u, v, w))
            if len(used) == n - 1:
                break
    return (total, used) if len(used) == n - 1 else None
```

Time `O(E log E)`.

### Prim (grow from one node with a heap)

```python
def prim(graph):
    """graph[u] = [(v, w)], assumed connected. Returns MST weight."""
    n = len(graph)
    seen = [False] * n
    heap = [(0, 0)]                      # (weight, node)
    total = 0
    count = 0
    while heap and count < n:
        w, u = heapq.heappop(heap)
        if seen[u]:
            continue
        seen[u] = True
        total += w
        count += 1
        for v, wt in graph[u]:
            if not seen[v]:
                heapq.heappush(heap, (wt, v))
    return total
```

Time `O(E log V)`. Kruskal is better for sparse graphs / edge lists; Prim for dense graphs.

---

# Part 11 — Advanced Topics

## 11.1 Bridges and Articulation Points (Tarjan)

- **Bridge**: an edge whose removal disconnects the graph ("critical connection").
- **Articulation point**: a node whose removal disconnects the graph.

Idea: for every node store
- `disc[u]` = time when first discovered,
- `low[u]` = the earliest discovery time reachable from `u`'s subtree using at most one back edge.

For tree edge `u -> v`: if `low[v] > disc[u]` → edge `(u, v)` is a **bridge**.
For articulation point: `low[v] >= disc[u]` (for non-root `u`), and root is one if it has ≥ 2 DFS children.

```python
import sys
sys.setrecursionlimit(10**6)

def find_bridges(n, edges):
    graph = [[] for _ in range(n)]
    for u, v in edges:
        graph[u].append(v)
        graph[v].append(u)
    disc = [-1] * n
    low = [0] * n
    timer = 0
    bridges = []

    def dfs(u, parent):
        nonlocal timer
        disc[u] = low[u] = timer
        timer += 1
        for v in graph[u]:
            if v == parent:
                continue
            if disc[v] == -1:
                dfs(v, u)
                low[u] = min(low[u], low[v])
                if low[v] > disc[u]:
                    bridges.append((u, v))
            else:
                low[u] = min(low[u], disc[v])      # back edge

    for i in range(n):
        if disc[i] == -1:
            dfs(i, -1)
    return bridges
```

## 11.2 Strongly Connected Components (Kosaraju)

In a **directed** graph, an SCC is a maximal set of nodes where every node can reach every other.

Steps:
1. DFS on the original graph; push nodes to a stack in order of **finish time**.
2. Build the **reversed** graph.
3. Pop nodes from the stack; DFS on reversed graph. Each DFS tree = one SCC.

```python
def kosaraju_scc(n, edges):
    graph = [[] for _ in range(n)]
    rgraph = [[] for _ in range(n)]
    for u, v in edges:
        graph[u].append(v)
        rgraph[v].append(u)

    visited = [False] * n
    order = []

    def dfs1(u):
        visited[u] = True
        for v in graph[u]:
            if not visited[v]:
                dfs1(v)
        order.append(u)

    for i in range(n):
        if not visited[i]:
            dfs1(i)

    visited = [False] * n
    sccs = []

    def dfs2(u, comp):
        visited[u] = True
        comp.append(u)
        for v in rgraph[u]:
            if not visited[v]:
                dfs2(v, comp)

    for u in reversed(order):
        if not visited[u]:
            comp = []
            dfs2(u, comp)
            sccs.append(comp)
    return sccs
```

Condensing SCCs gives a DAG — useful in many hard problems.

## 11.3 BFS on "states" (implicit graphs)

Sometimes nodes are not given — a node is a **state** (a word, a lock combination, `(position, keys held)`, a puzzle board string). Edges = one legal move. BFS over states gives the minimum number of moves.
Examples: Word Ladder, Open the Lock, Sliding Puzzle, Minimum Genetic Mutation.

## 11.4 Eulerian path (just know the condition)

- Undirected: Euler circuit exists if all degrees are even and graph is connected; Euler path if exactly 0 or 2 odd-degree nodes.
- Directed: in-degree = out-degree for all nodes (circuit). Algorithm: Hierholzer. Problem: "Reconstruct Itinerary".

## 11.5 Dijkstra variants you will meet

- **Minimise the maximum edge on a path** (Path With Minimum Effort, Swim in Rising Water): same Dijkstra, but the "distance" is `max(d, edge)` instead of `d + edge` (Problem 15).
- **Count shortest paths**: keep `ways[v]`; when `nd == dist[v]` add `ways[u]`.
- **Dijkstra with extra state**: `(cost, node, stops_left)` for flights with limits.

---

# Part 12 — Pattern Recognition Cheat Sheet

| If the problem says... | Think... |
|---|---|
| "minimum steps / moves / hops", unweighted | **BFS** |
| Many sources spreading simultaneously | **Multi-source BFS** |
| "number of islands / regions / groups / provinces" | **DFS / BFS / DSU** |
| "is there a path from A to B" | DFS/BFS/DSU |
| prerequisites, ordering, dependencies | **Topological sort** |
| "can't finish / deadlock / cycle" | Cycle detection (3-state DFS or Kahn) |
| Two groups, conflict pairs, "dislike" | **Bipartite** colouring |
| Shortest path, positive weights | **Dijkstra** |
| Shortest path, at most K stops/edges, negative edges | **Bellman-Ford** |
| Min cost to connect everything | **MST** (Kruskal/Prim) |
| Add edges one by one, check connectivity / redundant edge | **DSU** |
| Critical link / bridge / single point of failure | **Tarjan low-link** |
| Mutual reachability in directed graph | **SCC** |
| Smallest *maximum* edge on path | Dijkstra with `max`, or binary search + BFS, or DSU |
| Words / strings as nodes, one change per step | BFS on states |
| Ordering derived from sorted data (alien dictionary) | Build graph, then topo sort |

**Complexity summary**

| Algorithm | Time | Space |
|---|---|---|
| BFS / DFS | O(V+E) | O(V) |
| Topological sort | O(V+E) | O(V) |
| Dijkstra | O((V+E) log V) | O(V) |
| Bellman-Ford | O(VE) | O(V) |
| Floyd-Warshall | O(V³) | O(V²) |
| Kruskal | O(E log E) | O(V) |
| Prim | O(E log V) | O(V) |
| DSU op | ~O(1) | O(V) |
| Tarjan bridges / Kosaraju | O(V+E) | O(V) |

---

# Part 13 — 15 Standard Interview Problems

For each: **Problem → Idea → Code → Complexity**. All functions below were run against test cases.
(LeetCode numbers given so you can find them. Some solutions reuse the `DSU` class from Part 10 and `deque`, `heapq`, `defaultdict` imports.)

```python
from collections import deque, defaultdict
import heapq
```

---

## Problem 1 — Number of Islands (LC 200) · Easy/Medium · Grid DFS/BFS

**Problem:** Grid of `'1'` (land) and `'0'` (water). Count islands (groups of connected land, 4-directional).

**Idea:** Scan each cell. When you find an unvisited `'1'`, count one island and flood-fill (DFS) to mark the entire island as visited.

```python
def num_islands(grid):
    if not grid:
        return 0
    R, C = len(grid), len(grid[0])
    count = 0
    for i in range(R):
        for j in range(C):
            if grid[i][j] == '1':
                count += 1
                grid[i][j] = '0'
                stack = [(i, j)]
                while stack:
                    r, c = stack.pop()
                    for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
                        nr, nc = r + dr, c + dc
                        if 0 <= nr < R and 0 <= nc < C and grid[nr][nc] == '1':
                            grid[nr][nc] = '0'
                            stack.append((nr, nc))
    return count
```

**Complexity:** Time `O(R·C)`, Space `O(R·C)` worst case (stack).
**Variants:** Max Area of Island (LC 695), Surrounded Regions (LC 130), Number of Enclaves (LC 1020).

---

## Problem 2 — Clone Graph (LC 133) · Medium · BFS/DFS + hashmap

**Problem:** Deep copy a connected undirected graph given a reference to one node.

**Idea:** Use a dict `old -> new`. BFS from the start node; for each neighbour, create its clone if it does not exist yet, then connect clones.

```python
class Node:
    def __init__(self, val=0, neighbors=None):
        self.val = val
        self.neighbors = neighbors if neighbors is not None else []

def clone_graph(node):
    if not node:
        return None
    copy = {node: Node(node.val)}
    q = deque([node])
    while q:
        cur = q.popleft()
        for nb in cur.neighbors:
            if nb not in copy:
                copy[nb] = Node(nb.val)
                q.append(nb)
            copy[cur].neighbors.append(copy[nb])
    return copy[node]
```

**Complexity:** Time `O(V+E)`, Space `O(V)`.
**Key trick:** the `copy` dict doubles as the `visited` set.

---

## Problem 3 — Rotting Oranges (LC 994) · Medium · Multi-source BFS

**Problem:** Grid: `0` empty, `1` fresh, `2` rotten. Each minute a rotten orange rots its 4 neighbours. Return minutes until no fresh orange is left, or `-1` if impossible.

**Idea:** Push **all** rotten oranges into the queue first (time 0). BFS level by level; each level = one minute. Count fresh oranges; if any remain, return -1.

```python
def oranges_rotting(grid):
    R, C = len(grid), len(grid[0])
    q = deque()
    fresh = 0
    for i in range(R):
        for j in range(C):
            if grid[i][j] == 2:
                q.append((i, j))
            elif grid[i][j] == 1:
                fresh += 1
    minutes = 0
    while q and fresh:
        for _ in range(len(q)):              # process one full minute
            r, c = q.popleft()
            for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
                nr, nc = r + dr, c + dc
                if 0 <= nr < R and 0 <= nc < C and grid[nr][nc] == 1:
                    grid[nr][nc] = 2
                    fresh -= 1
                    q.append((nr, nc))
        minutes += 1
    return minutes if fresh == 0 else -1
```

**Complexity:** Time `O(R·C)`, Space `O(R·C)`.
**Variants:** 01 Matrix (LC 542), Walls and Gates (LC 286), Map of Highest Peak.

---

## Problem 4 — Course Schedule II (LC 210) · Medium · Topological Sort

**Problem:** `numCourses` courses; `prerequisites[i] = [a, b]` means you must take `b` before `a`. Return a valid order, or `[]` if impossible. (Course Schedule I, LC 207, just asks `len(order) == n`.)

**Idea:** Edge `b -> a`. Run Kahn's algorithm. If not all courses are output, there is a cycle.

```python
def find_order(num_courses, prerequisites):
    graph = [[] for _ in range(num_courses)]
    indeg = [0] * num_courses
    for a, b in prerequisites:
        graph[b].append(a)
        indeg[a] += 1
    q = deque(i for i in range(num_courses) if indeg[i] == 0)
    order = []
    while q:
        u = q.popleft()
        order.append(u)
        for v in graph[u]:
            indeg[v] -= 1
            if indeg[v] == 0:
                q.append(v)
    return order if len(order) == num_courses else []
```

**Complexity:** Time `O(V+E)`, Space `O(V+E)`.
**Variants:** Course Schedule I (207), Parallel Courses (LC 1136), Find All Possible Recipes, Minimum Height Trees (LC 310, peel leaves like Kahn).

---

## Problem 5 — Is Graph Bipartite? (LC 785) · Medium · BFS colouring

**Problem:** Given an undirected graph as adjacency list, can nodes be split into two sets with every edge crossing between the sets?

**Idea:** 2-colour with BFS; same colour on an edge → no. Loop over all components.

```python
def is_bipartite_graph(graph):
    n = len(graph)
    color = [-1] * n
    for s in range(n):
        if color[s] != -1:
            continue
        color[s] = 0
        q = deque([s])
        while q:
            u = q.popleft()
            for v in graph[u]:
                if color[v] == -1:
                    color[v] = color[u] ^ 1
                    q.append(v)
                elif color[v] == color[u]:
                    return False
    return True
```

**Complexity:** Time `O(V+E)`, Space `O(V)`.
**Variants:** Possible Bipartition (LC 886) — build graph from "dislike" pairs then same code.

---

## Problem 6 — Pacific Atlantic Water Flow (LC 417) · Medium · Reverse multi-source BFS

**Problem:** `heights` grid. Water flows from a cell to a neighbour with height ≤ current. Return all cells from which water can reach **both** the Pacific (top/left edges) and Atlantic (bottom/right edges).

**Idea:** Instead of starting from every cell, go **backwards**: start from ocean-border cells and move to neighbours with height **≥** current (uphill). Find cells reachable from Pacific and from Atlantic; answer = intersection.

```python
def pacific_atlantic(heights):
    R, C = len(heights), len(heights[0])

    def bfs(starts):
        seen = set(starts)
        q = deque(starts)
        while q:
            r, c = q.popleft()
            for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
                nr, nc = r + dr, c + dc
                if (0 <= nr < R and 0 <= nc < C and (nr, nc) not in seen
                        and heights[nr][nc] >= heights[r][c]):
                    seen.add((nr, nc))
                    q.append((nr, nc))
        return seen

    pac = [(0, c) for c in range(C)] + [(r, 0) for r in range(1, R)]
    atl = [(R - 1, c) for c in range(C)] + [(r, C - 1) for r in range(R - 1)]
    return [list(p) for p in sorted(bfs(pac) & bfs(atl))]
```

**Complexity:** Time `O(R·C)`, Space `O(R·C)`.
**Lesson:** "Reverse the direction" is a powerful graph trick.

---

## Problem 7 — Word Ladder (LC 127) · Hard-ish · BFS on states

**Problem:** Transform `beginWord` to `endWord` changing one letter at a time; every intermediate word must be in `wordList`. Return the number of words in the shortest sequence (0 if impossible).

**Idea:** Words are nodes; two words are adjacent if they differ by one letter. Comparing all pairs is slow, so use **wildcard patterns**: `hot -> *ot, h*t, ho*`. Words sharing a pattern are neighbours. BFS gives the shortest path.

```python
def ladder_length(begin_word, end_word, word_list):
    words = set(word_list)
    if end_word not in words:
        return 0
    buckets = defaultdict(list)
    for w in words | {begin_word}:
        for i in range(len(w)):
            buckets[w[:i] + '*' + w[i + 1:]].append(w)
    q = deque([(begin_word, 1)])
    seen = {begin_word}
    while q:
        w, d = q.popleft()
        if w == end_word:
            return d
        for i in range(len(w)):
            key = w[:i] + '*' + w[i + 1:]
            for nxt in buckets[key]:
                if nxt not in seen:
                    seen.add(nxt)
                    q.append((nxt, d + 1))
            buckets[key] = []              # each bucket needs to be used only once
    return 0
```

**Complexity:** Time `O(N · L²)` (N words, length L), Space `O(N · L²)`.
**Variants:** Open the Lock (LC 752), Minimum Genetic Mutation (LC 433), Word Ladder II (hard: BFS + path rebuild).

---

## Problem 8 — Network Delay Time (LC 743) · Medium · Dijkstra

**Problem:** Directed weighted graph `times[i] = (u, v, w)`, nodes `1..n`. Signal starts at `k`. Return the time for all nodes to receive it, or `-1`.

**Idea:** Dijkstra from `k`. Answer = max of all distances (if any is infinite → -1).

```python
def network_delay_time(times, n, k):
    graph = [[] for _ in range(n + 1)]
    for u, v, w in times:
        graph[u].append((v, w))
    dist = [float('inf')] * (n + 1)
    dist[k] = 0
    heap = [(0, k)]
    while heap:
        d, u = heapq.heappop(heap)
        if d > dist[u]:
            continue
        for v, w in graph[u]:
            if d + w < dist[v]:
                dist[v] = d + w
                heapq.heappush(heap, (d + w, v))
    ans = max(dist[1:])
    return -1 if ans == float('inf') else ans
```

**Complexity:** Time `O(E log V)`, Space `O(V+E)`.
**Variants:** Path with Maximum Probability (LC 1514, max-heap with probabilities), Number of Ways to Arrive at Destination (LC 1976, count shortest paths), Shortest Path in Binary Matrix (LC 1091, BFS 8-dir).

---

## Problem 9 — Cheapest Flights Within K Stops (LC 787) · Medium · Bellman-Ford (K rounds)

**Problem:** Flights `(from, to, price)`. Cheapest price from `src` to `dst` with at most `k` stops (so at most `k+1` flights). `-1` if impossible.

**Idea:** Plain Dijkstra breaks because the cheapest path may use too many stops. Run **Bellman-Ford for exactly `k+1` rounds**, and in every round relax using a **copy** of the previous round's distances (so each round extends paths by at most one edge).

```python
def find_cheapest_price(n, flights, src, dst, k):
    INF = float('inf')
    dist = [INF] * n
    dist[src] = 0
    for _ in range(k + 1):
        nxt = dist[:]
        for u, v, w in flights:
            if dist[u] != INF and dist[u] + w < nxt[v]:
                nxt[v] = dist[u] + w
        dist = nxt
    return -1 if dist[dst] == INF else dist[dst]
```

**Complexity:** Time `O(K · E)`, Space `O(V)`.
**Common bug:** using the same array for read and write lets a path use more than one new edge per round.

---

## Problem 10 — Number of Provinces (LC 547) · Medium · DSU

**Problem:** `isConnected[i][j] = 1` means city `i` and `j` are directly connected. A province is a group of directly/indirectly connected cities. Count provinces.

**Idea:** DSU. Union every connected pair; answer = number of components left.

```python
def find_circle_num(is_connected):
    n = len(is_connected)
    dsu = DSU(n)
    for i in range(n):
        for j in range(i + 1, n):
            if is_connected[i][j]:
                dsu.union(i, j)
    return dsu.components
```

**Complexity:** Time `O(n² · α(n))`, Space `O(n)`.
**Variants:** Number of Connected Components (LC 323), Accounts Merge (LC 721), Most Stones Removed (LC 947), Number of Islands II (LC 305, online DSU).

---

## Problem 11 — Redundant Connection (LC 684) · Medium · DSU cycle detection

**Problem:** A tree with `n` nodes got one extra edge, so it now has one cycle. Return the edge that can be removed (if many, the one appearing **last** in input).

**Idea:** Process edges in order with DSU. The first edge whose endpoints are already in the same set closes the cycle → that is the answer.

```python
def find_redundant_connection(edges):
    dsu = DSU(len(edges) + 1)           # nodes are 1-indexed
    for u, v in edges:
        if not dsu.union(u, v):
            return [u, v]
    return []
```

**Complexity:** Time `O(E · α)`, Space `O(V)`.
**Variants:** Redundant Connection II (LC 685, directed version — hard), Graph Valid Tree (LC 261: `E == n-1` and no cycle), Number of Operations to Make Network Connected (LC 1319).

---

## Problem 12 — Min Cost to Connect All Points (LC 1584) · Medium · MST

**Problem:** Points on a 2D plane; cost between two points = Manhattan distance. Minimum total cost to connect all points.

**Idea:** Complete graph + MST. With `n ≤ 1000`, use **Prim in O(n²)** (no heap, no edge list needed — dense graph).

```python
def min_cost_connect_points(points):
    n = len(points)
    INF = float('inf')
    min_dist = [INF] * n               # cheapest edge connecting node to the tree
    in_tree = [False] * n
    min_dist[0] = 0
    total = 0
    for _ in range(n):
        u = -1
        for i in range(n):
            if not in_tree[i] and (u == -1 or min_dist[i] < min_dist[u]):
                u = i
        in_tree[u] = True
        total += min_dist[u]
        for v in range(n):
            if not in_tree[v]:
                d = abs(points[u][0] - points[v][0]) + abs(points[u][1] - points[v][1])
                if d < min_dist[v]:
                    min_dist[v] = d
    return total
```

**Complexity:** Time `O(n²)`, Space `O(n)`.
**Variants:** Connecting Cities With Minimum Cost (LC 1135), Optimize Water Distribution (LC 1168, add a dummy node), Find Critical and Pseudo-Critical Edges in MST (LC 1489, hard).

---

## Problem 13 — Critical Connections in a Network (LC 1192) · Hard · Tarjan bridges

**Problem:** `n` servers, undirected `connections`. Return all connections that are **critical** (removing it disconnects some servers).

**Idea:** Exactly the bridge-finding algorithm from Part 11.1: `disc`, `low`, and the condition `low[v] > disc[u]`.

```python
import sys
sys.setrecursionlimit(10**6)

def critical_connections(n, connections):
    graph = [[] for _ in range(n)]
    for u, v in connections:
        graph[u].append(v)
        graph[v].append(u)
    disc = [-1] * n
    low = [0] * n
    timer = 0
    res = []

    def dfs(u, parent):
        nonlocal timer
        disc[u] = low[u] = timer
        timer += 1
        for v in graph[u]:
            if v == parent:
                continue
            if disc[v] == -1:
                dfs(v, u)
                low[u] = min(low[u], low[v])
                if low[v] > disc[u]:
                    res.append([u, v])
            else:
                low[u] = min(low[u], disc[v])

    dfs(0, -1)
    return res
```

**Complexity:** Time `O(V+E)`, Space `O(V+E)`.
**Interview tip:** if asked to explain `low`: "the smallest discovery time reachable from this subtree using one back edge. If a child's low is still greater than my discovery time, nothing in its subtree can bypass me, so edge (u, child) is a bridge."

---

## Problem 14 — Alien Dictionary (LC 269) · Hard · Build graph + Topological sort

**Problem:** Given words sorted in an alien language, return a valid order of the letters (`""` if invalid).

**Idea:**
1. Every letter that appears is a node.
2. Compare **adjacent** words; the first position where they differ gives an edge `a -> b` (a comes before b).
3. Edge case: if a longer word comes before its own prefix (`"abc"` before `"ab"`) → invalid.
4. Topological sort (Kahn). If a cycle → `""`.

```python
def alien_order(words):
    graph = {c: set() for w in words for c in w}
    indeg = {c: 0 for c in graph}
    for w1, w2 in zip(words, words[1:]):
        if len(w1) > len(w2) and w1.startswith(w2):
            return ""
        for a, b in zip(w1, w2):
            if a != b:
                if b not in graph[a]:
                    graph[a].add(b)
                    indeg[b] += 1
                break
    q = deque(c for c in indeg if indeg[c] == 0)
    out = []
    while q:
        c = q.popleft()
        out.append(c)
        for nxt in graph[c]:
            indeg[nxt] -= 1
            if indeg[nxt] == 0:
                q.append(nxt)
    return "".join(out) if len(out) == len(graph) else ""
```

**Complexity:** Time `O(total characters)`, Space `O(1)` for 26 letters (or `O(U + min(U², N))`).
**Variants:** Sequence Reconstruction (LC 444), Verifying an Alien Dictionary (LC 953, easy).

---

## Problem 15 — Path With Minimum Effort (LC 1631) · Medium · Dijkstra on grid with `max`

**Problem:** Grid of heights. Move 4-directionally from top-left to bottom-right. Effort of a path = **maximum absolute height difference** between consecutive cells. Minimise the effort.

**Idea:** Dijkstra where distance of a cell = the minimum possible "max edge so far". Relaxation: `new_effort = max(current_effort, |h[next] - h[cur]|)`.

```python
def minimum_effort_path(heights):
    R, C = len(heights), len(heights[0])
    INF = float('inf')
    effort = [[INF] * C for _ in range(R)]
    effort[0][0] = 0
    heap = [(0, 0, 0)]                     # (effort, r, c)
    while heap:
        e, r, c = heapq.heappop(heap)
        if (r, c) == (R - 1, C - 1):
            return e
        if e > effort[r][c]:
            continue
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < R and 0 <= nc < C:
                ne = max(e, abs(heights[nr][nc] - heights[r][c]))
                if ne < effort[nr][nc]:
                    effort[nr][nc] = ne
                    heapq.heappush(heap, (ne, nr, nc))
    return 0
```

**Complexity:** Time `O(R·C · log(R·C))`, Space `O(R·C)`.
**Variants:** Swim in Rising Water (LC 778), Minimum Cost to Make at Least One Valid Path in Grid (LC 1368, 0-1 BFS). Alternative solutions: binary search on answer + BFS, or sort edges + DSU.

---

# Part 14 — Extra Practice, Mistakes, Checklist

## 14.1 Extra problems (do these after the 15)

| Level | Problem |
|---|---|
| Easy | Find if Path Exists in Graph (1971), Find the Town Judge (997), Flood Fill (733), Find Center of Star Graph (1791), Verifying an Alien Dictionary (953) |
| Medium | Surrounded Regions (130), Walls and Gates (286), Evaluate Division (399), Reconstruct Itinerary (332), Is Graph Bipartite (785), Find Eventual Safe States (802), Accounts Merge (721), Minimum Height Trees (310), Shortest Bridge (934), Number of Enclaves (1020), Open the Lock (752), Snakes and Ladders (909), Keys and Rooms (841), Course Schedule (207), Possible Bipartition (886) |
| Hard | Word Ladder II (126), Shortest Path Visiting All Nodes (847, BFS + bitmask), Swim in Rising Water (778), Making A Large Island (827), Redundant Connection II (685), Number of Islands II (305), Bus Routes (815), Longest Increasing Path in a Matrix (329, DFS + memo) |

## 14.2 Common mistakes

1. Forgetting `visited` → infinite loop.
2. Marking visited when popping (BFS) instead of when pushing → TLE.
3. Forgetting disconnected components — always loop over all start nodes.
4. Undirected graph: adding the edge only in one direction.
5. Directed cycle detection using plain `visited` (use 3 states or Kahn).
6. Using Dijkstra with negative weights.
7. Wrong grid bounds (`R` vs `C` swapped) or forgetting `0 <= nr < R`.
8. Recursion depth in Python — use iterative DFS for large `n`.
9. 1-indexed nodes with 0-indexed arrays (allocate `n+1`).
10. Floyd-Warshall: `k` must be the **outermost** loop.
11. Dijkstra: forgetting the `if d > dist[u]: continue` check (not wrong, but slower).
12. Bellman-Ford: relaxing from a node whose distance is still `INF` — always check `dist[u] != INF` first.

## 14.3 Interview approach (say this out loud)

1. **Clarify:** directed? weighted? negative weights? cycles? disconnected? duplicates? size of `n`?
2. **Model:** "nodes are ___, edges are ___."
3. **Choose:** use the cheat sheet in Part 12.
4. State **time and space complexity** before coding.
5. Code with clean names, handle edge cases (empty graph, single node, no path).
6. Dry-run a small example.

## 14.4 7-day revision plan (before 12 November)

| Day | Focus |
|---|---|
| 1 | Parts 1–5: representation, BFS, DFS, grids. Problems 1, 2, 3 |
| 2 | Parts 6–7: components, cycles, bipartite. Problems 5, 6 + 3 extra grid problems |
| 3 | Part 8: topological sort. Problems 4, 14 + Course Schedule, Minimum Height Trees |
| 4 | Part 9: Dijkstra, Bellman-Ford. Problems 8, 9, 15 |
| 5 | Part 10: DSU + MST. Problems 10, 11, 12 + Accounts Merge |
| 6 | Part 11: Tarjan, SCC. Problems 7, 13 |
| 7 | Mock: solve 3 random problems from the 15 in 90 minutes, no notes |

Good luck. Graph problems look scary, but they are almost always one of: **BFS, DFS, Topo sort, Dijkstra, DSU, MST**. Learn to recognise the pattern and the code writes itself.
