# Rust Dependencies: With or Without Crates?

A practical guide to choosing between Rust's standard library, third-party crates, and `no_std` environments. The goal is not to minimize dependencies at all costs; it is to choose a design that is clear, correct, maintainable, and suitable for the project's constraints.

> **In short:** Start with the standard library when it fits. Add a crate when it provides meaningful correctness, capability, maintainability, or development-time benefits. Judge the dependency's fit and total cost—not just its popularity or line count.

## Contents

- [Quick decision guide](#quick-decision-guide)
- [What `std`, crates, and `no_std` mean](#what-std-crates-and-no_std-mean)
- [A dependency decision worksheet](#a-dependency-decision-worksheet)
- [Evaluating and maintaining dependencies](#evaluating-and-maintaining-dependencies)
- [Cargo essentials](#cargo-essentials)
- [Side-by-side: parse a configuration file](#side-by-side-parse-a-configuration-file)
- [Testing examples](#testing-examples)
- [Performance and measurement](#performance-and-measurement)
- [Common mismatches](#common-mismatches)
- [Project patterns](#project-patterns)
- [Curated crate map](#curated-crate-map)
- [Security and cryptography](#security-and-cryptography)
- [Glossary](#glossary)
- [Further reading and contribution](#further-reading-and-contribution)

## Quick decision guide

| Need | Reasonable starting point | Things to consider |
| --- | --- | --- |
| Small one-off utility | `std` | Manual argument handling is fine while it stays simple. Add a CLI crate as options, help, or validation grow. |
| User-facing CLI | [`clap`](https://crates.io/crates/clap) or [`argh`](https://crates.io/crates/argh) | Help text, validation, shell completion, and compatibility may justify a dependency. |
| JSON | [`serde`](https://crates.io/crates/serde) + [`serde_json`](https://crates.io/crates/serde_json) | Prefer established parsers and serializers over hand-written JSON handling. |
| HTTP client | [`reqwest`](https://crates.io/crates/reqwest) | Check TLS, async/blocking needs, enabled features, and target support. |
| Async network service | [`tokio`](https://crates.io/crates/tokio) plus a framework such as [`axum`](https://crates.io/crates/axum) | Async helps with I/O concurrency; it is not automatically faster for CPU-bound work. |
| Embedded or bare metal | `core`, possibly `alloc`, and ecosystem crates | Confirm target support, runtime, allocator, and hardware requirements. `no_std` does not mean “no dependencies.” |
| Configuration | `std` for a tiny private format; [`toml`](https://crates.io/crates/toml) or another parser for an established format | Parsing a standard format correctly is usually worth using a parser. |
| Database | [`sqlx`](https://crates.io/crates/sqlx), [`diesel`](https://crates.io/crates/diesel), or [`rusqlite`](https://crates.io/crates/rusqlite) | Compare async needs, compile-time checks, migrations, backend support, and operational model. |
| CSV | [`csv`](https://crates.io/crates/csv) | CSV has quoting and escaping rules; splitting on commas is only valid for a deliberately restricted format. |
| Binary protocol or grammar | A parser crate such as [`nom`](https://crates.io/crates/nom), [`winnow`](https://crates.io/crates/winnow), or a small local parser | Follow the actual grammar, handle malformed input, and test boundaries. |
| Platform APIs | Platform-specific crates such as [`windows`](https://crates.io/crates/windows), [`nix`](https://crates.io/crates/nix), or [`libc`](https://crates.io/crates/libc) | Isolate platform-specific code and test supported targets. |
| Browser WebAssembly | [`wasm-bindgen`](https://crates.io/crates/wasm-bindgen) and [`web-sys`](https://crates.io/crates/web-sys) | Browser APIs, target support, and generated bindings shape the design. |

These are starting points, not prescriptions. Crate capabilities and maintenance status change; check current documentation, releases, and advisories before adopting one.

## What `std`, crates, and `no_std` mean

### The standard library

Rust's standard library (`std`) provides facilities such as collections, filesystem and process APIs, networking primitives, synchronization, and I/O. It is often enough for small tools and domain-specific logic.

Start with `std` when the implementation is short, easy to test, and clearly correct. Reconsider when custom code begins duplicating a mature format, protocol, platform abstraction, or widely used capability.

### Third-party crates

A crate is a Rust compilation unit. Cargo packages distribute one or more crates. A dependency can bring useful functionality, but also APIs to learn, versions to resolve, features to configure, and code to build, maintain, and audit.

“No dependencies” is not inherently safer. A hand-written parser or cryptographic routine can be much riskier than a well-maintained, well-matched library. Compare the total risk and cost of both choices.

### `no_std`

`#![no_std]` opts a crate out of the Rust standard library. The crate can still use `core`; it may use `alloc` if the final program supplies an allocator. `no_std` libraries can be used in larger `std` programs, and many crates support both environments.

A `no_std` executable also needs target-specific startup/runtime and panic behavior; bare-metal projects commonly use a board support package or runtime crate. This is a **library snippet**, not a complete firmware image:

```rust
#![no_std]

pub fn checked_add(a: u32, b: u32) -> Option<u32> {
    a.checked_add(b)
}
```

For a complete embedded project, start with the [Embedded Rust Book](https://rust-embedded.org/book/) and the documentation for the target's runtime and board support crate. Verify each dependency's target and `no_std` support rather than inferring it from its name.

## A dependency decision worksheet

Copy and fill this out in an issue, pull request, or architecture note when a dependency decision is consequential:

```text
Problem and requirements:
Supported platforms and minimum Rust version (MSRV):
Input trust level / threat model:

Options considered:
1. std or local implementation:
2. Candidate crate(s):

Why the selected option fits:
Correctness and edge cases to test:
Costs (build, binary, runtime, transitive deps, license, maintenance):
Features required / features disabled:
Public API impact:
How and when we will review this decision:
```

Not every tiny dependency needs a formal record. Use the worksheet in proportion to the risk: a parser for untrusted input, a foundational library, or a dependency exposed in a public API deserves more scrutiny than a development-only color formatter.

## Evaluating and maintaining dependencies

Before adding a crate, ask:

1. **Fit:** Does it solve the actual problem and support required platforms and Rust versions?
2. **Correctness:** Does it handle edge cases, invalid input, and failure? Is a local alternative realistically as reliable?
3. **Maintenance:** Are releases, documentation, issue responses, and security advisories appropriate for the project's risk?
4. **Cost:** What are the compile-time, binary-size, runtime, licensing, and transitive-dependency costs?
5. **Features:** Can optional features be limited? Does the default feature set enable capabilities we do not need?
6. **API commitment:** Is this an internal implementation detail or a public API choice users will depend on?
7. **Alternatives:** Is there a standard-library facility, a smaller crate, or a more suitable established solution?

Do not use download counts or repository activity as the sole measure of quality. Consider your threat model and maintenance capacity.

### Dependency hygiene

- Declare only dependencies the package directly uses. Do not add a direct dependency merely to access a transitive dependency.
- Enable only needed features where practical, and inspect feature unification across the dependency graph.
- Review licenses and supported targets, especially for products distributed to others.
- Commit `Cargo.lock` for applications and binaries so collaborators and CI resolve the same dependency versions. Libraries commonly omit it so downstream users can resolve compatible versions; follow the project's policy.
- A lockfile improves consistency of dependency resolution, but does not by itself guarantee reproducible builds across toolchains, operating systems, or external inputs.
- Review dependency changes in pull requests. Tools such as `cargo tree`, `cargo audit`, and `cargo deny` can help; they complement, rather than replace, review.
- Update deliberately: review changelogs, test the change, and avoid assuming every update is behavior-free.

### A lightweight review cadence

For actively maintained applications, review dependency updates as part of normal maintenance (for example, monthly or quarterly), and respond promptly to relevant security advisories. On each review:

1. Check direct and transitive changes with `cargo tree` and the lockfile diff.
2. Check advisories and license policy using the project's chosen tooling.
3. Confirm MSRV and target compatibility for the selected versions and features.
4. Run formatting, lint, test, and build checks on supported targets where practical.
5. Record intentional pins, exclusions, or deferred upgrades with a reason and a revisit condition.

Choose a cadence appropriate to release frequency and risk; a calendar schedule is not a substitute for monitoring urgent advisories.

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

`cargo check` checks code without producing a final executable; it is not a substitute for tests or a release build. `cargo fmt --check` checks formatting without changing files. Review Clippy suggestions in context.

For dependency inspection and updates:

```sh
cargo tree
cargo tree -e features
cargo update --dry-run
```

`cargo update` updates the lockfile within the version requirements in `Cargo.toml`; it does not generally rewrite those requirements. Review and test lockfile changes before committing them.

### Minimal manifest example

Versions and features are illustrative. Select versions that fit your MSRV and verify feature requirements in each crate's documentation.

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

## Side-by-side: parse a configuration file

Suppose a private tool accepts exactly two required settings, `host` and `port`. A tiny, explicitly constrained `key=value` format can be handled locally. This implementation rejects unknown keys, missing values, duplicates, malformed lines, and invalid ports:

```rust
use std::collections::HashMap;

#[derive(Debug, PartialEq, Eq)]
struct Config {
    host: String,
    port: u16,
}

fn parse_config(input: &str) -> Result<Config, String> {
    let mut values = HashMap::new();

    for (index, line) in input.lines().enumerate() {
        let line = line.trim();
        if line.is_empty() || line.starts_with('#') {
            continue;
        }

        let (key, value) = line
            .split_once('=')
            .ok_or_else(|| format!("line {}: expected key=value", index + 1))?;
        let key = key.trim();
        let value = value.trim();

        if key.is_empty() || value.is_empty() {
            return Err(format!("line {}: key and value must be non-empty", index + 1));
        }
        if !matches!(key, "host" | "port") {
            return Err(format!("line {}: unknown key `{key}`", index + 1));
        }
        if values.insert(key, value).is_some() {
            return Err(format!("line {}: duplicate key `{key}`", index + 1));
        }
    }

    let host = values.remove("host").ok_or("missing `host`")?.to_owned();
    let port = values
        .remove("port")
        .ok_or("missing `port`")?
        .parse::<u16>()
        .map_err(|error| format!("invalid `port`: {error}"))?;

    Ok(Config { host, port })
}
```

This can be a reasonable choice if the format is private, stable, and intentionally limited. It is still custom parsing code that needs tests and future maintenance. It does not implement TOML, support quoted values, or define escaping rules.

For a standard TOML file, use a parser instead. Add `serde` with `derive` and `toml`:

```toml
[dependencies]
serde = { version = "1", features = ["derive"] }
toml = "0.8"
```

```rust
use serde::Deserialize;

#[derive(Debug, Deserialize)]
struct Config {
    host: String,
    port: u16,
}

fn parse_config(input: &str) -> Result<Config, toml::de::Error> {
    toml::from_str(input)
}
```

The crate-based version handles TOML syntax and reports deserialization errors, while the application still owns domain validation (for example, whether a host is allowed). The right comparison is not “two lines versus a dependency”; it is the cost of maintaining a custom format versus using the format users expect.

## Testing examples

Test behavior at boundaries, not only the happy path. Put unit tests near private parsing code in a `#[cfg(test)]` module; use `tests/` for tests that exercise the crate as an external consumer would.

### Unit tests for valid and invalid input

Using `parse_numbers` as an example:

```rust
fn parse_numbers(input: &str) -> Result<Vec<i32>, std::num::ParseIntError> {
    input
        .split(',')
        .map(|field| field.trim().parse())
        .collect()
}

#[cfg(test)]
mod tests {
    use super::parse_numbers;

    #[test]
    fn parses_and_trims_values() {
        assert_eq!(parse_numbers("1, 2,-3").unwrap(), vec![1, 2, -3]);
    }

    #[test]
    fn rejects_an_invalid_field_instead_of_silently_dropping_it() {
        assert!(parse_numbers("1,nope,3").is_err());
    }

    #[test]
    fn rejects_an_empty_field() {
        assert!(parse_numbers("1,,3").is_err());
    }
}
```

If the format permits empty input or empty fields, encode that policy explicitly and test it. Avoid changing parser behavior accidentally while “simplifying” code.

### Property-based tests

For parsers and transformations with many possible inputs, [`proptest`](https://crates.io/crates/proptest) can generate cases beyond a few hand-picked examples. Add it as a dev-dependency (`proptest = "1"`):

```rust
#[cfg(test)]
mod property_tests {
    use proptest::prelude::*;

    proptest! {
        #[test]
        fn parsing_a_formatted_integer_round_trips(value in any::<i32>()) {
            let text = value.to_string();
            prop_assert_eq!(text.parse::<i32>().unwrap(), value);
        }
    }
}
```

This small example demonstrates the technique; for application code, properties should express meaningful invariants of your own implementation. Property tests complement, not replace, examples and edge-case tests.

## Performance and measurement

Do not infer performance from dependency count, source-line count, or an anecdotal benchmark. Measure the actual workload and compare alternatives under the same conditions.

1. **State the question:** runtime latency, throughput, memory use, compile time, or binary size?
2. **Build a representative workload:** include realistic input sizes and distributions, not just a tiny best case.
3. **Establish a baseline:** record toolchain, target, build profile, hardware, and commands. Keep the environment and inputs the same between variants.
4. **Use suitable tools:** `cargo bench` runs benchmark targets; [`Criterion`](https://crates.io/crates/criterion) supports statistical microbenchmarks. Use OS and toolchain tools for compile-time and binary-size analysis where appropriate.
5. **Avoid common traps:** benchmark optimized builds when measuring release behavior, prevent dead-code elimination, separate setup/I/O from the measured operation when appropriate, and run enough repetitions to understand noise.
6. **Inspect trade-offs:** a faster implementation may use more memory, compile more slowly, or be harder to maintain. Report all relevant measures.
7. **Keep the benchmark reproducible:** describe commands, inputs, and environment; do not present one machine's result as universal.

Add a benchmark only when the result can inform a decision. A microbenchmark can be useful for a hot function, but end-to-end profiling is often better for finding the real bottleneck. Never invent performance claims; publish measured results with their conditions and limitations.

## Common mismatches

These are trade-offs, not universal prohibitions. Reassess when requirements change.

### Using async without an I/O-concurrency need

Tokio is useful for async I/O and services with many concurrent operations. A synchronous file transformation or short script may be clearer without an async runtime. Async is not a general CPU-performance optimization; CPU-bound work needs a different strategy.

### Using `anyhow` as a public library's error contract

`anyhow` is convenient at application boundaries. A reusable library often benefits from a concrete error type so callers can distinguish and handle documented failure cases. Consider the audience and stability of the API.

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

Use `std` for straightforward file and stream work. Prefer format-aware crates for CSV, JSON, compressed data, or other established formats. Measure before adding parallelism; it can add complexity without improving the bottleneck.

### Reusable library

Keep the public API intentional and avoid exposing implementation details unnecessarily. Consider whether dependencies appear in public types, their impact on downstream MSRV and feature selection, and whether callers need typed errors.

### Desktop, terminal, and browser apps

Choose UI frameworks according to target, accessibility, distribution, and ecosystem needs. Desktop GUI, terminal UI, and browser WebAssembly are distinct environments; a crate suited to one is not automatically suited to another.

## Curated crate map

This is a discovery list, not an endorsement, ranking, or claim that all projects need these crates. Check current documentation, maintenance, licensing, platform support, and advisories before adopting one.

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

Prefer a maintained, high-level library or protocol implementation that fits the use case. Do not design a cryptographic protocol or combine primitives based only on short examples. Keep secrets out of logs, validate inputs, use secure defaults, and seek expert review for security-critical systems. A crate provides building blocks; it does not make an application secure by itself.

## Glossary

- **Crate:** A Rust compilation unit. A Cargo package may contain a library crate, binary crates, or both.
- **Dependency:** Another package or crate that a package uses, directly or indirectly.
- **Feature:** A named Cargo configuration option that can enable optional code or dependencies. Features for the same package may be unified across a dependency graph.
- **MSRV:** Minimum Supported Rust Version—the oldest Rust compiler version a project promises to support.
- **`no_std`:** A crate configuration that does not link Rust's standard library; it can still use `core` and, with an allocator, `alloc`.
- **Transitive dependency:** A dependency required by one of your direct dependencies rather than declared and used directly by your package.
- **Lockfile:** `Cargo.lock`, which records selected dependency versions for an application or workspace resolution.
- **SemVer:** Semantic Versioning, a versioning convention; a version number alone is not a guarantee of compatibility or quality.

## Further reading and contribution

- [The Rust Book](https://doc.rust-lang.org/book/)
- [The Rust Standard Library](https://doc.rust-lang.org/std/)
- [The Cargo Book](https://doc.rust-lang.org/cargo/)
- [Rust Embedded Book](https://rust-embedded.org/book/)
- [RustSec Advisory Database](https://rustsec.org/)

Contributions are welcome when they improve accuracy, clarity, or practical usefulness. Please:

- Keep recommendations conditional and explain the context; avoid presenting a crate as universally best.
- Include links to primary documentation for new technical claims or crate entries.
- For performance claims, include reproducible methodology, environment, and measured results—or omit the claim.
- For code examples, state required dependencies/features and ensure the example's limitations are clear.
- Prefer focused changes; avoid adding crate names without explaining when they are useful.

This guide is intentionally opinionated, but every recommendation depends on the project's requirements, threat model, supported platforms, and maintenance capacity. Revisit dependency choices as those constraints change.
