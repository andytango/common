# TypeScript Development Guidelines

## Core Principles

- Use the simplest possible solution
- Prefer procedural programming and functional programming paradigms
- **Simplicity**: Use the simplest possible solution. Service Oriented Architecture is intended for COMPLEX projects. For simple tasks, keep it simple.
- **Avoid Classes**: Prefer object literals returned by factory functions, along with TypeScript interfaces.
- Use well-maintained, mature libraries where possible (check npm weekly downloads and last update date)
- Ensure you include JSDoc comments for all exported functions, classes, and interfaces
- Run the linter (ESLint) as frequently as possible to detect any issues and fix them
- Avoid overriding the linter or using language escape hatches such as:
  - `any` type assertions
  - `@ts-ignore` or `@ts-expect-error` comments
  - Type assertions without proper validation
- After you have finished your implementation:
  - Run the TypeScript compiler (`tsc`) to check for type errors
  - Test your implementation using the project's test framework (Jest, Vitest, etc.)
  - If you don't know how to test your implementation, ask the user for help
- When handling unknown data structures, use a validation library such as Zod, io-ts, or yup

## Architecture & Design

- **Single Responsibility Principle**: Each module, class, or function should have one, and only one, reason to change. Decompose large, multi-purpose components into smaller, focused ones.
- **Composition over Configuration/Inheritance**: Favor composition to define behavior. If a component operates in different "modes" with overlapping but distinct dependencies or logic, do not use internal flags or configuration options to switch behavior. Instead, extract shared dependencies and create distinct implementations (strategies) for each mode.
- **Service Oriented Architecture**: For **COMPLEX** projects, decompose systems into small, individually testable services located under a `lib` folder.
- **Implementation Style**: When implementing services, prefer a **procedural** or **functional** style over complex object-oriented patterns.
- **Dependency Structure**: Design services with orthogonality, layering, and proper abstraction levels in mind. Tend towards a tree or diamond dependency pattern. Apply **SOLID principles**.
- **Adapter Pattern**: Always wrap external APIs or services in an adapter service with minimal logic.
- **Design Review**: You MUST always have the user review your service design before implementation.
- **Test Confirmation**: You MUST always ask the user if they would like tests to be added when performing a task.
- **Manual Testing**: If the user declines automated tests, you MUST try to test manually. If you don't know how to test manually, you MUST ask the user for help.

## Anti-patterns

- **Mode Flags**: Avoid functions or components that take a boolean flag or "mode" enum to significantly alter their behavior.
  - **Bad**:
    ```typescript
    function processUser(user: User, mode: 'full' | 'lite') {
       if (mode === 'full') { /* ... */ }
    }
    ```
  - **Good**: Use separate functions (`processUserFull`, `processUserLite`) or inject a strategy object that handles the processing variation.

## TypeScript-Specific Guidelines

### Type Safety

- **Always enable strict mode** in `tsconfig.json`:
  ```json
  {
    "compilerOptions": {
      "strict": true,
      "noImplicitAny": true,
      "strictNullChecks": true,
      "strictFunctionTypes": true
    }
  }
  ```
- Prefer `unknown` over `any` when the type is truly unknown
- Use type guards and narrowing for runtime type checking
- Define explicit return types for functions to catch errors early
- Use `const` assertions for literal types when appropriate

### Types and Interfaces

- Prefer `interface` for object shapes that might be extended
- Use `type` for unions, intersections, and utility types
- Keep types close to where they are used
- Export types that are part of the public API
- Use descriptive names for type parameters: `<TUser>` instead of `<T>`

### Functions and Methods

- Use arrow functions for callbacks and inline functions
- Use regular function declarations for top-level functions
- **Context Objects**: When functions share common context or state (e.g., database client, logger, external service, callbacks, or common data parameters), use a "context object" passed as the first argument with an explicit interface:

  ```typescript
  interface ServiceContext {
    db: DatabaseClient;
    logger: Logger;
    config: AppConfig;
  }

  function getUser(ctx: ServiceContext, userId: string): Promise<User> {
    ctx.logger.info("Fetching user", { userId });
    return ctx.db.users.findById(userId);
  }

  function updateUser(
    ctx: ServiceContext,
    userId: string,
    data: UserUpdate,
  ): Promise<User> {
    ctx.logger.info("Updating user", { userId });
    return ctx.db.users.update(userId, data);
  }
  ```

