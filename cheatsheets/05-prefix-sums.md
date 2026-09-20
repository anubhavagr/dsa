# 05 · Prefix Sums & Kadane
> **Reach for it when:** the question is about subARRAY sums/ranges — especially with negatives — or repeated range queries on a static array.

## 🧠 Mental model
A prefix array freezes n numbers of work so any range sum costs two lookups: `sum(l..r) = pre[r+1] − pre[l]`. The moment a problem asks about subarrays *by value* (count with sum k, divisible by k, length ≥ 2), switch from ranges to **prefix-value hashing**: a subarray (i, j] has sum k exactly when `pre[j] − pre[i] = k`, so count earlier prefixes equal to `running − k` — one pass, O(n) space. Because prefixes can be negative, this is the tool that replaces sliding window the instant monotonicity dies (sheet 03). Kadane is the same running-summary instinct collapsed to O(1) state: the best subarray ending here either extends the previous best or restarts at the new element. For products, carry a (min, max) pair — a negative flips which one matters.

Rules of thumb: "subarray" + "sum" → this sheet first; "subarray" + positive-only values + "at most/exactly K" → sliding window also works (sheet 03); negatives anywhere → the hashmap here is the safe answer. When they want the max subarray AND its bounds, extend Kadane to record the restart index. And "what if the array updates between queries?" is your cue to say Fenwick tree / segment tree — name it, don't build it mid-interview.

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "many range-sum queries, array never changes" | prefix array, O(1) per query |
| "count subarrays with sum exactly k" | prefix hashmap `{sum: count}` |
| "subarray sums divisible by k" | hashmap keyed on `running % k` |
| "subarray length ≥ 2 with sum multiple of k" | prefix → earliest index, seed `{0: -1}` |
| "does a zero-sum subarray exist?" | prefix repeats (or `running - k` with k = 0) |
| "maximum subarray / best contiguous run" | Kadane |
| "maximum product subarray" | Kadane with a (min, max) pair |
| "weighted random pick" | prefix sums + bisect over `r · total` |
| "sum of a submatrix many times" | 2-D prefix + inclusion–exclusion |

## 🛠️ Templates

### T1 — Prefix array + O(1) range queries
Precompute once; answer any inclusive range in constant time.
```python
class NumArray:
    def __init__(self, nums):
        self.pre = [0]                      # pre[i] = sum(nums[:i])
        for x in nums:
            self.pre.append(self.pre[-1] + x)
        # one-liner: [0, *itertools.accumulate(nums)]
        # 2-D: pre[i][j] = rect sum (0,0)..(i-1,j-1), built by
        #   pre[i][j] = pre[i-1][j] + pre[i][j-1] - pre[i-1][j-1] + grid[i-1][j-1]
        # region (r1,c1)..(r2,c2) =
        #   pre[r2+1][c2+1] - pre[r1][c2+1] - pre[r2+1][c1] + pre[r1][c1]

    def sum_range(self, left, right):       # both bounds inclusive
        return self.pre[right + 1] - self.pre[left]
```
O(n) build, O(1) per query, O(n) space.

### T2 — Prefix hashmap (sum → count / earliest index)
Counts subarrays hitting a target, negatives included.
```python
def subarray_sum(nums, k):
    seen = {0: 1}                    # prefix sum -> count of such prefixes
    running = count = 0
    for x in nums:
        running += x
        count += seen.get(running - k, 0)   # each match ends a subarray here
        seen[running] = seen.get(running, 0) + 1
    return count

def subarrays_divisible_by_k(nums, k):
    seen = {0: 1}                    # same machine, mod-k keys
    running = count = 0
    for x in nums:
        running = (running + x) % k  # Python % already lands in [0, k)
        count += seen.get(running, 0)
        seen[running] = seen.get(running, 0) + 1
    return count
    # length >= 2 (continuous-subarray-sum): store the EARLIEST index per
    # prefix instead, seed {0: -1}, and require the gap to be >= 2.
```
O(n) time, O(n) space (O(k) in the mod-k variant).

### T3 — Kadane's with running best
Maximum subarray sum in one pass, no memory.
```python
def max_subarray(nums):
    best = cur = nums[0]             # cur: best sum of a subarray ENDING here
    for x in nums[1:]:
        cur = max(x, cur + x)        # extend the run, or restart at x
        best = max(best, cur)
    return best
    # to also RETURN the subarray: note the restart index whenever
    # cur = x wins, and best_lo/best_hi whenever best improves.
```
O(n) time, O(1) space.

### T4 — Kadane's min/max variant (products)
Maximum product subarray: negatives swap min and max.
```python
def max_product(nums):
    best = cur_max = cur_min = nums[0]
    for x in nums[1:]:
        if x < 0:
            cur_max, cur_min = cur_min, cur_max   # a negative flips roles
        cur_max = max(x, cur_max * x)
        cur_min = min(x, cur_min * x)
        best = max(best, cur_max)
    return best
    # carry both because the best product ending here is x alone,
    # old_max * x, or -- via an earlier negative -- old_MIN * x;
    # a zero resets both candidates to itself for free.
```
O(n) time, O(1) space.

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| build prefix array | O(n) | O(n) |
| range query | O(1) | O(1) |
| subarray-sum-k hashmap pass | O(n) | O(n) |
| divisible-by-k hashmap pass | O(n) | O(k) |
| Kadane | O(n) | O(1) |
| max product (min/max pair) | O(n) | O(1) |
| 2-D prefix build / query | O(mn) / O(1) | O(mn) |
| weighted random pick | O(n) init, O(log n) per pick | O(n) |
| enumerate all subarrays, sum each | O(n²)–O(n³) | O(1) |
| Fenwick / segment tree (updates too) | O(log n) per op | O(n) |

## ⚠️ Traps interviewers probe
- Seed the map: `{0: 1}` when counting, `{0: -1}` when you need earliest index — forget it and every subarray starting at index 0 goes missing.
- Count BEFORE inserting the current prefix, else `k = 0` counts the empty subarray against itself.
- Python's `%` is always non-negative for positive k, so `running % k` is already normalized; in Java/C++ you must add k before taking `%` — interviewers love this detail.
- Kadane initializes with `nums[0]`, never 0: an all-negative array must return its largest element, not 0.
- Max-product: swap (min, max) BEFORE multiplying by a negative; the zero-reset falls out for free.
- The prefix array has n + 1 slots; range queries index `pre[right + 1]` — the single most common WA here.
- random-pick-with-weight: sample `random.randint(0, total - 1)`, then bisect; the boundary between prefix slots is where off-by-ones live.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Range Sum Query - Immutable | Easy | https://leetcode.com/problems/range-sum-query-immutable/ |
| Subarray Sum Equals K | Medium | https://leetcode.com/problems/subarray-sum-equals-k/ |
| Maximum Subarray | Medium | https://leetcode.com/problems/maximum-subarray/ |
| Continuous Subarray Sum | Medium | https://leetcode.com/problems/continuous-subarray-sum/ |
| Subarray Sums Divisible by K | Medium | https://leetcode.com/problems/subarray-sums-divisible-by-k/ |
| Maximum Product Subarray | Medium | https://leetcode.com/problems/maximum-product-subarray/ |
| Random Pick with Weight | Medium | https://leetcode.com/problems/random-pick-with-weight/ |
