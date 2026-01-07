# Rust Development Guidelines

## Core Principles

- Use the simplest possible solution
- Prefer procedural programming and functional programming paradigms
- **Simplicity**: Prioritize **Cognitive Simplicity** over **Code Brevity**. Explicit is better than implicit. Unnested code is better than nested. Code that can be read top-to-bottom is better than clever abstractions. Service Oriented Architecture is intended for COMPLEX projects. For simple tasks, keep it simple.
- **Conflict Resolution**: If "Simplicity" conflicts with "Robustness" (e.g., Error Handling or SRP), **Robustness wins**. It is better to have verbose, safe, and testable code than concise but fragile code.
- Use well-maintained, mature crates where possible (check crates.io downloads and recent activity)
- If you feel a need to use complex trait hierarchies or excessive polymorphism, ask the user about your design first
- Ensure you include documentation comments for all public items
- Run clippy as frequently as possible to detect any issues and fix them
- Avoid overriding the linter or using language escape hatches such as:
  - `unsafe` blocks without thorough justification
  - `#[allow(clippy::...)]` or `#[allow(warnings)]` attributes
  - `unwrap()` in production code
  - Suppressing compiler warnings
- **Warning Suppression Protocol**: If you believe suppressing a clippy or compiler warning is genuinely necessary:
  1. You MUST first attempt to fix the underlying issue
  2. You MUST document the specific reason why suppression is unavoidable (e.g., false positive, external API constraint)
  3. You MUST ask the user for approval before adding any `#[allow(...)]` attribute
  4. Use the most specific allow attribute possible (e.g., `#[allow(clippy::needless_return)]` not `#[allow(clippy::all)]`)
  5. Add a comment explaining why the suppression is necessary
- After you have finished your implementation:
  - Run `cargo build` to check for compilation errors
  - Run `cargo test` to execute tests
  - Run `cargo clippy` for linting
  - If you don't know how to test your implementation, ask the user for help
- When handling unknown data structures, use serde for serialization/deserialization with proper validation

## Pre-Implementation Design Protocol

**Analysis & Planning**: For architectural changes, new services, or complex logic, you MUST first output a brief plan. This plan serves as the basis for the **Design Review** you conduct with the user. It should explicitly address:
1.  **Safety**: Identify potential edge cases or error states.
2.  **Architecture**: Verify if Single Responsibility Principle (SRP) is maintained. If a "mode" or variation is detected, explicitly list the strategy traits/structs to be created.
3.  **Result Definition**: Define the specific `Result<T, E>` types that will be returned, specifying the `thiserror` definitions for `E`.
4.  **External API Exploration**: Before implementing external API integrations, manually test the API (e.g., using curl) to understand actual response shapes and document them.

## Architecture & Design

- **Single Responsibility Principle**: Each module, struct, trait, or function should have one, and only one, reason to change. Decompose large, multi-purpose components into smaller, focused ones.
- **Composition over Configuration/Inheritance**: Favor composition to define behavior. If a component operates in different "modes" with overlapping but distinct dependencies or logic, do not use internal flags or configuration options to switch behavior. Instead, extract shared dependencies and create distinct implementations (strategies) for each mode.
- **Service Oriented Architecture**: For **COMPLEX** projects, decompose systems into small, individually testable services located under a `lib` folder.
- **Implementation Style**: When implementing services, prefer a **procedural** or **functional** style over complex object-oriented patterns.
- **Dependency Structure**: Design services with orthogonality, layering, and proper abstraction levels in mind. Tend towards a tree or diamond dependency pattern. Apply **SOLID principles**.
- **Adapter Pattern**: Always wrap external APIs or services in an adapter service with minimal logic.
- **Design Review**: You MUST always have the user review your service design before implementation.
- **Test Confirmation**: You MUST always ask the user if they would like tests to be added when performing a task.
- **Manual Testing**: If the user declines automated tests, you MUST try to test manually. If you don't know how to test manually, you MUST ask the user for help.

