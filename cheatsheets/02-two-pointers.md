# 02 · Two Pointers
> **Reach for it when:** the input is sorted (or sortable) and one converging scan from both ends — or a read/write cursor pair — replaces a nested loop.

## 🧠 Mental model
Two pointers earns its O(n) through a **discard argument**: after each comparison, at least one endpoint provably cannot be part of any *better* answer, so shrinking the range never loses a candidate. On a sorted array, if `numbers[lo] + numbers[hi] < target` then `lo` is hopeless against every remaining partner — advance it; the `>` case is symmetric. Both cursors only ever move toward each other, so the scan is linear with O(1) space beyond the sort. Unsorted input is fine if you sort first and carry original indices (or reordered output is acceptable). The same physical idea aimed one direction — a read cursor plus a write cursor — is how you compact, dedup, or filter arrays in place.

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "sorted array, find the pair with sum T" | converging lo/hi |
| "all UNIQUE triplets/quadruplets summing to T" | sort, fix outer elements, converge the rest, skip duplicates |
| "maximum water between two lines" | converge; always move the shorter wall |
| "elevation map, how much rain is trapped" | two pointers + running max on the limiting side |
| "valid palindrome, ignore non-alphanumeric" | converge with skip helpers |
| "palindrome if you delete at most one character" | converge; on mismatch retry both one-sided ranges |
| "boats carry 2 people, minimize trips" | sort; pair heaviest with lightest |
| "remove/compact in place, O(1) space" | read cursor + write cursor |

## 🛠️ Templates

### T1 — Converging pointers on sorted input
Pair-sum on sorted data: each step provably discards one endpoint.
```python
def two_sum_sorted(numbers, target):
    lo, hi = 0, len(numbers) - 1
    while lo < hi:                    # pairs only: cursors never cross
        s = numbers[lo] + numbers[hi]
        if s == target:
            return [lo + 1, hi + 1]   # LC wants 1-indexed answers here
        if s < target:
            lo += 1                   # too small with EVERY partner left
        else:
            hi -= 1                   # too big with EVERY partner left
    return []
```
O(n) time, O(1) space.

### T2 — Dedup-aware 3sum scan
All unique triplets summing to zero, no hash-set dedup needed.
```python
def three_sum(nums):
    nums.sort()
    out = []
    for i in range(len(nums) - 2):
        if i > 0 and nums[i] == nums[i - 1]:
            continue                  # duplicate anchor -> same triplets again
        if nums[i] > 0:
            break                     # sorted, positive anchor: sum can't be 0
        lo, hi = i + 1, len(nums) - 1
        while lo < hi:
            s = nums[i] + nums[lo] + nums[hi]
            if s < 0:
                lo += 1
            elif s > 0:
                hi -= 1
            else:
                out.append([nums[i], nums[lo], nums[hi]])
                lo += 1
                hi -= 1
                while lo < hi and nums[lo] == nums[lo - 1]:
                    lo += 1           # skip duplicate second values
                while lo < hi and nums[hi] == nums[hi + 1]:
                    hi -= 1           # skip duplicate third values
    return out
```
O(n²) time, O(1) extra space beyond the sort.

### T3 — Two-pointer rain trapping
Water above each bar in one pass, no prefix-max arrays.
```python
def trap(height):
    lo, hi = 0, len(height) - 1
    lo_max = hi_max = 0
    water = 0
    while lo < hi:
        if height[lo] < height[hi]:
            # proof sketch: some wall right of hi is >= height[hi] >
            # height[lo], so the water level at lo is decided by lo_max
            # alone -- the right side can never be the binding wall.
            lo_max = max(lo_max, height[lo])
            water += lo_max - height[lo]
            lo += 1
        else:
            hi_max = max(hi_max, height[hi])
            water += hi_max - height[hi]
            hi -= 1
    return water
```
O(n) time, O(1) space.

### T4 — In-place write pointer
Compact or filter an array in place (move-zeroes family).
```python
def move_zeroes(nums):
    write = 0                         # invariant: nums[:write] holds kept items
    for read in range(len(nums)):
        if nums[read] != 0:
            nums[write], nums[read] = nums[read], nums[write]
            write += 1
    # by invariant nums[write:] is all zeros; kept order is preserved
```
O(n) time, O(1) space.

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| converging pair scan (sorted input) | O(n) | O(1) |
| sort-then-scan (unsorted input) | O(n log n) | O(1)–O(n) per sort |
| 3sum scan | O(n²) | O(1) |
| container-with-most-water | O(n) | O(1) |
| trapping-rain-water, two pointers | O(n) | O(1) |
| trapping-rain-water, prefix maxima | O(n) | O(n) |
| read/write compaction | O(n) | O(1) |

## ⚠️ Traps interviewers probe
- two-sum-ii returns **1-indexed** positions — read the output spec before writing a line of code.
- 3sum dedup is three-headed: skip repeated anchors, and after a hit advance past repeated inner values; skipping *before* recording loses legitimate triplets.
- Container: always move the shorter wall — width shrinks either way, so keeping the taller one can only lose area. That sentence is the whole interview answer.
- valid-palindrome-ii: on the first mismatch you must try BOTH `ok(lo+1, hi)` and `ok(lo, hi-1)`; a single-guess branch fails inputs where only the other side's deletion works.
- boats-to-save-people: sort first; if heaviest + lightest fits, they sail together, else the heaviest sails alone — the lightest is the only candidate partner for the heaviest.
- Loop conditions: `lo < hi` for pairs; `lo <= hi` when the lone middle element can be the answer.
- Empty and single-element inputs: converging loops must exit cleanly — say you checked.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Valid Palindrome | Easy | https://leetcode.com/problems/valid-palindrome/ |
| Two Sum II - Input Array Is Sorted | Medium | https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/ |
| 3Sum | Medium | https://leetcode.com/problems/3sum/ |
| Container With Most Water | Medium | https://leetcode.com/problems/container-with-most-water/ |
| Trapping Rain Water | Hard | https://leetcode.com/problems/trapping-rain-water/ |
| Valid Palindrome II | Medium | https://leetcode.com/problems/valid-palindrome-ii/ |
| Boats to Save People | Medium | https://leetcode.com/problems/boats-to-save-people/ |
