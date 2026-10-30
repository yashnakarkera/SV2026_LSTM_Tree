# IronKV: A Verified Mini LSM-Tree Key-Value Store in Rust

**Team:** Ironlog
**Members:** _Name 1_, _Name 2_, _Name 3_
**Course:** Software Verification Systems, RPTU Kaiserslautern-Landau (Project-Based Learning)

---

## Overview

IronKV is a small embedded key-value store built on a log-structured merge (LSM) tree, written in Rust and **formally verified** for functional correctness, crash consistency, and durability. The goal is not to compete with RocksDB, but to show that the core ideas behind modern storage engines can be specified and proved correct, and to measure what that costs in performance.

## Motivation

Storage engines sit under almost every database, cache, and distributed system. A single bug (a lost write after a crash, a deleted key that reappears after compaction, a torn log record accepted on recovery) can silently corrupt user data. These bugs are hard to find by testing because they appear only under rare interleavings and crash points. Formal verification can rule them out for all executions, not just the ones we tested.

## Features (planned)

- `put(key, value)`, `get(key)`, `delete(key)`, `scan(start, end)`
- Write-ahead log (WAL) with checksummed records
- In-memory memtable, flushed to immutable sorted tables (SSTables)
- Tombstones for deletes
- Simple compaction (merging SSTables)
- Crash recovery from the WAL

## Verified properties

| # | Property | Informal statement |
|---|----------|--------------------|
| P1 | Functional correctness | Any sequence of operations behaves like a plain map (`HashMap` model): `get` returns exactly what the model returns. |
| P2 | Compaction correctness | Compaction preserves logical contents: no key is lost, no deleted key reappears, newest version wins. |
| P3 | Durability | Every acknowledged write survives a crash. |
| P4 | Crash consistency | After a crash at any point, recovery yields a state equal to some prefix of the acknowledged operations. Torn or partial WAL records are never exposed. |
| P5 | Memory safety | No out-of-bounds access or panics on malformed on-disk data (checked on the parsing and decoding code). |

Performance is evaluated empirically (see Benchmarks), not proved.

## Approach

1. **Specify:** a simple abstract map model plus a disk model that can crash between steps.
2. **Model:** a TLA+ model of the WAL and recovery protocol, checked with TLC.
3. **Implement:** the store in Rust, starting without compaction and single-level, then extending.
4. **Verify:** proofs of refinement between implementation and spec using Verus (SMT-based verification for Rust). Kani is used for bounded checks of low-level code where it is more convenient.
5. **Evaluate:** benchmarks, fuzzing, and fault injection compared against what the proofs cover.

## Tools

- Rust
- Verus (deductive verification of Rust)
- Kani (bounded model checking, optional)
- TLA+ / TLC (protocol-level model)
- Criterion (benchmarks), cargo-fuzz (fuzzing)

## Repository structure (planned)

```
ironkv/
├── spec/            # TLA+ models and abstract specification
├── src/
│   ├── memtable.rs
│   ├── wal.rs
│   ├── sstable.rs
│   ├── compaction.rs
│   └── lib.rs
├── proofs/          # Verus proofs and lemmas
├── tests/           # unit, property, and crash-injection tests
├── benches/         # performance benchmarks
├── docs/            # weekly notes, reports, slides
└── README.md
```

## Project plan

| Weeks | Milestone |
|-------|-----------|
| 1-2 | Literature review (LSM trees, ShardStore, Verus, crash consistency), finalize scope and tools |
| 3-4 | Abstract spec and TLA+ model of WAL and recovery |
| 5-7 | Implement memtable, WAL, SSTable read/write; verify memtable and WAL |
| 8-9 | Compaction and its proof; crash-recovery proof |
| 10-11 | Benchmarks, fuzzing and fault-injection comparison, limitations analysis |
| 12 | Final report and presentation |

Weekly progress notes are kept in `docs/`.

## Evaluation

- **Correctness:** all proofs pass; list of proven vs. tested-only components.
- **Performance:** throughput and latency for sequential and random writes and reads, compared to a naive baseline and an existing Rust LSM implementation.
- **Bug finding:** seeded bugs (e.g. skipped fsync, wrong tombstone handling) are caught by the proofs or model checker, and compared with what fuzzing detects.

## Scope and limitations

- Single-threaded; no concurrency in the first version.
- The operating system and hardware are trusted: the disk model assumes atomic sector writes and honest `fsync`.
- File I/O is modeled through a thin interface; the proofs cover the logic above it.
- Single-level or limited-level compaction only; no compression or bloom filters unless time allows.

## Stretch goals

- Bloom filters with a "no false negatives" proof
- Multi-level compaction
- Basic concurrent reads during compaction

## Related work

- LSM-tree: O'Neil et al., *The Log-Structured Merge-Tree* (1996)
- LevelDB and RocksDB
- ShardStore: *Using Lightweight Formal Methods to Validate a Key-Value Storage Node in Amazon S3* (AWS)
- Verus: verified Rust systems code
- FSCQ and Yggdrasil: crash-safe file system verification

## Getting started

```bash
git clone <repo-url>
cd ironkv
cargo build
cargo test
# verification (requires Verus installed)
verus src/lib.rs
```

## License

MIT (to be confirmed)
