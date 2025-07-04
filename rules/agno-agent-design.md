# Agno Agent Design Principles

## Description
Defines architectural principles and design strategies for building agents using the Agno framework. Agents should be modular, explainable, and robust to edge cases.

## Guidelines
- Use `ReasoningAgent` for core agents, extend only when necessary
- Clearly define the agent’s goal in its description
- Use declarative construction: pass tools, memory, and model explicitly
- Inject dependencies (e.g., vector DB, tools) via `kwargs` or factories
- Keep tools and models separate from agent logic
- Compose tools with descriptive names and scoped capabilities
- Document agent capabilities with example prompts

<agent_patterns>
- Encapsulate logic-heavy tools inside `Tool` subclasses, not agents
- Use a separate memory store per agent if goals diverge
</agent_patterns>
