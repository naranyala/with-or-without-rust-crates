# Rust Dependencies: With or Without Crates?

A practical guide to choosing between Rust's standard library, third-party crates, and `no_std` environments. The goal is not to minimize dependencies at all costs; it is to choose the smallest, clearest, and safest design that meets the project's requirements.

> **In short:** Start with the standard library when it fits. Add a crate when it provides meaningful correctness, capability, maintainability, or development-time benefits. Review the dependency's cost and fit—not just its popularity.

## Contents

- [A quick decision guide](#a-quick-decision-guide)
- [What `std`, crates, and `no_std` mean](#what-std-crates-and-no_std-mean)
- [Evaluating a dependency](#evaluating-a-dependency)
- [Cargo essentials](#cargo-essentials)
- [Examples: standard library and crates](#examples-standard-library-and-crates)
- [Common mismatches](#common-mismatches)
- [Project patterns](#project-patterns)
- [Curated crate map](#curated-crate-map)
- [Security and cryptography](#security-and-cryptography)
- [Further reading](#further-reading)

## A quick decision guide

| Need | Reasonable starting point | Things to consider |
| --- | --- | --- |
| Small one-off utility | `std` | Manual argument handling is fine while it stays simple. Add a CLI crate as options, help, or validation grow. |
| User-facing CLI | [`clap`](https://crates.io/crates/clap) or [`argh`](https://crates.io/crates/argh) | Shell completion, help text, validation, and compatibility may justify a dependency. |
| JSON | [`serde`](https://crates.io/crates/serde) + [`serde_json`](https://crates.io/crates/serde_json) | Prefer established parsers and serializers over hand-written JSON handling. |
| HTTP client | [`reqwest`](https://crates.io/crates/reqwest) | Check TLS, async/blocking needs, enabled features, and target support. |
| Async network service | [`tokio`](https://crates.io/crates/tokio) plus a framework such as [`axum`](https://crates.io/crates/axum) | Async is useful for I/O concurrency; it is not automatically faster for CPU-bound work. |
| Embedded or bare metal | `core`, possibly `alloc`, and ecosystem crates | Confirm target support, runtime, allocator, and hardware requirements. `no_std` is not a synonym for “no dependencies.” |
| Configuration | `std` for a tiny private format; [`toml`](https://crates.io/crates/toml) or another parser for a real format | Parsing a standard format correctly is usually worth using a parser. |
| Database | [`sqlx`](https://crates.io/crates/sqlx), [`diesel`](https://crates.io/crates/diesel), or [`rusqlite`](https://crates.io/crates/rusqlite) | Compare async needs, compile-time checks, migrations, backend support, and operational model. |
| CSV | [`csv`](https://crates.io/crates/csv) | CSV has quoting and escaping rules; splitting on commas is only valid for a deliberately restricted format. |
| Binary protocol or grammar | A documented parser crate such as [`nom`](https://crates.io/crates/nom), [`winnow`](https://crates.io/crates/winnow), or a small local parser | Follow the actual grammar, handle malformed input, and test boundaries. |
| Platform APIs | Platform-specific crates such as [`windows`](https://crates.io/crates/windows), [`nix`](https://crates.io/crates/nix), or [`libc`](https://crates.io/crates/libc) | Isolate platform-specific code and test on the supported targets. |
| Browser WebAssembly | [`wasm-bindgen`](https://crates.io/crates/wasm-bindgen) and [`web-sys`](https://crates.io/crates/web-sys) | Browser APIs, target support, and generated bindings shape the design. |

These are starting points, not prescriptions. A crate's maintenance status and capabilities can change; check its current documentation and release history before adopting it.

## What `std`, crates, and `no_std` mean

### The standard library

Rust's standard library (`std`) provides common facilities such as collections, filesystem and process APIs, networking primitives, synchronization, and I/O. It is often enough for small tools and domain-specific logic.

Start with `std` when the implementation is short, easy to test, and clearly correct. Reconsider that choice when custom code begins duplicating a mature format, protocol, platform abstraction, or widely used capability.

### Third-party crates

A crate is a Rust compilation and distribution unit. Cargo packages often provide one or more crates. A dependency can bring useful functionality, but also brings APIs to learn, versions to resolve, features to configure, and code to build, maintain, and audit.

A good dependency should improve the overall design—not merely reduce line count. “No dependencies” is not inherently safer: a hand-written parser or cryptographic routine can be much riskier than a well-maintained, well-matched library.

### `no_std`

`#![no_std]` opts a crate out of the Rust standard library. The crate can still use `core`; it may use `alloc` if the final program supplies an allocator. `no_std` libraries can be used in larger `std` programs, and many crates support both environments.

A `no_std` executable also needs target-specific startup/runtime and panic behavior; bare-metal projects commonly use a board support package or runtime crate. The small example below is a **library snippet**, not a complete firmware image.

```rust
#![no_std]

pub fn checked_add(a: u32, b: u32) -> Option<u32> {
    a.checked_add(b)
}
```

## Evaluating a dependency

Before adding a crate, ask:

1. **Fit:** Does it solve the actual problem and support the platforms and Rust versions we need?
2. **Correctness:** Does it correctly handle edge cases, invalid input, and failure? Is the alternative realistically as reliable?
3. **Maintenance:** Are releases, documentation, issue responses, and security advisories appropriate for this project's risk?
4. **Cost:** What are the compile-time, binary-size, runtime, licensing, and transitive-dependency costs?
5. **Features:** Can optional features be limited? Does the default feature set enable capabilities we do not need?
6. **API commitment:** Is this an internal implementation detail or a public API choice that users will depend on?
7. **Alternatives:** Is there a standard-library facility, a smaller crate, or a more suitable established solution?

Do not use download counts or repository activity as the sole measure of quality. Consider the project's threat model and maintenance capacity. For critical dependencies, record why the dependency is needed and what would trigger replacing or updating it.

### Dependency hygiene

- Add only direct dependencies that the project uses; avoid dependency duplication when one crate already fits.
- Enable only needed features where practical, and inspect feature unification across the dependency graph.
- Review licenses and supported targets, especially for products distributed to others.
- Commit `Cargo.lock` for applications and binaries so collaborators and CI resolve the same dependency versions. Libraries commonly omit it so downstream users can resolve compatible versions themselves; workspace policy may vary.
- A lockfile improves consistency of dependency resolution, but does not by itself make builds reproducible across different toolchains, operating systems, or external inputs.
- Review dependency changes in pull requests. Consider tools such as `cargo tree`, `cargo audit`, and `cargo deny` as part of an appropriate project workflow; they complement, rather than replace, review.
- Keep Rust and dependencies updated deliberately. Review the changelog and test the change instead of assuming every update is behavior-free.

## Cargo essentials

Create a binary project and run common checks:

```sh
cargo new my_app
cd my_app
cargo check
cargo test
cargo fmt --check
cargo clippy --all-targets
cargo run
```

`cargo check` checks code without producing a final executable; it is not a substitute for tests or a release build. `cargo fmt --check` checks formatting without changing files. `cargo clippy` reports lints; review suggestions in context.

For dependency inspection and updates:

```sh
cargo tree
cargo tree -e features
cargo update --dry-run
```

`cargo update` updates the lockfile within the version requirements in `Cargo.toml`; it does not generally rewrite those requirements. Review and test lockfile changes before committing them.

### Minimal manifest example

Versions and features below are illustrative. Select versions that fit your MSRV and project, and verify feature requirements in each crate's documentation.

```toml
[package]
name = "my_app"
version = "0.1.0"
edition = "2024"
rust-version = "1.85"

[dependencies]
serde = { version = "1", features = ["derive"] }
serde_json = "1"

[dev-dependencies]
pretty_assertions = "1"
```

- `[dependencies]` are available to the package's normal targets.
- `[dev-dependencies]` are for tests, examples, and benchmarks as applicable.
- Features select optional crate functionality; Cargo may unify features for the same dependency across the graph.
- A workspace groups related packages and can share dependency declarations and build settings.
- `rust-version` declares the minimum supported Rust version (MSRV) when the project has one; verify that dependencies and features respect it.

## Examples: standard library and crates

### Reading a file with useful error context

```rust
use std::{env, error::Error, fs};

fn run() -> Result<(), Box<dyn Error>> {
    let path = env::args().nth(1).ok_or("usage: app <path>")?;
    let contents = fs::read_to_string(&path)?;
    println!("Read {} bytes from {path}", contents.len());
    Ok(())
}

fn main() {
    if let Err(error) = run() {
        eprintln!("error: {error}");
        std::process::exit(1);
    }
}
```

This is enough for a small example. For a production CLI, decide how errors should be reported and whether a CLI or error-handling crate would improve usability.

### Parsing a simple delimited format

For a format that is explicitly defined as comma-separated integers with no quoting or escaping, a small parser can be appropriate:

```rust
fn parse_numbers(input: &str) -> Result<Vec<i32>, std::num::ParseIntError> {
    input
        .split(',')
        .map(|field| field.trim().parse())
        .collect()
}
```

This rejects invalid fields instead of silently dropping them. It is **not a general CSV parser**: real CSV supports quoting, escaped quotes, and other rules. Use the `csv` crate for general CSV input.

### JSON with Serde

Add `serde` with its `derive` feature and `serde_json` to the manifest, then define the data shape:

```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
struct User {
    name: String,
    age: u32,
}

fn main() -> Result<(), serde_json::Error> {
    let input = r#"{"name":"Ada","age":37}"#;
    let user: User = serde_json::from_str(input)?;
    println!("{user:?}");
    Ok(())
}
```

Serde and a format crate handle serialization mechanics; application code still needs to validate domain rules, such as age ranges or authorization.

### CLI parsing with `clap`

Add `clap` with its `derive` feature:

```toml
clap = { version = "4", features = ["derive"] }
```

```rust
use clap::Parser;

#[derive(Debug, Parser)]
#[command(version, about = "Greets a person")]
struct Args {
    /// Name to greet
    #[arg(long)]
    name: String,
}

fn main() {
    let args = Args::parse();
    println!("Hello, {}!", args.name);
}
```

For a tiny private utility, `std::env::args` may be simpler. For a distributed CLI, generated help, validation, and consistent behavior can make `clap` worthwhile.

### Minimal async HTTP server

This example uses `axum` 0.8 and Tokio. Add compatible versions of both crates; Tokio needs the macros, networking, and runtime features used by the example.

```toml
axum = "0.8"
tokio = { version = "1", features = ["macros", "rt-multi-thread", "net"] }
```

```rust
use axum::{routing::get, Router};
use std::error::Error;

#[tokio::main]
async fn main() -> Result<(), Box<dyn Error>> {
    let app = Router::new().route("/", get(|| async { "hello" }));
    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000").await?;
    axum::serve(listener, app).await?;
    Ok(())
}
```

This binds to loopback for local demonstration. A production service also needs deliberate decisions about configuration, graceful shutdown, observability, timeouts, resource limits, and deployment security.

## Common mismatches

These are trade-offs, not universal prohibitions. Reassess when requirements change.

### Using async without an I/O-concurrency need

Tokio is useful for async I/O and services with many concurrent operations. A synchronous file transformation or short script may be clearer without an async runtime. Async is not a general CPU-performance optimization; CPU-bound work usually needs a different strategy.

### Using `anyhow` as a public library's error contract

`anyhow` is convenient at application boundaries. A reusable library often benefits from a concrete error type so callers can distinguish and handle documented failure cases. This is a design choice, not a rule: consider the audience and stability of the API.

### Hand-parsing a standard format

A few `split` calls can be right for a tiny private format with a precisely limited grammar. They are not substitutes for complete CSV, JSON, URL, or protocol parsers. If input can be malformed or adversarial, correctness and resource limits matter.

### Choosing a framework before understanding the problem

A web framework is appropriate for a web application that benefits from routing, middleware, and request/response abstractions. It is usually unnecessary for a command-line tool that does not serve HTTP. Conversely, hand-building a production HTTP stack to avoid dependencies is rarely a good trade.

### Choosing based on popularity alone

A popular crate can still be the wrong fit, and a small crate can be excellent. Evaluate current maintenance, compatibility, security posture, license, feature set, and your ability to support it.

### Treating fewer dependencies as automatically safer

Fewer dependencies can reduce maintenance and review work. But custom replacements for mature parsers, TLS, cryptography, or database drivers can introduce substantially more risk. Compare total risk, not dependency count alone.

## Project patterns

### Small CLI or local utility

Start with `std` for a simple argument, file, and output flow. Add `clap`, `serde`, `csv`, or another focused crate when requirements justify them. Make errors actionable rather than relying on `unwrap()` for ordinary external input.

### Service or API

A common combination is a runtime such as Tokio, a framework such as Axum, Serde for data formats, and a database library if needed. Keep domain logic independent of transport where practical; choose timeouts, cancellation, input validation, and observability deliberately.

### Embedded or bare-metal application

Check the target triple, runtime, allocator, panic strategy, hardware support, and `no_std` compatibility of every dependency. Use abstractions that help test hardware-facing logic, and keep target-specific code at clear boundaries.

### Data pipeline

Use `std` for straightforward file and stream work. Prefer format-aware crates for CSV, JSON, compressed data, or other established formats. Measure before adding parallelism; it may add complexity without improving the actual bottleneck.

### Reusable library

Keep the public API intentional and avoid exposing implementation details unnecessarily. Consider whether dependencies appear in public types, their impact on downstream MSRV and feature selection, and whether callers need typed errors.

### Desktop, terminal, and browser apps

Choose UI frameworks according to target, accessibility, distribution, and ecosystem needs. Desktop GUI, terminal UI, and browser WebAssembly are distinct environments; a crate suited to one is not automatically suited to another.

## Curated crate map

This is a discovery list, not an endorsement, ranking, or claim that all projects need these crates. Check current crate documentation, maintenance, licensing, platform support, and security advisories before adopting one.

| Area | Examples |
| --- | --- |
| CLI and prompts | [`clap`](https://crates.io/crates/clap), [`argh`](https://crates.io/crates/argh), [`dialoguer`](https://crates.io/crates/dialoguer) |
| Serialization and formats | [`serde`](https://crates.io/crates/serde), [`serde_json`](https://crates.io/crates/serde_json), [`toml`](https://crates.io/crates/toml), [`csv`](https://crates.io/crates/csv), [`bincode`](https://crates.io/crates/bincode), [`quick-xml`](https://crates.io/crates/quick-xml) |
| HTTP and networking | [`reqwest`](https://crates.io/crates/reqwest), [`hyper`](https://crates.io/crates/hyper), [`axum`](https://crates.io/crates/axum), [`tower`](https://crates.io/crates/tower), [`rustls`](https://crates.io/crates/rustls), [`quinn`](https://crates.io/crates/quinn), [`tokio-tungstenite`](https://crates.io/crates/tokio-tungstenite) |
| Async and concurrency | [`tokio`](https://crates.io/crates/tokio), [`futures`](https://crates.io/crates/futures), [`rayon`](https://crates.io/crates/rayon), [`crossbeam`](https://crates.io/crates/crossbeam) |
| Logging and diagnostics | [`tracing`](https://crates.io/crates/tracing), [`log`](https://crates.io/crates/log), [`env_logger`](https://crates.io/crates/env_logger), [`indicatif`](https://crates.io/crates/indicatif) |
| Errors | [`thiserror`](https://crates.io/crates/thiserror), [`anyhow`](https://crates.io/crates/anyhow), [`eyre`](https://crates.io/crates/eyre) |
| Databases | [`sqlx`](https://crates.io/crates/sqlx), [`diesel`](https://crates.io/crates/diesel), [`rusqlite`](https://crates.io/crates/rusqlite) |
| Parsing and grammars | [`nom`](https://crates.io/crates/nom), [`winnow`](https://crates.io/crates/winnow), [`pest`](https://crates.io/crates/pest), [`lalrpop`](https://crates.io/crates/lalrpop), [`regex`](https://crates.io/crates/regex) |
| Testing and benchmarks | [`proptest`](https://crates.io/crates/proptest), [`insta`](https://crates.io/crates/insta), [`criterion`](https://crates.io/crates/criterion), [`tempfile`](https://crates.io/crates/tempfile) |
| Cryptography | [`ring`](https://crates.io/crates/ring), [`sha2`](https://crates.io/crates/sha2), [`blake3`](https://crates.io/crates/blake3), [`aes-gcm`](https://crates.io/crates/aes-gcm), [`argon2`](https://crates.io/crates/argon2), [`ed25519-dalek`](https://crates.io/crates/ed25519-dalek) |
| Embedded | [`embedded-hal`](https://crates.io/crates/embedded-hal), [`heapless`](https://crates.io/crates/heapless), [`defmt`](https://crates.io/crates/defmt), [`cortex-m`](https://crates.io/crates/cortex-m) |
| Operating systems and files | [`windows`](https://crates.io/crates/windows), [`windows-sys`](https://crates.io/crates/windows-sys), [`nix`](https://crates.io/crates/nix), [`libc`](https://crates.io/crates/libc), [`directories`](https://crates.io/crates/directories), [`walkdir`](https://crates.io/crates/walkdir), [`notify`](https://crates.io/crates/notify) |
| Browser and WebAssembly | [`wasm-bindgen`](https://crates.io/crates/wasm-bindgen), [`web-sys`](https://crates.io/crates/web-sys), [`js-sys`](https://crates.io/crates/js-sys), [`gloo`](https://crates.io/crates/gloo) |
| GUI and terminal UI | [`egui`](https://crates.io/crates/egui), [`iced`](https://crates.io/crates/iced), [`tauri`](https://crates.io/crates/tauri), [`slint`](https://crates.io/crates/slint), [`crossterm`](https://crates.io/crates/crossterm) |

## Security and cryptography

Cryptographic primitives are not interchangeable. A fast general-purpose hash such as SHA-256 or BLAKE3 is not a password-hashing function. Password storage needs a password-hashing scheme such as Argon2 with appropriate parameters and salts. Encryption, authentication, key management, and signatures have different requirements.

Prefer a maintained, high-level library or protocol implementation that fits the use case. Do not design a cryptographic protocol or combine primitives based only on short examples. Keep secrets out of logs, validate inputs, use secure defaults, and seek expert review for security-critical systems. A crate can provide useful building blocks; it does not make an application secure by itself.

## Further reading

- [The Rust Book](https://doc.rust-lang.org/book/)
- [The Rust Standard Library](https://doc.rust-lang.org/std/)
- [The Cargo Book](https://doc.rust-lang.org/cargo/)
- [Rust Embedded Book](https://rust-embedded.org/book/)
- [RustSec Advisory Database](https://rustsec.org/)

---

This guide is intentionally opinionated, but every recommendation depends on the project's requirements, threat model, supported platforms, and maintenance capacity. Revisit dependency choices as those constraints change.
