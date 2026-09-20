# 01 · Arrays & Hashing
> **Reach for it when:** the ask is O(n) — "do these exist / group these / count these" over an array or string — and a hash map kills the nested loop.

## 🧠 Mental model

### Big-O refresher
| Complexity | What it means practically at n = 10⁵ |
|---|---|
| O(1) | Hash get/set, array index. Free — do it inside loops without guilt. |
| O(log n) | ~17 probes. Binary search, balanced-tree operations. |
| O(n) | One pass, ~10⁵ ops, milliseconds. The interview target. |
| O(n log n) | Sort scale, ~1.7·10⁶ ops. Still comfortably passes. |
| O(n²) | 10¹⁰ ops, seconds-to-minutes. Rejected — hunt for the hash/index trick. |

Most brute-force array answers are O(n²) because for each element you rescan the array to find its partner; a hash map trades O(n) space to make that inner "find" O(1). The invariant behind every template on this sheet: **the map is always a complete summary of the elements already seen**, so each new element is answered against the past in constant time. Grouping is the same trick keyed by a *canonical form*, so rearrangements collide in one bucket. When division is banned or zeros lurk, think prefix × suffix products. And the moment the interviewer says "sorted", consider graduating to two pointers (sheet 02) or binary search (sheet 04).

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "return indices of the pair that sums to T" | one-pass hash: `seen[value] -> index` |
| "any value appears twice / all distinct?" | set with early return |
| "valid anagram / same multiset?" | length check + Counter equality |
| "group strings that are rearrangements" | canonical key: `tuple(sorted(s))` or 26-count tuple |
| "top K frequent elements" | Counter → heap of size k, or frequency buckets |
| "product of all except self, no division" | prefix × suffix two-pass, O(1) extra |
| "longest consecutive sequence, O(n)" | set; only start counting at heads (`x-1 not in s`) |
| "move zeroes / compact in place" | read cursor + write cursor |
| "insert, delete, getRandom all O(1)" | dict + array, swap-with-last to delete |

## 🛠️ Templates

### T1 — One-pass hash map
Finds a pair hitting a target in a single sweep.
```python
def two_sum(nums, target):
    seen = {}                        # value -> index: summary of the past
    for i, x in enumerate(nums):
        if target - x in seen:       # partner already recorded
            return [seen[target - x], i]
        seen[x] = i                  # insert AFTER checking: i != j guaranteed
    return []
```
O(n) time, O(n) space.

### T2 — Counting with Counter / defaultdict
Frequency facts and bucket-sorted top-K without a heap.
```python
from collections import Counter

def is_anagram(s, t):
    return len(s) == len(t) and Counter(s) == Counter(t)  # multiset equality

def top_k_frequent(nums, k):
    count = Counter(nums)
    buckets = [[] for _ in range(len(nums) + 1)]  # frequency is in 1..n
    for v, c in count.items():
        buckets[c].append(v)                      # bucket sort on frequency
    out = []
    for c in range(len(buckets) - 1, 0, -1):      # highest frequency first
        out.extend(buckets[c])
        if len(out) >= k:
            return out[:k]
```
O(n) time, O(n) space — beats the O(n log k) heap when asked for true linear.

### T3 — Canonical grouping keys
Anything "rearranges to the same thing" lands in one bucket.
```python
from collections import defaultdict

def group_anagrams(strs):
    groups = defaultdict(list)
    for s in strs:
        key = tuple(sorted(s))       # canonical form: rearrangements collide
        groups[key].append(s)
    return list(groups.values())
    # Long words: 26-count key is O(L) instead of O(L log L):
    #   cnt = [0] * 26; for ch in s: cnt[ord(ch) - 97] += 1; key = tuple(cnt)
    # Arbitrary iterables: key = frozenset(Counter(x).items())
```
O(n · L log L) time with the sorted key, O(n · L) space; the count key drops per-word cost to O(L).

### T4 — Prefix × suffix product
Product of everything except self, division-free, O(1) extra space.
```python
def product_except_self(nums):
    n = len(nums)
    out = [1] * n
    pre = 1
    for i in range(n):               # out[i] = product of all left of i
        out[i] = pre
        pre *= nums[i]
    suf = 1
    for i in range(n - 1, -1, -1):   # fold in everything right of i
        out[i] *= suf
        suf *= nums[i]
    return out
```
O(n) time, O(1) extra space (output array excluded by convention).

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| dict / set lookup (average) | O(1) | O(n) stored |
| build set or Counter over n items | O(n) | O(n) |
| canonical key, word length L (sorted tuple) | O(L log L) | O(L) |
| canonical key (26-count tuple) | O(L) | O(1) |
| product-except-self | O(n) | O(1) extra |
| top-K via frequency buckets | O(n) | O(n) |
| top-K via heap (`nlargest`) | O(n log k) | O(k) |
| nested-loop brute force | O(n²) | O(1) |

## ⚠️ Traps interviewers probe
- Two-sum: insert **after** the check, or `target = 6, x = 3` happily pairs with itself.
- Anagram: compare lengths first; a `[0] * 26` array silently assumes lowercase — state the assumption or use Counter.
- Group-anagrams: the sorted-string key costs O(L log L) per word; volunteer the count-tuple key for long strings — that's the standard follow-up.
- Top-K: buckets give true O(n); if you reach for `heapq.nlargest`, say "O(n log k)" out loud — complexity is the actual question.
- Longest-consecutive: the `x - 1 not in seen` guard is what makes it O(n); without it, degenerate runs re-scan and you're back to O(n²) despite the set.
- Product-except-self: zeros and negatives are exactly why division is banned; the follow-up is "fewer passes / O(1) extra" — this template already answers it.
- insert-delete-getrandom: `list.pop(middle)` is O(n) — swap the victim with the last element, then pop; the dict maps value → slot, so values must be unique (ask if duplicates are allowed).

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Two Sum | Easy | https://leetcode.com/problems/two-sum/ |
| Contains Duplicate | Easy | https://leetcode.com/problems/contains-duplicate/ |
| Valid Anagram | Easy | https://leetcode.com/problems/valid-anagram/ |
| Move Zeroes | Easy | https://leetcode.com/problems/move-zeroes/ |
| Group Anagrams | Medium | https://leetcode.com/problems/group-anagrams/ |
| Top K Frequent Elements | Medium | https://leetcode.com/problems/top-k-frequent-elements/ |
| Product of Array Except Self | Medium | https://leetcode.com/problems/product-of-array-except-self/ |
| Longest Consecutive Sequence | Medium | https://leetcode.com/problems/longest-consecutive-sequence/ |
| Insert Delete GetRandom O(1) | Medium | https://leetcode.com/problems/insert-delete-getrandom-o1/ |
