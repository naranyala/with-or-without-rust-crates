# Crate Catalog

This catalog preserves the richness of the ecosystem without forcing every project to adopt every crate. It is meant to show the important crate families and the most representative names in each area.

## 1. Core crate families

### CLI and argument parsing

| Crate | Purpose | Typical use |
| --- | --- | --- |
| `clap` | command-line interfaces | real CLIs |
| `argh` | attribute-based args | simple CLIs |
| `dialoguer` | interactive prompts | terminal UIs |

### Serialization and data formats

| Crate | Purpose | Typical use |
| --- | --- | --- |
| `serde` | serialization abstraction | almost any data model |
| `serde_json` | JSON support | APIs and config |
| `toml` | TOML parsing | config files |
| `bincode` | compact binary format | storage and IPC |
| `ron` | Rust Object Notation | readable Rust-like config |
| `csv` | CSV parsing | tabular imports and exports |

### HTTP, networking, and APIs

| Crate | Purpose | Typical use |
| --- | --- | --- |
| `reqwest` | HTTP client | REST APIs and downloads |
| `hyper` | HTTP primitives | high-performance network stacks |
| `tokio` | async runtime | async service work |
| `axum` | web framework | API backends |
| `tower` | service abstraction | middleware and service composition |
| `rustls` | TLS support | secure network connections |
| `quinn` | QUIC networking | modern transport support |
| `tokio-tungstenite` | WebSockets | async real-time comms |

### Logging and observability

| Crate | Purpose | Typical use |
| --- | --- | --- |
| `tracing` | structured logging | production systems |
| `log` | simple logging facade | broad ecosystem use |
| `env_logger` | quick logger | local setup |
| `colored` | terminal color | CLI UX |
| `indicatif` | progress bars | long-running CLI tasks |

### Error handling

| Crate | Purpose | Typical use |
| --- | --- | --- |
| `thiserror` | typed errors | libraries and domain models |
| `anyhow` | ergonomic propagation | app-level code |
| `eyre` | rich error reports | debugging and CLI work |

### Database and storage

| Crate | Purpose | Typical use |
| --- | --- | --- |
| `sqlx` | async DB access | Postgres, MySQL, SQLite |
| `diesel` | ORM/query builder | strongly typed DB work |
| `rusqlite` | SQLite driver | local storage |

### Async and concurrency

| Crate | Purpose | Typical use |
| --- | --- | --- |
| `tokio` | async runtime | networked services |
| `async-std` | alternative async runtime | simpler async apps |
| `futures` | async utilities | async composition |
| `rayon` | data parallelism | CPU-bound parallel tasks |
| `crossbeam` | concurrency primitives | advanced multi-threading |

### Embedded and low-level

| Crate | Purpose | Typical use |
| --- | --- | --- |
| `embedded-hal` | hardware abstraction | embedded systems |
| `heapless` | no-heap containers | resource-constrained code |
| `nb` | non-blocking ops | embedded I/O |
| `defmt` | embedded logging | microcontroller debugging |
| `cortex-m` | Cortex-M support | ARM embedded work |

### Testing and benchmarking

| Crate | Purpose | Typical use |
| --- | --- | --- |
| `criterion` | performance benchmarking | measuring improvements |
| `proptest` | property-based testing | edge-case validation |
| `insta` | snapshot testing | output regression tests |
| `pretty_assertions` | readable assert diffs | developer-friendly tests |

### Cryptography and security

| Crate | Purpose | Typical use |
| --- | --- | --- |
| `ring` | secure primitives | TLS and hashing |
| `sha2` | hashing | integrity checks |
| `blake3` | fast hashing | high-throughput workloads |
| `aes-gcm` | symmetric encryption | payload encryption |
| `argon2` | password hashing | auth systems |
| `ed25519-dalek` | signatures | identity and signing |

### Platform-specific and system integration

| Crate | Purpose | Typical use |
| --- | --- | --- |
| `windows` | Windows API | system integration |
| `windows-sys` | low-level bindings | Win32 precision |
| `nix` | Unix APIs | Linux/macOS system programming |
| `libc` | C ABI bindings | OS interop |
| `which` | PATH lookup | CLI helpers |
| `directories` | standard OS directories | config/cache/data paths |
| `signal-hook` | signal handling | daemon and process lifecycle |
| `walkdir` | recursive traversal | file tooling |
| `glob` | file globbing | pattern-based file discovery |
| `notify` | file change watching | rebuild triggers |
| `tempfile` | temporary files | tests and sandboxing |

### Browser and WASM

| Crate | Purpose | Typical use |
| --- | --- | --- |
| `wasm-bindgen` | JS interop | WASM apps |
| `web-sys` | browser APIs | browser features |
| `js-sys` | JS interop primitives | low-level browser integration |
| `gloo` | browser utilities | convenience web tooling |
| `console_error_panic_hook` | browser panic output | debugging in browsers |

### GUI and desktop

| Crate | Purpose | Typical use |
| --- | --- | --- |
| `egui` | immediate-mode GUI | lightweight UIs |
| `iced` | Rust-native GUI | desktop apps |
| `tauri` | desktop app with web frontend | hybrid desktop apps |
| `slint` | declarative UI | polished UI systems |

## 2. Practical pairings

These are frequent, healthy combinations in real projects:

- `clap` + `serde` + `serde_json` for CLIs that read or write JSON
- `axum` + `tokio` + `tracing` for API services
- `sqlx` + `serde` + `tokio` for database-backed services
- `thiserror` + `anyhow` for structured application code
- `windows` or `nix` when target-specific OS integration matters
- `embedded-hal` + `heapless` + `defmt` for constrained hardware targets

## 3. When a crate is a good fit

A crate is worth it when:

- the problem is narrow but real
- the crate solves it much better than custom code
- the crate is mature and maintained
- the problem is common enough that reimplementation is costly or error-prone

## 4. When a crate is not worth it

A crate is a poor fit when:

- it only saves a few lines but adds a large abstraction layer
- the problem is tiny and custom
- the team is better served by explicit local logic
- the dependency graph becomes far larger than the functionality

## 5. Quick crate examples

### Example: CLI

```rust
use clap::Parser;

#[derive(Parser, Debug)]
struct Args {
    #[arg(long)]
    name: String,
}

fn main() {
    let args = Args::parse();
    println!("Hello {}!", args.name);
}
```

### Example: JSON

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Person {
    name: String,
    age: u32,
}
```

### Example: async server

```rust
use axum::{routing::get, Router};

#[tokio::main]
async fn main() {
    let app = Router::new().route("/", get(|| async { "hello" }));
    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000")
        .await
        .unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

### Example: embedded

```rust
#![no_std]

pub fn square(x: u32) -> u32 {
    x * x
}
```

## 6. Bottom line

Rust’s ecosystem is powerful because it combines broad, general-purpose libraries with highly specific tools for OS, platform, protocol, and domain constraints.

The crate catalog is not about “using everything.” It is about choosing the right dependency for the actual problem.
