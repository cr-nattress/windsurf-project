# Agent Tool Usage Standards

## Description
Outlines best practices for integrating and managing tools inside Agno agents. Tools should be composable, purpose-driven, and safe.

## Guidelines
- Name tools clearly, e.g., `WebSearchTool`, `CalendarQueryTool`
- Each tool should have a clear purpose (one job)
- Tool input/output contracts should be Pydantic models
- Tools should log usage or return errors clearly
- Avoid calling tools from tools (avoid nested tool logic)
- Use `Tool` subclasses to encapsulate 3rd-party APIs

<agent_patterns>
- Register tools explicitly during agent instantiation
- Avoid tools that mutate shared state without memory injection
</agent_patterns>
