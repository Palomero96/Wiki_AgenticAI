# LangGraph

While LangChain Chains (using LCEL) are excellent for predictable and straightforward workflows (`A → B → C`), true autonomous agents rarely operate in a linear fashion. The ReAct framework requires cycles: the agent must think, act, observe, and then think again based on what it just observed.

This is where LangGraph comes in. It is an extension of LangChain designed specifically for creating cyclic, stateful agents as graphs (networks of interconnected nodes).

## 1. From Linear Chains to Cyclic Graphs

In a traditional chain (`Prompt | LLM | OutputParser`), data flows in a single direction. If the response requires correction or additional steps, the chain cannot "go back" on its own.

LangGraph models execution as a Directed Graph:

* **Nodes:** These are Python functions or Chains. They represent the "work" being done (e.g., "Call the LLM" or "Execute a tool").
* **Edges:** These define where the execution flow should go after a node finishes.
* **Conditional Edges:** This is the key to agentic reasoning. They are routing functions that dynamically decide the next step. For example: *"If the LLM decided to use a tool, go to the tool node. If the LLM gave the final answer, go to the end node."*

### The Basic Agent Graph (ReAct in LangGraph)

```text
       +-------------------+
       |                   |
-----> |     LLM Node      | <------+
       |   (Reasoning)     |        |
       +-------------------+        |
                 |                  |
         [Conditional Edge]         | (Loop / Cycle)
          /              \          |
  (Tool invoked)    (Final Answer)  |
        /                  \        |
+-------------------+       [END]   |
|    Tool Node      |               |
|   (Execution)     |               |
+-------------------+               |
        |                           |
        +---------------------------+
```

## 2. Managing Shared State

For a loop (cycle) to work, the system must remember what happened in the previous iteration. LangGraph introduces the concept of a `State` that is passed from node to node throughout the entire execution of the graph.

### Defining the State

The state is typically defined using a Python `TypedDict` (or a Pydantic model). Every node in the graph receives this state as input, performs its work, and returns an update to that state.

```python
from typing import TypedDict, Annotated

class AgentState(TypedDict):
    # The list of all messages exchanged (user, AI, tools)
    messages: list
    # A string to keep track of the current plan
    current_plan: str
    # A counter to prevent infinite loops
    iteration_count: int
```

### State Flow Example:

* **Start:** The user provides an initial state: `{"messages": ["What is the weather like in Madrid?"]}`.
* **LLM Node:** Reads the state, decides to use the weather tool, and returns the update: `{"messages": [AIMessage(tool_call="get_weather", args={"city": "Madrid"})]}`.
* **Tool Node:** Reads the state, executes the function, and returns the update: `{"messages": [ToolMessage(content="22 degrees and sunny")]}`.
* **LLM Node (2nd iteration):** Reads the complete history, formulates the final response: `{"messages": [AIMessage(content="It is 22 degrees and sunny in Madrid.")]}`.

## 3. Reducers: Merging and Updating Information

When a node returns a state update (for example, the Tool Node returns a `ToolMessage`), how is that update combined with the existing global state? This is where Reducers come in.

If no reducer is specified, LangGraph simply overwrites the previous value.

### The Overwriting Problem

If our state is `{"messages": ["Hello"]}` and the LLM node returns `{"messages": ["How can I help you?"]}`, simple overwriting will cause us to lose the original "Hello". The new state would just be `{"messages": ["How can I help you?"]}`.

### The Solution: Annotated and Reducer Functions

For critical attributes like the message history, we need the state to be appended to, not overwritten. We use `Annotated` and LangChain's pre-built `add_messages` function.

```python
from typing import TypedDict, Annotated
from langgraph.graph.message import add_messages

class AgentState(TypedDict):
    # 'add_messages' acts as the reducer.
    # When a node returns new messages, they are APPENDED to the existing list,
    # instead of wiping out the previous list.
    messages: Annotated[list, add_messages]
    
    # Since it has no annotated reducer, this value WILL BE OVERWRITTEN 
    # every time a node modifies it.
    current_status: str 
```

### Why are Reducers vital?

* **Conversational Memory:** They allow the graph to maintain the full context of the chat history (`add_messages` is the most common use case).
* **Complex Structures:** You can write custom reducer functions in Python to perform complex merging logic (e.g., merging dictionaries, or keeping only the 5 most recent items in a short-term memory list).
* **Parallel Work:** If multiple nodes operate in parallel within a complex graph, the reducer defines how conflicts are resolved when different nodes attempt to update the global state simultaneously.

## Conclusion

LangGraph transforms LLM orchestration by enabling:

* **Cyclic Graphs:** Agents can enter ReAct loops (Think → Act → Observe), correcting errors and fetching data until the task is complete.
* **Persistent State:** A single state object travels through the graph, providing short-term memory.
* **Smart Reducers:** They define strict rules on how new information merges with old information, preventing data loss and maintaining coherent chat histories.