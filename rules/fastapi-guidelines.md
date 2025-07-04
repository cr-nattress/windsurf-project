# FastAPI Usage Guidelines

## Description
This rule outlines best practices for building API layers for agents using FastAPI. The goal is to maintain lightweight, clear, and well-documented APIs suitable for development and production.

## Guidelines
- Use `async def` for all route handlers
- Use Pydantic models for request and response validation
- Organize endpoints into FastAPI `APIRouter` modules
- Return structured error responses with `HTTPException`
- Use `ResponseModel` for automatic docs
- Keep endpoints thin—business logic belongs in services or tools
- Use environment-based configuration via `pydantic.BaseSettings`

<coding_guidelines>
- Use `/health` and `/version` endpoints for diagnostics
- Serve docs at `/docs` (Swagger) and `/redoc`
</coding_guidelines>
