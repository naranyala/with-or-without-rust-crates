# Rust Crates Catalog: Using Rust With or Without External Dependencies

A practical, opinionated catalog for understanding Rust programming in both directions:

- Using crates from the Rust ecosystem
- Building Rust projects without crates at all
- Knowing when each approach is the right choice

This repository is a quick-reference guide for Rust developers who want to make good decisions about dependency use, code structure, performance, security, and maintainability.

## Why this catalog exists

Rust is famous for two things:

1. High safety and correctness guarantees
2. A rich ecosystem of reusable libraries called crates

But not every Rust project needs a large dependency graph. Sometimes the best Rust code is the smallest and most explicit code possible.

This catalog helps answer questions such as:

- When should I reach for a crate?
- When should I write the logic myself?
- What is the difference between the standard library and external crates?
- Can I build Rust without crates in embedded, CLI, or no_std environments?
- Which crates are useful for common real-world tasks?

---

## Table of contents

- [1. Rust at a glance](#1-rust-at-a-glance)
- [2. Crates vs no crates](#2-crates-vs-no-crates)
- [3. Rust ecosystem overview](#3-rust-ecosystem-overview)
- [4. When to use crates](#4-when-to-use-crates)
- [5. When not to use crates](#5-when-not-to-use-crates)
- [6. The standard library first](#6-the-standard-library-first)
- [7. no_std and minimal Rust](#7-no_std-and-minimal-rust)
- [8. Cargo and dependency management](#8-cargo-and-dependency-management)
- [9. Common crates by category](#9-common-crates-by-category)
- [10. Typical Rust project layouts](#10-typical-rust-project-layouts)
- [11. Design patterns for safe dependency use](#11-design-patterns-for-safe-dependency-use)
- [12. Pitfalls and anti-patterns](#12-pitfalls-and-anti-patterns)
- [13. Example: projects using crates](#13-example-projects-using-crates)
- [14. Example: projects without crates](#14-example-projects-without-crates)
- [15. Quick decision guide](#15-quick-decision-guide)
- [16. Learning roadmap](#16-learning-roadmap)
- [17. Glossary](#17-glossary)

---

## 1. Rust at a glance

Rust is a systems programming language designed around:

- memory safety without a garbage collector
- fearless concurrency
- explicit ownership and borrowing
- zero-cost abstractions
- reliable performance

Core strengths:

- excellent for command-line tools
- great for backend services
- suitable for embedded and bare-metal systems
- strong performance for game engines, media, networking, and systems software

Rust code is usually compiled ahead of time and can be extremely efficient. The language encourages correctness by default, and crates amplify that philosophy by providing tested building blocks.

---

## 2. Crates vs no crates

### Using crates

A crate is a Rust package or library. The Rust community publishes crates to crates.io, and Cargo is the official package manager.

Benefits:

- faster development
- battle-tested functionality
- easier integration with external systems
- broad ecosystem coverage
- community support and documentation

Costs:

- dependency risk
- compile time increases
- semver compatibility concerns
- transitive dependency complexity
- security and maintenance auditing

### Not using crates

A project may intentionally avoid external dependencies and rely only on:

- the Rust standard library (`std`)
- custom local modules
- minimal or no_std code
- a single well-contained binary or library

Benefits:

- simpler build graph
- lower compile times
- better control and predictability
- easier reasoning about security and maintenance
- ideal for constrained or embedded targets

Costs:

- more implementation work
- more custom maintenance burden
- less ecosystem leverage
- potential duplication of functionality

A good Rust developer knows both styles and chooses intentionally.

---

## 3. Rust ecosystem overview

### Core pieces

- Rust compiler: `rustc`
- Build tool: `cargo`
- Package registry: `crates.io`
- Dependency format: `Cargo.toml`
- Lock file: `Cargo.lock`

### Rust dependency layers

- `std`: standard library for desktop/server environments
- `core`: foundational primitives for low-level code
- `alloc`: heap allocation utilities
- `no_std`: environment without the standard library
- external crates: community-published libraries

### Typical crate categories

- CLI and parsing: `clap`, `serde`, `serde_json`, `toml`, `regex`
- HTTP and networking: `reqwest`, `hyper`, `tokio`, `axum`
- Data and serialization: `serde`, `serde_json`, `bincode`, `chrono`, `uuid`
- Error handling: `anyhow`, `thiserror`
- Logging and tracing: `tracing`, `log`, `env_logger`
- Async runtime: `tokio`, `async-std`
- Database access: `sqlx`, `diesel`, `rusqlite`
- Web frameworks: `axum`, `actix-web`, `rocket`
- Embedded and systems: `embedded-hal`, `esp-idf-hal`, `heapless`
- Testing: `criterion`, `proptest`, `tokio-test`

---

## 4. When to use crates

Use a crate when:

- the problem already has a mature Rust solution
- you need networking, serialization, database, async runtime, or parsing
- the feature is complex and would take a long time to implement correctly
- the crate is supported, documented, and well-maintained
- you want to leverage established community patterns

### Good reasons to use crates

- JSON parsing with `serde` + `serde_json`
- CLI argument parsing with `clap`
- HTTP clients with `reqwest`
- asynchronous systems with `tokio`
- error modeling with `thiserror`
- structured logging with `tracing`
- web APIs with `axum`

### Decision signs

Choose a crate if:

- you are writing production code, not a toy example
- the logic is repetitive, standard, and broadly solved
- your team wants something well-ratified in the ecosystem
- performance and correctness are important but not at the cost of reinventing a wheel

---

## 5. When not to use crates

Avoid crates when:

- the functionality is tiny, custom, and simple
- you need full control over logic or memory
- the project is a learning exercise
- the target is constrained or embedded
- a crate introduces unnecessary abstractions or dependency bloat
- you are writing a kernel, bootloader, or low-level library with very strict constraints

### Common no-crate scenarios

- parsing small custom file formats
- writing a tiny utility binary
- implementing a simple algorithm yourself
- designing a core library with minimal dependencies
- building embedded or bare-metal software

### Rule of thumb

When the logic is local, specific, and clearly bounded, writing it by hand is often better than an external dependency.

---

## 6. The standard library first

Rust's standard library is powerful and should always be considered first.

Helpful `std` features:

- collections: `Vec`, `HashMap`, `HashSet`, `BTreeMap`
- strings and text: `String`, `str`
- filesystem: `std::fs`, `std::path`
- networking: `std::net`
- threads: `std::thread`
- time and duration: `std::time`
- I/O: `Read`, `Write`, `BufRead`
- synchronization: `Mutex`, `Arc`, `RwLock`

### Good standard-library-only patterns

- tiny CLI tools
- local data processing
- simple file reading/writing
- custom algorithms
- robust in-house business logic

### Example: using `std` directly

```rust
use std::fs;
use std::env;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let path = env::args().nth(1).unwrap_or_else(|| "input.txt".to_string());
    let content = fs::read_to_string(path)?;
    println!("{}", content.trim());
    Ok(())
}
```

This approach is minimal, transparent, and avoids dependency cost.

---

## 7. no_std and minimal Rust

`no_std` means a program or library does not depend on Rust's standard library. Instead, it uses `core` and sometimes `alloc`.

### When to use `no_std`

- embedded systems
- bare-metal environments
- kernel-level or bootloader work
- custom allocators
- highly constrained runtimes

### Core concepts

- `core`: always available
- `alloc`: for heap operations if available
- `#![no_std]`: disable the standard library
- `#![no_main]`: custom entry point in some embedded targets
- `static mut` and raw pointers: possible but careful

### Benefits of `no_std`

- smaller binary
- less runtime dependency
- strong control over environment
- ideal for embedded systems

### Trade-offs

- no filesystem APIs from `std`
- no threads or dynamic allocation by default
- more care needed for memory and startup

### Example: minimal `no_std` library

```rust
#![no_std]

pub fn add(a: u32, b: u32) -> u32 {
    a + b
}
```

This is the minimal expression of a Rust library without external crates and without the standard library.

---

## 8. Cargo and dependency management

Cargo is the core tool for Rust development.

### Basic commands

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

- normal dependencies: used in the application
- dev-dependencies: used only in tests and development tooling
- optional features: enable only when required
- workspace dependencies: shared across multiple crates
- lockfile: ensures reproducible builds

### Features patterns

```toml
[dependencies]
serde = { version = "1", features = ["derive"] }
json = { version = "0.12", optional = true }

[features]
default = ["json"]
serde_json = ["dep:json"]
```

### Best practices

- prefer narrow, needed dependencies
- enable only required features
- check semantic version compatibility
- keep `Cargo.lock` in applications
- audit transitive dependencies regularly
- prefer mature crates with active maintenance

---

## 9. Common crates by category

### CLI and argument parsing

- `clap`: command-line argument parsing
- `argh`: attribute-based CLI parsing
- `dialoguer`: interactive prompts

### Serialization and data formats

- `serde`: serialization abstraction
- `serde_json`: JSON handling
- `toml`: TOML parsing
- `ron`: Rust Object Notation
- `bincode`: compact binary serialization

### HTTP and networking

- `reqwest`: HTTP client
- `hyper`: low-level HTTP library
- `tokio`: async runtime
- `axum`: web framework
- `tower`: service abstractions

### Logging and observability

- `tracing`: structured logging and instrumentation
- `log`: logging facade
- `env_logger`: simple logger

### Error handling

- `anyhow`: ergonomic error propagation
- `thiserror`: typed error definitions
- `eyre`: rich error reporting

### Data, time, and identity

- `chrono`: date/time support
- `uuid`: UUID generation and parsing
- `rand`: random number generation
- `regex`: regular expressions

### Database and storage

- `sqlx`: async database access
- `diesel`: ORM and query builder
- `rusqlite`: SQLite binding

### Async and concurrency

- `tokio`
- `async-std`
- `futures`

### Web and backend

- `axum`
- `actix-web`
- `rocket`

### Embedded and no_std support

- `embedded-hal`
- `heapless`
- `nb`
- `defmt`

### Testing and benchmarking

- `criterion`
- `proptest`
- `insta`
- `tokio-test`

---

## 10. Typical Rust project layouts

### Simple binary without crates

```text
project/
├── Cargo.toml
├── src/
│   └── main.rs
└── README.md
```

Use this when you need a single, explicit application.

### Library with local modules

```text
project/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── config.rs
│   ├── parser.rs
│   └── engine.rs
└── tests/
```

Use this when your logic is substantial but still self-contained.

### Application with multiple crates in a workspace

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

Use this for large systems or multiple related components.

---

## 11. Design patterns for safe dependency use

### 1. Prefer the standard library for the obvious things

Use `std` before reaching for a crate for common primitives and file operations.

### 2. Keep the dependency graph small

A large crate graph increases compile time and weakens maintainability.

### 3. Prefer crates that solve a narrow problem well

Big frameworks are useful, but small, focused libraries are often easier to reason about.

### 4. Use features deliberately

Never enable every crate feature by default unless it is necessary.

### 5. Prefer semver-compatible, actively maintained crates

A dependency should have an active maintainer base and a clear maintenance story.

### 6. Write custom logic when the domain is highly specific

If a crate is a generic abstraction for a problem you control entirely, custom code may be cleaner.

---

## 12. Pitfalls and anti-patterns

### Dependency bloat

Adding crates casually creates faster build times and more complexity.

### Framework overreach

A large dependency can hide the true behavior of the code.

### Unnecessary abstraction

If a crate adds complexity without clear value, it is likely not worth it.

### Feature creep

A crate with dozens of features may encourage pulling in everything.

### Blind semver upgrades

Dependency versions should be reviewed and constrained carefully.

### Using crates for trivial code

Parsing a tiny config file with a crate may be overkill when a small custom parser is readable and simpler.

### Building without understanding `std`/`no_std`

A developer can mistakenly use `std` features in embedded or low-level contexts.

---

## 13. Example: projects using crates

### Example A: Web API with crates

```toml
[dependencies]
axum = "0.7"
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

```rust
use axum::{routing::get, Router};

#[tokio::main]
async fn main() {
    let app = Router::new().route("/", get(|| async { "hello from Rust" }));

    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000").await.unwrap();
    axum::serve(listener, app).await.unwrap();
}
```

This is a classic high-productivity Rust setup using modern crates.

### Example B: CLI utility with crates

```toml
[dependencies]
clap = { version = "4", features = ["derive"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

This is ideal for a tool that needs robust argument parsing and JSON I/O.

---

## 14. Example: projects without crates

### Example A: pure standard-library CLI

```rust
use std::env;
use std::fs;

fn main() {
    let args: Vec<String> = env::args().collect();
    if args.len() < 2 {
        eprintln!("usage: app <file>");
        std::process::exit(1);
    }

    let content = fs::read_to_string(&args[1]).expect("read failed");
    println!("Read {} bytes", content.len());
}
```

### Example B: custom parser without dependencies

```rust
fn parse_ints(input: &str) -> Vec<i32> {
    let mut result = Vec::new();
    for token in input.split(',') {
        let trimmed = token.trim();
        if !trimmed.is_empty() {
            result.push(trimmed.parse::<i32>().unwrap());
        }
    }
    result
}
```

This kind of code is perfect when the data format is tightly controlled and you want full responsibility over behavior.

### Example C: `no_std` core library

```rust
#![no_std]

pub fn square(x: u32) -> u32 {
    x * x
}
```

This is very common in constrained or embedded contexts where dependencies are intentionally limited.

---

## 15. Quick decision guide

### Use crates when:

- you need common infrastructure
- you want a library with active maintenance
- task complexity is high
- speed of delivery matters
- you need broad ecosystem interoperability

### Avoid crates when:

- the task is small and bespoke
- you value simplicity and explicit control
- the project must stay minimal
- the target environment is resource-constrained
- you want to minimize lockfile and build complexity

### Use `std` when:

- the project is a local CLI, helper, or data processor
- you are writing application logic in a standard runtime

### Use `no_std` when:

- the environment is embedded or custom
- minimal dependencies are essential

---

## 16. Learning roadmap

### Beginner path

1. Learn variables, ownership, borrowing, and lifetimes
2. Write a few tiny CLI programs with `std`
3. Learn `Result`, `Option`, and error handling
4. Explore `cargo build`, `cargo run`, and `cargo test`
5. Add one crate intentionally, such as `serde` or `clap`
6. Understand `Cargo.toml` and feature flags

### Intermediate path

1. Learn workspace organization
2. Explore async with `tokio`
3. Build a small HTTP server with `axum`
4. Learn logging, configuration, and tests
5. Practice dependency auditing and feature minimization

### Advanced path

1. Work with `no_std` and embedded systems
2. Build reusable libraries with careful API design
3. Design traits and generic abstractions carefully
4. Optimize compile time and dependency graph size
5. Understand semver, maintainership, and long-term compatibility

---

## 17. Glossary

- crate: a Rust package or library
- Cargo: the Rust build system and package manager
- std: Rust's standard library
- no_std: Rust without the standard library
- feature: optional functionality that can be enabled in a crate
- dependency graph: the structure of packages and transitive dependencies
- semver: semantic versioning
- workspace: a multi-crate Rust project managed together
- async: non-blocking execution model based on futures
- lockfile: fixed dependency versions for reproducible builds

---

## Final recommendation

The best Rust code is not always the code with the most dependencies.

A strong Rust developer learns to ask:

- What does the standard library already solve?
- Is this a problem that a mature crate solves better?
- Is the dependency small, proven, and necessary?
- Would a custom implementation be clearer and safer in this project?

The answer is often a mix of both approaches:

- use a crate for well-established external problems
- write local, explicit code for custom or domain-specific logic
- keep dependencies narrow and purposeful

That balance is the heart of practical Rust engineering.

---

## Quick reference cheat sheet

### When to prefer crates

- networking
- JSON and config parsing
- database access
- async runtime
- API frameworks
- logging
- command-line interfaces

### When to prefer no-crate / std-only approach

- tiny tools
- educational projects
- custom parsers
- small reusable internal logic
- constrained runtimes
- bare-metal or embedded code

### Best practice mindset

- Start small
- Use the standard library first
- Add crates only when they clearly buy you something
- Keep your dependency graph honest and minimal

---

If you want, this repository can also be expanded with:

- a curated list of the best crates by category
- Rust project templates (binary, library, workspace, no_std)
- a comparison table between `std` vs `no_std` vs crates
- example code for common patterns in CLI, web, and embedded Rust

This README is designed to be a durable catalog and starting point for learning or teaching Rust in a practical, dependency-aware way.

