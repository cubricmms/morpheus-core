---
name: rust-full-reference
description: >
  Complete reference of all 179 Rust best practices rules. Load on-demand for
  comprehensive rule lookup. Organized by category with priority levels.
license: MIT
---

# Rust Best Practices - Full Reference

Comprehensive guide with 179 rules across 14 categories, prioritized by impact.

---

## Priority Categories

| Priority | Category | Rules |
|----------|----------|-------|
| 1 | Ownership & Borrowing | 12 |
| 2 | Error Handling | 12 |
| 3 | Memory Optimization | 15 |
| 4 | API Design | 15 |
| 5 | Async/Await | 15 |
| 6 | Compiler Optimization | 12 |
| 7 | Naming Conventions | 16 |
| 8 | Type Safety | 10 |
| 9 | Testing | 13 |
| 10 | Documentation | 11 |
| 11 | Performance Patterns | 11 |
| 12 | Project Structure | 11 |
| 13 | Clippy & Linting | 11 |
| 14 | Anti-patterns | 15 |

---

## 1. Ownership & Borrowing (CRITICAL)

| Rule ID | Name | Description |
|---------|------|-------------|
| `own-borrow-over-clone` | Prefer borrowing | Use `&T` over `.clone()` when possible |
| `own-slice-over-vec` | Accept slices | `&[T]` not `&Vec<T>`, `&str` not `&String` |
| `own-cow-conditional` | Conditional ownership | Use `Cow<'a, T>` for conditional ownership |
| `own-arc-shared` | Thread-safe shared | `Arc<T>` for thread-safe shared ownership |
| `own-rc-single-thread` | Single-thread shared | `Rc<T>` for single-threaded sharing |
| `own-refcell-interior` | Interior mutability (single) | `RefCell<T>` for interior mutability |
| `own-mutex-interior` | Interior mutability (multi) | `Mutex<T>` for thread-safe interior mutability |
| `own-rwlock-readers` | Read-heavy | `RwLock<T>` when reads dominate writes |
| `own-copy-small` | Small types | Derive `Copy` for small, trivial types |
| `own-clone-explicit` | Explicit clone | Make `Clone` explicit, avoid implicit copies |
| `own-move-large` | Move large data | Move large data instead of cloning |
| `own-lifetime-elision` | Lifetime elision | Rely on lifetime elision when possible |

---

## 2. Error Handling (CRITICAL)

| Rule ID | Name | Description |
|---------|------|-------------|
| `err-thiserror-lib` | Library errors | Use `thiserror` for library error types |
| `err-anyhow-app` | Application errors | Use `anyhow` for application error handling |
| `err-result-over-panic` | Return Result | Return `Result`, don't panic on expected errors |
| `err-context-chain` | Add context | Use `.context()` or `.with_context()` |
| `err-no-unwrap-prod` | No unwrap in prod | Never use `.unwrap()` in production code |
| `err-expect-bugs-only` | Expect for bugs | Use `.expect()` only for programming errors |
| `err-question-mark` | Use ? operator | Use `?` for clean error propagation |
| `err-from-impl` | From trait | Use `#[from]` for automatic error conversion |
| `err-source-chain` | Source chaining | Use `#[source]` to chain underlying errors |
| `err-lowercase-msg` | Error message style | Lowercase, no trailing punctuation |
| `err-doc-errors` | Document errors | Include `# Errors` section in docs |
| `err-custom-type` | Custom error types | Create custom error types, not `Box<dyn Error>` |

---

## 3. Memory Optimization (CRITICAL)

| Rule ID | Name | Description |
|---------|------|-------------|
| `mem-with-capacity` | Pre-allocate | Use `with_capacity()` when size is known |
| `mem-smallvec` | Small vectors | Use `SmallVec` for usually-small collections |
| `mem-arrayvec` | Bounded vectors | Use `ArrayVec` for bounded-size collections |
| `mem-box-large-variant` | Box large variants | Box large enum variants to reduce type size |
| `mem-boxed-slice` | Fixed slices | Use `Box<[T]>` instead of `Vec<T>` when fixed |
| `mem-thinvec` | Often-empty | Use `ThinVec` for often-empty vectors |
| `mem-clone-from` | Clone from | Use `clone_from()` to reuse allocations |
| `mem-reuse-collections` | Reuse collections | Reuse collections with `clear()` in loops |
| `mem-avoid-format` | Avoid format! | Avoid `format!()` when literals work |
| `mem-write-over-format` | Use write! | Use `write!()` instead of `format!()` |
| `mem-arena-allocator` | Arena allocators | Use arena allocators for batch allocations |
| `mem-zero-copy` | Zero-copy | Use zero-copy patterns with slices and `Bytes` |
| `mem-compact-string` | Compact strings | Use `CompactString` for small string optimization |
| `mem-smaller-integers` | Small integers | Use smallest integer type that fits |
| `mem-assert-type-size` | Size assertions | Assert hot type sizes to prevent regressions |

