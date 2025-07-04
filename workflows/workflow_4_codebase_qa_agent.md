# Workflow 4: Codebase Knowledge Q&A Agent

## Purpose
AI assistant to answer developer questions about codebase using RAG.

## Agent Roles
- **Query Analyzer**: Understands query scope.
- **Retriever**: Searches vector DB or Cascade context.
- **Responder**: Answers using context, with citations.

## Tech Stack
- Agno (retrieval agent)
- Vector DB (LanceDB, Chroma, etc.)
- Windsurf memory + Cascade context
- FastAPI endpoint `/ask`
- Pydantic for structured Q&A input/output

## Deployment
- Local: IDE plugin or CLI bot
- Cloud: Dev portal bot or Slack app

## Use Cases
- “How is login handled?”
- “Where is X defined?”
- “Generate a summary of class Y”
