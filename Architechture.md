# 🧠 LangGraph Chatbot with Groq LLM + Wikipedia Tool

This project is a **LangGraph-based conversational chatbot** that uses **Groq LLaMA models** as the reasoning engine and integrates a **Wikipedia tool** for retrieving factual information.  

---

## 🚀 Features
- Uses **LangGraph** to manage conversational flow.
- Integrates **Groq LLaMA 3.1 8B-Instant** as the LLM.
- Includes a **Wikipedia API wrapper** as an external knowledge tool.
- Maintains **conversation state** across messages.
- Provides **modular and extensible architecture** (easily add more tools).

---

## 🏗️ Solution Architecture

```text
+--------------------------------------------------------+
|                        User                            |
|   (Sends query: "Who is Virat Kohli?")                 |
+-------------------------+------------------------------+
                          |
                          v
+--------------------------------------------------------+
|                   LangGraph Orchestrator               |
|                                                        |
|  StateGraph (Conversation Flow Manager)                |
|   - Maintains conversation state                       |
|   - Routes messages between nodes                      |
+--------------------------------------------------------+
        |                          |
        v                          v
+----------------------+     +---------------------------+
|   Chatbot Node       |     |   Tool Node (Wikipedia)   |
| - Runs LLM (Groq)    |     | - Executes Wikipedia API  |
| - Decides if tools   |     | - Returns short summary   |
|   are needed         |     |   back to Chatbot         |
+----------------------+     +---------------------------+
        |                          ^
        +--------------------------+
        |
        v
+--------------------------------------------------------+
|                    Groq LLM (LLaMA)                   |
| - Processes messages                                   |
| - Generates responses                                  |
| - Uses tools if needed (Wikipedia binding)             |
+--------------------------------------------------------+
                          |
                          v
+--------------------------------------------------------+
|                        Output                          |
|     AI Final Answer (e.g., "Virat Kohli is an          |
|     Indian cricketer and former captain of the         |
|     Indian national team...")                          |
+--------------------------------------------------------+


⚙️ Explanation of Components
1. User Layer

Entry point — user sends input text.

2. LangGraph Orchestrator (StateGraph)

Manages flow:

Keeps track of conversation history (messages).

Routes input between nodes (chatbot, tools, end).

3. Chatbot Node

Uses Groq LLM (llama-3.1-8b-instant).

Checks if query needs external info.

If yes → forwards to Tool Node.

If no → generates direct response.

4. Tool Node (Wikipedia)

Executes Wikipedia API call.

Gets top result (limited to 300 chars).

Sends result back to Chatbot node.

5. Groq LLM

Core reasoning engine.

Integrates tool outputs into natural language responses.

6. Output Layer

Returns final AI message to user.

🚀 End-to-End Flow

User → asks question.

LangGraph → routes to Chatbot node.

Chatbot node → runs LLM, decides if Wikipedia is needed.

If needed → Tool Node fetches info.

Tool result → returned to Chatbot node.

LLM integrates everything → generates final answer.

Answer → returned to user.