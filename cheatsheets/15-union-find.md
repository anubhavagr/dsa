# 15 · Union-Find
> **Reach for it when:** items merge into groups over time and you keep asking "are these two in the same group?"

## 🧠 Mental model
Union-find maintains a partition under two operations: `find(x)` (who represents x's component?) and `union(a, b)` (merge two components). Done naively with parent pointers it degrades to linked-list chains; done with path compression (re-point every visited node toward the root while walking) plus union by rank (attach the shorter tree under the taller), both operations run in inverse-Ackermann α(n) time — effectively constant, since α of anything smaller than the observable universe is ≤ 4. The power move is mapping non-integer keys (emails, strings, grid cells) to integer ids first, then unioning relations, then grouping everything by `find(root)`. Streaming edges makes UF a cycle detector — the first edge whose endpoints already share a root closes a loop — and a live component counter — the same class runs Kruskal's MST by sorting edges and skipping the ones whose union fails.

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "connected components / provinces / friend circles" | UF, count components |
| "are a and b in the same group right now?" | find(a) == find(b) — the online query |
| "redundant edge — where does the cycle close?" | first union that returns False |
| "merge accounts / records sharing any key" | key→id map, union the ids |
| "longest consecutive run of values" | UF over values — or set + streak scan |
| "equations with ratios on unions" | weighted UF (multiplier to parent) |
| "grid regions / holes via border" | UF plus a virtual outside node |
| "Kruskal MST edges one by one" | sort edges, union, skip cycle edges |

## 🛠️ Templates
### T1 — UF class (path compression + union by rank)
The one class to reproduce from memory.

```python
class UF:
    def __init__(self, n):
        self.parent = list(range(n))    # parent[i] points up the tree
        self.rank = [0] * n             # bound on subtree height
        self.count = n                  # live component count

    def find(self, x):
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]   # path halving:
            x = self.parent[x]                             # point at grandparent
        return x

    def union(self, a, b):
        ra, rb = self.find(a), self.find(b)   # ALWAYS compare roots, not nodes
        if ra == rb:
            return False                      # already connected (cycle signal)
        if self.rank[ra] < self.rank[rb]:
            ra, rb = rb, ra                   # attach shorter under taller
        self.parent[rb] = ra
        if self.rank[ra] == self.rank[rb]:
            self.rank[ra] += 1
        self.count -= 1
        return True
```
Near O(α(n)) ≈ O(1) amortized per op with both optimizations — say that phrase.

### T2 — Count-components driver
Provinces: matrix in, component count out.

```python
def find_circle_num(is_connected):
    n = len(is_connected)
    uf = UF(n)
    for i in range(n):
        for j in range(i + 1, n):      # symmetric matrix: upper triangle suffices
            if is_connected[i][j]:
                uf.union(i, j)
    return uf.count                    # maintained by union — no final scan
```
O(n² α(n)) time, O(n) space.

### T3 — Cycle detection for a redundant edge
First edge that reconnects its own component is the answer.

```python
def find_redundant_connection(edges):
    uf = UF(len(edges) + 1)            # nodes are 1-indexed: allocate n+1
    for a, b in edges:
        if not uf.union(a, b):         # endpoints already share a root:
            return [a, b]              # THIS edge closes the cycle
    return []
```
O(E α(n)) time, O(n) space.

### T4 — Email→id mapping for accounts merge
Strings can't index an array — map keys to ids first.

```python
from collections import defaultdict

def accounts_merge(accounts):
    owner = {}                          # email -> first account id seen
    uf = UF(len(accounts))
    for i, acc in enumerate(accounts):
        for email in acc[1:]:           # acc[0] is the name
            if email in owner:
                uf.union(i, owner[email])   # shared email = same person
            else:
                owner[email] = i
    groups = defaultdict(list)              # root id -> its emails
    for email, i in owner.items():
        groups[uf.find(i)].append(email)
    return [[accounts[root][0]] + sorted(emails) for root, emails in groups.items()]
```
O(N log N) dominated by the final sorts; N = total emails.

### T5 — Weighted variant (pointer)
For evaluate-division, keep the exact skeleton of T1 but store one extra field: `weight[x]` = the value of x measured in units of its representative. `find` multiplies weights along the compressed path; `union(a, b, w)` solves for the multiplier that the new parent link must carry so that a/b = w stays true. Everything else — rank, compression, counting — is unchanged; only find and union do one extra multiplication/division each.

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| find / union (compression + rank) | O(α(n)) ≈ O(1) amortized | — |
| UF array storage | — | O(n) |
| Count components over matrix | O(n² α(n)) | O(n) |
| Redundant edge over E edges | O(E α(n)) | O(n) |
| Accounts merge (N emails) | O(N log N) | O(N) |
| Set-based longest-consecutive | O(n) | O(n) |
| Online connectivity query find(a)==find(b) | O(α(n)) | O(1) |

## ⚠️ Traps interviewers probe
- Parent pointers alone (no compression, no rank) degrade find to O(n) on chains — name both optimizations and why α(n) ≈ 4.
- union must compare ROOTS; comparing raw elements merges ghosts when a node's root has changed since the last check.
- Component counting: decrement inside a successful union, or count distinct roots at the end — never both.
- 1-indexed nodes (redundant-connection) need `UF(n + 1)`; an off-by-one here silently corrupts edge 1.
- longest-consecutive-sequence: the set + "only start a streak where x-1 is absent" scan is simpler and equally O(n) — present UF as the alternative view, and the interviewer decides.
- Weighted UF (evaluate-division): path compression must multiply the weights it compresses through, or cached ratios go stale.
- Keys that aren't ints (emails, words): allocate ids via a dict BEFORE any union — retroactive mapping is where bugs breed.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Number of Provinces | Medium | https://leetcode.com/problems/number-of-provinces/ |
| Redundant Connection | Medium | https://leetcode.com/problems/redundant-connection/ |
| Accounts Merge | Medium | https://leetcode.com/problems/accounts-merge/ |
| Longest Consecutive Sequence | Medium | https://leetcode.com/problems/longest-consecutive-sequence/ |
| Evaluate Division | Medium | https://leetcode.com/problems/evaluate-division/ |