## Anti-patterns

- **Mode Flags**: Avoid functions or structs that take a "mode" enum or boolean flag to significantly alter their control flow or dependencies.
  - **Exception**: Simple configuration flags that do not drastically change logic (e.g., `verbose`, `dry_run`) are acceptable.
  - **Bad**:
    ```rust
    enum Mode { Fast, Safe }
    fn process(data: Data, mode: Mode) {
        match mode {
            Mode::Fast => { /* fast logic */ }
            Mode::Safe => { /* safe logic */ }
        }
    }
    ```
  - **Good**: **Strongly Prefer** the Strategy Pattern by defining a `Processor` trait and implementing `FastProcessor` and `SafeProcessor`. Use the trait bound or dynamic dispatch to invoke the correct behavior.
- **Testing Against Mental Model**: Avoid writing tests that validate assumptions rather than real API contracts. Tests that pass based on expected behavior but fail against actual external system responses indicate this anti-pattern.
  - **Remediation**: Test adapters with real API calls whenever practical. When real calls are impractical, use fixture-captured real responses. Unit test all string manipulation (URL construction, ID parsing).

## Rust-Specific Guidelines

### Memory Safety and Ownership

- Understand and follow Rust's ownership rules
- Prefer borrowing (`&T`) over cloning when possible
- Use `Arc` and `Rc` judiciously for shared ownership
- Avoid reference cycles (use `Weak` when needed)
- Prefer stack allocation over heap allocation where possible

### Error Handling

- **Never use `unwrap()` or `expect()` in production code** outside of tests
- Use the `?` operator for error propagation
- **Library Code (`thiserror`)**: For libraries and shared modules, you MUST use `thiserror` to define structured, distinct error types. This allows consumers to handle specific failure cases programmatically.
  ```rust
  use thiserror::Error;

  #[derive(Error, Debug)]
  pub enum AppError {
      #[error("IO error: {0}")]
      Io(#[from] std::io::Error),
      #[error("Invalid configuration: {msg}")]
      Config { msg: String },
  }
  ```
- **Application Code (`anyhow`)**: For binary entry points (`main.rs`) or top-level application logic, you MAY use `anyhow` to handle errors where the specific type doesn't matter as much as the report.
- Use `Result<T, E>` as return type for fallible operations
- Consider using `Option<T>` for values that might not exist

### Type System

- Leverage Rust's type system for compile-time guarantees
- Use newtype pattern for domain modeling:
  ```rust
  struct UserId(u64);
  struct Email(String);
  ```
- Prefer enums for state machines and variants
- Use phantom types for additional compile-time safety when needed

### Documentation

- Write documentation comments for all public items:
  ````rust
  /// Calculates the factorial of a number.
  ///
  /// # Arguments
  ///
  /// * `n` - A non-negative integer
  ///
  /// # Returns
  ///
  /// The factorial of `n`
  ///
  /// # Examples
  ///
  /// ```
  /// assert_eq!(factorial(5), 120);
  /// ```
  ///
  /// # Panics
  ///
  /// Panics if the result would overflow
  pub fn factorial(n: u32) -> u32 {
      // implementation
  }
  ````
- Include examples in doc comments
- Document panic conditions
- Document safety requirements for `unsafe` functions

### Code Organization

- **Declaration Order**: To improve readability and maintainability, declarations should be logically ordered. While `rustfmt` handles most formatting, a good practice is to order declarations based on reverse dependency and the likely call sequence:
  1.  `use` statements.
  2.  `static` and `const` items.
  3.  `type` definitions (`struct`, `enum`, `union`).
  4.  `trait` definitions.
  5.  `impl` blocks.
  6.  Public functions (`pub fn`).
  7.  Private functions (`fn`).
- Keep modules small and focused
- Use `mod.rs` or separate files for module organization
- Follow Rust naming conventions:
  - `snake_case` for functions, variables, modules
  - `PascalCase` for types, traits
  - `SCREAMING_SNAKE_CASE` for constants
