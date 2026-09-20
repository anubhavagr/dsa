# 17 · Dynamic Programming
> **Reach for it when:** the search space is exponential but the *state* is small — "what must I remember about the past to decide the future?"

## 🧠 Mental model
DP is recursion plus a cache, and the cache key IS the design: find the smallest tuple of facts (index, remaining budget, last choice, bitmask of used items) such that the future depends only on those facts — that tuple is the state. Write the brute-force recursion first, memoize it top-down to prove correctness, then translate to a bottom-up table for the interview-clean constant factors. The second half of the design is iteration order: dependencies must be computed before dependents (prefix → suffix, smaller amount → larger, shorter interval → longer).

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "ways / min cost to reach step i" | linear DP, O(1) rolling pair (T1) |
| "coins/items with unlimited reuse, hit amount" | unbounded knapsack — loop order decides combinations vs permutations (T2) |
| "pick each item at most once, hit target sum" | 0/1 knapsack in 1-D, REVERSE inner loop (T3) |
| "grid paths / squares in a grid" | 2-D DP with rolling row (T4) |
| "two strings, align / edit / count matches" | prefix-prefix table dp[i][j] (T5) |
| "palindromic subsequence / split costs" | interval DP, length order (T6) |
| "longest increasing subsequence" | patience tails + bisect (T7) |
| "buy/sell with cooldown or fees" | state-machine DP (T8) |
| "match string against pattern with * / ?" | 2-D matching DP with '*' branches (T9) |

## 🛠️ Templates
### T1 — Memo → table translation (house robber)
The canonical top-down → rolling-variables move.

```python
def rob(nums):
    take, skip = 0, 0                # best ending WITH / WITHOUT nums[i]
    for x in nums:
        take, skip = skip + x, max(take, skip)   # simultaneous assignment!
    return max(take, skip)
# Top-down memo: f(i) = max(nums[i] + f(i+2), f(i+1)) — two live values suffice.
```
O(n) time, O(1) space. Worked table for `nums = [2, 7, 9, 3, 1]`:

| x | take ← skip+x | skip ← max(prev take, prev skip) |
|---|---|---|
| 2 | 2 | 0 |
| 7 | 7 | 2 |
| 9 | 11 | 7 |
| 3 | 10 | 11 |
| 1 | 12 | 11 |

Answer `max(12, 11) = 12` → pick 2 + 9 + 1. House-robber-ii runs this twice (drop first element, drop last element) and takes the max.

### T2 — Unbounded knapsack (coin change) + loop order
The inner/outer loop order is the whole problem.

```python
def coin_change(coins, amount):
    dp = [0] + [float('inf')] * amount     # dp[a]: fewest coins making a
    for c in coins:                        # OUTER COINS -> combinations
        for a in range(c, amount + 1):     # each amount sees each coin once
            dp[a] = min(dp[a], dp[a - c] + 1)
    return dp[amount] if dp[amount] != float('inf') else -1

# Swap the loops — outer amount + inner coins — and dp counts PERMUTATIONS
# (combination-sum-iv: dp[a] += dp[a-c] sees orderings ending in any coin, so
# 1+2 and 2+1 differ). coin-change-ii counts combinations: coins stay outside.
```
O(coins · amount) time, O(amount) space.

### T3 — 0/1 knapsack in 1-D with REVERSE iteration
Each item used at most once — partition-equal-subset-sum.

```python
def can_partition(nums):
    total = sum(nums)
    if total % 2:
        return False
    target = total // 2
    dp = [True] + [False] * target         # dp[s]: some subset sums to s
    for x in nums:                         # each item considered ONCE
        for s in range(target, x - 1, -1): # REVERSE: dp[s-x] must still be
            dp[s] = dp[s] or dp[s - x]     # the PREVIOUS item's row (0/1 rule)
    return dp[target]
# Forward iteration would let x be reused -> unbounded knapsack (T2).
```
O(n · target) time, O(target) space. Target-sum and last-stone-weight-ii are the same subset-sum core with a signed/offset twist.

### T4 — 2-D grids with rolling row
Unique paths (paths in) and maximal square (min of three neighbors).

```python
def unique_paths(m, n):
    row = [1] * n                          # top row: exactly one way each
    for _ in range(1, m):
        for c in range(1, n):
            row[c] += row[c - 1]           # from above (old row[c]) + left
    return row[-1]

def maximal_square(grid):
    n, row, best = len(grid[0]), [0] * (len(grid[0]) + 1), 0
    for cells in grid:
        diag = 0                           # saves dp[i-1][c-1] before overwrite
        for c in range(1, n + 1):
            up = row[c]
            row[c] = min(row[c], row[c - 1], diag) + 1 if cells[c - 1] == "1" else 0
            best, diag = max(best, row[c]), up    # side length; area = best²
    return best * best
```
O(m·n) time, O(n) space.

