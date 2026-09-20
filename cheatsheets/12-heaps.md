# 12 · Heaps & Top-K
> **Reach for it when:** you repeatedly need the current extreme — not everything sorted, just the best-so-far.

## 🧠 Mental model
A heap answers "give me the min (or max) right now" in O(log n) and nothing else — it is a priority, not an order. Three moves cover the family: (1) cap the heap at size k so "top-k" costs O(n log k) instead of O(n log n); (2) negate values, because `heapq` is min-only, and put a tie-breaking index in tuples before any non-comparable payload; (3) split the stream between a max-half and a min-half to get order statistics like the running median. Merging k sorted sources never needs more than the k current heads in the heap. Lazy deletion — pushing now, skipping stale entries on pop — is how Dijkstra rides a heap (sheet 16).

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "k largest / smallest / closest" | size-k min-heap (or quickselect, sheet 10) |
| "k most frequent" | Counter + size-k heap keyed on (-count, val) |
| "running median of a stream" | two heaps balanced to within 1 |
| "merge k sorted lists/arrays" | heap of current heads, pop-append-push |
| "k pairs with smallest sums" | seed (0,0), expand one index per pop |
| "repeatedly take the max, wait, retake" | max-heap + cooldown queue (task scheduler) |
| "kth largest in a stream" | heap of size k via pushpop; h[0] is the answer |
| "schedule / simulate with priorities" | heap + clock loop, skip stale entries |

## 🛠️ Templates
### T1 — Size-k min-heap for top-k
Keep only the k best seen so far.

```python
import heapq

def k_closest(points, k):
    heap = []                                  # min-heap over negated distance
    for x, y in points:
        d = x * x + y * y                      # no sqrt: squaring preserves order
        if len(heap) < k:
            heapq.heappush(heap, (-d, x, y))
        elif d > -heap[0][0]:                  # beats the current worst of the k
            heapq.heapreplace(heap, (-d, x, y))
    return [(x, y) for _, x, y in heap]
```
O(n log k) time, O(k) space.

### T2 — heapq basics + negate for max-heap
The operations worth memorizing.

```python
import heapq

h = [5, 1, 9, 4]
heapq.heapify(h)                     # O(n), in place; h[0] is the minimum
heapq.heappush(h, 3)
smallest = heapq.heappop(h)          # pops h[0]

maxh = [-x for x in h]               # max-heap = min-heap of negated values
heapq.heapify(maxh)
largest = -heapq.heappop(maxh)       # negate on the way in AND out

top3 = heapq.nlargest(3, h)          # O(n log k); fine for small k
# kth-largest-in-a-stream: push, then heappushpop while len > k — h[0] = kth best
```
heapify O(n); push/pop O(log n); nlargest O(n log k).

### T3 — Two-heap running median
Split the stream into a lower max-half and an upper min-half.

```python
import heapq

class MedianFinder:
    def __init__(self):
        self.lo = []   # max-heap (negated): lower half
        self.hi = []   # min-heap: upper half
        # invariants: every lo <= every hi; len(lo) is len(hi) or len(hi)+1

    def addNum(self, num):
        heapq.heappush(self.lo, -num)                    # route through lo
        heapq.heappush(self.hi, -heapq.heappop(self.lo)) # keeps ordering inv.
        if len(self.hi) > len(self.lo):                  # rebalance sizes
            heapq.heappush(self.lo, -heapq.heappop(self.hi))

    def findMedian(self):
        if len(self.lo) > len(self.hi):
            return -self.lo[0]
        return (-self.lo[0] + self.hi[0]) / 2
```
O(log n) per add, O(1) median, O(n) space.

### T4 — Heap-of-heads k-way merge
Merge k sorted sources by always taking the smallest head.

```python
import heapq

def merge_k_lists(lists):               # lists: array of sorted ListNodes
    heap = [(node.val, i, node) for i, node in enumerate(lists) if node]
    heapq.heapify(heap)                 # (val, i): idx breaks ties, nodes can't
    dummy = tail = ListNode()
    while heap:
        val, i, node = heapq.heappop(heap)
        tail.next = tail = node         # append the smallest head
        if node.next:
            heapq.heappush(heap, (node.next.val, i, node.next))
    return dummy.next
```
O(N log k) time for N total nodes, O(k) space — beats concatenating and sorting.

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| heapify | O(n) | O(1) |
| push / pop / pushpop | O(log n) | O(1) |
| Top-k via size-k heap | O(n log k) | O(k) |
| Full sort instead | O(n log n) | O(n) |
| Two-heap median | O(log n) add, O(1) query | O(n) |
| k-way merge | O(N log k) | O(k) |
| Quickselect (alternative top-k) | O(n) avg | O(1) |

## ⚠️ Traps interviewers probe
- `heapq` is min-only: negate for max, and always put an int index in the tuple before a non-comparable object (ListNode will raise on ties).
- Pushing n items one by one is O(n log n); build a list then `heapify` for O(n).
- Two-heap median: pick which side holds the extra element and route every insert through one heap — otherwise the ordering invariant drifts.
- Size-k heap for "k largest" is a MIN-heap (you evict the smallest of the kept best); flipping that is the classic error.
- find-k-pairs: cap the heap at k and only push (i, j+1) when the popped pair had i == 0 — pushing both children visits every pair twice.
- Task scheduler: the formula answer is `(max_count - 1) * (n + 1) + num_at_max` — the heap simulation is the fallback when they ban the formula.
- Lazy deletion (stale entries) is expected in Dijkstra-style uses — skip popped entries already finalized rather than trying to delete from the middle.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Last Stone Weight | Easy | https://leetcode.com/problems/last-stone-weight/ |
| Kth Largest Element in a Stream | Easy | https://leetcode.com/problems/kth-largest-element-in-a-stream/ |
| K Closest Points to Origin | Medium | https://leetcode.com/problems/k-closest-points-to-origin/ |
| Task Scheduler | Medium | https://leetcode.com/problems/task-scheduler/ |
| Find K Pairs with Smallest Sums | Medium | https://leetcode.com/problems/find-k-pairs-with-smallest-sums/ |
| Merge k Sorted Lists | Hard | https://leetcode.com/problems/merge-k-sorted-lists/ |
| Find Median from Data Stream | Hard | https://leetcode.com/problems/find-median-from-data-stream/ |