- One type/trait per file for complex items
- Group related functionality in modules

### Traits and Generics

- **Services as Traits**: Define services using traits to allow for easy mocking and dependency injection.
- Implement standard traits where appropriate:
  - `Debug`, `Clone`, `PartialEq`, `Eq`
  - `Default` for types with sensible defaults
  - `From`/`Into` for conversions
  - `Display` for user-facing output
- Use trait bounds judiciously
- Prefer concrete types over generics when generality isn't needed
- Use associated types in traits when appropriate

### Performance

- Profile before optimizing
- Use `&str` instead of `String` for function parameters when possible
- Prefer iterators over collecting into vectors
- Use `Vec::with_capacity` when size is known
- Consider using `SmallVec` for small collections
- Use compile-time computations with `const fn` where possible

### Concurrency

- Prefer message passing with channels over shared state
- Use `Arc<Mutex<T>>` or `Arc<RwLock<T>>` for shared state when necessary
- Consider using `tokio` or `async-std` for async programming
- Avoid blocking in async contexts
- Use `Send` and `Sync` bounds appropriately

### Testing

#### Test Coverage Requirements

- **95% Coverage Target**: You MUST aim for 95% test coverage on all code changes. This is a hard requirement, not a suggestion.
- **The 5% Exception**: The remaining 5% is reserved for code that is truly untestable (e.g., platform-specific code paths, certain error handlers that cannot be triggered in tests, or FFI edge cases). Use your judgment to determine what qualifies as truly untestable.
- **Coverage Measurement**: You MUST run the test suite with coverage reporting after every code change using `cargo tarpaulin` or `cargo llvm-cov`.
- **Failure Handling**: If coverage is below 95%, you MUST ask the user for help. Do not proceed with incomplete coverage without explicit user guidance.

#### Refactoring for Testability

To achieve 95% coverage, you should proactively refactor and rearchitect the application:

- **Extract Testable Modules**: When encountering difficult-to-reach code paths, extract them into separate, independently testable modules. This makes the code more modular and easier to verify.
- **Dependency Injection via Traits**: Use traits as the primary mechanism for testability. Define service behavior as traits and inject concrete implementations, allowing tests to substitute mock implementations.
- **Adapter Pattern for External Dependencies**: Wrap all external APIs, databases, file systems, and third-party services in adapter modules behind traits. This isolates side effects and makes the core logic testable in isolation.
- **Identify and Restructure Untestable Code**: If code cannot achieve coverage, propose architectural changes to the user. Do not accept "untestable" as a final answer without exploring restructuring options.

#### Test Strategy Documentation

- **Mandatory Test Strategy**: For any code changes, you MUST document a test strategy as part of the pre-implementation design review.
- **Strategy Contents**: The test strategy should describe:
  - What will be tested (unit, integration, e2e)
  - How it will be tested (mocks, fixtures, real dependencies)
  - Expected coverage impact
  - Any code that may be excluded from coverage and why

#### Unit & Integration Testing

- Write unit tests in the same file using `#[cfg(test)]`:

  ```rust
  #[cfg(test)]
  mod tests {
      use super::*;

      #[test]
      fn test_function_name() {
          // test implementation
      }
  }
  ```

- Use integration tests in `tests/` directory
- **Adapter Testing**: Only tests for adapters should invoke actual external dependencies. Other services should use mocks of the traits.

#### External API Integration Workflow

1. **Explore the API first** - Use curl or similar to understand actual response shapes before writing any code.
2. **Derive models from real responses** - Application models should be based on actual API responses, not documentation assumptions.
3. **Adapters parse and validate** - Adapters make real calls and deserialize responses into application models. Adapter integration tests should make real API calls whenever practical; use fixtures only when real calls are impractical (rate limits, costs, destructive operations).
4. **Mock adapters internally** - All other services use mock versions of adapters. These mocks return properly-typed application models.
5. **Unit test string manipulation** - URL construction, query parameter handling, ID parsing all need explicit unit tests.

