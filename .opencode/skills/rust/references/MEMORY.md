---
name: rust-memory
description: >
  Memory optimization patterns for Rust. Covers pre-allocation,
  collection reuse, type sizing, zero-copy, and allocation avoidance.
license: MIT
---

# Memory Optimization Patterns

## Core Principles

| Rule ID | Principle |
|---------|-----------|
| `mem-with-capacity` | Use `with_capacity()` when size is known |
| `mem-reuse-collections` | Reuse collections with `clear()` in loops |
| `mem-box-large-variant` | Box large enum variants to reduce type size |
| `mem-avoid-format` | Avoid `format!()` when literals work |
| `mem-smaller-integers` | Use smallest integer type that fits |
| `mem-assert-type-size` | Assert hot type sizes to prevent regressions |

## Pre-Allocation

```rust
// BAD: Multiple reallocations as vec grows
let mut v = Vec::new();
for i in 0..1000 {
    v.push(i);  // May reallocate multiple times
}

// GOOD: Single allocation
let mut v = Vec::with_capacity(1000);
for i in 0..1000 {
    v.push(i);  // No reallocations
}

// Same for String, HashMap, etc.
let mut s = String::with_capacity(256);
let mut map = HashMap::with_capacity(100);
```

## Reusing Collections

```rust
// BAD: New allocation each iteration
for chunk in data.chunks(100) {
    let mut buffer = Vec::new();  // Allocates each time
    process(chunk, &mut buffer);
}

// GOOD: Reuse allocation
let mut buffer = Vec::with_capacity(100);
for chunk in data.chunks(100) {
    buffer.clear();  // Keeps capacity
    process(chunk, &mut buffer);
}

// Clone from to reuse destination allocation
let mut dest = Vec::with_capacity(1000);
// ... later ...
dest.clone_from(&source);  // Reuses dest's allocation
```

## Type Size Optimization

```rust
// BAD: Large enum variant increases size for all
enum Message {
    Small(u32),           // 4 bytes
    Large([u8; 1024]),    // 1024 bytes - makes Message 1024+ bytes!
}

// GOOD: Box large variants
enum Message {
    Small(u32),               // 4 bytes
    Large(Box<[u8; 1024]>),   // 8 bytes (pointer)
}
// Message is now ~16 bytes (discriminant + largest variant)

// Use smallest integer that fits
struct Header {
    version: u8,   // not u32 (saves 3 bytes)
    flags: u16,    // not u32 (saves 2 bytes)
    length: u32,   // need 32 bits for large lengths
}

// Assert type sizes
const _: () = assert!(std::mem::size_of::<Header>() <= 8);
const _: () = assert!(std::mem::size_of::<Message>() <= 16);
```

## Small Collections

```rust
use smallvec::SmallVec;

// Stack-allocated until exceeds N, then heap
let mut v: SmallVec<[i32; 8]> = SmallVec::new();
v.push(1);  // On stack
// ... up to 8 elements on stack ...
v.push(9);  // Now on heap

use arrayvec::ArrayVec;

// Fixed capacity, never heap allocates
let mut v: ArrayVec<i32, 8> = ArrayVec::new();
v.push(1);
// v.push(9);  // Would panic - capacity exceeded

use thin_vec::ThinVec;

// No size overhead for empty vector (just pointer)
let v: ThinVec<i32> = ThinVec::new();  // 8 bytes (just pointer)
// vs Vec<i32> which is 24 bytes (ptr, len, cap)
```

## String Optimization

```rust
// BAD: Allocates when literal works
let msg = format!("error");  // Heap allocation

// GOOD: No allocation
let msg: &str = "error";

// BAD: Multiple allocations
let s = format!("{}-{}-{}", a, b, c);

// GOOD: Write to existing buffer
use std::fmt::Write;
let mut s = String::with_capacity(64);
write!(&mut s, "{}-{}-{}", a, b, c).unwrap();

// Consider compact strings
use compact_str::CompactString;
// Small string optimization (24 bytes inline)
let s = CompactString::from("hello");  // No heap allocation
```

## Zero-Copy Patterns

```rust
use bytes::Bytes;

// Reference-counted slice, no copying
let data: Bytes = file_contents.into();
let chunk1 = data.slice(0..100);   // No copy
let chunk2 = data.slice(100..200); // No copy
// All share the same underlying buffer

// Cow for conditional copying
use std::borrow::Cow;
fn process(input: &str) -> Cow<'_, str> {
    if needs_modification(input) {
        Cow::Owned(modify(input))  // Copy only when needed
    } else {
        Cow::Borrowed(input)        // No copy
    }
}
```

## Fixed Collections

```rust
// BAD: Vec when size never changes
let v: Vec<i32> = vec![1, 2, 3, 4, 5];  // 24 bytes + heap

// GOOD: Boxed slice when fixed size
let v: Box<[i32]> = Box::new([1, 2, 3, 4, 5]);  // 16 bytes + heap
// Or from vec
let v: Box<[i32]> = vec.into_boxed_slice();
```

## Memory Checklist

- [ ] Growing vec in loop → `with_capacity()`
- [ ] Collection in loop → reuse with `clear()`
- [ ] Large enum variant → Box it
- [ ] `format!("literal")` → use `&str`
- [ ] String concatenation in loop → pre-allocate, `write!()`
- [ ] u32 for small values → use u8/u16
- [ ] Usually small vec → `SmallVec`
- [ ] Fixed-size vec → `Box<[T]>`
- [ ] Type size matters → add `const` assertion

## Profiling

```bash
# Check type sizes
cargo rustc -- -Z print-type-sizes

# Memory profiling
valgrind --tool=massif ./target/release/myapp

# Heap profiling
MALLOC_CONF="prof:true,prof_prefix:jeprof.out" ./target/release/myapp
jeprof --svg ./target/release/myapp jeprof.out.*.heap > profile.svg
```
