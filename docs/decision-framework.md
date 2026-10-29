# Decision Framework

This page is the core of the repository: it explains how to decide between `std`, `no_std`, and external crates.

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

## 5. The Cargo ecosystem

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

## 11. Practical decision guide

If the task is:

- a tiny script -> `std`
- a custom parser -> manual logic or a tiny parser crate
- a CLI with real flags -> `clap`
- JSON config or API -> `serde` + `serde_json`
- async service -> `tokio` + framework crates
- firmware -> `no_std` + embedded crates
- a library with domain logic -> keep dependencies minimal

## 12. Final rule

The best Rust project is not the one with the most crates. It is the one with the smallest honest abstraction that matches the task.

Use crates to reduce real friction. Keep code local when the design is simple and explicit.
