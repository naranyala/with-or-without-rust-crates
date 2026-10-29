# Project Archetypes

Rust is not one-size-fits-all. The right tool depends on the kind of project you are building.

## 1. Tiny CLI utility

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

## 2. REST API service

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

## 3. Embedded system

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

## 4. Data-processing pipeline

Project type:

- CSV or log processing
- local ETL jobs
- file ingestion
- transform-and-emit workloads

Typical choice:

- `std` for smaller jobs
- selected crates for parsing, concurrency, or structured output
- `csv`, `serde`, and `regex` when useful

## 5. Internal reusable library

Project type:

- domain logic
- shared algorithms
- service semantics
- cross-project business rules

Typical choice:

- small dependency footprint
- carefully designed public API
- prefer local implementation over a broad framework

A library should not become an excuse to import a large ecosystem just because it is available.

## 6. Tooling and developer workflow projects

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

These crates are ergonomic and highly practical in toolchain work.

## 7. GUI or desktop applications

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

This is a domain-specific ecosystem and a good example of targeted dependency use.

## 8. Browser and WebAssembly apps

Project type:

- browser-side Rust logic
- interactive frontend logic
- web interop with JavaScript

Typical choice:

- `wasm-bindgen`
- `web-sys`
- `js-sys`
- `gloo`

This is another strong case for platform- and environment-specific crate choice.

## 9. Cryptographic or security-critical work

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

This is where the ecosystem is extremely important, because the value is not just convenience. It is correctness and security.

## 10. Summary

The right crate depends on the project type, not just the crate popularity.

A good Rust team tries to match the dependency footprint to the work:

- small scripts stay simple
- real services use real framework support
- embedded work uses low-level support
- browser or system integration use platform-specific crates

## 11. Final guidance

Choose the approach that keeps the code honest, explicit, and easy to reason about.

That is the core of the repo.
