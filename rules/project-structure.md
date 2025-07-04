# Project Structure and File Layout

## Description
Defines the expected file structure for Windsurf Python AI agent projects. Enforces consistency across projects and agents.

## Guidelines
```
project-root/
├─ run.py
├─ .env
├─ .windsurf/
│  └─ rules/
├─ src/
│  ├─ agents/
│  │  ├─ primary.py
│  │  └─ team_agent.py
│  ├─ tools/
│  ├─ api/  # FastAPI routes
│  ├─ db/
│  ├─ settings.py
│  └─ utils/
├─ tests/
│  ├─ test_agents.py
│  └─ test_tools.py
```
- Use `run.py` as canonical entrypoint
- Store agents in `src/agents/`, tools in `src/tools/`
- Keep rules in `.windsurf/rules/`
- Use `.env` and `settings.py` for config
