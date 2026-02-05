# Common Agent Guidelines

A collection of development guidelines and setup instructions for AI coding agents across multiple programming languages and frameworks.

## Overview

This repository provides standardized guidelines that help AI coding agents write high-quality, maintainable code. The guidelines emphasize simplicity, type safety, proper tooling, and comprehensive validation.

## Contents

### Language-Agnostic Guidelines

- **[AGENT_GUIDELINES_BASE.md](./AGENT_GUIDELINES_BASE.md)**: Core principles applicable to all languages
  - Simplicity and maintainability
  - Standardized project directory structure (`tmp/`, `credentials/`, `logs/`)
  - Functional and procedural programming preferences
  - Linting and formatting requirements
  - Post-implementation validation workflows
  - Runtime data validation strategies

### Language-Specific Guidelines

- **[AGENT_GUIDELINES_TS.md](./AGENT_GUIDELINES_TS.md)**: TypeScript development standards
  - Strict type safety configuration
  - JSDoc documentation requirements
  - Testing with Jest/Vitest
  - React patterns and best practices
  - Runtime validation with Zod

- **[AGENT_GUIDELINES_PYTHON.md](./AGENT_GUIDELINES_PYTHON.md)**: Python development standards
  - Type hints and mypy/pyright usage
  - PEP 8 compliance
  - Testing with pytest
  - Data validation with Pydantic
  - Async/concurrency patterns

- **[AGENT_GUIDELINES_RUST.md](./AGENT_GUIDELINES_RUST.md)**: Rust development standards
  - Memory safety and ownership
  - Error handling with Result types
  - Clippy linting configuration
  - Documentation standards
  - Unsafe code guidelines

### Setup Instructions

- **[SETUP_PROMPT_TS.md](./SETUP_PROMPT_TS.md)**: TypeScript project setup
  - Complete tooling configuration (TypeScript, ESLint, Prettier)
  - Strict compiler settings
  - Required npm scripts
  - JSDoc enforcement

- **[SETUP_PROMPT_PYTHON.md](./SETUP_PROMPT_PYTHON.md)**: Python project setup
  - Tooling configuration (mypy, ruff, black)
  - Type checking settings
  - Testing setup with pytest

- **[SETUP_PROMPT_RUST.md](./SETUP_PROMPT_RUST.md)**: Rust project setup
  - Clippy and rustfmt configuration
  - Cargo.toml best practices
  - Documentation requirements

### Claude Code Skill

- **[SETUP_SKILL.md](./SETUP_SKILL.md)**: Claude Code skill for automated setup
  - Auto-detects project language(s)
  - Fetches and combines appropriate guidelines
  - Creates AGENTS.md and CLAUDE.md symlink
  - Handles monorepos with nested projects
  - Installation: `mkdir -p ~/.claude/skills/setup-guidelines && cp SETUP_SKILL.md ~/.claude/skills/setup-guidelines/SKILL.md`
  - Usage: `/setup-guidelines` or `/setup-guidelines [project-path]`

## Core Principles

All guidelines share these fundamental principles:

1. **Simplicity First**: Use the simplest solution that solves the problem
2. **Type Safety**: Leverage static type systems to catch errors early
3. **Validation**: Validate data at system boundaries using appropriate libraries
4. **Testing**: Write tests alongside implementation
5. **Documentation**: Document all public APIs with examples
6. **Linting**: Run linters frequently and fix all issues before committing
7. **No Escape Hatches**: Avoid language features that bypass safety checks (`any`, `unsafe`, `type: ignore`, etc.)

## Post-Implementation Checklist

After making code changes, agents should always:

1. **Format**: Run the language-specific formatter
2. **Lint**: Check and fix linting errors
3. **Type Check**: Verify type correctness
4. **Test**: Run the test suite
5. **Build**: Ensure the project builds successfully

## Usage

These guidelines are designed to be:

- **Referenced**: By AI coding agents during development sessions
- **Extended**: Teams can build upon these with project-specific requirements
- **Adapted**: Modified to fit specific project needs while maintaining core principles

## Language-Specific Quick Reference

| Language | Formatter | Linter | Type Checker | Test Framework |
|----------|-----------|--------|--------------|----------------|
| TypeScript | Prettier | ESLint | tsc | Jest/Vitest |
| Python | Black | Ruff/Pylint | mypy/pyright | pytest |
| Rust | rustfmt | clippy | cargo check | cargo test |

## Contributing

These guidelines are meant to evolve. Contributions that improve clarity, add missing best practices, or cover additional languages are welcome.

## License

These guidelines are provided as-is for use in AI-assisted development workflows.