#### End-to-End Testing

- **Functional Requirements Validation**: E2E tests MUST validate that the system aligns with high-level functional requirements.
- **User Clarification**: If the functional requirements are not obvious or clearly documented, you MUST ask the user to clarify them before writing E2E tests.
- **E2E Test Scope**: E2E tests should cover critical user journeys and integration points between services. They complement, but do not replace, unit and integration tests.
- **E2E in Rust**: Place E2E tests in the `tests/` directory. Consider using test binaries or integration test modules that spin up the full application.

#### Additional Testing Practices

- Use property-based testing with `proptest` or `quickcheck` for complex logic
- Test error cases explicitly
- Use `#[should_panic]` for tests that should panic

### Dependencies

- Prefer well-established crates from the ecosystem
- Common essential crates:
  - `serde` for serialization
  - `tokio` or `async-std` for async runtime
  - `reqwest` for HTTP clients
  - `clap` for CLI arguments
  - `thiserror` or `anyhow` for error handling
  - `tracing` for logging
  - `chrono` or `time` for date/time handling
- Keep dependencies minimal
- Audit dependencies for security with `cargo audit`

### Unsafe Code

- Avoid `unsafe` unless absolutely necessary
- When using `unsafe`:
  - Document safety invariants
  - Keep unsafe blocks as small as possible
  - Write extensive tests
  - Consider using existing safe abstractions
  - Mark containing functions as `unsafe` if they have safety requirements

### Common Patterns

- **Builder Pattern**: For complex object construction

  ```rust
  pub struct ConfigBuilder {
      // fields
  }

  impl ConfigBuilder {
      pub fn new() -> Self { /* ... */ }
      pub fn with_timeout(mut self, timeout: Duration) -> Self { /* ... */ }
      pub fn build(self) -> Result<Config, Error> { /* ... */ }
  }
  ```

- **Type State Pattern**: For compile-time state machines
- **RAII**: Leverage `Drop` trait for cleanup
- **Interior Mutability**: Use `Cell`, `RefCell` when needed

### Validation and Parsing

- Use serde with validation:

  ```rust
  use serde::{Deserialize, Serialize};
  use validator::Validate;

  #[derive(Debug, Deserialize, Serialize, Validate)]
  struct User {
      #[validate(length(min = 1, max = 100))]
      name: String,
      #[validate(email)]
      email: String,
      #[validate(range(min = 0, max = 150))]
      age: u8,
  }
  ```

- Parse, don't validate (make invalid states unrepresentable)
- Use strong types to enforce invariants

### Cargo Configuration

- Use workspace for multi-crate projects
- Configure useful lints in `Cargo.toml`:

  ```toml
  [lints.rust]
  unsafe_code = "warn"
  missing_docs = "warn"

  [lints.clippy]
  all = "warn"
  pedantic = "warn"
  ```

- Use `cargo fmt` for consistent formatting
- Configure `rustfmt.toml` for project style

## Version Control

- **Conventional Commits**: You MUST use **Conventional Commits** for all git commit messages.
- **User Confirmation**: You MUST always ask the user for confirmation before committing any files.
- **Cleanup**: You MUST clean up any temporary files or build artifacts before committing.

## Checklist Before Committing

- [ ] `cargo build --release` succeeds
- [ ] `cargo test` passes
- [ ] Test coverage meets 95% threshold (`cargo tarpaulin --fail-under 95` or `cargo llvm-cov --fail-under-lines 95`)
- [ ] `cargo clippy -- -W clippy::all` shows no warnings
- [ ] `cargo fmt --check` passes
- [ ] No `unwrap()` or `expect()` in production code
- [ ] No `unsafe` blocks without justification and documentation
- [ ] Public API items have documentation comments
- [ ] Examples in documentation compile and run
- [ ] Error handling uses `Result` and `?`
- [ ] No compiler warnings
- [ ] Dependencies are justified and minimal
