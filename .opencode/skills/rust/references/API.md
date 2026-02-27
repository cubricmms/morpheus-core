---
name: rust-api
description: >
  API design patterns for Rust. Covers builder pattern, newtypes,
  typestate pattern, trait design, and public API conventions.
license: MIT
---

# API Design Patterns

## Core Principles

| Rule ID | Principle |
|---------|-----------|
| `api-builder-pattern` | Use Builder pattern for complex construction |
| `api-newtype-safety` | Use newtypes for type-safe distinctions |
| `api-impl-into` | Accept `impl Into<T>` for flexible string inputs |
| `api-impl-asref` | Accept `impl AsRef<T>` for borrowed inputs |
| `api-must-use` | Add `#[must_use]` to Result-returning functions |
| `api-non-exhaustive` | Use `#[non_exhaustive]` for future-proof types |
| `api-from-not-into` | Implement `From`, not `Into` (auto-derived) |

## Flexible Parameters

```rust
// Accept any string-like input
fn greet(name: impl Into<String>) {
    let name = name.into();
    println!("Hello, {}", name);
}
greet("world");              // &str
greet(String::from("world")); // String
greet(format!("user-{}", 1)); // String

// Accept borrowed string-like
fn process(data: impl AsRef<str>) {
    let s: &str = data.as_ref();
    // work with s
}
process("hello");
process(String::from("hello"));
```

## Builder Pattern

```rust
#[derive(Debug, Default)]
#[must_use]  // Warn if builder not used
pub struct ServerBuilder {
    host: Option<String>,
    port: Option<u16>,
    timeout: Option<Duration>,
}

impl ServerBuilder {
    pub fn new() -> Self {
        Self::default()
    }
    
    pub fn host(mut self, host: impl Into<String>) -> Self {
        self.host = Some(host.into());
        self
    }
    
    pub fn port(mut self, port: u16) -> Self {
        self.port = Some(port);
        self
    }
    
    pub fn timeout(mut self, timeout: Duration) -> Self {
        self.timeout = Some(timeout);
        self
    }
    
    pub fn build(self) -> Result<Server, BuildError> {
        let host = self.host.unwrap_or_else(|| "localhost".to_string());
        let port = self.port.ok_or(BuildError::MissingPort)?;
        Ok(Server { host, port, timeout: self.timeout })
    }
}

// Usage
let server = ServerBuilder::new()
    .host("example.com")
    .port(8080)
    .timeout(Duration::from_secs(30))
    .build()?;
```

## Newtypes for Safety

```rust
// Type-safe IDs - can't mix up UserId and OrderId
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct UserId(u64);

#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
pub struct OrderId(u64);

fn get_user(id: UserId) -> User { /* ... */ }
fn get_order(id: OrderId) -> Order { /* ... */ }

// Compiler catches mistakes:
// get_user(OrderId(123));  // Compile error!

// Validated types
pub struct Email(String);

impl Email {
    pub fn new(s: String) -> Result<Self, ValidationError> {
        if s.contains('@') && s.contains('.') {
            Ok(Self(s))
        } else {
            Err(ValidationError::InvalidEmail)
        }
    }
    
    pub fn as_str(&self) -> &str {
        &self.0
    }
}

// Parse, don't validate - once parsed, always valid
pub fn parse_email(s: String) -> Result<Email, ValidationError> {
    Email::new(s)
}
```

## Typestate Pattern

```rust
// Compile-time state machine
pub struct Connection<S> {
    stream: TcpStream,
    _state: S,
}

pub struct Disconnected;
pub struct Connected;
pub struct Authenticated;

impl Connection<Disconnected> {
    pub fn connect(addr: &str) -> Result<Connection<Connected>, io::Error> {
        let stream = TcpStream::connect(addr)?;
        Ok(Connection { stream, _state: Connected })
    }
}

impl Connection<Connected> {
    pub fn authenticate(self, creds: Credentials) -> Result<Connection<Authenticated>, AuthError> {
        // auth logic
        Ok(Connection { stream: self.stream, _state: Authenticated })
    }
}

impl Connection<Authenticated> {
    pub fn query(&mut self, q: &str) -> Result<ResultSet, QueryError> {
        // Can only query when authenticated
    }
}

// Can't call query on disconnected connection - compile error!
```

## Sealed Traits

```rust
// Prevent external implementations
mod private {
    pub trait Sealed {}
}

pub trait Plugin: private::Sealed {
    fn name(&self) -> &str;
    fn execute(&self);
}

// Only we can implement Plugin
impl private::Sealed for MyPlugin {}
impl Plugin for MyPlugin { /* ... */ }
```

## Common Traits

```rust
// Implement these eagerly for public types
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
pub struct User {
    id: UserId,
    name: String,
}

// Default for sensible defaults
impl Default for Config {
    fn default() -> Self {
        Self {
            timeout: Duration::from_secs(30),
            retries: 3,
        }
    }
}
```

## Future-Proofing

```rust
// Non-exhaustive prevents breaking changes
#[non_exhaustive]
pub enum Status {
    Pending,
    Running,
    Completed,
    // Can add more variants later without breaking
}

// Users must handle _ variant
match status {
    Status::Pending => {},
    Status::Running => {},
    Status::Completed => {},
    _ => {},  // Required even if we list all
}

#[non_exhaustive]
pub struct Options {
    pub timeout: Duration,
    pub retries: u32,
    // Can add more fields later
}
```

## Serde Gate

```rust
// Gate serde behind feature for smaller compile times
#[derive(Debug, Clone)]
#[cfg_attr(feature = "serde", derive(serde::Serialize, serde::Deserialize))]
pub struct Config {
    pub name: String,
    pub value: i32,
}
```

## API Checklist

- [ ] String parameters → `impl Into<String>` or `impl AsRef<str>`
- [ ] Complex construction → Builder pattern with `#[must_use]`
- [ ] Distinct IDs/values → Newtypes
- [ ] State machine → Typestate pattern
- [ ] Public traits → Consider sealing
- [ ] Public types → `Debug`, `Clone`, `PartialEq`
- [ ] Breaking changes possible → `#[non_exhaustive]`
- [ ] Serde → Feature-gated
