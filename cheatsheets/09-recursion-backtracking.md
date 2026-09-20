# 09 · Recursion & Backtracking
> **Reach for it when:** you must ENUMERATE all subsets/permutations/arrangements — inputs are small (n ≤ ~20) and "try it and undo it" beats any clever formula.

## 🧠 Mental model
Backtracking is a DFS over an explicit choice tree: at each node pick a candidate, recurse, then UNDO the pick so sibling branches start from identical state. The code is always the same three lines (choose / explore / unchoose); the engineering is all in dedup and pruning. Sort first so duplicates sit adjacent, then skip a candidate equal to its predecessor at the same tree level — that one `if` is what makes outputs unique without a set. Use `used[]` for permutations (order matters, elements are positions), a `start` index for combinations/subsets (order doesn't matter, never look left), and pass `i` instead of `i + 1` when candidates are reusable. Grids mark in place and unmark on the way out; constraint problems (n-queens, sudoku) put cols/diagonals/boxes into sets so legality checks are O(1). Counting or optimizing rather than enumerating? That's DP, not backtracking — the tell is repeated subproblems.

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "return ALL subsets/combinations/permutations" | choice tree with explicit undo |
| "input n ≤ ~20" | exponential enumeration is expected — don't hunt for DP |
| "count the ways / minimum cost" | DP, not enumeration (repeated subproblems) |
| "find a path in a grid spelling a word" | DFS with mark/unmark |
| "place n non-attacking queens / fill the sudoku" | constraint sets, prune at each row |
| "split the string into valid pieces" | backtracking over cut positions |
| "candidates may be reused unlimited times" | recurse with i, not i + 1 |
| "duplicates in input, output must be unique" | sort + same-level skip |
| "find many words in one grid" | build a Trie first, share the walk |

## 🛠️ Templates

### T1 — Include/exclude subsets with start-index dedup
All unique subsets of a possibly-duplicate array.
```python
def subsets_with_dup(nums):
    nums.sort()                        # duplicates become adjacent
    out = []

    def backtrack(start, path):
        out.append(path[:])            # every tree node is a valid subset
        for i in range(start, len(nums)):
            if i > start and nums[i] == nums[i - 1]:
                continue               # same tree level: repeated branch
            path.append(nums[i])       # choose
            backtrack(i + 1, path)     # explore (i+1: never look left)
            path.pop()                 # unchoose

    backtrack(0, [])
    return out
```
O(n · 2^n) time, O(n) depth space.

### T2 — used[] permutations
Every ordering, with a boolean array as the occupancy record.
```python
def permute(nums):
    out, used = [], [False] * len(nums)

    def backtrack(path):
        if len(path) == len(nums):
            out.append(path[:])
            return
        for i in range(len(nums)):
            if used[i]:
                continue
            used[i] = True            # choose
            path.append(nums[i])
            backtrack(path)           # explore
            path.pop()                # unchoose, in reverse order
            used[i] = False

    backtrack([])
    return out
```
O(n · n!) time, O(n) space.

### T3 — Grid DFS with unmark
Word search: temporarily claim cells, always give them back.
```python
def exist(board, word):
    rows, cols = len(board), len(board[0])

    def dfs(r, c, i):
        if i == len(word):
            return True               # consumed the whole word
        if (r < 0 or r >= rows or c < 0 or c >= cols
                or board[r][c] != word[i]):
            return False
        board[r][c] = '#'             # mark: no separate seen-set needed
        found = (dfs(r + 1, c, i + 1) or dfs(r - 1, c, i + 1) or
                 dfs(r, c + 1, i + 1) or dfs(r, c - 1, i + 1))
        board[r][c] = word[i]         # UNMARK: other paths may need this cell
        return found

    return any(dfs(r, c, 0) for r in range(rows) for c in range(cols))
```
O(R · C · 3^L) time (3, not 4: never revisit where you came from), O(L) space.

### T4 — Combination-sum unbounded picks
Candidates reusable: recurse with the SAME index.
```python
def combination_sum(candidates, target):
    out = []

    def backtrack(start, remain, path):
        if remain == 0:
            out.append(path[:])
            return
        for i in range(start, len(candidates)):
            if candidates[i] > remain:
                continue              # (sorted input: `break` instead)
            path.append(candidates[i])
            backtrack(i, remain - candidates[i])   # i, not i+1: reuse allowed
            path.pop()

    backtrack(0, target, [])
    return out
```
O(n^(T/M)) nodes in the worst case (T = target, M = min candidate), O(T/M) depth.

### T5 — Constraint-set backtracking (n-queens)
Conflicts in hash sets: O(1) legality, no board rescans.
```python
def solve_n_queens(n):
    out, queens = [], []              # queens[r] = chosen column per row
    cols, d1, d2 = set(), set(), set()

    def backtrack(r):
        if r == n:
            out.append(["." * c + "Q" + "." * (n - c - 1) for c in queens])
            return
        for c in range(n):
            if c in cols or (r - c) in d1 or (r + c) in d2:
                continue              # O(1) safety via constraint sets
            queens.append(c)
            cols.add(c)
            d1.add(r - c)             # r - c is fixed on "\" diagonals
            d2.add(r + c)             # r + c is fixed on "/" diagonals
            backtrack(r + 1)
            queens.pop()              # undo all four, in reverse
            cols.remove(c)
            d1.remove(r - c)
            d2.remove(r + c)

    backtrack(0)
    return out
```
O(n!) time, O(n) space — sudoku is the same shape with row/col/box sets.

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| subsets | O(n · 2^n) | O(n) |
| subsets-ii / combination-sum-ii | O(n · 2^n) worst | O(n) |
| permutations | O(n · n!) | O(n) |
| combination-sum (reuse) | O(n^(T/M)) | O(T/M) |
| word-search | O(R · C · 3^L) | O(L) |
| palindrome-partitioning | O(n · 2^n) | O(n) |
| restore-ip-addresses | O(1) (≤ 3^3 cuts) | O(4) |
| n-queens | O(n!) | O(n) |
| sudoku-solver | O(9^m), m = empty cells | O(81) |

## ⚠️ Traps interviewers probe
- Always append `path[:]` — appending the live list aliases every answer to the same (eventually empty) object. The #1 silent WA in this family.
- The dedup condition is `i > start`, not `i > 0`: same-level repeats are forbidden, cross-level repeats are legitimate — the latter kills valid answers.
- Permutations with duplicate inputs: sort, then skip `nums[i] == nums[i-1]` when `not used[i-1]` — the "not" is what keeps one representative per value per level.
- Grid DFS: unmark on EVERY exit path; a forgotten unmark poisons later searches in the same board run.
- combination-sum passes `i` (reuse); combination-sum-ii passes `i + 1` and level-skips — mixing the two is the most common WA on this sheet.
- word-search-ii: DFS-per-word is O(words × R·C·3^L) and TLEs; build a Trie, share one walk, and prune matched leaves so found words don't resurface.
- Prune loudly and early (remain < 0, placed queens > free rows, first-invalid-cut in partitions) — "how would you cut this down?" is the guaranteed follow-up.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Subsets | Medium | https://leetcode.com/problems/subsets/ |
| Subsets II | Medium | https://leetcode.com/problems/subsets-ii/ |
| Combination Sum | Medium | https://leetcode.com/problems/combination-sum/ |
| Permutations | Medium | https://leetcode.com/problems/permutations/ |
| Word Search | Medium | https://leetcode.com/problems/word-search/ |
| Palindrome Partitioning | Medium | https://leetcode.com/problems/palindrome-partitioning/ |
| Restore IP Addresses | Medium | https://leetcode.com/problems/restore-ip-addresses/ |
| N-Queens | Hard | https://leetcode.com/problems/n-queens/ |
| Word Search II | Hard | https://leetcode.com/problems/word-search-ii/ |
| Sudoku Solver | Hard | https://leetcode.com/problems/sudoku-solver/ |
