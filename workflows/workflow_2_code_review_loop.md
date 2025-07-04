# Workflow 2: AI Code Review & Improvement Loop

## Purpose
Automated pull request/code review loop with smart suggestions and fixes.

## Agent Roles
- **Reviewer**: Inspects code for issues, referencing Windsurf rules.
- **Refactorer**: Fixes or improves code per review.
- **Verifier**: Runs tests or confirms all issues are resolved.

## Tech Stack
- Agno with JSON-structured output
- FastAPI endpoint `/review_pr`
- Pydantic models for comments and refactoring tasks
- Windsurf rule files for standards
- Optional vector DB of past code reviews

## Deployment
- Local: Git pre-commit hook, Cascade plugin
- Cloud: CI/CD pipeline integration

## Use Cases
- PR auto-review and fix
- Continuous refactoring enforcement
- Instant local code suggestions
