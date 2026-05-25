# Defining Tools: Python Functions and Pydantic Schemas

For an AI agent to interact with the external world, it needs Tools. A tool is simply a bridge between the Large Language Model's text generation capabilities and a classical programming function (like querying a database, fetching a web page, or calculating a value).

However, you cannot just hand a raw Python function to an LLM. The LLM needs a highly structured, descriptive manual on what the tool does, when to use it, and exactly what arguments it requires. This is where `Pydantic` schemas come in.

## 1. The Anatomy of an Agentic Tool

To successfully bind a tool to an LLM, three components are strictly required:

* **The Execution Logic:** The actual Python code that runs when the tool is called.
* **The Tool Description:** A natural language explanation of the tool's purpose. (This replaces the traditional "prompt" in deciding whether to use the tool).
* **The Argument Schema:** A strict definition of the input parameters, their data types, and what each parameter represents.

## 2. The Simple Approach: Type Hints and Docstrings

Modern frameworks like LangChain can automatically generate schemas for simple functions just by reading your Python docstrings and type hints.

By using a simple `@tool` decorator, the framework parses the metadata and converts it into a JSON Schema for the LLM.

```python
from langchain_core.tools import tool

@tool
def get_weather(location: str, unit: str = "celsius") -> str:
    """
    Fetches the current weather for a given location.
    Always use this tool when the user asks about the weather.
    """
    # ... actual API call logic here ...
    return f"The weather in {location} is 22 degrees {unit}."

# The framework automatically generates a schema requiring a string 'location' 
# and an optional string 'unit'.
```

**The limitation:** While this is fast, the LLM has no description for the individual arguments (`location` and `unit`). It only has the top-level docstring. For complex tools, this leads to LLM hallucinations or poorly formatted inputs.

## 3. The Robust Approach: Pydantic Schemas

When tools become complex, require specific formatting, or involve multiple parameters, relying on simple type hints is not enough. We need Pydantic.

Pydantic is a data validation library for Python. It allows developers to define strict data models using Python classes. When defining tools, we use Pydantic to explicitly define the input arguments.

### Step 3.1: Defining the Input Schema

We create a class inheriting from `BaseModel` and use `Field` to provide natural language descriptions for every single parameter.

```python
from pydantic import BaseModel, Field
from typing import Optional

class SearchFlightsSchema(BaseModel):
    departure_airport: str = Field(
        description="The 3-letter IATA code of the departure airport (e.g., 'JFK', 'LHR')."
    )
    arrival_airport: str = Field(
        description="The 3-letter IATA code of the arrival airport."
    )
    departure_date: str = Field(
        description="The date of departure in 'YYYY-MM-DD' format."
    )
    num_passengers: Optional[int] = Field(
        default=1,
        description="The number of adult passengers. Defaults to 1."
    )
```

**Why is this powerful?** The `Field` descriptions are sent directly to the LLM. If the user asks for a flight from "New York", the LLM reads the schema, realizes it needs a 3-letter IATA code, and internally translates "New York" to "JFK" before it even calls your function.

### Step 3.2: Wrapping the Function with the Schema

Now, we define our actual Python function and explicitly tell the framework to use our Pydantic model to validate the arguments.

```python
from langchain_core.tools import tool

@tool(args_schema=SearchFlightsSchema)
def search_flights(departure_airport: str, arrival_airport: str, departure_date: str, num_passengers: int = 1):
    """
    Searches the airline database for available flights matching the criteria.
    """
    # The variables passed here are guaranteed to match the Pydantic schema
    print(f"Searching flights from {departure_airport} to {arrival_airport} on {departure_date}...")
    
    # ... Database logic ...
    return "Found 3 available flights."
```

## 4. How the LLM Ingests the Schema (Under the Hood)

When you bind this tool to a model (e.g., `llm.bind_tools([search_flights])`), the framework takes your Pydantic class and translates it into a standard OpenAPI JSON Schema.

This JSON payload is sent alongside the system prompt to the LLM API (like OpenAI or Anthropic).

The generated JSON Schema looks something like this:

```json
{
  "name": "search_flights",
  "description": "Searches the airline database for available flights matching the criteria.",
  "parameters": {
    "type": "object",
    "properties": {
      "departure_airport": {
        "type": "string",
        "description": "The 3-letter IATA code of the departure airport (e.g., 'JFK', 'LHR')."
      },
      "arrival_airport": {
        "type": "string",
        "description": "The 3-letter IATA code of the arrival airport."
      },
      "departure_date": {
        "type": "string",
        "description": "The date of departure in 'YYYY-MM-DD' format."
      }
    },
    "required": ["departure_airport", "arrival_airport", "departure_date"]
  }
}
```

## Conclusion

Writing tools requires a shift in mindset: you are writing code not just for the computer to execute, but for the AI to read and understand.

By leveraging Pydantic, you provide the LLM with strict guardrails, detailed argument descriptions, and format constraints. This dramatically reduces agent errors, prevents infinite failure loops, and ensures that when your Python function is finally triggered, it receives exactly the data it expects.
