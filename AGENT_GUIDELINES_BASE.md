- use the simplest possible solution
- prefer procedural programming and functional programming paradigms
- use well maintained, mature libraries where possible
- if you feel a need to use object-oriented programming or polymorphism, then
  ask the user about your design first
- ensure you include docstrings for all top level statements
- run the linter as frequently as possible to detect any issues and fix them
- avoid using overriding the linter or using language escape hatches, such as
  typescript "any" assertions or rust unsafe blocks
- after you have finished your implementation, run the compiler and try to
  test your implementation if possible. if you don't know how to test your
  implementation, then ask the user for help.
- when handling unknown data structures, use a validation library such as zod,
  or in rust you might use serde or zod_rs

## Post-Implementation Validation (MANDATORY)

After making ANY code changes, you MUST run the following commands in order and fix ALL errors before considering the task complete:

1. **Format code**: Run prettier to ensure consistent code formatting
   - For TypeScript: `npm run format` or `prettier --write "src/**/*.ts"`
   - For Python: `black .` or your configured formatter
   - For Rust: `cargo fmt`

2. **Lint code**: Run the linter to check for code quality issues
   - For TypeScript: `npm run lint` or `eslint . --ext .ts`
   - For Python: `pylint` or `ruff check .`
   - For Rust: `cargo clippy -- -D warnings`

3. **Type check**: Run the type checker/compiler to verify type correctness
   - For TypeScript: `npm run type-check` or `tsc --noEmit`
   - For Python: `mypy .` or `pyright`
   - For Rust: `cargo check`

If any of these commands fail, you MUST fix the errors before moving on. Do not mark a task as complete if linting, formatting, or type checking fails.

### Quick Check Command

If available, run all checks at once:
- TypeScript: `npm run check`
- Python: Create a script that runs `black . && pylint && mypy .`
- Rust: `cargo fmt && cargo clippy -- -D warnings && cargo check`