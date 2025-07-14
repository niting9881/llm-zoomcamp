# From RAG to Agents: Building Smart AI Assistants
## Study Notes and FAQ

This document serves as a comprehensive study guide and FAQ for understanding Retrieval Augmented Generation (RAG) and its evolution into more dynamic, agent-based AI systems, drawing insights from the "GitHub - alexeygrigorev/rag-agents-workshop" and "Unleashing AI Agents: Beyond Simple RAG" sources.

---

## 📚 Study Guide: Key Concepts

### Part 1: Basic Retrieval Augmented Generation (RAG)

**What is RAG?**
RAG is a technique that combines retrieval of information with generation by a Large Language Model (LLM). It is particularly effective when you have a specific knowledge base and want the LLM to answer questions only using that context.

**The Three Core Components of Basic RAG:**

1. **🔍 Search**: This component queries a knowledge base (like an FAQ database) to find relevant documents based on a user's query. In the workshop, `minsearch` is used as a simple in-memory search engine. The search function takes a query and returns relevant documents, often with unique IDs for later deduplication.

2. **📝 Prompt**: Search results are formatted into a structured context that the LLM can use. This involves embedding the user's question and the retrieved context into a `prompt_template`. Modern prompts often use XML-like tags (e.g., `<QUESTION>`, `<CONTEXT>`) to clearly delineate sections for the model.

