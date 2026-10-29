# Rust Crates Catalog

A practical, opinionated guide to understanding Rust in both directions:

- building with external crates from the Rust ecosystem
- building without crates at all
- deciding when each approach is the right tool for the job

This repository exists to help Rust developers, learners, and teams make thoughtful decisions about dependency use, project architecture, performance, safety, security, and maintainability.

## Why this repo exists

Rust is a language of clarity, safety, and control. It has an excellent standard library, a rich ecosystem of reusable libraries, and a culture of building small, explicit, correct systems.

The central question is not:

- "Should I always use crates?"
- "Should I avoid crates entirely?"

The real question is:

- "What is the smallest and best design for this project?"

This project helps answer that question in a practical, engineering-minded way.

---

## Quick decision matrix

| Situation | Best default | Why |
| --- | --- | --- |
| Tiny script or utility | `std` | Minimal, explicit, no dependency cost |
| CLI app with options | `clap` | Strong UX and validation |
| JSON API payloads | `serde` + `serde_json` | Standard, safe, productive |
| HTTP client | `reqwest` | Ergonomic and widely adopted |
| Async service | `tokio` + web framework | Mature async ecosystem |
| Embedded firmware | `no_std` + `embedded-hal` | Thin runtime, low-level control |
| Local config file | `std` or `toml` | Depends on complexity |
| Database app | `sqlx` or `diesel` | Strong typing and safety |
| Custom binary protocol | `nom` or manual parser | Domain-specific complexity |
| WASM or browser interop | `wasm-bindgen` + `web-sys` | Browser-specific integration |
| Platform integration | `windows`, `nix`, `directories`, `which` | OS-specific access and ergonomics |

---

## Core idea

Use the standard library first. Use crates when they solve a real, recurring, mature problem. Avoid crates when the task is small, specific, or easier to express locally.

The best Rust project is not the one with the most crates. It is the one with the smallest honest abstraction that matches the task.

---

## 1. Rust in one sentence

Rust is a systems programming language designed to give you memory safety, explicit control, and performance without sacrificing correctness.

It works well for:

- command-line tools
- backend services
- data processing
- embedded systems
- networking
- game engines
- kernel, firmware, and low-level work

---

## 2. `std` vs crates vs `no_std`

### Standard library only

Best when:

- the problem is small and local
- the logic is domain-specific and clear
- the project is a tiny CLI or utility
- there is no strong need for networking, parsing, or async runtime support

Examples:

- file reading and writing
- local transformation scripts
- custom algorithms
- simple CLI tools

### Crates

Best when:

- the problem is common and already solved well
- the API is well-known and stable
- the crate reduces boilerplate without hiding too much logic
- the crate is mature and maintained

Examples:

- `clap` for CLI parsing
- `serde` + `serde_json` for data handling
- `tokio` for async execution
- `reqwest` for HTTP
- `tracing` for structured logs

### `no_std`

Best when:

- the environment is constrained or embedded
- the standard library is not available
- the runtime assumptions are custom
- the project targets bare metal or microcontrollers

Examples:

- firmware
- drivers
- bootloaders
- tiny embedded libraries

---

## 3. Why the decision matters

Crates can speed development, but they also add:

- compile-time cost
- dependency graph complexity
- maintenance risk
- semver compatibility concerns
- hidden behavior behind abstractions

Manual code can be very clean, but it may mean:

- more code to maintain
- duplicated logic
- slower delivery for common tasks
- more custom validation

The best project is usually the one that stays understandable and maintainable.

---

## 4. Standard library patterns worth knowing

### Files and arguments

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

### String parsing

```rust
let number: i32 = "42".parse().unwrap();
```

### Custom parser example

```rust
fn parse_csv_row(line: &str) -> Vec<&str> {
    line.split(',').collect()
}
```

This is often cleaner than adding a parsing crate for a small, domain-specific format.

---

## 5. Cargo ecosystem

Rust uses Cargo for dependency management and builds.

### Common commands

```bash
cargo new my_project
cargo build
cargo run
cargo test
cargo check
cargo fmt
cargo clippy
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

### Key concepts

- normal dependencies: used by the program
- dev-dependencies: used only in tests or tooling
- features: optional functionality inside a crate
- workspaces: multi-crate projects managed together
- Cargo.lock: reproducible dependency resolution

---

## 6. When to use crates

Use a crate when the problem is:

- common
- shared across many projects
- stable and well-understood
- difficult to implement correctly by hand
- already solved by the Rust ecosystem

### Good reasons to use crates

- JSON parsing: `serde`, `serde_json`
- CLI parsing: `clap`
- HTTP: `reqwest`
- async runtime: `tokio`
- database access: `sqlx`, `diesel`
- structured logging: `tracing`
- error propagation: `anyhow`, `thiserror`

### Example: JSON handling

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct User {
    name: String,
    age: u32,
}
```

