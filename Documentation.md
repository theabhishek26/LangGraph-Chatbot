# LangGraph Chatbot Project Documentation

## Overview

This project implements a conversational chatbot using **LangGraph**, **LangChain**, and **Groq LLM**. The chatbot is enhanced with external tools (e.g., Wikipedia) to provide factual answers. The system is designed with modularity, making it easy to integrate additional tools in the future.

## Key Features

* **LangGraph-based workflow** to define chatbot state transitions.
* **Groq LLM integration** for natural language understanding and response generation.
* **Wikipedia tool integration** for fetching factual data.
* **Streaming response handling** for step-by-step event processing.
* **Colab-compatible implementation** with environment variable management.

## Architecture

The chatbot workflow is represented as a directed graph:

1. **START → Chatbot Node**: User input enters the chatbot node.
2. **Chatbot Node → Tool Node (Conditional)**: If the chatbot requires external information, it calls the tool node.
3. **Tool Node → Chatbot Node**: The tool fetches the required data and returns it to the chatbot.
4. **Chatbot Node → END**: If no tool call is required, or after processing the tool response, the chatbot produces the final answer.

### Architecture Flowchart

![Chatbot Architecture](/mnt/data/A_flowchart_diagram_in_digital_2D_illustrates_a_ch.png)

## Implementation Details

### 1. Environment Setup

```python
!pip install langgraph langsmith
!pip install langchain langchain_groq langchain_community
!pip install wikipedia
```

### 2. API Keys and Environment Variables

```python
from google.colab import userdata
import os

groq_api_key = userdata.get('groq_api_key')
langsmith_api_key = userdata.get('LANGSMITH_API_KEY')

os.environ["LANGSMITH_API_KEY"] = langsmith_api_key
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "LangGraph"
```

### 3. Tools Setup

```python
from langchain_community.utilities import WikipediaAPIWrapper
from langchain_community.tools import WikipediaQueryRun

wikipedia_wrapper = WikipediaAPIWrapper(top_k_results=1, doc_content_chars_max=300)
wikipedia_tool = WikipediaQueryRun(
    api_wrapper=wikipedia_wrapper,
    name="wikipedia",
    description="Useful for looking up information about people, places, or topics on Wikipedia."
)

tools = [wikipedia_tool]
```

### 4. LangGraph State Definition

```python
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from typing import Annotated
from typing_extensions import TypedDict

class State(TypedDict):
    messages: Annotated[list, add_messages]

graph_builder = StateGraph(State)
```

### 5. LLM Integration

```python
from langchain_groq import ChatGroq

llm = ChatGroq(groq_api_key=groq_api_key, model_name="llama-3.1-8b-instant")
llm_with_tools = llm.bind_tools(tools)

def chatbot(state: State):
    return {"messages": [llm_with_tools.invoke(state['messages'])]}
```

### 6. Graph Construction

```python
from langgraph.prebuilt import ToolNode, tools_condition

# Define chatbot node
graph_builder.add_node("chatbot", chatbot)

# Add tool node
tool_node = ToolNode(tools=tools)
graph_builder.add_node("tools", tool_node)

# Define edges
graph_builder.add_edge(START, "chatbot")
graph_builder.add_conditional_edges("chatbot", tools_condition)
graph_builder.add_edge("tools", "chatbot")
graph_builder.add_edge("chatbot", END)

# Compile
graph = graph_builder.compile()
```

### 7. Example Execution

```python
user_input = "What are you doing"

events = graph.stream(
    {"messages": [("user", user_input)]}, stream_mode="values"
)

for event in events:
    if "messages" in event:
        event["messages"][-1].pretty_print()
```

## Output Example

```
User: What are you doing
Assistant: I'm here to assist you with information and tasks. I can also look up facts using tools like Wikipedia.
```

## Conclusion

This chatbot demonstrates how **LangGraph** can be used to design structured conversational flows with tool integration. By combining **Groq LLM** with **LangChain tools**, the chatbot provides both conversational abilities and factual lookup capabilities.

The modular design allows easy integration of additional tools beyond Wikipedia, such as web search, databases, or APIs.
