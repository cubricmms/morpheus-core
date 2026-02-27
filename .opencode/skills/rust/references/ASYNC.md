---
name: rust-async
description: >
  Async/await patterns for Rust with Tokio. Covers runtime setup, channels,
  concurrency primitives, cancellation, and common async anti-patterns.
license: MIT
---

# Async/Await Patterns

## Core Principles

| Rule ID | Principle |
|---------|-----------|
| `async-tokio-runtime` | Use Tokio for production async runtime |
| `async-no-lock-await` | Never hold `Mutex`/`RwLock` across `.await` |
| `async-spawn-blocking` | Use `spawn_blocking` for CPU-intensive work |
| `async-tokio-fs` | Use `tokio::fs` not `std::fs` in async code |
| `async-cancellation-token` | Use `CancellationToken` for graceful shutdown |
| `async-clone-before-await` | Clone data before await, release locks |

## Runtime Setup

```rust
#[tokio::main]
async fn main() -> Result<()> {
    // Your async code here
    Ok(())
}

// Or manual setup for more control
fn main() -> Result<()> {
    let rt = tokio::runtime::Runtime::new()?;
    rt.block_on(async {
        // Your async code
    })
}
```

## The Golden Rule: No Locks Across Await

```rust
// BAD: Holding lock across .await - DEADLOCK RISK!
async fn bad() {
    let guard = mutex.lock().unwrap();
    some_async_op().await;  // Guard held here!
    guard.do_something();
}

// GOOD: Clone what you need before await
async fn good() {
    let data = {
        let guard = mutex.lock().unwrap();
        guard.clone()  // Get the data
    };  // Lock released
    some_async().await;  // No lock held
    process(data);
}

// GOOD: Use async-aware Mutex
use tokio::sync::Mutex;
async fn better() {
    let guard = mutex.lock().await;  // Async mutex
    guard.do_async_op().await;  // OK with tokio::sync::Mutex
}
```

## Channels

### mpsc (Work Queue)

```rust
use tokio::sync::mpsc;

// Bounded for backpressure
let (tx, mut rx) = mpsc::channel::<Job>(100);

// Multiple producers
let tx2 = tx.clone();

// Producer
tokio::spawn(async move {
    tx.send(job).await.expect("receiver dropped");
});

// Consumer
while let Some(job) = rx.recv().await {
    process(job).await;
}
```

### broadcast (Pub/Sub)

```rust
use tokio::sync::broadcast;

// All subscribers get messages
let (tx, mut rx1) = broadcast::channel::<Event>(16);
let mut rx2 = tx.subscribe();

tx.send(event)?;  // rx1 and rx2 both receive
```

### watch (Latest Value)

```rust
use tokio::sync::watch;

// Only most recent value matters
let (tx, mut rx) = watch::channel(State::default());

// Update
tx.send(State::Running)?;

// Read (always has latest)
if *rx.borrow() == State::Running { /* ... */ }

// Wait for change
rx.changed().await?;
let new_state = rx.borrow().clone();
```

### oneshot (Request/Response)

```rust
use tokio::sync::oneshot;

let (tx, rx) = oneshot::channel::<Response>();

// Responder
tokio::spawn(async move {
    let response = handle_request().await;
    tx.send(response).expect("requester cancelled");
});

// Requester
let response = rx.await.expect("responder dropped");
```

## Concurrency Patterns

```rust
// Parallel operations (both run to completion)
let (a, b) = tokio::join!(fetch_a(), fetch_b());

// Fallible parallel (returns on first error)
let (a, b) = tokio::try_join!(fetch_a(), fetch_b())?;

// Racing (first to complete wins)
tokio::select! {
    result = async_op() => {
        println!("Completed: {:?}", result);
    }
    _ = tokio::time::sleep(Duration::from_secs(5)) => {
        println!("Timeout!");
    }
}

// Cancellation
use tokio_util::sync::CancellationToken;
let token = CancellationToken::new();
let child_token = token.child_token();

tokio::select! {
    _ = async_op() => { /* completed */ }
    _ = token.cancelled() => { /* cancelled */ }
}
// Elsewhere: token.cancel();

// Dynamic task groups
let mut set = tokio::task::JoinSet::new();
for job in jobs {
    set.spawn(process(job));
}
while let Some(res) = set.join_next().await {
    match res {
        Ok(output) => handle(output),
        Err(e) => eprintln!("Task failed: {}", e),
    }
}
```

## Blocking Operations

```rust
// BAD: Blocking in async
async fn bad() {
    let data = std::fs::read_to_string("file.txt")?;  // Blocks executor!
}

// GOOD: Use async fs
async fn good() {
    let data = tokio::fs::read_to_string("file.txt").await?;
}

// GOOD: Offload CPU work
async fn cpu_intensive() {
    let result = tokio::task::spawn_blocking(|| {
        heavy_computation()  // Runs on blocking thread pool
    }).await?;
}
```

## Channel Selection Guide

| Need | Channel | Pattern |
|------|---------|---------|
| Work queue | `mpsc` | Multiple producers, single consumer |
| Pub/sub | `broadcast` | All subscribers receive |
| Latest value | `watch` | Only most recent matters |
| Request/response | `oneshot` | Single response |
| Dynamic tasks | `JoinSet` | Spawn N tasks, collect results |

## Async Checklist

- [ ] Using `std::fs` in async → use `tokio::fs`
- [ ] Holding sync Mutex across `.await` → use `tokio::sync::Mutex` or clone data
- [ ] Unbounded channel → add bounds for backpressure
- [ ] CPU work in async → `spawn_blocking`
- [ ] Need graceful shutdown → `CancellationToken`
- [ ] Multiple concurrent operations → `join!` or `JoinSet`
- [ ] Timeout needed → `select!` with `sleep`
