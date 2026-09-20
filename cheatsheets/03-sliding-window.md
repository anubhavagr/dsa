# 03 · Sliding Window
> **Reach for it when:** the ask is "longest/shortest subarray or substring with property P" and P responds monotonically as the window grows or shrinks.

## 🧠 Mental model
Sliding window is two pointers plus an invariant: window `[start, j]` is kept just barely valid (when maximizing) or just barely one step past invalid (when minimizing). Both cursors only move right, so even with an inner `while`, every element enters and leaves the window exactly once — O(n) total, no matter how the loops nest. The precondition is **monotonicity**: growing the window must push the predicate in one consistent direction (more distinct letters is strictly worse; a longer window is strictly bigger; more zeros flipped is strictly worse). Fixed-size windows maintain exact counts for length k; variable windows shrink only while invalid. "At most K" variants track a violation counter instead of full equality. The moment values can go negative (subarray-sum targets), monotonicity dies — switch to prefix-sum hashing (sheet 05).

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "longest/shortest subarray or substring with property P" | variable window, shrink-while-invalid |
| "subarray of size k with max sum/average" | fixed window: seed k, slide add-one-drop-one |
| "contains a permutation of p" | fixed window + Counter equality |
| "at most K distinct" / "at most k zeros flipped" | violation counter; shrink while violations > k |
| "smallest window covering all of t" | expand until formed, shrink while still formed |
| "longest run of one char with k replacements" | valid while `window_len - maxfreq <= k` |
| the word "subsequence" appears | NOT a window problem — different family |
| "subarray sum = k", negatives possible | prefix-sum hashmap (sheet 05), not a window |

## 🛠️ Templates

### T1 — Fixed-size window with need/have counters
All anagram start indices in one pass over s.
```python
from collections import Counter

def find_anagrams(s, p):
    need = Counter(p)
    have = Counter()
    out = []
    for i, c in enumerate(s):
        have[c] += 1
        if i >= len(p):                      # slide: drop the char leaving
            old = s[i - len(p)]
            have[old] -= 1
            if have[old] == 0:
                del have[old]                # keep == exact on any Python
        if have == need:
            out.append(i - len(p) + 1)
    return out
```
O(n) time, O(Σ) space for the counters (Σ = alphabet).

### T2 — Variable window, shrink-while-invalid
Longest substring without a repeated character.
```python
def length_of_longest_substring(s):
    last = {}                        # char -> most recent index
    best = start = 0
    for i, c in enumerate(s):
        if c in last and last[c] >= start:
            start = last[c] + 1      # jump past the previous occurrence
        last[c] = i
        best = max(best, i - start + 1)
    return best
```
O(n) time, O(min(n, Σ)) space.

### T3 — Minimum window substring (formed count)
Smallest window covering all of t, via one `missing` counter.
```python
from collections import Counter

def min_window(s, t):
    need = Counter(t)
    missing = len(t)                 # total required chars not yet in window
    best = (float('inf'), 0, 0)      # (length, start, end)
    start = 0
    for j, c in enumerate(s):
        if need[c] > 0:
            missing -= 1             # this char fills a required slot
        need[c] -= 1                 # negatives = surplus of that char
        while missing == 0:          # window covers t -- shrink hard
            if j - start + 1 < best[0]:
                best = (j - start + 1, start, j)
            need[s[start]] += 1
            if need[s[start]] > 0:   # we just evicted a required char
                missing += 1
            start += 1
    return "" if best[0] == float('inf') else s[best[1]:best[2] + 1]
```
O(|s| + |t|) time, O(Σ) space.

### T4 — At-most-K distinct generalization
Longest window with at most k distinct values; fruit-into-baskets is k = 2.
```python
from collections import defaultdict

def longest_at_most_k_distinct(s, k):
    count = defaultdict(int)
    distinct = best = start = 0
    for j, c in enumerate(s):
        if count[c] == 0:
            distinct += 1
        count[c] += 1
        while distinct > k:          # shrink until valid again
            count[s[start]] -= 1
            if count[s[start]] == 0:
                distinct -= 1
            start += 1
        best = max(best, j - start + 1)
    return best                      # max-consecutive-ones-iii: count zeros
```
O(n) amortized time, O(k) space.

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| fixed-size window pass | O(n) | O(Σ) |
| variable shrink-while-invalid | O(n) amortized | O(Σ) |
| minimum window substring | O(|s| + |t|) | O(Σ) |
| at-most-K distinct | O(n) | O(k) |
| character replacement (maxfreq) | O(n) | O(Σ) |
| naive rescan of every window | O(n·k) | O(k) |

## ⚠️ Traps interviewers probe
- Record at the tight point: maximizers record after restoring validity; minimizers record inside the shrink loop while still valid — swapping the two silently drops the optimum.
- character-replacement's `maxfreq` may be stale (too big) after shrinking — and that's safe: a stale max only prevents growth past a previously valid size; it never certifies an invalid window. Say this when asked.
- Exactly-K distinct = atMost(K) − atMost(K−1); don't try to shrink on exact equality.
- Evict `s[start]`, never `s[j]`; delete counter keys at zero so `have == need` compares exactly on every Python version.
- Fixed windows: fully seed the first k elements before the first comparison; gate output collection on `i >= k - 1`.
- permutation-in-string / anagrams: bail immediately when `len(p) > len(s)`.
- k = 0 or empty-string edges: max-consecutive-ones-iii with k = 0 degenerates to the plain longest run of ones.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Maximum Average Subarray I | Easy | https://leetcode.com/problems/maximum-average-subarray-i/ |
| Permutation in String | Medium | https://leetcode.com/problems/permutation-in-string/ |
| Find All Anagrams in a String | Medium | https://leetcode.com/problems/find-all-anagrams-in-a-string/ |
| Longest Substring Without Repeating Characters | Medium | https://leetcode.com/problems/longest-substring-without-repeating-characters/ |
| Longest Repeating Character Replacement | Medium | https://leetcode.com/problems/longest-repeating-character-replacement/ |
| Minimum Window Substring | Hard | https://leetcode.com/problems/minimum-window-substring/ |
| Fruit Into Baskets | Medium | https://leetcode.com/problems/fruit-into-baskets/ |
| Max Consecutive Ones III | Medium | https://leetcode.com/problems/max-consecutive-ones-iii/ |