This is much easier and safer than writing a custom serializer by hand.

---

## 7. When not to use crates

Avoid external crates when:

- the logic is tiny and custom
- the requirement is narrow and easy to implement directly
- you want full, explicit control
- the target environment is minimal or embedded
- the dependency graph would exceed the value of the abstraction
- you are teaching, prototyping, or experimenting

### Rule of thumb

If a crate only saves a few lines but adds complexity or unnecessary dependencies, keep it local.

---

## 8. `no_std` and minimal Rust

`no_std` tells Rust not to use the standard library. Instead, it relies on `core` and may use `alloc` when available.

### Typical use cases

- embedded software
- bare-metal targets
- firmware
- bootloaders
- kernel-like code
- tiny runtime environments

### Example

```rust
#![no_std]

pub fn add(a: u32, b: u32) -> u32 {
    a + b
}
```

### Why it matters

In constrained systems, the standard library may be unavailable or too heavy. Rust still gives you useful lower-level abstractions in those environments.

---

## 9. Cargo workspace patterns

Rust workspaces let you manage multiple crates as one project.

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

### Principles

- separate app code from domain logic
- keep shared logic in a core crate
- avoid monolithic binaries unless the project justifies them

---

## 10. Dependency hygiene

Good dependency hygiene means:

- prefer the minimum number of dependencies
- prefer small targeted crates over giant frameworks
- only enable the features you need
- audit transitive dependencies
- prefer maintained crates with active maintainers
- avoid crates that are over-engineered for your task

### Red flags

- a crate solves a problem you barely need
- it pulls in many unrelated dependencies
- it has poor or sparse documentation
- it is poorly maintained or inactive
- it adds a large abstraction layer for a small task

---

## 11. Rust crate families to know

This repo keeps the important ecosystem buckets, without assuming every project needs all of them.

### CLI and argument parsing

- `clap`
- `argh`
- `dialoguer`

### Serialization and data formats

- `serde`
- `serde_json`
- `toml`
- `bincode`
- `ron`
- `csv`

### HTTP, networking, and APIs

- `reqwest`
- `hyper`
- `tokio`
- `axum`
- `tower`
- `rustls`
- `quinn`
- `tokio-tungstenite`

### Logging and observability

- `tracing`
- `log`
- `env_logger`
- `colored`
- `indicatif`

### Error handling

- `thiserror`
- `anyhow`
- `eyre`

### Database and storage

- `sqlx`
- `diesel`
- `rusqlite`

### Async and concurrency

- `tokio`
- `async-std`
- `futures`
- `rayon`
- `crossbeam`

### Embedded and low-level

- `embedded-hal`
- `heapless`
- `nb`
- `defmt`
- `cortex-m`

### Testing and benchmarking

- `criterion`
- `proptest`
- `insta`
- `pretty_assertions`

### Cryptography and security

- `ring`
- `sha2`
- `blake3`
- `aes-gcm`
- `argon2`
- `ed25519-dalek`

### Platform-specific and system integration

- `windows`
- `windows-sys`
- `nix`
- `libc`
- `which`
- `directories`
- `signal-hook`
- `walkdir`
- `glob`
- `notify`
- `tempfile`

### Browser and WASM

- `wasm-bindgen`
- `web-sys`
- `js-sys`
- `gloo`
- `console_error_panic_hook`

### GUI and desktop

- `egui`
- `iced`
- `tauri`
- `slint`

---

## 12. Common anti-patterns and wrong crate usage

This is one of the most important sections of the project: the crates that are technically usable, but are a poor fit for the task or the architecture.

### 12.1 The main trap

A dependency is not automatically a win just because it reduces boilerplate.

The real question is not:

- "Can I use this crate?"

It is:

- "Does this crate improve the design, or simply hide the design?"

### 12.2 Using async everywhere

`tokio` is excellent for real async I/O and networked services.

It is a poor fit for:

- short-lived CLI tools
- mostly synchronous data processing scripts
- tiny utilities that do not need non-blocking I/O

Wrong pattern:

```rust
#[tokio::main]
async fn main() {
    // a tiny script that just reads a file and exits
}
```

This adds async runtime complexity without real benefit.

### 12.3 Using `anyhow` too broadly

`anyhow` is excellent for app-level code and CLI tooling.

It is often a poor choice for:

- public libraries
- domain APIs
- reusable abstractions
- code that should expose precise error types

Wrong pattern:

```rust
fn parse_user() -> anyhow::Result<User> {
    // hides precise domain error semantics behind a generic error type
}
```

This is ergonomic, but it loses information and weakens correctness signals.

### 12.4 Using `clap` for tiny utilities

