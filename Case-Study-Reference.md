# Case-Study Reference — Real Architectures for the Part E Studios

> **Instructor companion to the [System Design Course Roadmap](System-Design-Course-Roadmap.md).**
> Every case study in Modules 15–17 is backed here by how the *real* system is (or plausibly is)
> built, with concrete numbers and primary sources. Use these to run the design studios: let
> students derive the design with the rubric first, then reveal what the real companies actually did
> and *why* — the gap between the two is where the learning happens.
>
> **Live-verified July 2026.** Sources are engineering blogs, canonical write-ups, and reputable
> system-design references. Where a company has never published exact internals (e.g., bit.ly), the
> design is the well-established community consensus and is labeled as such.
>
> **How to teach each one:** run the [Universal Design Rubric](System-Design-Course-Roadmap.md#appendix-1--the-universal-design-rubric)
> — requirements → estimation → API → data model → high-level → deep dive → scale & failure → trade-offs.
> Each entry below is organized around the **one key challenge** it exists to teach.

---

## Tier 1 — Easy Systems (Module 15)

*Purpose: internalize the rubric on tractable problems where one or two ideas dominate.*

### 1. URL Shortener (TinyURL / bit.ly)

- **Key challenge it teaches:** unique ID generation + a massively read-heavy path.
- **The design:**
  - **ID generation — two families:** (a) **counter-based** — a distributed counter mints a unique
    integer, then **Base62-encode** it to a short code; guaranteed-unique and predictable. (b)
    **Snowflake IDs** — 64 bits packing `timestamp | machine ID | per-ms sequence`, so every app
    server mints IDs locally with **no coordination**. Hashing (MD5/SHA + truncate) is the third
    option but needs collision handling.
  - **Base62** (`[A-Za-z0-9]`) keeps codes URL-safe; **7 characters ≈ 3.5 trillion** combinations.
  - **Storage:** a partitioned key-value/NoSQL store (DynamoDB/Cassandra) with the short code as the
    primary key — billions of rows, simple point lookups, high write throughput, easy horizontal
    scaling. A relational DB is fine at smaller scale.
  - **Read path:** hot codes in **Redis (LRU)**; a **CDN** absorbs most repeat redirect traffic; the
    KV store is the source of truth. Redirect returns **301 (permanent, cacheable)** vs **302
    (temporary, lets you count clicks)** — a nice trade-off discussion.
- **Estimation hook:** read:write ratio is often **~100:1** — this asymmetry is the whole reason the
  read path gets a CDN + cache and the write path just needs a fast ID mint.
- **Teaching point:** the "unique ID generator" (Snowflake) is itself a reusable building block that
  reappears in the feed, chat, and payment systems.
- **Sources:** [Design Bitly (System Design Handbook)](https://www.systemdesignhandbook.com/guides/design-bitly/) ·
  [Snowflake IDs + Base62 (DEV)](https://dev.to/speaklouder/url-shortener-using-snowflake-ids-and-base62-encoding-4179) ·
  [DesignGurus walkthrough](https://www.designgurus.io/blog/url-shortening)

### 2. Distributed Rate Limiter

- **Key challenge it teaches:** correctness under concurrency (atomicity) in a distributed counter.
- **The design:**
  - **Token bucket** is the workhorse — **both Amazon and Stripe** use it to throttle their APIs.
    Bucket of capacity B, refilled at rate R; allows bursts up to B. Tiny memory footprint.
  - **Sliding-window log** for precision: store each request's timestamp in a Redis **sorted set**,
    evict entries older than the window, count the rest — no fixed-window boundary bursts.
  - **The subtle bug (great teaching moment):** token bucket's Redis ops are **not atomic** — the
    naive "read-then-write" is a race condition that makes the limiter too lenient under load. **Fix:
    a Redis Lua script** makes the whole read-calculate-update decision atomic.
  - Return **`429 Too Many Requests` + `Retry-After`**. Scope per user / IP / API key / endpoint.
- **Real-world stakes:** **Figma's** Redis rate limiter stopped a spam attack that mass-invited random
  emails (which would have meant skyrocketing email costs + reputation damage). **Stripe** uses rate
  limiting to stop a buggy client from accidentally firing thousands of payment requests.
- **Teaching point:** this is the cleanest example in the course of "distributed = the obvious
  single-node algorithm is now subtly wrong." Directly reinforces Module 11.
- **Sources:** [Figma — an alternative approach to rate limiting](https://www.figma.com/blog/an-alternative-approach-to-rate-limiting/) ·
  [Redis — build 5 rate limiters](https://redis.io/tutorials/howtos/ratelimiting/) ·
  [ByteByteGo — Design a Rate Limiter](https://bytebytego.com/courses/system-design-interview/design-a-rate-limiter)

### 3. Unique ID Generator + 4. Key-Value Store + 5. Pastebin

- **Unique ID generator (Snowflake):** covered in #1 — teach the 64-bit layout, clock-skew handling,
  and why local minting beats a central auto-increment (no coordination bottleneck).
- **Distributed key-value store (Dynamo-in-miniature):** the capstone of Part C — **consistent
  hashing** for placement, **quorums (W+R>N)** for tunable consistency, **replication + read-repair +
  anti-entropy (Merkle trees)** for convergence. This is where Modules 7/9/10 come together.
- **Pastebin:** object/blob storage for the paste body + a metadata row (id, expiry, visibility) +
  the URL-shortener ID scheme for the key. A gentle warm-up to the storage-vs-metadata split that
  returns in Dropbox and YouTube.

---

## Tier 2 — Medium Systems (Module 16)

*Purpose: fan-out, real-time delivery, and search-shaped problems.*

### 6. News Feed / Twitter (X) Timeline — **the canonical fan-out lesson**

- **Key challenge it teaches:** **fan-out on write vs. on read**, and the **celebrity problem**.
- **The design:**
  - **Fan-out on write (push):** when a user posts, push the tweet into every follower's
    precomputed timeline (cache). Reads are then trivial — just read your list. But **write
    amplification** is brutal for large accounts.
  - **Fan-out on read (pull):** assemble the timeline at read time by fetching from everyone you
    follow. Cheap writes, expensive reads.
  - **The celebrity problem, with real numbers:** Twitter internally fought this for years. A
    celebrity tweet can trigger **~30 million timeline updates**; at 1 ms each that's **8 hours of
    sequential work**. When **Katy Perry (~80M followers)** tweeted, a pure-push design attempted
    ~80M cache writes and **saturated the write path for 3–5 seconds**, delaying *everyone's* tweets.
  - **The hybrid solution (what to actually teach):** **push for normal users** (< ~10K followers,
    manageable amplification); **pull for celebrities** (> ~10K followers — store their tweets
    separately). On read, **merge** the precomputed timeline with a real-time fetch of the few
    celebrity accounts you follow. The ~10K threshold is a **business/SLA decision, not a formula.**
- **Scale anchor (2013 numbers, still great for teaching):** ~150M active users, ~300K QPS on the
  timeline read path, a ~22 MB/s firehose, target tweet delivery **under 5 seconds**.
- **Teaching point:** there is no "right" answer — the design *is* the hybrid, chosen from the
  follower-count distribution. This is the exemplar of "name the trade-off."
- **Sources:** [The Architecture Twitter Uses… (High Scalability)](http://highscalability.com/blog/2013/7/8/the-architecture-twitter-uses-to-deal-with-150m-active-users.html) ·
  [Twitter's Fanout Strategy at Scale (DEV)](https://dev.to/gabrielanhaia/twitters-fanout-strategy-at-scale-the-trade-off-most-designs-miss-55oa) ·
  [Twitter/X News Feed deep dive (techinterview)](https://www.techinterview.org/post/3233474168/system-design-twitter-news-feed-timeline-fanout-on-write-fanout-on-read-celebrity-problem-ranking-caching/)

### 7. Chat / Messaging (WhatsApp)

- **Key challenge it teaches:** massive persistent connections + real-time delivery on stateful servers.
- **The design (WhatsApp specifics):**
  - **Erlang/BEAM** for connection handling: each user connection is its own lightweight Erlang
    process using **~300 bytes** each; the runtime is built for massive concurrency + soft failure +
    fast recovery. Historically optimized to **~2 million connections per server** (socket-limited,
    not memory-limited — a 64 GB box could theoretically host 200M+ *processes*).
  - **Mnesia** for routing state; a **binary protocol** on the wire; **direct delivery** out.
  - **Scale, with real numbers:** **~40 billion messages/day** (some sources cite 100B+), **billions
    of users**, historically **>70 million Erlang messages/second**, hundreds of nodes / thousands of
    cores / hundreds of TB of RAM — famously run by **~50 engineers**.
  - **Delivery semantics:** store-and-forward when the recipient is offline; delete on delivery;
    delivery/read receipts; presence; group fan-out.
- **Teaching point:** the counterpoint to the stateless-tier gospel — sometimes **stateful,
  connection-oriented** design (WebSockets + a concurrency-optimized runtime) is exactly right. Ties
  back to Module 3 (WebSockets) and Module 5 (stateful services).
- **Sources:** [How WhatsApp Handles 40B Messages/Day (ByteByteGo)](https://blog.bytebytego.com/p/how-whatsapp-handles-40-billion-messages) ·
  [WhatsApp → 500M users, 11k cores (High Scalability)](https://highscalability.com/how-whatsapp-grew-to-nearly-500-million-users-11000-cores-an/) ·
  [WhatsApp architecture deep dive (GetStream)](https://getstream.io/blog/whatsapp-works/)

### 8. Search Autocomplete / Typeahead

- **Key challenge it teaches:** a hard per-keystroke latency budget forces **read/build path separation**.
- **The design:**
  - **Trie** for prefix matching (beats hash tables / BSTs for prefixes). **Precompute the top-K
    suggestions at each node** so a lookup is a short walk, not a subtree scan.
  - **The binding constraint:** "*latency forbids ranking at read time; relevance forbids skipping
    ranking → so rank offline.*" You cannot rank or hit disk per keystroke.
  - **Two decoupled halves** (a clean architecture lesson): a **hot read path** (client → cache →
    router → trie shard) and an **offline build path** (search/click events → pub-sub log →
    aggregation computes popularity → builds a fresh ranked trie → hot-swaps it onto the serving
    tier on a schedule).
  - **Cache** short, hugely-popular prefixes ("a", "th") in front of the serving tier. Track **p50/
    p95/p99 latency**, not averages.
- **Teaching point:** the exemplar of **precomputation** and **read/write path separation** — the
  same pattern as the news feed's fan-out-on-write.
- **Sources:** [Typeahead system design (System Design School)](https://systemdesignschool.io/problems/typeahead/solution) ·
  [Designing typeahead (TianPan)](https://tianpan.co/notes/179-designing-typeahead-search-or-autocomplete) ·
  [Trie at scale (DEV)](https://dev.to/matt_frank_usa/designing-search-autocomplete-trie-data-structures-at-scale-5f4k)

### 9. Web Crawler

- **Key challenge it teaches:** the **URL frontier** (priority + politeness) and probabilistic dedup.
- **The design:**
  - **URL frontier** manages **billions of URLs** via **consistent hashing + domain-based sharding**.
    A **two-level frontier**: *front queues* prioritize by PageRank/freshness; *back queues* enforce
    **politeness (~1 req/sec/domain)**. Each domain's queue is owned by exactly one worker pool — a
    single global FIFO makes politeness nearly impossible.
  - **Dedup with a Bloom filter:** no false negatives; **10B URLs × 10 bits ≈ 12 GB fits in RAM** at
    ~1% false-positive rate. **Two-tier:** Bloom filter for a fast reject, then an exact Redis check
    for the small fraction that passes.
  - Frontier, Bloom filter, and DNS resolver are **distributed microservices**; workers scale
    horizontally and **treat crashed workers as routine**.
- **Teaching point:** Bloom filters and consistent hashing (both taught earlier) do the heavy
  lifting — a great "the building blocks pay off" moment.
- **Sources:** [Design a Distributed Web Crawler (System Design Sandbox)](https://www.systemdesignsandbox.com/learn/design-web-crawler) ·
  [Web Crawler guide (Grokking)](https://grokkingthesystemdesign.com/guides/web-crawler-system-design/)

### 10. Notification System

- **Key challenge it teaches:** multi-channel fan-out with dedup, provider rate limits, and retries.
- **The design:** a common notification interface with **per-channel adapters** (push → FCM/APNs,
  email → SES, SMS → Twilio). Core concerns: **fan-out**, **priority queues**, **idempotency/dedup**
  (client sends a unique ID; check against **Redis** before sending), **retries with backoff +
  circuit breakers**, **provider rate limits** (token/leaky bucket at the gateway + Redis counters
  with TTL for per-user caps like "≤5 SMS/day"), scheduling + leader election, and **webhook
  reconciliation** for delivery status.
- **Scale anchor:** designs routinely target **~50M devices**; APNs/FCM connection management is the
  push-path bottleneck.
- **Teaching point:** this is Part D (Module 11/12) applied — nearly every resilience primitive shows
  up in one system.
- **Sources:** [Push notifications to 50M devices (DesignGurus)](https://designgurus.substack.com/p/push-notification-architecture-apns) ·
  [Notification system design (Ajit Singh)](https://singhajit.com/notification-system-design/)

---

## Tier 3 — Hard Systems (Module 17)

*Purpose: integrate storage, streaming, geospatial, and money under real constraints.*

### 11. Video Streaming (Netflix / YouTube)

- **Key challenge it teaches:** an offline **transcoding pipeline** + a purpose-built **CDN**, with
  control plane / data plane separation.
- **The design (Netflix specifics):**
  - **Control plane vs. data plane:** backend on **AWS** handles everything *before* you hit play
    (auth, browsing, manifests); the **data plane is Open Connect**, Netflix's own CDN.
  - **Transcoding pipeline:** a studio master (**4K, 50+ GB**) is converted into **dozens of
    encodings** (resolutions/bitrates/codecs) for different devices and network conditions —
    petabytes processed daily. Enables **adaptive bitrate** (HLS/DASH) so the client switches quality
    on the fly.
  - **Open Connect CDN:** **18,000+ servers across 6,000+ locations**; **Open Connect Appliances
    (OCAs)** are placed **inside ISP networks**, each pre-loaded with **100+ TB** of popular content.
    On play, the control plane picks the closest OCA; **video traffic never crosses the internet
    backbone** — it stays inside the ISP.
- **Teaching point:** the definitive **CDN + precompute** case, and a clean **control/data plane**
  split. Contrast push-CDN (Netflix pre-positions content) vs. pull-CDN (most of the web).
- **Sources:** [Netflix Open Connect deep dive (Medium/Harsh Jain)](https://medium.com/@hjain5164/inside-netflixs-video-streaming-delivery-architecture-e2c848e98a85) ·
  [Netflix tech stack — Open Connect (VdoCipher)](https://www.vdocipher.com/blog/netflix-tech-stack-and-architecture/) ·
  [Netflix TechBlog](https://netflixtechblog.com/)

### 12. Ride-Hailing / Proximity (Uber) — **the geospatial lesson**

- **Key challenge it teaches:** **geospatial indexing** to turn "nearest driver" from millions of
  distance calcs into dozens of cell lookups.
- **The design (Uber specifics):**
  - **H3 hexagonal hierarchical index** (Uber-built, open-sourced): models Earth as an icosahedron
    subdivided into **hexagons**, each a **unique 64-bit ID**. **122 base cells** (12 pentagons),
    **16 resolution levels**; finest ≈ 1 m². Hexagons are chosen because **all neighbors are
    equidistant** from center (unlike squares/triangles), simplifying distance logic.
  - **Matching:** convert rider + driver lat/long into H3 cells; use **`kRing(cell, k)`** to expand
    the search to neighboring cells. At **resolution 8** (~0.74 km²/cell), Uber/Grab find the nearest
    driver in **<100 ms** by checking **~7 cells** instead of every driver — millions of comparisons
    reduced to dozens.
  - **Storage:** driver location as **(H3 cell ID, timestamp)** in **Cassandra** (write-optimized,
    highly available) — a torrent of location pings.
- **Teaching point:** the right **data structure** collapses the problem. Alternatives to compare:
  **geohash** (string prefix), **quadtree** (adaptive subdivision). Ties to Module 4 (indexing) and
  Module 9 (partitioning by cell).
- **Sources:** [H3: Uber's Hexagonal Hierarchical Spatial Index (Uber Eng)](https://www.uber.com/us/en/blog/h3/) ·
  [How Uber finds nearby drivers with H3 (BestHub)](https://www.besthub.dev/articles/how-uber-uses-h3-hexagonal-indexing-to-power-real-time-driver-matching-cc74c0fa2742)

### 13. File Storage & Sync (Dropbox)

- **Key challenge it teaches:** **content-defined chunking + block-level dedup**, and the build-vs-buy
  storage decision.
- **The design (Dropbox specifics):**
  - **Chunking:** files split into **~4 MB blocks**, each identified by a **SHA-256 hash**
    (content-addressable). **Delta sync:** on change, only the **changed blocks** are uploaded.
  - **Dedup:** if a block's hash already exists, the metadata service just adds a reference row — the
    bytes are never re-uploaded (saves bandwidth and storage across all users).
  - **Metadata vs. blocks split:** a metadata DB (files → ordered block list, versions, sharing)
    separate from the block store — the same split as Pastebin/YouTube, at extreme scale.
  - **Magic Pocket:** in **2015–2016 Dropbox migrated off S3** to its own **immutable, erasure-coded**
    block store on spinning-disk **JBOD** across multiple datacenters. Blocks up to 4 MB, encrypted,
    **write-once/never-changed**. The move **cut storage costs ~40%** and gave them control over
    durability. (Great "when does building your own infra pay off?" discussion.)
- **Teaching point:** hashing + immutability + erasure coding compose into a real storage system;
  and the economics of build-vs-buy at scale.
- **Sources:** [Inside the Magic Pocket (Dropbox Tech)](https://dropbox.tech/infrastructure/inside-the-magic-pocket) ·
  [How Dropbox delta sync works (Akshay Ghalme)](https://akshayghalme.com/blogs/how-dropbox-delta-sync-works/) ·
  [How Dropbox scaled storage (Scale Engineer)](https://scaleengineer.com/blog/how-dropbox-scaled-its-storage-infrastructure)

### 14. Payment System — **the exactly-once / consistency lesson**

- **Key challenge it teaches:** never double-charge — the strongest consistency demands in the course.
- **The design (Stripe specifics):**
  - **Idempotency keys:** the client sends a unique **`Idempotency-Key`** header on mutating
    requests. If Stripe has seen the key before (**within 24 h**), it returns the **cached response**
    instead of reprocessing — so a retried request is processed **exactly once**. This is the
    concrete realization of "exactly-once effect = at-least-once delivery + idempotent processing"
    from Module 10.
  - **Double-entry ledger:** every money movement writes **balanced debit/credit entries**; the
    ledger is the **single source of truth** for balances, fees, refunds, disputes — not just
    accounting, but the **consistency foundation**.
  - **Layered safety:** Stripe/Adyen/Block stack **API-level idempotency → DB-level optimistic
    locking → immutable double-entry bookkeeping** to move trillions with near-perfect reliability.
- **Teaching point:** where CAP/consistency stops being academic — here you choose **consistency over
  availability** every time, and idempotency is non-negotiable. Directly pays off Module 10.
- **Sources:** [Designing APIs with idempotency (Stripe Blog)](https://stripe.com/blog/idempotency) ·
  [How Stripe prevents double payment (System Design One)](https://newsletter.systemdesign.one/p/idempotent-api) ·
  [Payment Intents API (Stripe Docs)](https://docs.stripe.com/payments/payment-intents)

### 15. Ad-Click Aggregation — **the stream-processing lesson**

- **Key challenge it teaches:** high-throughput, **exactly-once** real-time aggregation.
- **The design (the "KFC" stack — Kafka, Flink, ClickHouse):**
  - **Ingest:** click events → **Kafka** (durable, replayable log).
  - **Process:** **Apache Flink** does **windowed aggregation with event-time semantics** (out-of-
    order clicks land in the correct minute bucket via **watermarks**). **Exactly-once** comes from
    **periodic checkpoints** (state snapshotted to S3/HDFS) **+ a two-phase commit** on output — on
    crash, a new node reloads the last checkpoint and replays Kafka from that offset.
  - **Serve:** aggregates land in an **OLAP columnar store (ClickHouse / Apache Druid / Pinot)** for
    fast real-time dashboards.
  - **Uber's real system** does exactly this — **exactly-once ad event processing with Flink + Kafka
    + Pinot**.
  - **Architecture evolution to teach:** **Lambda** (parallel batch + speed layers → duplicate code +
    constant reconciliation) → **Kappa** (a single streaming path) — the modern preference.
- **Teaching point:** the payoff of Module 12 — delivery guarantees, event-time vs. processing-time,
  watermarks, and checkpointing all become concrete here.
- **Sources:** [Real-Time Exactly-Once Ad Event Processing (Uber Eng)](https://www.uber.com/us/en/blog/real-time-exactly-once-ad-event-processing/) ·
  [The KFC Architecture: Kafka, Flink, ClickHouse (BigDataBoutique)](https://bigdataboutique.com/blog/kfc-architecture-blueprint-kafka-flink-and-clickhouse) ·
  [Ad Click Aggregator (Hello Interview)](https://www.hellointerview.com/learn/system-design/problem-breakdowns/ad-click-aggregator)

---

## The Recurring Patterns (say these out loud as they reappear)

These are the connective tissue — point at them every time a case study reuses one. By Module 17,
students should recognize them on sight:

| Pattern | Shows up in |
|---|---|
| **Unique ID generation (Snowflake / counter + Base62)** | URL shortener, feed, chat, payments |
| **Consistent hashing** | KV store, web crawler frontier, sharded caches |
| **Fan-out on write vs. read (+ hybrid)** | News feed, notifications |
| **Precompute + read/write path separation** | Feed (fan-out-on-write), typeahead, video transcoding |
| **CDN / edge + push vs. pull** | URL shortener, Netflix, static assets everywhere |
| **Geospatial cell indexing (H3 / geohash / quadtree)** | Uber, proximity, "nearby X" |
| **Content-addressable storage + dedup (hashing)** | Dropbox, Pastebin, any blob store |
| **Bloom filter for cheap probabilistic dedup** | Web crawler, cache penetration defense |
| **Idempotency keys → exactly-once effect** | Payments, notifications, any retried write |
| **Stream processing: event-time + watermarks + checkpoints** | Ad-click aggregation, real-time analytics |
| **Metadata / blob split** | Dropbox, YouTube, Pastebin |
| **Tail latency (p99) governs the design** | Typeahead, feed, every read path |

---

## How to Run a Studio (90-minute template)

1. **(10 min)** Put the prompt on the board. Students ask clarifying questions; you play interviewer,
   pinning down functional + non-functional requirements and scope.
2. **(10 min)** Estimation together — QPS, storage, bandwidth. Let the numbers *force* decisions.
3. **(25 min)** Students propose the high-level design (API → data model → boxes). You steer with "what
   breaks at 10×?" and "what does that cost?"
4. **(20 min)** Deep-dive the one key challenge (the fan-out, the geo-index, the idempotency).
5. **(10 min)** **The reveal:** show what the real company did (this doc) and *why it differs* from the
   class design. The gap is the lesson.
6. **(15 min)** Failure modes + trade-offs wrap-up. Assign the written version as homework.

---

*Companion to [System-Design-Course-Roadmap.md](System-Design-Course-Roadmap.md). Verified July 2026.
Re-check the scale numbers periodically — architectures evolve, but the patterns are durable.*
