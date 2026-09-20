# 🎖️ The Google L5 Bar — read this first, re-read it every phase

You are targeting **L5 (Senior Software Engineer) at Google with 3.5 YOE**. That is above the normal leveling band for that experience (typical L5 hires have 5–8 YOE), which changes what "good enough" means: you don't get to clear the bar — you have to visibly exceed it. This document defines that bar, how the plan trains it, and how to know you're on track.

## The honest reality check

- With 3.5 YOE, Google's default offer is **L4**. L5 happens when you *demonstrate senior scope*, not when you argue for it. Two rounds decide it: **system design** (usually the swing vote) and **coding depth** (speed + optimality + follow-ups).
- You cannot control leveling fully. You CAN control walking into the onsite over-prepared on both axes. That is what this repo does — DSA to a level where coding rounds are your *strength*, plus the [parallel system-design track](interview/system-design-track.md).
- A strong L4 offer with L5-level interview performance often converts to L5 within a year internally. The worst outcome is failing rounds that preparation could have won.

## What L5 coding performance looks like (vs L4)

| Signal | L4 (clears the bar) | L5 (exceeds it) |
|---|---|---|
| Cold solve, Medium | Optimal in ≤ 30 min with hints | ≤ 20 min, no hints, while narrating |
| Cold solve, Hard | Near-optimal in ≤ 45 min | Optimal or optimal-with-nudge in ≤ 35 min |
| First submission | Works after a bug or two | Works, clean, tested against edge cases BEFORE submitting |
| Follow-ups | Handles "what if X" with nudges | Anticipates 1–2 follow-ups; answers the rest by deriving, not guessing |
| Complexity | States it when asked | States time/space unprompted, including amortized where relevant |
| Communication | Explains the approach | LEADS: clarifies constraints, offers the brute force to reject it, discusses tradeoffs, drives the interview |
| Code | Correct | Readable: names, small helpers, no dead code, no magic numbers |

**The single biggest L5 tell:** you reject your own first idea out loud ("brute force is n², that dies at n=10⁵ from the constraints — here's the structure that deletes one loop"). L4s wait to be told; L5s say it first.

## How this plan trains each signal

| Signal | Trained by |
|---|---|
| Speed | Timed targets on every problem (Easy < 15, Medium ≤ 25, Hard ≤ 40) + 5 simulation days (Week 12) at interview tempo |
| Optimality-first | Pattern-of-the-day snippets = the optimal template for each family; hints force the narrowing question, not the answer |
| Hard-problem depth | 45 🎖️ L5 stretch problems (Google-caliber twists woven into Weeks 3–11) + [L5-SET.md](L5-SET.md) for after Dec 27 |
| Follow-ups | 🪜 follow-up ladders on every simulation day + the grand mock — streaming, memory-bounded, concurrent, amortized variants |
| Company calibration | Week 11 Meta/Google tagged sets with asked-at frequency |
| Communication | The reps line on every day: clarify → example → approach aloud → complexity → code → test. Non-optional, every problem |
| Design (the L5 swing vote) | [interview/system-design-track.md](interview/system-design-track.md) — 13 weekly sessions, run in parallel |

## Phase gates — do not soft-pedal these

At each phase retro day (Oct 25, Nov 22, Dec 6), grade yourself against the gate. One "no" = that phase's 🔁 queue gets heavier next week. Two "no"s = repeat the Saturdays, don't skip them.

- **Gate 1 (end of Phase 1, Oct 25):** any Blind-75 array/string/linked-list problem cold in ≤ 25 min. Sliding window, two pointers, binary-search-on-answer: derive the template from memory, not recognition.
- **Gate 2 (end of Phase 2, Nov 22):** tree traversal + BFS/DFS on grids cold in ≤ 20 min; topological sort and union-find from memory; you can explain WHY Dijkstra needs a heap and when it degenerates.
- **Gate 3 (end of Week 10, Dec 6):** any 1-D DP from a fresh statement in ≤ 30 min — identify state, transition, order, base cases OUT LOUD before writing code.
- **Gate 4 (end of Week 12, Dec 20):** two unseen mediums, 35 min each, both optimal, both narrated — this is literally the phone screen. If you can't do it on a tired Tuesday evening after work, you can't do it in the real one.
- **Gate 5 (Dec 24 grand mock):** 3 problems / 75 min at L5 tempo + every follow-up ladder answered.

## Interview-day protocol (memorize now, use in December)

1. **First 3 minutes are yours:** restate the problem, ask 2–3 clarifying questions (input size? duplicates? negative numbers? streaming or in-memory?), give one example.
2. **Name the brute force, reject it with the constraint math**, then commit to one approach and state complexity before coding.
3. **Narrate while coding** — silence reads as flailing at L5 even when the code is right.
4. **Test before submitting:** empty input, single element, all-duplicates, extremes. Say what you're checking.
5. When the follow-up comes: **derive on the whiteboard of your mind — "if X then Y, so Z"** — never "I think maybe".
6. Stuck 5+ minutes: verbalize the blocker and shrink the problem ("what if the array were sorted?"). Interviewers rescue candidates who think aloud; they let silent ones drown.
