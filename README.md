# Rust Crates Catalog: Working With Rust Using Crates or Without Them

A practical, opinionated catalog for understanding Rust in both directions:

- building with external crates from the Rust ecosystem
- building without crates at all
- deciding when each approach is the right tool for the job

This repository is intended to serve as a reference for Rust developers, learners, and teams who want to make thoughtful decisions about dependency use, project architecture, performance, safety, security, and maintainability.

## Why this catalog exists

Rust is a language of clarity, safety, and control. It has an excellent standard library, a rich ecosystem of reusable libraries, and a culture of building small, explicit, correct systems.

The central question is not:

- "Should I always use crates?"

or

- "Should I avoid crates entirely?"

The real question is:

- "What is the smallest and best design for this project?"

This catalog helps answer that question in a way that is both practical and engineering-minded.

---

## Table of contents

- [1. Rust in one sentence](#1-rust-in-one-sentence)
- [2. Crates vs no crates: the big picture](#2-crates-vs-no-crates-the-big-picture)
- [3. The Rust ecosystem](#3-the-rust-ecosystem)
- [4. The standard library first](#4-the-standard-library-first)
- [5. no_std and minimal Rust](#5-no_std-and-minimal-rust)
- [6. Cargo and dependency management](#6-cargo-and-dependency-management)
- [7. Decision framework: when to use crates](#7-decision-framework-when-to-use-crates)
- [8. Decision framework: when not to use crates](#8-decision-framework-when-not-to-use-crates)
- [9. Crates by category](#9-crates-by-category)
- [10. Standard library patterns worth knowing](#10-standard-library-patterns-worth-knowing)
- [11. Cargo workspace patterns](#11-cargo-workspace-patterns)
- [12. Dependency hygiene](#12-dependency-hygiene)
- [13. Performance and compile-time reality](#13-performance-and-compile-time-reality)
- [14. Security and maintenance considerations](#14-security-and-maintenance-considerations)
- [15. Real-world project archetypes](#15-real-world-project-archetypes)
- [16. Example code catalog](#16-example-code-catalog)
- [17. Comparison tables](#17-comparison-tables)
- [18. Learning roadmap](#18-learning-roadmap)
- [19. Glossary](#19-glossary)
- [20. Final recommendation](#20-final-recommendation)

---

## 1. Rust in one sentence

Rust is a systems programming language designed to give you memory safety, explicit control, and performance without sacrificing correctness.

It is used successfully for:

- command-line tools
- backend services
- data processing
- embedded systems
- networking
- game engines
- kernel, firmware, and low-level work

Rust's strengths are amplified by crates, but they are not required for every project.

---

## 2. Crates vs no crates: the big picture

### What is a crate?

A crate is a Rust package or library. In most cases, crates are distributed through crates.io and managed by Cargo.

A project may use:

- no external crates at all
- only the standard library (`std`)
- a few carefully selected dependencies
- a large dependency graph with many crates

### Quick comparison

| Approach | Strengths | Weaknesses | Best for |
| --- | --- | --- | --- |
| Standard library only | Simple, minimal, transparent, fast to build | More code to write manually | Tiny tools, custom logic, constrained targets |
| Small dependency set | Productive, mature libraries, faster delivery | Some maintenance and compatibility overhead | Most real-world apps |
| Large dependency ecosystem | Very fast development, broad capability | Build time, graph complexity, dependency risk | Complex services and frameworks |
| `no_std` minimal Rust | Very small footprint, low-level control | No `std` APIs, more specialized work | Embedded, bare-metal, kernels, bootloaders |

### Why the decision matters

Using crates can speed up development, but it can also:

- increase compile times
- add semver compatibility concerns
- bring transitive dependencies
- hide important behavior behind abstraction
- increase maintenance burden

Not using crates can be very clean and intentional, but it means:

- more code to maintain yourself
- duplicated logic
- more custom validation
- slower development for common tasks

The best choice is usually the one that keeps the project understandable and maintainable.

---

## 3. The Rust ecosystem

### Core pieces

- `rustc`: the Rust compiler
- `cargo`: build tool and package manager
- `crates.io`: the public registry for Rust libraries
- `Cargo.toml`: manifest for crate metadata and dependencies
- `Cargo.lock`: lock file for reproducible builds

### Dependency layers

Rust development is built around several layers of capability:

1. `core`
   - the core language primitives
2. `alloc`
   - heap allocation support
3. `std`
   - the standard library for normal environments
4. external crates
   - community-published reusable code

### Typical crate categories

- CLI tools: `clap`, `argh`, `dialoguer`
- Serialization: `serde`, `serde_json`, `toml`, `bincode`
- HTTP and networking: `reqwest`, `hyper`, `tokio`, `axum`
- Logging: `tracing`, `log`, `env_logger`
- Error handling: `anyhow`, `thiserror`, `eyre`
- Database: `sqlx`, `diesel`, `rusqlite`
- Async: `tokio`, `async-std`, `futures`
- Embedded / `no_std`: `embedded-hal`, `heapless`, `nb`, `defmt`
- Testing: `criterion`, `proptest`, `insta`

### The ecosystem philosophy

Rust's ecosystem favors:

- explicitness
- testability
- composability
- stable APIs
- careful versioning

That is one reason the ecosystem is so strong for production systems.

---

## 4. The standard library first

The Rust standard library is often overlooked because developers reach for crates too quickly. But `std` is already very capable.

### Good things in `std`

- `Vec`, `HashMap`, `HashSet`, `BTreeMap`
- `String`, `str`, `Path`, `PathBuf`
- file system tooling: `std::fs`
- networking: `std::net`
- threads: `std::thread`
- synchronization: `Arc`, `Mutex`, `RwLock`
- I/O traits: `Read`, `Write`, `BufRead`
- time and duration: `std::time`

### When `std` is ideal

- tiny CLI utilities
- internal scripts
- local file processing
- straightforward data transformation
- logic that does not need network or async stacks
- custom algorithms that are better kept local

### Example: a simple CLI without external crates

```rust
use std::env;
use std::fs;

fn main() {
    let args: Vec<String> = env::args().collect();

    if args.len() < 2 {
        eprintln!("usage: app <path>");
        std::process::exit(1);
    }

    let content = fs::read_to_string(&args[1]).expect("failed to read file");
    println!("Read {} bytes", content.len());
}
```

This is fast, explicit, and dependency-free.

### The golden rule

Do not reach for a crate until you know the standard library is insufficient or awkward for your case.

---

## 5. no_std and minimal Rust

`no_std` is a key Rust concept that many developers should understand, especially when building all kinds of low-level or embedded software.

### What does `no_std` mean?

A crate annotated with `#![no_std]` does not use the standard library. It uses the core library instead and must often define its own runtime assumptions.

### Typical use cases

- embedded software
- bare-metal systems
- firmware
- bootloaders
- kernel-like code
- tiny runtime environments
- highly constrained devices

### Example: minimal `no_std` library

```rust
#![no_std]

pub fn add(a: u32, b: u32) -> u32 {
    a + b
}
```

### Why `no_std` matters

Because in constrained systems, the standard library may be unavailable or too heavy. Rust still provides useful lower-level abstractions for those environments.

### `core`, `alloc`, and `std`

- `core`: mandatory low-level primitives
- `alloc`: heap allocation support when available
- `std`: full runtime environment for normal apps

### Benefits

- smaller binaries
- more control
- less runtime dependency
- common in embedded environments

### Trade-offs

- no filesystem APIs by default
- no threading as usual
- no standard environment assumptions
- requires deeper understanding of runtime behavior

### When to prefer `no_std`

- you are writing firmware or embedded code
- you are targeting microcontrollers
- you need a tiny-core library
- a custom platform environment is required

---

## 6. Cargo and dependency management

Rust's package manager and build system is called Cargo.

### Common commands

```bash
cargo new my_project
cargo build
cargo run
cargo test
cargo check
cargo fmt
cargo clippy
cargo doc --open
cargo update
```

### `Cargo.toml` basics

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = { version = "1", features = ["derive"] }
reqwest = "0.12"

[dev-dependencies]
pretty_assertions = "1"
```

### Dependency concepts

- normal dependencies: used by the program
- dev-dependencies: used only for tests or tooling
- features: optional functionality enabled selectively
- workspace: a multi-crate project managed together
- lock file: reproducible dependency resolution

### Features example

```toml
[dependencies]
serde = { version = "1", features = ["derive"] }
json = { version = "0.12", optional = true }

[features]
default = ["json"]
```

### Best practices

- keep dependencies minimal
- avoid enabling unnecessary features
- check crate maintenance and activity
- prefer stable, mature crates with clear semver usage
- use `cargo check` often during editing
- keep `Cargo.lock` for applications and service binaries

---

## 7. Decision framework: when to use crates

Use a crate when the problem is common, nontrivial, or already solved well by the ecosystem.

### Good reasons to use a crate

- parsing JSON, TOML, YAML, or CSV
- argument parsing for CLI apps
- HTTP requests and API clients
- async runtimes
- database access
- web frameworks
- structured logging
- tracing and metrics
- robust error handling

### Typical examples

- `serde` + `serde_json` for JSON
- `clap` for CLI
- `reqwest` for HTTP
- `tokio` for async
- `tracing` for observability
- `sqlx` or `diesel` for DB work
- `axum` for HTTP APIs

### Signs you should consider a crate

- the feature is standard and well-understood
- writing it manually would take considerable design effort
- the crate is mature and has active maintenance
- the code would be hard to validate without the crate's proven behavior
- you want an established pattern used in production systems

### Example: JSON handling

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct User {
    name: String,
    age: u32,
}
```

This is much easier and safer than writing a custom serializer.

---

## 8. Decision framework: when not to use crates

Avoid external crates when the solution is small, specific, or best kept local.

### Good reasons not to use a crate

- the logic is tiny and custom
- the requirement is narrow and easy to implement directly
- you want full explicit control
- your target environment is minimal or embedded
- dependency bloat would hurt compile performance or maintainability
- you're teaching, prototyping, or experimenting

### Common examples of no-crate code

- custom file format parsing
- a tiny binary utility
- internal algorithm implementation
- strongly bounded business logic
- library code with very few moving parts

### Rule of thumb

If a crate only saves a few lines but adds complexity or unnecessary dependencies, keep it local.

---

## 9. Crates by category

### 9.1 CLI and argument parsing

| Crate | Purpose | When to use |
| --- | --- | --- |
| `clap` | Command-line interfaces | Most CLI tools |
| `argh` | Attribute-based argument parsing | Simple, annotation-driven CLIs |
| `dialoguer` | Interactive prompts | Terminal UIs |

### 9.2 Serialization and data formats

| Crate | Purpose | When to use |
| --- | --- | --- |
| `serde` | Serialization abstraction | Almost any structured data |
| `serde_json` | JSON support | APIs, configs, payloads |
| `toml` | TOML reading/writing | Config files |
| `bincode` | Fast binary format | Compact storage or IPC |
| `ron` | Rust Object Notation | Human-readable Rust-like config |

### 9.3 HTTP, networking, and APIs

| Crate | Purpose | When to use |
| --- | --- | --- |
| `reqwest` | HTTP client | REST APIs, downloads |
| `hyper` | HTTP server/client primitives | High-performance HTTP stacks |
| `tokio` | Async runtime | Web apps, non-blocking I/O |
| `axum` | Web framework | API backends |
| `tower` | Service abstraction | Middleware and service composition |

### 9.4 Logging and observability

| Crate | Purpose | When to use |
| --- | --- | --- |
| `tracing` | Structured logging / instrumentation | Production systems |
| `log` | Simple logging facade | Broad ecosystem use |
| `env_logger` | Simple logger | Quick local setup |

### 9.5 Error handling

| Crate | Purpose | When to use |
| --- | --- | --- |
| `thiserror` | Typed error types | Domain-specific error modeling |
| `anyhow` | Ergonomic error propagation | Application-level code |
| `eyre` | Rich error reports | Troubleshooting and CLI work |

### 9.6 Database and storage

| Crate | Purpose | When to use |
| --- | --- | --- |
| `sqlx` | Async DB access | Postgres, MySQL, SQLite |
| `diesel` | ORM/query builder | Strongly typed database work |
| `rusqlite` | SQLite driver | Local relational storage |

### 9.7 Async and concurrency

| Crate | Purpose | When to use |
| --- | --- | --- |
| `tokio` | Runtime and async ecosystem | Networked services |
| `async-std` | Alternative async runtime | Simpler async apps |
| `futures` | Futures utilities | Async composition |

### 9.8 Embedded and low-level

| Crate | Purpose | When to use |
| --- | --- | --- |
| `embedded-hal` | Hardware abstraction layer | Embedded systems |
| `heapless` | No-heap containers | Resource-constrained code |
| `nb` | Non-blocking operations | Embedded I/O flow |
| `defmt` | Logging in embedded | Microcontroller debugging |

### 9.9 Testing and benchmarking

| Crate | Purpose | When to use |
| --- | --- | --- |
| `criterion` | Performance benchmarking | Measuring improvements |
| `proptest` | Property-based testing | Edge-case validation |
| `insta` | Snapshot testing | CLI output or serialization tests |

---

## 10. Standard library patterns worth knowing

These patterns are common and useful in Rust without adding external dependencies.

### 10.1 Working with files

```rust
use std::fs;

let contents = fs::read_to_string("data.txt")?;
```

### 10.2 Command-line arguments

```rust
use std::env;

let args: Vec<String> = env::args().collect();
```

### 10.3 Strings and parsing

```rust
let number: i32 = "42".parse().unwrap();
```

### 10.4 Collections

```rust
let mut items = Vec::new();
items.push(1);
items.push(2);

let mut map = std::collections::HashMap::new();
map.insert("a", 1);
```

### 10.5 Error handling

```rust
fn read_number() -> Result<i32, std::num::ParseIntError> {
    let s = "42";
    s.parse::<i32>()
}
```

### 10.6 Custom parsing patterns

```rust
fn parse_csv_row(line: &str) -> Vec<&str> {
    line.split(',').collect()
}
```

This is often cleaner than dragging in a parsing dependency when the format is simple and domain-specific.

---

## 11. Cargo workspace patterns

Rust workspaces help organize multi-crate projects.

### Single-crate project

```text
my_app/
├── Cargo.toml
├── src/
│   └── main.rs
└── README.md
```

### Multi-crate workspace

```text
workspace/
├── Cargo.toml
├── crates/
│   ├── app/
│   ├── core/
│   ├── api/
│   └── cli/
└── README.md
```

### Example root `Cargo.toml`

```toml
[workspace]
members = [
  "crates/app",
  "crates/core",
  "crates/api",
  "crates/cli",
]
```

### Why workspaces are useful

- separation of concerns
- shared internal libraries
- easier testing and organization
- better code ownership in larger teams

### Good workspace style

- keep application code and domain logic separate
- keep shared logic in a core crate
- avoid monolithic binaries unless project size warrants it

---

## 12. Dependency hygiene

Adding crates is not free, even when worthwhile.

### Good dependency hygiene rules

- prefer the minimum number of dependencies
- prefer small targeted crates over giant frameworks
- only enable the features you need
- audit transitive dependencies
- prefer maintained crates with active maintainers
- avoid crates that are over-engineered for your task

### Signs of a bad dependency choice

- it solves a problem you barely need
- it pulls in lots of unrelated functionality
- it has poor or sparse docs
- it is barely maintained
- it adds a large abstraction layer for a small task

### Example of a good choice

Using `serde` and `serde_json` for JSON handling is idiomatic and standard. This is a mature, well-supported dependency choice.

### Example of a questionable case

Pulling in a huge framework for a tiny internal utility when a few lines of `std` code solve it more directly.

---

## 13. Performance and compile-time reality

One of the biggest concerns with crates is compile time.

### Crates can hurt compile performance

Large dependency graphs can mean:

- longer build times
- slower CI pipelines
- more time for every change
- harder repros in constrained environments

### A good balance

- use `cargo check` for fast feedback
- build full binaries only when needed
- prefer focused dependencies
- keep your project separate from huge ecosystems when unnecessary

### Performance is not just runtime speed

In Rust, performance also includes:

- compile time
- cognitive load
- maintainability
- dependency churn
- team velocity

### Runtime performance vs. dependency cost

A crate may offer runtime speed, but the cost of complexity and compile time could outweigh the benefit for a small project.

---

## 14. Security and maintenance considerations

Dependency use is partly a security and governance decision.

### Security checklist

- prefer crates with active maintenance
- review direct and transitive dependencies
- check release history and compatibility
- prefer stable, widely used crates where possible
- be cautious with heavy, unmaintained libraries

### Maintenance signals

Look for:

- recent releases
- active issue resolution
- responsive maintainers
- clear documentation
- healthy community adoption

### Why this matters

In production software, dependency health is part of the long-term safety of the system. A crate can become a liability if it is not maintained.

---

## 15. Real-world project archetypes

### 15.1 Tiny CLI utility

Project type:

- reads a file
- transforms data
- prints output

Recommended approach:

- no external crates or only one tiny dependency
- standard library first

Example:

```rust
use std::fs;

fn main() {
    let content = fs::read_to_string("input.txt").unwrap();
    println!("{}", content.trim());
}
```

### 15.2 REST API service

Project type:

- HTTP endpoints
- JSON request/response
- database access
- async processing

Recommended approach:

- `axum` or `actix-web`
- `tokio`
- `serde` and `serde_json`
- `sqlx` or `diesel`

### 15.3 Embedded system

Project type:

- microcontroller
- firmware
- sensor hardware
- bare metal or custom runtime

Recommended approach:

- `no_std`
- `embedded-hal`
- heapless containers
- target-specific abstraction crates

### 15.4 Data-processing pipeline

Project type:

- CSV or log processing
- local ETL jobs
- file ingestion

Recommended approach:

- standard library for smaller jobs
- selected crates for parsing or concurrency when needed

### 15.5 Internal reusable library

Project type:

- domain logic
- shared algorithms
- service semantics

Recommended approach:

- keep dependencies minimal
- expose clean APIs
- prefer local implementation over a broad framework

---

## 16. Example code catalog

### 16.1 Standard library-only CLI

```rust
use std::env;
use std::fs;

fn main() {
    let args: Vec<String> = env::args().collect();
    if args.len() < 2 {
        eprintln!("usage: app <file>");
        std::process::exit(1);
    }

    let data = fs::read_to_string(&args[1]).expect("unable to read file");
    println!("Length: {}", data.len());
}
```

### 16.2 CLI with `clap`

```rust
use clap::Parser;

#[derive(Parser, Debug)]
#[command(name = "demo")]
struct Args {
    #[arg(long)]
    name: String,
}

fn main() {
    let args = Args::parse();
    println!("Hello {}!", args.name);
}
```

### 16.3 JSON with `serde`

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Person {
    name: String,
    age: u32,
}

fn main() {
    let p = Person { name: "Alice".to_string(), age: 32 };
    let json = serde_json::to_string(&p).unwrap();
    println!("{}", json);
}
```

### 16.4 Async server with `tokio` and `axum`

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

### 16.5 Custom parser without external crates

```rust
fn parse_numbers(input: &str) -> Vec<i32> {
    input
        .split(',')
        .filter_map(|token| {
            let trimmed = token.trim();
            if trimmed.is_empty() { None } else { Some(trimmed.parse::<i32>().unwrap()) }
        })
        .collect()
}
```

### 16.6 `no_std` library

```rust
#![no_std]

pub fn square(x: u32) -> u32 {
    x * x
}
```

---

## 17. Comparison tables

### 17.1 `std` vs external crates

| Topic | `std`-only approach | External crates |
| --- | --- | --- |
| Simplicity | High | Medium to low |
| Build complexity | Low | Higher |
| Dependency management | None | Required |
| Time to implement | More manual work | Faster |
| Flexibility | Very high | High but layered |
| Best use | Small tools, simple logic | Common infrastructure and ecosystems |

### 17.2 `std` vs `no_std`

| Topic | `std` | `no_std` |
| --- | --- | --- |
| Filesystem | Yes | Usually no |
| Threads | Yes | Usually no |
| Embedded support | Limited | Strong |
| Binary size | Larger | Smaller |
| Runtime assumptions | Normal OS environment | Custom environment |

### 17.3 Crate-first vs custom implementation

| Scenario | Prefer crate | Prefer custom implementation |
| --- | --- | --- |
| JSON serialization | Yes | Sometimes not |
| HTTP client | Yes | No |
| CLI parser | Yes | If extremely tiny |
| custom config format | Sometimes | Often yes |
| business logic | Maybe | Often yes |
| embedded runtime | Sometimes | Often yes |

---

## 18. Learning roadmap

### Beginner path

1. Learn variables, ownership, borrowing, and lifetimes.
2. Write a few small programs with `std` only.
3. Master `Result`, `Option`, and basic error handling.
4. Learn `cargo build`, `cargo run`, and `cargo test`.
5. Add one or two crates intentionally, like `clap` or `serde`.
6. Learn how `Cargo.toml` and feature flags work.

### Intermediate path

1. Understand workspaces and library layouts.
2. Learn async with `tokio`.
3. Build a small API server with `axum`.
4. Use structured logging and tracing.
5. Learn dependency hygiene and crate auditing.

### Advanced path

1. Work with `no_std` and embedded targets.
2. Build reusable libraries with careful API design.
3. Keep compile times under control.
4. Design long-lived error and data models.
5. Understand semver, maintenance, and ecosystem maturity.

---

## 19. Glossary

- crate: a Rust package or library
- Cargo: Rust's package manager and build tool
- `std`: the standard library for common environments
- `core`: language and primitive support without OS assumptions
- `alloc`: heap and allocation support
- `no_std`: Rust without the standard library
- feature flag: optional functionality inside a crate
- workspace: multiple Rust crates managed as one project
- semver: semantic versioning
- lockfile: `Cargo.lock`, which pins dependency versions
- transitive dependency: a dependency that is required by another dependency
- async: non-blocking execution model

---

## 20. Final recommendation

The strongest Rust engineering practice is not to choose a side dogmatically.

The best approach is to balance three things:

- correctness
- productivity
- simplicity

### The practical rule

- Start with the standard library.
- Use crates when they solve a real, mature, recurring problem.
- Avoid dependencies if the task is small, specific, or easy to do correctly by hand.
- Use `no_std` when the environment demands it.
- Keep the dependency graph honest and minimal.

### Final short guidance

Use crates for:

- common infrastructure
- networked systems
- serialization
- async runtime
- APIs
- logging and error handling

Use no-crate / `std`-only code for:

- small utilities
- custom logic
- domain-specific implementation
- constrained or embedded systems

In other words:

A good Rust project is usually a disciplined combination of local, explicit code and carefully chosen external crates.

That is the real Rust philosophy: not more dependencies, but the right dependencies at the right time.

---

## Quick decision guide

If the task is:

- a tiny script -> `std`
- a custom data parser -> `std` or tiny custom code
- a service with HTTP and async -> `tokio` + framework crates
- a CLI with commands -> `clap`
- JSON data -> `serde` + `serde_json`
- a firmware target -> `no_std` + embedded crates
- a library with domain logic -> minimal internal logic, few external crates

---

## Closing note

This repository is intended as a practical catalog and reference, not just a tutorial. The goal is to help developers understand the trade-offs between:

- writing direct Rust code
- leaning on the standard library
- depending on crates for speed and ergonomics
- working in constrained or low-level environments

Rust is at its best when you know why a dependency exists and why it does not.

---

## Optional extension ideas for the repo

If you want to go beyond the README, this project can also evolve into:

- a curated crate catalog by project type
- a comparison matrix of common Rust crates
- a `docs/` section with real code samples
- workspace templates for binary, library, and `no_std` projects
- a best-practices guide for dependency governance

This README is a strong foundation for that expansion.
