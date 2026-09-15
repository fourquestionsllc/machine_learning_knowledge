If you mean **Claude**, **models**, and **programming instructions**, the roles are different:

### 1. Claude = the AI model family

Anthropic develops Claude.

Claude is a family of LLMs, such as:

```text
Claude
 ├── Claude Opus
 ├── Claude Sonnet
 └── Claude Haiku
```

The **model** is what actually understands your prompt and generates code/text.

---

### 2. Prompt = telling the model what to do

For example:

```text
You are a senior Python engineer.

Write a Python function that reads a CSV
and removes duplicate records.
```

This is your **instruction/prompt**.

The model takes:

```text
Instruction
    +
Your data/context
    ↓
Claude model
    ↓
Generated code
```

---

### 3. System prompt = defining the role

If you tell Claude:

```text
You are a senior software engineer.
Always write production-quality Python.
Include error handling and unit tests.
```

you're defining the **role/behavior** of the model.

Conceptually:

```text
System instruction
      ↓
"You are a Python expert"
      ↓
User instruction
      ↓
"Build an API"
      ↓
Claude
      ↓
Python code
```

### 4. What does "role" mean in API calls?

When using an LLM API, messages commonly have roles such as:

```python
messages = [
    {
        "role": "system",
        "content": "You are a senior Python developer."
    },
    {
        "role": "user",
        "content": "Write a FastAPI application."
    }
]
```

The important roles are:

| Role          | Meaning                             |
| ------------- | ----------------------------------- |
| **system**    | Defines model behavior/instructions |
| **user**      | User's request                      |
| **assistant** | Model's previous response           |
| **tool**      | Result returned by an external tool |

So if you're asking **"what is the role that tells the model how to program?"**, the answer is usually **system instruction/system prompt**.

One important modern distinction: with Claude's newer APIs, the exact message structure differs somewhat from OpenAI's API, but the concept of **instructions + user request + model response + tools** remains the same.
