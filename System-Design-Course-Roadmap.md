# System Design — Complete Course Roadmap

> A full-length, hands-on system design course that takes **junior engineers** from first
> principles all the way to **real-world production engineering** and **interview mastery**.
>
> **Design philosophy:** teach the *vocabulary and the single machine first*, then the
> network, then storage, then the genuinely *distributed* concepts in strict dependency
> order — and only **then** recombine everything in case studies. Every concept is paired
> with a hands-on lab, because retention roughly doubles when a student actually stands up a
> Redis cache or watches a load balancer split traffic instead of only hearing about it.

---

## Table of Contents

1. [Who This Course Is For](#1-who-this-course-is-for)
2. [What Students Will Be Able To Do](#2-what-students-will-be-able-to-do-outcomes)
3. [The Teaching Philosophy (read this before teaching)](#3-the-teaching-philosophy)
4. [How the Course Works — Format & Cadence](#4-how-the-course-works)
5. [The Course Map at a Glance](#5-the-course-map-at-a-glance)
6. [Part A — Foundations](#part-a--foundations-the-vocabulary--the-single-machine)
7. [Part B — Scaling the Stateless Tier](#part-b--scaling-the-stateless-tier)
8. [Part C — Distributed Systems Core](#part-c--distributed-systems-core)
9. [Part D — Reliability, Async & Operations](#part-d--reliability-async--operations-real-world-engineering)
10. [Part E — Putting It Together: Case Studies](#part-e--putting-it-together-case-studies)
11. [Part F — Interview Mastery](#part-f--interview-mastery)
12. [The Capstone Project](#the-capstone-project)
13. [Assessment & Grading](#assessment--grading)
14. [Scheduling Options](#scheduling-options)
15. [The Universal Design Rubric](#appendix-1--the-universal-design-rubric)
16. [Estimation Cheat Sheet & The Numbers](#appendix-2--estimation-cheat-sheet--the-numbers)
17. [Bibliography & Canonical Resources](#appendix-3--bibliography--canonical-resources)
18. [Glossary](#appendix-4--glossary)
19. [Instructor Notes: How to Prepare Each Lecture](#appendix-5--instructor-notes)
20. [2026 Technology Snapshot & Research Verification Log](#appendix-6--2026-technology-snapshot--research-verification-log)

---

## 1. Who This Course Is For

**Primary audience:** junior software engineers (roughly 0–3 years) who can already build a
CRUD app and write SQL, but have never designed a system that spans more than one machine,
and have little exposure to distributed-systems concepts.

**Assumed prerequisites (the floor):**
- Comfortable programming in at least one language (we use Python/Node/Go for labs — any works).
- Basic SQL and a rough idea of what a database is.
- Can use a terminal, run Docker, and read a diagram.
- *Not* assumed: networking internals, distributed systems, cloud infrastructure, queueing theory.

**Prerequisite gap-fillers** (offer as optional pre-reading — the research shows these missing
foundations are the root cause of most later confusion):
- What a socket, thread, and page cache actually are, and what a disk seek costs.
- Big-O intuition and the core data structures (hash tables, trees, heaps) — they *reappear*
  as consistent hashing, indexes, and priority queues.
- A one-page refresher on probability/averages vs. percentiles.

---

## 2. What Students Will Be Able To Do (Outcomes)

By the end, a student can:

1. **Speak the language** — correctly use latency vs. throughput, availability, consistency,
   durability, and the "nines," and reason in **percentiles (p99)**, not averages.
2. **Estimate before designing** — do back-of-the-envelope QPS/storage/bandwidth/memory math
   and use it to justify every component choice.
3. **Design the request path** — from DNS → CDN → load balancer → stateless app tier → cache →
   database → async workers, and explain *why* each piece exists.
4. **Choose a data store from access patterns**, not hype — SQL vs. NoSQL, B-tree vs. LSM,
   normalized vs. denormalized, and shard/replicate it correctly.
5. **Reason about distribution honestly** — replication lag, the *real* meaning of CAP/PACELC,
   consistency models, consensus, and distributed transactions vs. sagas.
6. **Design for failure as the default** — timeouts, retries with backoff+jitter, circuit
   breakers, bulkheads, rate limiting, idempotency, and graceful degradation.
7. **Operate what they build** — logs/metrics/traces, SLIs/SLOs/error budgets, health checks.
8. **Attack any "Design X" problem** with a repeatable rubric and communicate the trade-offs.
9. **Pass a system design interview** at the junior/mid level with confidence.

---

## 3. The Teaching Philosophy

Six principles distilled from how the best resources (DDIA, Alex Xu Vol 1 & 2, the System
Design Primer, Grokking, ByteByteGo, MIT 6.824) actually teach this material — and from the
specific ways juniors get it wrong.

1. **Building blocks before compositions.** Teach reusable primitives (load balancer, cache,
   queue, rate limiter, key-value store, consistent hashing) *first*. Then every case study is
   an assembly of things students already understand — never magic bolted on ("just add a
   cache"). This also creates natural **spaced repetition**: each primitive reappears across
   many case studies.

2. **Respect the dependency graph — never teach a concept before its prerequisites.** The most
   common curriculum failure is **teaching CAP before replication and partitions**, which
   produces students who recite "pick 2 of 3" with zero understanding. The correct order is
   *replication → consistency models → CAP/PACELC → partitioning → consensus*.

3. **Front-load estimation and the latency numbers.** Every downstream decision (how many
   shards, how big a cache, fan-out or not) needs this. Teach it in week 2 and use it every
   week after.

4. **Failure is a first-class, recurring theme — not a final chapter.** From the moment we
   distribute state, we ask "what happens when this node/network/clock misbehaves?" Drawing
   boxes is easy; reasoning about partial failure is the actual skill.

5. **Always name the trade-off.** The single most important habit to instill: no component is
   "correct," each one *costs* something. "We denormalize to make reads fast, and we pay for it
   with harder writes and possible drift." Naming the cost is the skill being graded.

6. **Calibrate against reality so students don't over-engineer.** Juniors reach for sharding,
   microservices, and Kafka when one beefy Postgres box (millions of rows, tens of thousands of
   QPS) would do. Give them a felt sense of how much a single machine handles, and teach
   "scale up first, scale out when you must."

**The recurring cross-cutting themes** (call them out every time they appear — they're the
connective tissue of the whole course): *consistent hashing, fan-out (write vs. read),
read/write path separation, CDNs & edge, geospatial indexing, event-driven pipelines,
idempotency, and the tail-latency problem.*

---

## 4. How the Course Works

**Format:** ~18 lecture modules across a full semester, grouped into 6 parts. Most modules are
one 90-minute lecture + one lab; the heavier ones (distributed core, case studies) may run two
sessions.

**Every lecture follows the same rhythm** (predictability aids learning):
1. **Hook (5 min)** — a real outage or scale story that the day's concept explains.
2. **Concept (35–45 min)** — the ideas, with diagrams, always ending on the trade-off.
3. **Misconception check (5 min)** — name the wrong mental model *explicitly* and correct it.
4. **Worked example / live design (20 min)** — apply it on the whiteboard.
5. **Lab handoff (5 min)** — set up the hands-on exercise.

**The hands-on spine.** Labs are Docker-based so nobody fights environment setup. They build,
lab by lab, toward a small but real system the student has stood up with their own hands:
a load-balanced, cached, replicated, observable service with async workers. This mirrors the
MIT 6.824 "you don't understand Raft until you've built it" philosophy, scaled down for juniors.

**Design studios.** Roughly every other week, run a 60-minute **group design studio**: put a
"Design X" prompt on the board and facilitate the whole class designing it live, using the
rubric. This is where interview skills are actually built.

---

## 5. The Course Map at a Glance

| # | Module | Part | Lab / Studio |
|---|--------|------|--------------|
| 1 | Introduction & Thinking at Scale | A | Measure p50/p99 of a toy service |
| 2 | Back-of-the-Envelope Estimation & The Numbers | A | Estimation drills |
| 3 | Networking & the Request Path | A | Trace a request end-to-end |
| 4 | Databases & Storage Fundamentals | A | Indexing & query plans in Postgres |
| 5 | Horizontal Scaling, Load Balancing & Statelessness | B | Stand up a load balancer over N app instances |
| 6 | Caching & CDNs | B | Add a Redis cache-aside layer; measure hit rate |
| 7 | Replication | C | Postgres primary + replica; observe lag |
| 8 | Consistency, CAP & PACELC | C | **Studio:** consistency trade-off design |
| 9 | Partitioning & Consistent Hashing | C | Implement consistent hashing from scratch |
| 10 | Consensus, Coordination & Distributed Transactions | C | Explore Raft (etcd) + idempotency keys |
| 11 | Designing for Failure | D | Build a rate limiter + circuit breaker |
| 12 | Asynchronous Processing & Messaging | D | Producer/consumer with a queue |
| 13 | Observability & Operating in Production | D | Instrument a service (Prometheus/Grafana/OTel) |
| 14 | Architecture Patterns & Modern Infrastructure | D | Containerize & deploy the service |
| 15 | Case Study Method + Easy Systems | E | **Studio:** URL shortener, rate limiter, ID gen |
| 16 | Medium Systems | E | **Studio:** news feed, chat, typeahead |
| 17 | Hard Systems | E | **Studio:** YouTube, Uber, payments |
| 18 | System Design Interview Mastery | F | Mock interviews |

---

# Part A — Foundations (the vocabulary & the single machine)

*Goal of Part A: give students the words, the numbers, the request path, and how one database
works — before anything is distributed.*

### Module 1 — Introduction & Thinking at Scale

- **Where it fits:** the foundation everything rests on. Depends on nothing.
- **Objectives:** define system design; build the non-functional vocabulary; internalize why
  we measure in percentiles.
- **Topics:**
  - What "system design" means and why it's different from writing a feature.
  - The tiers: client → application → data; stateless vs. stateful at a glance.
  - The five properties: **scalability, availability, reliability, durability, maintainability.**
  - **Latency vs. throughput** — distinct, often traded off. Batching raises throughput but adds
    latency. Intro to **Little's Law** (`L = λ × W`).
  - **Availability & the "nines":** 99.9% ≈ 8.8 h/yr down; 99.99% ≈ 52 min; 99.999% ≈ 5 min.
    Serial dependencies *multiply* (three 99.9% components in series ≈ 99.7%).
  - **Percentiles & tail latency (p50/p95/p99/p99.9):** why the average lies, and why one slow
    dependency fanned out across many calls dominates user-perceived latency ("The Tail at Scale").
  - **SLI / SLO / SLA** and the **error budget** idea (light touch; revisited in Module 13).
- **Misconceptions to preempt:** "average latency is fine"; "availability is just uptime%";
  "reliable = available" (they differ — durability vs. serving).
- **Lab:** run a toy HTTP service, fire load at it, and plot p50 vs. p99 — make the tail *visible*.
- **Core reading:** DDIA (2nd ed.) Ch. 1–2 (Trade-Offs; Defining Nonfunctional Requirements);
  "Latency Numbers Every Programmer Should Know."
- **Homework:** for three apps they use daily, write down the likely availability target and one
  place tail latency would hurt.

### Module 2 — Back-of-the-Envelope Estimation & The Numbers

- **Where it fits:** the quantitative backbone. Front-loaded on purpose — every later module and
  every case study uses it.
- **Objectives:** estimate QPS, storage, bandwidth, and memory quickly; memorize the key numbers.
- **Topics:**
  - **Powers of two & ten;** a day ≈ 86,400 s ≈ **10⁵ s** (the handy approximation).
  - **The latency numbers** (memory ~100 ns, SSD read ~16–100 µs, intra-DC RTT ~0.5 ms, disk seek
    ~ms, cross-continent RTT ~150 ms). Mental model: **RAM ≈ 100,000× faster than a disk seek.**
  - **The method:** QPS = DAU × actions/day ÷ 10⁵, then ×2–10 for peak. Storage = items/day ×
    size × retention. Bandwidth = QPS × payload. Cache memory = hot working set (~20% rule).
  - **Sanity limits of one machine:** ~10⁴–10⁵ simple QPS, tens of GB RAM, ~1 Gbps NIC ≈ 125 MB/s.
  - Worked example end-to-end (e.g., a 500M-DAU read-heavy service).
- **Misconceptions to preempt:** precision theater (estimation is about *order of magnitude*);
  forgetting peak-to-average multipliers; ignoring what one box can do.
- **Lab:** timed estimation drills — students estimate a Twitter-like feed's storage/QPS/bandwidth
  in 10 minutes, then compare and defend assumptions.
- **Core reading:** Alex Xu Vol 1, Ch. 2; the latency-numbers gist.
- **Homework:** three estimation problems with written assumptions.

### Module 3 — Networking & the Request Path

- **Where it fits:** prerequisite for load balancing, real-time systems, and every case study.
- **Objectives:** trace exactly what happens between "user hits enter" and "bytes come back."
- **Topics:**
  - **Client-server & DNS:** resolution hierarchy, TTL & caching, record types, **GeoDNS/anycast**,
    why DNS is a common SPOF and why failover via DNS is slow.
  - **TCP vs. UDP:** 3-way handshake (1 RTT setup), reliability/ordering/flow & congestion control
    vs. UDP's fire-and-forget. When each is used.
  - **HTTP evolution:** HTTP/1.1 (head-of-line blocking, ~6 connections) → **HTTP/2** (multiplexing,
    HPACK; still TCP-level HOL blocking) → **HTTP/3 over QUIC** (UDP, per-stream delivery, 0-RTT,
    connection migration). Current share (Cloudflare Radar, 2026): **HTTP/2 ≈ 51%, HTTP/1.x ≈ 28%,
    HTTP/3 ≈ 21%** — a useful teaching point that "newest" ≠ "dominant": HTTP/3 peaked near 28% in
    2023 and has since plateaued around 21% as HTTP/2 consolidated.
  - **TLS/HTTPS:** TLS 1.3 1-RTT handshake, certificates & ACME/Let's Encrypt, **mTLS** (foreshadows
    service mesh).
  - **Real-time transports:** **WebSockets** (full-duplex, stateful — scaling implications),
    **SSE** (one-way), long-polling as a fallback.
  - **API styles:** **REST vs. RPC/gRPC** (gRPC over HTTP/2 + protobuf, streaming) — a first look;
    full treatment in Module 14.
- **Misconceptions to preempt:** "HTTP/2 fixes all head-of-line blocking" (no — TCP-level remains);
  "WebSockets are free to scale like HTTP" (they're stateful).
- **Lab:** use `dig`, `curl -v`, and browser devtools to trace a real request — DNS resolution, TLS
  handshake, protocol version, headers — and narrate each step.
- **Core reading:** System Design Primer (networking sections); Cloudflare Learning Center (DNS,
  HTTP/3).
- **Homework:** diagram the full request path for loading a photo on a social app.

### Module 4 — Databases & Storage Fundamentals

- **Where it fits:** you must understand *one* database deeply before you distribute it.
- **Objectives:** choose a store from access patterns; understand how DBs store and index data.
- **Topics:**
  - **Relational model & SQL:** schema, relationships, joins, constraints, ACID.
  - **NoSQL families:** key-value, document, wide-column, graph — what each is good at.
  - **SQL vs. NoSQL — chosen from access patterns**, not hype. Polyglot persistence.
  - **Storage engines: B-tree vs. LSM-tree** — read-optimized in-place updates vs. write-optimized
    memtable→SSTable→compaction (with bloom filters). Which databases use which, and why.
  - **Indexing:** primary/clustered vs. secondary; composite indexes & leftmost prefix; covering
    indexes; the write/space cost of every index. Inverted indexes as a teaser for search.
  - **Normalization vs. denormalization** — the read/write trade-off; "model the schema from the
    queries."
  - **Transactions & isolation levels** (read committed → repeatable read → serializable), the
    anomalies each prevents, and **MVCC**.
- **Misconceptions to preempt:** "NoSQL scales, SQL doesn't" (Postgres scales far); "add an index,
  it's free" (writes and space); designing tables from entities instead of from queries.
- **Lab:** in Postgres, load a dataset, run a slow query, read the `EXPLAIN ANALYZE` plan, add an
  index, and measure the change. Show a write-heavy workload where the index *hurts*.
- **Core reading:** DDIA (2nd ed.) Ch. 3 (Data Models & Query Languages) & Ch. 4 (Storage &
  Retrieval); primer DB section.
- **Homework:** given a set of queries, design the schema and choose which indexes to create — and
  justify each.

---

# Part B — Scaling the Stateless Tier

*Goal of Part B: take the single-machine app and scale the "easy" (stateless) part horizontally,
plus the highest-leverage performance tool — caching.*

### Module 5 — Horizontal Scaling, Load Balancing & Statelessness

- **Where it fits:** the gateway to everything distributed; depends on networking (M3).
- **Objectives:** scale a stateless tier out; understand load balancing deeply.
- **Topics:**
  - **Vertical vs. horizontal scaling** — trade-offs; "scale up first, out when you must."
  - **Stateless vs. stateful services** — why statelessness makes horizontal scaling and failover
    trivial; push state to shared stores or the client (tokens/JWT).
  - **Load balancing:** **L4 vs. L7**; algorithms (round-robin, weighted, **least-connections**,
    latency-based, **power-of-two-choices**, IP/consistent-hash for stickiness).
  - **Reverse proxies & API gateways** — TLS termination, caching, routing, auth, rate limiting;
    the gateway as a potential SPOF/bottleneck (must be HA & thin). **BFF** pattern.
  - **Health checks** (active vs. passive) and **HA of the balancer itself** (VIP/VRRP, anycast, GSLB).
- **Misconceptions to preempt:** "the load balancer is a single point of failure" (make it redundant);
  "sticky sessions are fine" (they undermine statelessness — prefer shared session store).
- **Lab:** run 3 identical app containers behind NGINX/HAProxy; watch requests distribute; kill one
  and watch health checks reroute; switch algorithms and observe.
- **Core reading:** Primer (load balancer, reverse proxy); Grokking building blocks.
- **Homework:** redesign a single-server app to be horizontally scalable — list every piece of
  state and where it now lives.

### Module 6 — Caching & CDNs

- **Where it fits:** the highest-leverage latency/scale tool; depends on storage (M4) + numbers (M2).
- **Objectives:** apply the right caching strategy and reason about the failure modes.
- **Topics:**
  - **Where caches live:** client, DNS, **CDN/edge**, reverse proxy, **application in-memory**,
    **distributed cache**, DB buffer pool, materialized views.
  - **Read/write strategies:** **cache-aside** (lazy), read-through, **write-through**,
    **write-back/behind**, **write-around** — and when each fits.
  - **Eviction:** LRU (default), LFU, TTL, W-TinyLFU.
  - **Invalidation — "one of the two hard things":** TTL, explicit purge, versioned keys,
    event-driven invalidation.
  - **The failure modes** (teach by name — they show up in interviews and outages):
    **thundering herd / stampede**, **cache penetration**, **cache avalanche**, **hot key**, and
    their mitigations (request coalescing, negative caching/bloom filters, jittered TTLs, key
    replication).
  - **Redis vs. Memcached** (structures/persistence/HA vs. dead-simple ephemeral); note **Valkey**.
  - **CDNs:** push vs. pull, static vs. dynamic (edge compute), `stale-while-revalidate`,
    cache-busting via fingerprinted URLs.
- **Misconceptions to preempt:** "just add a cache" without invalidation or stampede handling;
  caching data that's never re-read; assuming the cache is always consistent with the DB.
- **Lab:** add a Redis **cache-aside** layer to the Module-4 service; measure latency and DB load
  before/after; then *induce* a thundering herd by expiring a hot key and fix it with a lock.
- **Core reading:** Primer (cache); AWS caching whitepaper; Redis docs.
- **Homework:** for three read patterns, pick a caching strategy and eviction policy and justify.

---

# Part C — Distributed Systems Core

*Goal of Part C: the intellectual heart of the course. Taught in strict dependency order —
replication → consistency → partitioning → consensus. Failure is now a constant companion.*

### Module 7 — Replication

- **Where it fits:** the foundation of all distributed data. **Must precede consistency and CAP.**
- **Objectives:** understand why we replicate, the three topologies, and the cost (lag & conflicts).
- **Topics:**
  - **Why replicate:** availability, read scaling, durability, latency (data near users).
  - **Leader–follower** (primary–replica): all writes to leader; sync vs. async replication;
    reads on followers scale but risk **staleness**.
  - **Replication lag** and the anomalies it causes — sets up the session guarantees in M8.
  - **Multi-leader** (multi-DC, offline clients) — write availability/locality vs. **write conflicts**.
  - **Leaderless (Dynamo-style)** — any replica serves; **read repair** & **anti-entropy** (Merkle
    trees). Introduces the idea of **quorums** (fully developed in M10).
  - **Failover:** leader election, and the dangers — **lost writes** and **split-brain**.
  - Replication mechanics: WAL/binlog shipping, statement vs. logical (row-based).
- **Misconceptions to preempt:** "replicas are always in sync" (async lag is real); "failover is
  automatic and safe" (split-brain, fencing needed).
- **Lab:** set up Postgres **primary + read replica**; write to the primary, read from the replica,
  and *observe replication lag* directly; simulate a stale read.
- **Core reading:** DDIA (2nd ed.) Ch. 6 (Replication); Kleppmann's distributed systems lectures.
- **Homework:** for a read-heavy app, design the replication topology and state where stale reads
  are/aren't acceptable.

### Module 8 — Consistency, CAP & PACELC

- **Where it fits:** *only* teachable after replication + the notion of a network partition.
  This ordering is the single most important sequencing decision in the course.
- **Objectives:** understand consistency models honestly; state CAP and PACELC precisely.
- **Topics:**
  - **Consistency models, strongest → weakest:** **linearizable** (single global order, sees latest
    write) → sequential → **causal** (preserves cause/effect; achievable with high availability) →
    **eventual** (converges if writes stop).
  - **Client-centric session guarantees:** **read-your-writes**, **monotonic reads**, monotonic
    writes, writes-follow-reads — these are what "eventual consistency" needs to feel sane.
  - **CAP theorem — stated correctly:** under a **partition**, choose **C or A**; absent a partition
    you can have both. CP (etcd/ZooKeeper/HBase/Spanner) vs. AP (Cassandra/Dynamo/Riak).
  - **PACELC:** if **P**artition → A vs. C; **E**lse → **L**atency vs. **C**onsistency. Even with no
    failures, replication forces a latency/consistency choice. Examples: Cassandra = PA/EL,
    Spanner = PC/EC.
  - **Conflict resolution:** last-writer-wins (can lose data), version vectors, **CRDTs**
    (merge without coordination — collaborative editing).
- **Misconceptions to preempt (this is the big one):** "CAP means pick any 2 of 3 always" — no, it's
  only about behavior *during a partition*; "eventual consistency = broken."
- **Studio (in place of a lab):** the class designs a shopping-cart / like-counter / bank-balance
  feature three times, each demanding a different consistency level, and argues the trade-offs.
- **Core reading:** DDIA (2nd ed.) Ch. 10 (Consistency & Consensus — rewritten for clarity in the
  2nd ed.); Jepsen consistency map; Abadi's PACELC paper.
- **Homework:** classify five real features (DM read receipts, bank transfer, follower count,
  collaborative doc, ad impression) by the consistency model they need.

### Module 9 — Partitioning & Consistent Hashing

- **Where it fits:** scaling *writes* and data volume beyond one node; depends on hashing + M7.
- **Objectives:** shard data without creating hotspots; implement consistent hashing.
- **Topics:**
  - **Why partition:** spread data and write load past a single node's ceiling.
  - **Strategies:** **range** (great for scans, risks hotspots on sequential keys), **hash** (even
    spread, loses range queries), **directory/lookup-based** (flexible, extra hop/SPOF), geo/entity.
  - **Consistent hashing** in depth — the ring, why plain `mod N` reshuffles everything on a
    membership change, and **virtual nodes** for smooth distribution. Core to sharded caches, DHTs,
    Cassandra/DynamoDB.
  - **Hotspots / the celebrity problem** — key salting, splitting hot partitions, caching.
  - **Secondary indexes:** local (document-partitioned, scatter-gather reads) vs. global
    (term-partitioned, harder writes).
  - **Rebalancing** — fixed partition count ≫ nodes; avoid mod-N.
  - **Cross-shard operations** are expensive — co-locate related data via the shard key.
- **Misconceptions to preempt:** "hash mod N is fine for sharding" (it isn't — reshuffle);
  "sharding is free" (cross-shard joins/transactions hurt).
- **Lab:** implement **consistent hashing from scratch** (ring + virtual nodes); add/remove nodes
  and measure the fraction of keys that move — compare to naive mod-N.
- **Core reading:** DDIA (2nd ed.) Ch. 7 (Sharding); Grokking (consistent hashing); Dynamo paper (skim).
- **Homework:** choose a shard key for a chat app and for a ride-hailing app; explain hotspot risk.

### Module 10 — Consensus, Coordination & Distributed Transactions

- **Where it fits:** the hardest material; depends on replication, partitioning, and failure models.
- **Objectives:** understand quorums, what consensus buys you, and how to get atomicity across nodes.
- **Topics:**
  - **Quorums:** `W + R > N` guarantees a read overlaps the latest write; tuning W/R for
    durability vs. latency (e.g., N=3, W=2, R=2).
  - **Consensus:** the problem (agree on a value/order despite failures); **FLP impossibility** in
    one line (so we use timeouts/randomization); **Raft** conceptually — leader election (terms,
    randomized timeouts), log replication, commit on majority. Where it powers real systems
    (etcd, Consul, CockroachDB, **Kafka KRaft**). Paxos named but not derived.
  - **Leader election & leases;** **fencing tokens** to stop a paused old leader from acting.
  - **Distributed transactions:** **2PC** (prepare/commit; blocks if coordinator dies — poor
    availability); why it doesn't scale.
  - **Saga pattern** (orchestration vs. choreography) with **compensating actions** — the
    microservices default; provides atomicity-via-compensation, *not* isolation.
  - **The outbox pattern** (+ CDC) to solve the dual-write problem.
  - **Idempotency** as the load-bearing concept: idempotency keys, dedup, conditional writes;
    **"exactly-once effect" = at-least-once delivery + idempotent processing.**
- **Misconceptions to preempt:** "2PC makes microservices consistent and scalable" (it blocks);
  "exactly-once delivery exists" (you get effectively-once via idempotency).
- **Lab:** run a 3-node **etcd** cluster; watch leader election; kill the leader and watch
  re-election; then implement an **idempotency key** table that dedupes retried writes.
- **Core reading:** DDIA (2nd ed.) Ch. 8–10 (Transactions; The Trouble with Distributed Systems;
  Consistency & Consensus); the Raft paper / raft.github.io visualization; saga & outbox articles.
- **Homework:** design an order-placement flow across 3 services using a saga; list every
  compensating action and where idempotency is required.

---

# Part D — Reliability, Async & Operations (real-world engineering)

*Goal of Part D: turn "a design that works on the happy path" into "a system you can actually run
in production." This is the part interview-only courses skip — and it's what makes real engineers.*

### Module 11 — Designing for Failure

- **Objectives:** make failure the default assumption and apply the resilience toolkit.
- **Topics:**
  - **The fallacies of distributed computing;** partial failure as the norm.
  - **Redundancy:** N+1/N+2; **active-active vs. active-passive**; multi-AZ/multi-region.
  - **Timeouts everywhere** (never wait forever).
  - **Retries with exponential backoff + jitter**, retry budgets, and **retry amplification**
    (why naive retries make outages worse).
  - **Circuit breakers** (closed/open/half-open) and **bulkheads** (isolate pools) to stop
    cascading failure.
  - **Rate limiting** — **token bucket**, leaky bucket, fixed window, sliding window; distributed
    rate limiting via Redis; `429 + Retry-After`.
  - **Graceful degradation & load shedding** — fallbacks, feature flags/kill switches, read-only
    mode, graceful shutdown/connection draining.
  - **Chaos engineering** as validation.
- **Misconceptions to preempt:** "retries make things more reliable" (without jitter/budgets they
  cause storms); "we'll add reliability later."
- **Lab:** implement a **token-bucket rate limiter** (in-process, then backed by Redis for the
  distributed case) and wrap a flaky dependency in a **circuit breaker**; demonstrate fail-fast.
- **Core reading:** Google SRE book (handling overload, addressing cascading failures); AWS
  "Exponential Backoff and Jitter."
- **Homework:** take a case-study design and annotate every remote call with its timeout, retry
  policy, and fallback.

### Module 12 — Asynchronous Processing & Messaging

- **Objectives:** decouple systems with queues/logs and reason about delivery semantics.
- **Topics:**
  - **Why async:** decoupling, buffering/load-leveling, resilience, elastic scaling.
  - **Queue vs. pub/sub vs. log:** point-to-point (one consumer) vs. fan-out (all subscribers) vs.
    **log-based** (Kafka offsets → replay + both patterns).
  - **The brokers (2026):** **Kafka** (high-throughput durable log, replay, KRaft/no-ZooKeeper,
    Streams/Connect), **RabbitMQ** (rich AMQP routing, task queues, quorum queues), **SQS/SNS/
    EventBridge** (managed, zero-ops). A selection heuristic.
  - **Event-driven architecture:** notification vs. event-carried state transfer vs. event
    sourcing; schema registry & event versioning.
  - **Delivery guarantees:** at-most-once / **at-least-once** (default → consumers must be
    idempotent) / exactly-once (effectively-once). Ordering only within a partition.
  - **Stream processing:** windowing, event-time vs. processing-time, watermarks (Flink/Kafka
    Streams) — a conceptual intro.
  - **Backpressure:** bounded buffers, pull-based flow, load shedding — why its absence causes OOM.
- **Misconceptions to preempt:** "a queue guarantees exactly-once and global ordering" (it doesn't);
  "async is always better" (it trades away simplicity and immediate consistency).
- **Lab:** build a producer/consumer pipeline (RabbitMQ or Kafka/Redpanda in Docker): offload a
  slow task from the request path to a worker; then crash the consumer mid-batch and show that
  **at-least-once + idempotent processing** recovers correctly.
- **Core reading:** DDIA (2nd ed.) Ch. 12–13 (Stream Processing; A Philosophy of Streaming
  Systems); ByteByteGo messaging material.
- **Homework:** redesign a synchronous "upload → process → notify" flow to be event-driven and
  mark the delivery guarantee at each hop.

### Module 13 — Observability & Operating in Production

- **Objectives:** know whether the system is healthy and debug it when it isn't.
- **Topics:**
  - **The three pillars:** **logs** (structured, correlation/trace IDs, centralized),
    **metrics** (cheap time series; **RED** for services, **USE** for resources, the **four golden
    signals**), **traces** (end-to-end across services; **OpenTelemetry** as the 2026 standard).
  - **SLIs / SLOs / error budgets** and **burn-rate alerting**; **alert on symptoms, not causes.**
  - **Health checks:** liveness vs. readiness vs. startup; shallow vs. deep (and cascade risk).
  - Cardinality/cost pitfalls; the rise of **eBPF-based** zero-instrumentation observability.
  - Deploys & rollouts (context for M14): rolling, blue-green, canary; feature flags.
- **Misconceptions to preempt:** "logs are enough"; "monitor everything" (cardinality/cost);
  "alert on every anomaly" (alert on SLO burn).
- **Lab:** instrument the running service with **Prometheus + Grafana** (or an OTel collector);
  build a dashboard for the four golden signals; define one SLO and a burn-rate alert; then
  inject latency and watch it fire.
- **Core reading:** Google SRE book (monitoring, SLOs); OpenTelemetry docs.
- **Homework:** define SLIs/SLOs for a case-study system and the dashboards you'd build.

### Module 14 — Architecture Patterns & Modern Infrastructure

- **Objectives:** place everything learned into current architectural patterns and the 2026 stack;
  a broad map so no case study uses an unfamiliar building block.
- **Topics:**
  - **Monolith vs. microservices vs. SOA;** the **modular monolith** as a sane default; **Conway's
    Law**; "don't start with microservices."
  - **Serverless (FaaS/BaaS)** — cold starts, limits, when it fits; serverless containers.
  - **Event sourcing & CQRS** — what they buy and what they cost (revisits M10/M12).
  - **API design deep dive:** **REST** (pagination, versioning, idempotency keys, RFC 9457 errors),
    **GraphQL** (over/under-fetching, N+1/DataLoader, federation), **gRPC** (internal, typed,
    streaming). Contract-first everywhere.
  - **Service discovery** (client- vs. server-side; Consul/etcd/Kubernetes DNS).
  - **Modern infrastructure map:** **containers & Kubernetes** (Pod/Deployment/Service/Ingress-
    Gateway-API/HPA; control plane on etcd), **service mesh** (sidecar → **ambient/eBPF** in 2026),
    **object storage (S3 API)**, **data lake/warehouse/lakehouse** (Iceberg/Delta), **search
    (Elasticsearch/OpenSearch)**, **time-series DBs**, **vector DBs** (HNSW, pgvector, RAG),
    **edge computing**.
- **Misconceptions to preempt:** "microservices are the goal"; "Kubernetes is required";
  cargo-culting the newest tech without a driving requirement.
- **Lab:** containerize the service, write a docker-compose (or minimal Kubernetes manifests) that
  brings up app + LB + cache + DB + worker + monitoring together — the whole system, running.
- **Core reading:** DDIA (2nd ed.) Ch. 5 (Encoding & Evolution) + Ch. 11 (Batch Processing);
  ByteByteGo "System Design 101"; Kubernetes docs (concepts). See the **2026 Technology Snapshot**
  (Appendix 6) for the current state of every tool named here.
- **Homework:** for the capstone, decide monolith vs. services and justify; pick the API style per
  boundary.

---

# Part E — Putting It Together: Case Studies

*Goal of Part E: recombine every building block under the pressure of a concrete "Design X" prompt,
using one repeatable rubric. Each case study is run as a live design studio, then written up.*

> **Instructor prep:** every case study below is backed by a real-world architecture (with concrete
> numbers and primary sources) in the companion **[Case-Study Reference](Case-Study-Reference.md)** —
> Twitter's hybrid fan-out & the Katy-Perry celebrity problem, Uber's H3 hexagonal index, WhatsApp's
> Erlang connection model, Netflix Open Connect, Dropbox's Magic Pocket, Stripe idempotency, the
> Kafka-Flink-ClickHouse ad-click pipeline, and more. Run the studio, then reveal the real design —
> the gap is the lesson.

**The rubric applied every single time** (full version in [Appendix 1](#appendix-1--the-universal-design-rubric)):
1. Clarify functional & non-functional requirements + scope
2. Back-of-the-envelope estimation
3. API design / interface
4. Data model & storage choice (justified from access patterns)
5. High-level architecture (get buy-in)
6. Deep dive on 1–2 components
7. Identify bottlenecks; scale (cache, shard, replicate, queue); discuss failure modes & trade-offs

### Module 15 — Case Study Method + Easy Systems

- **Objectives:** internalize the rubric on tractable problems.
- **Systems (each maps to specific building blocks):**
  - **URL shortener (TinyURL/bit.ly)** — unique ID generation (counter/base62/**Snowflake**),
    read-heavy redirect path, caching, DB choice.
  - **Distributed rate limiter** — token bucket at scale, Redis, per-key sharding (revisits M11).
  - **Unique ID generator** — Snowflake-style (timestamp + machine + sequence), clock issues.
  - **Key-value store / distributed cache** — consistent hashing + quorums + replication
    (revisits M7/M9/M10) — the "Dynamo in miniature" that ties Part C together.
  - **Pastebin** — object storage + metadata + expiry.
- **Studio:** design the URL shortener live as a class, then students write up the rate limiter solo.
- **Homework:** full written design of one easy system using the rubric.

### Module 16 — Medium Systems

- **Objectives:** handle fan-out, real-time, and search-shaped problems.
- **Systems & the key challenge each teaches:**
  - **News feed / Twitter timeline** — **fan-out on write vs. on read**, the **celebrity problem**,
    timeline caching. *The canonical fan-out lesson.*
  - **Chat / WhatsApp / Messenger** — WebSockets, presence, delivery/read receipts, message storage,
    group chat fan-out, offline delivery.
  - **Search autocomplete (typeahead)** — **trie**, ranking, prefix caching, sub-100 ms latency.
  - **Web crawler** — crawl frontier, politeness, dedup (bloom filters), inverted index.
  - **Notification system** — multi-channel (push/SMS/email), fan-out, dedup, prioritization,
    rate limiting.
- **Studio:** design the news feed live (make the fan-out trade-off the centerpiece); students
  design chat solo.
- **Homework:** full written design of one medium system, with the estimation driving the fan-out
  decision.

### Module 17 — Hard Systems

- **Objectives:** integrate storage, streaming, geospatial, and money under real constraints.
- **Systems & the key challenge each teaches:**
  - **YouTube / Netflix** — upload → **transcoding pipeline**, **CDN** delivery, adaptive bitrate,
    metadata vs. blob storage, recommendations (overview).
  - **Uber / Lyft (proximity)** — **geospatial indexing** (geohash / quadtree / **H3**), matching,
    real-time location streams, surge (event aggregation).
  - **Google Drive / Dropbox** — **chunking, dedup, sync**, metadata service, conflict resolution.
  - **Payment system** — **idempotency, exactly-once effect, ledger/double-entry, reconciliation**
    (the strongest consistency demands in the course).
  - **Ad-click aggregation** — high-throughput **stream processing**, exactly-once counting,
    lambda/kappa architecture (batch + stream).
- **Studio:** design Uber's proximity/matching live (geohash is the star); students design the
  payment system solo (idempotency is the star).
- **Homework:** full written design of one hard system; explicitly call out the recurring
  cross-cutting themes it uses.

---

# Part F — Interview Mastery

### Module 18 — System Design Interview Mastery

- **Objectives:** convert design competence into interview performance.
- **Topics:**
  - **The framework for a 45-minute interview** (Alex Xu's 4-step method):
    1. **Understand the problem & scope** (~5 min) — clarifying questions, functional vs.
       non-functional requirements, agree on scope.
    2. **Propose high-level design & get buy-in** (~15 min) — the box diagram, the API, the data
       model; *think out loud.*
    3. **Deep dive** (~15 min) — the interviewer steers; go deep on 1–2 components (data model,
       sharding, the bottleneck).
    4. **Wrap up** (~5 min) — bottlenecks, failure modes, scaling, trade-offs, "what I'd do with
       more time."
  - **An alternative mnemonic — RESHADED** (Educative/Grokking; teach it as a checklist so students
    don't forget a step): **R**equirements → **E**stimation → **S**torage schema → **H**igh-level
    design → **A**PI design → **D**etailed design → **E**valuation (does it meet the requirements?)
    → **D**istinctive/edge concerns. It maps onto the 4-step method but spells out every sub-step —
    useful for nervous juniors who otherwise skip estimation or the data model.
  - **Requirements gathering** — the clarifying questions to always ask; scoping down.
  - **Estimation under pressure** — a fast, rehearsed template (from Module 2).
  - **What interviewers actually assess — by level.** Junior/mid: solid fundamentals, structured
    approach, sensible choices, communication. Senior: depth, trade-off nuance, driving the
    conversation, failure/ops thinking.
  - **The top mistakes** (straight from the research): jumping to components before requirements;
    skipping estimation; no structure; misusing CAP; cargo-culting; over-engineering; ignoring
    failure modes; designing from entities not queries.
  - **Communication & collaboration** — narrate your reasoning, state assumptions, propose then
    invite critique, manage time, recover gracefully when corrected.
- **Lab / assessment:** **peer mock interviews** — students pair up and alternate as interviewer/
  candidate on the case-study bank, using a scoring rubric; instructor runs 1–2 live mocks and
  debriefs.
- **Core reading:** Alex Xu Vol 1 Ch. 3; Grokking's interview approach; the System Design Primer's
  "how to approach" section.
- **The interview question bank** (assign for practice — grouped by difficulty):
  - *Easy:* URL shortener, pastebin, rate limiter, unique ID generator, key-value store.
  - *Medium:* Twitter/news feed, chat/WhatsApp, Instagram, typeahead, web crawler, notification
    system, Twitter search, nearby friends.
  - *Hard:* YouTube/Netflix, Uber/proximity, Google Maps, Google Drive/Dropbox, payment/digital
    wallet, ad-click aggregation, distributed message queue, S3-like object store, ticketing
    (Ticketmaster), stock exchange, distributed job scheduler.

---

## The Capstone Project

A single project carried across the whole course, so every concept lands somewhere concrete.

- **Deliverable:** a written design document + a *running* minimal implementation (the labs
  accumulate into it) for a system of the student's choice from a curated list (e.g., a
  link-shortener with analytics, a mini chat app, a job board with search, a notification service).
- **Milestones tied to the parts:**
  - After Part A: requirements, estimation, API, data model.
  - After Part B: horizontally scalable design with LB + cache.
  - After Part C: replication/partitioning/consistency decisions written up.
  - After Part D: failure handling, async workers, observability plan — and the running system.
  - After Part E: a full design-doc rewrite applying the rubric, plus a 15-minute presentation
    defended in front of the class as a mock design review.

This gives the "operated a system in production" experience the research flags as the missing
foundation behind most junior mistakes.

---

## Assessment & Grading

A suggested weighting (adjust to your institution):

| Component | Weight | What it measures |
|---|---|---|
| Weekly homework (design write-ups + estimation) | 25% | Applying the rubric & the numbers |
| Labs (accumulating into the capstone system) | 25% | Hands-on competence |
| Design studios / participation | 10% | Live reasoning & communication |
| Capstone project (doc + running system + presentation) | 25% | Integration of everything |
| Mock interviews (final) | 15% | Interview readiness |

**Grading rubric for any design (use for homework, capstone, and mocks):**
requirements & scope clarity · estimation correctness · sound storage/data-model choice ·
appropriate scaling · **failure & edge-case reasoning** · **explicit trade-offs** · communication.

---

## Scheduling Options

**Option 1 — Full semester (16 weeks, recommended).** One module/week for the 18 modules, with the
distributed core (M7–M10) and case studies (M15–M17) each allowed to spill into a second session.
Weeks 15–17 run as studios; week 18 is mock-interview week; capstone presentations in the final week.

**Option 2 — Compressed (10–12 weeks).** Merge M1+M2 (foundations), M5+M6 (scaling), and run the
case studies as two combined sessions. Keep the full distributed core intact — it's the part that
doesn't compress well.

**Option 3 — Interview-focused bootcamp (6 weeks).** Weeks 1–2: Part A + LB/cache condensed.
Week 3: distributed core essentials (replication, CAP done right, consistent hashing, idempotency).
Week 4: failure + async. Weeks 5–6: case studies + mock interviews. (Sacrifices depth in ops and
modern infra.)

> **My recommendation:** teach the full semester if you can. For junior engineers, the ops/failure
> modules (Part D) and the hands-on labs are exactly what separate "can recite CAP" from "can build
> and run a real system" — and they're what most competing courses omit.

---

## Appendix 1 — The Universal Design Rubric

Print this. It's used in every case study, the capstone, and every interview.

1. **Requirements & scope**
   - *Functional:* what must it do? (the core 2–3 features — resist scope creep)
   - *Non-functional:* scale (DAU, QPS), read/write ratio, latency SLO, availability target,
     consistency needs, data size & growth, durability.
2. **Estimation** — QPS (avg + peak), storage (× retention), bandwidth, cache memory. Sanity-check
   against one-machine limits.
3. **API / interface** — the handful of endpoints or RPCs; request/response shapes; idempotency keys.
4. **Data model & storage** — entities, access patterns, **SQL vs. NoSQL chosen from the queries**,
   indexes, partitioning key.
5. **High-level architecture** — client → DNS/CDN → LB → stateless app tier → cache → data store(s)
   → async workers/queues. Draw the boxes; get buy-in.
6. **Deep dive** — pick the 1–2 hardest components (the bottleneck, the data model, the sharding
   scheme, the real-time path) and go deep.
7. **Scale & harden** — cache, shard, replicate, async; then **failure modes** (hot keys, thundering
   herd, cascading failure, partition behavior) and **graceful degradation**.
8. **Trade-offs & wrap-up** — name what every choice costs; what you'd do with more time.

---

## Appendix 2 — Estimation Cheat Sheet & The Numbers

**Powers & the handy approximation**
- KB 10³ · MB 10⁶ · GB 10⁹ · TB 10¹² · PB 10¹⁵
- 1 day ≈ 86,400 s ≈ **10⁵ s**
- 2¹⁰ ≈ 1K · 2²⁰ ≈ 1M · 2³⁰ ≈ 1B

**Latency numbers every engineer should know (order of magnitude, 2026)**

| Operation | Time |
|---|---|
| L1 cache reference | ~1 ns |
| Main memory (RAM) reference | ~100 ns |
| Compress 1 KB (Snappy) | ~2 µs |
| NVMe SSD random read | ~16–100 µs |
| Read 1 MB sequentially from RAM | ~3–10 µs |
| Read 1 MB sequentially from SSD | ~50–200 µs |
| Round trip within a datacenter | ~0.5 ms |
| Disk seek (HDD) | ~3–10 ms |
| Read 1 MB from HDD | ~2–20 ms |
| Round trip cross-continent (CA↔EU) | ~150 ms |

> **Mental model:** memory is nanoseconds, SSD is microseconds, disk seeks and networks are
> milliseconds. RAM is ~100,000× faster than a disk seek. Cross-continent RTT dominates everything.

**The estimation method**
1. **QPS** = DAU × actions/user/day ÷ 10⁵ s → then **× 2–10** for peak.
2. **Storage** = items/day × bytes/item × retention (days). Add replication factor.
3. **Bandwidth** = QPS × payload size.
4. **Cache memory** = hot working set (often ~20% of data serves ~80% of requests).
5. **One machine, roughly:** 10⁴–10⁵ simple QPS · tens of GB RAM · ~1 Gbps NIC ≈ 125 MB/s.

**The nines**

| Availability | Downtime/year |
|---|---|
| 99% | ~3.65 days |
| 99.9% | ~8.8 hours |
| 99.99% | ~52 minutes |
| 99.999% | ~5 minutes |

---

## Appendix 3 — Bibliography & Canonical Resources

**Primary texts**
- **Martin Kleppmann & Chris Riccomini — *Designing Data-Intensive Applications*, 2nd edition
  (O'Reilly, published March 24, 2026)** — the theory backbone. The 2nd-edition chapters (all
  citations in this roadmap use this numbering): **1.** Trade-Offs in Data Systems Architecture ·
  **2.** Defining Nonfunctional Requirements · **3.** Data Models & Query Languages · **4.** Storage
  & Retrieval · **5.** Encoding & Evolution · **6.** Replication · **7.** Sharding · **8.**
  Transactions · **9.** The Trouble with Distributed Systems · **10.** Consistency & Consensus
  (substantially rewritten for clarity) · **11.** Batch Processing · **12.** Stream Processing ·
  **13.** A Philosophy of Streaming Systems · **14.** Doing the Right Thing. New in the 2nd ed.:
  vector indexes for semantic search, dataframes, sync engines & local-first software, durable
  execution/workflow engines, formal methods & randomized testing. https://dataintensive.net/
- **Alex Xu — *System Design Interview* Vol 1 (16 ch.) & Vol 2 (13 ch.)** — the case-study &
  interview backbone. *Vol 1:* Scale from Zero to Millions → Estimation → Interview Framework →
  Rate Limiter, Consistent Hashing, Key-Value Store, Unique ID Generator, URL Shortener, Web
  Crawler, Notification System, News Feed, Chat, Search Autocomplete, YouTube, Google Drive.
  *Vol 2:* Proximity Service, Nearby Friends, Google Maps, Distributed Message Queue, Metrics
  Monitoring, Ad Click Aggregation, Hotel Reservation, Distributed Email, S3-like Object Storage,
  Gaming Leaderboard, Payment System, Digital Wallet, Stock Exchange. https://bytebytego.com/
- **The System Design Primer** (donnemartin, GitHub) — free topic index + worked problems.
  https://github.com/donnemartin/system-design-primer
- **Grokking the Modern System Design Interview** (Educative) — the "building blocks then case
  studies" pedagogy. https://www.educative.io/
- **ByteByteGo course + "System Design 101"** (visual glossary).
  https://github.com/ByteByteGoHq/system-design-101

**Deepening & reference**
- **Google SRE Books** (free) — reliability & operations (Part D). https://sre.google/books/
- **Jepsen** — consistency models map + real-world analyses. https://jepsen.io/consistency
- **Raft** — visualization & paper. https://raft.github.io/
- **Latency numbers every programmer should know.** https://gist.github.com/jboner/2841832
- **Cloudflare Learning Center** — DNS, CDN, HTTP/3, TLS. https://www.cloudflare.com/learning/
- **AWS Well-Architected Framework.** https://aws.amazon.com/architecture/well-architected/
- **"The Tail at Scale"** (Dean & Barroso, CACM 2013) — tail latency.
- **Kleppmann's Distributed Systems lecture series** (free, YouTube) — accessible DDIA companion.
- **MIT 6.5840 (formerly 6.824) Distributed Systems** — for the labs-first philosophy & deep dives.
  The Spring 2026 arc covers MapReduce, RPC/threads, GFS, **Paxos**, **Raft (×2)**, Linearizability,
  ZooKeeper, Distributed Transactions, **Spanner**, Chain Replication, OCC, formal verification,
  Memcached-at-Facebook, AWS Lambda, Ray, Bitcoin, and BFT. **Five Go labs:** (1) MapReduce, (2)
  Key/Value server, (3) Raft, (4) KV-on-Raft, (5) Sharded KV. https://pdos.csail.mit.edu/6.5840/

**Key papers to skim (optional deep dives):** Google File System; MapReduce; Dynamo; Bigtable;
Spanner; Raft; Kafka. Great for motivated students; not required for juniors.

> **Verification note (updated):** this roadmap has been **live-verified against current sources
> (July 2026)**. All canonical tables of contents (DDIA 2nd ed., Alex Xu Vol 1 & 2, the System
> Design Primer, MIT 6.5840) were confirmed against the publishers/repos, and every fast-moving
> figure (HTTP/3 share, Kafka/Istio/pgvector/Valkey/Iceberg versions, latency numbers) was checked
> — see the dated **Research Verification Log** in Appendix 6. Re-run that log's checks each term,
> since version numbers and adoption percentages drift.

---

## Appendix 4 — Glossary

*(Quick-reference definitions for the terms students will meet. Keep on the course wiki.)*

- **ACID / BASE** — transactional guarantees (Atomicity/Consistency/Isolation/Durability) vs.
  Basically-Available/Soft-state/Eventual-consistency.
- **CAP / PACELC** — the partition-time C-vs-A choice; PACELC adds the normal-time Latency-vs-C choice.
- **Consistent hashing** — hashing keys and nodes onto a ring so membership changes move ~K/N keys.
- **CQRS / Event sourcing** — separate read/write models; persist state as a log of events.
- **CRDT** — data type that merges concurrent updates without coordination.
- **Fan-out (write vs. read)** — push updates to followers' timelines at write time vs. assemble at
  read time.
- **Idempotency** — applying an operation many times equals applying it once.
- **Little's Law** — `L = λ × W` (concurrency = arrival rate × latency).
- **LSM-tree / B-tree** — write-optimized log-structured storage vs. read-optimized in-place tree.
- **Quorum (W+R>N)** — read/write overlap that guarantees seeing the latest write.
- **Replication lag** — the delay before a write reaches a follower replica.
- **Saga** — a long transaction split into local steps with compensating actions.
- **SLI / SLO / SLA / error budget** — measured indicator / internal target / external contract /
  allowed unreliability.
- **Tail latency (p99)** — the slow end of the latency distribution that dominates UX at scale.
- **Thundering herd / cache stampede** — many requests hitting the DB at once when a hot key expires.

---

## Appendix 5 — Instructor Notes

**How to prepare each lecture (a repeatable checklist for you):**
1. Open with a **real story** — an outage postmortem or a scale milestone that the concept explains
   (e.g., the 2016 Dyn DNS outage for Module 3; the "celebrity problem" at Twitter for Module 16).
2. Draw the **dependency line back** — remind students which earlier building block this composes.
3. Teach to the **misconception** listed in the module — say the wrong model out loud, then correct it.
4. End every concept on its **trade-off** — never leave a component looking free.
5. Keep a **running system diagram** on the board that grows all semester as new pieces are added.

**Common failure points to watch for in students (and the fix):**
- Reciting CAP without understanding → make them reason about a specific partition scenario.
- Skipping estimation → refuse to accept a design that hasn't been sized.
- "Just add a cache/queue/shard" → always ask "what does that cost, and what breaks?"
- Over-engineering → ask "would one Postgres box handle this? prove it can't first."
- Designing from entities → force them to list the queries before the schema.

**On labs:** ship a Docker-based starter repo so setup never eats class time. Each lab should end
with a *measurement* (latency, hit rate, keys moved, lag observed) — seeing the number is the point.

**On studios:** you facilitate, students design. Your job is to ask the clarifying questions a good
interviewer asks, keep them on the rubric, and surface the trade-offs — not to give the answer.

---

## Appendix 6 — 2026 Technology Snapshot & Research Verification Log

*Everything here was **live-verified in July 2026** against the linked sources. Use it to keep
lecture slides current, and re-check the fast-moving rows each term — versions and adoption
percentages drift. Teach the **concept**; treat the specific numbers as perishable.*

### A. Current state of the tools named in the course

| Area | 2026 state to teach | Why it matters for the course |
|---|---|---|
| **HTTP protocols** | HTTP/2 ≈ 51%, HTTP/1.x ≈ 28%, **HTTP/3 ≈ 21%** (Cloudflare Radar). HTTP/3 peaked ~28% in 2023, then *retreated/plateaued*. | Great teaching point: "newest ≠ dominant." HTTP/3's QUIC benefits are real but adoption is gated by middleboxes/UDP handling. |
| **Kafka** | **4.0** (Mar 2025) removed ZooKeeper — **KRaft-only**; **4.2** shipped Feb 2026. Queues/KIP features maturing. | Reinforces the consensus module (KRaft = Raft for metadata) and "operational simplicity" as a design goal. |
| **Redis / Valkey** | Redis relicensed Mar 2024 (SSPL/RSAL; added AGPLv3 May 2025). **Valkey** (Linux Foundation; AWS/Google/Oracle) forked from Redis 7.2.4 and is now the **default in AWS ElastiCache/MemoryDB** and most Linux distros. Valkey 9.x in 2026. | Teach "Redis-compatible" as the interface; note that open-source licensing shapes real infra choices. |
| **Service mesh** | **Istio Ambient (sidecar-less)** GA since 1.22 (2024); **~1.29 in Apr 2026**, now the recommended model — per-node **ztunnel** (L4/mTLS) + optional **waypoint** (L7). ~70% less memory than sidecars. Cilium pushes mesh into the kernel via eBPF. | Concrete example of the 2026 shift away from per-pod sidecars; ties to the mTLS/observability discussion. |
| **Vector search** | **pgvector 0.8.x** (+ pgvectorscale) — parallel HNSW builds, **iterative index scans** for filtered search, **halfvec** quantization. Single-digit-ms queries under ~5M vectors. Dedicated stores (Qdrant, Milvus, Weaviate, Pinecone) for billion-scale. | The 2026 default advice: **"just use pgvector"** unless scale/latency forces a specialist — a clean "don't over-engineer" lesson. |
| **Lakehouse** | **Apache Iceberg** is the default for open lakehouses; **Iceberg v3 GA mid-2026** (deletion vectors, row lineage, VARIANT). AWS **S3 Tables** native. Delta Lake remains best inside Databricks/Fabric (UniForm gives Iceberg compatibility). | Illustrates OLTP-vs-OLAP separation and how open formats decouple storage from compute engines. |
| **Observability** | **OpenTelemetry** is the vendor-neutral standard (traces + metrics + logs, one SDK/collector). eBPF-based zero-instrumentation tooling rising. | Anchors Module 13; students should learn OTel, not a single vendor. |
| **Latency numbers** | Confirmed order-of-magnitude: NVMe random read **~20–70 µs**, intra-DC RTT **~0.3–0.5 ms**, SATA SSD ~100–200 µs, HDD seek ~5–10 ms, cross-continent RTT ~150 ms. | The estimation backbone (Module 2 / Appendix 2) is current. |

### B. Canonical curricula — verified tables of contents

- **DDIA 2nd edition (2026):** 14 chapters — full list in Appendix 3. Note the renumbering vs. the
  1st edition and the new vector/dataframe/local-first material.
- **Alex Xu Vol 1 (16 ch.) & Vol 2 (13 ch.):** verified — full lists in Appendix 3.
- **System Design Primer:** topic index (performance/scalability, latency/throughput, CAP,
  consistency & availability patterns, DNS, CDN, load balancer, reverse proxy, application layer,
  databases, cache, asynchronism, communication, security) + 8 worked questions + 7 OOD questions —
  all confirmed against the repo.
- **MIT 6.5840 (Spring 2026):** lecture arc + 5 Go labs — confirmed (see Appendix 3).

### C. Research method & honesty note

The **first** research pass (in an earlier session) ran while Claude's web-search backend was
returning sustained overload errors, so it fell back to built-in knowledge current to Jan 2026.
This **second pass was fully live** (July 2026) and produced the corrections above — most notably
the HTTP/3 share (was overstated as "~⅓") and the DDIA 2nd-edition chapter numbering. The
underlying **concepts, teaching order, and pedagogy did not change** — only the perishable
specifics were corrected and enriched.

**Primary sources verified this pass:** Cloudflare Radar (HTTP adoption); O'Reilly & dataintensive.net
(DDIA 2e ToC); bytebytego.com / Amazon (Xu Vol 1 & 2); github.com/donnemartin/system-design-primer;
pdos.csail.mit.edu/6.5840 (schedule + labs); kafka.apache.org (4.x releases); istio.io & KubeCon EU
2026 coverage (Ambient); github.com/pgvector/pgvector & postgresql.org (0.8.x); Linux Foundation /
TechCrunch / AWS (Valkey); Dremio / Apache (Iceberg v3); jboner latency-numbers gist (verified).

---

*End of roadmap. Next step (optional): I can expand any single module into full lecture notes +
slides outline + the complete lab handout, or generate the Docker-based starter repo for the labs.*
