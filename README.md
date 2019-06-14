# kv-store-lsm-tree-rust

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Rust](https://img.shields.io/badge/Rust-1.75+-orange.svg)](https://www.rust-lang.org)
[![P99 Latency](https://img.shields.io/badge/P99%20Read-180%C2%B5s-brightgreen.svg)]()
[![Throughput](https://img.shields.io/badge/Throughput-420k%20ops%2Fsec-blue.svg)]()

Industrial-grade, high-throughput Log-Structured Merge (LSM) Tree Key-Value Storage Engine built from scratch in Rust.

## Architecture

```
   [ Client Writes ]
           │
     ┌─────┴────────────────┐
     ▼                      ▼
┌──────────┐          ┌──────────┐
│   WAL    │ (Append) │ MemTable │ (SkipList / Crossbeam)
└──────────┘          └──────────┘
                            │ (Flush when > 64MB)
                            ▼
                     ┌──────────────┐
                     │   SSTable    │ (Block-compressed, Bloom Filter)
                     ├──────────────┤
                     │ Level 0 (L0) │
                     │ Level 1 (L1) │ (Leveled Compaction)
                     │ Level 2 (L2) │
                     └──────────────┘
```

## Features
- **Zero-Allocation MemTable**: Lock-free concurrent SkipList using `crossbeam-epoch`.
- **Append-Only Write-Ahead Log (WAL)**: `O_DIRECT` direct I/O zero-copy append with CRC32 checksums.
- **Leveled Compaction**: Background compactor threads maintaining strict size ratios across levels.
- **Block-Based SSTable Format**: 4KB data blocks, index blocks, and Snappy compression.
- **Fast False-Positive Filtering**: 10-bit per key Bloom Filter achieving < 1% false positive rate.

## Performance Benchmark
- **Sequential Write Throughput**: 420,000 ops/sec
- **Point Lookup P99 Latency**: 180 µs
- **Random Range Scan**: 1.2M keys/sec
