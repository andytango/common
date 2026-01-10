# Python Development Guidelines

## Core Principles

- Use the simplest possible solution
- Prefer procedural programming and functional programming paradigms
- **Simplicity**: Prioritize **Cognitive Simplicity** over **Code Brevity**. Explicit is better than implicit. Unnested code is better than nested. Code that can be read top-to-bottom is better than clever abstractions. Service Oriented Architecture is intended for COMPLEX projects. For simple tasks, keep it simple.
- **Conflict Resolution**: If "Simplicity" conflicts with "Robustness" (e.g., Error Handling or SRP), **Robustness wins**. It is better to have verbose, safe, and testable code than concise but fragile code.
- Use well-maintained, mature packages where possible (check PyPI downloads and recent releases)
- If you feel a need to use complex class hierarchies or metaclasses, ask the user about your design first
- Ensure you include docstrings for all modules, classes, and functions
- Run linters (pylint, flake8, or ruff) as frequently as possible to detect any issues and fix them
- Avoid overriding the linter or using language escape hatches such as:
  - `# type: ignore` comments without justification
  - `# noqa` comments
  - Bare `except:` clauses
  - `eval()` or `exec()` in production code
- After you have finished your implementation:
  - Run mypy or pyright for type checking
  - Run your test suite (pytest, unittest)
  - Check code coverage if configured
  - If you don't know how to test your implementation, ask the user for help
- When handling unknown data structures, use a validation library such as Pydantic, marshmallow, or cerberus

## Pre-Implementation Design Protocol

**Analysis & Planning**: For architectural changes, new services, or complex logic, you MUST first output a brief plan. This plan serves as the basis for the **Design Review** you conduct with the user. It should explicitly address:
1.  **Safety**: Identify potential edge cases or error states.
2.  **Architecture**: Verify if Single Responsibility Principle (SRP) is maintained. If a "mode" or variation is detected, explicitly list the strategy classes/functions to be created.
3.  **Result Definition**: Define the specific `Result[T, E]` types that will be returned, ensuring errors are typed.
4.  **External API Exploration**: Before implementing external API integrations, manually test the API (e.g., using curl) to understand actual response shapes and document them.

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

- **Mode Flags**: Avoid functions or services that take a boolean flag or "mode" enum to significantly alter their control flow or dependencies.
  - **Exception**: Simple configuration flags that do not drastically change logic (e.g., `verbose=True`, `dry_run=True`) are acceptable.
  - **Bad**:
    ```python
    def process_payment(amount, mode="stripe"):
        if mode == "stripe":
            # ... stripe logic ...
        elif mode == "paypal":
            # ... paypal logic ...
    ```
  - **Good**: **Strongly Prefer** the Strategy Pattern by creating distinct services/functions (`StripePaymentProcessor`, `PayPalPaymentProcessor`) that implement a common protocol or interface. Inject the specific implementation required.
- **Testing Against Mental Model**: Avoid writing tests that validate assumptions rather than real API contracts. Tests that pass based on expected behavior but fail against actual external system responses indicate this anti-pattern.
  - **Remediation**: Test adapters with real API calls whenever practical. When real calls are impractical, use fixture-captured real responses. Unit test all string manipulation (URL construction, ID parsing).
- **Dead Code**: Avoid leaving unused code in the codebase, including unused imports, unreachable code paths, commented-out code, unused variables, unused functions, and unused classes. Dead code increases cognitive load, obscures intent, and can mislead future readers.
  - **Remediation**: Use tools like `vulture`, `ruff` (rules `F401` for unused imports, `F841` for unused variables), or `pylint` to detect dead code. Remove it immediately rather than commenting it out. Version control preserves history if code needs to be recovered later.

## Python-Specific Guidelines

### Python Version and Environment

- Target Python 3.8+ for better type hints support
- Use virtual environments (`venv`, `virtualenv`, or `poetry`)
- Specify dependencies in `requirements.txt` or `pyproject.toml`
- Include a `.python-version` file for version specification

### Type Hints

- **Always use type hints** for function signatures:
  ```python
  def calculate_total(price: float, tax_rate: float) -> float:
      """Calculate total price including tax."""
      return price * (1 + tax_rate)
  ```
- Use `typing` module for complex types:
  ```python
  from typing import List, Dict, Optional, Union, Tuple, Any
  from typing import TypeVar, Generic, Protocol  # For advanced typing
  ```
- For Python 3.10+, use built-in generics:
  ```python
  def process_items(items: list[str]) -> dict[str, int]:
      ...
  ```
- Use `TypedDict` for dictionary schemas
- Prefer `Optional[T]` over `Union[T, None]`
- Use `typing.Protocol` for structural subtyping

### Error Handling

- **Favor Result Objects over Exceptions**: Use a structured Result pattern for expected domain errors. Reserve exceptions for truly exceptional, unrecoverable states (e.g., memory errors, configuration errors preventing startup).

