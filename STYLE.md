# ✍️ Writing style contract — applies to every file in this repo

Read this before writing or editing any content (day files, cheatsheets, design sessions, docs). The reader is a working engineer learning under time pressure. If a sentence needs the author nearby to explain it, it is broken.

## Rules

1. **Plain, explanatory, technically precise English.** Every sentence must be understandable on first read by someone who does not already know the answer. Correct terminology is required; in-group shorthand is banned.
2. **No aphorisms, no code-poetry.** Banned style: "the rescue", "order, pre-paid", "the bend stays home", "states travel as tuples". If a metaphor helps, use it AND explain it in the same sentence: "the parent gets only a straight path (a *leg*) — the best *bending* path already ended at this node and cannot extend upward."
3. **Tables carry complete thoughts.** A cell like "the target" or "the thing to eliminate" explains nothing. Write the actual meaning: "Too slow at n = 10,000 (100 million steps) — remove the inner loop."
4. **No filler.** Every line must teach something: a mechanism, a decision rule, a number, a failure mode. Delete anything that only sounds motivating.
5. **Define terms on first use.** First mention of an invariant, a sentinel, amortized cost, etc. gets a one-clause definition. After that the term is free to use.
6. **Explain why, not just what.** "Walk capacity in reverse" is incomplete; "walk capacity in reverse so dp[s − w] still holds the previous item's row — forward order would reuse the current item and turn 0/1 knapsack into unbounded knapsack" is complete.
7. **Quantify when possible.** Real numbers beat adjectives: "n = 10⁵ rules out O(n²) — that's 10¹⁰ steps" beats "very slow".
8. **Preserve the formats that software parses.** Never modify checkbox lines (`- [ ]` / `- [x]` and their exact `- [ ] N. [Title](url) (Difficulty)` problem form), `## ` section headers, code blocks, URLs, or LeetCode slugs. Prose and bullet *text* may change; structure may not.

## Good vs bad (from a real fix in this repo)

- ❌ `| hash lookup inside the loop | O(n) total | the rescue |`
- ✅ `| Hash map / set lookup inside a single loop | O(n) average | The map answers "have I seen x before?" in constant time, replacing the inner loop entirely. You pay with extra memory. |`

- ❌ `- **Interview soundbite:** "I return a leg and a best — the leg feeds my parent, the bend stays home."`
- ✅ `- **How to say it in an interview:** "Each node returns two values to its parent: the longest downward path starting at this node, and the best answer found anywhere in this subtree. Only the first may be extended by the parent, because a path through both children already turns at this node and cannot grow upward."`
