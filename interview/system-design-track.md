# 🏗️ System Design Track — 13 dated sessions (REQUIRED for L5)

**Why this exists:** at Google, the system design round is usually what decides L4 vs L5. Coding rounds get you to "no-hire is off the table"; design rounds get the senior level. You cannot wing this track — 90 minutes once a week, every Saturday, tracked separately from the DSA calendar.

**Where the sessions live:** each Saturday has its own dated checklist file in [design/](../design/) (SD-01 … SD-13, Oct 3 → Dec 26), with its own tracking — open the 🏗️ **System Design tab** in the dashboard (`python3 app.py`) or watch the design section of README.md (via `python3 tracker.py`). Every session's debrief lands in [design-notes.md](design-notes.md) — your running error catalog.

**Format per session (90 min):** 15 min requirements → 25 min one-page design (boxes + arrows + numbers) → 20 min "interviewer mode": answer the escalation questions aloud → 20 min compare with a reference discussion → 10 min: write 3 things you missed in `interview/design-notes.md`.

**Core reading (buy/borrow once, use all 13 weeks):** *Designing Data-Intensive Applications* (Kleppmann) — chapters assigned below. Free supplements: [system-design-primer](https://github.com/donnemartin/system-design-primer), [Google's SRE book chapters](https://sre.google/books/).

**Back-of-envelope numbers to memorize by Week 6** (Jeff Dean's classic set — say them in under 15 seconds): L1 cache ref ~0.5 ns · main-memory ref ~100 ns · SSD random read ~150 µs · disk seek ~10 ms · same-datacenter RTT ~0.5 ms · cross-continent RTT ~150 ms · 1 Gbps link ≈ 125 MB/s. Know them cold — an L5 candidate writes these without pausing.

## The 13 sessions (each links to its dated checklist file)

| Wk | Date (Sat) | Design prompt | Session file | DDIA |
|---|---|---|---|---|
| 1 | Oct 3 | URL shortener at L5 depth | [design/2026-10-03.md](../design/2026-10-03.md) | primer |
| 2 | Oct 10 | Distributed rate limiter | [design/2026-10-10.md](../design/2026-10-10.md) | ch. 1 |
| 3 | Oct 17 | Google Search autocomplete | [design/2026-10-17.md](../design/2026-10-17.md) | ch. 2 |
| 4 | Oct 24 | Web crawler | [design/2026-10-24.md](../design/2026-10-24.md) | primer |
| 5 | Oct 31 | Google Docs (OT/CRDT) | [design/2026-10-31.md](../design/2026-10-31.md) | ch. 5 |
| 6 | Nov 7 | YouTube video pipeline | [design/2026-11-07.md](../design/2026-11-07.md) | ch. 6 |
| 7 | Nov 14 | Google Drive sync | [design/2026-11-14.md](../design/2026-11-14.md) | ch. 5–6 |
| 8 | Nov 21 | Gmail | [design/2026-11-21.md](../design/2026-11-21.md) | ch. 3 |
| 9 | Nov 28 | Maps route planning | [design/2026-11-28.md](../design/2026-11-28.md) | ch. 8 |
| 10 | Dec 5 | Distributed Pub/Sub | [design/2026-12-05.md](../design/2026-12-05.md) | ch. 7, 11 |
| 11 | Dec 12 | Sharded KV cache (Bigtable-lite) | [design/2026-12-12.md](../design/2026-12-12.md) | ch. 3 |
| 12 | Dec 19 | Google Photos | [design/2026-12-19.md](../design/2026-12-19.md) | ch. 9 |
| 13 | Dec 26 | **FULL MOCK — 45 min, recorded** | [design/2026-12-26.md](../design/2026-12-26.md) | your notes |

## The L5 design rubric (grade every session)

- [ ] **Requirements first, and you drove them** — functional AND the 2–3 numbers that shape everything (QPS, storage, latency)
- [ ] **Back-of-envelope math on the whiteboard** — storage/day, read/write ratio, cache hit rate needed
- [ ] **API + data model before boxes** — what's stored, what's indexed, who owns writes
- [ ] **Every box has a reason** — you can say what fails if it's removed
- [ ] **Scaling story volunteered** — partition key chosen and defended, hot spots named, rebalancing mentioned
- [ ] **Failure story volunteered** — what's lost on crash, replication, consistency level tradeoff (you named CAP honestly)
- [ ] **You led the last 20 minutes**, not the interviewer

> L4 designs answer questions. **L5 candidates raise the questions themselves** — "before we shard this, what's the write skew?" — then answer them.