- **The Canonical Result Pattern**: Do not invent your own Result type. Use this exact structure for consistency across the project. The `Failure` variant MUST contain a strongly typed error object (e.g., an Enum or Dataclass), NOT a raw string, to allow the caller to handle specific error cases programmatically.

  ```python
  from dataclasses import dataclass
  from typing import Generic, TypeVar, Union

  T = TypeVar("T")
  E = TypeVar("E")

  @dataclass
  class Success(Generic[T]):
      value: T
      is_success: bool = True

  @dataclass
  class Failure(Generic[E]):
      error: E
      is_success: bool = False

  Result = Union[Success[T], Failure[E]]

  # Example Usage with Typed Error
  @dataclass
  class DivisionError:
      message: str
      code: int

  def divide(a: float, b: float) -> Result[float, DivisionError]:
      if b == 0:
          return Failure(DivisionError("Division by zero", 100))
      return Success(a / b)

  # Usage
  res = divide(10, 0)
  if res.is_success:
      print(res.value)
  else:
      # Error is strongly typed
      print(f"Error Code: {res.error.code}")
  ```

- **Avoid Bare Except Clauses**: Never use bare `except:` clauses.
- **Specific Exceptions**: If you must use exceptions (e.g., when implementing standard library protocols), use specific exception types.


### Functions and Methods

- Keep functions small and focused (single responsibility)
- **Services as Functions**: Services can be implemented as module-level functions that consume objects and call methods on them.
- Use default arguments carefully (avoid mutable defaults):

  ```python
  # Bad
  def append_to_list(item, target=[]):
      target.append(item)
      return target

  # Good
  def append_to_list(item, target=None):
      if target is None:
          target = []
      target.append(item)
      return target
  ```

- Use `*args` and `**kwargs` judiciously
- Prefer returning early over nested conditionals
- Use generator functions for large data sets

### Classes and Objects

- **Services as Classes**: Services can be implemented as plain classes with dependencies "injectable" via the constructor.
- Prefer composition over inheritance
- Use `dataclasses` for simple data containers:

  ```python
  from dataclasses import dataclass

  @dataclass
  class User:
      id: int
      name: str
      email: str
      active: bool = True
  ```

- Use `__slots__` for memory efficiency when needed
- Implement `__str__` and `__repr__` for debugging
- Use properties for computed attributes:

  ```python
  class Circle:
      def __init__(self, radius: float):
          self._radius = radius

      @property
      def area(self) -> float:
          return 3.14159 * self._radius ** 2
  ```

### Testing

#### Test Coverage Requirements

- **95% Coverage Target**: You MUST aim for 95% test coverage on all code changes. This is a hard requirement, not a suggestion.
- **The 5% Exception**: The remaining 5% is reserved for code that is truly untestable (e.g., platform-specific code paths, certain error handlers that cannot be triggered in tests, or third-party integration edge cases). Use your judgment to determine what qualifies as truly untestable.
- **Coverage Measurement**: You MUST run the test suite with coverage reporting after every code change using `pytest --cov` or `coverage run -m pytest`.
- **Failure Handling**: If coverage is below 95%, you MUST ask the user for help. Do not proceed with incomplete coverage without explicit user guidance.

#### Refactoring for Testability

To achieve 95% coverage, you should proactively refactor and rearchitect the application:

- **Extract Testable Modules**: When encountering difficult-to-reach code paths, extract them into separate, independently testable modules. This makes the code more modular and easier to verify.
- **Dependency Injection**: Use dependency injection as the primary mechanism for testability. Pass dependencies as function parameters or constructor arguments rather than instantiating them directly.
- **Adapter Pattern for External Dependencies**: Wrap all external APIs, databases, file systems, and third-party services in adapter modules. This isolates side effects and makes the core logic testable in isolation.
- **Identify and Restructure Untestable Code**: If code cannot achieve coverage, propose architectural changes to the user. Do not accept "untestable" as a final answer without exploring restructuring options.

#### Test Strategy Documentation

- **Mandatory Test Strategy**: For any code changes, you MUST document a test strategy as part of the pre-implementation design review.
- **Strategy Contents**: The test strategy should describe:
  - What will be tested (unit, integration, e2e)
  - How it will be tested (mocks, fixtures, real dependencies)
  - Expected coverage impact
  - Any code that may be excluded from coverage and why

#### Unit & Integration Testing

- Use pytest for testing (preferred over unittest)
- Write tests alongside implementation
- Use descriptive test names:

  ```python
  def test_user_creation_with_valid_data():
      ...

  def test_user_creation_raises_error_for_invalid_email():
      ...
  ```

- Use fixtures for test setup:

  ```python
  import pytest

  @pytest.fixture
  def sample_user():
      return User(id=1, name="Test User", email="test@example.com")

  def test_user_update(sample_user):
      sample_user.name = "Updated Name"
      assert sample_user.name == "Updated Name"
  ```

