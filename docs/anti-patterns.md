# Anti-Patterns and Misused Crates

This is one of the most important sections of the project: the crates that are technically usable, but are a poor fit for the task or the architecture.

## 1. The main trap

A dependency is not automatically a win just because it reduces boilerplate.

The real question is not:

- “Can I use this crate?”

It is:

- “Does this crate improve the design, or simply hide the design?”

## 2. Common wrong-usage patterns

### 2.1 Using async everywhere

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

### 2.2 Using `anyhow` too broadly

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

### 2.3 Using `clap` for tiny utilities

`clap` is great for real CLIs.

It is overkill for:

- scripts with two flags
- internal tools with a handful of positional args
- quick prototypes

Wrong pattern:

- “I used `clap` because it is the default”
- “It is popular, so it must be correct for this case”

If the project is tiny, `std::env::args()` and a few manual checks may be clearer.

### 2.4 Using `serde` for trivial fixed formats

`serde` is one of the best crates in Rust.

But it is not always the right answer when:

- the file format is tiny and rigid
- the format is simple enough to parse with `split` or a few direct checks
- the schema is fixed and does not need a general data model

Wrong pattern:

- reading a tiny config file with a large `serde` struct graph when a few simple `String` splits are enough
- forcing a structured data model onto a custom format that is not actually complex

### 2.5 Using a web framework for a trivial tool

Frameworks like `axum`, `actix-web`, and `rocket` are excellent for APIs.

They are a poor fit when:

- a project is really a CLI utility
- there is only one endpoint and the code is not a full web product
- the app does not need routing, middleware, request typing, or web ergonomics

Wrong pattern:

- building a tiny service with an application framework before proving the workload is genuinely web-heavy

### 2.6 Choosing a crate just because it is famous

Popular crates are not automatically correct choices.

Wrong pattern:

- “Everyone uses `X`, so I should use it too”
- “This crate has a million downloads, so it must be the right abstraction”

A dependency should match the actual architecture and workload, not just the popularity list.

## 3. Misleading ergonomics

Some crates are ergonomic in the short-term but create long-term confusion.

Examples:

- `anyhow` in a library API
- `tokio` in a mostly sequential tool
- `serde` applied to a format that does not benefit from structured modeling
- `axum` in a project that only does a single task and exits

The danger is not broken code. The danger is code that works but is harder to reason about than the original problem.

## 4. Maintenance and dependency hygiene problems

A bad dependency decision often looks like this:

- a crate solves a tiny problem but brings a big graph
- a crate is not actively maintained
- the dependency relies on many transitive libraries for a narrow task
- the project is now more expensive to build, audit, and understand than the original problem

Look out for:

- large transitive trees
- sparse or unclear documentation
- stale issue history
- a mismatch between “library power” and project size

## 5. Practical examples of bad decisions

### Example A: giant async runtime for a short-lived script

```rust
#[tokio::main]
async fn main() {
    let content = std::fs::read_to_string("input.txt").unwrap();
    println!("{}", content);
}
```

This is technically valid, but it is a lot of runtime complexity for a simple synchronous script.

### Example B: `anyhow` everywhere

```rust
fn parse_config() -> anyhow::Result<Config> {
    // fine for a tool, bad for a reusable library API
}
```

### Example C: `serde` for a trivial config file

```rust
// fine for a real config system
// bad when the config format is a couple of fixed key-value pairs
```

### Example D: framework for trivial HTTP

```rust
// good for a real API service
// poor for a one-off endpoint or small utility
```

## 6. A better rule of thumb

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

## 7. Final principle

The most dangerous crate is not the broken crate.
It is the crate that is technically working, widely used, and still a poor fit.

The best Rust designs usually stay local, explicit, and honest about complexity.
