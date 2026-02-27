---
name: rust-performance
description: >
  Performance patterns for Rust. Covers iterator optimization,
  compiler hints, profiling, and common performance anti-patterns.
license: MIT
---

# Performance Patterns

## Core Principles

| Rule ID | Principle |
|---------|-----------|
| `perf-iter-over-index` | Prefer iterators over manual indexing |
| `perf-iter-lazy` | Keep iterators lazy, `collect()` only when needed |
| `perf-entry-api` | Use `entry()` API for map insert-or-update |
| `perf-profile-first` | Profile before optimizing |
| `opt-inline-small` | Use `#[inline]` for small hot functions |
| `opt-bounds-check` | Use iterators to avoid bounds checks |

## Iterators Over Indexing

```rust
// BAD: Bounds check on every access
let sum: i32 = (0..slice.len()).map(|i| slice[i]).sum();

// GOOD: Iterator avoids bounds check
let sum: i32 = slice.iter().sum();

// BAD: Manual loop with indexing
let mut result = Vec::new();
for i in 0..items.len() {
    result.push(items[i].process());
}

// GOOD: Iterator chain
let result: Vec<_> = items.iter().map(|x| x.process()).collect();
```

## Lazy Iterators

```rust
// BAD: Multiple intermediate collections
let v1: Vec<_> = data.iter().filter(|x| x.valid()).collect();
let v2: Vec<_> = v1.iter().map(|x| x.transform()).collect();
let result: Vec<_> = v2.iter().take(10).collect();

// GOOD: Single pass, one allocation
let result: Vec<_> = data
    .iter()
    .filter(|x| x.valid())
    .map(|x| x.transform())
    .take(10)
    .collect();

// Keep lazy until you need to collect
let iter = data.iter().filter(|x| x.valid());
if condition {
    iter.for_each(|x| process(x));  // No allocation
} else {
    let v: Vec<_> = iter.collect(); // Only collect if needed
}
```

## Entry API

```rust
use std::collections::HashMap;

// BAD: Two lookups
if !map.contains_key(&key) {
    map.insert(key, default());
}
let value = map.get_mut(&key).unwrap();

// GOOD: Single lookup
let value = map.entry(key).or_insert_with(|| expensive_default());

// or_insert for simple defaults
map.entry(key).or_insert(0);
```

## Collection Operations

```rust
// Use extend for batch insertions
vec.extend(other_vec);  // More efficient than loop with push

// Use drain to reuse allocation
let old: Vec<i32> = vec.drain(..).collect();  // vec is empty but keeps capacity

// Use append to move all elements
vec1.append(&mut vec2);  // vec2 is emptied
```

## Compiler Hints

```rust
// Inline small hot functions
#[inline]
fn hash(x: u64) -> u64 {
    x.wrapping_mul(0x9e3779b97f4a7c15)
}

// Rarely use always (let compiler decide)
#[inline(always)]  // Only for very small, very hot functions
fn critical_path() {}

// Cold paths
#[cold]
fn handle_rare_error() {}

// Never inline for cold or very large functions
#[inline(never)]
fn large_cold_function() {}

// Branch hints (nightly: std::intrinsics::{likely, unlikely})
// Portable alternative
fn unlikely<T>(x: T) -> T { x }  // Hint to compiler
if unlikely(error_condition) {
    return Err(/* ... */);
}
```

## Release Profile

```toml
[profile.release]
opt-level = 3          # Maximum optimization
lto = "fat"            # Link-time optimization
codegen-units = 1      # Better optimizations (slower compile)
panic = "abort"        # Smaller binary, no unwinding
strip = true           # Remove symbols

[profile.bench]
inherits = "release"
debug = true           # Keep debug for profiling
strip = false

# Optimize dependencies even in dev
[profile.dev.package."*"]
opt-level = 3

# PGO for production builds (requires rustup component)
# RUSTFLAGS="-C profile-generate=<profile>" cargo build --release
# Run benchmark
# RUSTFLAGS="-C profile-use=<profile>" cargo build --release
```

## SIMD

```rust
// Portable SIMD (nightly)
#![feature(portable_simd)]
use std::simd::*;

let a = f32x4::from_array([1.0, 2.0, 3.0, 4.0]);
let b = f32x4::splat(2.0);
let c = a * b;  // 4 multiplies in parallel

// Stable: use packed_simd_2 or explicit intrinsics
```

## Benchmarking

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn bench(c: &mut Criterion) {
    c.bench_function("my_function", |b| {
        b.iter(|| {
            // black_box prevents optimization away
            my_function(black_box(&input))
        })
    });
}

criterion_group!(benches, bench);
criterion_main!(benches);
```

## Profiling Commands

```bash
# CPU profiling
perf record -g ./target/release/myapp
perf report

# Flamegraph
perf record -g ./target/release/myapp
perf script | stackcollapse-perf.pl | flamegraph.pl > flame.svg

# Or use cargo-flamegraph
cargo flamegraph --release

# Cache analysis
perf stat -e cache-references,cache-misses ./target/release/myapp

# Call graph
cargo rustc --release -- -Z print-type-sizes
```

## Common Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Indexing when iter works | Bounds checks | `.iter()` |
| Multiple `.collect()` | Extra allocations | Chain lazy |
| `format!()` in loop | Allocations | Pre-allocate |
| `contains_key` + `insert` | Two lookups | `entry()` |
| `clone()` in hot path | Copy cost | Borrow or Cow |
| `chain()` in hot loop | Iterator overhead | Flatten input first |
| Premature optimization | Wasted time | Profile first |

## Performance Checklist

- [ ] Manual indexing → iterators
- [ ] Multiple collects → single lazy chain
- [ ] `contains_key` + insert → `entry()`
- [ ] Hot small function → `#[inline]`
- [ ] Error path → `#[cold]`
- [ ] Release profile → LTO, codegen-units=1
- [ ] Claiming "slow" → show profile data
