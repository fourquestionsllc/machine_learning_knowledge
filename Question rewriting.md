**question rewriting** or **query rewriting** — turning a follow-up or context-dependent question into a **standalone query** that captures all the necessary information from chat history.

---

### ✅ **Why This is Important**
In chatbot development, especially with RAG (Retrieval-Augmented Generation), search works best when you give it a **fully self-contained** query. So instead of:

> “What about next week?”

You want to rewrite it as:

> “What is the weather forecast in Paris next week?”

---

## 🔧 Methods to Convert Contextual Questions into Standalone Search Strings

### 1. **Prompt-based Rewriting (LLM) — Most Common**
You give the **LLM** the chat history and the latest question, and prompt it like:

```plaintext
Given the following chat history and question, rewrite the question as a standalone query.

Chat history:
User: Tell me about Tesla stock performance.
Bot: Tesla has been volatile, but had a strong Q4.
User: And what about Apple?

Rewritten query: Tell me about Apple stock performance in Q4.
```

- **Pros**: Easy, flexible, LLMs do a great job.
- **Cons**: Slight latency, context window limits if history is long.

---

### 2. **Heuristic + Rules-based Rewriting**
You build custom logic to detect pronouns, time references, or ellipsis:

```python
if "he" or "she" in question:
   replace with last mentioned name from history
if "next week":
   convert to actual date range
```

- **Pros**: Fast, predictable
- **Cons**: Brittle, not scalable for open domain

---

### 3. **Sequence-to-Sequence Model (Fine-Tuned)**
You fine-tune a small transformer (like T5 or FLAN-T5) on a dataset of:
- Chat history + contextual question → Standalone question

- **Pros**: Lower latency than GPT, can run locally
- **Cons**: Needs training data, may not generalize

---

### 4. **Retrieval-Augmented Rewriting**
Before rewriting, retrieve the relevant context from long history using embeddings + vector search, then use the top-k history turns with the LLM.

- **Example**: Use Milvus to get the most similar past utterances.

- **Pros**: Handles long context well
- **Cons**: Adds vector infra, more moving parts

---

### 5. **Agent-based Decomposition (Optional)**
Use LangChain or similar to decompose the user intent into substeps, rewrite it, and send to search.

- **Pros**: Very smart for complex multi-hop questions
- **Cons**: Slower, complex to orchestrate

---

## 🧠 Recommendations

If you're using LLMs already, here’s a simple yet powerful approach:

1. **Prompt-based Rewriting** with history truncation (last 3–5 turns).
2. Optionally enhance with **embedding-based context selector**.
3. Use OpenAI's `text-davinci-003`, GPT-4, or Mistral with a prompt like:

```plaintext
Rewrite the following question using the chat history so that it is self-contained.

Chat History:
User: Who is the CEO of Tesla?
Bot: Elon Musk is the CEO of Tesla.
User: What is his net worth?

Standalone Question:
What is Elon Musk’s net worth?
```

---
