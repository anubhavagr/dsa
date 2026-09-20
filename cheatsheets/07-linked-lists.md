# 07 · Linked Lists
> **Reach for it when:** the input is a chain of nodes, the answer is pointer surgery, and O(1) space rules out copying into an array.

## 🧠 Mental model
Linked lists reward three reflexes: draw the boxes first, use a dummy head whenever the head itself can change, and save `.next` before you overwrite it. Fast/slow pointers convert positional questions (middle, kth-from-end, cycle) into a single pass because a 2× runner's position encodes distance traveled. The cycle math worth memorizing: after the runners meet inside the loop, a pointer reset to the head and the meeting pointer both reach the entrance together — the head-to-entrance distance equals the leftover loop distance. Deep copies and reordering share the interleave trick: weave clone or second-half nodes through the original, fix cross-pointers, then unweave. There is no random access: "sorted" or "kth largest" on a list wants merge sort or a heap (sheet 12), never index arithmetic.

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "reverse the list / a subrange / k-groups" | prev / cur / nxt pointer surgery |
| "kth from end / delete kth / find the middle" | two runners with a fixed gap |
| "does it cycle? where does it start?" | Floyd: meet, then reset-to-head |
| "merge two sorted lists" | dummy head + tail-append the smaller node |
| "deep copy with random pointers" | interleave clones, wire randoms, unweave |
| "sort the list" | merge sort: split at the slow runner, O(n log n) |
| "merge k lists in O(n log k)" | heap of heads (sheet 12) |
| "reorder L0→Ln→L1→Ln−1…" | split at middle, reverse second half, interleave |

## 🛠️ Templates

### T1 — 3-pointer in-place reverse
Reverse a whole list (and, extended, any subrange).
```python
class ListNode:                              # LC's definition, for reference
    def __init__(self, val=0, next=None):
        self.val, self.next = val, next

def reverse_list(head):
    prev = None
    while head:
        nxt = head.next          # save what the flip would orphan
        head.next = prev         # flip the arrow
        prev = head              # advance both, one node per iteration
        head = nxt
    return prev                  # old tail is the new head
```
O(n) time, O(1) space.

### T2 — Dummy head + fast/slow gap of n
Delete the nth-from-end node in one pass.
```python
def remove_nth_from_end(head, n):
    dummy = ListNode(0, head)    # head may change: dummy removes that branch
    fast = slow = dummy
    for _ in range(n):           # open a gap of exactly n between the runners
        fast = fast.next
    while fast.next:             # when fast hits the last node,
        fast = fast.next         # slow sits just before the victim
        slow = slow.next
    slow.next = slow.next.next   # unlink; GC does the rest
    return dummy.next
```
O(n) time, O(1) space.

### T3 — Cycle detect + meeting point math (Floyd)
Detect a cycle and return the node where it begins.
```python
def detect_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:         # meeting point is somewhere inside the loop
            slow = head          # math: head->entrance distance equals the
            while slow is not fast:   # leftover loop distance, so they meet
                slow = slow.next      # exactly at the entrance when both
                fast = fast.next      # step one node at a time
            return slow
    return None
```
O(n) time, O(1) space.

### T4 — Interleave-copy for random pointer
Deep copy with O(1) extra space (no map from old to new).
```python
def copy_random_list(head):
    if not head:
        return None
    cur = head
    while cur:                   # 1) weave clones in: A -> A' -> B -> B' -> ...
        cur.next = Node(cur.val, cur.next)
        cur = cur.next.next
    cur = head
    while cur:                   # 2) each clone's random = original's random.next
        if cur.random:
            cur.next.random = cur.random.next
        cur = cur.next.next
    dummy = tail = Node(0)
    cur = head
    while cur:                   # 3) unweave into two independent lists
        tail.next = cur.next
        tail = tail.next
        cur.next = cur.next.next # restore the original list as we go
        cur = cur.next
    return dummy.next
```
O(n) time, O(1) extra space (the interleaved state is transient).

**merge-k-sorted-lists:** heap of list heads — `heap = [(h.val, i, h) for i, h in enumerate(lists) if h]; heapq.heapify(heap)`, pop-min and push that node's successor. Full heap mechanics on sheet 12 (Heaps).

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| reverse (iterative) | O(n) | O(1) |
| middle / kth-from-end runners | O(n) | O(1) |
| Floyd detect + entrance | O(n) | O(1) |
| merge two sorted lists | O(n + m) | O(1) |
| interleave deep copy | O(n) | O(1) |
| sort-list (merge sort) | O(n log n) | O(log n) recursion |
| merge-k via heap of heads | O(n log k) | O(k) |
| reverse k-group | O(n) | O(1) iterative |

## ⚠️ Traps interviewers probe
- A dummy head costs one line and deletes every "but what if it's the head?" branch — default to it on any problem that can remove or re-link the head.
- The n-gap trick: advance fast n times BEFORE the joint walk, so slow stops one node before the victim — otherwise you're deleting the wrong node.
- Guard with `while fast and fast.next`; dropping either half of the guard None-crashes on even/odd lengths and single-node lists.
- Copy-random's key line is `cur.next.random = cur.random.next` — the clone always sits one `.next` ahead of its original.
- sort-list: sever the halves (`prev.next = None` where prev trails the slow runner) or the recursion never shrinks and you stack-overflow.
- Heap tuples need an index tie-breaker: ListNodes aren't comparable, so `(val, node)` raises TypeError the moment two heads tie on value.
- reverse-k-group: verify k nodes remain before reversing each group, and reconnect the PREVIOUS group's tail to the new sub-head — losing that pointer orphans the rest of the list.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Reverse Linked List | Easy | https://leetcode.com/problems/reverse-linked-list/ |
| Merge Two Sorted Lists | Easy | https://leetcode.com/problems/merge-two-sorted-lists/ |
| Linked List Cycle | Easy | https://leetcode.com/problems/linked-list-cycle/ |
| Remove Nth Node From End of List | Medium | https://leetcode.com/problems/remove-nth-node-from-end-of-list/ |
| Reorder List | Medium | https://leetcode.com/problems/reorder-list/ |
| Copy List with Random Pointer | Medium | https://leetcode.com/problems/copy-list-with-random-pointer/ |
| Sort List | Medium | https://leetcode.com/problems/sort-list/ |
| Merge k Sorted Lists | Hard | https://leetcode.com/problems/merge-k-sorted-lists/ |
| Reverse Nodes in k-Group | Hard | https://leetcode.com/problems/reverse-nodes-in-k-group/ |
