- Use the simplest possible solution
- Prefer procedural programming and functional programming paradigms.
- **Simplicity**: Prioritize **Cognitive Simplicity** over **Code Brevity**. Explicit is better than implicit. Unnested code is better than nested. Code that can be read top-to-bottom is better than clever abstractions. Service Oriented Architecture is intended for COMPLEX projects. For simple tasks, keep it simple.
- **Conflict Resolution**: If "Simplicity" conflicts with "Robustness" (e.g., Error Handling or SRP), **Robustness wins**. It is better to have verbose, safe, and testable code than concise but fragile code.
- Use well maintained, mature libraries where possible
- If you feel a need to use object-oriented programming or polymorphism, then
  ask the user about your design first
- Ensure you include docstrings for all top level statements
- Run the linter as frequently as possible to detect any issues and fix them
- Avoid using overriding the linter or using language escape hatches.
- After you have finished your implementation, run the compiler/interpreter and verify your changes.
- When handling unknown data structures, use a validation library.

## Pre-Implementation Design Protocol

**Analysis & Planning**: For architectural changes, new services, or complex logic, you MUST first output a brief plan. This plan serves as the basis for the **Design Review** you conduct with the user. It should explicitly address:
1.  **Safety**: Identify potential edge cases or error states.
2.  **Architecture**: Verify if Single Responsibility Principle (SRP) is maintained. If a "mode" or variation is detected, explicitly list the strategy classes/functions to be created.
3.  **Result Definition**: Define the specific `Result<T, E>` types that will be returned, ensuring errors are typed.
4.  **External API Exploration**: Before implementing external API integrations, manually test the API (e.g., using curl) to understand actual response shapes and document them.

## Architecture & Design

- **Single Responsibility Principle**: Each module, class, or function should have one, and only one, reason to change. Decompose large, multi-purpose components into smaller, focused ones.
- **Composition over Configuration/Inheritance**: Favor composition to define behavior. If a component operates in different "modes" with overlapping but distinct dependencies or logic, do not use internal flags or configuration options to switch behavior. Instead, extract shared dependencies and create distinct implementations (strategies) for each mode, then inject the correct implementation.
- **Service Oriented Architecture**: For **COMPLEX** projects, decompose systems into small, individually testable services located under a `lib` folder.
- **Implementation Style**: When implementing services, prefer a **procedural** or **functional** style over complex object-oriented patterns.
- **Declaration Order**: To improve readability and maintainability, declarations should be logically ordered. A good practice is to order declarations based on reverse dependency and the likely call sequence, with exported declarations preceding local ones.
- **Dependency Structure**: Design services with orthogonality, layering, and proper abstraction levels in mind. Tend towards a tree or diamond dependency pattern. Apply **SOLID principles**.
- **Adapter Pattern**: Always wrap external APIs or services in an adapter service with minimal logic.
- **Error Handling**: Favor returning structured **Result Objects** (indicating Success or Failure) for expected domain errors rather than throwing exceptions. This makes error handling explicit in the API contract.
- **Design Review**: You MUST always have the user review your service design before implementation.
- **Test Confirmation**: You MUST always ask the user if they would like tests to be added when performing a task.
- **Manual Testing**: If the user declines automated tests, you MUST try to test manually. If you don't know how to test manually, you MUST ask the user for help.

## Anti-patterns

 - **Mode Flags**: Avoid functions, methods, or services that take a boolean flag or "mode" enum to significantly alter their control flow or dependencies (e.g., `process_data(data, mode='fast')`).
 - **Remediation**: For complex variations, **Strongly Prefer** the Strategy Pattern by creating distinct functions or implementations for each mode (e.g., `FastDataProcessor`, `SafeDataProcessor`) and using composition.
 - **Testing Against Mental Model**: Avoid writing tests that validate assumptions rather than real API contracts. Tests that pass based on expected behavior but fail against actual external system responses indicate this anti-pattern.
 - **Remediation**: Test adapters with real API calls whenever practical. When real calls are impractical, use fixture-captured real responses. Unit test all string manipulation (URL construction, ID parsing).


## Testing Strategy

