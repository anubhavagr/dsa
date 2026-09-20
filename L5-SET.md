# 🎖️ L5-SET — the post-program Google hardening set (~26 problems)

**Use:** the 93-day program ends Dec 27 with ~225 unique problems solved. This set is the continuation lap for interview season (late Dec → Feb): Google-grade problems with a twist, grouped by the pattern they stress. Work 3–4/week between interviews using [templates/day-template.md](templates/day-template.md); every solve gets the full L5 treatment — timed, narrated, follow-up variant answered. All links free-tier, no premium.

## Google parsing / simulation discipline
| # | Problem | Difficulty | Why it's L5 |
|---|---|---|---|
| 1 | [Text Justification](https://leetcode.com/problems/text-justification/) | Hard | Greedy line packing with exact-width simulation — Google's favorite "do you sweat details" problem |
| 2 | [Integer to English Words](https://leetcode.com/problems/integer-to-english-words/) | Hard | *(if not already done Day 76)* group-by-thousands decomposition |
| 3 | [Robot Bounded In Circle](https://leetcode.com/problems/robot-bounded-in-circle/) | Medium | State after k instructions — invariant math, no simulation of unbounded time |

## Sweep / streaming state
| # | Problem | Difficulty | Why it's L5 |
|---|---|---|---|
| 4 | [Stock Price Fluctuation](https://leetcode.com/problems/stock-price-fluctuation/) | Medium | Timestamp-keyed stream: hashmap + two heaps or sorted container |
| 5 | [Detect Squares](https://leetcode.com/problems/detect-squares/) | Medium | Counting structure + point-pair enumeration |
| 6 | [My Calendar II](https://leetcode.com/problems/my-calendar-ii/) | Medium | Double-booking boundary sweep |
| 7 | [My Calendar III](https://leetcode.com/problems/my-calendar-iii/) | Hard | Maximum concurrent intervals — sweep line with a map |

## Binary search on hard answer spaces
| # | Problem | Difficulty | Why it's L5 |
|---|---|---|---|
| 8 | [Find K-th Smallest Pair Distance](https://leetcode.com/problems/find-k-th-smallest-pair-distance/) | Medium | Two-level binary search: value space + sliding count |
| 9 | [Range Module](https://leetcode.com/problems/range-module/) | Hard | Interval tracking with no gaps — balanced-tree thinking in Python |

## Graphs at Google tempo
| # | Problem | Difficulty | Why it's L5 |
|---|---|---|---|
| 10 | [Shortest Path in a Grid with Obstacles Elimination](https://leetcode.com/problems/shortest-path-in-a-grid-with-obstacles-elimination/) | Hard | BFS with an extra state dimension — the "visit" set is 3-D |
| 11 | [Bricks Falling When Hit](https://leetcode.com/problems/bricks-falling-when-hit/) | Hard | Reverse-time union-find — senior-level inversion insight |
| 12 | [Odd Even Jump](https://leetcode.com/problems/odd-even-jump/) | Hard | Monotonic stack builds the jump graph, then DP over it |

## Sequences with hidden structure
| # | Problem | Difficulty | Why it's L5 |
|---|---|---|---|
| 13 | [Split Array into Consecutive Subsequences](https://leetcode.com/problems/split-array-into-consecutive-subsequences/) | Medium | Greedy extension vs new-run decision — proof sketch required |
| 14 | [Ugly Number II](https://leetcode.com/problems/ugly-number-ii/) | Medium | Three-pointer merged generation |
| 15 | [Maximum Points You Can Obtain from Cards](https://leetcode.com/problems/maximum-points-you-can-obtain-from-cards/) | Medium | Sliding window on the COMPLEMENT |
| 16 | [Airplane Seat Assignment Probability](https://leetcode.com/problems/airplane-seat-assignment-probability/) | Medium | The answer is a symmetry argument, not a simulation |

## Backtracking with constraints
| # | Problem | Difficulty | Why it's L5 |
|---|---|---|---|
| 17 | [Remove Invalid Parentheses](https://leetcode.com/problems/remove-invalid-parentheses/) | Hard | BFS over edit distance + dedup pruning |
| 18 | [24 Game](https://leetcode.com/problems/24-game/) | Hard | Recursive pair-reduction over rationals |
| 19 | [Cracking the Safe](https://leetcode.com/problems/cracking-the-safe/) | Hard | De Bruijn sequence / Hierholzer — obscure, occasionally asked at L5+ |

## Iterator / API design (L5 favorite)
| # | Problem | Difficulty | Why it's L5 |
|---|---|---|---|
| 20 | [Flatten Nested List Iterator](https://leetcode.com/problems/flatten-nested-list-iterator/) | Medium | Lazy generator vs eager stack — tradeoffs matter |
| 21 | [Peeking Iterator](https://leetcode.com/problems/peeking-iterator/) | Medium | Wrapper-state discipline |
| 22 | [Zigzag Iterator](https://leetcode.com/problems/zigzag-iterator/) | Medium | Round-robin queue of iterators |

## Probability / sampling
| # | Problem | Difficulty | Why it's L5 |
|---|---|---|---|
| 23 | [Random Pick with Weight](https://leetcode.com/problems/random-pick-with-weight/) | Medium | *(redo if shaky)* prefix sums + bisect + prove uniformity |
| 24 | [Guess the Word](https://leetcode.com/problems/guess-the-word/) | Hard | Minimax elimination strategy — interactive, unique style |

## Filler weeks (pick any)
| # | Problem | Difficulty | Why it's L5 |
|---|---|---|---|
| 25 | [Candy Crush](https://leetcode.com/problems/candy-crush/) | Medium | Pure simulation discipline — Google phone screen classic |
| 26 | [Expressive Words](https://leetcode.com/problems/expressive-words/) | Medium | Run-length canonicalization |

---

### How to know you're L5-ready on problems alone
You can take any row above cold, narrate the approach in under 3 minutes, code it optimally inside the target time, and answer "now make it stream / shrink the memory / parallelize it" without pausing. When that's true for 80% of this page, problem prep is no longer your bottleneck — spend the time on [system design](interview/system-design-track.md) and your [STAR stories](interview/googleyness-star.md).
