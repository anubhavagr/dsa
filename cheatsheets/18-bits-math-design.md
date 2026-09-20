# 18 · Bit Manipulation · Math & Random · Design
> **Reach for it when:** the trick is arithmetic on bits, a classic math identity, uniform randomness — or a data structure whose ops must all be O(1) by construction.

## 🧠 Mental model
**Bits:** treat an int as a bit vector: XOR cancels pairs (a^a=0, order-free), `n & (n-1)` clears the lowest set bit, `n & -n` isolates it, and addition is `(a^b)` (sum) looped with `((a&b)<<1)` (carries) until carries vanish — masked to 32 bits, because Python ints never wrap. **Math:** three shapes cover it — binary search over the answer space (sqrt), exponentiation by squaring (pow), or index arithmetic replacing cleverness (rotation = transpose + reverse; spiral = shrinking bounds). **Random:** correctness means uniformity: prefix-sum buckets + bisect for weighted picks, Fisher–Yates for permutations. **Design:** every O(1) requirement is a dict for lookup welded to a structure for order/recency/randomness — LRU = dict + doubly-linked list, O(1) random set = dict + array with swap-delete, running median = two heaps (sheet 12).

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "every element appears twice except one" | XOR the whole array (Bits) |
| "count set bits / is power of two / reverse bits" | `n & (n-1)` clears lowest bit; shift-and-OR 32x for reversal (Bits) |
| "add two numbers without +" | XOR + carry loop, masked to 32 bits (Bits) |
| "compute x^n / sqrt fast" | square-and-multiply / binary search (Math) |
| "rotate a matrix 90°, spiral order" | transpose + reverse / shrinking bounds (Math) |
| "pick index proportional to weights" | prefix sums + bisect_left (Random) |
| "shuffle uniformly / reset" | Fisher–Yates backward swap (Random) |
| "get and put in O(1), evict least recent" | dict + doubly-linked list (Design) |
| "insert/remove/getRandom all O(1)" | dict + array with swap-delete (Design) |

## 🛠️ Templates

**Bits**

### T1 — XOR family (single number)
Every paired value cancels; the loner survives.

```python
def single_number(nums):
    x = 0
    for n in nums:          # a^a = 0, a^0 = a, XOR commutes -> order-free
        x ^= n
    return x                # pairs annihilate, the singleton remains
```
O(n) time, O(1) space.

### T2 — n & (n-1): clear / isolate the lowest set bit
Popcount, powers of two, lowest bit.

```python
def hamming_weight(n):
    count = 0
    while n:                # each loop kills exactly one set bit
        n &= n - 1          # clear lowest set bit
        count += 1
    return count
# n & -n ISOLATES the lowest bit; counting-bits DP: bits[i] = bits[i>>1] + (i&1)
# power of two: n > 0 and n & (n - 1) == 0
```
O(k) time for k set bits, O(1) space; counting-bits is O(n).

### T3 — Add via carries (sum of two integers)
No `+` allowed: sum bits and carry bits, loop.

```python
def get_sum(a, b):
    mask = 0xFFFFFFFF                       # 32-bit two's-complement sandbox
    while b:
        a, b = (a ^ b) & mask, ((a & b) << 1) & mask   # sum / carry
    return a if a <= 0x7FFFFFFF else ~(a ^ mask)       # re-sign Python ints
```
O(1) iterations in practice (≤ 32), O(1) space.

**Math & Random**

### T4 — Fast pow, iterative
x^n in O(log n) multiplications.

```python
def my_pow(x, n):
    if n < 0:
        x, n = 1 / x, -n          # negative exponent: invert once
    res = 1
    while n:                      # n in binary = which squares to multiply
        if n & 1:
            res *= x              # this binary digit is set: take the square
        x *= x
        n >>= 1
    return res
```
O(log n) time, O(1) space.

### T5 — Transpose + reverse rotation
Rotate an n×n matrix 90° clockwise, in place.

```python
def rotate(matrix):
    n = len(matrix)
    for i in range(n):                  # transpose over the main diagonal
        for j in range(i + 1, n):
            matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
    for row in matrix:                  # reverse each row -> clockwise
        row.reverse()
# Counter-clockwise: reverse rows first, then transpose. Spiral: shrink 4 bounds.
```
O(n²) time, O(1) space.

### T6 — Prefix + bisect weighted random
Pick index i with probability w[i]/total.

```python
from bisect import bisect_left
from itertools import accumulate
from random import randint

class Solution:
    def __init__(self, w):
        self.prefix = list(accumulate(w))     # bucket boundaries per index
    def pickIndex(self):
        target = randint(1, self.prefix[-1])  # uniform over total weight
        return bisect_left(self.prefix, target)   # first bucket reaching it
```
O(n) build, O(log n) per pick, O(n) space. sqrtx: binary search `lo, hi = 0, x` on `mid*mid <= x`.

### T7 — Fisher–Yates shuffle
Every permutation equally likely.

```python
from random import randint

class Solution:
    def __init__(self, nums):
        self.base, self.cur = nums, list(nums)
    def reset(self):
        self.cur = list(self.base)            # fresh copy, never shuffle base
        return self.cur
    def shuffle(self):
        for i in range(len(self.cur) - 1, 0, -1):
            j = randint(0, i)                 # uniform in [0, i] — the crux
            self.cur[i], self.cur[j] = self.cur[j], self.cur[i]
        return self.cur
```
O(n) shuffle/reset, O(n) space. Sampling i from [0, n) instead of [0, i] is biased — that's the probe.

**Design**

