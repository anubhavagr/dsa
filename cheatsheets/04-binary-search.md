# 04 · Binary Search
> **Reach for it when:** there's a monotone boundary — a sorted array, or an answer space where "if v works, v+1 works" — and you need the first value on the far side.

## 🧠 Mental model
Binary search is not about sorted arrays; it's about a **monotone predicate**: a boundary where everything left fails and everything right succeeds, or vice versa. Keep the half-open convention `[lo, hi)` with `while lo < hi` and `hi = mid`: mid is always a live candidate, `lo = mid + 1` guarantees progress, and the loop ends with `lo == hi` holding exactly the boundary — off-by-ones become structurally impossible instead of something you debug. The same skeleton searches indices into an array, insertion points, timestamps, or values of the **answer itself** (eating speed, ship capacity, largest split sum) — only `feasible()` changes. Rotated arrays add one question per step: which half is sorted here, and does the target lie inside that half's range? If you can state the monotone claim, you can binary search it.

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "sorted" + "O(log n)" | classic half-open search |
| "first/last position / count occurrences" | two boundary searches (bisect_left/right) |
| "rotated sorted array, find target" | per step: which half is sorted, is target in its range |
| "find minimum in rotated array" | compare mid vs hi, drift toward the unsorted side |
| "minimize the maximum / eat speed / ship within D days" | binary search the ANSWER with feasible() |
| "matrix rows sorted, row[0] > prev row's last" | flatten virtually: `idx // cols`, `idx % cols` |
| "value at the latest timestamp ≤ t" | append `(t, val)` log, bisect_right |
| "median of two sorted arrays, O(log)" | partition search on the shorter array |

## 🛠️ Templates

### T1 — Half-open [lo, hi) classic
Exact-match search where the invariant is "if present, target ∈ [lo, hi)".
```python
def search(nums, target):
    lo, hi = 0, len(nums)            # half-open: hi sits one past the end
    while lo < hi:                   # invariant: target in [lo, hi) if present
        mid = (lo + hi) // 2
        if nums[mid] == target:
            return mid
        if nums[mid] < target:
            lo = mid + 1             # mid and everything left is ruled out
        else:
            hi = mid                 # mid itself may still be the answer
    return -1
```
O(log n) time, O(1) space.

### T2 — Hand-rolled bisect_left (first-true boundary)
The reusable core: smallest i where `ok(i)` flips False → True.
```python
def first_true(lo, hi, ok):
    """Smallest i in [lo, hi] with ok(i); requires ok(hi) is True."""
    while lo < hi:
        mid = (lo + hi) // 2
        if ok(mid):
            hi = mid                 # mid works: answer is mid or earlier
        else:
            lo = mid + 1             # mid fails: answer is strictly later
    return lo

def lower_bound(nums, target):       # first index of target, len(nums) if absent
    return first_true(0, len(nums), lambda i: nums[i] >= target)
```
O(log n) time, O(1) space — `bisect_left`/`bisect_right` are this exact machine.

### T3 — Search in rotated sorted array
One extra decision per step: which half is sorted, is target inside it?
```python
def search_rotated(nums, target):
    lo, hi = 0, len(nums) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if nums[mid] == target:
            return mid
        if nums[lo] <= nums[mid]:            # left half sorted (<=: 1 elem)
            if nums[lo] <= target < nums[mid]:
                hi = mid - 1                 # target inside the sorted half
            else:
                lo = mid + 1                 # must be in the other half
        else:                                # right half is the sorted one
            if nums[mid] < target <= nums[hi]:
                lo = mid + 1
            else:
                hi = mid - 1
    return -1
```
O(log n) time, O(1) space.

### T4 — Binary search on the answer
Minimize the maximum: search the value space with a monotone feasible().
```python
def minimize_max(nums, budget):      # koko / ship-packages / split-array shape
    def feasible(v):                 # monotone: if v works, v + 1 works
        need = 0
        for x in nums:
            need += (x - 1) // v     # ceil(x / v) - 1 splits under cap v
        return need <= budget

    lo, hi = 1, max(nums)            # the ANSWER lives in [1, max(nums)]
    while lo < hi:
        mid = (lo + hi) // 2
        if feasible(mid):
            hi = mid                 # feasible -> try smaller
        else:
            lo = mid + 1             # infeasible -> must go bigger
    return lo
```
O(n log(max − min)) time, O(1) space — swap feasible() for greedy day-counting on capacity problems.

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| exact search / bisect | O(log n) | O(1) |
| first + last position (two bounds) | O(log n) | O(1) |
| rotated search / find-min | O(log n) | O(1) |
| 2-D matrix as virtual 1-D | O(log(mn)) | O(1) |
| answer-space search | O(n log(max−min)) | O(1) |
| time-based get | O(log n) per query | O(n) log |
| median of two sorted (partition) | O(log min(m, n)) | O(1) |

## ⚠️ Traps interviewers probe
- Pick one convention and never mix: `lo < hi` + `hi = mid`, or `lo <= hi` + `hi = mid - 1`. Crossing them is the classic infinite loop (`hi = mid` never shrinks a `lo <= hi` range).
- With `hi = mid` you must use `lo = mid + 1` on the False branch — never `lo = mid`.
- Rotated search compares `nums[lo] <= nums[mid]` — the equality case is a one-element "half".
- find-first-and-last is two bisects; scanning outward from a hit degrades to O(n) on all-equal input, which is exactly the case they test.
- Answer-space search: state the monotone claim aloud ("if speed v finishes within h hours, v+1 does too") — that sentence IS the correctness proof, and senior interviewers score it.
- capacity-to-ship / split-array swap in a greedy feasible(): count days/pieces needed at capacity v, not the split formula.
- median-of-two: binary search the cut on the SHORTER array; even totals average max(left) and min(right); one array may be empty.
- time-based store: per-key timestamps are strictly increasing, so append + `bisect_right` for "latest ≤ t" with no dedup logic.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Binary Search | Easy | https://leetcode.com/problems/binary-search/ |
| Search in Rotated Sorted Array | Medium | https://leetcode.com/problems/search-in-rotated-sorted-array/ |
| Find Minimum in Rotated Sorted Array | Medium | https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/ |
| Find First and Last Position of Element in Sorted Array | Medium | https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/ |
| Search a 2D Matrix | Medium | https://leetcode.com/problems/search-a-2d-matrix/ |
| Time Based Key-Value Store | Medium | https://leetcode.com/problems/time-based-key-value-store/ |
| Koko Eating Bananas | Medium | https://leetcode.com/problems/koko-eating-bananas/ |
| Capacity To Ship Packages Within D Days | Medium | https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/ |
| Split Array Largest Sum | Hard | https://leetcode.com/problems/split-array-largest-sum/ |
| Median of Two Sorted Arrays | Hard | https://leetcode.com/problems/median-of-two-sorted-arrays/ |
