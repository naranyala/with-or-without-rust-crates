# Rust Crates Catalog

A practical, opinionated guide to understanding Rust in both directions:

- building with external crates from the Rust ecosystem
- building without crates at all
- deciding when each approach is the right tool for the job

This repository exists to help Rust developers, learners, and teams make thoughtful decisions about dependency use, project architecture, performance, safety, security, and maintainability.

## Why this repo exists

Rust is a language of clarity, safety, and control. It has an excellent standard library, a rich ecosystem of reusable libraries, and a culture of building small, explicit, correct systems.

The central question is not:

- “Should I always use crates?”
- “Should I avoid crates entirely?”

The real question is:

- “What is the smallest and best design for this project?”

This project helps answer that question in a practical, engineering-minded way.

## Start here

- [Decision framework](docs/decision-framework.md)
- [Crate catalog](docs/crate-catalog.md)
- [Misuse and anti-patterns](docs/anti-patterns.md)
- [Project archetypes](docs/project-archetypes.md)
- [Learning roadmap](docs/roadmap.md)

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

## Core idea

Use the standard library first. Use crates when they solve a real, recurring, mature problem. Avoid crates when the task is small, specific, or easier to express locally.

## Rust crate families to know

This repo keeps the important ecosystem buckets, without assuming every project needs all of them:

- CLI: `clap`, `argh`, `dialoguer`
- Serialization: `serde`, `serde_json`, `toml`, `bincode`, `ron`
- HTTP and networking: `reqwest`, `hyper`, `tokio`, `axum`, `tower`
- Logging and tracing: `tracing`, `log`, `env_logger`
- Error handling: `thiserror`, `anyhow`, `eyre`
- Database: `sqlx`, `diesel`, `rusqlite`
- Async: `tokio`, `async-std`, `futures`
- Embedded / `no_std`: `embedded-hal`, `heapless`, `nb`, `defmt`
- Testing: `criterion`, `proptest`, `insta`
- Platform-specific: `windows`, `windows-sys`, `nix`, `libc`, `which`, `directories`
- Web/wasm: `wasm-bindgen`, `web-sys`, `js-sys`, `gloo`
- Cryptography: `ring`, `sha2`, `blake3`, `aes-gcm`, `argon2`, `ed25519-dalek`

## A few practical rules

- Prefer the standard library when it is enough.
- Add crates when they solve a shared, stable problem.
- Avoid dependency sprawl when a local implementation is clearer.
- Keep the dependency graph honest and minimal.
- In embedded or constrained environments, `no_std` may be the right answer.

## Recommended repo flow

1. Read the [decision framework](docs/decision-framework.md)
2. Browse the [crate catalog](docs/crate-catalog.md)
3. Review the [common anti-patterns](docs/anti-patterns.md)
4. Check the [project archetypes](docs/project-archetypes.md)
5. Use the [roadmap](docs/roadmap.md) to learn progressively

## Final takeaway

Rust is at its best when you combine:

- local, explicit code
- a minimal and honest dependency graph
- carefully chosen crates only where they buy real leverage

The right question is not “Should I use crates?” but “What is the right dependency footprint for this project?”

---

This repository is structured as a practical catalog and reference for Rust decisions. The documentation is intentionally split so the project stays approachable while still preserving the rich Rust crate ecosystem examples.
