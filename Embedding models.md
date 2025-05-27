Here’s a breakdown of the **most popular embedding models** from providers like **OpenAI**, **Cohere**, **Stella (by Stability AI)**, and others as of 2025, with details on how to use them, how they work, and their dimensions.

---

### 🔵 **1. OpenAI Embedding Models**

OpenAI offers several embedding models via API. The most widely used ones include:

#### 🔹 `text-embedding-3-small`

* **Release:** 2024
* **Dimensions:** 1536
* **Pricing:** Lower than previous models
* **Use Case:** General-purpose embeddings for semantic search, RAG, classification.
* **How to Use (Python):**

  ```python
  from openai import OpenAI

  client = OpenAI()

  response = client.embeddings.create(
      input="Hello, world!",
      model="text-embedding-3-small"
  )
  embedding = response.data[0].embedding
  ```

#### 🔹 `text-embedding-3-large`

* **Release:** 2024
* **Dimensions:** 3072
* **Use Case:** High-precision semantic search or classification.
* **Note:** Supports **dimensionality reduction** (you can request 256, 512, etc.).

#### ⚙️ How It Works

* Trained on a mix of web, books, and code.
* Transformer-based architecture.
* Outputs a dense vector (high-dimensional) representing the input’s meaning.

---

### 🟣 **2. Cohere Embedding Models**

Cohere has released several strong models, particularly optimized for long-context or retrieval use.

#### 🔹 `embed-v3.0`

* **Release:** 2024
* **Dimensions:** 1024
* **Multilingual:** Yes
* **Use Case:** RAG, search, classification.
* **How to Use (Python):**

  ```python
  import cohere
  co = cohere.Client('YOUR_API_KEY')

  response = co.embed(texts=["Hello, world!"], model="embed-v3.0")
  embedding = response.embeddings[0]
  ```

#### ⚙️ How It Works

* Transformer model trained contrastively to bring similar texts closer.
* Optimized for cross-lingual and long-context embeddings.

---

### 🟡 **3. Mistral / Stella AI (Stability AI)**

Stella from Stability AI released open-source models, and **BGE (BAAI)** is often used in their stacks.

#### 🔹 `bge-base-en-v1.5` or `bge-large-en-v1.5` (from BAAI)

* **Dimensions:**

  * `bge-base`: 768
  * `bge-large`: 1024
* **Use Case:** Open-source alternative for RAG and semantic search.
* **How to Use (with Hugging Face Transformers):**

  ```python
  from sentence_transformers import SentenceTransformer

  model = SentenceTransformer('BAAI/bge-large-en-v1.5')
  embedding = model.encode("Hello, world!")
  ```

#### ⚙️ How It Works

* Trained with contrastive loss + instruction tuning.
* Outputs dense embeddings that work well in vector search.

---

### 🔶 **4. Google - Gecko & Universal Sentence Encoder**

Still in use but now often outperformed by OpenAI and Cohere’s latest models.

* **USE Dimensions:** 512
* **Gecko Dimensions:** 768–1024

---

### 📌 Summary Table

| Provider          | Model                  | Dimensions | Type  | API/Open-Source | Best For                         |
| ----------------- | ---------------------- | ---------- | ----- | --------------- | -------------------------------- |
| **OpenAI**        | text-embedding-3-small | 1536       | Dense | API             | General semantic tasks           |
| **OpenAI**        | text-embedding-3-large | 3072 max   | Dense | API             | High-precision RAG               |
| **Cohere**        | embed-v3.0             | 1024       | Dense | API             | Multilingual, high-accuracy RAG  |
| **Stella / BAAI** | bge-large-en-v1.5      | 1024       | Dense | Open-source     | Open-source RAG, local inference |
| **Google**        | USE / Gecko            | 512–768    | Dense | API             | Lightweight semantic tasks       |

---

### 🧠 How Embedding Models Work

* **Architecture:** Based on Transformers (like BERT or GPT).
* **Training:** Contrastive learning — similar pairs are pulled together, dissimilar pushed apart.
* **Input:** Text or document.
* **Output:** A dense vector that captures the semantic meaning.
* **Use:** Vector stores (e.g., FAISS, Weaviate, Pinecone), cosine or dot-product search, RAG pipelines.

---

If you're working on a RAG pipeline (like in your game narrative system), I’d recommend:

* **`text-embedding-3-small`** (fast, high quality) or
* **`bge-large-en-v1.5`** (if you want open-source & local inference)

