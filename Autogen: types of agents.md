AutoGen provides several types of agents designed for different use cases. The main types of agents include:

### 1. **UserProxyAgent**
   - Acts as a proxy for human users to interact with the system.
   - Useful for providing input and making decisions in an automated process.

### 2. **AssistantAgent**
   - Represents an AI assistant that can execute tasks, generate responses, and interact with other agents.
   - Commonly used to process and analyze user queries.

### 3. **GroupChatManagerAgent**
   - Manages a group of agents to facilitate multi-agent collaboration.
   - Can coordinate interactions between multiple agents in a conversation.

### 4. **CodeExecutorAgent**
   - Specialized in executing code snippets and returning results.
   - Useful for automating coding tasks.

### 5. **CSVAgent**
   - Designed to handle CSV files, including reading, processing, and analyzing data.

---

## **Example Usage of AutoGen Agents**
Below is an example of how to use `UserProxyAgent` and `AssistantAgent` to create a simple conversation:

```python
import autogen

# Create a UserProxyAgent (acts as a human proxy)
user_proxy = autogen.UserProxyAgent(name="User", human_input_mode="ALWAYS")

# Create an AssistantAgent (acts as an AI assistant)
assistant = autogen.AssistantAgent(name="Assistant")

# Start an interaction
user_proxy.initiate_chat(assistant, message="What is the capital of France?")
```

### **Example: Code Execution with CodeExecutorAgent**
This example demonstrates how to use `CodeExecutorAgent` to execute Python code:

```python
import autogen

# Create a UserProxyAgent
user_proxy = autogen.UserProxyAgent(name="User", human_input_mode="ALWAYS")

# Create a CodeExecutorAgent
code_executor = autogen.CodeExecutorAgent(name="CodeRunner")

# Start a conversation where the agent executes code
user_proxy.initiate_chat(code_executor, message="Execute: print(2 + 2)")
```

### **Example: GroupChatManagerAgent for Multi-Agent Collaboration**
This example shows how to manage multiple agents in a conversation:

```python
import autogen

# Define agents
user_proxy = autogen.UserProxyAgent(name="User", human_input_mode="ALWAYS")
assistant1 = autogen.AssistantAgent(name="MathAssistant")
assistant2 = autogen.AssistantAgent(name="ScienceAssistant")

# Create a chat manager
manager = autogen.GroupChatManagerAgent(name="Manager", agents=[user_proxy, assistant1, assistant2])

# Start a group chat
user_proxy.initiate_chat(manager, message="Solve 2x + 3 = 7 and explain the science behind it.")
```