### T5 — Two-string table (LCS / edit distance)
dp[i][j] over prefixes of both strings.

```python
def lcs(a, b):
    m, n = len(a), len(b)
    dp = [[0] * (n + 1) for _ in range(m + 1)]   # dp[i][j] = LCS(a[:i], b[:j])
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if a[i - 1] == b[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1  # match extends the diagonal
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    return dp[m][n]
# edit-distance: same table, same order; equal chars -> copy dp[i-1][j-1],
# else 1 + min(dp[i-1][j-1], dp[i-1][j], dp[i][j-1])  (replace/delete/insert).
```
O(m·n) time and space (O(min(m,n)) with a rolling row). Worked LCS of `a="ace"`, `b="abcde"`:

```
        ""  a   b   c   d   e
    ""   0  0   0   0   0   0
    a    0  1   1   1   1   1
    c    0  1   1   2   2   2
    e    0  1   1   2   2   3      answer dp[3][5] = 3  ("ace")
```

### T6 — Interval / gap DP (longest palindromic subsequence)
Grow answers from the inside out, by length.

```python
def longest_palindromic_subsequence(s):
    n = len(s)
    dp = [[0] * n for _ in range(n)]        # dp[i][j]: best inside s[i..j]
    for i in range(n - 1, -1, -1):          # i decreasing, j increasing
        dp[i][i] = 1                        # single char: length 1
        for j in range(i + 1, n):
            if s[i] == s[j]:
                dp[i][j] = dp[i + 1][j - 1] + 2     # bookend match
            else:
                dp[i][j] = max(dp[i + 1][j], dp[i][j - 1])  # drop an end
    return dp[0][n - 1]
# palindrome-partitioning-ii: dp[i] = min cuts of s[:i] over j<i with s[j:i] pal.
```
O(n²) time and space.

### T7 — LIS with bisect
Patience sorting: tails, not the subsequence itself.

```python
from bisect import bisect_left

def length_of_lis(nums):
    tails = []                              # tails[k]: min tail of any
    for x in nums:                          # increasing subseq of length k+1
        i = bisect_left(tails, x)           # first tail >= x (strict LIS)
        if i == len(tails):
            tails.append(x)                 # x extends the longest one
        else:
            tails[i] = x                    # same length, smaller tail: better
    return len(tails)
# tails is NOT a real subsequence — only its length is the answer.
# number-of-LIS adds a parallel cnt[]; russian-doll sorts width ASC, LIS on height.
```
O(n log n) time, O(n) space.

### T8 — State-machine DP (stock with cooldown)
Fixed small set of states, transitions per day.

```python
def max_profit(prices):
    hold, sold, rest = float('-inf'), float('-inf'), 0
    for p in prices:                        # all values = "after today"
        hold, sold, rest = max(hold, rest - p), hold + p, max(rest, sold)
    return max(sold, rest)
# hold = owning stock; sold = sold TODAY (forced cooldown tomorrow);
# rest = free to buy. Answer excludes `hold` (unsold stock is worthless).
```
O(n) time, O(1) space — maximum-product-subarray is the same shape with (max, min) running products.

### T9 — Matching DP with '*' branches (regex)
The '*' branches are: zero copies, or one-more copy.

```python
def is_match(s, p):
    m, n = len(s), len(p)
    dp = [[False] * (n + 1) for _ in range(m + 1)]  # s[:i] matches p[:j]
    dp[0][0] = True
    for j in range(2, n + 1):               # empty s vs "a*b*c*..." prefixes
        if p[j - 1] == '*':
            dp[0][j] = dp[0][j - 2]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if p[j - 1] == '*':
                dp[i][j] = dp[i][j - 2]               # '*' = zero occurrences
                if p[j - 2] in (s[i - 1], '.'):
                    dp[i][j] = dp[i][j] or dp[i - 1][j]   # or absorb s[i-1]
            elif p[j - 1] in (s[i - 1], '.'):
                dp[i][j] = dp[i - 1][j - 1]           # plain consume
    return dp[m][n]
# wildcard-matching: same table; '*' = any run: dp[i][j] = dp[i-1][j] or dp[i][j-1].
```
O(m·n) time and space.

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| Linear DP (T1, T8) | O(n) | O(1) |
| Unbounded / 0/1 knapsack (T2, T3) | O(n · target) | O(target) |
| Grid DP rolling row (T4) | O(m·n) | O(n) |
| Two-string table (T5, T9) | O(m·n) | O(m·n) |
| Interval DP (T6) | O(n²)–O(n³) with split | O(n²) |
| LIS via bisect (T7) | O(n log n) | O(n) |
| Bitmask over subsets (k-subsets, TSP shape) | O(n · 2ⁿ) | O(2ⁿ) |

