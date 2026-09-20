# 13 · Intervals
> **Reach for it when:** inputs are (start, end) pairs — every classic reduces to a sweep order plus one running value or one pointer.

## 🧠 Mental model
Intervals are two-dimensional keys, so the entire discipline is choosing which endpoint drives the sort and what single value you carry through the sweep. Sorted by start, one pass merges (extend the running interval or emit it). Sorted by end, one pass counts how many you can keep — the earliest-finishing interval is always a safe pick (exchange argument, sheet 10). Inserting into an already-sorted disjoint list is three phases: copy the before, absorb the overlap, copy the after. When two sorted lists must intersect, two pointers advance whichever interval ends first — it can never intersect anything more. When the question is about load or concurrency over time, stop thinking in intervals and think in endpoint events: +1 at starts, −1 at ends, swept in coordinate order.

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "merge all overlapping intervals" | sort by start, extend-or-emit |
| "max intervals you can keep / min to remove" | sort by end, greedy keep (sheet 10 T1) |
| "min arrows/points hitting every interval" | sort by end, shoot at each uncovered end |
| "insert one into sorted disjoint list" | three phases: before / absorb / after |
| "intersections of two sorted lists" | two pointers, advance the earlier-ending |
| "max simultaneous / capacity check over time" | endpoint sweep line, +1/-1 events |
| "meeting room count / car pooling" | sweep line (or heap of end times) |
| "does this new booking overlap?" | sorted list + bisect on starts |

## 🛠️ Templates
### T1 — Sort-by-start merge
Merge overlapping intervals in one sweep.

```python
def merge(intervals):
    intervals.sort()                            # by start, then end
    out = []
    for start, end in intervals:
        if out and start <= out[-1][1]:         # overlaps (or touches) last one
            out[-1][1] = max(out[-1][1], end)   # extend with MAX, not `end`
        else:
            out.append([start, end])            # disjoint: start a new interval
    return out
```
O(n log n) time, O(n) output space.

### T2 — Sort-by-end maximum non-overlap / arrows
Fewest removals, fewest arrows — same earliest-end sweep.

```python
def find_min_arrows(points):
    points.sort(key=lambda p: p[1])             # earliest end first
    arrows, last = 0, float('-inf')
    for start, end in points:
        if start > last:                        # not covered by previous arrow
            arrows += 1
            last = end                          # arrow AT the end covers most ahead
    return arrows
# non-overlapping-intervals is the mirror: count kept (start >= last_end) and
# return len(intervals) - kept.
```
O(n log n) time, O(1) extra space.

### T3 — Insertion three-phase
Insert one interval into a sorted disjoint list without re-sorting.

```python
def insert(intervals, new):
    out, i, n = [], 0, len(intervals)
    while i < n and intervals[i][1] < new[0]:   # phase 1: strictly before
        out.append(intervals[i]); i += 1
    while i < n and intervals[i][0] <= new[1]:  # phase 2: absorb all overlaps
        new[0] = min(new[0], intervals[i][0])
        new[1] = max(new[1], intervals[i][1]); i += 1
    out.append(new)
    out.extend(intervals[i:])                   # phase 3: strictly after
    return out
```
O(n) time — the input is already sorted, so sorting again wastes the gift.

### T4 — Two-pointer intersection of sorted lists
Common overlap of two sorted interval lists.

```python
def interval_intersection(A, B):
    i = j = 0
    out = []
    while i < len(A) and j < len(B):
        lo = max(A[i][0], B[j][0])              # overlap = [max starts, min ends]
        hi = min(A[i][1], B[j][1])
        if lo <= hi:
            out.append([lo, hi])
        if A[i][1] < B[j][1]:                   # whoever ends first cannot
            i += 1                              # intersect any later interval
        else:
            j += 1
    return out
```
O(m + n) time, O(1) extra beyond output.

### T5 — Endpoint sweep line
Load/concurrency over time (car pooling, meeting rooms II shape).

```python
from collections import defaultdict

def car_pool_ok(trips, capacity):               # (count, start, end) triples
    delta = defaultdict(int)
    for num, start, end in trips:
        delta[start] += num                     # events only at endpoints
        delta[end] -= num
    load = 0
    for t in sorted(delta):                     # sweep event points in time order
        load += delta[t]
        if load > capacity:
            return False
    return True
```
O(n log n) time for n events, O(n) space.

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| Sort by start + merge sweep | O(n log n) | O(n) |
| Sort by end + keep/arrow sweep | O(n log n) | O(1) |
| Insertion three-phase (pre-sorted) | O(n) | O(n) |
| Two-list intersection | O(m + n) | O(1) |
| Endpoint sweep line | O(n log n) | O(n) |
| Bisect insert into sorted list | O(log n) find, O(n) insert | O(n) |

## ⚠️ Traps interviewers probe
- Touching endpoints: [1,4] and [4,5] merge in merge-intervals (<=), but arrows counts them as one shot while non-overlap treats start == last_end as compatible — read each problem's boundary rule out loud.
- Extending with `end` instead of `max(end, last_end)` loses the outer of two nested intervals.
- Sort-key mixing: merge needs by-start, keep-max needs by-end — grabbing the wrong template is the most common miss here.
- Insert-interval assumes sorted AND disjoint input; if the interviewer drops that, sort + merge first and say why.
- Intersection advance rule is "smaller END moves", not "smaller start" — a long early interval still has future intersections.
- Sweep with full intervals instead of events is O(n²) in the worst case; only event points need sorting.
- Endpoints may be negative or reversed in some variants — don't assume 0-based coordinates.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Merge Intervals | Medium | https://leetcode.com/problems/merge-intervals/ |
| Insert Interval | Medium | https://leetcode.com/problems/insert-interval/ |
| Non-overlapping Intervals | Medium | https://leetcode.com/problems/non-overlapping-intervals/ |
| Minimum Number of Arrows to Burst Balloons | Medium | https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/ |
| Interval List Intersections | Medium | https://leetcode.com/problems/interval-list-intersections/ |
