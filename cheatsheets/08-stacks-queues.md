# 08 · Stacks, Queues & Monotonic structures
> **Reach for it when:** the structure is nesting (brackets, expressions) or the question is "nearest greater/smaller" — a stack or deque turns it into one pass.

## 🧠 Mental model
A stack is the machine for nesting: push state on the way in, pop and combine on the way out — brackets, RPN, infix expressions, and run-length decoding are all that one shape. Monotonic stacks add a single rule that unlocks a whole family: **while the new element violates the stack's order, pop — each popped element just found its answer.** That converts next-greater-element, daily temperatures, and histogram rectangles into one O(n) pass where every index is pushed and popped at most once. For sliding-window maxima, the same idea becomes a deque of indices whose values strictly decrease: the front is always the window's max, and the back is where doomed smaller values go. Store indices, not values, by default — the answer usually needs the distance (i − j) or the popped element's extent, which values alone can't give you.

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "are these brackets balanced?" | pair-matching stack |
| "stack with getMin in O(1)" | (val, min-so-far) pairs |
| "evaluate RPN / infix expression" | value stack / deferred term stack |
| "first greater element to the right / days until warmer" | monotonic decreasing stack of indices |
| "largest rectangle under a histogram" | increasing stack + sentinel flush |
| "max of every window of size k" | monotonic deque |
| "nested repeats k[encoded]" | stack of (prefix, count) |
| "minimum removals for valid parens" | index stack marks offenders, two directions |

## 🛠️ Templates

### T1 — Pair-matching stack
Balanced-brackets and all "closest opener" logic.
```python
def is_valid(s):
    pairs = {')': '(', ']': '[', '}': '{'}
    stack = []
    for ch in s:
        if ch in pairs:
            if not stack or stack.pop() != pairs[ch]:
                return False        # nothing to close, or the wrong opener
        else:
            stack.append(ch)        # openers wait on top
    return not stack                # leftovers = unclosed openers
```
O(n) time, O(n) space.

### T2 — Min-stack via (val, min) pairs
Every operation O(1), including the minimum.
```python
class MinStack:
    def __init__(self):
        self.stack = []                    # (value, min_so_far) per entry

    def push(self, val):
        m = val if not self.stack else min(val, self.stack[-1][1])
        self.stack.append((val, m))

    def pop(self):
        self.stack.pop()                   # the pair carries the min away

    def top(self):
        return self.stack[-1][0]

    def get_min(self):
        return self.stack[-1][1]           # O(1): min travels with the top
```
All ops O(1) time, O(n) space.

### T3 — Index-stack for calculators / min-remove
Store indices so pops can measure distance or mark deletions.
```python
def calculate(s):                          # basic-calculator-ii, no parens
    stack, num, op = [], 0, '+'
    for i, ch in enumerate(s):
        if ch.isdigit():
            num = num * 10 + int(ch)       # multi-digit accumulation
        if ch in '+-*/' or i == len(s) - 1:    # flush on op OR last char
            if op == '+':
                stack.append(num)
            elif op == '-':
                stack.append(-num)
            elif op == '*':
                stack.append(stack.pop() * num)
            else:
                stack.append(int(stack.pop() / num))  # trunc toward 0
            op, num = ch, 0
    return sum(stack)
```
O(n) time, O(n) space.

### T4 — Monotonic decreasing stack
Next-greater answers and histogram rectangles, one shape.
```python
def daily_temperatures(temps):
    res = [0] * len(temps)
    stack = []                     # indices whose answer is still unknown;
    for i, t in enumerate(temps):  # their temps are strictly decreasing
        while stack and temps[stack[-1]] < t:
            j = stack.pop()        # t is j's first warmer day
            res[j] = i - j         # indices -> distance comes free
        stack.append(i)
    return res

def largest_rectangle(heights):
    stack, best = [], 0            # indices, heights increasing
    for i, h in enumerate(heights + [0]):   # sentinel 0 flushes at the end
        while stack and heights[stack[-1]] > h:
            height = heights[stack.pop()]
            left = stack[-1] if stack else -1   # first bar strictly shorter
            best = max(best, height * (i - left - 1))
        stack.append(i)
    return best
```
Each O(n) time — every index pushed and popped once, O(n) space.

### T5 — Monotonic deque (sliding window max)
Max of every window with two distinct eviction rules.
```python
from collections import deque

def max_sliding_window(nums, k):
    dq = deque()                   # indices, values decreasing: front = max
    out = []
    for i, x in enumerate(nums):
        while dq and nums[dq[-1]] <= x:    # back eviction: dominated values
            dq.pop()
        dq.append(i)
        if dq[0] <= i - k:                 # front eviction: left the window
            dq.popleft()
        if i >= k - 1:
            out.append(nums[dq[0]])
    return out
```
O(n) time — each index enters and leaves the deque once — O(k) space.

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| pair matching | O(n) | O(n) |
| min-stack (all ops) | O(1) | O(n) |
| RPN / calculator | O(n) | O(n) |
| next-greater via monotonic stack | O(n) | O(n) |
| largest rectangle in histogram | O(n) | O(n) |
| sliding window max via deque | O(n) | O(k) |
| decode-string | O(output) | O(depth) |
| naive per-window rescan | O(n·k) | O(1) |

## ⚠️ Traps interviewers probe
- RPN division truncates toward zero: `int(a / b)`, never `a // b` — floor division fails the moment a negative appears.
- The calculator flushes on the LAST character too, not only on operators; missing that drops the final term silently.
- Histogram: append the sentinel 0; after popping, width is `i - left - 1` where left is the stack top AFTER the pop (-1 if empty).
- The deque's two evictions are different rules: pop dominated values off the BACK on insert, popleft expired indices off the FRONT — mixing them corrupts the max.
- Monotonic stacks store indices so `res[j] = i - j` computes distances for free; storing values forces a second pass.
- min-stack pop() discards the whole pair — rescanning for the new min is the O(n) answer they're fishing for.
- decode-string: push (current, count) at '[', pop-and-merge at ']'; counts above 9 ("100[leetcode]") are the classic WA.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Valid Parentheses | Easy | https://leetcode.com/problems/valid-parentheses/ |
| Min Stack | Medium | https://leetcode.com/problems/min-stack/ |
| Evaluate Reverse Polish Notation | Medium | https://leetcode.com/problems/evaluate-reverse-polish-notation/ |
| Daily Temperatures | Medium | https://leetcode.com/problems/daily-temperatures/ |
| Largest Rectangle in Histogram | Hard | https://leetcode.com/problems/largest-rectangle-in-histogram/ |
| Sliding Window Maximum | Hard | https://leetcode.com/problems/sliding-window-maximum/ |
| Basic Calculator II | Medium | https://leetcode.com/problems/basic-calculator-ii/ |
| Decode String | Medium | https://leetcode.com/problems/decode-string/ |
| Minimum Remove to Make Valid Parentheses | Medium | https://leetcode.com/problems/minimum-remove-to-make-valid-parentheses/ |