- Mock external dependencies
- **Adapter Testing**: Only tests for adapters should invoke actual external dependencies. Other services should use mocks or stubs of the adapters.

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
- **E2E Frameworks**: Consider using pytest with appropriate fixtures, or frameworks like `behave` for BDD-style E2E tests.

#### Additional Testing Practices

- Use parametrize for multiple test cases:
  ```python
  @pytest.mark.parametrize("input,expected", [
      (2, 4),
      (3, 9),
      (4, 16),
  ])
  def test_square(input, expected):
      assert square(input) == expected
  ```

### Performance

- Profile before optimizing (use cProfile or line_profiler)
- Use built-in functions and libraries (they're usually optimized in C)
- Prefer list comprehensions over loops for simple transformations
- Use generators for memory efficiency
- Consider using `functools.lru_cache` for expensive computations
- Use `collections.deque` for queue operations
- Leverage NumPy/Pandas for numerical operations

### Concurrency

- Use `asyncio` for I/O-bound concurrent operations:
  ```python
  async def fetch_data(url: str) -> dict:
      async with aiohttp.ClientSession() as session:
          async with session.get(url) as response:
              return await response.json()
  ```
- Use `threading` for I/O-bound parallel operations
- Use `multiprocessing` for CPU-bound parallel operations
- Be aware of the GIL (Global Interpreter Lock) limitations
- Use `concurrent.futures` for simple parallelism

### Data Validation

- Use Pydantic for data validation and settings:

  ```python
  from pydantic import BaseModel, EmailStr, validator
  from typing import Optional

  class UserModel(BaseModel):
      id: int
      name: str
      email: EmailStr
      age: Optional[int] = None

      @validator('age')
      def validate_age(cls, v):
          if v is not None and (v < 0 or v > 150):
              raise ValueError('Age must be between 0 and 150')
          return v
  ```

- Validate at system boundaries
- Use type checking with mypy or pyright

### Logging

- Use the `logging` module, not `print()`:

  ```python
  import logging

  logger = logging.getLogger(__name__)

  logger.debug("Debug information")
  logger.info("Informational message")
  logger.warning("Warning message")
  logger.error("Error message")
  logger.critical("Critical error")
  ```

- Configure logging properly in main entry point
- Use structured logging when appropriate
- Include context in log messages

### Security

- Never hardcode secrets or credentials
- Use environment variables or secure vaults for sensitive data
- Validate and sanitize all user inputs
- Use parameterized queries for database operations
- Keep dependencies updated (`pip list --outdated`)
- Use `secrets` module for cryptographic randomness

### Common Libraries

- **Web frameworks**: FastAPI, Flask, Django
- **Data processing**: pandas, numpy, polars
- **HTTP requests**: httpx, requests, aiohttp
- **Testing**: pytest, pytest-mock, pytest-asyncio
- **Validation**: pydantic, marshmallow
- **CLI**: click, typer, argparse
- **Database**: SQLAlchemy, psycopg2, pymongo
- **Task queues**: celery, rq
- **Date/time**: pendulum, arrow, python-dateutil

### Project Structure

```
project/
├── src/
│   └── myproject/
│       ├── __init__.py
│       ├── main.py
│       ├── models/
│       ├── services/
│       ├── utils/
│       └── api/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── conftest.py
├── requirements.txt
├── requirements-dev.txt
├── pyproject.toml
├── setup.py
├── README.md
└── .gitignore
```

### Configuration

- Use environment variables for configuration
- Consider using Pydantic Settings:

  ```python
  from pydantic import BaseSettings

  class Settings(BaseSettings):
      app_name: str = "My App"
      debug: bool = False
      database_url: str

      class Config:
          env_file = ".env"
  ```

- Keep configuration separate from code
- Use different configs for dev/test/prod

## Version Control

- **Conventional Commits**: You MUST use **Conventional Commits** for all git commit messages.
- **User Confirmation**: You MUST always ask the user for confirmation before committing any files.
- **Cleanup**: You MUST clean up any temporary files or build artifacts before committing.

## Checklist Before Committing

- [ ] Code passes all linters (ruff, flake8, or pylint)
- [ ] Type checking passes (mypy or pyright)
- [ ] All tests pass (`pytest`)
- [ ] Test coverage meets 95% threshold (`pytest --cov --cov-fail-under=95`)
- [ ] Code formatted with Black or similar formatter
- [ ] No hardcoded secrets or credentials
- [ ] Docstrings for all public functions/classes
- [ ] Type hints for all function signatures
- [ ] No bare `except:` clauses
- [ ] No `print()` statements (use logging)
- [ ] No dead code (unused imports, variables, functions, classes)
- [ ] Dependencies are documented
- [ ] Security vulnerabilities checked (`pip audit` or `safety check`)
