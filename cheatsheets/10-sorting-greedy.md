# 10 · Sorting & Greedy
> **Reach for it when:** the input can be reordered for free, or a single running extreme (earliest end, furthest reach, cumulative balance) decides everything.

## 🧠 Mental model
Sorting turns a global, order-dependent question into a local, one-pass one: once items are sorted by the right key, each item only has to be compared against a single running value. Greedy is justified by an exchange argument — show that any optimal solution can be morphed into the greedy one without loss ("the interval I kept ends earliest, so swapping any kept interval for mine never breaks the rest"). If you can state the invariant ("last_end is minimal over all valid prefixes"), you can defend the greedy in an interview; if you can't, it is probably DP. Partial-order tools (quickselect, 3-way partition) buy average O(n) when only order statistics are needed.

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "maximum non-overlapping intervals / min removals" | sort by **end**, keep earliest-finishing |
| "min arrows/points to hit all intervals" | sort by end, shoot at ends |
| "merge overlapping intervals" | sort by start, extend-or-emit |
| "kth largest, don't need full order" | quickselect partition, avg O(n) |
| "three-way classify in-place (0/1/2)" | Dutch national flag |
| "can you reach the last index / min jumps" | maintain a reach window over i |
| "circular tour with gas + cost" | cumulative balance with reset |
| "tasks with cooldown n, min time" | max-heap by count + cooldown queue (sheet 12) |
| "two per boat under a weight limit" | sort, two pointers lightest + heaviest |

## 🛠️ Templates
### T1 — Sort-by-end interval greedy
Maximum set of non-overlapping intervals (arrows, erase-to-remove).

```python
def max_non_overlapping(intervals):
    intervals.sort(key=lambda iv: iv[1])        # earliest end first
    kept, last_end = 0, float('-inf')
    for start, end in intervals:
        if start >= last_end:                   # compatible with everything kept
            kept += 1
            last_end = end                      # invariant: last_end stays minimal
    return kept
# Exchange argument: any optimal solution can swap its first interval for the
# earliest-ending one and stay valid — so the greedy choice is always safe.
```
O(n log n) time, O(1) extra space.

### T2 — Quickselect partition (kth largest)
Order statistic without a full sort.

```python
import random

def find_kth_largest(nums, k):
    target, lo, hi = len(nums) - k, 0, len(nums) - 1   # index if fully sorted
    while True:
        p = random.randint(lo, hi)
        nums[p], nums[hi] = nums[hi], nums[p]          # park pivot at hi
        pivot, store = nums[hi], lo
        for i in range(lo, hi):                        # Lomuto scan
            if nums[i] < pivot:
                nums[i], nums[store] = nums[store], nums[i]
                store += 1
        nums[store], nums[hi] = nums[hi], nums[store]  # pivot to final spot
        if store == target:
            return nums[store]
        if store < target:
            lo = store + 1                             # answer is on the right
        else:
            hi = store - 1                             # discard ~half each round
```
Average O(n) (geometric shrink), O(n²) worst case, O(1) space.

### T3 — Dutch national flag 3-way
Sort three classes in one pass, in place.

```python
def sort_colors(nums):
    lo, i, hi = 0, 0, len(nums) - 1
    while i <= hi:                       # invariant: [0,lo)=0  [lo,i)=1  (hi,end]=2
        if nums[i] == 0:
            nums[lo], nums[i] = nums[i], nums[lo]
            lo += 1; i += 1              # swapped-in value is already classified
        elif nums[i] == 2:
            nums[i], nums[hi] = nums[hi], nums[i]
            hi -= 1                      # do NOT advance i: new nums[i] is unseen
        else:
            i += 1
```
O(n) single pass, O(1) space — also the fix for quickselect on duplicate-heavy arrays.

### T4 — Jump-game reach window
Can you reach the end? (Min-jumps layers the same idea into level windows.)

```python
def can_jump(nums):
    reach = 0                             # furthest index reachable so far
    for i, x in enumerate(nums):
        if i > reach:                     # there is a gap: index i is unreachable
            return False
        reach = max(reach, i + x)         # invariant: all indices <= reach are live
    return True
```
O(n) time, O(1) space.

### T5 — Gas-station cumulative balance reset
Where (if anywhere) can the circular tour start?

```python
def can_complete_circuit(gas, cost):
    if sum(gas) < sum(cost):              # global deficit: no start can win
        return -1
    start = tank = 0
    for i in range(len(gas)):
        tank += gas[i] - cost[i]          # balance accumulated since `start`
        if tank < 0:                      # no j in (start, i] works either:
            start, tank = i + 1, 0        # every prefix balance there was <= tank
    return start
```
O(n) time, O(1) space.

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| Python sort (Timsort) | O(n log n) | O(n) |
| Sorted sweep with running extreme | O(n log n) | O(1) |
| Quickselect | O(n) avg, O(n²) worst | O(1) |
| Dutch national flag | O(n) | O(1) |
| Reach window (jump games) | O(n) | O(1) |
| Balance-reset tour | O(n) | O(1) |
| Sort + two pointers (boats) | O(n log n) | O(1) |

## ⚠️ Traps interviewers probe
- Sorting by start when the proof needs earliest end (max non-overlap / arrows) — the counterexample is one long interval that starts first.
- Merge must extend with `max(last_end, end)`, not overwrite — nested intervals silently lose length otherwise.
- Quickselect degrades toward O(n²) on arrays full of duplicates with a 2-way partition; 3-way (T3) restores O(n).
- Gas station: check `sum(gas) >= sum(cost)` first; the reset loop alone does not establish global infeasibility.
- Jump game: `i + x` may exceed the last index — that is fine, don't clamp the comparison around it.
- Boats: when lightest + heaviest fit, both board (advance both pointers); otherwise only the heaviest (advance the heavy pointer alone).
- Interval-list intersections advance the pointer of the interval that **ends** first, not the one that starts first.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Sort Colors | Medium | https://leetcode.com/problems/sort-colors/ |
| Kth Largest Element in an Array | Medium | https://leetcode.com/problems/kth-largest-element-in-an-array/ |
| Merge Intervals | Medium | https://leetcode.com/problems/merge-intervals/ |
| Jump Game | Medium | https://leetcode.com/problems/jump-game/ |
| Gas Station | Medium | https://leetcode.com/problems/gas-station/ |
| Partition Labels | Medium | https://leetcode.com/problems/partition-labels/ |
| Task Scheduler | Medium | https://leetcode.com/problems/task-scheduler/ |
| Interval List Intersections | Medium | https://leetcode.com/problems/interval-list-intersections/ |
| Boats to Save People | Medium | https://leetcode.com/problems/boats-to-save-people/ |
