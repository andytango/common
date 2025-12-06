# Python Development Guidelines

## Core Principles

- Use the simplest possible solution
- Prefer procedural programming and functional programming paradigms
- **Simplicity**: Use the simplest possible solution. Service Oriented Architecture is intended for COMPLEX projects. For simple tasks, keep it simple.
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

## Architecture & Design

- **Service Oriented Architecture**: For **COMPLEX** projects, decompose systems into small, individually testable services located under a `lib` folder.
- **Implementation Style**: When implementing services, prefer a **procedural** or **functional** style over complex object-oriented patterns.
- **Dependency Structure**: Design services with orthogonality, layering, and proper abstraction levels in mind. Tend towards a tree or diamond dependency pattern. Apply **SOLID principles**.
- **Adapter Pattern**: Always wrap external APIs or services in an adapter service with minimal logic.
- **Design Review**: You MUST always have the user review your service design before implementation.
- **Test Confirmation**: You MUST always ask the user if they would like tests to be added when performing a task.
- **Manual Testing**: If the user declines automated tests, you MUST try to test manually. If you don't know how to test manually, you MUST ask the user for help.

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

### Code Style (PEP 8)

- Follow PEP 8 style guide
- Use 4 spaces for indentation (never tabs)
- Maximum line length of 88 characters (Black formatter default)
- Use descriptive variable names:
  - `snake_case` for functions, variables, modules
  - `PascalCase` for classes
  - `UPPER_SNAKE_CASE` for constants
- Two blank lines between top-level definitions
- One blank line between methods

### Docstrings

- Use docstrings for all public modules, functions, classes, and methods:

  ```python
  def fetch_user_data(user_id: int, include_profile: bool = False) -> Dict[str, Any]:
      """
      Fetch user data from the database.

      Args:
          user_id: The unique identifier of the user.
          include_profile: Whether to include profile information.

      Returns:
          A dictionary containing user data.

      Raises:
          UserNotFoundError: If the user does not exist.
          DatabaseConnectionError: If database connection fails.

      Example:
          >>> user = fetch_user_data(123, include_profile=True)
          >>> print(user['name'])
      """
      ...
  ```

- Use Google, NumPy, or Sphinx docstring style consistently
- Include type information in docstrings if not using type hints

### Error Handling

- Use specific exception types:

  ```python
  class ValidationError(Exception):
      """Raised when validation fails."""
      pass

  class ConfigurationError(Exception):
      """Raised when configuration is invalid."""
      pass
  ```

- Never use bare `except:` clauses
- Always specify exception type:
  ```python
  try:
      result = risky_operation()
  except (ValueError, TypeError) as e:
      logger.error(f"Operation failed: {e}")
      raise
  ```
- Use `finally` for cleanup operations
- Consider using context managers for resource management

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

### Imports

- Order imports according to PEP 8:
  1. Standard library imports
  2. Related third-party imports
  3. Local application imports
- Use absolute imports when possible
- Avoid wildcard imports (`from module import *`)
- Group imports logically:

  ```python
  import os
  import sys
  from datetime import datetime
  from pathlib import Path

  import numpy as np
  import pandas as pd
  from pydantic import BaseModel, Field

  from myapp.models import User
  from myapp.utils import calculate_hash
  ```

### Code Organization

- **Declaration Order**: To improve readability and maintainability, declarations should be logically ordered. While Python's dynamic nature is flexible, a good practice is to order declarations based on reverse dependency and the likely call sequence:
  1.  **Constants**: `UPPER_SNAKE_CASE` variables.
  2.  **Public Classes**: Classes intended for external use.
  3.  **Public Functions**: Functions intended for external use.
  4.  **Private Classes**: Internal helper classes, prefixed with an underscore (`_`).
  5.  **Private Functions**: Internal helper functions, prefixed with an underscore (`_`).

### Testing

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
- [ ] Code formatted with Black or similar formatter
- [ ] No hardcoded secrets or credentials
- [ ] Docstrings for all public functions/classes
- [ ] Type hints for all function signatures
- [ ] No bare `except:` clauses
- [ ] No `print()` statements (use logging)
- [ ] Dependencies are documented
- [ ] Security vulnerabilities checked (`pip audit` or `safety check`)
- [ ] Code coverage meets project standards