`clap` is great for real CLIs.

It is overkill for:

- scripts with two flags
- internal tools with a handful of positional args
- quick prototypes

Wrong pattern:

- "I used `clap` because it is the default"
- "It is popular, so it must be correct for this case"

If the project is tiny, `std::env::args()` and a few manual checks may be clearer.

### 12.5 Using `serde` for trivial fixed formats

`serde` is one of the best crates in Rust.

But it is not always the right answer when:

- the file format is tiny and rigid
- the format is simple enough to parse with `split` or a few direct checks
- the schema is fixed and does not need a general data model

Wrong pattern:

- reading a tiny config file with a large `serde` struct graph when a few simple `String` splits are enough
- forcing a structured data model onto a custom format that is not actually complex

### 12.6 Using a web framework for a trivial tool

Frameworks like `axum`, `actix-web`, and `rocket` are excellent for APIs.

They are a poor fit when:

- a project is really a CLI utility
- there is only one endpoint and the code is not a full web product
- the app does not need routing, middleware, request typing, or web ergonomics

Wrong pattern:

- building a tiny service with an application framework before proving the workload is genuinely web-heavy

### 12.7 Choosing a crate just because it is famous

Popular crates are not automatically correct choices.

Wrong pattern:

- "Everyone uses `X`, so I should use it too"
- "This crate has a million downloads, so it must be the right abstraction"

A dependency should match the actual architecture and workload, not just the popularity list.

### 12.8 Misleading ergonomics

Some crates are ergonomic in the short-term but create long-term confusion.

Examples:

- `anyhow` in a library API
- `tokio` in a mostly sequential tool
- `serde` applied to a format that does not benefit from structured modeling
- `axum` in a project that only does a single task and exits

The danger is not broken code. The danger is code that works but is harder to reason about than the original problem.

### 12.9 Maintenance and dependency hygiene problems

A bad dependency decision often looks like this:

- a crate solves a tiny problem but brings a big graph
- a crate is not actively maintained
- the dependency relies on many transitive libraries for a narrow task
- the project is now more expensive to build, audit, and understand than the original problem

Look out for:

- large transitive trees
- sparse or unclear documentation
- stale issue history
- a mismatch between "library power" and project size

### 12.10 Better rule of thumb

Use a dependency when it:

- matches the real problem
- keeps the code clearer instead of more abstract
- is easy to maintain and understand
- adds real leverage for the project

Avoid it when it:

- is more clever than necessary
- brings too much abstraction for a small task
- makes the code harder to reason about
- encourages architecture by trend instead of fit

---

## 13. Project archetypes

Rust is not one-size-fits-all. The right tool depends on the kind of project you are building.

### 13.1 Tiny CLI utility

Project type:

- reads a file
- transforms data
- prints output

Typical choice:

- `std` first
- small custom logic
- maybe a tiny helper crate like `clap` if the CLI grows

Example:

```rust
use std::fs;

fn main() {
    let content = fs::read_to_string("input.txt").unwrap();
    println!("{}", content.trim());
}
```

### 13.2 REST API service

Project type:

- HTTP endpoints
- JSON request/response
- database access
- async processing

Typical choice:

- `axum` or `actix-web`
- `tokio`
- `serde` + `serde_json`
- `sqlx` or `diesel`
- `tracing`

This is where the ecosystem really shines.

### 13.3 Embedded system

Project type:

- microcontroller
- firmware
- sensor hardware
- bare metal or custom runtime

Typical choice:

- `no_std`
- `embedded-hal`
- `heapless`
- `defmt`
- target-specific abstractions

This is one of the strongest reasons to avoid the default “just add a crate” mindset.

### 13.4 Data-processing pipeline

Project type:

- CSV or log processing
- local ETL jobs
- file ingestion
- transform-and-emit workloads

Typical choice:

- `std` for smaller jobs
- selected crates for parsing, concurrency, or structured output
- `csv`, `serde`, and `regex` when useful

### 13.5 Internal reusable library

Project type:

- domain logic
- shared algorithms
- service semantics
- cross-project business rules

Typical choice:

- small dependency footprint
- carefully designed public API
- prefer local implementation over a broad framework

### 13.6 Tooling and developer workflow projects

Project type:

- file scanning and watching
- code generation
- build helpers
- command wrappers

Typical choice:

- `which`
- `walkdir`
- `glob`
- `notify`
- `tempfile`
- `directories`
- `colored`
- `indicatif`

### 13.7 GUI or desktop applications

Project type:

- local desktop UI
- hybrid web/native app
- TUI or interactive terminal app

Typical choice:

- `egui`
- `iced`
- `tauri`
- `slint`
- `crossterm`
- `termion`

### 13.8 Browser and WebAssembly apps

Project type:

