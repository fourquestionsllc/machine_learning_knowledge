In **AutoGen**, **memory** refers to a mechanism for storing and sharing contextual information between agents during multi-turn conversations. This memory allows agents to maintain state, recall past interactions, and enable coherent and context-aware task completion. Memory sharing is essential in multi-agent setups where agents collaborate and must remember each other's progress or updates.

---

### 🔍 AutoGen Memory Overview

AutoGen provides a `GroupChat` memory by default, but you can also define custom memory classes. Each **agent** can:

* Have **private memory** (only it can read/write).
* Share memory via a **shared message history**.
* Access a **central memory object** (e.g., `GroupChatMemory`).

---

### 🧠 Memory in AutoGen Components

1. **`GroupChatMemory`**: Shared memory where all agent messages are stored.
2. **`UserProxyAgent` / `AssistantAgent`**: Can access the memory to make decisions.
3. **`GroupChat`**: Coordinates the message flow and uses memory to maintain chat history.

---

### ✅ End-to-End Example: Memory Sharing in AutoGen

Let's walk through a **simple AutoGen conversation** between a user and a code assistant, where the assistant remembers previous messages.

#### 🔧 Step 1: Install AutoGen

```bash
pip install pyautogen
```

#### 🧱 Step 2: Define Agents and Memory

```python
import autogen
from autogen import UserProxyAgent, AssistantAgent, GroupChat, GroupChatManager, GroupChatMemory

# Shared memory between agents
shared_memory = GroupChatMemory()

# Define User Agent
user = UserProxyAgent(
    name="user",
    human_input_mode="NEVER",  # No need for manual input
)

# Define Assistant Agent
assistant = AssistantAgent(
    name="assistant",
    llm_config={"config_list": autogen.config_list_from_json("OAI_CONFIG_LIST")},
)

# Define a group chat
group_chat = GroupChat(
    agents=[user, assistant],
    messages=[],
    max_round=5,
    memory=shared_memory  # Share memory here
)

# GroupChatManager to coordinate the chat
manager = GroupChatManager(groupchat=group_chat)
```

#### 💬 Step 3: Start the Chat

```python
user.initiate_chat(
    manager,
    message="Please write a function to reverse a string in Python."
)
```

#### 🧠 Memory Behavior:

* **Round 1**:

  * `user` sends the initial request.
  * `assistant` replies with code.
  * `GroupChatMemory` records both messages.

* **Round 2 (if continued)**:

  * If user sends: *"Now modify it to ignore case"*
  * The assistant can **refer back** to the last message from memory and update the function accordingly.

#### 🧾 Check the Memory

You can inspect the memory contents like this:

```python
for msg in shared_memory.messages:
    print(f"{msg['role']}: {msg['content']}")
```

---

### 🔁 Custom Memory Example

You can also define your own memory logic:

```python
from autogen.memory import MemoryBase

class MyCustomMemory(MemoryBase):
    def __init__(self):
        self.data = []

    def add(self, message):
        self.data.append(message)

    def get(self, role=None):
        if role:
            return [msg for msg in self.data if msg["role"] == role]
        return self.data
```

Then plug it into agents or `GroupChat`.

---

### 🧩 Summary

| Component         | Role                              |
| ----------------- | --------------------------------- |
| `GroupChatMemory` | Shared chat history memory        |
| `GroupChat`       | Shares memory among agents        |
| `Agent`           | Can access and reason over memory |
| `Custom Memory`   | Optional override for control     |

---

