# 16 · Graphs: BFS/DFS, Topo, Shortest Paths
> **Reach for it when:** the question is reachability, distance, ordering, or spread — model nodes and edges, then pick the traversal that matches the question.

## 🧠 Mental model
Three design decisions define any graph solution: what is a node (a cell, a word, a route, a board square), what is an edge (explicit list, 4-neighbors, one-letter mutation), and what is the question. Reachability or region size → DFS/BFS. Shortest path unweighted → BFS, because first arrival is optimal. Weighted and non-negative → Dijkstra: pop order is nondecreasing distance, so skip stale heap entries lazily. Dependencies or ordering → Kahn's peel: repeatedly take nodes with indegree 0; leftovers mean a cycle. Spread from many sources at once → multi-source BFS seeding the whole frontier at distance 0. Constraint like "at most k edges" → Bellman-Ford-style relax rounds. Grids are graphs with implicit edges; use the grid itself as the visited set to stay O(1) memory. Modeling failure, not traversal code, is what actually fails interviews here.

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "islands / flood fill / region size" | grid DFS with in-place visited |
| "shortest transformation / min steps" | BFS on implicit graph, mark on enqueue |
| "order with prerequisites / can you finish" | Kahn's topo; leftover nodes = cycle |
| "distance to nearest X / spreading from sources" | multi-source BFS (rotting oranges, 01-matrix) |
| "min time / cost with weights ≥ 0" | heapq Dijkstra, lazy-skip stale pops |
| "minimize the MAX edge on a path" | Dijkstra with max-aggregation, or binary search + BFS |
| "cheapest path with at most k stops" | Bellman-Ford rounds, two arrays |
| "can you visit everything / board hops / bus routes" | DFS/BFS counter; BFS levels = moves taken |
| "longest path in a DAG-like grid" | memoized DFS over cells (LIS in a matrix) |

## 🛠️ Templates
### T1 — Grid DFS/BFS with in-place visited
Islands, flood fill, regions — mark the grid itself.

```python
def max_area_of_island(grid):
    m, n = len(grid), len(grid[0])

    def dfs(r, c):
        if not (0 <= r < m and 0 <= c < n) or grid[r][c] != 1:
            return 0                        # out of bounds or water or visited
        grid[r][c] = 0                      # in-place visited mark: O(1) memory
        return 1 + dfs(r + 1, c) + dfs(r - 1, c) + dfs(r, c + 1) + dfs(r, c - 1)

    return max((dfs(r, c) for r in range(m) for c in range(n)), default=0)
```
O(m·n) time, O(m·n) recursion worst case (swap in an explicit stack for huge grids).

### T2 — Adjacency build + BFS shortest-in-unweighted
Word ladder: implicit edges = one-letter mutations.

```python
from collections import deque

def ladder_length(begin, end, word_list):
    words = set(word_list)
    if end not in words:
        return 0
    q, seen = deque([(begin, 1)]), {begin}
    while q:
        word, steps = q.popleft()
        if word == end:
            return steps                 # BFS: first arrival IS shortest
        for i in range(len(word)):       # generate neighbors implicitly
            for ch in "abcdefghijklmnopqrstuvwxyz":
                nxt = word[:i] + ch + word[i + 1:]
                if nxt in words and nxt not in seen:
                    seen.add(nxt)        # mark on ENQUEUE, never on pop
                    q.append((nxt, steps + 1))
    return 0
```
O(26·L·N) time, O(N·L) space — or pre-bucket words by "h*t" patterns to drop the 26.

### T3 — Kahn's topo peel
Course order; leftover nodes expose the cycle.

```python
from collections import deque, defaultdict

def find_order(num_courses, prerequisites):
    adj, indeg = defaultdict(list), [0] * num_courses
    for course, pre in prerequisites:
        adj[pre].append(course)           # edge pre -> course
        indeg[course] += 1
    q = deque(i for i in range(num_courses) if indeg[i] == 0)  # ready now
    order = []
    while q:
        c = q.popleft()
        order.append(c)
        for nxt in adj[c]:
            indeg[nxt] -= 1              # peel one incoming edge
            if indeg[nxt] == 0:          # all prerequisites satisfied
                q.append(nxt)
    return order if len(order) == num_courses else []  # short = cycle
```
O(V + E) time, O(V + E) space; minimum-height-trees = peel entire outer layers inward.

### T4 — Multi-source BFS
Rotting oranges / 01-matrix: seed every source at distance 0.

