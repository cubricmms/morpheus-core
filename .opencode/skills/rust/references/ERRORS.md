---
name: rust-errors
description: >
  Error handling patterns for Rust. Covers thiserror for libraries,
  anyhow for applications, Result propagation, context chaining,
  and production error conventions.
license: MIT
---

# Error Handling Patterns

## Core Principles

| Rule ID | Principle |
|---------|-----------|
| `err-thiserror-lib` | Use `thiserror` for library error types |
| `err-anyhow-app` | Use `anyhow` for application error handling |
| `err-result-over-panic` | Return `Result`, don't panic on expected errors |
| `err-context-chain` | Add context with `.context()` or `.with_context()` |
| `err-no-unwrap-prod` | Never use `.unwrap()` in production code |
| `err-expect-bugs-only` | Use `.expect()` only for programming errors |
| `err-question-mark` | Use `?` operator for clean propagation |

## Library Errors (thiserror)

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum DataStoreError {
    #[error("data store disconnected")]
    Disconnect(#[from] io::Error),  // Auto-convert with #[from]
    
    #[error("the data for key `{0}` is not available")]
    Redaction(String),
    
    #[error("invalid header (expected {expected:?}, found {found:?})")]
    InvalidHeader {
        expected: String,
        found: String,
    },
    
    #[error("unknown data store error")]
    Unknown,
}

// Usage
fn get_data() -> Result<Data, DataStoreError> {
    let file = File::open("data.bin")?;  // io::Error auto-converts
    Ok(parse(file)?)
}
```

## Application Errors (anyhow)

```rust
use anyhow::{Context, Result, bail, ensure};

fn load_config(path: &str) -> Result<Config> {
    let content = std::fs::read_to_string(path)
        .context(format!("Failed to read config from {}", path))?;
    
    let config: Config = toml::from_str(&content)
        .context("Failed to parse config TOML")?;
    
    // Early return with error
    ensure!(config.port > 0, "Port must be positive");
    
    // Bail out with custom error
    if config.database.is_empty() {
        bail!("Database path is required");
    }
    
    Ok(config)
}

// Context adds a chain: "Failed to read config" -> "No such file or directory"
```

## Result Propagation

```rust
// Use ? for clean propagation
fn process() -> Result<Output, Error> {
    let data = read_input()?;       // Returns on error
    let parsed = parse(&data)?;     // Returns on error
    let result = compute(parsed)?;  // Returns on error
    Ok(result)
}

// Works with From/Into for automatic conversion
fn example() -> Result<(), AppError> {
    let file = File::open("data.txt")?;  // io::Error -> AppError via From
    Ok(())
}
```

## Error Documentation

```rust
/// Loads user data from the database.
///
/// # Errors
///
/// Returns an error if:
/// - The database connection fails
/// - The user ID does not exist
/// - The data is corrupted
pub fn load_user(id: u64) -> Result<User, DbError> {
    // ...
}
```

## Never Do This

```rust
// BAD: unwrap in production
fn load() -> Config {
    let s = std::fs::read_to_string("config.toml").unwrap();  // PANICS!
    toml::from_str(&s).unwrap()  // PANICS!
}

// BAD: expect for recoverable errors
fn parse_id(s: &str) -> u64 {
    s.parse().expect("invalid ID")  // Should return Result!
}

// BAD: swallowing errors
if let Err(_) = operation() {
    // Error silently ignored!
}

// BAD: empty catch
match operation() {
    Ok(val) => process(val),
    Err(_) => {},  // Silent failure!
}
```

## When to Use Each

| Situation | Use |
|-----------|-----|
| Library public API | `thiserror` with custom error type |
| Application code | `anyhow::Result` |
| Expected failure | Return `Result` |
| Programming bug (unreachable) | `.expect("invariant violated")` |
| Need error context | `.context("...")` |
| Need to add info | `bail!("...")` or `ensure!(cond, "...")` |

## Error Message Style

```rust
// GOOD: lowercase, no trailing punctuation
#[error("connection timeout after {seconds} seconds")]

// BAD: capitalized, trailing period
#[error("Connection timeout after {seconds} seconds.")]
```

## Custom Error with Source Chaining

```rust
#[derive(Error, Debug)]
pub enum AppError {
    #[error("configuration error")]
    Config(#[from] ConfigError),  // Auto From impl
    
    #[error("database error")]
    Database {
        #[source]  // Chain underlying error
        source: DbError,
        table: String,
    },
    
    #[error("IO error in {context}")]
    Io {
        #[source]
        source: io::Error,
        context: String,
    },
}
```
