# Multi-Agent Patterns

## Description
Guidelines for building multi-agent systems in Agno. This includes collaborative agents, handoff patterns, and orchestration strategies.

## Guidelines
- Assign each agent a distinct, single responsibility
- Use structured message formats for inter-agent comms
- Avoid global state—use shared memory vectors or event buses
- Validate assumptions about upstream/downstream agent output
- Log transitions between agents for observability

<agent_patterns>
- Use an “Orchestrator” agent to manage parallel or sequenced agents
- Design each agent as stateless except when using memory modules
</agent_patterns>
