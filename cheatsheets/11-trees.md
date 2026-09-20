# 11 · Trees & BSTs
> **Reach for it when:** the data is hierarchical or already ordered — one recursive question per node is the whole game.

## 🧠 Mental model
Every tree problem is answered by one question per node: "what do I need from my children to compute my contribution?" Aggregates flowing up (height, sums, diameters) are post-order; decisions flowing down (validity bounds, LCA splits, serialization) are pre-order; anything about rows or left-to-right order is BFS with a frozen level size. BSTs add one invariant — everything in the left subtree < node < everything in the right subtree — which turns search into a single walk and validation into passing (lo, hi) ranges down, never just the parent's value. Most bugs are: forgetting the empty tree, confusing height with depth (edges vs nodes), and re-visiting what an iterative stack already remembers.

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "return values in inorder / preorder / level order" | pick the matching traversal template |
| "depth / height / is it balanced" | post-order returns height; parent combines |
| "diameter / max path sum — path bends at a node" | post-order pair: (best-through-node, height-to-parent) |
| "is this a valid BST" | pass (lo, hi) bounds down, not parent-only checks |
| "kth smallest in a BST" | inorder = sorted order; stop at k |
| "view from the right / per-level statistics" | BFS with frozen `len(q)` per level |
| "rebuild the tree from traversals" | preorder root + inorder split via index map |
| "pick nodes, no two adjacent" | (take, skip) tree DP returned as a pair |
| "next-right pointers / zigzag" | level walk with a dummy head per level |

## 🛠️ Templates
### T1 — The three orders (+ iterative inorder)
All traversal questions; iterative inorder is the one to know without recursion.

```python
# class TreeNode: val, left, right
def preorder(root):            # root, left, right — decisions flowing down
    return [root.val] + preorder(root.left) + preorder(root.right) if root else []

def inorder(root):             # left, root, right — a BST yields sorted order
    return inorder(root.left) + [root.val] + inorder(root.right) if root else []

def postorder(root):           # left, right, root — children before parent
    return postorder(root.left) + postorder(root.right) + [root.val] if root else []

def inorder_iter(root):        # explicit stack, same order as inorder()
    out, stack, cur = [], [], root
    while cur or stack:
        while cur:             # slide left, remembering the path
            stack.append(cur)
            cur = cur.left
        cur = stack.pop()      # leftmost unvisited node
        out.append(cur.val)    # its left subtree is finished — invariant
        cur = cur.right
    return out
```
All four: O(n) time, O(h) stack space (O(n) worst for a skew).

### T2 — BFS by levels (right-side view)
Anything per-level: level-order lists, right view, zigzag, averages.

```python
from collections import deque

def right_side_view(root):
    view, q = [], deque([root] if root else [])
    while q:
        for _ in range(len(q)):        # freeze level size = exactly one level
            node = q.popleft()
            last = node.val            # overwritten; ends at the rightmost node
            if node.left:  q.append(node.left)
            if node.right: q.append(node.right)
        view.append(last)
    return view
```
O(n) time, O(w) space where w = max level width.

### T3 — Post-order (depth, best) tuple pattern
Diameter, balance, max-path-sum: return what the parent needs, record what the world needs.

```python
def diameter_of_binary_tree(root):
    best = 0
    def depth(node):                    # returns height in edges below node
        nonlocal best
        if not node:
            return 0
        l, r = depth(node.left), depth(node.right)
        best = max(best, l + r)         # best path BENDING here (no +1: edges)
        return 1 + max(l, r)            # height handed to the parent
    depth(root)
    return best
```
O(n) time, O(h) recursion space. Balanced-binary-tree returns `abs(l-r) <= 1` up as validity; max-path-sum returns `max(0, l, r) + node.val` to clamp negative arms.

### T4 — BST validation with min/max bounds
Prove the invariant, not just local order.

```python
def is_valid_bst(root):
    def ok(node, lo, hi):               # node.val must lie in open (lo, hi)
        if not node:
            return True
        if not (lo < node.val < hi):
            return False
        return ok(node.left, lo, node.val) and ok(node.right, node.val, hi)
    return ok(root, float('-inf'), float('inf'))
```
O(n) time, O(h) space; kth-smallest/LCA-in-BST reuse the same walk in O(h).

### T5 — Preorder + inorder construction with index map
Rebuild a unique binary tree from its traversals.