---

## 4. API Design (HIGH)

| Rule ID | Name | Description |
|---------|------|-------------|
| `api-builder-pattern` | Builder pattern | Use Builder pattern for complex construction |
| `api-builder-must-use` | Must use builder | Add `#[must_use]` to builder types |
| `api-newtype-safety` | Newtype safety | Use newtypes for type-safe distinctions |
| `api-typestate` | Typestate pattern | Use typestate for compile-time state machines |
| `api-sealed-trait` | Sealed traits | Seal traits to prevent external implementations |
| `api-extension-trait` | Extension traits | Use extension traits for foreign type methods |
| `api-parse-dont-validate` | Parse don't validate | Parse into validated types at boundaries |
| `api-impl-into` | impl Into | Accept `impl Into<T>` for flexible string inputs |
| `api-impl-asref` | impl AsRef | Accept `impl AsRef<T>` for borrowed inputs |
| `api-must-use` | Must use Result | Add `#[must_use]` to Result-returning functions |
| `api-non-exhaustive` | Non-exhaustive | Use `#[non_exhaustive]` for future-proof types |
| `api-from-not-into` | From not Into | Implement `From`, not `Into` (auto-derived) |
| `api-default-impl` | Default impl | Implement `Default` for sensible defaults |
| `api-common-traits` | Common traits | Implement `Debug`, `Clone`, `PartialEq` eagerly |
| `api-serde-optional` | Optional serde | Gate serde behind feature flag |

---

## 5. Async/Await (HIGH)

| Rule ID | Name | Description |
|---------|------|-------------|
| `async-tokio-runtime` | Tokio runtime | Use Tokio for production async runtime |
| `async-no-lock-await` | No lock across await | Never hold locks across `.await` |
| `async-spawn-blocking` | Spawn blocking | Use `spawn_blocking` for CPU-intensive work |
| `async-tokio-fs` | Async fs | Use `tokio::fs` not `std::fs` in async code |
| `async-cancellation-token` | Cancellation | Use `CancellationToken` for graceful shutdown |
| `async-join-parallel` | Join parallel | Use `tokio::join!` for parallel operations |
| `async-try-join` | Try join | Use `tokio::try_join!` for fallible parallel ops |
| `async-select-racing` | Select racing | Use `tokio::select!` for racing/timeouts |
| `async-bounded-channel` | Bounded channels | Use bounded channels for backpressure |
| `async-mpsc-queue` | mpsc queues | Use `mpsc` for work queues |
| `async-broadcast-pubsub` | Broadcast | Use `broadcast` for pub/sub patterns |
| `async-watch-latest` | Watch | Use `watch` for latest-value sharing |
| `async-oneshot-response` | Oneshot | Use `oneshot` for request/response |
| `async-joinset-structured` | JoinSet | Use `JoinSet` for dynamic task groups |
| `async-clone-before-await` | Clone before await | Clone data before await, release locks |

---

## 6. Compiler Optimization (HIGH)

| Rule ID | Name | Description |
|---------|------|-------------|
| `opt-inline-small` | Inline small | Use `#[inline]` for small hot functions |
| `opt-inline-always-rare` | Inline always rare | Use `#[inline(always)]` sparingly |
| `opt-inline-never-cold` | Inline never cold | Use `#[inline(never)]` for cold paths |
| `opt-cold-unlikely` | Cold attribute | Use `#[cold]` for error/unlikely paths |
| `opt-likely-hint` | Branch hints | Use `likely()`/`unlikely()` for branch hints |
| `opt-lto-release` | LTO | Enable LTO in release builds |
| `opt-codegen-units` | Codegen units | Use `codegen-units = 1` for max optimization |
| `opt-pgo-profile` | PGO | Use PGO for production builds |
| `opt-target-cpu` | Target CPU | Set `target-cpu=native` for local builds |
| `opt-bounds-check` | Avoid bounds check | Use iterators to avoid bounds checks |
| `opt-simd-portable` | Portable SIMD | Use portable SIMD for data-parallel ops |
| `opt-cache-friendly` | Cache friendly | Design cache-friendly data layouts (SoA) |

---

## 7. Naming Conventions (MEDIUM)

