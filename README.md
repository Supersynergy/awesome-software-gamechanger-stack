# ⚡ Gamechanger Stack 2026

> **Drop legacy tools. Benchmarks, not vibes.**
> Curated from 18,000+ GitHub repos · 11 research agents · March 2026

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![Tools](https://img.shields.io/badge/tools-31%20categories-blue)](#-decision-matrix)
[![Agent Ready](https://img.shields.io/badge/agent--optimized-✓-brightgreen)](#-agent-query-api)
[![Last Updated](https://img.shields.io/badge/updated-April%202026-orange)](.)

---

## ⚡ TL;DR — Top 10 Drops

| Drop | Use instead | Speed gain |
|------|-------------|------------|
| `serde_json` | [sonic-rs] | 1.5–2× |
| `Nginx` | [Pingora] | −60% CPU, −70% RAM |
| `Kafka` | [Iggy.rs] | 3–5× throughput, sub-ms P99 |
| `Elasticsearch` | [Tantivy] | 6.5× faster, 56% less RAM |
| `pandas` | [Polars] / [DuckDB] | 5–10× |
| `Parquet` (ML) | [Lance] | 100× random access |
| `Spark` | [DataFusion] | SIGMOD 2024 winner, embeddable |
| `RocksDB` (Rust) | [Fjall] | pure Rust, <2.2MB binary |
| `SHA-256` | [BLAKE3] | 4–10× faster >4KB |
| `jemalloc` | [mimalloc] | 20–40% less memory |

---

## 📑 Contents

| Category | Tools | Quick pick |
|----------|-------|-----------|
| [JSON Parsing](#-json-parsing) | sonic-rs · simdjson · yyjson | sonic-rs (Rust), simdjson (C++) |
| [Serialization](#-serialization) | Cap'n Proto · FlatBuffers · rkyv | Cap'n Proto for RPC, rkyv for Rust |
| [Query & Columnar](#-query-engines--columnar) | DataFusion · Lance · DuckDB · Polars | DuckDB for analytics, Lance for ML |
| [Async & I/O](#-async-runtime--io) | Monoio · tokio-quiche | Monoio (Linux), Tokio (cross-platform) |
| [HTTP](#-http-servers--proxies) | Pingora · Hono | Pingora (proxy), Hono (edge API) |
| [Message Queues](#-message-queues) | Iggy.rs · Redpanda · Aeron | Iggy.rs (Kafka killer) |
| [KV Stores](#-key-value-stores) | Fjall · SpeedB · LMDB | Fjall (Rust), SpeedB (RocksDB compat) |
| [Full-Text Search](#-full-text-search) | Tantivy · TypeSense | Tantivy (lib), TypeSense (service) |
| [Memory Allocators](#-memory-allocators) | mimalloc · snmalloc | mimalloc (default choice) |
| [Hash Functions](#-hash-functions) | rapidhash · BLAKE3 · AHash | rapidhash (speed), BLAKE3 (crypto) |
| [Compression](#-compression) | LZ4 · zstd · Brotli | LZ4 (realtime), zstd (storage) |
| [Regex](#-regex--string-search) | Hyperscan · RE2 · Teddy | Hyperscan (multi-pattern), RE2 (safe) |
| [Cryptography](#-cryptography) | aws-lc-rs · BLAKE3 | aws-lc-rs (FIPS), BLAKE3 (hash) |

---

## 🔥 JSON Parsing

### [sonic-rs](https://github.com/cloudwego/sonic-rs) `Rust` `2.5K ⭐`

> Replaces **`serde_json`** → **1.5–2× faster**

```toml
[dependencies]
sonic-rs = "0.3"
```

| Benchmark | sonic-rs | simd-json | serde_json |
|-----------|:--------:|:---------:|:----------:|
| Twitter parse | **796 µs** | 1,061 µs | ~3,000 µs |
| Canada parse | **3.9 ms** | 5.2 ms | — |

**Why faster:** Parses directly into your struct — no "tape" intermediate representation, no intermediate copy. Runs SIMD on both x86-64 and aarch64. ByteDance battle-tested.

**When NOT to use:** Non-SIMD targets (graceful fallback, but no speed gain).

---

### [simdjson](https://github.com/simdjson/simdjson) `C++` `18K ⭐`

> Replaces **`RapidJSON`**, **`nlohmann/json`** → **6 GB/s minification**

- `v4.3 (Mar 2025)` — +30% from LoongArch LSX/LASX SIMD
- Used by: Node.js · ClickHouse · Apache Doris · Milvus · Shopify · Meta Velox · Intel
- Two APIs: **DOM** (full parse) + **On Demand** (reads only what you access)

**When NOT to use:** Need zero C++ toolchain dependency → use [yyjson] instead.

---

### [yyjson](https://github.com/ibireme/yyjson) `C` `4.2K ⭐`

> Replaces **`cJSON`**, **`jansson`** → **2.3× faster stringify than simdjson**

| | yyjson | simdjson |
|--|:------:|:--------:|
| Twitter parse | 1.72 GB/s | 1.52 GB/s |
| **Stringify** | **1.42 GB/s** | 0.61 GB/s |

Zero dependencies. No C++ compiler needed. Embedded/IoT-ready. The stringify win (2.3×) is decisive for write-heavy workloads.

> 💡 **Agent note:** 58% of heap allocations in production Rust APIs come from JSON deserialization ([Mozilla/Cloudflare 2024](https://hacks.mozilla.org)). For zero-alloc JSON → [json-zero](https://github.com/ingon/json-zero) or [Lite³](https://github.com/fastserial/lite3).

---

## 📦 Serialization

### [Cap'n Proto](https://capnproto.org) `Multi`

> Replaces **`Protobuf`** → **0 ns deserialization**

Wire format = memory format. Reading a Cap'n Proto message is reading directly from the buffer — no deserialization step exists.

| Framework | Zero-Copy | Deser Speed | Built-in RPC | Size |
|-----------|:---------:|:-----------:|:------------:|:----:|
| **Cap'n Proto** | ✅ | **Fastest** | ✅ | Medium |
| FlatBuffers | ✅ | Fast | ❌ | Large |
| Protobuf | ❌ | Medium | ❌ | **Smallest** |
| MessagePack | ❌ | Medium | ❌ | Small |

**When to use:** Microsecond-latency RPC, inter-process communication, anything latency-critical.

---

### [FlatBuffers](https://flatbuffers.dev) `Multi` · Google

> Replaces **`Protobuf`** for IoT → **60–80% CPU + size reduction**

Per-field offset table = lazy loading. Only reads fields you access. Proven 60–80% savings on IoT hardware. Use when: embedded/IoT, mobile, or when you need selective field access.

---

### [rkyv](https://github.com/rkyv/rkyv) `Rust` `5.5K ⭐`

> Replaces **`serde`** for internal serialization → **200% throughput · 65% less memory**

Zero-copy deserialization for Rust. Serialize once, access forever without any copying. See [RustConf 2024](https://2024.rustconf.com/programs/) for the deep-dive.

---

## 🗄️ Query Engines & Columnar

### [Apache Arrow DataFusion](https://github.com/apache/datafusion) `Rust` `8.5K ⭐`

> Replaces **`Apache Spark`** → **SIGMOD 2024 fastest Parquet parser · 6 GB/s IPC**

Embeds directly into your service — no cluster, no JVM, no ZooKeeper. Zero-copy across language boundaries via Arrow IPC (6 GB/s). Vectorized execution engine.

**Paper:** [SIGMOD 2024 — fastest Parquet parser of all measured systems](https://andrew.nerdnetworks.org/pdf/SIGMOD-2024-lamb.pdf)

---

### [Lance](https://github.com/lancedb/lance) `Rust`

> Replaces **`Parquet`** for ML/AI → **100× faster random access**

Not a typo. Parquet is column-sequential; Lance is random-access optimized. Built for AI workloads: embedding vectors, multi-modal data, ML training datasets.

**Paper:** [arxiv.org/abs/2504.15247](https://arxiv.org/html/2504.15247v1)

---

### [DuckDB](https://github.com/duckdb/duckdb) `C++` `36.9K ⭐`

> Replaces **`pandas`**, **`SQLite`** (analytics), **`Spark`** (small-medium data) → **5–10×**

- Multi-hypothesis CSV parsing (tests multiple delimiters in parallel)
- Vectorized JSON parsing at wire speed
- Embeds in Python, Rust, Go, Node — no server process
- Reads 10GB CSV in ~2s on M4 Max

```python
import duckdb
duckdb.sql("SELECT * FROM 'data/*.parquet' WHERE stars > 1000")
```

---

### [Polars](https://github.com/pola-rs/polars) `Rust` `37.8K ⭐`

> Replaces **`pandas`** → **5–10× faster · lazy evaluation**

SIMD multi-threaded parsing + lazy query planning. Evaluates minimum required computation.

**Rule:** DuckDB when you need SQL · Polars when you need a DataFrame API.

---

## ⚡ Async Runtime & I/O

### [Monoio](https://github.com/bytedance/monoio) `Rust` · ByteDance

> Replaces **`Tokio`** on Linux throughput workloads → **3× faster on 16 cores**

| Cores | vs Tokio | vs NGINX |
|:-----:|:--------:|:--------:|
| 4 | +2× | — |
| 16 | **+3×** | **+20%** |
| RPC | +26% | — |

Thread-per-core + io_uring native. **Trade-off:** Linux 5.1+ only. Use Tokio for cross-platform.

---

### io_uring vs epoll

| | io_uring | epoll |
|--|:--------:|:-----:|
| Syscall overhead | **Batch (1 for N ops)** | 1 per op |
| 0-syscall polling | ✅ | ❌ |
| Max IOPS | **Millions** | Thousands |
| Portability | Linux 5.1+ | Everywhere |

---

### [tokio-quiche](https://github.com/cloudflare/quiche) `Rust` · Cloudflare

> Replaces **`Quinn`** → **2× faster · production HTTP/3**

Cloudflare's `quiche` (sans-io QUIC) + Tokio async. Handles millions of HTTP/3 req/s in Cloudflare production. [InfoQ writeup (Dec 2025)](https://www.infoq.com/news/2025/12/quic-http3-rust/).

---

## 🌐 HTTP Servers & Proxies

### [Pingora](https://github.com/cloudflare/pingora) `Rust` · Cloudflare

> Replaces **`Nginx`** → **−60% CPU · −70% RAM · 1T req/day**

| | Pingora | Nginx |
|--|:-------:|:-----:|
| Scale | **1T req/day** global | — |
| CPU savings | **−60%** | baseline |
| RAM savings | **−70%** | baseline |
| P50 latency | **5ms better** | baseline |

Cloudflare rewrote their entire proxy layer in Rust. Every Cloudflare customer is behind Pingora right now.

---

### [Hono](https://github.com/honojs/hono) `TypeScript` `21K ⭐`

> Replaces **`Express`**, **`Fastify`** → **402K req/s · 14KB bundle**

Same code runs on: Cloudflare Workers · Bun · Deno · Node.js · AWS Lambda. 14KB total. Ultra-minimal, ultra-fast.

---

## 📨 Message Queues

### [Iggy.rs](https://github.com/iggy-rs/iggy) `Rust` · Apache License

> Replaces **`Apache Kafka`** → **5M+ msg/s · sub-ms P99 · zero JVM**

| | Iggy.rs | Apache Kafka |
|--|:-------:|:------------:|
| Throughput | **5M+ msg/s** | ~500K msg/s |
| P99 Latency | **<1 ms** | ~300 ms |
| Ops overhead | **Single binary** | ZooKeeper + JVM + brokers |

---

### [Redpanda](https://github.com/redpanda-data/redpanda) `C++` `10K ⭐`

> Drop-in **Kafka API** compatible → **70× better P99 · no JVM**

Use when you need Kafka compatibility (existing tooling, connectors) but not Kafka's operational complexity. C++ ground-up, no ZooKeeper.

---

### [Aeron](https://github.com/real-logic/aeron) `C++/Java`

> **29µs P99 · 4.7M msg/s** · kernel-bypass UDP

Not a Kafka replacement — a different class. Use for: HFT, financial exchanges, anything needing sub-100µs guaranteed latency. Industry standard in finance.

---

## 🗃️ Key-Value Stores

### [Fjall](https://github.com/fjall-rs/fjall) `Rust` · v3.0 Jan 2026

> Replaces **`RocksDB`** for Rust projects → **pure Rust · <2.2MB binary**

Zero C++ FFI. Zero unsafe (mostly). Tiny binary. Jan 2026 v3.0. The default choice for any new Rust service needing an embedded KV store.

---

### [SpeedB](https://github.com/speedb-io/speedb) `C++`

> Replaces **`RocksDB`** (100% API compatible) → **−80% write amplification**

Drop-in. Same config, same API. 80% reduction in write amplification = dramatically better SSD endurance + write latency. 10× P99 improvement.

---

### [LMDB](https://github.com/LMDB/lmdb) `C`

> **567ms / 1M reads** — fastest bulk reads · zero-copy MMAP

Use when: read-heavy (embeddings lookup, config DB, caches). MMAP-based = OS page cache does the work.

---

## 🔍 Full-Text Search

### [Tantivy](https://github.com/quickwit-oss/tantivy) `Rust` `14.8K ⭐`

> Replaces **`Elasticsearch`**, **`Lucene`** → **6.5× faster · 56% less RAM**

| | Tantivy | Elasticsearch |
|--|:-------:|:-------------:|
| Query (1M docs) | **0.8 ms** | 5.2 ms |
| Indexing | **45K docs/s** | 12K docs/s |
| RAM (1M docs) | **2.1 GB** | 4.8 GB |

Use as a Rust library, or via [Quickwit](https://github.com/quickwit-oss/quickwit) (distributed) or [Meilisearch](https://github.com/meilisearch/meilisearch) (simple API) which build on Tantivy.

---

### [TypeSense](https://github.com/typesense/typesense) `C++` `22K ⭐`

> Replaces **`Algolia`**, **`Elasticsearch`** → **<50ms · RAM-based · simple REST**

RAM-first = instant search. Typo tolerance built in. Simple REST API. Zero operational complexity. Algolia quality without Algolia pricing.

---

## 🧠 Memory Allocators

### [mimalloc](https://github.com/microsoft/mimalloc) `C` · Microsoft

> Replaces **`jemalloc`**, **`tcmalloc`** → **20–40% less memory overhead**

```toml
[dependencies]
mimalloc = "0.1"
```
```rust
#[global_allocator]
static GLOBAL: mimalloc::MiMalloc = mimalloc::MiMalloc;
```

Default choice. Outperforms jemalloc + tcmalloc for small allocations. Drop-in via Rust crate.

---

### [snmalloc](https://github.com/microsoft/snmalloc) `C++` · Microsoft

> **Highest concurrent throughput**

Use when: high-concurrency with heavy inter-thread message passing. Higher memory overhead than mimalloc, but wins on concurrent throughput benchmarks.

---

## #️⃣ Hash Functions

### [rapidhash](https://github.com/Nicoshev/rapidhash) `Rust/C`

> Replaces **`xxHash3`**, **`wyhash`** → **71 GB/s on M4** · 2025 champion

Newest speed champion (benchmarked 2025). Beats xxHash3 and wyhash on Apple Silicon M4.

---

### [BLAKE3](https://github.com/BLAKE3-team/BLAKE3) `Rust`

> Replaces **`SHA-256`**, **`SHA-3`** → **8.4 GB/s multi-core · 4–10× faster >4KB**

Parallelizable (SHA-256 is sequential). Use whenever you need a cryptographic hash without a compliance requirement forcing SHA-256.

---

### [AHash](https://github.com/tkaitchuck/aHash) `Rust`

> Replaces **`SipHash`** (Rust default) → **fastest for `HashMap`**

Default in `hashbrown` (which `std::HashMap` uses). For non-adversarial contexts: `use ahash::AHashMap` and you're done.

---

## 🗜️ Compression

| Use Case | Tool | Decomp Speed | Ratio | Notes |
|----------|------|:------------:|:-----:|-------|
| **Realtime / sub-ms** | [LZ4](https://github.com/lz4/lz4) | **3.5 GB/s** | 1.12× | Default for streaming |
| **Network / storage** | [zstd](https://github.com/facebook/zstd) | 1 GB/s | **2.8×** | Balanced default |
| **HTTP / web** | Brotli | 0.4 GB/s | **Best** | Slow compress, great ratio |
| **Game / proprietary** | Oodle Kraken | ~2 GB/s | ~2.5× | Industry standard in games |

**Rule:** `LZ4` when speed > ratio · `zstd` when ratio matters · `Brotli` for HTTP responses

---

## 🔎 Regex & String Search

### [Hyperscan](https://github.com/intel/hyperscan) → [Vectorscan](https://github.com/VectorCamp/vectorscan) `C`

> Replaces **`PCRE`** for multi-pattern → **8.73× faster · 986 Mbps multi-pattern**

Intel SIMD regex engine. Matches thousands of patterns simultaneously. Standard in Suricata, Snort, Zeek. Use Vectorscan for ARM/portable builds.

---

### [RE2](https://github.com/google/re2) `C++` · Google

> Replaces **`PCRE`** for untrusted input → **Linear O(n) guaranteed · no ReDoS**

PCRE can be catastrophically slow against adversarial inputs (ReDoS). RE2 guarantees linear time. **Always use RE2 when processing user-supplied regex patterns.**

---

### Teddy (in [aho-corasick](https://github.com/BurntSushi/aho-corasick)) `Rust`

> **3.2–8.8× SIMD speedup** · powers [ripgrep](https://github.com/BurntSushi/ripgrep)

16B (SSE) / 32B (AVX) SIMD chunks. Use aho-corasick for multi-pattern string search without full regex overhead.

---

## 🔐 Cryptography

### [aws-lc-rs](https://github.com/aws/aws-lc-rs) `Rust` · AWS

> Replaces **`ring`**, **`openssl`** → **15–30% faster · FIPS 140-3**

Drop-in for the `ring` Rust crate. Formally verified. FIPS 140-3 certified. If you need compliance: **only choice**. If you don't: still faster than ring.

---

## 🎯 Decision Matrix

> For AI agents and quick lookups — structured format

```
JSON_RUST      → sonic-rs      (replaces: serde_json,     gain: 1.5-2x)
JSON_CPP       → simdjson      (replaces: RapidJSON,       gain: 6GB/s)
JSON_C_IOT     → yyjson        (replaces: cJSON,           gain: 2.3x stringify)
COLUMNAR_ML    → Lance         (replaces: Parquet,         gain: 100x random access)
QUERY_ENGINE   → DataFusion    (replaces: Spark,           gain: SIGMOD2024 winner)
ANALYTICS      → DuckDB        (replaces: pandas/SQLite,   gain: 5-10x)
DATAFRAME      → Polars        (replaces: pandas,          gain: 5-10x)
SERIALIZE_RPC  → Cap'n Proto   (replaces: Protobuf,        gain: 0ns deser)
SERIALIZE_IOT  → FlatBuffers   (replaces: Protobuf,        gain: 60-80% CPU+size)
SERIALIZE_RUST → rkyv          (replaces: serde,           gain: 200% throughput)
ASYNC_LINUX    → Monoio        (replaces: Tokio,           gain: 3x on 16 cores)
ASYNC_CROSS    → Tokio         (replaces: async-std,       gain: ecosystem standard)
HTTP3          → tokio-quiche  (replaces: Quinn,           gain: 2x faster)
HTTP_PROXY     → Pingora       (replaces: Nginx,           gain: -60% CPU -70% RAM)
HTTP_EDGE      → Hono          (replaces: Express,         gain: 402K req/s 14KB)
MQ_KAFKA_KILL  → Iggy.rs       (replaces: Kafka,           gain: 3-5x 5M msg/s)
MQ_KAFKA_COMPAT→ Redpanda      (replaces: Kafka,           gain: 70x P99 no JVM)
MQ_HFT         → Aeron         (replaces: ZeroMQ,          gain: 29us P99)
KV_RUST        → Fjall         (replaces: RocksDB,         gain: pure Rust <2.2MB)
KV_ROCKSDB     → SpeedB        (replaces: RocksDB,         gain: -80% write amp)
KV_READS       → LMDB          (replaces: LevelDB,         gain: 567ms/1M reads)
FTS_LIB        → Tantivy       (replaces: Lucene,          gain: 6.5x 56% less RAM)
FTS_SERVICE    → TypeSense     (replaces: Algolia/ES,      gain: <50ms RAM-based)
ALLOC_GENERAL  → mimalloc      (replaces: jemalloc,        gain: 20-40% less mem)
ALLOC_CONC     → snmalloc      (replaces: jemalloc,        gain: highest conc tput)
HASH_SPEED     → rapidhash     (replaces: xxHash3,         gain: 71GB/s M4)
HASH_CRYPTO    → BLAKE3        (replaces: SHA-256,         gain: 4-10x >4KB)
HASH_HASHMAP   → AHash         (replaces: SipHash,         gain: fastest HashMap)
COMPRESS_RT    → LZ4           (replaces: gzip,            gain: 3.5GB/s sub-ms)
COMPRESS_STORE → zstd          (replaces: gzip,            gain: 2.8x ratio 1GB/s)
REGEX_MULTI    → Hyperscan     (replaces: PCRE,            gain: 8.73x SIMD)
REGEX_SAFE     → RE2           (replaces: PCRE,            gain: O(n) no ReDoS)
CRYPTO_TLS     → aws-lc-rs     (replaces: openssl,         gain: FIPS +15-30%)
```

---

## 🏆 The 2026 Stack

```
┌─────────────────────────────────────────────────────────────────┐
│                     GAMECHANGER STACK 2026                      │
├───────────────────┬─────────────────────────────────────────────┤
│ JSON              │ sonic-rs (Rust) · simdjson (C++) · yyjson   │
│ Serialization     │ Cap'n Proto (RPC/0ns) · rkyv (Rust intern.) │
│ Columnar / ML     │ Lance (AI/random) · DuckDB (analytics SQL)  │
│ Query Engine      │ DataFusion · Polars (DataFrame)             │
│ Async (Linux)     │ Monoio (io_uring, 3x Tokio)                │
│ Async (cross)     │ Tokio                                       │
│ HTTP Proxy        │ Pingora (-60% CPU vs Nginx)                 │
│ HTTP Edge API     │ Hono (14KB, multi-runtime)                  │
│ HTTP/3            │ tokio-quiche (Cloudflare-proven)            │
│ Message Queue     │ Iggy.rs (Kafka killer) · Aeron (HFT µs)    │
│ KV Store          │ Fjall (Rust) · SpeedB (RocksDB drop-in)    │
│ Full-Text Search  │ Tantivy (lib) · TypeSense (service)        │
│ Memory Allocator  │ mimalloc (default) · snmalloc (concurrent)  │
│ Hash (general)    │ rapidhash · AHash (Rust HashMaps)           │
│ Hash (crypto)     │ BLAKE3                                      │
│ Compression       │ zstd (storage) · LZ4 (realtime)            │
│ Regex             │ Hyperscan (multi-pattern) · RE2 (safe)      │
│ Crypto / TLS      │ aws-lc-rs (FIPS) · ring (general Rust)     │
└───────────────────┴─────────────────────────────────────────────┘
```

---

## 🤖 Agent Query API

This list is stored as structured records in SurrealDB for agent consumption:

```surql
-- Find replacement for a legacy tool
SELECT name, speed_value, github_url
FROM gamechanger_tool
WHERE replaces CONTAINS 'Kafka';

-- Find fastest tool in a category
SELECT name, speed_value, language
FROM gamechanger_tool
WHERE category = 'Message Queue'
ORDER BY speed_value;

-- Semantic search (HNSW, requires embeddings)
SELECT name, description, stars,
       vector::similarity::cosine(embedding, $query_vec) AS score
FROM repo WHERE embedding IS NOT NONE
ORDER BY score DESC LIMIT 10;
```

**Local KB:** `http://127.0.0.1:9926` · NS `awesome` · DB `github`
**CLI:** `bash /path/to/awesome-indexer/query.sh gamechanger`
**Semantic:** `python3 /path/to/awesome-indexer/semantic_search.py "your query"`

---

## 📊 Emerging — Watch 2026

| Tool | Status | Why |
|------|--------|-----|
| [Vortex](https://github.com/spiraldb/vortex) | Prototype | Adaptive encoding, 10× Parquet compression |
| [rkyv](https://github.com/rkyv/rkyv) | Stable | 200% throughput, 65% less memory |
| [Glaze](https://github.com/stephenberry/glaze) | Active | Claims to beat simdjson |
| [Fjall 3.0](https://github.com/fjall-rs/fjall) | ✅ Jan 2026 | Pure-Rust RocksDB replacement, stable |
| [sonic-go](https://github.com/bytedance/sonic) | Active | SIMD-JSON for Go |
| [iggy.rs v1.0](https://github.com/iggy-rs/iggy) | Active | Approaching stable API |

---

## 🔬 Methodology

Research process:
1. **18,286 repos** indexed from `awesome-rust` · `awesome-go` · `awesome-cpp` · `awesome-selfhosted` · `awesome-mcp-servers` + 15 other lists
2. All repos stored in **SurrealDB** (NS: `awesome` / DB: `github`) with stars, language, description, last commit
3. **768-dim HNSW embeddings** (nomic-embed-text) built over all repos for semantic discovery
4. **68 RELATE edges** in SurrealDB graph: `gamechanger_tool → replaces → legacy_repo`
5. Benchmarks sourced from official repos, SIGMOD papers, production case studies — not marketing

---

## Contributing

PR criteria:
1. **Real benchmark** with a source URL — no "blazing fast" claims without numbers
2. **Production-proven** OR actively maintained (commit in last 6 months)
3. **Clear target** — must replace a specific legacy tool
4. **Honest trade-offs** — must include "When NOT to use"

[Open a PR](../../compare) · [Suggest a tool](../../issues/new/choose)

---

<div align="center">

**⭐ Star this if you dropped a legacy tool because of it**

Powered by: [SurrealDB](https://surrealdb.com) · [nomic-embed-text](https://ollama.ai) · [awesome-rust](https://github.com/rust-unofficial/awesome-rust) · 18,286 repos analyzed

</div>
