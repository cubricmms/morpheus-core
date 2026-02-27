---
name: rust-ownership
description: >
  Ownership and borrowing patterns for Rust. Covers borrowing vs cloning,
  Cow for conditional ownership, Arc/Rc for shared ownership, and
  RefCell/Mutex/RwLock for interior mutability.
license: MIT
---

# Ownership & Borrowing Patterns

## Core Principles

| Rule ID | Principle |
|---------|-----------|
| `own-borrow-over-clone` | Prefer `&T` borrowing over `.clone()` |
| `own-slice-over-vec` | Accept `&[T]` not `&Vec<T>`, `&str` not `&String` |
| `own-copy-small` | Derive `Copy` for small, trivial types |
| `own-clone-explicit` | Make `Clone` explicit, avoid implicit copies |
| `own-move-large` | Move large data instead of cloning |
| `own-lifetime-elision` | Rely on lifetime elision when possible |

## Borrowing Over Cloning

```rust
// PREFER: Borrow when you don't need ownership
fn process(data: &str) -> String {
    format!("processed: {}", data)
}

// AVOID: Unnecessary clone
fn process(data: String) -> String {  // Bad if you don't modify
    format!("processed: {}", data)
}

// BETTER: Accept slice, most flexible
fn sum(numbers: &[i32]) -> i32 {
    numbers.iter().sum()
}
// Can call with: vec, array, slice
```

## Conditional Ownership with Cow

```rust
use std::borrow::Cow;

// Return borrowed when possible, owned when needed
fn normalize(input: &str) -> Cow<'_, str> {
    if input.chars().any(|c| c.is_uppercase()) {
        // Need to allocate - return owned
        Cow::Owned(input.to_lowercase())
    } else {
        // No changes needed - return borrowed
        Cow::Borrowed(input)
    }
}

// Usage
let s = normalize("hello");     // Borrowed, no allocation
let s = normalize("HELLO");     // Owned, allocation happened
```

## Shared Ownership

### Arc (Thread-Safe)

```rust
use std::sync::Arc;

// Arc for thread-safe shared ownership
let data = Arc::new(vec![1, 2, 3]);
let data_clone = Arc::clone(&data);  // Atomic refcount increment

std::thread::spawn(move || {
    println!("{:?}", data_clone);  // Access from another thread
});
```

### Rc (Single-Threaded)

```rust
use std::rc::Rc;

// Rc for single-threaded shared ownership (cheaper than Arc)
let data = Rc::new(vec![1, 2, 3]);
let data_clone = Rc::clone(&data);  // Non-atomic refcount increment

// NOT Send - cannot cross threads
```

## Interior Mutability

### RefCell (Single-Threaded)

```rust
use std::cell::RefCell;

// RefCell: runtime borrow checking for single-threaded
let data = RefCell::new(vec![1, 2, 3]);

{
    let mut borrowed = data.borrow_mut();
    borrowed.push(4);
}  // Borrow ends

println!("{:?}", data.borrow());  // [1, 2, 3, 4]

// Panics at runtime if borrow rules violated:
// let a = data.borrow();
// let b = data.borrow_mut();  // PANIC: already borrowed
```

### Mutex (Thread-Safe)

```rust
use std::sync::Mutex;

// Mutex: thread-safe interior mutability
let data = Mutex::new(vec![1, 2, 3]);

{
    let mut guard = data.lock().unwrap();
    guard.push(4);
}  // Lock released when guard drops

// WARNING: Never hold across .await!
// BAD:
let guard = mutex.lock().unwrap();
async_op().await;  // DEADLOCK RISK!
drop(guard);

// GOOD:
let data = {
    let guard = mutex.lock().unwrap();
    guard.clone()  // Copy what you need
};
async_op().await;  // Lock released
```

### RwLock (Read-Heavy)

```rust
use std::sync::RwLock;

// RwLock: multiple readers OR single writer
let data = RwLock::new(vec![1, 2, 3]);

// Multiple readers can hold simultaneously
let r1 = data.read().unwrap();
let r2 = data.read().unwrap();  // OK: multiple reads

drop(r1);
drop(r2);

// Writer needs exclusive access
let mut w = data.write().unwrap();  // Blocks until all readers drop
w.push(4);
```

## Decision Matrix

| Need | Use |
|------|-----|
| Shared ownership (multi-thread) | `Arc<T>` |
| Shared ownership (single-thread) | `Rc<T>` |
| Interior mutability (multi-thread) | `Mutex<T>` or `RwLock<T>` |
| Interior mutability (single-thread) | `RefCell<T>` |
| Conditional ownership | `Cow<'a, T>` |
| Both shared + mutable (multi-thread) | `Arc<Mutex<T>>` |
| Both shared + mutable (single-thread) | `Rc<RefCell<T>>` |

## Quick Checklist

- [ ] Accepting `&String` → change to `&str`
- [ ] Accepting `&Vec<T>` → change to `&[T]`
- [ ] Calling `.clone()` → can you borrow instead?
- [ ] Using `Arc` in single-threaded code → consider `Rc`
- [ ] Using `Mutex` when reads dominate → consider `RwLock`
- [ ] Holding lock across `.await` → clone data first
