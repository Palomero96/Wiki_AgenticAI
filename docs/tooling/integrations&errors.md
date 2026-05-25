# Standard Integrations & Error Handling in Agent Tools

Building custom tools is powerful, but most AI agents rely on a core set of standard integrations to interact with the world. Furthermore, because the real world involves network latency, API limits, and unpredictable formatting, an agent is only as good as its ability to handle errors and self-correct.

This document explores the foundational tools every agent should have and the strategies required to keep them running smoothly.

## 1. Standard Integrations: The Agent's Toolkit

Before writing complex custom logic, most developers equip their agents with these fundamental capabilities.

### 1.1 Web Search (Overcoming Knowledge Cutoffs)

LLMs have a static training data cutoff. To access current events, they need search tools.

* **Modern Standard:** APIs like Tavily, Exa (Metaphor), or Brave Search are specifically designed for LLMs. Unlike traditional Google scraping, these APIs return clean, parsed markdown of web pages, skipping ads and popups.
* **Use Case:** *"What were the stock market results today?"*

### 1.2 Calculator and Code Execution (Overcoming Poor Arithmetic)

LLMs predict the next token; they do not actually compute math. Asking an LLM to multiply `14523 * 891` often results in confident hallucinations.

* **The Solution:** A Calculator tool or a Python REPL (Read-Eval-Print Loop).
* **How it works:** The agent writes a Python script to calculate the math or process the data, executes it in a secure sandbox, and reads the output.

### 1.3 File I/O (Reading and Writing Local State)

For agents operating locally (like a coding assistant), they need to interact with the file system.

* **Standard Tools:** `ReadFile`, `WriteFile`, `ListDirectory`.
* **Security Warning:** Giving an LLM raw access to your file system is dangerous. Always restrict File I/O tools to a specific working directory to prevent the agent from accidentally deleting critical system files.

### 1.4 API Execution (Interacting with SaaS)

The generic interface for the modern web.

* **Implementation:** Usually implemented by providing the agent with OpenAPI specifications (Swagger files). The framework dynamically creates tools for every endpoint (e.g., `GET /users`, `POST /tickets`).
* **Use Case:** Reading emails from Gmail, creating Jira tickets, or posting messages to Slack.

## 2. Error Handling: Enabling Self-Correction

When a traditional Python script hits an HTTP 404 or a KeyError, it crashes. If an AI agent's tool crashes, the entire agent loop dies.

To achieve true autonomy, tools must capture errors and return them to the LLM as text. The LLM can then use its reasoning capabilities to fix the mistake and try again.

### Strategy 1: Catching Exceptions and Returning Strings

Never let a tool throw an unhandled exception. Wrap the core logic in a `try/except` block.

```python
from langchain_core.tools import tool
import requests

@tool
def fetch_user_data(user_id: str) -> str:
    """Fetches user data from the database. Requires a valid user_id."""
    try:
        response = requests.get(f"https://api.mycompany.com/users/{user_id}")
        response.raise_for_status() # Raises an error for 404, 500, etc.
        return response.json()
    except requests.exceptions.HTTPError as e:
        # Crucial: Return the error as a string, DO NOT raise it!
        return f"Error: Failed to fetch user. The API returned: {str(e)}. Please check if the user_id is correct."
    except Exception as e:
        return f"An unexpected system error occurred: {str(e)}"
```

**Why this works:** In the next ReAct cycle, the LLM reads the observation: *"Error: Failed to fetch user..."* It can then think: *"Ah, the user ID might be wrong. Let me search for the correct ID first."*

### Strategy 2: Handling Schema/Validation Errors

Sometimes the LLM hallucinates an argument or formats it incorrectly (e.g., passing "tomorrow" instead of "2026-05-26").
Frameworks like LangChain allow you to handle tool-calling errors gracefully. If `Pydantic` rejects the LLM's input, the framework can automatically feed the validation error back to the LLM.

```python
# Conceptual LangChain implementation
# If the LLM passes a bad argument, it receives this as an Observation:
# "ValidationError: 'date' must be in YYYY-MM-DD format. You provided 'tomorrow'."
```

### Strategy 3: Guardrails and Fallbacks

For highly unreliable APIs, you can implement fallback tools at the graph level (in LangGraph).

* **The Logic:** If `WebSearchAPI_1` times out -> Route to `WebSearchAPI_2`.

This ensures the agent doesn't get stuck in an infinite loop trying a broken endpoint.

## 3. The Self-Correction Loop in Action

Here is how a properly error-handled tool looks inside the agent's execution loop:

* **Thought:** I need to find the weather in Madrid. I will use the weather tool.
* **Action:** `GetWeather({"location": "Madrid, Spain"})`
* **Observation:** "Error: the 'location' argument must be a 3-letter airport code, not a city name."
* **Thought (Self-Correction):** The tool rejected my input. I need the 3-letter airport code for Madrid. I will search the web for it.
* **Action:** `WebSearch("Madrid airport 3 letter code")`
* **Observation:** "The IATA code for Adolfo Suárez Madrid–Barajas Airport is MAD."
* **Thought:** Now I have the correct code. I will try the weather tool again.
* **Action:** `GetWeather({"location": "MAD"})`
* **Observation:** "Temperature: 24°C, Sunny."
* **Final Answer:** The weather in Madrid is currently 24°C and Sunny.

## Conclusion

An agent's resilience is entirely dependent on how you design its tools. By providing standard integrations for math, search, and system interaction, and by strictly trapping errors and returning them as informative text, you transform fragile scripts into highly robust, self-healing autonomous systems.