---
name: rust
description: >
  Rust best practices for high-quality, idiomatic code. Covers ownership patterns,
  error handling, async/await, API design, memory optimization, and performance.
  Load when writing, reviewing, or refactoring Rust code.
license: MIT
metadata:
  version: "1.0.0"
  sources:
    - Rust API Guidelines
    - Rust Performance Book
    - ripgrep, tokio, serde, polars codebases
---

# Rust Best Practices

## When to Use This Skill

- Writing new Rust functions, structs, or modules
- Implementing error handling or async code
- Designing public APIs for libraries
- Reviewing code for ownership/borrowing issues
- Optimizing memory usage or performance

## Task Reference Files

Load the appropriate reference for your task:

| Task | Reference File |
|------|---------------|
| Ownership, borrowing, Cow, Arc/Rc, RefCell/Mutex | [references/OWNERSHIP.md](references/OWNERSHIP.md) |
| Error handling, thiserror, anyhow, Result patterns | [references/ERRORS.md](references/ERRORS.md) |
| Async/await, tokio, channels, concurrency | [references/ASYNC.md](references/ASYNC.md) |
| API design, builders, newtypes, traits | [references/API.md](references/API.md) |
| Memory optimization, allocations, type sizing | [references/MEMORY.md](references/MEMORY.md) |
| Performance, iterators, compiler hints | [references/PERFORMANCE.md](references/PERFORMANCE.md) |
| Complete 179-rule reference by category | [references/RUST_FULL.md](references/RUST_FULL.md) |

## Quick Reference: Anti-Patterns

| Anti-Pattern | Problem | Instead |
|--------------|---------|---------|
| `.unwrap()` in production | Panics on error | Return `Result`, use `?` |
| `.expect()` for recoverable errors | Should be Result | `anyhow::Context` |
| Clone when borrow works | Unnecessary allocation | `&T` reference |
| `&String` parameter | Less flexible | `&str` |
| `&Vec<T>` parameter | Less flexible | `&[T]` |
| Indexing when iterators work | Bounds checks, verbose | `.iter()` |
| Panic on expected errors | Should recover | Return `Result` |
| Empty `if let Err(_) = ...` | Swallowed error | Log or propagate |
| `Box<dyn Trait>` when `impl Trait` works | Heap allocation | `impl Trait` |
| `format!()` in hot paths | Allocations | `write!()` to buffer |
| Intermediate `.collect()` | Extra allocation | Keep lazy |
| String for structured data | No type safety | Enums/newtypes |
| Hold lock across `.await` | Deadlock risk | Clone before await |

## Essential Dependencies

```toml
[dependencies]
thiserror = "1.0"  # Library errors
anyhow = "1.0"     # Application errors
tokio = { version = "1", features = ["full"] }  # Async runtime
serde = { version = "1", features = ["derive"] }  # Serialization
smallvec = "1.0"   # Stack-allocated small vectors
itertools = "0.12" # Iterator utilities
```

## Recommended Cargo.toml

```toml
[profile.release]
opt-level = 3
lto = "fat"
codegen-units = 1
panic = "abort"
strip = true

[profile.dev.package."*"]
opt-level = 3  # Optimize dependencies in dev

[workspace.lints.clippy]
correctness = { level = "deny", priority = -1 }
suspicious = { level = "warn", priority = -1 }
perf = { level = "warn", priority = -1 }
```