### T8 — dict + doubly-linked-list LRU
O(1) get/put with true eviction order.

```python
class Node:
    __slots__ = ("key", "val", "prev", "next")
    def __init__(self, key=0, val=0):
        self.key, self.val = key, val

class LRUCache:
    def __init__(self, capacity):
        self.cap, self.map = capacity, {}
        self.head, self.tail = Node(), Node()     # sentinels; MRU after head
        self.head.next, self.tail.prev = self.tail, self.head

    def _unlink(self, n):
        n.prev.next, n.next.prev = n.next, n.prev

    def _push_front(self, n):                     # insert as most-recent
        n.next, n.prev = self.head.next, self.head
        self.head.next.prev = n
        self.head.next = n

    def get(self, key):
        if key not in self.map:
            return -1
        n = self.map[key]
        self._unlink(n); self._push_front(n)      # refresh recency
        return n.val

    def put(self, key, value):
        if key in self.map:
            self._unlink(self.map[key])
        n = Node(key, value)
        self.map[key] = n
        self._push_front(n)
        if len(self.map) > self.cap:              # evict LRU (before tail)
            lru = self.tail.prev
            self._unlink(lru)
            del self.map[lru.key]                 # node carries its key so eviction can delete it from the dict
```
O(1) per op. The OrderedDict shortcut (`move_to_end` + `popitem(last=False)`) is 5 lines — mention it, then hand-roll, because interviewers expect the list. LFU-cache adds a freq → DLL-of-nodes layer plus a min-freq pointer.

### T9 — array + dict swap-delete for O(1) random
Insert/delete/getRandom all O(1).

```python
import random

class RandomizedSet:
    def __init__(self):
        self.vals, self.idx = [], {}      # array: uniform pick; dict: O(1) locate
    def insert(self, val):
        if val in self.idx:
            return False
        self.idx[val] = len(self.vals)
        self.vals.append(val)
        return True
    def remove(self, val):
        if val not in self.idx:
            return False
        last, i = self.vals[-1], self.idx[val]    # swap val with the last...
        self.vals[i], self.idx[last] = last, i
        self.vals.pop()                           # ...then truncate: O(1)
        del self.idx[val]
        return True
    def getRandom(self):
        return random.choice(self.vals)           # uniform because array is dense
```
O(1) per op. Design-twitter/design-underground-system are the same discipline: dicts indexing queues/timestamps, no cleverness needed.

### T10 — Two-heap median (pointer)
Running median is the sheet-12 T3 pattern (max-half + min-half, rebalance ±1) — see `12-heaps.md`, not repeated here.

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| XOR cancel / hamming weight | O(n) / O(k bits) | O(1) |
| Add-via-carries | O(1) (≤32 iters) | O(1) |
| Fast pow | O(log n) | O(1) |
| Rotate matrix / spiral | O(n²) | O(1) |
| Weighted random pick | O(log n) per pick | O(n) |
| Fisher–Yates shuffle | O(n) | O(n) |
| LRU get/put | O(1) | O(capacity) |
| RandomizedSet ops | O(1) | O(n) |

## ⚠️ Traps interviewers probe
- Python ints are unbounded: two's-complement tricks need an explicit `0xFFFFFFFF` mask and a re-sign step at the end — the standard `get_sum` bug is omitting it.
- `n & (n-1)` clears the lowest set bit, `n & -n` isolates it; power-of-two is `n > 0 and n & (n-1) == 0`.
- reverse-bits must process all 32 positions even after n's own bits run out (shift in zeros).
- sqrt/pow: binary search on `mid*mid <= x` with the `x < 2` guard; fast-pow handles negative n by one inversion, not per-step.
- rotate: transpose-then-reverse-rows is clockwise; doing it in the other order gives counter-clockwise — state which you're doing.
- Weighted pick: `randint(1, total)` + `bisect_left` matches bucket boundaries; `bisect_right` with a 0-based target is the off-by-one they fish for.
- LRU: the node must store its KEY so eviction can delete from the dict; with only values, `del map[?]` is impossible. getRandom requires a dense array — hence swap-delete.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Single Number | Easy | https://leetcode.com/problems/single-number/ |
| Counting Bits | Easy | https://leetcode.com/problems/counting-bits/ |
| Number of 1 Bits | Easy | https://leetcode.com/problems/number-of-1-bits/ |
| Reverse Bits | Easy | https://leetcode.com/problems/reverse-bits/ |
| Happy Number | Easy | https://leetcode.com/problems/happy-number/ |
| Sqrt(x) | Easy | https://leetcode.com/problems/sqrtx/ |
| Sum of Two Integers | Medium | https://leetcode.com/problems/sum-of-two-integers/ |
| Rotate Image | Medium | https://leetcode.com/problems/rotate-image/ |
| Spiral Matrix | Medium | https://leetcode.com/problems/spiral-matrix/ |
| Pow(x, n) | Medium | https://leetcode.com/problems/powx-n/ |
| Random Pick with Weight | Medium | https://leetcode.com/problems/random-pick-with-weight/ |
| Shuffle an Array | Medium | https://leetcode.com/problems/shuffle-an-array/ |
| LRU Cache | Medium | https://leetcode.com/problems/lru-cache/ |
| Insert Delete GetRandom O(1) | Medium | https://leetcode.com/problems/insert-delete-getrandom-o1/ |
| Design Twitter | Medium | https://leetcode.com/problems/design-twitter/ |
| Design Underground System | Medium | https://leetcode.com/problems/design-underground-system/ |
| LFU Cache | Hard | https://leetcode.com/problems/lfu-cache/ |
