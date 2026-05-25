# Agent Personas & The ReAct Framework

If the Large Language Model (LLM) is the "engine" of an AI agent, then System Prompts are the steering wheel, and the ReAct Framework is the transmission that translates engine power into forward momentum.

This document explains how developers define an agent's identity and constraints, and how the agent practically executes its reasoning loop.

## 1. Agent Personas & System Prompts

A standard LLM is a generalist. To make it a useful agent, you must restrict its scope, give it a specific role, and teach it how to use its tools. This is done via the System Prompt (or "Persona").

### 1.1 What is a System Prompt?

A system prompt is a hidden set of overarching instructions injected into the LLM at the very beginning of its context window. It acts as the immutable laws governing the agent's behavior.

### 1.2 Key Elements of an Agentic Persona

Designing a robust system prompt for an agent requires specific components:

- **Role Definition**: "You are an expert financial analysis agent." (Anchors the LLM's latent space to financial knowledge).

- **Scope & Guardrails**: "You only answer questions related to the stock market. If asked about cooking, politely decline."

- **Tool Documentation**: Explicitly listing the tools available and exactly how the LLM should format its output to trigger them (e.g., To search the web, output exactly: ACTION: web_search | QUERY: [your query]).

**Format Requirements**: Forcing the LLM to adhere strictly to an execution framework (like ReAct).

### 1.3 Example System Prompt snippet

```text
You are 'DataBot', a highly accurate research assistant. 
Your goal is to answer user queries truthfully. 
You have access to the following tools:
1. [WikipediaSearch]: searches Wikipedia.
2. [Calculator]: evaluates math expressions.

You MUST use the following format for every step:
- Thought: think about what to do next
- Action: the tool to use, must be one of [WikipediaSearch, Calculator]
- Action Input: the input to the tool
Observation: the result of the action

If you know the final answer, output:
Final Answer: [the answer]
```

## 2. The ReAct Framework

First introduced in a 2022 research paper, ReAct (Reason + Act) is the most foundational prompting strategy for autonomous agents.

Before ReAct, models were prompted to either reason (Chain of Thought) OR act (generate a command). ReAct proved that interleaving reasoning and acting creates a massive boost in an agent's ability to solve complex, multi-step problems.

### 2.1 The ReAct Cycle

The ReAct cycle forces the LLM to operate in a strict, step-by-step format:

- Thought: The agent reasons about its current state, the overall goal, and what information it is missing.

- Action: The agent decides to use a specific tool to get the missing information. The system pauses the LLM, executes the tool, and retrieves the data.

- Observation: The system feeds the tool's result back into the LLM.
(The loop repeats until the LLM's "Thought" determines the goal is complete).

# 3. ReAct in Practice: A Concrete Example

User Goal: "Who is the current CEO of Apple, and how old were they in 2010?"

(The system injects the Persona, the User Goal, and begins the ReAct loop)

**Step 1**

- Thought: I need to find out who the current CEO of Apple is. I will use the Web Search tool.

- Action: WebSearch("current CEO of Apple")

- Observation: "Tim Cook is the chief executive officer of Apple Inc."

**Step 2**

Thought: The CEO is Tim Cook. Now I need to find out his birth year to calculate his age in 2010.

- Action: WebSearch("Tim Cook birth year")

- Observation: "Timothy Donald Cook was born on November 1, 1960."

**Step 3**

- Thought: Tim Cook was born in 1960. I need to calculate his age in the year 2010. I will subtract 1960 from 2010.

- Action: Calculator("2010 - 1960")

- Observation: "50"

**Step 4**

- Thought: I now have all the information needed to answer the user's question. I can formulate the final answer.

- Final Answer: "The current CEO of Apple is Tim Cook. In the year 2010, he was 50 years old."

# 4. Why This Works

If you asked a static LLM this question, it might hallucinate a math error, or it might confidently state outdated information.

By combining a strict Persona with the ReAct Framework:

- The agent is forced to verify facts (WebSearch) instead of guessing.

- It offloads tasks it is bad at (like arithmetic) to specialized tools (Calculator).

- It creates a traceable, auditable log of why it made the decisions it did, making AI systems safer and much easier to debug.