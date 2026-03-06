When upgrading an **agent built with one OpenAI model (like o4)** to a **newer model (like GPT-5)**, usually you **do NOT rewrite everything**, but you should adjust a few things because newer models often behave differently.

Below are the **main areas to update** when switching models.

---

# 1. Change the Model Name in API Calls

The simplest change is the model parameter.

Example before:

```python
from openai import OpenAI
client = OpenAI()

response = client.responses.create(
    model="o4-mini",
    input="Summarize this document"
)
```

After upgrade:

```python
response = client.responses.create(
    model="gpt-5",
    input="Summarize this document"
)
```

Most of your code can remain the same.

---

# 2. Simplify Prompts (New Models Need Less Prompting)

Older models often required **very detailed instructions**.

Example old prompt:

```text
You are a helpful assistant. Follow the instructions carefully.
Step 1: Analyze the question.
Step 2: Break down reasoning.
Step 3: Produce final answer.
```

Newer models usually perform better with **simpler prompts**.

Better prompt:

```text
Answer the user's question clearly and concisely.
```

Over-engineering prompts can **hurt performance** on newer models.

---

# 3. Remove Chain-of-Thought Instructions

Older prompts often used:

```
Let's think step by step
```

Newer reasoning models **already reason internally**, so explicit chain-of-thought prompting may be unnecessary.

Instead:

```
Provide the final answer and short explanation.
```

---

# 4. Update Tool / Function Calling Prompts

If your agent uses tools, older prompts often looked like this:

```
If a tool is needed, return JSON with the function name and arguments.
```

Newer models handle tool usage more naturally.

You should rely on **structured tool definitions** rather than prompt instructions.

Example tool:

```python
tools = [{
  "type": "function",
  "function": {
    "name": "search_documents",
    "description": "Search internal documents",
    "parameters": {
        "type": "object",
        "properties": {
            "query": {"type": "string"}
        }
    }
  }
}]
```

Then let the model decide when to call it.

---

# 5. Adjust Temperature and Reasoning Settings

New models often require **different sampling settings**.

Typical values:

| Setting     | Older models | Newer models  |
| ----------- | ------------ | ------------- |
| temperature | 0.7          | 0.2–0.5       |
| max tokens  | larger       | often smaller |

Example:

```python
response = client.responses.create(
    model="gpt-5",
    input="Analyze this dataset",
    temperature=0.3
)
```

---

# 6. Review Context Window Usage

New models often have **larger context windows**.

You may simplify memory management.

Before:

```
summarize conversation every 10 turns
```

Now you may keep **full conversation history**.

---

# 7. Update Safety / Guardrails Prompts

Older prompts sometimes forced behavior:

```
Never reveal confidential information.
```

New models often have **built-in safety policies**, so prompts can be lighter.

Still keep **system messages** for policy rules.

Example:

```python
messages=[
 {"role":"system","content":"You are an enterprise AI assistant. Follow company policies."},
 {"role":"user","content":question}
]
```

---

# 8. Re-evaluate RAG Prompt Templates

If you use RAG, your prompt might be:

```
Use the context below to answer the question.
If the answer is not in the context, say you don't know.
```

New models often **hallucinate less**, so prompts can be simpler.

Example:

```
Answer using the provided context.
```

---

# 9. Run Prompt Regression Tests

When upgrading models, test:

* accuracy
* hallucination rate
* tool usage
* formatting

Create a **prompt test suite**.

Example test questions:

```
test_cases = [
 "financial report summary",
 "database query question",
 "tool invocation scenario"
]
```

Compare old vs new outputs.

---

# 10. Monitor Cost and Latency

Newer models may change:

* pricing
* speed
* token usage

Always monitor:

* response latency
* token consumption
* cost per request

---

# Recommended Upgrade Workflow

```text
Step 1  change model name
Step 2  run existing prompts
Step 3  simplify prompts
Step 4  validate tool calling
Step 5  run evaluation tests
Step 6  tune parameters
```

---

✅ **Key takeaway**

Switching models usually requires:

* changing the **model name**
* **simplifying prompts**
* validating **tool calling**
* adjusting **parameters**