- browser-side Rust logic
- interactive frontend logic
- web interop with JavaScript

Typical choice:

- `wasm-bindgen`
- `web-sys`
- `js-sys`
- `gloo`

### 13.9 Cryptographic or security-critical work

Project type:

- hashing
- signatures
- password storage
- payload encryption

Typical choice:

- `ring`
- `sha2`
- `blake3`
- `aes-gcm`
- `argon2`
- `ed25519-dalek`

---

## 14. Practical crate examples

### 14.1 CLI with `clap`

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

### 14.2 JSON with `serde`

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Person {
    name: String,
    age: u32,
}
```

### 14.3 Async server with `tokio` and `axum`

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

### 14.4 Embedded example with `no_std`

```rust
#![no_std]

pub fn square(x: u32) -> u32 {
    x * x
}
```

### 14.5 Custom parser without external crates

```rust
fn parse_numbers(input: &str) -> Vec<i32> {
    input
        .split(',')
        .filter_map(|token| token.trim().parse().ok())
        .collect()
}
```

---

## 15. Intuitive and life-saver crates

“Intuitive” and “life-saver” crates are not absolute labels, but in practice they are the crates that make Rust feel ergonomic, less error-prone, and much easier to ship safely.

### 15.1 Intuitive crates

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

### 15.2 Life-saver crates

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

### 15.3 Strong practical shortlist

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

---

## 16. Scoped and platform-specific crates

There is a third class of crates that matters enormously in real Rust work:

- domain-specific crates
- platform-specific crates
- highly scoped crates that solve one narrow problem extremely well

### 16.1 OS and system integration crates

- `windows`
- `windows-sys`
- `nix`
- `libc`
- `which`
- `directories`
- `signal-hook`
- `walkdir`
- `glob`
- `notify`
- `tempfile`

### 16.2 Data parsing and format-specific crates

- `nom`
- `pest`
- `lalrpop`
- `csv`
- `serde_yaml`
- `quick-xml`

### 16.3 Cryptography and security crates

- `ring`
- `sha2`
- `blake3`
- `aes-gcm`
- `argon2`
- `ed25519-dalek`

### 16.4 Embedded and hardware crates

- `embedded-hal`
- `heapless`
- `nb`
- `defmt`
- `cortex-m`

### 16.5 Browser and WebAssembly crates

- `wasm-bindgen`
- `web-sys`
- `js-sys`
- `gloo`
- `console_error_panic_hook`

### 16.6 General rule for scoped crates

A crate in this category is worth it if all of the following are true:

- the problem is narrow but real
- the crate solves it much better than custom code
- the crate is mature and maintained
- the problem is common enough that reimplementation would be expensive or error-prone

---

## 17. Performance and compile-time reality

One of the biggest concerns with crates is compile time.

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

---

## 18. Security and maintenance considerations

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

---

## 19. Learning roadmap

### Beginner path

1. Learn variables, ownership, borrowing, and lifetimes.
2. Write a few programs with `std` only.
3. Master `Result`, `Option`, and basic error handling.
4. Learn `cargo build`, `cargo run`, and `cargo test`.
5. Add one or two crates intentionally, such as `clap` or `serde`.
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

### Recommended crate progression

Start with:

- `clap`
- `serde`
- `serde_json`
- `thiserror`
- `anyhow`

Then move to:

- `tokio`
- `reqwest`
- `tracing`
- `axum`
- `sqlx`
- `diesel`

Then, when needed:

- `windows` / `nix` / `directories`
- `embedded-hal` / `heapless` / `defmt`
- `wasm-bindgen` / `web-sys`
- `ring` / `sha2` / `argon2`

---

## 20. Final guidance

Use the standard library first. Use crates when they solve a real, recurring, mature problem. Avoid crates when the task is small, specific, or easier to express locally.

In practical terms:

- tiny scripts -> `std`
- a real CLI -> `clap`
- JSON or config -> `serde` + `serde_json` / `toml`
- HTTP client -> `reqwest`
- async service -> `tokio` + framework crates
- embedded target -> `no_std` + hardware crates
- browser interop -> `wasm-bindgen` + `web-sys`
- system integration -> `windows`, `nix`, `directories`, `which`
- cryptography -> `ring`, `sha2`, `argon2`, `ed25519-dalek`

---

## 21. Closing statement

Rust is at its best when you combine:

- local, explicit code
- a minimal and honest dependency graph
- carefully chosen crates only where they buy real leverage

That is the real Rust philosophy: not more dependencies, but the right dependencies at the right time.

This repository is intended as a practical catalog and reference for Rust decisions. The goal is to help developers understand the trade-offs between writing direct Rust code, leaning on the standard library, and depending on crates where the ecosystem genuinely improves safety, ergonomics, or maintainability.
