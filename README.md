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
- [9. Popular Rust crates and no-crate equivalents](#9-popular-rust-crates-and-no-crate-equivalents)
- [10. Crates by category](#10-crates-by-category)
- [11. Standard library patterns worth knowing](#11-standard-library-patterns-worth-knowing)
- [12. Cargo workspace patterns](#12-cargo-workspace-patterns)
- [13. Dependency hygiene](#13-dependency-hygiene)
- [14. Performance and compile-time reality](#14-performance-and-compile-time-reality)
- [15. Security and maintenance considerations](#15-security-and-maintenance-considerations)
- [16. Real-world project archetypes](#16-real-world-project-archetypes)
- [17. Example code catalog](#17-example-code-catalog)
- [18. Comparison tables](#18-comparison-tables)
- [19. Learning roadmap](#19-learning-roadmap)
- [20. Glossary](#20-glossary)
- [21. Final recommendation](#21-final-recommendation)
- [22. Intuitive and life-saver crates](#22-intuitive-and-life-saver-crates)
- [23. Scoped and platform-specific crates](#23-scoped-and-platform-specific-crates)

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

## 9. Popular Rust crates and no-crate equivalents

This section is designed to answer the practical question:

- "What does the crate buy me?"
- "What does the manual/no-crate option look like?"
- "When does the hand-written version still win?"

The best Rust decisions are not based on dogma. They are based on trade-offs.

### 9.1 CLI argument parsing

- Crate: `clap`
- No-crate option: `std::env::args()` and manual parsing
- What the crate gives you:
  - subcommands
  - typed option parsing
  - validation
  - help generation
  - cleaner UX for real CLIs
- Manual option:
  - ideal for tiny utilities
  - better if there are only a few flags or positional args
- Best choice:
  - use `clap` for real CLI tools
  - use manual parsing for scripts and one-off programs

Example manual parsing:

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    if args.len() < 2 {
        eprintln!("usage: app <name>");
        return;
    }

    println!("Hello {}!", args[1]);
}
```

### 9.2 JSON handling

- Crate: `serde` + `serde_json`
- No-crate option: manual parsing with `String`, `split`, and custom structs
- What the crate gives you:
  - structured data modeling
  - strong serialization support
  - less custom parsing logic
  - better interoperability with JSON APIs
- Manual option:
  - viable for tiny and stable JSON payloads
  - useful when the format is tightly known and simple
- Best choice:
  - use crates for real production JSON
  - use custom parsing only for constrained or extremely simple data

Example custom parser:

```rust
fn parse_simple_json_name(input: &str) -> Option<&str> {
    let marker = "\"name\":\"";
    input.find(marker).map(|start| {
        let after = &input[start + marker.len()..];
        let end = after.find('"').unwrap_or(after.len());
        &after[..end]
    })
}
```

### 9.3 HTTP client

- Crate: `reqwest`
- No-crate option: `std::net::TcpStream` plus manual HTTP request formatting
- What the crate gives you:
  - request/response abstractions
  - TLS support
  - body handling
  - retries and API ergonomics
- Manual option:
  - useful for extremely small custom protocol work
  - educational or low-level implementations
- Best choice:
  - choose `reqwest` for standard HTTP clients
  - use raw networking only in constrained or custom protocol scenarios

Example in principle:

```rust
use std::net::TcpStream;

let mut stream = TcpStream::connect("example.com:80").unwrap();
```

This gives you some control, but it is far more difficult than `reqwest` for real network APIs.

### 9.4 Web server

- Crate: `axum`, `actix-web`, or `rocket`
- No-crate option: `std::net::TcpListener` + manual request parsing
- What the crate gives you:
  - routing
  - middleware
  - request/response type handling
  - ergonomic API design
- Manual option:
  - good for educational projects
  - useful in very constrained systems
  - okay for tiny protocol servers
- Best choice:
  - use a framework for real web services
  - use manual parsing for minimal custom protocol servers

### 9.5 Async runtime

- Crate: `tokio`
- No-crate option: `std::thread` and manual scheduling / channels
- What the crate gives you:
  - asynchronous tasks
  - non-blocking I/O
  - timers, joins, channels, and actor-like patterns
- Manual option:
  - enough for a handful of threads
  - straightforward when the concurrency model is simple
- Best choice:
  - use `tokio` for network-heavy or asynchronous workloads
  - use threads when concurrency is simple and explicit

### 9.6 Logging

- Crate: `tracing` or `env_logger`
- No-crate option: `println!`, `eprintln!`, or a simple local logger module
- What the crate gives you:
  - structured logs
  - levels and filtering
  - tracing spans and context
  - scalable observability
- Manual option:
  - sufficient for tiny tools
  - often easiest for scripts or prototypes
- Best choice:
  - use logging crates for production systems
  - use manual logging for small command-line tools

### 9.7 Error handling

- Crate: `thiserror` or `anyhow`
- No-crate option: custom `enum` error types and `Result<T, MyError>`
- What the crate gives you:
  - ergonomic error conversion
  - typed domain errors
  - less boilerplate in production code
- Manual option:
  - great for library code where explicit error semantics matter
  - often cleaner in small projects
- Best choice:
  - use `thiserror` or custom errors for libraries and explicit APIs
  - use `anyhow` for app-level code where ergonomics matter more than strict typing

Example manual error model:

```rust
#[derive(Debug)]
enum AppError {
    Io(std::io::Error),
    Parse(std::num::ParseIntError),
}
```

### 9.8 Configuration parsing

- Crate: `toml` + `serde`
- No-crate option: read files manually and parse key/value pairs
- What the crate gives you:
  - typed config objects
  - validation
  - less fragile parsing logic
- Manual option:
  - enough for very simple configuration files
  - good when the format is rigid and tiny
- Best choice:
  - use crates for complex configuration
  - use manual parsing for simple config files or toy projects

### 9.9 Database access

- Crate: `sqlx` or `diesel`
- No-crate option: direct file storage or a custom serialization layer
- What the crate gives you:
  - connection management
  - query execution
  - safety and type checking
  - larger ecosystem integration
- Manual option:
  - adequate for simple local file-based storage
  - often good for embedded, tiny apps, or very controlled data stores
- Best choice:
  - use database crates for real database workloads
  - use custom storage when the data model is simple and storage is local

### 9.10 Binary serialization and compact formats

- Crate: `bincode`
- No-crate option: manual byte packing and parsing
- What the crate gives you:
  - compact binary encoding/decoding
  - less error-prone implementation
- Manual option:
  - useful in tightly controlled, resource-limited systems
  - common in embedded or custom protocol work
- Best choice:
  - choose a crate for normal binary serialization
  - choose manual binary packing for special low-level or minimal environments

### 9.11 Regex and text processing

- Crate: `regex`
- No-crate option: manual string scanning, splitting, and matching
- What the crate gives you:
  - expressive text matching
  - better safety and maintainability
- Manual option:
  - fine when patterns are simple and local
  - helps avoid dragging in extra dependencies in tiny tools
- Best choice:
  - use `regex` when the text logic is nontrivial
  - use manual logic for simple and controlled patterns

### 9.12 Timer and scheduling logic

- Crate: `tokio` or `chrono`
- No-crate option: use `std::time` and loops or threads
- What the crate gives you:
  - robust scheduling and time operations
  - easier async coordination
- Manual option:
  - acceptable for simple scripts or small loops
- Best choice:
  - use crate-based time and scheduler support for production systems
  - use `std::time` for simple delay logic

### Summary pattern

The rule is not “crates are always better” or “manual is always better.”

The real rule is:

- choose the crate when the problem is general, shared, stable, and well-solved
- choose the no-crate version when the problem is small, simple, controlled, or special

---

## 10. Crates by category

### 10.1 CLI and argument parsing

| Crate | Purpose | When to use |
| --- | --- | --- |
| `clap` | Command-line interfaces | Most CLI tools |
| `argh` | Attribute-based argument parsing | Simple, annotation-driven CLIs |
| `dialoguer` | Interactive prompts | Terminal UIs |

### 10.2 Serialization and data formats

| Crate | Purpose | When to use |
| --- | --- | --- |
| `serde` | Serialization abstraction | Almost any structured data |
| `serde_json` | JSON support | APIs, configs, payloads |
| `toml` | TOML reading/writing | Config files |
| `bincode` | Fast binary format | Compact storage or IPC |
| `ron` | Rust Object Notation | Human-readable Rust-like config |

### 10.3 HTTP, networking, and APIs

| Crate | Purpose | When to use |
| --- | --- | --- |
| `reqwest` | HTTP client | REST APIs, downloads |
| `hyper` | HTTP server/client primitives | High-performance HTTP stacks |
| `tokio` | Async runtime | Web apps, non-blocking I/O |
| `axum` | Web framework | API backends |
| `tower` | Service abstraction | Middleware and service composition |

### 10.4 Logging and observability

| Crate | Purpose | When to use |
| --- | --- | --- |
| `tracing` | Structured logging / instrumentation | Production systems |
| `log` | Simple logging facade | Broad ecosystem use |
| `env_logger` | Simple logger | Quick local setup |

### 10.5 Error handling

| Crate | Purpose | When to use |
| --- | --- | --- |
| `thiserror` | Typed error types | Domain-specific error modeling |
| `anyhow` | Ergonomic error propagation | Application-level code |
| `eyre` | Rich error reports | Troubleshooting and CLI work |

### 10.6 Database and storage

| Crate | Purpose | When to use |
| --- | --- | --- |
| `sqlx` | Async DB access | Postgres, MySQL, SQLite |
| `diesel` | ORM/query builder | Strongly typed database work |
| `rusqlite` | SQLite driver | Local relational storage |

### 10.7 Async and concurrency

| Crate | Purpose | When to use |
| --- | --- | --- |
| `tokio` | Runtime and async ecosystem | Networked services |
| `async-std` | Alternative async runtime | Simpler async apps |
| `futures` | Futures utilities | Async composition |

### 10.8 Embedded and low-level

| Crate | Purpose | When to use |
| --- | --- | --- |
| `embedded-hal` | Hardware abstraction layer | Embedded systems |
| `heapless` | No-heap containers | Resource-constrained code |
| `nb` | Non-blocking operations | Embedded I/O flow |
| `defmt` | Logging in embedded | Microcontroller debugging |

### 10.9 Testing and benchmarking

| Crate | Purpose | When to use |
| --- | --- | --- |
| `criterion` | Performance benchmarking | Measuring improvements |
| `proptest` | Property-based testing | Edge-case validation |
| `insta` | Snapshot testing | CLI output or serialization tests |

---

## 11. Standard library patterns worth knowing

These patterns are common and useful in Rust without adding external dependencies.

### 11.1 Working with files

```rust
use std::fs;

let contents = fs::read_to_string("data.txt")?;
```

### 11.2 Command-line arguments

```rust
use std::env;

let args: Vec<String> = env::args().collect();
```

### 11.3 Strings and parsing

```rust
let number: i32 = "42".parse().unwrap();
```

### 11.4 Collections

```rust
let mut items = Vec::new();
items.push(1);
items.push(2);

let mut map = std::collections::HashMap::new();
map.insert("a", 1);
```

### 11.5 Error handling

```rust
fn read_number() -> Result<i32, std::num::ParseIntError> {
    let s = "42";
    s.parse::<i32>()
}
```

### 11.6 Custom parsing patterns

```rust
fn parse_csv_row(line: &str) -> Vec<&str> {
    line.split(',').collect()
}
```

This is often cleaner than dragging in a parsing dependency when the format is simple and domain-specific.

---

## 12. Cargo workspace patterns

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

## 13. Dependency hygiene

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

### Crate anti-patterns: when a dependency is easy to add and hard to live with

Not every crate is a good choice just because it is popular, ergonomic, or well-documented. Some crates are excellent tools in the right place and terrible decisions in the wrong project.

This section is about the misleading cases: the dependencies that look like a shortcut, but quietly create complexity, abstraction, compile-time cost, and future maintenance pain.

#### The trap: "It saves a few lines"

A crate is not automatically a win just because it reduces boilerplate.

Typical examples:

- adding `tokio` to a CLI that is mostly synchronous and short-lived
- pulling in `clap` for a tiny script with three flags
- using `anyhow` in a public library where error types should remain explicit
- bringing in `serde` for a config format that is a few fixed key/value pairs
- adding a web framework to an internal tool with a single endpoint

The right question is not:

- "Can I use this crate?"

The more important question is:

- "Does this crate improve the design, or just hide the design?"

#### Misleading ergonomics

Some crates are ergonomic in the short term but create confusing long-term behavior.

Examples:

- `anyhow` is great for application code, but in library APIs it often erodes type precision and pushes error semantics outward.
- `tokio` is excellent for async I/O, but it is often used as a default even when a simple thread or synchronous pipeline would be clearer.
- `serde` is wonderful for structured data, but it is not always the right answer for small, controlled, human-written formats.
- `clap` is great for real CLIs, but a tiny internal utility should not import a large argument parsing framework just to parse two flags.
- `axum` or `actix-web` are strong choices for web services, but not for a small tool that merely reads input and sends a response once.

A crate can be ergonomic without being appropriate.

#### Red flags for a bad dependency choice

Be careful when a crate:

- solves a problem you barely need
- pulls in a large transitive graph for a small task
- requires learning a framework just to do one small job
- hides behavior behind macros or implicit conventions
- makes the code harder to understand than the problem itself
- becomes the dominant abstraction in the project
- is overkill for your team's level of experience
- is hard to replace later without rewriting the integration

If a dependency makes the code "less explicit" without providing real leverage, it is probably a bad trade.

#### Common wrong-usage patterns

##### 1. Using a framework for a tiny problem

This is one of the most common mistakes.

Example:

- a small internal utility is built with `axum` instead of a compact `TcpListener` or simple CLI flow
- a tiny script uses a full application framework to parse a couple of arguments and write a file

Why it is wrong:

- more dependencies
- more compile time
- more ceremony
- harder onboarding
- less obvious control flow

##### 2. Using async everywhere

Async is great when the workload is genuinely concurrent or I/O-bound.

It is not a universal default.

Wrong usage:

- using `tokio` in a short-lived, single-threaded CLI
- using `async` for code that is mostly CPU-bound or simple data transforms
- adding async runtime complexity before measuring whether it is actually needed

A good rule:

- if the program is short, direct, and mostly sequential, do not force async into the design.

##### 3. Using `anyhow` or `eyre` too early

These crates are excellent for tool and application-level code.

They are often the wrong choice in:

- library APIs
- domain models
- reusable abstractions
- code that should expose precise failure modes

Wrong pattern:

- every function returns `anyhow::Result<T>`
- error semantics are flattened into a generic message
- the caller cannot distinguish business errors from infrastructure errors

This creates flexibility at the cost of correctness.

##### 4. Using a “full-featured” crate for a simple format

A crate is often justified for:

- JSON payloads
- real config files
- networks and HTTP
- complex parsing

But not every format deserves a full serialization stack.

Wrong pattern:

- reading a small config file with `serde` and a massive struct graph when a few `fs::read_to_string` and `split` calls are enough
- parsing a single data file with a generic serialization crate when the format is fixed and narrow

The real question is not:

- "Is there a crate for this?"

It is:

- "Is the crate solving a general problem, or just making a tiny format feel more abstract?"

##### 5. Choosing the crate that is most visible, not the crate that is most suitable

Rust has many well-known libraries, but popularity is not a substitute for fit.

Wrong pattern:

- "Everyone uses `X`, so I should too"
- "This has a million downloads, so it must be the right abstraction"
- "The crate is famous, therefore the code is good"

Many projects fail because they adopt a famous ecosystem tool without first checking:

- whether it matches the architecture
- whether it adds needless abstraction
- whether it imposes concurrency or lifecycle models the project does not need

##### 6. Treating metaprogramming as a feature

Some crates are powerful because they hide a lot of logic in macros or derive attributes.

This can be useful, but it is also a source of confusion.

Wrong pattern:

- relying on generated code you do not understand
- adding proc-macros because they look magical and concise
- letting the crate define your design instead of the project defining its needs

Rust is very good at explicitness. A dependency that hides too much logic can easily become a maintenance trap.

#### A better rule of thumb

Use a dependency when it:

- matches the real problem
- keeps the code clearer, not more abstract
- has a stable and understandable maintenance story
- is worth the dependency cost
- can be replaced or understood by the team

Avoid a dependency when it:

- is more clever than necessary
- solves a generic problem with a specialized local need
- adds a large abstraction layer for a small task
- makes the code harder to reason about
- encourages “just because it exists” architecture

#### Example of a bad dependency decision

A project has a tiny command-line tool that reads a CSV file, filters some rows, and prints output.

Wrong:

- add `tokio`
- add `serde`
- add `clap`
- add `tracing`
- add a web framework in the same project "for future expansion"

Why it is wrong:

- the project is not doing async work
- the data format is simple
- the lifecycle is tiny
- the dependency graph becomes larger than the functionality

Better:

- keep it standard library first
- implement a small parser locally
- use `std::env` and plain `fs`
- make the design explicit and transparent

#### Final principle

The most dangerous crate is not the broken crate.
It is the crate that is technically working, widely used, and still a poor fit.

The best Rust code is not the one with the most dependencies.
It is the one with the smallest honest abstraction that matches the job.

---

## 14. Performance and compile-time reality

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

## 15. Security and maintenance considerations

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

## 16. Real-world project archetypes

### 16.1 Tiny CLI utility

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

### 16.2 REST API service

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

### 16.3 Embedded system

Project type:

- microcontroller
- firmware
- sensor hardware
- bare metal or custom runtime

Recommended approach:

- `no_std`
- `embedded-hal`
- `heapless` containers
- target-specific abstractions

### 16.4 Data-processing pipeline

Project type:

- CSV or log processing
- local ETL jobs
- file ingestion

Recommended approach:

- standard library for smaller jobs
- selected crates for parsing or concurrency when needed

### 16.5 Internal reusable library

Project type:

- domain logic
- shared algorithms
- service semantics

Recommended approach:

- keep dependencies minimal
- expose clean APIs
- prefer local implementation over a broad framework

---

## 17. Example code catalog

### 17.1 Standard library-only CLI

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

### 17.2 CLI with `clap`

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

### 17.3 JSON with `serde`

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

### 17.4 Async server with `tokio` and `axum`

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

### 17.5 Custom parser without external crates

```rust
fn parse_numbers(input: &str) -> Vec<i32> {
    input
        .split(',')
        .filter_map(|token| token.trim().parse().ok())
        .collect()
}
```

### 17.6 `no_std` library

```rust
#![no_std]

pub fn square(x: u32) -> u32 {
    x * x
}
```

---

## 18. Comparison tables

### 18.1 `std` vs external crates

| Topic | `std`-only approach | External crates |
| --- | --- | --- |
| Simplicity | High | Medium to low |
| Build complexity | Low | Higher |
| Dependency management | None | Required |
| Time to implement | More manual work | Faster |
| Flexibility | Very high | High but layered |
| Best use | Small tools, custom logic | Common infrastructure and APIs |

### 18.2 `std` vs `no_std`

| Topic | `std` | `no_std` |
| --- | --- | --- |
| Filesystem | Yes | Usually no |
| Threads | Yes | Usually no |
| Environment assumptions | Yes | Custom runtime required |
| Binary size | Larger | Smaller |
| Target environment | Normal OS | Embedded or minimal runtime |

### 18.3 Crate-first vs custom implementation

| Scenario | Prefer crate | Prefer custom |
| --- | --- | --- |
| JSON serialization | Yes | Sometimes |
| HTTP client | Yes | No |
| CLI parser | Yes | If very tiny |
| Custom config format | Sometimes | Often |
| Business logic | Often local | Frequently |
| Embedded runtime | Yes, if platform crate exists | Sometimes |

---

## 19. Learning roadmap

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

## 20. Glossary

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
- transitive dependency: a dependency required by another dependency
- async: non-blocking execution model

---

## 21. Final recommendation

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
- async runtimes
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

## 22. Intuitive and life-saver crates

“Intuitive” and “life-saver” crates are not absolute labels, but in practice they are the crates that make Rust feel ergonomic, less error-prone, and much easier to ship safely.

A rigorous categorization is:

### 22.1 Intuitive crates

These are crates whose APIs feel obvious, align with Rust conventions, and reduce friction without hiding too much.

Examples:

- `serde`
- `clap`
- `reqwest`
- `chrono`
- `uuid`
- `tracing`
- `anyhow`
- `thiserror`

Why they matter:

- they follow Rust idioms closely
- common operations feel natural and explicit
- they reduce boilerplate without sacrificing safety or control

### 22.2 Life-saver crates

These are crates that prevent correctness bugs, reduce boilerplate, make production code safer, or eliminate whole classes of mistakes that would otherwise be painful.

Examples:

- `thiserror`
- `anyhow`
- `eyre`
- `serde`
- `serde_json`
- `toml`
- `regex`
- `clap`
- `reqwest`
- `tokio`
- `tracing`
- `sqlx`
- `diesel`
- `proptest`
- `criterion`

Why they matter:

- they reduce runtime errors
- they replace brittle hand-written logic
- they give clearer failure modes
- they improve maintainability
- they create better debugging visibility in production systems

### 22.3 Error handling crates

These are among the most valuable and life-saving crates in the ecosystem.

#### `thiserror`

- Intuitive: very clear for typed errors
- Life-saver: eliminates repetitive error enum boilerplate
- Best for: libraries and domain error models
- Why: makes error modeling explicit and clean

#### `anyhow`

- Intuitive: very ergonomic for app-level code
- Life-saver: removes the burden of writing custom error types everywhere
- Best for: CLIs, services, scripts
- Why: perfect for “just propagate errors” code

#### `eyre`

- Intuitive: excellent error reporting for humans
- Life-saver: dramatically improves debugging
- Best for: tools and app-level diagnostics
- Why: rich context and readable error output

### 22.4 Serialization and parsing crates

#### `serde`

- Intuitive: standard pattern for Rust data modeling
- Life-saver: reduces custom serialization logic
- Best for: JSON, TOML, config, APIs
- Why: one of the most important crates in the ecosystem

#### `serde_json`

- Intuitive: familiar JSON workflow
- Life-saver: prevents hand-built JSON parsing issues
- Best for: API payloads, config, web services

#### `toml`

- Intuitive: simple config format support
- Life-saver: avoids brittle key/value parsing code
- Best for: app and tool configuration

#### `csv`

- Intuitive: structured data handling
- Life-saver: avoids writing manual CSV parsers
- Best for: data processing, ETL, imports

#### `regex`

- Intuitive: expressive textual matching
- Life-saver: prevents unreadable manual scanning logic
- Best for: validation, parsing, logs

### 22.5 CLI crates

#### `clap`

- Intuitive: extremely standard for Rust CLIs
- Life-saver: avoids fragile manual arg parsing
- Best for: official CLI apps
- Why: typed arguments, help text, validation

#### `argh`

- Intuitive: lightweight and readable
- Life-saver: simpler than hand-rolled parsing
- Best for: smaller tools

#### `dialoguer`

- Intuitive: interactive terminal UX
- Life-saver: reduces custom prompt boilerplate
- Best for: TUI-like command-line tools

### 22.6 HTTP and networking crates

#### `reqwest`

- Intuitive: idiomatic HTTP client
- Life-saver: eliminates custom networking code
- Best for: REST APIs, HTTP calls

#### `hyper`

- Intuitive: underlying HTTP primitives
- Life-saver: avoids implementing protocol details manually
- Best for: lower-level server/client work

#### `tokio`

- Intuitive: the de facto async ecosystem
- Life-saver: standardizes async runtime and I/O
- Best for: async services, networked apps

#### `axum`

- Intuitive: clean route and handler model
- Life-saver: reduces boilerplate for API services
- Best for: web APIs and backends

### 22.7 Data and time handling crates

#### `chrono`

- Intuitive: date/time handling is often painful in raw Rust
- Life-saver: avoids ad hoc date logic and timezone confusion
- Best for: timestamps, scheduling, validation

#### `uuid`

- Intuitive: standard unique ID generation
- Life-saver: avoids manual ID mistakes
- Best for: entities, tokens, correlation IDs

#### `rand`

- Intuitive: standard randomness
- Life-saver: avoids manual RNG design
- Best for: IDs, tests, simulations, probabilistic logic

### 22.8 Database crates

#### `sqlx`

- Intuitive: typed SQL access
- Life-saver: catches many errors at compile time
- Best for: production database-backed Rust apps

#### `diesel`

- Intuitive: strongly typed query builder
- Life-saver: makes database interaction safer and more structured
- Best for: complex relational systems

#### `rusqlite`

- Intuitive: straightforward local SQLite usage
- Life-saver: eliminates manual DB handling for local apps
- Best for: local apps, embedded-ish data stores

### 22.9 Testing and validation crates

#### `proptest`

- Intuitive: property-based testing is conceptually clean
- Life-saver: finds edge cases humans miss
- Best for: correctness-heavy logic

#### `criterion`

- Intuitive: benchmarks are easy to interpret
- Life-saver: prevents “performance guesses”
- Best for: optimization and regression awareness

#### `insta`

- Intuitive: snapshot testing
- Life-saver: catches unintended output changes
- Best for: serializer, compiler, CLI output snapshots

### 22.10 General ergonomics and developer workflow crates

- `anyhow`
- `thiserror`
- `clap`
- `tracing`
- `serde`
- `tokio`
- `axum`
- `itertools`
- `pretty_assertions`
- `colored`
- `indicatif`

These are not “core business logic” crates, but they are often the difference between a frustrating engineering workflow and a smooth one.

### 22.11 A strong practical shortlist

If I had to pick the most important categories of “intuitive and/or life-saver” crates, I’d rank them like this:

- `serde` / `serde_json`
- `thiserror` / `anyhow`
- `clap`
- `tokio`
- `tracing`
- `reqwest`
- `sqlx` / `diesel`
- `chrono`
- `uuid`
- `regex`
- `proptest`
- `criterion`

This is the set that most often makes Rust feel productive, safe, and maintainable.

---

## 23. Scoped and platform-specific crates

There is a third class of crates that matters enormously in real Rust work:

- domain-specific crates
- platform-specific crates
- highly scoped crates that solve one narrow problem extremely well

These are not always the broad “every project uses this” crates, but they are often essential in the right context. They are especially useful when you need to work across an OS boundary, target a specific platform, or solve a very narrow technical problem without reimplementing a lot of low-level logic.

A good rule for this category is:

- the problem is narrow but real
- the crate solves it much better than custom code
- the crate is mature and maintained
- the problem is common enough that reimplementation would be costly or error-prone

### 23.1 OS and system integration crates

These are classic platform-specific crates that are extremely practical when targeting operating systems or system-level APIs.

#### `windows`

- Use when: Windows desktop or system integration
- Why: better than raw FFI for many Windows-specific tasks
- Benefits: typed bindings, easier access to Win32 APIs

#### `windows-sys`

- Use when: low-level Windows API binding
- Why: direct and extremely precise access to Windows OS interfaces
- Benefits: thin layer, low-level control, precise API mapping

#### `nix`

- Use when: Unix system programming on Linux/macOS
- Why: safer and more idiomatic than raw libc for many system calls
- Benefits: better ergonomics and safer semantics than manual syscall wrappers

#### `libc`

- Use when: C ABI and low-level OS interop
- Why: fundamental crate for FFI and system interfaces
- Benefits: direct access to C-level primitives and platform declarations

#### `sysinfo`

- Use when: reading system metrics like CPU, memory, disks, processes
- Why: much easier than building this across platforms manually
- Benefits: easy monitoring and admin-tool integration

#### `which`

- Use when: locating executables in PATH
- Why: commonly needed in CLI utilities and tooling wrappers
- Benefits: cross-platform command discovery

#### `directories`

- Use when: standard app config/cache/data directories
- Why: cross-platform path handling is tedious without it
- Benefits: clean access to standard OS paths

#### `signal-hook`

- Use when: process lifecycle and graceful shutdown handling
- Why: robust signal support for long-running apps
- Benefits: safer signal handling for processes and daemons

### 23.2 File system and tooling crates

These are scoped but highly practical for project tooling, scanning, watching, and file-heavy tasks.

#### `walkdir`

- Use when: recursive directory traversal
- Why: easier and safer than hand-rolled recursive scanning
- Benefits: directory walking, filtering, and traversal control

#### `glob`

- Use when: file discovery using patterns
- Why: avoids brittle custom matching logic
- Benefits: near-glob semantics for CLI or file tooling

#### `tempfile`

- Use when: tests, local processing, and sandboxing
- Why: standardized temporary directory and file management
- Benefits: isolated temporary state and test safety

#### `notify`

- Use when: file watching and rebuild triggers
- Why: robust watch support for tools and developers
- Benefits: detects file changes across platforms

#### `fs_extra`

- Use when: advanced filesystem operations
- Why: `std::fs` is useful but limited for some workflows
- Benefits: copying, moving, and bulk filesystem handling

### 23.3 Data parsing and format-specific crates

These are extremely useful when the data format matters more than a general abstraction layer.

#### `nom`

- Use when: custom protocols, binary formats, or grammar-like parsing
- Why: parser combinators are excellent when a custom parser is needed
- Benefits: composable, efficient, explicit parsing design

#### `pest`

- Use when: formal grammar parsing
- Why: parser generators are easier than hand-writing grammar logic
- Benefits: grammar-first parsing model

#### `lalrpop`

- Use when: complex grammar-based language parsing
- Why: good for language-like syntax or domain-specific parsers
- Benefits: parser generation for LR grammars

#### `csv`

- Use when: tabular data imports or exports
- Why: manual CSV handling is tedious and error-prone
- Benefits: well-defined parsing and serialization support

#### `serde_yaml`

- Use when: YAML configuration or transport is required
- Why: integrates with the `serde` ecosystem
- Benefits: YAML support with `serde` data models

#### `quick-xml`

- Use when: XML-heavy inputs or outputs need parsing
- Why: XML handling by hand is brittle and tedious
- Benefits: efficient XML parsing for structured inputs

### 23.4 Cryptography and security crates

These are highly specialized but critical in the correct contexts.

#### `ring`

- Use when: TLS, signing, hashing, or secure primitives are required
- Why: trusted, security-focused implementation
- Benefits: good for modern cryptographic work

#### `sha2`

- Use when: hashing or content integrity checks are needed
- Why: standard, explicit hashing API
- Benefits: strong and simple hashing support

#### `blake3`

- Use when: very fast hashing is needed
- Why: high-throughput hashing for performance-sensitive workloads
- Benefits: excellent performance characteristics

#### `aes-gcm`

- Use when: encrypting data with symmetric encryption
- Why: modern encryption pattern for payload-level security
- Benefits: standard and widely used encryption API

#### `argon2`

- Use when: password hashing in authentication systems
- Why: modern password hashing is safer than custom approaches
- Benefits: strong password security by design

#### `ed25519-dalek`

- Use when: signing or identity primitives are needed
- Why: modern signature scheme with strong support
- Benefits: well-supported for modern key systems

### 23.5 GUI and desktop application crates

These are very scoped to app development, but powerful when that is the target domain.

#### `egui`

- Use when: lightweight GUI or immediate-mode UI is sufficient
- Why: easy and practical for simple native or browser-style apps
- Benefits: fast iteration and clean UI model

#### `iced`

- Use when: cross-platform desktop app UI is required
- Why: strong Rust-native UI framework
- Benefits: good for richer app UI work

#### `tauri`

- Use when: native desktop apps with a web frontend
- Why: strong hybrid app approach
- Benefits: Rust backend + web frontend

#### `slint`

- Use when: polished UI applications need declarative design
- Why: fine-grained UI development with Rust semantics
- Benefits: good for high-quality application interfaces

### 23.6 Browser and WebAssembly crates

These are highly relevant in the web/wasm world.

#### `wasm-bindgen`

- Use when: Rust interacts with JavaScript in browsers or WASM apps
- Why: necessary for browser interop
- Benefits: bridges between Rust and JS runtime

#### `web-sys`

- Use when: browser APIs are required from Rust
- Why: direct browser functionality mapping
- Benefits: works with wasm-bindgen ecosystem

#### `js-sys`

- Use when: JS interop primitives are required
- Why: low-level JS support for browser APIs
- Benefits: compatibility with browser functions and JS runtime

#### `gloo`

- Use when: browser-side Rust apps need convenience utilities
- Why: simplifies common browser tasks
- Benefits: helpful WASM helper layer

#### `console_error_panic_hook`

- Use when: browser debugging is required
- Why: makes panic visibility better in the browser
- Benefits: significantly better debugging experience in WASM

### 23.7 Embedded and hardware crates

These are crucial when targeting microcontrollers and hardware-driven environments.

#### `embedded-hal`

- Use when: microcontrollers and hardware drivers are involved
- Why: standard abstraction across hardware
- Benefits: portable hardware code across different MCUs

#### `heapless`

- Use when: no heap or constrained memory is available
- Why: fixed-capacity containers avoid heap issues
- Benefits: safer in embedded applications

#### `nb`

- Use when: non-blocking operations model is required
- Why: essential for embedded I/O flow
- Benefits: resource-efficient operation in constrained environments

#### `defmt`

- Use when: embedded device logging is required
- Why: optimized for microcontroller debugging
- Benefits: extremely practical in embedded work

#### `cortex-m`

- Use when: ARM Cortex-M targets are in use
- Why: direct support for the target architecture
- Benefits: hardware and runtime support for embedded ARM systems

### 23.8 Terminal and UI crates

These are very useful for command-line tools and developer utilities.

#### `colored`

- Use when: terminal output needs color and readability
- Why: improves CLI UX without much complexity
- Benefits: cleaner user-facing output

#### `indicatif`

- Use when: long-running CLI jobs need progress bars
- Why: high-quality progress UI for developer tools
- Benefits: user-friendly status output

#### `termion`

- Use when: interactive terminal control is required
- Why: useful for raw terminal manipulation
- Benefits: low-level terminal interaction for TUI apps

#### `crossterm`

- Use when: cross-platform terminal manipulation is required
- Why: widely used in terminal apps and TUI systems
- Benefits: cross-platform terminal support

### 23.9 Networking and protocol-specific crates

These are critical when the problem is not just “HTTP”, but a precise transport or protocol domain.

#### `rustls`

- Use when: secure network connections are needed
- Why: Rust-native TLS implementation
- Benefits: modern and portable TLS support

#### `hyper-rustls`

- Use when: HTTP stacks need TLS integration
- Why: integrates TLS into Hyper-based stacks
- Benefits: cleaner than custom TLS plumbing

#### `quinn`

- Use when: QUIC networking is desired
- Why: modern transport protocol support
- Benefits: efficient and modern network protocol handling

#### `tokio-tungstenite`

- Use when: WebSockets are required in async systems
- Why: good async WebSocket support
- Benefits: clean integration with Tokio

### 23.10 Performance and parallelism crates

These are narrow but very useful when performance and concurrency matter.

#### `rayon`

- Use when: data parallelism is a fit
- Why: easy parallel iteration without heavy thread management
- Benefits: simple speedups for CPU-bound tasks

#### `crossbeam`

- Use when: concurrency primitives beyond the standard library are needed
- Why: better support for channels, locks, and concurrency tools
- Benefits: advanced multi-threaded patterns

#### `smallvec`

- Use when: performance-sensitive code benefits from stack-backed small collections
- Why: reduces heap churn in hot paths
- Benefits: improves performance for small collections

#### `hashbrown`

- Use when: hash map implementations matter in highly optimized code
- Why: performant hash map implementation
- Benefits: useful in specialized systems work

#### `ordered-float`

- Use when: float ordering or comparison needs to be deterministic without NaN surprises
- Why: avoids subtle float comparison issues
- Benefits: safer ordering in data-heavy code

### 23.11 Developer workflow and tooling crates

These are less about application logic and more about the engineering workflow.

- `cargo-edit`
- `cargo-watch`
- `cargo-deny`
- `cargo-audit`
- `which`
- `tempfile`
- `directories`
- `colored`
- `indicatif`

These are not “core business logic” crates, but they are often the difference between a frustrating engineering workflow and a smooth one.

### 23.12 A good rule for scoped crates

A crate in this category is worth it if all of the following are true:

- the problem is narrow but real
- the crate solves it much better than custom code
- the crate is mature and maintained
- the problem is common enough that reimplementation would be expensive or error-prone

Examples of good scoped crates:

- `which` for command lookup
- `directories` for standard user paths
- `walkdir` for recursive file traversal
- `tempfile` for test isolation
- `notify` for directory watching
- `url` for robust URL parsing
- `sha2` for hashing
- `egui` for UI work
- `wasm-bindgen` for browser interop
- `embedded-hal` for hardware abstraction

Examples where a scoped crate is not worth it:

- a small custom CLI parser with just a few flags
- a one-off JSON file with a fixed schema
- simple config files with only a handful of fields
- tiny app logic that does not need OS or platform integration
- custom algorithms that do not match an established crate pattern

### 23.13 Summary

Broader “life-saver” crates improve almost every Rust project.

Scoped or platform-specific crates improve a specific target, environment, or domain.

That means the real Rust ecosystem is not just a mass of general-purpose libraries. It is a highly reliable toolbox built for precise engineering problems.

This is why Rust is so powerful: it gives you both:

- strong general-purpose foundations
- extremely good domain, platform, and specialized tooling support

---

## Quick decision guide

If the task is:

- a tiny script -> `std`
- a custom data parser -> `std` or tiny custom code
- a service with HTTP and async -> `tokio` + framework crates
- a CLI with commands -> `clap`
- JSON data -> `serde` + `serde_json`
- firmware target -> `no_std` + embedded crates
- a library with domain logic -> minimal internal logic, few external crates
- a Windows-specific tool -> `windows` or `windows-sys`
- a Linux/Unix tool -> `nix` or `libc`
- a file-watching or scanning tool -> `walkdir`, `glob`, `notify`
- a cryptographic tool -> `ring`, `sha2`, `argon2`, `ed25519-dalek`
- a browser or WASM app -> `wasm-bindgen`, `web-sys`, `gloo`
- an embedded or MCU project -> `embedded-hal`, `heapless`, `defmt`

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
