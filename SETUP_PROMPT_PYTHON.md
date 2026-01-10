# Python Project Setup Instructions

Set up a Python development environment with strict type checking, comprehensive linting, and automatic code formatting.

## Tools Configured

- **Python**: Target 3.11+ for modern type hints
- **Ruff**: Code quality, formatting, and import ordering (replaces Black, isort, Flake8)
- **mypy**: Strict static type checking
- **pytest**: Testing with coverage reporting

## Installation

```bash
pip install ruff mypy pytest pytest-cov
```

Or add to pyproject.toml:
```bash
pip install -e ".[dev]"
```

## Configuration Details

### pyproject.toml

Maximum strictness configuration:

```toml
[project]
name = "your-project"
version = "0.1.0"
requires-python = ">=3.11"

[project.optional-dependencies]
dev = [
    "ruff>=0.4.0",
    "mypy>=1.10.0",
    "pytest>=8.0.0",
    "pytest-cov>=5.0.0",
]

[tool.ruff]
target-version = "py311"
line-length = 88
src = ["src", "tests"]

[tool.ruff.lint]
select = [
    "E",      # pycodestyle errors
    "W",      # pycodestyle warnings
    "F",      # Pyflakes
    "I",      # isort
    "B",      # flake8-bugbear
    "C4",     # flake8-comprehensions
    "UP",     # pyupgrade
    "ARG",    # flake8-unused-arguments
    "SIM",    # flake8-simplify
    "TCH",    # flake8-type-checking
    "PTH",    # flake8-use-pathlib
    "ERA",    # eradicate (commented-out code)
    "PL",     # Pylint
    "RUF",    # Ruff-specific rules
    "N",      # pep8-naming
    "D",      # pydocstyle
]
ignore = [
    "D100",    # Missing docstring in public module
    "D104",    # Missing docstring in public package
    "PLR0913", # Too many arguments (handled by design review)
]

[tool.ruff.lint.pydocstyle]
convention = "google"

[tool.ruff.lint.isort]
known-first-party = ["your_package"]
force-sort-within-sections = true
combine-as-imports = true

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
docstring-code-format = true

[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
warn_unused_ignores = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
disallow_untyped_decorators = true
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_configs = true
show_error_codes = true
enable_error_code = ["ignore-without-code", "redundant-cast", "truthy-bool"]

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "--cov=src --cov-report=term-missing --cov-fail-under=95"
```

## Makefile Scripts

Add these scripts to your project:

```makefile
.PHONY: format lint type-check test coverage check

format:
	ruff format .

lint:
	ruff check . --fix

type-check:
	mypy src

test:
	pytest

coverage:
	pytest --cov=src --cov-report=html --cov-fail-under=95

check: format lint type-check test
```

## Required Workflow

After making any code changes, run:

```bash
ruff format .     # Format code
ruff check .      # Check linting
mypy src          # Type check
pytest            # Run tests with coverage
```

Or run all at once: `make check`

## Docstring Requirements

All public functions, classes, and modules require docstrings (Google style):

```python
def calculate_total(price: float, tax_rate: float) -> float:
    """Calculate the total price including tax.

    Args:
        price: The base price in dollars.
        tax_rate: The tax rate as a decimal (e.g., 0.08 for 8%).

    Returns:
        The total price including tax.

    Raises:
        ValueError: If price or tax_rate is negative.

    Example:
        >>> calculate_total(100.0, 0.08)
        108.0
    """
    if price < 0 or tax_rate < 0:
        raise ValueError("Price and tax_rate must be non-negative")
    return price * (1 + tax_rate)


@dataclass
class Config:
    """Configuration for the application.

    Attributes:
        port: The port to listen on.
        host: The host address to bind to.
    """

    port: int
    host: str
```

## Type Hints Requirements

All functions require explicit type hints - use `typing` module for complex types:

```python
from typing import Optional, List, Dict, TypeVar, Generic, Any
from dataclasses import dataclass

T = TypeVar("T")


def fetch_users(
    limit: int,
    offset: int = 0,
    active_only: bool = True,
) -> List[Dict[str, Any]]:
    """Fetch users from the database."""
    ...


@dataclass
class Result(Generic[T]):
    """A result type for operations that may fail."""

    success: bool
    value: Optional[T] = None
    error: Optional[str] = None
```