## ⚠️ Traps interviewers probe
- Knapsack loop order: outer items = combinations, outer capacity = permutations — state the invariant out loud before coding coin-change-ii vs combination-sum-iv.
- The 1-D 0/1 knapsack needs the REVERSE capacity loop; forward silently upgrades it to unbounded.
- Initialize impossible states as inf/False and only improve them, or you count unreachable states as free.
- tails in LIS is not a reconstructable subsequence; counting LIS needs the parallel cnt[] updated on equal lengths.
- Answer location varies by shape: interval DP reads dp[0][n-1], palindromic-substrings COUNTS true cells, burst-balloons reads the full-table max — know which before looping.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Climbing Stairs | Easy | https://leetcode.com/problems/climbing-stairs/ |
| Min Cost Climbing Stairs | Easy | https://leetcode.com/problems/min-cost-climbing-stairs/ |
| House Robber | Medium | https://leetcode.com/problems/house-robber/ |
| House Robber II | Medium | https://leetcode.com/problems/house-robber-ii/ |
| House Robber III | Medium | https://leetcode.com/problems/house-robber-iii/ |
| Coin Change | Medium | https://leetcode.com/problems/coin-change/ |
| Coin Change II | Medium | https://leetcode.com/problems/coin-change-ii/ |
| Combination Sum IV | Medium | https://leetcode.com/problems/combination-sum-iv/ |
| Longest Increasing Subsequence | Medium | https://leetcode.com/problems/longest-increasing-subsequence/ |
| Number of Longest Increasing Subsequence | Medium | https://leetcode.com/problems/number-of-longest-increasing-subsequence/ |
| Russian Doll Envelopes | Hard | https://leetcode.com/problems/russian-doll-envelopes/ |
| Longest String Chain | Medium | https://leetcode.com/problems/longest-string-chain/ |
| Unique Paths | Medium | https://leetcode.com/problems/unique-paths/ |
| Minimum Path Sum | Medium | https://leetcode.com/problems/minimum-path-sum/ |
| Maximal Square | Medium | https://leetcode.com/problems/maximal-square/ |
| Word Break | Medium | https://leetcode.com/problems/word-break/ |
| Decode Ways | Medium | https://leetcode.com/problems/decode-ways/ |
| Partition Equal Subset Sum | Medium | https://leetcode.com/problems/partition-equal-subset-sum/ |
| Target Sum | Medium | https://leetcode.com/problems/target-sum/ |
| Last Stone Weight II | Medium | https://leetcode.com/problems/last-stone-weight-ii/ |
| Longest Common Subsequence | Medium | https://leetcode.com/problems/longest-common-subsequence/ |
| Edit Distance | Medium | https://leetcode.com/problems/edit-distance/ |
| Distinct Subsequences | Hard | https://leetcode.com/problems/distinct-subsequences/ |
| Palindromic Substrings | Medium | https://leetcode.com/problems/palindromic-substrings/ |
| Longest Palindromic Subsequence | Medium | https://leetcode.com/problems/longest-palindromic-subsequence/ |
| Palindrome Partitioning II | Hard | https://leetcode.com/problems/palindrome-partitioning-ii/ |
| Regular Expression Matching | Hard | https://leetcode.com/problems/regular-expression-matching/ |
| Wildcard Matching | Hard | https://leetcode.com/problems/wildcard-matching/ |
| Interleaving String | Medium | https://leetcode.com/problems/interleaving-string/ |
| Burst Balloons | Hard | https://leetcode.com/problems/burst-balloons/ |
| Maximum Product Subarray | Medium | https://leetcode.com/problems/maximum-product-subarray/ |
| Best Time to Buy and Sell Stock with Cooldown | Medium | https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/ |
| Perfect Squares | Medium | https://leetcode.com/problems/perfect-squares/ |
| Partition to K Equal Sum Subsets | Medium | https://leetcode.com/problems/partition-to-k-equal-sum-subsets/ |
