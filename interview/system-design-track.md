# 🏗️ System Design Track — 13 weekly sessions (REQUIRED for L5)

**Why this exists:** at Google, the system design round is usually what decides L4 vs L5. Coding rounds get you to "no-hire is off the table"; design rounds get the senior level. You cannot wing this track — 90 minutes once a week, same slot every week (recommended: Saturday after the review, or Sunday morning if you opt out of full rest). By Dec 27 you'll have designed 13 Google-scale systems and read the core of *Designing Data-Intensive Applications* (DDIA).

**Format per session (90 min):** 15 min requirements → 25 min one-page design (boxes + arrows + numbers) → 20 min "interviewer mode": answer the escalation questions aloud → 20 min compare with a reference discussion → 10 min: write 3 things you missed in `interview/design-notes.md`.

**Core reading (buy/borrow once, use all 13 weeks):** *Designing Data-Intensive Applications* (Kleppmann) — chapters assigned below. Free supplements: [system-design-primer](https://github.com/donnemartin/system-design-primer), [Google's SRE book chapters](https://sre.google/books/).

**Back-of-envelope numbers to memorize by Week 6** (Jeff Dean's classic set — say them in under 15 seconds): L1 cache ref ~0.5 ns · main-memory ref ~100 ns · SSD random read ~150 µs · disk seek ~10 ms · same-datacenter RTT ~0.5 ms · cross-continent RTT ~150 ms · 1 Gbps link ≈ 125 MB/s. Know them cold — an L5 candidate writes these without pausing.

## The 13 sessions

| Wk | Date (Sat) | Design prompt | DDIA / reading | L5 escalation you must survive |
|---|---|---|---|---|
| 1 | Oct 3 | **Design a URL shortener** (warm-up — but do it at L5 depth: custom aliases, analytics, expiry) | Primer: URL shortener | 10⁴ new links/s, 100 reads/s per link hot |
| 2 | Oct 10 | **Design a distributed rate limiter** (token bucket vs sliding window; per-user, per-API) | DDIA ch. 1 | Sticky vs non-sticky users; thundering herd on config change |
| 3 | Oct 17 | **Design Google Search autocomplete** (trie + top-K per node, tiered caching) | DDIA ch. 2 (data models) | 100k QPS typing latency <100 ms; fresh trends within minutes |
| 4 | Oct 24 | **Design a web crawler** (politeness, frontier, dedup, recrawl scheduling) | Primer: crawler | Crawl the top-1B pages on 100 machines; JS-rendered pages |
| 5 | Oct 31 | **Design Google Docs** (operational transformation vs CRDTs, presence) | DDIA ch. 5 (replication) | Offline edit + reconnect; 50 collaborators |
| 6 | Nov 7 | **Design YouTube / video pipeline** (upload → transcode → CDN, adaptive bitrate) | DDIA ch. 6 (partitioning) | 500 h uploaded/minute; viral video cold-start |
| 7 | Nov 14 | **Design Google Drive / file sync** (chunking, consistency, conflict files) | DDIA ch. 5–6 | 1 TB accounts; sync 100k-file folders; mobile bandwidth |
| 8 | Nov 21 | **Design Gmail** (mail storage, indexing/search, push, spam pipeline) | DDIA ch. 3 (storage engines) | Billions of messages/day; search across 10 GB mailbox <500 ms |
| 9 | Nov 28 | **Design Google Maps route planning** (hierarchical graphs, contraction hierarchies, traffic) | DDIA ch. 8 (unreliable clocks — traffic freshness) | World-graph routing <200 ms; live rerouting |
| 10 | Dec 5 | **Design a distributed Pub/Sub** (delivery guarantees, ordering, replay) | DDIA ch. 7, 11 (transactions, streams) | Exactly-once illusion; 1M subscribers of one topic |
| 11 | Dec 12 | **Design a rate-limited, sharded key-value cache** (Bigtable-lite: memtable/SSTables, consistent hashing) | DDIA ch. 3 (LSM trees) | 99.9th percentile <10 ms at 1M QPS; node failure rebalancing |
| 12 | Dec 19 | **Design Google Photos** (upload, dedup by content hash, ML tagging, album sharing) | DDIA ch. 9 (consistency + consensus) | Petabytes/year; multi-region durability 11 nines |
| 13 | Dec 26 | **FULL MOCK — interviewer asks, you drive 45 min, no notes** (pick: design the entire earlier system by dice) | Your `design-notes.md` | Have a friend or record yourself; grade against the L5 rubric below |

## The L5 design rubric (grade every session)

- [ ] **Requirements first, and you drove them** — functional AND the 2–3 numbers that shape everything (QPS, storage, latency)
- [ ] **Back-of-envelope math on the whiteboard** — storage/day, read/write ratio, cache hit rate needed
- [ ] **API + data model before boxes** — what's stored, what's indexed, who owns writes
- [ ] **Every box has a reason** — you can say what fails if it's removed
- [ ] **Scaling story volunteered** — partition key chosen and defended, hot spots named, rebalancing mentioned
- [ ] **Failure story volunteered** — what's lost on crash, replication, consistency level tradeoff (you named CAP honestly)
- [ ] **You led the last 20 minutes**, not the interviewer

> L4 designs answer questions. **L5 candidates raise the questions themselves** — "before we shard this, what's the write skew?" — then answer them.
