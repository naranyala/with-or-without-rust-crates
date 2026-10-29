# Learning Roadmap

This repo is intentionally practical. A good learning path should start with the standard library and only add crates when they solve a real problem.

## Beginner path

1. Learn variables, ownership, borrowing, and lifetimes.
2. Write a few programs with `std` only.
3. Master `Result`, `Option`, and basic error handling.
4. Learn `cargo build`, `cargo run`, and `cargo test`.
5. Add one or two crates intentionally, such as `clap` or `serde`.
6. Learn how `Cargo.toml` and feature flags work.

## Intermediate path

1. Understand workspaces and library layouts.
2. Learn async with `tokio`.
3. Build a small API server with `axum`.
4. Use structured logging and tracing.
5. Learn dependency hygiene and crate auditing.

## Advanced path

1. Work with `no_std` and embedded targets.
2. Build reusable libraries with careful API design.
3. Keep compile times under control.
4. Design long-lived error and data models.
5. Understand semver, maintenance, and ecosystem maturity.

## Recommended crate progression

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

## Best practice in one sentence

Learn the standard library first, then add dependency power deliberately, only when the problem is real and the crate is worth the complexity.