```python
def build_tree(preorder, inorder):
    idx = {v: i for i, v in enumerate(inorder)}    # O(1) root location
    def build(pre_l, pre_r, in_l, in_r):
        if pre_l > pre_r:
            return None
        root = TreeNode(preorder[pre_l])
        m = idx[root.val]                          # inorder split point
        left_len = m - in_l                        # width of the left subtree
        root.left  = build(pre_l + 1, pre_l + left_len, in_l, m - 1)
        root.right = build(pre_l + left_len + 1, pre_r, m + 1, in_r)
        return root
    return build(0, len(preorder) - 1, 0, len(inorder) - 1)
```
O(n) time (each node built once), O(n) space for the map.

### T6 — (take, skip) tree DP
House Robber III: choose independent-set style maxima on a tree.

```python
def rob(root):
    def pair(node):                    # (best if we TAKE node, best if we skip)
        if not node:
            return (0, 0)
        lt, ls = pair(node.left)
        rt, rs = pair(node.right)
        return (node.val + ls + rs,          # take: children must be skipped
                max(lt, ls) + max(rt, rs))   # skip: children free to choose
    return max(pair(root))
```
O(n) time, O(h) space — no memo needed because each subtree is visited once.

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| Any traversal (recursive or stack) | O(n) | O(h) |
| BFS by levels | O(n) | O(w) |
| (depth, best) post-order pattern | O(n) | O(h) |
| BST search / insert / kth / LCA | O(h) avg, O(n) worst | O(1) / O(h) |
| BST validation with bounds | O(n) | O(h) |
| Build from preorder + inorder | O(n) | O(n) |
| (take, skip) tree DP | O(n) | O(h) |

## ⚠️ Traps interviewers probe
- Height in edges vs nodes: diameter counts edges, max-depth counts nodes — say which you mean.
- Validating a BST against only the parent misses grandchild range violations (e.g. 5 → right 6 → left 3); bounds, always.
- LCA of a general tree is the "split point": recurse both sides, return the node where both sides come back non-None.
- Freeze `len(q)` before popping a level; popping until the queue empties merges levels.
- Deep skewed trees hit Python's ~1000-frame recursion limit — the iterative version is the follow-up they want.
- Serialization: preorder with an explicit null marker is easiest to re-parse with an iterator; inorder alone cannot rebuild a tree.
- Sorted list → height-balanced BST: advance a slow/fast pointer to the middle (or index lo/hi over the array) — never insert greedily.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Binary Tree Inorder Traversal | Easy | https://leetcode.com/problems/binary-tree-inorder-traversal/ |
| Maximum Depth of Binary Tree | Easy | https://leetcode.com/problems/maximum-depth-of-binary-tree/ |
| Diameter of Binary Tree | Easy | https://leetcode.com/problems/diameter-of-binary-tree/ |
| Balanced Binary Tree | Easy | https://leetcode.com/problems/balanced-binary-tree/ |
| Range Sum of BST | Easy | https://leetcode.com/problems/range-sum-of-bst/ |
| Binary Tree Level Order Traversal | Medium | https://leetcode.com/problems/binary-tree-level-order-traversal/ |
| Binary Tree Right Side View | Medium | https://leetcode.com/problems/binary-tree-right-side-view/ |
| Count Good Nodes in Binary Tree | Medium | https://leetcode.com/problems/count-good-nodes-in-binary-tree/ |
| Validate Binary Search Tree | Medium | https://leetcode.com/problems/validate-binary-search-tree/ |
| Kth Smallest Element in a BST | Medium | https://leetcode.com/problems/kth-smallest-element-in-a-bst/ |
| Lowest Common Ancestor of a Binary Search Tree | Medium | https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/ |
| Lowest Common Ancestor of a Binary Tree | Medium | https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/ |
| Construct Binary Tree from Preorder and Inorder Traversal | Medium | https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/ |
| House Robber III | Medium | https://leetcode.com/problems/house-robber-iii/ |
| Populating Next Right Pointers in Each Node | Medium | https://leetcode.com/problems/populating-next-right-pointers-in-each-node/ |
| Convert Sorted List to Binary Search Tree | Medium | https://leetcode.com/problems/convert-sorted-list-to-binary-search-tree/ |
| Binary Tree Maximum Path Sum | Hard | https://leetcode.com/problems/binary-tree-maximum-path-sum/ |
| Serialize and Deserialize Binary Tree | Hard | https://leetcode.com/problems/serialize-and-deserialize-binary-tree/ |