| Rule ID | Name | Description |
|---------|------|-------------|
| `name-types-camel` | Types camelCase | `UpperCamelCase` for types, traits, enums |
| `name-variants-camel` | Variants camelCase | `UpperCamelCase` for enum variants |
| `name-funcs-snake` | Functions snake_case | `snake_case` for functions, methods, modules |
| `name-consts-screaming` | Constants SCREAMING | `SCREAMING_SNAKE_CASE` for constants/statics |
| `name-lifetime-short` | Short lifetimes | Use short lowercase: `'a`, `'de`, `'src` |
| `name-type-param-single` | Single type params | Single uppercase: `T`, `E`, `K`, `V` |
| `name-as-free` | as_ prefix | `as_` for free reference conversion |
| `name-to-expensive` | to_ prefix | `to_` for expensive conversion |
| `name-into-ownership` | into_ prefix | `into_` for ownership transfer |
| `name-no-get-prefix` | No get_ prefix | No `get_` prefix for simple getters |
| `name-is-has-bool` | Boolean methods | Use `is_`, `has_`, `can_` for boolean methods |
| `name-iter-convention` | Iter convention | Use `iter`/`iter_mut`/`into_iter` |
| `name-iter-method` | Iter methods | Name iterator methods consistently |
| `name-iter-type-match` | Iterator types | Iterator type names match method |
| `name-acronym-word` | Acronym as word | Treat acronyms as words: `Uuid` not `UUID` |
| `name-crate-no-rs` | No -rs suffix | Crate names: no `-rs` suffix |

---

## 8. Type Safety (MEDIUM)

| Rule ID | Name | Description |
|---------|------|-------------|
| `type-newtype-ids` | ID newtypes | Wrap IDs in newtypes: `UserId(u64)` |
| `type-newtype-validated` | Validated newtypes | Newtypes for validated data: `Email`, `Url` |
| `type-enum-states` | Enum for states | Use enums for mutually exclusive states |
| `type-option-nullable` | Option for nullable | Use `Option<T>` for nullable values |
| `type-result-fallible` | Result for fallible | Use `Result<T, E>` for fallible operations |
| `type-phantom-marker` | PhantomData | Use `PhantomData<T>` for type-level markers |
| `type-never-diverge` | Never type | Use `!` type for functions that never return |
| `type-generic-bounds` | Generic bounds | Add trait bounds only where needed |
| `type-no-stringly` | No stringly typed | Avoid stringly-typed APIs, use enums/newtypes |
| `type-repr-transparent` | repr transparent | Use `#[repr(transparent)]` for FFI newtypes |

---

## 9. Testing (MEDIUM)

| Rule ID | Name | Description |
|---------|------|-------------|
| `test-cfg-test-module` | Test module | Use `#[cfg(test)] mod tests { }` |
| `test-use-super` | Use super | Use `use super::*;` in test modules |
| `test-integration-dir` | Integration dir | Put integration tests in `tests/` directory |
| `test-descriptive-names` | Descriptive names | Use descriptive test names |
| `test-arrange-act-assert` | AAA pattern | Structure tests as arrange/act/assert |
| `test-proptest-properties` | Property testing | Use `proptest` for property-based testing |
| `test-mockall-mocking` | Mockall | Use `mockall` for trait mocking |
| `test-mock-traits` | Mock traits | Use traits for dependencies to enable mocking |
| `test-fixture-raii` | RAII fixtures | Use RAII pattern (Drop) for test cleanup |
| `test-tokio-async` | Tokio test | Use `#[tokio::test]` for async tests |
| `test-should-panic` | Should panic | Use `#[should_panic]` for panic tests |
| `test-criterion-bench` | Criterion | Use `criterion` for benchmarking |
| `test-doctest-examples` | Doctest | Keep doc examples as executable tests |

---

## 10. Documentation (MEDIUM)

| Rule ID | Name | Description |
|---------|------|-------------|
| `doc-all-public` | Document public | Document all public items with `///` |
| `doc-module-inner` | Module docs | Use `//!` for module-level documentation |
| `doc-examples-section` | Examples section | Include `# Examples` with runnable code |
| `doc-errors-section` | Errors section | Include `# Errors` for fallible functions |
| `doc-panics-section` | Panics section | Include `# Panics` for panicking functions |
| `doc-safety-section` | Safety section | Include `# Safety` for unsafe functions |
| `doc-question-mark` | Use ? in docs | Use `?` in examples, not `.unwrap()` |
| `doc-hidden-setup` | Hidden setup | Use `# ` prefix to hide example setup |
| `doc-intra-links` | Intra-doc links | Use intra-doc links: `[Vec]` |
| `doc-link-types` | Link types | Link related types and functions in docs |
| `doc-cargo-metadata` | Cargo metadata | Fill `Cargo.toml` metadata |

---

## 11. Performance Patterns (MEDIUM)

