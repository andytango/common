# Rust Project Setup Instructions

Set up a Rust development environment with strict linting, comprehensive type checking, and automatic code formatting.

## Tools Configured

- **Rust**: Strict compiler warnings with all strict options enabled
- **Clippy**: Code quality and correctness enforcement with pedantic rules
- **rustfmt**: Automatic code formatting
- **cargo-tarpaulin**: Code coverage reporting

## Installation

```bash
cargo install cargo-tarpaulin
```

## Project Structure

```bash
mkdir -p tmp && touch tmp/.gitkeep
# Also create credentials/ and logs/ if needed
```

Add to `.gitignore`: `tmp/*`, `!tmp/.gitkeep` (and similar for credentials/logs if used)

## Configuration Details

### Cargo.toml

Maximum strictness configuration with lint groups:

```toml
[package]
name = "your-project"
version = "0.1.0"
edition = "2021"

[lints.rust]
unsafe_code = "warn"
missing_docs = "warn"
rust_2018_idioms = "warn"
trivial_casts = "warn"
trivial_numeric_casts = "warn"
unused_lifetimes = "warn"
unused_qualifications = "warn"

[lints.clippy]
all = "warn"
pedantic = "warn"
nursery = "warn"
cargo = "warn"
# Production safety
unwrap_used = "deny"
expect_used = "warn"
panic = "warn"
# Code quality
missing_errors_doc = "warn"
missing_panics_doc = "warn"
```

### rustfmt.toml

Standard formatting configuration:

```toml
edition = "2021"
max_width = 100
tab_spaces = 4
use_small_heuristics = "Default"
imports_granularity = "Module"
group_imports = "StdExternalCrate"
reorder_imports = true
reorder_modules = true
```

### .cargo/config.toml (optional stricter defaults)

```toml
[build]
rustflags = ["-D", "warnings"]

[alias]
lint = "clippy --all-targets --all-features -- -D warnings"
```

## Makefile / justfile Scripts

Add these scripts to your project:

```makefile
.PHONY: build test lint format format-check coverage check

build:
	cargo build --release

test:
	cargo test

lint:
	cargo clippy --all-targets --all-features -- -D warnings

format:
	cargo fmt

format-check:
	cargo fmt --check

coverage:
	cargo tarpaulin --out Html --fail-under 95

check: format lint test build
```

## Required Workflow

After making any code changes, run:

```bash
cargo fmt          # Format code
cargo clippy       # Check linting
cargo test         # Run tests
cargo build        # Verify compilation
```

Or run all at once: `make check`

## Documentation Requirements

All public items require documentation comments:

```rust
/// Calculates the sum of two numbers.
///
/// # Arguments
///
/// * `a` - The first number
/// * `b` - The second number
///
/// # Returns
///
/// The sum of `a` and `b`
///
/// # Examples
///
/// ```
/// let result = add(2, 3);
/// assert_eq!(result, 5);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

/// Configuration for the application.
pub struct Config {
    /// The port to listen on.
    pub port: u16,
    /// The host address.
    pub host: String,
}
```

## Error Handling Requirements

Use `Result<T, E>` with `thiserror` for all fallible operations:

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
    #[error("Invalid configuration: {msg}")]
    Config { msg: String },
}

pub fn read_config(path: &str) -> Result<Config, AppError> {
    // Implementation using ? operator
}
```
