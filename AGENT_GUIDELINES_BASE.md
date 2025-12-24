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


## Testing Strategy

- Services should be individually testable.
- The test suite should expect external dependencies to be present.
- Ideally, only tests for adapters should invoke actual external dependencies. Other services should use mocks or stubs of the adapters to maximize performance.

## Version Control

- **Conventional Commits**: You MUST use **Conventional Commits** for all git commit messages.
- **User Confirmation**: You MUST always ask the user for confirmation before committing any files.
- **Cleanup**: You MUST clean up any temporary files or build artifacts before committing.

## Post-Implementation Validation (MANDATORY)

After making ANY code changes, you MUST run the following commands in order and fix ALL errors before considering the task complete:

1. **Format code**: Run the appropriate formatter to ensure consistent code formatting.
2. **Lint code**: Run the linter to check for code quality issues.
3. **Type check**: Run the type checker/compiler to verify type correctness (if applicable).

If any of these commands fail, you MUST fix the errors before moving on. Do not mark a task as complete if linting, formatting, or type checking fails.

### Quick Check Command

If available, run all checks at once using the project's configured check command.
