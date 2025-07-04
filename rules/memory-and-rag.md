# Agent Memory and RAG Use

## Description
Best practices for working with memory-enhanced agents and Retrieval-Augmented Generation (RAG) pipelines.

## Guidelines
- Use separate vector stores per knowledge domain
- Store embeddings alongside metadata (e.g., source, timestamp)
- Use `VectorMemory` or subclassed memory store for injection
- Clean and chunk documents consistently before ingestion
- Prefer retrieval pipelines with semantic filters or hybrid scoring
- Use local LanceDB, Chroma, or Redis for dev; scale to Pinecone, Weaviate, etc. in prod

<agent_patterns>
- Persist memory between sessions with versioned snapshots
- Use memory to store feedback loops, not just documents
</agent_patterns>
