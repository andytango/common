# TypeScript Project Setup Instructions

Set up a TypeScript development environment with strict type checking, comprehensive linting, and automatic code formatting.

## Tools Configured

- **TypeScript**: Strict type checking with all strict compiler options enabled
- **ESLint**: Code quality, documentation, and formatting enforcement
- **Prettier**: Automatic code formatting integrated with ESLint
- **JSDoc Plugin**: Documentation required for all module-level statements

## Installation

```bash
npm install
```

## Configuration Details

### TypeScript (tsconfig.json)

Maximum strictness configuration:
- All `strict` mode options enabled
- `noUnusedLocals`, `noUnusedParameters`, `noImplicitReturns`
- `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`
- `noPropertyAccessFromIndexSignature`

### ESLint (eslint.config.mjs)

Enforces:
- **Explicit return types** on all functions
- **JSDoc documentation** on all functions, classes, interfaces, types, enums, and exported constants
- **No `any` type**
- **Prettier integration** as lint errors

### Prettier (.prettierrc.json)

Standard formatting with single quotes, semicolons, 2-space tabs, trailing commas

## NPM Scripts

Add these scripts to your `package.json`:

```json
{
  "scripts": {
    "build": "tsc",
    "lint": "eslint . --ext .ts",
    "lint:fix": "eslint . --ext .ts --fix",
    "format": "prettier --write \"src/**/*.ts\"",
    "format:check": "prettier --check \"src/**/*.ts\"",
    "type-check": "tsc --noEmit",
    "check": "npm run format && npm run lint && npm run type-check"
  }
}
```

## Required Workflow

After making any code changes, run:

```bash
npm run format  # Format code
npm run lint    # Check linting
npm run type-check  # Type check
```

Or run all at once: `npm run check`

## JSDoc Requirements

All exported functions, classes, interfaces, types, enums, and constants require JSDoc:

```typescript
/**
 * Calculates the sum of two numbers.
 * @param a - The first number
 * @param b - The second number
 * @returns The sum of a and b
 */
export function add(a: number, b: number): number {
  return a + b;
}

/**
 * Configuration options
 */
export interface Config {
  port: number;
  host: string;
}

/**
 * Default configuration
 */
export const defaultConfig: Config = {
  port: 3000,
  host: 'localhost',
};
```

## Explicit Return Types

All functions require explicit return types - implicit returns are not allowed.
