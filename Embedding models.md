Here are some of the most popular / state-of-the-art text embedding models (open-source and proprietary), along with their embedding dimensions, strengths, and trade-offs. If you want, I can also include their context lengths or memory/compute requirements.

---

## Popular Embedding Models & Their Dimensions

| Model                                                    | Output Dimension(s)                                                                                                                                                                                                                               | Key Notes / Usage                                                                                                                                                                           |
| -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **OpenAI**                                               | 1,536 dim (text-embedding-ada-002) ([TECHCOMMUNITY.MICROSOFT.COM][1]) <br> 3,072 dim (text-embedding-3-large) ([datastax.com][2]) <br> 1,536 dim (text-embedding-3-small) ([Document360][3])                                                      | Good all-purpose embeddings; ada-002 has been a commonly used baseline. The “3-series” gives options that trade off precision vs cost/storage.                                              |
| **Cohere**                                               | 1,024 dim (e.g. Cohere English v3) ([datastax.com][2])                                                                                                                                                                                            | Multilingual/English general embedding tasks; often a good compromise of size vs performance.                                                                                               |
| **Jina Embeddings v3**                                   | 1,024 dim (default) ([arXiv][4])                                                                                                                                                                                                                  | These also support reducing dimension (via Matryoshka / truncation) down to much smaller sizes (e.g. ~ 32 dim) in certain use cases, while maintaining reasonable performance. ([arXiv][4]) |
| **Gecko**                                                | 256 dim and 768 dim variants ([arXiv][5])                                                                                                                                                                                                         | Compact embedding model; the paper shows that even the 256-dim version performs competitively with larger models in many tasks. ([arXiv][5])                                                |
| **Qwen3 Embedding series**                               | Several sizes (0.6B, 4B, 8B model sizes), dimensions vary (not always publicly detail the exact dim for each) ([arXiv][6])                                                                                                                        | Strong multilingual and reranking capabilities; tradeoffs between speed / memory vs performance. ([arXiv][6])                                                                               |
| **Open Source / Sentence Transformers / Smaller Models** | 768 dim is a very common size (MPNet, SBERT, RoBERTa derived, etc.) ([MarkTechPost][7]) <br> Also models with dim ~512 or lower (especially when used for faster retrieval, mobile, or when truncation / compression is used). ([bentoml.com][8]) |                                                                                                                                                                                             |

---

## Some Additional Examples & Variants

* **Word2Vec / GloVe**: older classic word embedding models usually with dimensions like 100, 200, 300, etc. Very lightweight compared to modern contextual / sentence embeddings. ([TECHCOMMUNITY.MICROSOFT.COM][1])
* **ELMo**: produces 512-dim token representations after some combination / projection. ([Wikipedia][9])

---

## Trends & Trade-Offs

* Higher dimensions generally capture more nuance / information, but cost more in storage, compute, and sometimes latency (especially for large vector similarity search).
* Many modern embedding models support *variable dimensions* or *truncation* (e.g. use a subset of the output dimensions) to reduce size while retaining most of the performance. Matryoshka Representation Learning is one approach here. ([arXiv][4])
* There’s a trade-off between embedding quality and speed / memory. For example, compact models (256-768 dims) are often used for large‐scale search, mobile, or resource-constrained deployments; larger dims (e.g. 1,024-3,072) for more precision.

---

[1]: https://techcommunity.microsoft.com/blog/educatordeveloperblog/a-visual-introduction-to-vector-embeddings/4418793?utm_source=chatgpt.com "A visual introduction to vector embeddings | Microsoft Community Hub"
[2]: https://www.datastax.com/jp/blog/best-embedding-models-information-retrieval-2025?utm_source=chatgpt.com "The Best Embedding Models for Information Retrieval in 2025 | DataStax"
[3]: https://document360.com/blog/text-embedding-model-analysis/?utm_source=chatgpt.com "Text Embedding Models Compared: OpenAI, Voyage, Cohere & More"
[4]: https://arxiv.org/abs/2409.10173?utm_source=chatgpt.com "jina-embeddings-v3: Multilingual Embeddings With Task LoRA"
[5]: https://arxiv.org/abs/2403.20327?utm_source=chatgpt.com "Gecko: Versatile Text Embeddings Distilled from Large Language Models"
[6]: https://arxiv.org/abs/2506.05176?utm_source=chatgpt.com "Qwen3 Embedding: Advancing Text Embedding and Reranking Through Foundation Models"
[7]: https://www.marktechpost.com/2024/11/16/a-comparison-of-top-embedding-libraries-for-generative-ai/?utm_source=chatgpt.com "A Comparison of Top Embedding Libraries for Generative AI - MarkTechPost"
[8]: https://www.bentoml.com/blog/a-guide-to-open-source-embedding-models?utm_source=chatgpt.com "A Guide to Open-Source Embedding Models"
[9]: https://en.wikipedia.org/wiki/ELMo?utm_source=chatgpt.com "ELMo"


------------------


Here's a concise yet comprehensive comparison of **widely used embedding models** from major providers (OpenAI, AWS, Google, Azure, and others like Cohere, Mistral, and Hugging Face). I’ll cover:

