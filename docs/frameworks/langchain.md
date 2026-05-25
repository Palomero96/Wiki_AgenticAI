# LangChain Fundamentals: Chains, Tools, and Parsers

While the ReAct framework provides the cognitive blueprint for AI agents, LangChain provides the plumbing. It is a framework designed to simplify the creation of applications powered by Large Language Models (LLMs).

To build an agent in LangChain, you need to master three core concepts: constructing Chains, Binding Tools, and parsing the results with Output Parsers.

## 1. Building Chains with LCEL

In LangChain, a "Chain" is a sequence of calls—whether to an LLM, a tool, or a data preprocessing step.

LangChain introduced LCEL (LangChain Expression Language), a declarative way to compose chains together easily. It uses the pipe operator (|), similar to Unix pipelines, to pass the output of one component directly into the next.

### Basic Chain Example

A standard chain takes a user dictionary, formats it through a prompt, sends it to the LLM, and parses the output as a simple string.

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# 1. Create the Prompt Template
prompt = ChatPromptTemplate.from_template("Tell me a short joke about {topic}")

# 2. Initialize the Model
model = ChatOpenAI(model="gpt-4")

# 3. Initialize the Parser
parser = StrOutputParser()

# 4. Build the Chain using LCEL
chain = prompt | model | parser

# 5. Invoke the Chain
result = chain.invoke({"topic": "space travels"})
print(result)
```

## 2. Binding Tools to Models

For an LLM to act as an agent, it needs to know what tools it has access to. Modern LLMs support native Function Calling, which means they are specifically fine-tuned to output JSON structures that match function signatures.

In LangChain, you define Python functions as tools, and then "bind" them to the model.

### Tool Binding Example

```python
from langchain_core.tools import tool

# 1. Define a tool using the @tool decorator
# The docstring is CRITICAL here, as it tells the LLM when to use the tool.
@tool
def multiply(a: int, b: int) -> int:
    """Multiply two integers together."""
    return a * b

@tool
def add(a: int, b: int) -> int:
    """Add two integers together."""
    return a + b

tools = [multiply, add]

# 2. Bind the tools to the model
llm_with_tools = model.bind_tools(tools)

# 3. Test the binding
# When prompted with a math question, the model won't answer directly with text.
# Instead, it will return an "AIMessage" containing a "tool_calls" object.
response = llm_with_tools.invoke("What is 5 multiplied by 12?")

print(response.tool_calls)
# Output: [{'name': 'multiply', 'args': {'a': 5, 'b': 12}, 'id': 'call_abc123'}]
```

## 3. Handling Output Parsers

LLMs output text (or structured message objects). Output Parsers take that raw output and transform it into a more usable format for your application—whether that's a Python dictionary, a specific Pydantic object, or a command to execute a tool.

### Common Types of Parsers:

* **`StrOutputParser`**: Extracts the exact string content from an AI message.
* **`JsonOutputParser`**: Ensures the LLM outputs valid JSON and parses it into a Python dictionary.
* **`Agent Output Parsers`**: Specialized parsers that look at the LLM's response and decide: *"Is this a final answer to the user, or is this a request to call a tool?"*

### JSON Parser Example

If you need the LLM to return structured data instead of conversational text:

```python
from langchain_core.output_parsers import JsonOutputParser
from pydantic import BaseModel, Field

# Define the desired data structure
class Recipe(BaseModel):
    name: str = Field(description="Name of the dish")
    ingredients: list[str] = Field(description="List of ingredients")

# Initialize the parser
json_parser = JsonOutputParser(pydantic_object=Recipe)

# Create a prompt that injects the required formatting instructions
prompt = ChatPromptTemplate.from_template(
    "Generate a recipe for {dish}.\n{format_instructions}"
)

# Build the chain
chain = prompt | model | json_parser

# Invoke (the output will be a clean Python dictionary, not a string)
result = chain.invoke({
    "dish": "pancakes", 
    "format_instructions": json_parser.get_format_instructions()
})

print(result['ingredients']) 
# Output: ['1 cup flour', '2 eggs', '1 cup milk', ...]
```

## Conclusion

By mastering these three fundamentals, you lay the groundwork for complex AI agents:

* **Chains (LCEL):** The infrastructure that connects components.
* **Tool Binding:** Giving the LLM the schemas needed to interact with the real world.
* **Output Parsers:** Translating the LLM's raw thoughts into actionable, structured data.