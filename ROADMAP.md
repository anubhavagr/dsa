# 🗺️ ROADMAP — 93 Days · 13 Weeks · 3 Phases

**Mission:** pass any DSA round at any MAANG company, at senior level. That means two things: (1) recognize the pattern behind any problem in < 5 minutes, (2) narrate your thinking the way senior interviewers expect. Every day file trains both.

**Rules of the road**

- **Mon–Fri** = learning days (2–3 problems, adaptive). **Sat** = review + timed mini-mock. **Sun** = rest.
- Load is adaptive: pattern-drill topics (arrays, two pointers) carry 3 problems; cognitively heavy topics (DP, hard backtracking) carry 2 but expect longer on each.
- ~185 scheduled problems (≈170 unique + redoes), drawn from Blind 75, NeetCode 150, and Meta/Google high-frequency tagged lists. Difficulty ramps: mostly Easy/Medium in Phase 1, Medium/Hard by Phase 3.

---

## Phase 1 · Foundations — Weeks 1–4 (Sep 26 → Oct 25)

> Goal: fluency in linear structures and the "window" family. By the end you should solve any Blind-75 array/string problem in < 25 min.

| Week | Dates | Mon | Tue | Wed | Thu | Fri | Sat / Sun |
|---|---|---|---|---|---|---|---|
| Kick | Sep 26–27 | — | — | — | — | — | **Sep 26:** Big-O + setup · **Sep 27:** setup/rest |
| 1 | Sep 28 – Oct 4 | Arrays & Hashing | Two Pointers | Two Pointers II (trapping) | Sliding Window (fixed) | Sliding Window (variable) | review · rest |
| 2 | Oct 5 – 11 | Binary Search I | BS on Answer | BS boundaries + 2D | Prefix Sums & Kadane | Strings (palindromes, parse) | review · rest |
| 3 | Oct 12 – 18 | Linked Lists I | Linked Lists II | Linked Lists III (hard) | Stacks | Monotonic Stack & Queue | review · rest |
| 4 | Oct 19 – 25 | Recursion & Subsets | Backtracking II | Backtracking III (hard) | Sorting & quickselect | Greedy I | **Phase 1 grand review** · retro |

## Phase 2 · Core Data Structures — Weeks 5–8 (Oct 26 → Nov 22)

> Goal: trees, heaps, tries, graphs, union-find, and DP's first contact. This is the phase that separates candidates — trees and graphs are the most-failed interview topics.

| Week | Dates | Mon | Tue | Wed | Thu | Fri | Sat / Sun |
|---|---|---|---|---|---|---|---|
| 5 | Oct 26 – Nov 1 | Tree traversals | Depth & diameter | BST operations | LCA + construct trees | Serialize + tree DP | review · rest |
| 6 | Nov 2 – 8 | Heaps & Top-K | Heaps (median, streams) | Intervals | Tries | Design (LRU etc.) | review · rest |
| 7 | Nov 9 – 15 | Graph BFS/DFS | Clone & regions | Topological sort | Union-Find | Matrix as graph | review · rest |
| 8 | Nov 16 – 22 | Dijkstra / weighted | 1-D DP intro | 1-D DP sequences | 2-D grid DP | DP state design | review · **Phase 2 retro** |

## Phase 3 · Advanced & Interview Mode — Weeks 9–13 (Nov 23 → Dec 27)

> Goal: DP mastery, then company reality. Weeks 11–13 simulate the actual interview loop — tagged problems, timed sets, mocks, and targeted repair of your weakest topics.

| Week | Dates | Mon | Tue | Wed | Thu | Fri | Sat / Sun |
|---|---|---|---|---|---|---|---|
| 9 | Nov 23 – 29 | Knapsack family | Unbounded knapsack | LIS family | DP on strings (LCS) | DP palindromes | review · rest |
| 10 | Nov 30 – Dec 6 | Regex/interleave DP | DP mixed hard | Bit manipulation | Math (rotate, pow) | Math & random | review · mid-point retro |
| 11 | Dec 7 – 13 | Meta favorites I | Meta favorites II | Meta favorites III | Google favorites I | Google favorites II | company redo · rest |
| 12 | Dec 14 – 20 | Timed sim: windows | Timed sim: intervals/2p | Timed sim: graphs | Timed sim: DP | Timed sim: design | review · rest |
| 13 | Dec 21 – 27 | Weak topic #1 | Weak topic #2 | Weak topic #3 | **Grand mock** (3/75min) | Final retro + flashcards | buffer · **done 🎉** |

---

## Cheatsheet index (read order follows the plan)

| # | File | First used |
|---|---|---|
| 01 | [arrays & hashing](cheatsheets/01-arrays-hashing.md) | Kickoff + W1 |
| 02 | [two pointers](cheatsheets/02-two-pointers.md) | W1 |
| 03 | [sliding window](cheatsheets/03-sliding-window.md) | W1 |
| 04 | [binary search](cheatsheets/04-binary-search.md) | W2 |
| 05 | [prefix sums & Kadane](cheatsheets/05-prefix-sums.md) | W2 |
| 06 | [strings](cheatsheets/06-strings.md) | W1–W2 |
| 07 | [linked lists](cheatsheets/07-linked-lists.md) | W3 |
| 08 | [stacks & queues](cheatsheets/08-stacks-queues.md) | W3 |
| 09 | [recursion & backtracking](cheatsheets/09-recursion-backtracking.md) | W4 |
| 10 | [sorting & greedy](cheatsheets/10-sorting-greedy.md) | W4 |
| 11 | [trees & BST](cheatsheets/11-trees.md) | W5 |
| 12 | [heaps & top-K](cheatsheets/12-heaps.md) | W6 |
| 13 | [intervals](cheatsheets/13-intervals.md) | W6 |
| 14 | [tries](cheatsheets/14-tries.md) | W6 |
| 15 | [union-find](cheatsheets/15-union-find.md) | W7 |
| 16 | [graphs](cheatsheets/16-graphs.md) | W7–W8 |
| 17 | [dynamic programming](cheatsheets/17-dynamic-programming.md) | W8–W10 |
| 18 | [bits, math & design](cheatsheets/18-bits-math-design.md) | W6, W10 |

## Solution folder conventions

Write every accepted solution into `solutions/<topic>/<slug>.py` with a 3-line header: the pattern, your original approach before you saw the optimal one, and the bug/insight you hit. Before Sat review, re-read the week's headers — that's your personal error catalog and, in Week 13, the input for picking your weak topics.