3. **🤖 LLM**: The formatted prompt is sent to a Language Model (like OpenAI's `gpt-4o-mini`) which then generates an answer based on the provided context.

**⚠️ Limitations of Basic RAG:**
While good for specific knowledge bases, a basic RAG system cannot answer questions that are not contained in its database or use its own general knowledge. It's constrained to the provided context, which can be a drawback if the goal is a more versatile assistant.

---

### Part 2: Introduction to AI Agents

**What are AI Agents?**
Agents are AI systems designed to:
- Make decisions about what actions to take
- Use tools to accomplish tasks
- Maintain state and context, remembering previous interactions and actions
- Learn from previous interactions
- Work towards specific goals

**🔄 Typical Agentic Flow:**
1. Receiving a user request
2. Analyzing the request and available tools
3. Deciding on the next action
4. Executing the action using appropriate tools
5. Evaluating the results
6. Either completing the task or continuing with more actions

**🎯 Key Differences from Basic RAG:**
The key difference from basic RAG is that agents can:
- Make multiple search queries
- Combine information from different sources
- Decide when to stop searching
- Use their own knowledge when appropriate
- Chain multiple actions together
- Access the history of previous actions and make decisions independently

---

### Part 3: Agentic RAG (Decision-Making)

**Enhanced Decision-Making Capabilities:**
Agentic RAG enhances basic RAG by allowing the AI assistant to decide whether to answer a question directly using its own knowledge or to perform a search in the FAQ database.

**Implementation through Modified Prompts:**
This is achieved by modifying the prompt to include instructions and output templates for different actions:

- **🔍 SEARCH**: If the context is empty and the LLM decides it needs more information from the FAQ database
- **📖 ANSWER (source: CONTEXT)**: If the LLM can answer the question using the provided context from a search
- **🧠 ANSWER (source: OWN_KNOWLEDGE)**: If the context does not contain the answer, or if the question can be answered without needing a search, the LLM uses its internal knowledge

**Processing Flow:**
The system then parses the LLM's structured output (e.g., JSON) to determine the next action. If the action is SEARCH, it performs the search, builds the context, and queries the LLM again with the new context.

---

### Part 4: Agentic Search (Deep Exploration)

**Advanced Multi-Query Search:**
Agentic search takes the concept further by allowing the agent to formulate multiple search queries and perform several iterations of search to deeply explore a topic.

**Complex Prompt Structure:**
The prompt for agentic search becomes more complex, including:

- **🛠️ Available Actions**: Explicitly listing actions like "Search in FAQ," "Answer using own knowledge," "Answer using information extracted from FAQ"
- **🧠 Previous Actions/Memory**: Providing access to a history of previously performed actions and search queries. This prevents repetition and helps the model make better decisions
- **🔍 Search Query Formulation**: Instructions to analyze the context and question to generate relevant and deep search queries, avoiding previously used ones
- **⏹️ Stop Criteria**: Defining a maximum number of iterations (e.g., 3 or 4) to prevent indefinite looping. If the limit is reached, the agent is instructed to give the best possible answer with available information
- **📊 Structured Output**: Specifying JSON templates for SEARCH (including keywords), ANSWER_CONTEXT, and ANSWER (source: OWN_KNOWLEDGE)

**🔄 Handling Search Results:**
After performing multiple searches based on the generated keywords, results are deduplicated to avoid redundant information being passed to the LLM, which also saves on token costs.

This iterative process allows for deep research where the agent can refine its understanding and queries based on previous search results.

---

### Part 5: Function Calling (Tool Use) in OpenAI

**What is Function Calling?**
Function calling (also known as "tool use") is a convenient API feature provided by OpenAI (and other providers) that allows models to natively decide when to call developer-defined functions and respond with structured data.

**✅ Benefits:**
- **Simplifies Agent Development**: Eliminates the need for developers to manually craft complex prompts with if-else logic and parse LLM-generated JSON to determine actions
- **Structured Output**: The model returns structured `function_call` objects, making it easy to extract function names and arguments
- **Reusability**: Functions can be described once and provided to the model, which decides when and how to use them

**🔧 Describing Tools:**
Functions are described using a specific JSON schema, which includes:
- `type`: "function"
- `name`: The name of the function (e.g., "search", "add_entry")
- `description`: A text description of when to use the function
- `parameters`: An object defining the function's arguments, including their types and descriptions

**🔄 Interacting with the API:**
- Instead of `chat.completions.create`, the `client.responses.create` API is used
- Messages can be split into a developer (system) prompt and user content
- The `tools` argument is passed with a list of available tool descriptions
- If the model decides to make a function call, its `response.output` will contain a `ResponseFunctionToolCall` object
- A `do_call` helper function can be created to execute the identified function with its arguments
- The results of the function call are then appended back to the `chat_messages` as `function_call_output` to maintain history for subsequent LLM calls

**🔀 Making Multiple Calls:**
The developer prompt can be tweaked to encourage the model to make multiple function calls (e.g., multiple search queries). The system then iterates through `response.output`, executing each `function_call` and appending its result back to the `chat_messages`.

---

### Part 6: Refactoring and Frameworks

**🏗️ Code Organization:**
To make the agent code more manageable and modular, it can be refactored into classes:

- **🛠️ Tools**: Manages available function tools, allowing registration and execution of calls
- **💬 ChatInterface**: Handles user input and display formatting (e.g., using HTML for clearer output)
- **🤖 ChatAssistant**: The main orchestrator that combines tools, prompts, and interface to manage the chat conversation, including the request-response loops

**🚀 PydanticAI Framework:**
PydanticAI is an example of a framework that simplifies agent creation by automatically generating tool descriptions from function docstrings. This reduces boilerplate code and makes tools easier to define.

---

## ❓ FAQ: Frequently Asked Questions

### Basic RAG Concepts

**Q: What is Retrieval Augmented Generation (RAG)?**
A: RAG is a system that enhances an LLM's ability to answer questions by first searching a knowledge base for relevant information and then providing that information as context to the LLM for generating a response.

**Q: What are the main limitations of a basic RAG system?**
A: A basic RAG system is limited to answering questions only from its specific knowledge base. It cannot use its broader internal knowledge to answer generic questions or questions not covered by the retrieved context.

### AI Agents

**Q: How do AI agents enhance RAG?**
A: AI agents enhance RAG by allowing the system to make decisions about its actions. This includes deciding whether to use its internal knowledge or perform a search, formulating multiple search queries, maintaining conversation history, and chaining actions together to achieve complex goals.

**Q: What is "Agentic Search"?**
A: Agentic search is an advanced form of RAG where the AI agent can perform multiple, iterative search queries to deeply explore a topic. It formulates search terms, analyzes prior results, and decides whether to continue searching or provide an answer based on collected information, with a predefined limit on iterations.

### Function Calling

**Q: How does OpenAI's "Function Calling" simplify agent development?**
A: Function calling allows the LLM to decide when and how to use pre-defined functions (tools). Instead of manual prompt crafting and parsing for action determination, developers describe functions, and the OpenAI SDK handles the internal prompt generation, providing structured outputs for function calls.

**Q: Can new functionalities be added to an agent using function calling?**
A: Yes, adding new functionalities (tools) is straightforward with function calling. You define the new function and its description (name, parameters, purpose), and the agent can then decide to use it when appropriate, such as an `add_entry` function to update an FAQ database.

### Technical Implementation

**Q: Is the search function used in the workshop based on a vector database?**
A: In the workshop, the primary search function uses `minsearch`, which is a simple in-memory keyword search engine. However, the sources note that search can be implemented with various knowledge bases, including text search in Elastic, vector search from Qdrant, or hybrid search.

**Q: Are the additions made via the add_entry tool persistent?**
A: In the workshop's example, the `add_entry` tool adds records to an in-memory minsearch index. These additions are not persistent; they will be lost when the kernel or application restarts. For persistence, a proper database like Qdrant would be needed.

### Frameworks

**Q: What is the benefit of PydanticAI for agent development?**
A: PydanticAI is a framework that helps in creating agents by automating the generation of tool descriptions. It can read function docstrings and automatically create the necessary function definitions for the LLM to use, reducing manual effort and potential errors in tool descriptions.

---

## 📚 Resources

- **📹 Video**: [From RAG to Agents: Building Smart AI Assistants](https://www.youtube.com/watch?v=GH3lrOsU3AU)
- **💻 Workshop Repository**: [alexeygrigorev/rag-agents-workshop](https://github.com/alexeygrigorev/rag-agents-workshop)

---

*This study guide provides a comprehensive overview of the evolution from basic RAG systems to sophisticated AI agents, covering key concepts, implementation details, and practical considerations for building smart AI assistants.*
