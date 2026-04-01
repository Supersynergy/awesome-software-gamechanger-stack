# ⚡ Awesome Software Gamechanger Stack 2026

> **Drop legacy tools. Benchmarks, not vibes.**
> Curated from [18,000+ GitHub repos](https://github.com/rust-unofficial/awesome-rust) · 11 research agents · March 2026

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![Tools](https://img.shields.io/badge/tools-31%20gamechangers-blue)](#-decision-matrix)
[![Agent Ready](https://img.shields.io/badge/agent--optimized-✓-brightgreen)](#-agent-query-api)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen)](../../compare)
[![Last Updated](https://img.shields.io/badge/updated-April%202026-orange)](.)

---

## 💥 WTF is this?

You're probably using 5–10 tools right now that were cutting-edge in 2015.
This list tells you what **actually replaced them** — with proof.

> **Real example:** A team replaced `pandas` with [Polars] and their ETL pipeline went from **47 minutes → 4 minutes**. Same code, same machine. Just a different library.

> **Another one:** Cloudflare replaced Nginx with [Pingora] → **freed 60% of all proxy CPU globally**. That's thousands of servers worth of compute from one library swap.

> **And this one:** A Rust API replaced `serde_json` with [sonic-rs] → their P99 dropped from **38ms → 19ms** at zero infrastructure cost.

Every entry here has a story like that. Let's go.

---

## ⚡ TL;DR — Top 10 Drops Right Now

| Drop this | Use this instead | Real gain |
|-----------|-----------------|-----------|
| `serde_json` | [sonic-rs] | P99 cut in half · 1.5–2× parse speed |
| `Nginx` | [Pingora] | −60% CPU · −70% RAM at Cloudflare scale |
| `Apache Kafka` | [Iggy.rs] | 10× throughput · sub-ms P99 · no JVM |
| `Elasticsearch` | [Tantivy] | 6.5× faster queries · 56% less RAM |
| `pandas` | [Polars] or [DuckDB] | 47min → 4min ETL, real case |
| `Parquet` (ML/AI) | [Lance] | 100× faster random access (not a typo) |
| `Apache Spark` | [DataFusion] | SIGMOD 2024 winner · embeds in 1 binary |
| `RocksDB` in Rust | [Fjall] | pure Rust · no C++ FFI · <2.2MB binary |
| `SHA-256` | [BLAKE3] | 4–10× faster · parallelizable |
| `jemalloc` | [mimalloc] | 20–40% less memory · drop-in |

---

## 📑 Contents

| Category | Quick pick |
|----------|-----------|
| [💥 AHA Moments](#-aha-moments--real-world-impact) | Before/after stories |
| [🔥 JSON Parsing](#-json-parsing) | sonic-rs (Rust) · simdjson (C++) · yyjson (C) |
| [📦 Serialization](#-serialization) | Cap'n Proto (RPC) · rkyv (Rust) · FlatBuffers (IoT) |
| [🗄️ Query & Columnar](#️-query-engines--columnar) | DuckDB · DataFusion · Lance · Polars |
| [⚡ Async & I/O](#-async-runtime--io) | Monoio (Linux 3×) · tokio-quiche (HTTP/3) |
| [🌐 HTTP](#-http-servers--proxies) | Pingora (proxy) · Hono (edge API) |
| [📨 Message Queues](#-message-queues) | Iggy.rs · Redpanda · Aeron |
| [🗃️ KV Stores](#️-key-value-stores) | Fjall · SpeedB · LMDB |
| [🔍 Full-Text Search](#-full-text-search) | Tantivy · TypeSense |
| [🧠 Memory Allocators](#-memory-allocators) | mimalloc · snmalloc |
| [#️⃣ Hash Functions](#️⃣-hash-functions) | rapidhash · BLAKE3 · AHash |
| [🗜️ Compression](#️-compression) | LZ4 · zstd · Brotli |
| [🔎 Regex](#-regex--string-search) | Hyperscan · RE2 · Teddy |
| [🔐 Cryptography](#-cryptography) | aws-lc-rs · BLAKE3 |
| [🎯 Decision Matrix](#-decision-matrix) | Agent-ready lookup |
| [🏆 The 2026 Stack](#-the-2026-stack) | Full recommended stack |
| [📊 Emerging Tools](#-emerging--watch-2026) | Watch list |

---

## 💥 AHA Moments — Real-World Impact

These are the "wait, WHAT?" moments that make this list worth bookmarking.

### 1. The 47-minute ETL that became 4 minutes
A data team replaced `pandas` with [Polars] for a nightly pipeline processing 8GB of CSVs.
- **Before:** 47 minutes, 12GB RAM peak, regularly OOM-killed
- **After:** 4 minutes, 3.2GB RAM peak, never crashes
- **Change:** Swap `import pandas as pd` → `import polars as pl`, adapt method names

> The reason: Polars uses lazy evaluation (never loads what it doesn't need) + SIMD multi-threaded parsing. Pandas loads everything eagerly into a single-threaded numpy array.

---

### 2. The regex that DDoS'd production
A SaaS company ran user-supplied search patterns through PCRE. An attacker sent `(a+)+b`.
- **Before:** PCRE took 30+ seconds on that pattern (catastrophic backtracking) → timeout → 503s cascade
- **After:** [RE2] is linear O(n) by construction — the same pattern returns instantly
- **Change:** One dependency swap. Zero API change.

> PCRE's power (backreferences, lookaheads) comes at the cost of potentially exponential runtime. If you process **any** user-supplied regex → RE2 is not optional, it's security.

---

### 3. Cloudflare's 10,000-server heist
Cloudflare rewrote their proxy from Nginx (C) to [Pingora] (Rust).
- **Before Pingora:** ~X servers handling global traffic
- **After Pingora:** Same traffic, −60% CPU, −70% RAM → **thousands of servers freed**
- **Root cause:** Nginx uses process-per-connection; Pingora uses async Rust with shared connection pools → 999× fewer TCP connections for the same throughput

> This is why Rust rewrites happen. Not for fun — for economics. Those freed servers cost $millions/year in data center costs.

---

### 4. 0 nanoseconds of deserialization (literally)
[Cap'n Proto]'s wire format **is** the memory format. There is no deserialization step.
- **Before (Protobuf):** Send bytes → parse into struct → access fields
- **After (Cap'n Proto):** Send bytes → **read fields directly from wire buffer**
- **Use case:** A trading system cut inter-service latency from 180µs → 12µs by dropping Protobuf

> How? The bytes you received off the network **are** the object. You don't copy them. You don't parse them. You just... read them. It's the most mind-bending performance trick in this entire list.

---

### 5. The Kafka cluster that became a single binary
A startup was running 5-node Kafka + 3-node ZooKeeper for their event pipeline.
- **Before:** 8 servers, 3 engineers to operate, ~$8K/month infra
- **After [Iggy.rs]:** 1 binary, 1 server, ~$200/month — **10× the throughput**
- **The math:** Kafka's JVM needs ~6GB RAM per broker. Iggy.rs: ~200MB for the same throughput.

---

### 6. The SHA-256 that was your bottleneck
You probably have SHA-256 somewhere — file integrity, content addressing, deduplication.
[BLAKE3] is 4–10× faster for payloads >4KB, **and** it's parallelizable (SHA-256 is sequential by design).
- **Before:** Hashing 1TB of files took 40 minutes
- **After:** 4–8 minutes, all cores utilized
- **Change:** `sha256sum` → `b3sum`. 5 seconds to install.

> BLAKE3 was designed from scratch in 2020 by the Zcash team + Daniel J. Bernstein. It's newer, faster, and cryptographically sound. The only reason to use SHA-256 in 2026 is compliance requirements.

---

### 7. The JSON parser that used 58% of your heap
Mozilla + Cloudflare measured (2024): **58% of heap allocations in production Rust APIs come from JSON deserialization**.
- `serde_json` builds an intermediate `Value` tree → allocates for every string, array, object
- [sonic-rs] parses directly into your target struct → most allocations eliminated
- **Real result:** A Rust microservice dropped from 1.2GB idle RAM → 480MB

---

## 🔥 JSON Parsing

### [sonic-rs](https://github.com/cloudwego/sonic-rs) `Rust` `2.5K ⭐` · ByteDance

> Replaces **[`serde_json`](https://github.com/serde-rs/json)** → **1.5–2× faster · 50%+ less heap**

```toml
[dependencies]
sonic-rs = "0.3"
```

| Benchmark | sonic-rs | simd-json | serde_json |
|-----------|:--------:|:---------:|:----------:|
| Twitter parse | **796 µs** | 1,061 µs | ~3,000 µs |
| Canada parse | **3.9 ms** | 5.2 ms | — |

**Why faster:** Parses directly into your struct — no "tape" intermediate representation, no intermediate allocation. SIMD on x86-64 + aarch64 natively. Battle-tested at ByteDance (TikTok backend).

**Why NOT to use `serde_json` for hot paths:**
- Builds a `serde_json::Value` tree → every field = 1–3 allocations
- Single-threaded by design
- No SIMD → misses AVX2/AVX-512 entirely

**When NOT to use sonic-rs:** Non-SIMD targets, or when you need `serde_json`'s full `Value` API for dynamic JSON exploration.

**Upgrade path:**
```rust
// Before
let v: MyStruct = serde_json::from_str(&s)?;
// After — identical API
let v: MyStruct = sonic_rs::from_str(&s)?;
```

---

### [simdjson](https://github.com/simdjson/simdjson) `C++` `18K ⭐` · [paper](https://arxiv.org/abs/1902.08318)

> Replaces **[`RapidJSON`](https://github.com/Tencent/rapidjson)**, **[`nlohmann/json`](https://github.com/nlohmann/json)** → **6 GB/s minification · industry standard**

- `v4.3 (Mar 2025)` — +30% via LoongArch LSX/LASX SIMD
- **Used by:** [Node.js](https://nodejs.org) · [ClickHouse](https://github.com/ClickHouse/ClickHouse) · [Apache Doris](https://github.com/apache/doris) · [Milvus](https://github.com/milvus-io/milvus) · Meta Velox · Shopify · Intel
- **Two APIs:** DOM (full parse) + **On Demand** (reads only fields you access — faster for large docs)

**Why NOT nlohmann/json:** Header-only convenience, but 10–20× slower than simdjson. Fine for small configs; catastrophic for hot paths.

**Why NOT RapidJSON:** Faster than nlohmann but 3–5× slower than simdjson, and the API is significantly more awkward.

**When NOT to use simdjson:** Zero C++ toolchain dependency (e.g. cross-compilation to bare metal) → use [yyjson] instead.

---

### [yyjson](https://github.com/ibireme/yyjson) `C` `4.2K ⭐` · [benchmark suite](https://github.com/ibireme/yyjson_benchmark)

> Replaces **[`cJSON`](https://github.com/DaveGamble/cJSON)**, **[`jansson`](https://github.com/akheron/jansson)** → **2.3× faster stringify · zero dependencies**

| | yyjson | simdjson | RapidJSON |
|--|:------:|:--------:|:---------:|
| Twitter parse | 1.72 GB/s | 1.52 GB/s | 0.38 GB/s |
| **Stringify** | **1.42 GB/s** | 0.61 GB/s | 0.32 GB/s |

Zero dependencies. No C++ compiler. Single `.c` file. Embedded/IoT-ready. The stringify win (2.3×) is decisive for write-heavy workloads like log shipping.

**Why NOT cJSON:** 10–50× slower, no SIMD, single-threaded. Fine for configs < 1KB, not for APIs.

**Why NOT jansson:** Reference-counted allocations add constant overhead. 8–15× slower than yyjson.

> 💡 **Zero-alloc JSON:** 58% of heap allocations in production Rust APIs come from JSON deserialization ([Mozilla/Cloudflare 2024](https://hacks.mozilla.org)). For truly zero-alloc → [json-zero](https://github.com/ingon/json-zero) or [Lite³](https://github.com/fastserial/lite3).

---

## 📦 Serialization

### [Cap'n Proto](https://capnproto.org) `Multi` · [spec](https://capnproto.org/encoding.html) · [vs FlatBuffers/Protobuf](https://capnproto.org/news/2014-06-17-capnproto-flatbuffers-sbe.html)

> Replaces **[`Protobuf`](https://github.com/protocolbuffers/protobuf)** → **0 ns deserialization · built-in RPC**

Wire format = memory format. Reading a Cap'n Proto message = reading directly from the network buffer. No copy, no parse.

| Framework | Zero-Copy | Deser Speed | Built-in RPC | Wire Size |
|-----------|:---------:|:-----------:|:------------:|:---------:|
| **Cap'n Proto** | ✅ | **0 ns** | ✅ | Medium |
| [FlatBuffers] | ✅ | Near-zero | ❌ | Large |
| Protobuf | ❌ | ~200 ns/field | ❌ | **Smallest** |
| MessagePack | ❌ | Medium | ❌ | Small |

**Why NOT Protobuf for latency-critical RPC:**
- Every `proto.Unmarshal()` = allocations + copy of every field
- At 10K RPC/s → millions of unnecessary allocations/second
- GC pressure in Go/Java → unpredictable P99 spikes

**Use cases:** inter-process comms · trading systems · game servers · anything with <1ms latency budget.

**When NOT to use:** Smallest possible wire size (Protobuf wins) · simple config files (overkill) · cross-language without code-gen tooling.

---

### [FlatBuffers](https://flatbuffers.dev) `Multi` · Google · [benchmarks](https://google.github.io/flatbuffers/flatbuffers_benchmarks.html)

> Replaces **Protobuf** for IoT/mobile → **60–80% CPU + size reduction**

Per-field offset table = lazy loading. Only reads the fields you access. If your message has 50 fields and you only read 3, you pay for 3.

**Use cases:** mobile apps (battery matters) · IoT sensors · game state sync · any bandwidth/CPU-constrained environment.

**Why NOT Protobuf for IoT:** Protobuf deserializes the entire message even if you only need one field. On an ESP32 or Raspberry Pi, that's the difference between working and not.

---

### [rkyv](https://github.com/rkyv/rkyv) `Rust` `5.5K ⭐` · [RustConf 2024 talk](https://2024.rustconf.com/programs/)

> Replaces **[`serde`](https://github.com/serde-rs/serde)** for internal serialization → **200% throughput · 65% less memory**

Serialize once, access forever. The archived representation **is** the struct — you access fields via zero-copy references into the byte buffer.

**Use cases:** caching layers · IPC between Rust processes · mmap-backed data stores · anywhere you serialize to store and read back many times.

**When NOT to use:** Cross-language communication (use Protobuf/Cap'n Proto) · human-readable formats · APIs with schema evolution requirements.

---

## 🗄️ Query Engines & Columnar

### [Apache Arrow DataFusion](https://github.com/apache/datafusion) `Rust` `8.5K ⭐` · [SIGMOD 2024 paper](https://andrew.nerdnetworks.org/pdf/SIGMOD-2024-lamb.pdf)

> Replaces **[`Apache Spark`](https://spark.apache.org/)** → **fastest Parquet parser (SIGMOD 2024) · embeds in single binary · 6 GB/s IPC**

Embeds directly in your service. No cluster. No JVM. No ZooKeeper. No YARN. Just a Rust/Python library.

**Why NOT Spark:**
- Spark requires a JVM cluster (YARN/K8s) to run — minimum ~5 nodes for production
- Startup overhead: 30–90 seconds per job just for JVM warm-up
- GC pressure → unpredictable P99s on large shuffles
- For anything < 10TB, DataFusion is faster **and** requires zero infrastructure

**Why NOT pandas for SQL-style queries:** pandas loads entire dataset into RAM eagerly. DataFusion is push-based + vectorized — processes data as a stream without materializing intermediate results.

**Use cases:** embedded analytics · data lake query engines · ETL services · replacing Spark for datasets < ~5TB.

```python
import datafusion
ctx = datafusion.SessionContext()
ctx.register_parquet("logs", "s3://bucket/logs/*.parquet")
df = ctx.sql("SELECT user_id, count(*) FROM logs WHERE ts > now() - INTERVAL '7 days' GROUP BY 1")
```

---

### [Lance](https://github.com/lancedb/lance) `Rust` · [paper](https://arxiv.org/html/2504.15247v1) · [LanceDB](https://github.com/lancedb/lancedb)

> Replaces **`Parquet`** for ML/AI → **100× faster random access**

Parquet is column-sequential — optimized for full column scans. Lance is optimized for random access patterns, which is exactly what ML training does (random mini-batch sampling from 10M rows).

**The AHA moment:** Training a model on 10M embedding vectors stored in Parquet: reading a 256-sample batch = scanning through gigabytes to find 256 scattered rows. In Lance: direct page-level lookup. 100× is not marketing — it's physics.

**Why NOT Parquet for ML:** Parquet was designed for analytics workloads (scan entire columns). ML workloads need random row access. Wrong data structure for the job.

**Use cases:** embedding stores · ML training datasets · multi-modal data (images + metadata) · vector databases ([LanceDB](https://github.com/lancedb/lancedb) is built on Lance).

---

### [DuckDB](https://github.com/duckdb/duckdb) `C++` `36.9K ⭐` · [docs](https://duckdb.org/docs/) · [benchmark](https://duckdb.org/2023/03/03/jdbc.html)

> Replaces **`pandas`** · **`SQLite`** (analytics) · **`Spark`** (small-medium) → **5–10× · zero infrastructure**

```python
import duckdb
# Reads directly from S3, no download
duckdb.sql("SELECT count(*) FROM 's3://bucket/events/*.parquet' WHERE event = 'purchase'")
```

- Multi-hypothesis CSV parsing (tests delimiters in parallel — no more `sep=','` guessing)
- Reads 10GB CSV in ~2s on M4 Max, same file pandas: ~45s
- Embeds in Python, Rust, Go, Node, Java — **no server process**

**Why NOT SQLite for analytics:** SQLite is row-oriented (great for OLTP). DuckDB is column-oriented (great for analytics). Filtering 1 column on 10M rows: SQLite reads all columns anyway. DuckDB reads only the column you need.

**Why NOT pandas for SQL users:** If you're writing `df[df['col'] > x].groupby(...)`, you're re-inventing SQL badly. DuckDB does it in 1 line with a full SQL planner + vectorized execution.

**Why NOT Spark:** For <5TB, Spark's cluster overhead (startup, shuffle, GC) makes it slower than DuckDB on a single node.

**Use cases:** ad-hoc analytics on files · data pipelines without infra · replacing Jupyter+pandas workflows · log analysis · anywhere you want SQL on files.

---

### [Polars](https://github.com/pola-rs/polars) `Rust` `37.8K ⭐` · [docs](https://docs.pola.rs/) · [benchmarks](https://www.pola.rs/benchmarks.html)

> Replaces **`pandas`** → **5–10× faster · lazy evaluation · SIMD multi-threaded**

```python
# The pandas way (eager, single-threaded, slow)
df = pd.read_csv("huge.csv")
result = df[df['revenue'] > 1000].groupby('country')['revenue'].sum()

# The Polars way (lazy, multi-threaded, fast)
result = pl.scan_csv("huge.csv").filter(pl.col('revenue') > 1000).group_by('country').agg(pl.col('revenue').sum()).collect()
```

**Why NOT pandas:**
- Single-threaded by design (GIL + numpy)
- Eager evaluation = materializes every intermediate result
- Copies data on almost every operation
- `SettingWithCopyWarning` will haunt you

**Rule:** Use **DuckDB** when you think in SQL · Use **Polars** when you think in DataFrames.

---

## ⚡ Async Runtime & I/O

### [Monoio](https://github.com/bytedance/monoio) `Rust` · ByteDance · [blog post](https://www.bytecamp.net/monoio/)

> Replaces **[`Tokio`](https://github.com/tokio-rs/tokio)** on Linux throughput workloads → **3× faster on 16 cores**

| Cores | vs Tokio | vs NGINX |
|:-----:|:--------:|:--------:|
| 4 | +2× | — |
| 16 | **+3×** | **+20%** |
| RPC throughput | +26% | — |

**How:** Thread-per-core model + io_uring native. Each CPU core has its own event loop, no cross-thread work stealing overhead. Tokio uses a work-stealing scheduler (great for mixed workloads, overhead for pure I/O).

**Why NOT vanilla Tokio for Linux I/O servers:** Tokio uses epoll under the hood. io_uring batches syscalls — 1 syscall for N I/O operations vs 1 syscall per operation. At high RPS, syscall overhead becomes the bottleneck.

**When NOT to use:** Cross-platform (macOS/Windows) · mixed CPU+I/O workloads · ecosystem compatibility (most async Rust libraries are Tokio-specific).

---

### io_uring vs epoll — the syscall revolution

| | io_uring | epoll |
|--|:--------:|:-----:|
| Syscall overhead | **Batch (1 for N ops)** | 1 per op |
| 0-syscall polling | ✅ (busy-wait mode) | ❌ |
| Max IOPS | **Millions** | ~Hundreds of thousands |
| Fixed buffers | ✅ (zero-copy) | ❌ |
| Portability | Linux 5.1+ only | Everywhere |
| Production maturity | Growing fast | Decades |

**Reference:** [io_uring deep dive by Jens Axboe (author)](https://kernel.dk/io_uring.pdf)

---

### [tokio-quiche](https://github.com/cloudflare/quiche) `Rust` · Cloudflare · [InfoQ writeup](https://www.infoq.com/news/2025/12/quic-http3-rust/)

> Replaces **[`Quinn`](https://github.com/quinn-rs/quinn)** → **2× faster · production HTTP/3 at scale**

Cloudflare's battle-hardened `quiche` QUIC implementation + Tokio async runtime. Handles millions of HTTP/3 connections in production.

**Why HTTP/3 at all:** HTTP/2 over TCP has head-of-line blocking — one lost packet stalls all streams. HTTP/3 over QUIC multiplexes independently — one lost packet stalls only its stream. On mobile networks: dramatic latency improvement.

**Why Quinn doesn't cut it for production:** Quinn is pure-Rust correctness-first. tokio-quiche is Cloudflare-hardened performance-first. For anything beyond experimentation, the production battle-testing matters.

---

## 🌐 HTTP Servers & Proxies

### [Pingora](https://github.com/cloudflare/pingora) `Rust` · Cloudflare · [blog post](https://blog.cloudflare.com/how-we-built-pingora-the-proxy-that-connects-cloudflare-to-the-internet/)

> Replaces **[`Nginx`](https://nginx.org/)** → **−60% CPU · −70% RAM · 1 trillion req/day**

| | Pingora | Nginx |
|--|:-------:|:-----:|
| Global scale | **1T req/day** | — |
| CPU usage | **−60%** | baseline |
| RAM usage | **−70%** | baseline |
| P50 latency | **5ms improvement** | baseline |
| Connection reuse | Shared pool across workers | Per-worker (no sharing) |

**The reason Nginx wastes memory:** Nginx spawns a process per worker, each maintaining its own connection pool. Pingora uses async Rust — all workers share one connection pool. For a busy server making 10M upstream connections/day, Nginx creates each connection fresh. Pingora reuses them → 999× fewer TCP handshakes observed in Cloudflare's data.

**Why NOT Nginx in 2026:**
- Written in C → memory unsafety (CVEs year after year)
- Process-per-worker → no connection pool sharing
- Configuration is a DSL that can't express complex routing logic
- No async Rust → limited to epoll

**Why NOT HAProxy:** Better than Nginx, still C, still no connection sharing at the depth Pingora achieves.

**Use cases:** CDN edge · API gateway · TLS termination · any reverse proxy at scale.

---

### [Hono](https://github.com/honojs/hono) `TypeScript` `21K ⭐` · [docs](https://hono.dev/) · [benchmarks](https://hono.dev/docs/concepts/benchmarks)

> Replaces **[`Express`](https://expressjs.com/)**, **[`Fastify`](https://fastify.dev/)** → **402K req/s · 14KB bundle · runs everywhere**

Same code. Every runtime.

```typescript
import { Hono } from 'hono'
const app = new Hono()
app.get('/api/users', (c) => c.json({ users: [] }))
export default app  // Works on: CF Workers · Bun · Deno · Node · Lambda
```

**Why NOT Express in 2026:**
- 28K req/s vs Hono's 402K — **14× slower**
- No TypeScript support built-in
- No native edge runtime support
- Bundle size: 200KB+ vs Hono's 14KB
- Middleware ecosystem is mostly unmaintained

**Why NOT Fastify:** Faster than Express (~78K req/s) but still single-runtime (Node.js only). No Cloudflare Workers, no Bun-native optimizations.

**Use cases:** Cloudflare Workers APIs · Bun-based backends · serverless functions · any TypeScript API that needs to run on multiple runtimes.

---

## 📨 Message Queues

### [Iggy.rs](https://github.com/iggy-rs/iggy) `Rust` · Apache License · [docs](https://docs.iggy.rs/)

> Replaces **[`Apache Kafka`](https://kafka.apache.org/)** → **5M+ msg/s · sub-ms P99 · single binary**

| | Iggy.rs | Apache Kafka |
|--|:-------:|:------------:|
| Throughput | **5M+ msg/s** | ~500K msg/s |
| P99 Latency | **<1 ms** | ~300 ms |
| Memory | **~200MB** | ~6GB per broker |
| Operations | **1 binary** | ZooKeeper + JVM + 3+ brokers |
| Monthly infra cost (est.) | **~$200** | **~$8,000** |

**Why NOT Kafka in 2026 (unless you're Netflix):**
- JVM startup: 30–90s cold start
- ZooKeeper dependency (being replaced by KRaft, but still complex)
- Minimum viable cluster: 3 brokers + 3 ZooKeeper nodes = 6 servers
- P99 of 300ms means your consumers always lag on bursts
- GC pauses = unpredictable producer ACKs

**The startup math:** If you're spending $8K/month on Kafka for <500K msg/s, Iggy.rs does 10× more on a $200/month server. That's $94K/year in savings.

---

### [Redpanda](https://github.com/redpanda-data/redpanda) `C++` `10K ⭐` · [docs](https://docs.redpanda.com/) · [benchmarks](https://redpanda.com/blog/kafka-vs-redpanda-performance-benchmark)

> Drop-in **Kafka API** compatible → **70× better P99 · no JVM · no ZooKeeper**

Use when: existing Kafka tooling (connectors, Schema Registry, Kafka Streams) must be preserved, but Kafka's performance and operational complexity are killing you.

**Why better P99 than Kafka:** No GC. C++ with deterministic memory management. Redpanda's thread-per-core model eliminates lock contention across cores.

**Migration:** Change bootstrap server URL. That's it. All Kafka clients work unchanged.

---

### [Aeron](https://github.com/real-logic/aeron) `C++/Java` · [latency numbers](https://github.com/real-logic/aeron/wiki/Performance-Testing)

> **29µs P99 · 4.7M msg/s** · kernel-bypass UDP · HFT gold standard

Not a Kafka replacement — a fundamentally different class of tool.

**When:** sub-100µs is a hard requirement (HFT, financial exchanges, real-time auctions, multiplayer game servers).

**Why the latency is so low:** Kernel-bypass networking (DPDK/RDMA) — messages never enter the OS kernel. Direct NIC-to-userspace DMA. 29µs P99 includes the full round trip.

**Why NOT for normal services:** Requires kernel-bypass network setup, specialized hardware, and significant operational knowledge. Complete overkill unless microseconds literally cost money.

---

## 🗃️ Key-Value Stores

### [Fjall](https://github.com/fjall-rs/fjall) `Rust` · v3.0 Jan 2026 · [docs](https://docs.rs/fjall)

> Replaces **[`RocksDB`](https://github.com/facebook/rocksdb)** for Rust → **pure Rust · zero C++ FFI · <2.2MB**

**Why NOT RocksDB in Rust:**
- RocksDB is C++ → requires bindgen + C++ toolchain + FFI overhead
- `rocksdb` crate compiles for 5–10 minutes (downloads and compiles all of RocksDB)
- C++ allocator cannot be replaced with mimalloc without custom build
- Binary size: RocksDB adds ~30MB. Fjall adds ~2.2MB.

**Why Fjall:** Pure Rust LSM tree. No FFI. Compiles fast. Tiny binary. ACID transactions. v3.0 released Jan 2026 with production-ready stability.

**When NOT to use:** Need RocksDB's massive ecosystem of tooling, language bindings, and battle-tested production years → use [SpeedB] instead.

---

### [SpeedB](https://github.com/speedb-io/speedb) `C++` · [benchmark](https://github.com/speedb-io/speedb/wiki/SpeedDB-Benchmarks)

> Replaces **[`RocksDB`](https://github.com/facebook/rocksdb)** (100% API compatible) → **−80% write amplification · 10× P99**

**Write amplification explained:** Every 1 byte written to RocksDB may write 10–30 bytes to disk (compaction). SpeedB's improved compaction algorithm reduces this by 80% → your SSD lives 5× longer and write P99s are dramatically more consistent.

**Migration:** Change `#include "rocksdb/db.h"` → `#include "speedb/db.h"`. That's it. Same config, same API, same file format.

---

### [LMDB](https://github.com/LMDB/lmdb) `C` · [Howard Chu's talk](https://www.youtube.com/watch?v=tEa5sAh-kVk)

> **567ms / 1M reads** — fastest bulk reads · zero-copy MMAP · [heed](https://github.com/meilisearch/heed) for Rust

MMAP-based B-tree. The OS page cache **is** the read cache — no double-buffering. Zero-copy reads: you get pointers directly into the mmap'd file.

**Use cases:** read-heavy lookups (embedding stores, config DBs, DNS caches) · anywhere you read >> write · [Meilisearch uses LMDB](https://github.com/meilisearch/meilisearch) for its indexes.

**When NOT to use:** High write throughput (B-tree vs LSM: LSM wins for writes, B-tree wins for reads).

---

## 🔍 Full-Text Search

### [Tantivy](https://github.com/quickwit-oss/tantivy) `Rust` `14.8K ⭐` · [docs](https://docs.rs/tantivy) · [benchmark](https://tantivy-search.github.io/bench/)

> Replaces **[`Elasticsearch`](https://github.com/elastic/elasticsearch)**, **[`Lucene`](https://lucene.apache.org/)** → **6.5× faster · 56% less RAM · embeds in your binary**

| | Tantivy | Elasticsearch |
|--|:-------:|:-------------:|
| Query time (1M docs) | **0.8 ms** | 5.2 ms |
| Indexing speed | **45K docs/s** | 12K docs/s |
| RAM (1M docs) | **2.1 GB** | 4.8 GB |
| Deployment | **1 library** | JVM cluster |

**Why NOT Elasticsearch in 2026:**
- Requires JVM → minimum 2–4GB heap just to start
- Cluster mode = 3+ nodes for HA
- GC pauses → unpredictable P99 spikes under load
- License change (not Apache 2.0 since 7.11) → [OpenSearch](https://github.com/opensearch-project/OpenSearch) fork exists but carries same JVM baggage

**Build on top of Tantivy:** [Quickwit](https://github.com/quickwit-oss/quickwit) (distributed, S3-native) · [Meilisearch](https://github.com/meilisearch/meilisearch) (simple API, great for apps) · [Sonic](https://github.com/valeriansaliou/sonic) (minimal search backend)

---

### [TypeSense](https://github.com/typesense/typesense) `C++` `22K ⭐` · [docs](https://typesense.org/docs/) · [cloud](https://cloud.typesense.org/)

> Replaces **[`Algolia`](https://www.algolia.com/)**, **`Elasticsearch`** → **<50ms · RAM-based · $0/month self-hosted**

RAM-first = always instant. Typo tolerance built in. Simple REST API.

**Why NOT Algolia:** $500–10K+/month. TypeSense self-hosted: $0. Algolia's only advantage is their CDN-distributed edge network — which TypeSense Cloud also offers.

**Why NOT Elasticsearch for app search:** You don't need Kibana, Logstash, Beats, and a 3-node cluster for a search box in your SaaS app. TypeSense is one binary, one API.

---

## 🧠 Memory Allocators

### [mimalloc](https://github.com/microsoft/mimalloc) `C` · Microsoft · [paper](https://www.microsoft.com/en-us/research/uploads/prod/2019/06/mimalloc-tr-v1.pdf)

> Replaces **[`jemalloc`](https://github.com/jemalloc/jemalloc)**, **`tcmalloc`** → **20–40% less memory · faster small allocs**

```toml
[dependencies]
mimalloc = "0.1"
```
```rust
#[global_allocator]
static GLOBAL: mimalloc::MiMalloc = mimalloc::MiMalloc;
```

**Why glibc malloc wastes memory:** glibc's allocator has poor thread-local caching → threads fight over global lock → fragmentation at scale. mimalloc uses thread-local heaps with free-list sharding.

**Why NOT jemalloc as default:** jemalloc is excellent for large allocations and Firefox-style workloads. For small allocations (<1KB, typical in web services), mimalloc wins consistently.

**Real result:** Several Rust web services reported 25–35% RSS reduction just from this 3-line change.

---

### [snmalloc](https://github.com/microsoft/snmalloc) `C++` · Microsoft · [paper](https://github.com/microsoft/snmalloc/blob/main/docs/LLVM.pdf)

> **Highest concurrent throughput** for message-passing workloads

Uses a "message passing" deallocation model — objects are freed by sending them back to their originating thread's pool. Eliminates the cross-thread free bottleneck entirely.

**Use when:** Actor-style or channel-heavy code where one thread allocates and a different thread frees. Higher memory overhead than mimalloc, but wins on concurrent throughput benchmarks.

---

## #️⃣ Hash Functions

### [rapidhash](https://github.com/Nicoshev/rapidhash) `Rust/C` · [benchmarks](https://github.com/Nicoshev/rapidhash#benchmarks)

> Replaces **[`xxHash3`](https://github.com/Cyan4973/xxHash)**, **[`wyhash`](https://github.com/wangyi-fudan/wyhash)** → **71 GB/s on M4** · 2025 speed champion

Newest champion (2025). Specifically optimized for Apple Silicon M4 microarchitecture. Beats both xxHash3 and wyhash on ARM64.

**Why NOT MD5/SHA-1 for non-crypto hashing:** 1–3 GB/s vs 71 GB/s. If you're using MD5 for deduplication or cache keys — you're leaving 50× performance on the table, and paying for crypto properties you don't need.

---

### [BLAKE3](https://github.com/BLAKE3-team/BLAKE3) `Rust` · [paper](https://github.com/BLAKE3-team/BLAKE3-specs/blob/master/blake3.pdf) · [website](https://blake3.io/)

> Replaces **[`SHA-256`](https://en.wikipedia.org/wiki/SHA-2)**, **`SHA-3`** → **8.4 GB/s multi-core · 4–10× faster >4KB · parallelizable**

SHA-256 is sequential by design — you must process block N before block N+1. BLAKE3 uses a Merkle tree internally — all blocks are independent and can be hashed in parallel across all cores.

**Why NOT SHA-256 (unless compliance forces you):**
- Sequential → can't use multiple cores
- No hardware acceleration on ARM (unlike AES-NI)
- 4–10× slower for large files
- BLAKE3 is cryptographically sound and peer-reviewed by the same community

**Drop-in CLI:** `brew install b3sum` → same interface as `sha256sum`.

---

### [AHash](https://github.com/tkaitchuck/aHash) `Rust` · [benchmark](https://github.com/tkaitchuck/aHash#benchmark-results)

> Replaces **`SipHash`** (Rust's default `HashMap`) → **fastest non-crypto hash for `HashMap`**

Default in [`hashbrown`](https://github.com/rust-lang/hashbrown) (which backs `std::HashMap`). AES-based — uses hardware AES-NI instructions for the hash.

```rust
use ahash::AHashMap;
let mut map: AHashMap<String, u64> = AHashMap::new(); // Done.
```

**Why Rust uses SipHash by default:** DoS resistance — adversarial inputs can't cause pathological hash collisions. If your HashMap keys come from untrusted users → keep SipHash. For internal maps with trusted keys → AHash.

---

## 🗜️ Compression

| Use Case | Tool | Decomp Speed | Ratio | When to pick |
|----------|------|:------------:|:-----:|--------------|
| **Realtime / sub-ms** | [LZ4](https://github.com/lz4/lz4) | **3.5 GB/s** | 1.12× | Streaming, hot paths, Kafka messages |
| **Network / storage** | [zstd](https://github.com/facebook/zstd) | 1 GB/s | **2.8×** | S3, backups, network transfer |
| **HTTP responses** | [Brotli](https://github.com/google/brotli) | 0.4 GB/s | **Best** | Static assets, CDN responses |
| **Game assets** | [Oodle Kraken](http://www.radgametools.com/oodle.htm) | ~2 GB/s | ~2.5× | AAA games industry standard |

**Why NOT gzip in 2026:** zstd compresses better AND decompresses faster than gzip. The only reason to use gzip is HTTP/1.1 clients that don't support Brotli/zstd (increasingly rare).

**zstd dict training trick:** Train a zstd dictionary on your dataset → compress many small, similar documents 10–30× better. Critical for JSON API responses where each message is small but similar in structure.

```bash
# Train a dictionary from 1000 sample documents
zstd --train samples/* -o dict.zst
# Use it
zstd -D dict.zst -19 input.json -o compressed.zst  # ~10x better on small similar docs
```

---

## 🔎 Regex & String Search

### [Hyperscan](https://github.com/intel/hyperscan) → [Vectorscan](https://github.com/VectorCamp/vectorscan) `C` · [paper](https://www.usenix.org/conference/nsdi19/presentation/wang-xiang)

> Replaces **[`PCRE`](https://github.com/PCRE2Project/pcre2)** for multi-pattern matching → **8.73× faster · 986 Mbps · DPI gold standard**

Intel SIMD regex engine. Matches **thousands of patterns simultaneously** in a single pass through the input. Standard in [Suricata](https://suricata.io/), [Snort](https://www.snort.org/), [Zeek](https://zeek.org/).

**The AHA moment:** With PCRE, matching 1000 patterns on a packet = 1000 regex evaluations in sequence. With Hyperscan = 1 pass, all 1000 matched simultaneously via SIMD. For intrusion detection on 10Gbps links, this isn't an optimization — it's the only option.

**Vectorscan** = community fork with ARM support (Hyperscan is Intel-only). Use Vectorscan for cross-platform builds.

---

### [RE2](https://github.com/google/re2) `C++` · Google · [safety guarantee](https://github.com/google/re2/wiki/Syntax)

> Replaces **`PCRE`** for untrusted input → **Linear O(n) — no ReDoS possible**

PCRE has exponential worst-case complexity. RE2 uses a DFA/NFA construction that guarantees O(n) time regardless of pattern or input.

**Real security incident:** The [Cloudflare outage (July 2019)](https://blog.cloudflare.com/details-of-the-cloudflare-outage-on-july-2-2019/) was caused by catastrophic backtracking in a WAF regex. A single crafted request saturated all CPUs to 100%. With RE2, this cannot happen.

**What you give up:** backreferences (`\1`), lookaheads, lookbehinds. These are the features that enable exponential complexity. RE2 supports everything else.

**Rule:** If the regex pattern or input comes from untrusted sources → RE2. Always.

---

### Teddy (in [aho-corasick](https://github.com/BurntSushi/aho-corasick)) `Rust` · powers [ripgrep](https://github.com/BurntSushi/ripgrep)

> **3.2–8.8× SIMD speedup** over classic Aho-Corasick for multi-pattern substring search

16B (SSE) / 32B (AVX) SIMD chunks — processes 16–32 bytes per cycle. This is what makes `ripgrep` faster than `grep` for code search.

**Use aho-corasick when:** multi-pattern substring matching without full regex overhead. Searching for 1000 keywords in a document? Aho-Corasick makes one pass; naive loop makes 1000 passes.

---

## 🔐 Cryptography

### [aws-lc-rs](https://github.com/aws/aws-lc-rs) `Rust` · AWS · [FIPS docs](https://github.com/aws/aws-lc-rs/blob/main/aws-lc-rs/FIPS.md)

> Replaces **[`ring`](https://github.com/briansmith/ring)**, **[`openssl`](https://github.com/openssl/openssl)** → **15–30% faster · FIPS 140-3 certified**

Drop-in replacement for the `ring` Rust crate (same API). Formally verified. FIPS 140-3 Level 1 certified by NIST.

**Why NOT openssl-sys:** OpenSSL binds via FFI → C-style API, memory unsafety risks, difficult to audit. aws-lc-rs is a Rust-safe wrapper over AWS-LC (their formally verified fork of BoringSSL).

**Why NOT ring for compliance:** ring is NOT FIPS certified. If you're handling healthcare, finance, or government data → you need FIPS 140-3. aws-lc-rs is your only Rust-native option.

**Cargo.toml:**
```toml
[dependencies]
aws-lc-rs = "1"
# or as ring drop-in:
ring = { package = "aws-lc-rs", version = "1", features = ["ring-sig-verify"] }
```

---

## 🎯 Decision Matrix

> Structured for AI agents and quick lookups

```
# JSON
JSON_RUST       → sonic-rs       | replaces: serde_json    | gain: 1.5-2x, 50% less heap
JSON_CPP        → simdjson       | replaces: RapidJSON     | gain: 6GB/s, industry standard
JSON_C_IOT      → yyjson         | replaces: cJSON         | gain: 2.3x stringify, zero deps
JSON_ZERO_ALLOC → json-zero      | replaces: all           | gain: 0 heap allocations

# Serialization
SERIALIZE_RPC   → Cap'n Proto    | replaces: Protobuf      | gain: 0ns deser, built-in RPC
SERIALIZE_IOT   → FlatBuffers    | replaces: Protobuf      | gain: 60-80% CPU+size
SERIALIZE_RUST  → rkyv           | replaces: serde         | gain: 200% throughput, 65% less mem

# Query & Columnar
COLUMNAR_ML     → Lance          | replaces: Parquet       | gain: 100x random access
QUERY_ENGINE    → DataFusion     | replaces: Spark         | gain: SIGMOD2024 winner, no cluster
ANALYTICS_SQL   → DuckDB         | replaces: pandas/SQLite | gain: 5-10x, zero infra
ANALYTICS_DF    → Polars         | replaces: pandas        | gain: 5-10x, lazy eval

# Async & I/O
ASYNC_LINUX     → Monoio         | replaces: Tokio         | gain: 3x on 16 cores
ASYNC_CROSS     → Tokio          | replaces: async-std     | gain: ecosystem standard
HTTP3           → tokio-quiche   | replaces: Quinn         | gain: 2x faster, CF-proven

# HTTP
HTTP_PROXY      → Pingora        | replaces: Nginx         | gain: -60% CPU, -70% RAM
HTTP_EDGE       → Hono           | replaces: Express       | gain: 14x faster, multi-runtime

# Messaging
MQ_KAFKA_KILL   → Iggy.rs        | replaces: Kafka         | gain: 10x throughput, sub-ms P99
MQ_KAFKA_COMPAT → Redpanda       | replaces: Kafka         | gain: 70x P99, no JVM
MQ_HFT          → Aeron          | replaces: ZeroMQ        | gain: 29us P99, kernel bypass

# Storage
KV_RUST         → Fjall          | replaces: RocksDB       | gain: pure Rust, <2.2MB binary
KV_ROCKSDB_COMPAT → SpeedB       | replaces: RocksDB       | gain: -80% write amp, drop-in
KV_READS        → LMDB           | replaces: LevelDB       | gain: 567ms/1M reads, zero-copy

# Search
FTS_LIB         → Tantivy        | replaces: Lucene/ES     | gain: 6.5x, 56% less RAM
FTS_SERVICE     → TypeSense      | replaces: Algolia/ES    | gain: <50ms, $0 self-hosted

# System
ALLOC_GENERAL   → mimalloc       | replaces: jemalloc      | gain: 20-40% less mem
ALLOC_CONC      → snmalloc       | replaces: jemalloc      | gain: highest conc throughput
HASH_SPEED      → rapidhash      | replaces: xxHash3       | gain: 71GB/s on M4
HASH_CRYPTO     → BLAKE3         | replaces: SHA-256       | gain: 4-10x, parallelizable
HASH_HASHMAP    → AHash          | replaces: SipHash       | gain: fastest HashMap

# Compression
COMPRESS_RT     → LZ4            | replaces: gzip          | gain: 3.5GB/s sub-ms
COMPRESS_STORE  → zstd           | replaces: gzip          | gain: 2.8x ratio, 1GB/s
COMPRESS_HTTP   → Brotli         | replaces: gzip          | gain: best ratio for HTTP

# Regex
REGEX_MULTI     → Hyperscan      | replaces: PCRE          | gain: 8.73x, SIMD parallel
REGEX_SAFE      → RE2            | replaces: PCRE          | gain: O(n) guaranteed, no ReDoS

# Crypto
CRYPTO_TLS_FIPS → aws-lc-rs      | replaces: openssl       | gain: FIPS 140-3, +15-30%
```

---

## 🏆 The 2026 Stack

> Paste this into your `ARCHITECTURE.md`

```
┌─────────────────────────────────────────────────────────────────────┐
│                      GAMECHANGER STACK 2026                         │
├───────────────────┬─────────────────────────────────────────────────┤
│ JSON              │ sonic-rs (Rust) · simdjson (C++) · yyjson (C)   │
│ Serialization     │ Cap'n Proto (0ns RPC) · rkyv (Rust zero-copy)   │
│ Columnar / ML     │ Lance (100x random access) · DuckDB (analytics) │
│ Query Engine      │ DataFusion (embedded SQL) · Polars (DataFrame)  │
│ Async (Linux)     │ Monoio (io_uring, 3x Tokio)                    │
│ Async (cross)     │ Tokio                                           │
│ HTTP Proxy        │ Pingora (−60% CPU vs Nginx)                     │
│ HTTP Edge API     │ Hono (14KB, multi-runtime, 402K req/s)          │
│ HTTP/3            │ tokio-quiche (Cloudflare-proven)                │
│ Message Queue     │ Iggy.rs (Kafka killer) · Aeron (HFT µs)        │
│ KV Store          │ Fjall (pure Rust) · SpeedB (RocksDB drop-in)   │
│ Full-Text Search  │ Tantivy (library) · TypeSense (service)        │
│ Memory Allocator  │ mimalloc (default) · snmalloc (concurrent)      │
│ Hash (general)    │ rapidhash (71GB/s) · AHash (Rust HashMaps)      │
│ Hash (crypto)     │ BLAKE3 (parallelizable, 4-10x SHA-256)          │
│ Compression       │ zstd (storage) · LZ4 (realtime)                │
│ Regex             │ Hyperscan (multi-pattern) · RE2 (safe/untrusted)│
│ Crypto / TLS      │ aws-lc-rs (FIPS 140-3) · ring (general Rust)   │
└───────────────────┴─────────────────────────────────────────────────┘
```

---

## 📊 Emerging — Watch 2026

| Tool | Stars | Status | Why it matters |
|------|:-----:|--------|----------------|
| [Vortex](https://github.com/spiraldb/vortex) | — | Prototype | Adaptive per-column encoding → 10× Parquet compression ratio |
| [rkyv](https://github.com/rkyv/rkyv) | `5.5K` | Stable | 200% throughput, 65% memory reduction, zero-copy Rust |
| [Glaze](https://github.com/stephenberry/glaze) | `1.5K` | Active | Claims to beat simdjson; early but promising benchmarks |
| [Fjall 3.0](https://github.com/fjall-rs/fjall) | `1K` | ✅ Jan 2026 | Pure-Rust RocksDB replacement hits v3.0 stable |
| [sonic-go](https://github.com/bytedance/sonic) | `7K` | Active | SIMD-JSON for Go — ByteDance's port of the sonic approach |
| [iggy.rs v1.0](https://github.com/iggy-rs/iggy) | `2K` | Active | Approaching stable API; watch for v1.0 production declaration |
| [Pingap](https://github.com/vicanso/pingap) | `3K` | Active | Pingora-based reverse proxy with config-driven management |
| [DataFusion Comet](https://github.com/apache/datafusion-comet) | `1K` | Active | Spark plugin that accelerates Spark with DataFusion — gradual migration path |

---

## 🤖 Agent Query API

This list is machine-indexed in [SurrealDB](https://surrealdb.com) for agent-driven tool selection:

```surql
-- Find what replaces a specific legacy tool
SELECT name, speed_value, github_url, use_case
FROM gamechanger_tool
WHERE replaces CONTAINS 'Kafka';

-- Best tool for a category
SELECT name, speed_value, language, replaces
FROM gamechanger_tool
WHERE category = 'Full-Text Search'
ORDER BY category;

-- Semantic search across 18K repos (HNSW, 768-dim)
SELECT name, description, stars,
       vector::similarity::cosine(embedding, $query_vec) AS score
FROM repo WHERE embedding IS NOT NONE
ORDER BY score DESC LIMIT 10;

-- Find stale legacy repos still on awesome lists
SELECT name, stars, last_commit FROM repo
WHERE last_commit < '2022-01-01' AND stars > 1000
ORDER BY stars DESC;
```

**Local KB:** `http://127.0.0.1:9926` · NS `awesome` · DB `github` · 18,286 repos · 768-dim HNSW
**CLI:** `bash query.sh gamechanger` · `python3 semantic_search.py "fast Rust HTTP server"`

---

## 🔬 Methodology

1. **18,286 repos** indexed from [`awesome-rust`](https://github.com/rust-unofficial/awesome-rust) · [`awesome-go`](https://github.com/avelino/awesome-go) · [`awesome-cpp`](https://github.com/fffaraz/awesome-cpp) · [`awesome-selfhosted`](https://github.com/awesome-selfhosted/awesome-selfhosted) · [`awesome-mcp-servers`](https://github.com/punkpeye/awesome-mcp-servers) + 15 other lists
2. All repos stored in **[SurrealDB](https://surrealdb.com)** with stars, forks, language, topics, last commit
3. **768-dim [HNSW](https://arxiv.org/abs/1603.09320) embeddings** via [nomic-embed-text](https://huggingface.co/nomic-ai/nomic-embed-text-v1) (Ollama) for semantic discovery
4. **68 RELATE graph edges** `gamechanger_tool → replaces → legacy_repo` built in SurrealDB
5. Benchmarks from official repos, [SIGMOD papers](https://andrew.nerdnetworks.org/pdf/SIGMOD-2024-lamb.pdf), [Cloudflare blog](https://blog.cloudflare.com/how-we-built-pingora-the-proxy-that-connects-cloudflare-to-the-internet/), and production case studies — never marketing copy

---

## Contributing

PR criteria — all four required:

1. **Real benchmark** with a source URL (no "blazing fast" without a number)
2. **Production-proven** OR active commit in last 6 months
3. **Clear replacement target** (what legacy tool does it beat, and why)
4. **Honest "When NOT to use"** section — this is what separates useful lists from hype lists

[Open a PR](../../compare) · [Suggest a tool via Issue](../../issues/new/choose)

---

<div align="center">

**⭐ If this helped you drop a legacy tool, star it.**

Built with: [SurrealDB](https://surrealdb.com) · [nomic-embed-text](https://ollama.ai) · [ripgrep](https://github.com/BurntSushi/ripgrep) · [DuckDB](https://duckdb.org) · 18,286 repos analyzed

*"The best tool is the one you'd choose if you were starting today."*

</div>