1. **Popular embedding models**
2. **How they work**
3. **How to use them**
4. **Embedding dimensions**
5. **Normalization**
6. **Similarity metrics and ranges**

---

## 🔥 1. Popular Embedding Models (2024–2025)

| Provider        | Model Name                                      | Dim      | Normalized? | Notes                      |
| --------------- | ----------------------------------------------- | -------- | ----------- | -------------------------- |
| **OpenAI**      | `text-embedding-3-small`                        | 1536     | ✅ Yes       | Fast, cheap, new standard  |
|                 | `text-embedding-3-large`                        | 3072     | ✅ Yes       | Higher quality             |
| **AWS**         | Amazon Titan Text Embeddings G1                 | 1536     | ❌ No        | Powerful multilingual      |
| **Google**      | Vertex AI `textembedding-gecko`                 | 768      | ✅ Yes       | Good general-purpose       |
| **Azure**       | Mirrors OpenAI models                           | Same     | Same        | Uses OpenAI under hood     |
| **Cohere**      | `embed-english-v3.0`, `embed-multilingual-v3.0` | 1024     | ✅ Yes       | High quality, multilingual |
| **HuggingFace** | `BAAI/bge-large-en`, `all-MiniLM-L6-v2`, etc.   | 384–1024 | ❌ (varies)  | Open source, flexible      |

---

## ⚙️ 2. How Embeddings Work (Briefly)

Embeddings are fixed-size vector representations of text (words, sentences, or documents), where **semantic similarity** corresponds to **geometric closeness** in vector space. They're generated by transformer models trained with contrastive loss or other objectives like MLM + alignment.

---

## 🛠️ 3. How to Use Them

### OpenAI Example:

```python
import openai

response = openai.embeddings.create(
    model="text-embedding-3-small",
    input="The quick brown fox jumps over the lazy dog"
)
embedding = response['data'][0]['embedding']
```

### Hugging Face Example:

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('BAAI/bge-large-en')
embedding = model.encode("The quick brown fox...", normalize_embeddings=True)
```

### Google Vertex Example:

```python
from vertexai.language_models import TextEmbeddingModel

model = TextEmbeddingModel.from_pretrained("textembedding-gecko@001")
embedding = model.get_embeddings(["text here"]).embeddings[0].values
```

### AWS Bedrock (Pseudocode):

```python
client = boto3.client('bedrock-runtime')
response = client.invoke_model(body={"inputText": "..."}, modelId="amazon.titan-embed-text-v1")
embedding = response['embedding']
```

---

## 📏 4. Dimensions

| Model                    | Dimensionality |
| ------------------------ | -------------- |
| `text-embedding-3-small` | 1536           |
| `text-embedding-3-large` | 3072           |
| Amazon Titan             | 1536           |
| Cohere v3                | 1024           |
| Gecko                    | 768            |
| BAAI BGE / MiniLM        | 384–1024       |

---

## ⚖️ 5. Are They Normalized?

| Model                        | Normalized Output   |
| ---------------------------- | ------------------- |
| OpenAI (`text-embedding-3*`) | ✅ Yes (L2 norm = 1) |
| Cohere v3                    | ✅ Yes               |
| Google Gecko                 | ✅ Yes               |
| AWS Titan                    | ❌ No                |
| Hugging Face                 | ❌ (often optional)  |

👉 If not normalized, normalize manually using L2:

```python
from numpy import linalg as LA
embedding = embedding / LA.norm(embedding)
```

---

## 🔍 6. Similarity Metric

**Use Cosine Similarity** for most embedding models, especially if vectors are normalized:

```python
from numpy import dot
from numpy.linalg import norm

def cosine_similarity(a, b):
    return dot(a, b) / (norm(a) * norm(b))
```

### 🔄 Normalized vectors:

* Cosine similarity becomes a **dot product**
* Range: **\[-1, 1]**
* Most models output **positive scores** (0 to 1) if inputs are similar types

### ✳️ Alternative: Euclidean Distance

* Less common for text embeddings
* Use if the model doesn't normalize and you want absolute geometric distance

---

## 🧠 Tips for Usage in RAG / Search / Clustering

* Use **cosine similarity** over **normalized embeddings** for:

  * Semantic search
  * RAG
  * Duplicate detection
* Consider **dimensionality reduction** (e.g., PCA) for visualization
* Use **FAISS** or **PGVector** for scalable similarity search

---

## 🔚 TL;DR

| Provider     | Best Model               | Dim  | Norm? | Compare With         | Score Range |
| ------------ | ------------------------ | ---- | ----- | -------------------- | ----------- |
| OpenAI       | `text-embedding-3-small` | 1536 | ✅     | Cosine               | \~0–1       |
| AWS          | Titan Embeddings         | 1536 | ❌     | Cosine or Euclidean  | \~-1–1      |
| Google       | Gecko                    | 768  | ✅     | Cosine               | 0–1         |
| Cohere       | `embed-english-v3.0`     | 1024 | ✅     | Cosine               | 0–1         |
| Hugging Face | `BAAI/bge-*`             | 768+ | ❌     | Cosine (manual norm) | \~-1–1      |

