1. Core Concepts & Foundations
What is an AI Agent? Defining the transition from static LLM generation to autonomous, goal-oriented execution.

The LLM as a Reasoning Engine: How models plan, decide, and parse information rather than just predicting text.

Agent Personas & System Prompts: Designing instructions that govern an agent's behavior and scope.

The ReAct Framework: The cycle of Reasoning and Acting (Thought, Action, Observation).

2. RAG & Knowledge Retrieval (New)
The RAG Pipeline: The core architecture of Ingestion, Indexing, Retrieval, and Generation.

Document Processing:

Using document loaders (PDFs, Markdown, web scraping).

Chunking strategies: recursive character splitting, semantic chunking, and handling overlap.

Vector Stores & Similarity Search: Integrating databases like Chroma, FAISS, Pinecone, or pgvector.

Advanced Retrieval Techniques:

Hybrid search (keyword + semantic).

Cross-encoders and Reranking to improve context precision.

Self-Querying retrievers (translating natural language to metadata filters).

Agentic RAG: The shift from a static pipeline to building agents that can dynamically choose when to search, how to refine their queries, and whether they need to search again.

2. Frameworks & Ecosystems
LangChain Fundamentals: Building chains, binding tools to models, and handling output parsers.

LangGraph Deep Dive:

Transitioning from linear chains to cyclic graphs.

Managing shared state across nodes.

Using state reducers to merge and update information during execution.

Alternative Frameworks: Brief comparisons with frameworks like AutoGen, CrewAI, or Semantic Kernel.

3. Tooling & Actions (The Agent's Hands)
Defining Tools: Writing Python functions and wrapping them with Pydantic schemas for LLM ingestion.

Tool Binding & Invocation: How models decide when to call a tool and how to parse the arguments.

Standard Integrations: Web search, calculator, file I/O, and API execution.

Error Handling in Tools: Strategies for when a tool fails or returns an unexpected format, allowing the agent to self-correct.

4. Memory & State Management
Short-Term Memory: Managing the conversation window and token limits.

Long-Term Memory: Vector databases, embedding retrieval, and RAG (Retrieval-Augmented Generation) within agents.

Persistence & Checkpointing: Using databases (like SQLite or Postgres) to save state and resume interrupted graphs.

5. Multi-Agent Systems
Architectural Patterns:

Supervisor/Worker: A central agent delegating tasks to specialized sub-agents.

Hierarchical Teams: Complex graphs where nodes are entire sub-graphs.

Collaborative Chat: Agents passing a shared state back and forth.

Handling Handoffs: Managing the transition of context and control between different agents securely.

6. Local Models & Infrastructure
Running Local LLMs: Integrating Ollama to run models like Llama 3 or Mistral directly on your hardware.

Prompting for Local Models: Adjusting tool descriptions and system prompts for models with smaller parameter counts.

Serving Agents: Wrapping agent graphs in REST APIs (using FastAPI) or user interfaces (like Streamlit).

7. Evaluation & Observability
Tracing Execution: Tracking tool inputs, outputs, and model reasoning steps.

Debugging Graphs: Visualizing the flow of a LangGraph execution and inspecting state transitions.

Evaluating Agent Performance: Frameworks for testing whether an agent reliably achieves its target outcome without hallucinating tool calls.