| Rule ID | Name | Description |
|---------|------|-------------|
| `perf-iter-over-index` | Iterator over index | Prefer iterators over manual indexing |
| `perf-iter-lazy` | Lazy iterators | Keep iterators lazy, collect() only when needed |
| `perf-collect-once` | Collect once | Don't `collect()` intermediate iterators |
| `perf-entry-api` | Entry API | Use `entry()` API for map insert-or-update |
| `perf-drain-reuse` | Drain reuse | Use `drain()` to reuse allocations |
| `perf-extend-batch` | Extend batch | Use `extend()` for batch insertions |
| `perf-chain-avoid` | Avoid chain | Avoid `chain()` in hot loops |
| `perf-collect-into` | Collect into | Use `collect_into()` for reusing containers |
| `perf-black-box-bench` | Black box | Use `black_box()` in benchmarks |
| `perf-release-profile` | Release profile | Optimize release profile settings |
| `perf-profile-first` | Profile first | Profile before optimizing |

---

## 12. Project Structure (LOW)

| Rule ID | Name | Description |
|---------|------|-------------|
| `proj-lib-main-split` | lib/main split | Keep `main.rs` minimal, logic in `lib.rs` |
| `proj-mod-by-feature` | Feature modules | Organize modules by feature, not type |
| `proj-flat-small` | Flat structure | Keep small projects flat |
| `proj-mod-rs-dir` | mod.rs | Use `mod.rs` for multi-file modules |
| `proj-pub-crate-internal` | pub(crate) | Use `pub(crate)` for internal APIs |
| `proj-pub-super-parent` | pub(super) | Use `pub(super)` for parent-only visibility |
| `proj-pub-use-reexport` | pub use reexport | Use `pub use` for clean public API |
| `proj-prelude-module` | Prelude module | Create `prelude` module for common imports |
| `proj-bin-dir` | bin directory | Put multiple binaries in `src/bin/` |
| `proj-workspace-large` | Workspace | Use workspaces for large projects |
| `proj-workspace-deps` | Workspace deps | Use workspace dependency inheritance |

---

## 13. Clippy & Linting (LOW)

| Rule ID | Name | Description |
|---------|------|-------------|
| `lint-deny-correctness` | Deny correctness | `#![deny(clippy::correctness)]` |
| `lint-warn-suspicious` | Warn suspicious | `#![warn(clippy::suspicious)]` |
| `lint-warn-style` | Warn style | `#![warn(clippy::style)]` |
| `lint-warn-complexity` | Warn complexity | `#![warn(clippy::complexity)]` |
| `lint-warn-perf` | Warn perf | `#![warn(clippy::perf)]` |
| `lint-pedantic-selective` | Pedantic selective | Enable `clippy::pedantic` selectively |
| `lint-missing-docs` | Missing docs | `#![warn(missing_docs)]` |
| `lint-unsafe-doc` | Document unsafe | `#![warn(clippy::undocumented_unsafe_blocks)]` |
| `lint-cargo-metadata` | Cargo metadata | `#![warn(clippy::cargo)]` for published crates |
| `lint-rustfmt-check` | Rustfmt check | Run `cargo fmt --check` in CI |
| `lint-workspace-lints` | Workspace lints | Configure lints at workspace level |

---

## 14. Anti-patterns (REFERENCE)

| Rule ID | Name | Problem | Solution |
|---------|------|---------|----------|
| `anti-unwrap-abuse` | Unwrap abuse | Panics on error | Return `Result`, use `?` |
| `anti-expect-lazy` | Lazy expect | Recoverable as panic | Use `anyhow::Context` |
| `anti-clone-excessive` | Excessive clone | Unnecessary allocation | Borrow when possible |
| `anti-lock-across-await` | Lock across await | Deadlock risk | Clone before await |
| `anti-string-for-str` | &String param | Less flexible | Use `&str` |
| `anti-vec-for-slice` | &Vec param | Less flexible | Use `&[T]` |
| `anti-index-over-iter` | Index over iter | Bounds checks | Use iterators |
| `anti-panic-expected` | Panic expected | Should recover | Return `Result` |
| `anti-empty-catch` | Empty catch | Swallowed error | Log or propagate |
| `anti-over-abstraction` | Over abstraction | Excessive generics | Keep it simple |
| `anti-premature-optimize` | Premature opt | Wasted effort | Profile first |
| `anti-type-erasure` | Type erasure | Heap allocation | Use `impl Trait` |
| `anti-format-hot-path` | Format in hot | Allocations | Use `write!()` |
| `anti-collect-intermediate` | Collect intermediate | Extra allocation | Keep lazy |
| `anti-stringly-typed` | Stringly typed | No type safety | Use enums/newtypes |

---

## Sources

This reference synthesizes best practices from:
- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- [Rust Performance Book](https://nnethercote.github.io/perf-book/)
- [Rust Design Patterns](https://rust-unofficial.github.io/patterns/)
- Production codebases: ripgrep, tokio, serde, polars, axum, deno
- Clippy lint documentation
- Community conventions (2024-2025)
