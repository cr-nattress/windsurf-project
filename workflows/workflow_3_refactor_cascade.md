# Workflow 3: Intelligent Codebase Refactoring Cascade

## Purpose
Systematic multi-file refactoring and migration.

## Agent Roles
- **Analyzer**: Locates refactoring targets.
- **Planner**: Orders and scopes work.
- **Executor**: Applies changes across codebase.
- **Verifier**: Tests refactor outcome.

## Tech Stack
- Agno multi-agent
- Windsurf Cascade for context-aware edits
- FastAPI endpoint `/refactor`
- Pydantic models for task lists and outcomes
- Vector DB for semantic refactor detection

## Deployment
- Local: CLI or Cascade plugin
- Cloud: Automated service via GitOps or webhook

## Use Cases
- Python 2→3 or library migration
- Cross-project style enforcement
- Technical debt removal
