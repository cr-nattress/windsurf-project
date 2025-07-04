# Pydantic Model Standards

## Description
Defines how to use Pydantic models effectively within your agent codebase for input validation, configuration, and serialization.

## Guidelines
- Inherit from `BaseModel` (v2) and use strict types
- Use `Field(...)` for required values and metadata
- Validate emails, URLs, and constrained fields with built-in types
- Use `model_dump()` instead of `.dict()` in Pydantic v2
- Separate config models (`BaseSettings`) from data models
- Prefer composition over inheritance for nested structures
- Document fields with `title` and `description` in Field()

<agent_patterns>
- Use validators (`@field_validator`, `@model_validator`) sparingly and clearly
- Do not override `__init__`; use `@model_validator(mode="before")` instead
</agent_patterns>
