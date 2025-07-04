# Testing and Observability

## Description
Guidelines for testing your agents and surfacing observability data (logs, traces, metrics) during development and production.

## Guidelines
- Write unit tests for tools and services using `pytest`
- Test agents with scripted conversation flows
- Use `agent.run(message, trace=True)` for interactive trace logs
- Log all tool calls and decisions via structured logging
- Export traces to JSONL or external systems if needed
- Use `print_response(stream=True)` for debugging outputs

<agent_patterns>
- Include a sample conversation per agent as a test fixture
- Test against both successful and failure scenarios
</agent_patterns>
