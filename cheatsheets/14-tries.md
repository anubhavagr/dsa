# 14 · Tries
> **Reach for it when:** many strings share structure and queries are about prefixes — pay one node per character, walk queries in O(len).

## 🧠 Mental model
A trie shares prefixes physically: N words of length L cost O(total characters) of space and every membership/prefix test is a single root-to-node walk, independent of dictionary size. The node is minimal — a `children` dict (char → node) plus whatever the query needs stamped on it (`is_word`, a word string, a count). The wildcard '.' query replaces one dict lookup with a fan-out DFS over children. The underrated half is pruning: in board-search problems, if the current cell's character has no child, an entire subtree of candidate words dies instantly — and deleting emptied leaves as words are found keeps the trie shrinking as the search proceeds. A plain `set` beats a trie for exact-match lookups; reach for the trie only when prefixes, wildcard walks, or early termination matter.

## 🚦 Signals → pattern
| If the problem says… | Think… |
|---|---|
| "insert words, query prefixes" | dict-based TrieNode insert / startsWith |
| "'.' matches any one character" | DFS fan-out at each wildcard |
| "find all dictionary words on a board" | board DFS walking the trie + leaf pruning |
| "autocomplete / longest common prefix" | walk until branching, collect subtree |
| "count words with this prefix" | store per-node pass_counts, not just is_word |
| "many exact-match lookups only" | a set suffices — say so, then justify the trie |

## 🛠️ Templates
### T1 — Dict-based TrieNode: insert / search / startsWith
The foundation everything else reuses.

```python
class TrieNode:
    __slots__ = ("children", "is_word", "word")
    def __init__(self):
        self.children = {}            # char -> TrieNode
        self.is_word = False          # a full word terminates here
        self.word = None              # stamp for search problems (optional)

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for ch in word:
            node = node.children.setdefault(ch, TrieNode())
        node.is_word = True           # mark the terminal node, children may live on

    def search(self, word):           # exact word only
        node = self._walk(word)
        return node is not None and node.is_word

    def startsWith(self, prefix):     # any word with this prefix
        return self._walk(prefix) is not None

    def _walk(self, s):               # follow chars; None if path dies
        node = self.root
        for ch in s:
            node = node.children.get(ch)
            if node is None:
                return None
        return node
```
O(L) per operation, O(total chars) space.

### T2 — Wildcard '.' DFS search
Add-and-search-words: fan out at every dot.

```python
class WordDictionary:
    def __init__(self):
        self.root = TrieNode()

    def addWord(self, word):
        node = self.root
        for ch in word:
            node = node.children.setdefault(ch, TrieNode())
        node.is_word = True

    def search(self, word):
        def dfs(i, node):
            if i == len(word):
                return node.is_word   # path exists — but is it a whole word?
            ch = word[i]
            if ch == ".":             # wildcard: try every child
                return any(dfs(i + 1, c) for c in node.children.values())
            child = node.children.get(ch)
            return child is not None and dfs(i + 1, child)
        return dfs(0, self.root)
```
O(L) without dots, O(26^d · L) worst case with d dots; space O(total chars).

### T3 — Board DFS with trie-walk + node removal pruning
Word search II: walk the trie as you walk the board.

```python
def find_words(board, words):
    root = TrieNode()
    for w in words:                            # build trie, stamp terminal nodes
        node = root
        for ch in w:
            node = node.children.setdefault(ch, TrieNode())
        node.is_word, node.word = True, w

    m, n, found = len(board), len(board[0]), set()

    def dfs(r, c, node):
        ch = board[r][c]
        nxt = node.children.get(ch)
        if nxt is None:
            return                             # PRUNE: no word continues this way
        if nxt.is_word:
            found.add(nxt.word)
            nxt.is_word = False                # report each word only once
        board[r][c] = "#"                      # in-place visited mark
        for dr, dc in ((1,0),(-1,0),(0,1),(0,-1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < m and 0 <= nc < n and board[nr][nc] != "#":
                dfs(nr, nc, nxt)
        board[r][c] = ch                       # restore on backtrack
        if not nxt.children and not nxt.is_word:
            del node.children[ch]              # PRUNE: leaf is dead — shrink trie

    for r in range(m):
        for c in range(n):
            dfs(r, c, root)
    return list(found)
```
O(M·N·4·3^(L-1)) worst case but pruning makes it practical; the leaf deletion is the part interviewers want named.

## ⏱️ Complexities
| Operation / pattern | Time | Space |
|---|---|---|
| insert / search / startsWith | O(L) | O(total chars) nodes |
| Wildcard DFS | O(L) clean, O(26^d·L) dotted | O(L) recursion |
| Board DFS + trie pruning | ~O(MN·3^L) worst, far less pruned | O(total chars) |
| Plain set lookup (alternative) | O(L) hash | O(total chars) |

## ⚠️ Traps interviewers probe
- `is_word` is independent of having children: "be" terminates while "bee" continues — both facts live on the same chain.
- search must check `is_word` at the end; startsWith must NOT — conflating them fails both problems.
- Word search II can report the same word via several paths: dedupe by clearing `is_word` (or a result set), not by mutating the board permanently.
- Forgetting to restore `board[r][c]` on backtrack corrupts every later path through the cell.
- Without leaf deletion, the trie never shrinks and you re-walk dead branches — this pruning is the difference between TLE and accepted.
- `setdefault(ch, TrieNode())` both reads and inserts in one step; a plain `children[ch] = ...` overwrites and silently orphans subtrees.
- Array-of-26 children is a constant-factor optimization to offer, not to open with — dict nodes are the interview default.

## 🔗 Problems in this plan
| Problem | Difficulty | Link |
|---|---|---|
| Implement Trie (Prefix Tree) | Medium | https://leetcode.com/problems/implement-trie-prefix-tree/ |
| Design Add and Search Words Data Structure | Medium | https://leetcode.com/problems/design-add-and-search-words-data-structure/ |
| Word Search II | Hard | https://leetcode.com/problems/word-search-ii/ |