```python
from collections import deque

def update_matrix(mat):                  # distance to nearest 0 for each cell
    m, n = len(mat), len(mat[0])
    dist = [[-1] * n for _ in range(m)]  # -1 doubles as "not visited"
    q = deque()
    for r in range(m):
        for c in range(n):
            if mat[r][c] == 0:           # ALL sources start simultaneously
                dist[r][c] = 0
                q.append((r, c))
    while q:
        r, c = q.popleft()
        for dr, dc in ((1,0),(-1,0),(0,1),(0,-1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < m and 0 <= nc < n and dist[nr][nc] == -1:
                dist[nr][nc] = dist[r][c] + 1   # first reach = nearest source
                q.append((nr, nc))
    return dist
```
O(m·n) time and space — one BFS total, not one per cell.

### T5 — heapq Dijkstra with lazy skip
Non-negative weights, min total cost.

```python
import heapq
from collections import defaultdict

def network_delay_time(times, n, k):
    adj = defaultdict(list)
    for u, v, w in times:
        adj[u].append((v, w))
    dist, heap = {}, [(0, k)]
    while heap:
        d, node = heapq.heappop(heap)    # pops in nondecreasing d order
        if node in dist:
            continue                     # LAZY skip: stale, already finalized
        dist[node] = d                   # first pop of a node = final distance
        for nb, w in adj[node]:
            if nb not in dist:
                heapq.heappush(heap, (d + w, nb))
    return max(dist.values()) if len(dist) == n else -1
```
O(E log E) time, O(V + E) space; path-with-minimum-effort swaps `d + w` for `max(d, w)`.

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| Grid DFS/BFS | O(m·n) | O(m·n) worst |
| Adjacency BFS (V nodes, E edges) | O(V + E) | O(V + E) |
| Kahn's topo | O(V + E) | O(V + E) |
| Multi-source BFS | O(V + E) | O(V + E) |
| Dijkstra (lazy) | O(E log E) | O(V + E) |
| Bellman-Ford k rounds | O(k·E) | O(V) |
| Word ladder (implicit) | O(26·L·N) | O(N·L) |
| Memoized DFS (LIS in matrix) | O(m·n) | O(m·n) |

## ⚠️ Traps interviewers probe
- Mark visited on enqueue, not on pop — otherwise the same node enters the queue many times and BFS blows up.
- Python recursion depth ≈ 1000: a 300×300 all-land grid overflows the stack; switch to an iterative stack or BFS.
- Dijkstra assumes non-negative weights; with negatives (or a k-stop cap) use Bellman-Ford rounds, and relax from a COPY of last round's distances so longer paths don't leak into earlier stops.
- Pacific-atlantic and surrounded-regions invert the flow: search inward from the borders/oceans rather than outward from every cell — same code, one-third the work.
- Snakes-and-ladders and bus-routes are modeling traps: nodes are board cells or whole routes, with one move = one edge; level count in BFS = number of dice throws / buses taken.
- Clone-graph needs the visited map keyed on ORIGINAL node, storing its clone, set before recursing into neighbors.
- Longest-increasing-path memoizes on (r, c) with 4-directional DFS; strictly increasing cells guarantee acyclicity, which is why plain DFS terminates.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Flood Fill | Easy | https://leetcode.com/problems/flood-fill/ |
| Number of Islands | Medium | https://leetcode.com/problems/number-of-islands/ |
| Max Area of Island | Medium | https://leetcode.com/problems/max-area-of-island/ |
| Clone Graph | Medium | https://leetcode.com/problems/clone-graph/ |
| Surrounded Regions | Medium | https://leetcode.com/problems/surrounded-regions/ |
| Pacific Atlantic Water Flow | Medium | https://leetcode.com/problems/pacific-atlantic-water-flow/ |
| Course Schedule | Medium | https://leetcode.com/problems/course-schedule/ |
| Course Schedule II | Medium | https://leetcode.com/problems/course-schedule-ii/ |
| Minimum Height Trees | Medium | https://leetcode.com/problems/minimum-height-trees/ |
| Rotting Oranges | Medium | https://leetcode.com/problems/rotting-oranges/ |
| 01 Matrix | Medium | https://leetcode.com/problems/01-matrix/ |
| Network Delay Time | Medium | https://leetcode.com/problems/network-delay-time/ |
| Path With Minimum Effort | Medium | https://leetcode.com/problems/path-with-minimum-effort/ |
| Cheapest Flights Within K Stops | Medium | https://leetcode.com/problems/cheapest-flights-within-k-stops/ |
| Keys and Rooms | Medium | https://leetcode.com/problems/keys-and-rooms/ |
| Snakes and Ladders | Medium | https://leetcode.com/problems/snakes-and-ladders/ |
| Word Ladder | Hard | https://leetcode.com/problems/word-ladder/ |
| Bus Routes | Hard | https://leetcode.com/problems/bus-routes/ |
| Longest Increasing Path in a Matrix | Hard | https://leetcode.com/problems/longest-increasing-path-in-a-matrix/ |
