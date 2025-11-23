- Use the simplest possible solution
- Prefer procedural programming and functional programming paradigms.
- **Simplicity**: Use the simplest possible solution. Service Oriented Architecture is intended for COMPLEX projects. For simple tasks, keep it simple.
- Use well maintained, mature libraries where possible
- If you feel a need to use object-oriented programming or polymorphism, then
  ask the user about your design first
- Ensure you include docstrings for all top level statements
- Run the linter as frequently as possible to detect any issues and fix them
- Avoid using overriding the linter or using language escape hatches.
- After you have finished your implementation, run the compiler/interpreter and verify your changes.
- When handling unknown data structures, use a validation library.

## Architecture & Design

- **Service Oriented Architecture**: For **COMPLEX** projects, decompose systems into small, individually testable services located under a `lib` folder.
- **Implementation Style**: When implementing services, prefer a **procedural** or **functional** style over complex object-oriented patterns.
- **Dependency Structure**: Design services with orthogonality, layering, and proper abstraction levels in mind. Tend towards a tree or diamond dependency pattern. Apply **SOLID principles**.
- **Adapter Pattern**: Always wrap external APIs or services in an adapter service with minimal logic.
- **Design Review**: You MUST always have the user review your service design before implementation.
- **Test Confirmation**: You MUST always ask the user if they would like tests to be added when performing a task.
- **Manual Testing**: If the user declines automated tests, you MUST try to test manually. If you don't know how to test manually, you MUST ask the user for help.

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