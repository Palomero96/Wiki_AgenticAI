# Handling Handoffs: Managing Context and Control

In a multi-agent system, the architecture (Supervisor, Sequential, etc.) dictates who talks to whom. However, the system's success relies entirely on handoffs: the actual mechanism of transferring control and context from one agent to another.

If a handoff is sloppy, the receiving agent loses the context of the user's original goal, hallucinates, or repeats work that has already been done.

## 1. The Challenge of Handoffs

When Agent A finishes its sub-task and passes control to Agent B, two critical things must happen:

* **Transfer of Control:** The system must stop Agent A's generation loop and start Agent B's loop.
* **Transfer of Context (State):** Agent B must receive the exact information it needs to continue the work, without being overwhelmed by Agent A's internal scratchpad.

If we simply pass the entire chat history to every agent, the context window fills up with irrelevant tool calls and observations, confusing the subsequent agents.

## 2. Strategies for Managing Context During Handoffs

There are three primary strategies for handling the state during a transition.

### 2.1 The Global State (The "Shared Blackboard")

In this approach (common in LangGraph), a single global state object (e.g., a dictionary or Pydantic model) is passed between all agents.

* **How it works:** Agent A updates specific fields in the global state (e.g., `state["research_notes"] = "..."`). Agent B reads those fields when it wakes up.
* **Pros:** Very structured; strict typing prevents data loss.
* **Cons:** Every agent has access to everything, which can lead to agents trying to solve parts of the problem outside their scope.

### 2.2 Message Filtering (The "Need to Know" Basis)

Instead of passing the entire global state, the routing mechanism (or the Supervisor) filters the message history before handing it to the next agent.

* **How it works:** The Supervisor strips out Agent A's internal reasoning (Thoughts and Observations) and only passes Agent A's final Final Answer to Agent B, along with the original user prompt.
* **Pros:** Keeps the context window clean; forces Agent B to focus only on the finished output of Agent A.
* **Cons:** Agent B might lack crucial context about why Agent A made certain decisions if that context was filtered out.

### 2.3 The "Summary" Handoff

For very long processes, even filtered messages become too long. The system forces the departing agent to write a summary of its work before the handoff.

* **How it works:** Before Agent A goes to sleep, it executes a final tool or prompt: *"Summarize your findings for the Writer Agent."* This summary is the only thing passed to Agent B.
* **Pros:** Highly efficient use of tokens; excellent for hierarchical sub-graphs.
* **Cons:** Requires additional LLM calls (compute cost) to generate the summaries.

## 3. Mechanisms of Transferring Control

How does the code actually move from one agent to the next?

### 3.1 The LangGraph Approach: Conditional Edges

In a graph-based framework, control transfer is handled by the framework's routing logic, outside of the LLM's direct text generation.

1. Agent A finishes its node.
2. The framework evaluates a predefined Python function (the conditional edge).
3. The Python function reads the State and decides: `return "agent_b"`.
4. The framework wakes up Agent B.

### 3.2 The Tool-Based Handoff (Agentic Routing)

In this approach, the LLM itself actively initiates the handoff by calling a specific tool.

* **How it works:** You give the Supervisor Agent a tool like `TransferTo(agent_name: str, context_message: str)`.
* **Execution:**
  * **Supervisor Thought:** *"The research is done. I need the writer."*
  * **Supervisor Action:** `TransferTo("writer_agent", "Here are the notes on batteries: ...")`
  * **The Framework:** The framework intercepts this tool call, pauses the Supervisor, and injects the `context_message` into the Writer Agent's prompt.

## 4. Example: A Clean Handoff Flow

Let's look at a well-designed handoff using the Tool-Based approach in a Supervisor architecture.

---- CONTROL TRANSFERS ---
---- CONTROL TRANSFERS ---
---- CONTROL TRANSFERS ---
**Scenario:** User asks for a data analysis report.

* **Supervisor:** Reads the prompt. Calls `TransferTo("Data_Analyst", "User needs Q3 revenue analysis.")`
  * *--- CONTROL TRANSFERS ---*
* **Data Analyst Agent:** Wakes up. Reads the context.
  * Uses SQL tools to get data.
  * Analyzes data.
  * Calls `TransferTo("Supervisor", "Q3 Revenue was $4M, up 10%.")`
  * *--- CONTROL TRANSFERS ---*
* **Supervisor:** Wakes up. Reads the context. Calls `TransferTo("Writer", "Draft a report stating Q3 revenue was $4M (up 10%).")`
  * *--- CONTROL TRANSFERS ---*
* **Writer Agent:** Wakes up. Drafts the report. Calls `TransferTo("Supervisor", "Draft complete: [Report Text]")`.

##  Conclusion

Effective handoffs are the glue that holds a multi-agent system together. By carefully choosing how to transfer control (via edges or specialized tools) and how to manage context (via shared state, filtering, or summarization), you ensure that specialized agents collaborate smoothly without polluting each other's context windows.