### Test Coverage Requirements

- **95% Coverage Target**: You MUST aim for 95% test coverage on all code changes. This is a hard requirement, not a suggestion.
- **The 5% Exception**: The remaining 5% is reserved for code that is truly untestable (e.g., platform-specific code paths, certain error handlers that cannot be triggered in tests, or third-party integration edge cases). Use your judgment to determine what qualifies as truly untestable.
- **Coverage Measurement**: You MUST run the test suite with coverage reporting after every code change.
- **Failure Handling**: If coverage is below 95%, you MUST ask the user for help. Do not proceed with incomplete coverage without explicit user guidance.

### Refactoring for Testability

To achieve 95% coverage, you should proactively refactor and rearchitect the application:

- **Extract Testable Modules**: When encountering difficult-to-reach code paths, extract them into separate, independently testable modules. This makes the code more modular and easier to verify.
- **Dependency Injection**: Use dependency injection as the primary mechanism for testability. Inject dependencies rather than instantiating them directly, allowing tests to substitute mocks or stubs.
- **Adapter Pattern for External Dependencies**: Wrap all external APIs, databases, file systems, and third-party services in adapter modules. This isolates side effects and makes the core logic testable in isolation.
- **Identify and Restructure Untestable Code**: If code cannot achieve coverage, propose architectural changes to the user. Do not accept "untestable" as a final answer without exploring restructuring options.

### Test Strategy Documentation

- **Mandatory Test Strategy**: For any code changes, you MUST document a test strategy as part of the pre-implementation design review.
- **Strategy Contents**: The test strategy should describe:
  - What will be tested (unit, integration, e2e)
  - How it will be tested (mocks, fixtures, real dependencies)
  - Expected coverage impact
  - Any code that may be excluded from coverage and why

### Unit & Integration Testing

- Services should be individually testable.
- The test suite should expect external dependencies to be present.
- Ideally, only tests for adapters should invoke actual external dependencies. Other services should use mocks or stubs of the adapters to maximize performance.

### End-to-End Testing

- **Functional Requirements Validation**: E2E tests MUST validate that the system aligns with high-level functional requirements.
- **User Clarification**: If the functional requirements are not obvious or clearly documented, you MUST ask the user to clarify them before writing E2E tests.
- **E2E Test Scope**: E2E tests should cover critical user journeys and integration points between services. They complement, but do not replace, unit and integration tests.

### External API Integration Workflow

1. **Explore the API first** - Use curl or similar to understand actual response shapes before writing any code.
2. **Derive models from real responses** - Application models should be based on actual API responses, not documentation assumptions.
3. **Adapters parse and validate** - Adapters make real calls and deserialize responses into application models. Adapter integration tests should make real API calls whenever practical; use fixtures only when real calls are impractical (rate limits, costs, destructive operations).
4. **Mock adapters internally** - All other services use mock versions of adapters. These mocks return properly-typed application models.
5. **Unit test string manipulation** - URL construction, query parameter handling, ID parsing all need explicit unit tests.

## Version Control

- **Conventional Commits**: You MUST use **Conventional Commits** for all git commit messages.
- **User Confirmation**: You MUST always ask the user for confirmation before committing any files.
- **Cleanup**: You MUST clean up any temporary files or build artifacts before committing.

## Post-Implementation Validation (MANDATORY)

After making ANY code changes, you MUST run the following commands in order and fix ALL errors before considering the task complete:

1. **Format code**: Run the appropriate formatter to ensure consistent code formatting.
2. **Lint code**: Run the linter to check for code quality issues.
3. **Type check**: Run the type checker/compiler to verify type correctness (if applicable).
4. **Run tests with coverage**: Run the test suite with coverage reporting enabled.
5. **Verify coverage threshold**: Ensure coverage meets the 95% threshold. If coverage is below 95%, you MUST ask the user for help before proceeding.

If any of these commands fail, you MUST fix the errors before moving on. Do not mark a task as complete if linting, formatting, type checking, or coverage checks fail.

### Quick Check Command

If available, run all checks at once using the project's configured check command. Ensure the check command includes coverage reporting.
