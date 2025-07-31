**“What is semantic search?”**

---

> **“Semantic search is a method of retrieving information based on meaning rather than exact keyword matches. It uses vector embeddings to understand the intent and context of queries and documents.”**

---

### 🧠 **Key Concepts**

* **Traditional search** = Keyword match (e.g., BM25, TF-IDF)

  * Example: "AI in medicine" only matches those exact words.
* **Semantic search** = Meaning match using embeddings

  * "AI in healthcare" → matched with "machine learning for hospital diagnostics"

---

### ⚙️ **How Semantic Search Works**

1. **Embed documents**

   * Use models like OpenAI, HuggingFace, or Cohere to turn documents into vectors.

2. **Embed the query**

   * The user query is also converted into a vector in the same semantic space.

3. **Vector similarity**

   * Retrieve top documents by computing cosine similarity (or dot product) between query vector and document vectors.

```python
from sentence_transformers import SentenceTransformer, util

model = SentenceTransformer("all-MiniLM-L6-v2")
docs = ["LLMs are used for summarization.", "Tableau is used for BI."]
doc_embeddings = model.encode(docs)

query = "What is business intelligence?"
query_embedding = model.encode(query)

similarities = util.cos_sim(query_embedding, doc_embeddings)
```

---

### ✅ **Advantages**

* **Captures synonyms & paraphrasing**
  (e.g., "capital of France" matches "Paris is the French capital")
* **Improves recall for vague or abstract queries**
* **Better for LLM-powered RAG, QA, and chatbot systems**

---

### 🧰 Common Tools / Libraries

| Library                | Role                         |
| ---------------------- | ---------------------------- |
| FAISS / Qdrant         | Vector DB for fast search    |
| OpenAI / Cohere        | Embedding generation         |
| LlamaIndex / LangChain | Frameworks for RAG pipelines |

---

### 📌 Use in Practice

> “In my RAG systems, I use semantic search to retrieve document chunks relevant to a user's question. This improves grounding and reduces hallucination by providing high-context semantic matches.”

