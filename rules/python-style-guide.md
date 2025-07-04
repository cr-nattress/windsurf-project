# Python Style Guide

## Description
This rule defines the code style standards for all Python files used in Windsurf-based AI agent projects. It ensures clean, maintainable, and consistent code across the project, following PEP8 with some pragmatic deviations.

## Guidelines
- Follow [PEP8](https://peps.python.org/pep-0008/) as the baseline
- Limit line length to 100 characters
- Use `black` for code formatting, and `isort` for import sorting
- Use type hints and `mypy` to enforce static typing
- Avoid deeply nested logic—use early returns instead
- Keep functions small and focused (ideally <30 lines)
- Document all public functions with docstrings
- Avoid single-letter variable names except for loops or indexes

<coding_guidelines>
- Always define `__all__` in modules to make exports explicit
- Use f-strings instead of `%` or `.format()`
- Prefer list comprehensions over manual loops
- Do not use wildcard imports
</coding_guidelines>