- **Avoid Destructuring in Function Arguments**: Do not destructure objects in function parameters. Instead, accept the object as-is and destructure at the top of the function body. This keeps function signatures and JSDoc cleaner, and makes it easy to pass the object to nested function calls:

  ```typescript
  // Avoid this:
  function processOrder({ orderId, items, customer }: OrderInput): Result {
    // ...
  }

  // Prefer this:
  function processOrder(input: OrderInput): Result {
    const { orderId, items, customer } = input;
    // Now 'input' can easily be passed to helper functions
    validateOrder(input);
    // ...
  }
  ```

- Provide JSDoc comments with examples:
  ```typescript
  /**
   * Calculates the total price including tax
   * @param price - Base price in cents
   * @param taxRate - Tax rate as a decimal (e.g., 0.08 for 8%)
   * @returns Total price in cents
   * @example
   * calculateTotal(1000, 0.08) // returns 1080
   */
  ```

### Error Handling

- **Favor Result Objects over Exceptions**: For expected domain errors, return a structured result object (discriminated union) instead of throwing exceptions. This forces consumers to handle the failure case.

  ```typescript
  type Result<T, E> =
    | { success: true; value: T }
    | { success: false; error: E };

  function divide(a: number, b: number): Result<number, string> {
    if (b === 0) {
      return { success: false, error: "Division by zero" };
    }
    return { success: true, value: a / b };
  }

  // Usage
  const res = divide(10, 2);
  if (res.success) {
    console.log(res.value);
  } else {
    console.error(res.error);
  }
  ```

- **Exceptions**: Reserve `throw` for truly exceptional, unrecoverable system errors (e.g., programmer errors, out of memory).
- Always handle Promise rejections.

### Code Organization

- **Declaration Order**: To improve readability and maintainability, declarations should be ordered based on reverse dependency and likely call sequence. The standard order is as follows:
  1.  **Exported Types**: Interfaces and types that are part of the public API.
  2.  **Local Types**: Interfaces and types used only within the current file.
  3.  **Exported Assignments**: Constants and variables that are part of the public API.
  4.  **Local Assignments**: Constants and variables used only within the current file.
  5.  **Exported Functions**: Functions that are part of the public API.
  6.  **Local Functions**: Helper functions used only within the current file.
- One exported item per file for components/classes
- Group related utilities in a single file
- Use barrel exports (index.ts) sparingly to avoid circular dependencies
- Follow consistent file naming: `kebab-case.ts` or `PascalCase.tsx` for React

### Dependencies and Imports

- Use named imports instead of default imports where possible
- Order imports: external libs, internal modules, types, styles
- Avoid circular dependencies
- Use path aliases for cleaner imports when configured

### Testing

- Write tests alongside implementation
- Use descriptive test names that explain the behavior
- Test types using utility types like `Expect` or `AssertTrue`
- Mock external dependencies properly
- **Adapter Testing**: Only tests for adapters should invoke actual external dependencies. Other services should use mocks or stubs of the adapters.
- Aim for meaningful coverage, not just high percentages

### Performance

- Use `const enum` for compile-time constants
- Avoid premature optimization
- Use lazy imports for large libraries
- Consider bundle size when adding dependencies

### React-Specific (if applicable)

- Use functional components with hooks
- Define prop types using interfaces
- Use `React.FC` sparingly (prefer explicit return types)
- Memoize expensive computations with `useMemo`
- Use `useCallback` for stable function references

### Common Patterns

- **Builder Pattern**: Use method chaining with proper type inference
- **Factory Functions**: **PREFERRED** over classes for creating objects and services.
- **Discriminated Unions**: Use for state machines and variant types
- **Type Predicates**: Create reusable type guards
- **Utility Types**: Leverage built-in types like `Partial`, `Required`, `Pick`, `Omit`

### Validation

- Runtime validation at system boundaries:

  ```typescript
  import { z } from "zod";

  const UserSchema = z.object({
    id: z.string().uuid(),
    email: z.string().email(),
    age: z.number().min(0).max(120),
  });

  type User = z.infer<typeof UserSchema>;
  ```

### Debugging

- Use the TypeScript Language Server for IDE support
- Enable source maps for debugging
- Use `debugger` statements sparingly and remove before committing
- Leverage Chrome DevTools for runtime debugging

## Version Control

- **Conventional Commits**: You MUST use **Conventional Commits** for all git commit messages.
- **User Confirmation**: You MUST always ask the user for confirmation before committing any files.
- **Cleanup**: You MUST clean up any temporary files or build artifacts before committing.

## Checklist Before Committing

- [ ] All TypeScript compilation errors resolved
- [ ] ESLint warnings and errors addressed
- [ ] Tests passing
- [ ] No `any` types without justification
- [ ] No `@ts-ignore` comments
- [ ] JSDoc comments for public APIs
- [ ] Types exported where needed
- [ ] No circular dependencies
- [ ] Bundle size impact considered for new dependencies
