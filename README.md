**Ember AI — Stateful Conversational AI Platform**

Ember AI is a full-stack conversational AI backend designed around persistent memory, retrieval-augmented generation (RAG), and graph-based reasoning orchestration.
The system supports authenticated multi-chat sessions, document-scoped knowledge retrieval, real-time streaming responses, and long-running conversations with controlled context growth.
This project emphasizes system design, robustness, and extensibility rather than a minimal demo implementation.

**Overview**

The core objective of Ember AI is to provide a stable, stateful conversational experience that scales beyond stateless prompt-response interactions.
Key goals of the system include:
-Maintaining long-term conversational context without uncontrolled token growth
-Enabling document-aware conversations through per-chat RAG
-Supporting real-time streaming responses
-Separating reasoning, tool execution, and summarization into explicit control flow
-Persisting all relevant state across restarts
The backend is implemented using FastAPI and LangGraph, with PostgreSQL serving both as a relational datastore and a vector database (via pgvector).

**High-Level Architecture**

At a system level, Ember AI is composed of the following layers:
**1. API Layer (FastAPI)**

Exposes authenticated REST endpoints for:
-Chat lifecycle management
-Conversational interaction
-Document upload and ingestion
-Speech-to-text (ASR)
-Handles request validation, streaming responses, and error handling
-Enforces user authentication via Supabase JWTs

**2. Conversational Orchestration (LangGraph)**

Conversations are modeled as a state machine, not a linear chain
Explicit nodes handle:
-LLM reasoning
-Tool execution
-Context summarization
-Routing logic determines when tools are invoked and when summaries are generated
-PostgreSQL-backed checkpointing ensures full persistence of graph state

**3. AI & Reasoning Layer**

Primary LLM (streaming enabled) handles conversational reasoning and tool selection.
Secondary LLM handles auxiliary tasks such as:
-Conversation summarization
-Translation
-Rewriting
-Automatic chat title generation

A fixed system identity ensures consistent assistant behavior across sessions

**4. Retrieval-Augmented Generation (RAG)**

-Documents are uploaded per chat and ingested into a vector store
-Each chunk is embedded using a normalized embedding model
-Retrieval is strictly scoped to the active chat thread
-Retrieved context is injected transparently without exposing retrieval mechanics to the model

**5. Persistence Layer**

Supabase (PostgreSQL) stores:
-User accounts and authentication data
-Chat metadata
-Message history
-Document metadata
pgvector is used for efficient semantic similarity search
LangGraph checkpoints are stored directly in PostgreSQL for durability

**Core Features**

-Authenticated multi-chat sessions with isolated state per conversation
-Graph-based reasoning flow using LangGraph
-Rolling conversation summarization to prevent context overflow
-Per-chat document ingestion and retrieval
-Token-level streaming responses
-Tool-augmented reasoning, including:
                                      -Web search
                                      -Weather queries
                                      -Summarization
                                      -Translation
                                      -Rewriting
                                      -Automatic chat title generation
                                      -Speech-to-text (ASR) using Whisper via Groq
                                      -Persistent state across restarts
**Technology Stack**

-**Backend**
  -Python
  -FastAPI
  -LangChain
  -LangGraph

-**AI & ML**
  -Groq-hosted LLMs
  -HuggingFace embeddings (intfloat/e5-large-v2)
  -Whisper (speech-to-text)

-**Data & Storage**
  - PostgreSQL
  -pgvector
  -Supabase

-**Infrastructure**
  -Async PostgreSQL connection pooling
  -PostgreSQL-backed LangGraph checkpointing
  -Token streaming over HTTP
  -Conversation State Management

Rather than storing entire conversation histories indefinitely, Ember AI uses a rolling summarization strategy:
-Recent messages are kept verbatim
-Older messages are compressed into a concise summary
-The summary is continuously updated as conversations evolve
This preserves semantic continuity while keeping token usage bounded
This approach allows long-running conversations without degrading performance or exceeding context limits.

**Retrieval-Augmented Generation Design**

RAG in Ember AI is intentionally scoped and controlled:
-Documents are associated with a specific chat
-Retrieval queries only search within that chat’s documents
-Embeddings are normalized for cosine similarity
The LLM is explicitly instructed to:
-Use retrieved context as the primary source of truth
-Avoid hallucination when context is insufficient
-Avoid referencing internal retrieval mechanics
This design prevents cross-chat data leakage and keeps retrieval behavior predictable.

**Authentication & Security**

-User authentication is handled via Supabase
-JWTs are validated on every protected endpoint
-All chat data is scoped to the authenticated user
-Refresh tokens are supported for session continuity

**Environment Variables**

The following environment variables are required:
-SUPABASE_URL
-SUPABASE_KEY
-SUPABASE_DB_URL
-GROQ_API_KEY
-TAVILY_API_KEY
-WEATHER_API_KEY

**Known Limitations**

-No rate limiting is currently enforced
-Vector storage assumes pgvector availability
-Authorization is enforced at the API layer, not per-graph node

**Future Improvements**

-Role-based access control
-Improved document metadata filtering
-Fine-grained tool permissioning
-Conversation export and analytics
-Background ingestion for large documents
