## 📌 **Embedding Models Comparison Table (Top 6 per Provider)**

| **Provider**         | **Model**                           | **Embedding Dimensions** | **Max Input / Context** | **Typical Pricing**        | **Notes / Important Parameters**                                           |
| -------------------- | ----------------------------------- | ------------------------ | ----------------------- | -------------------------- | -------------------------------------------------------------------------- |
| **OpenAI**           | text-embedding-3-small              | ~1536                    | 8192 tokens             | ~$20 / 1M tokens           | Efficient low-cost embed, good quality ([Cloud Price][2])                  |
|                      | text-embedding-3-large              | ~3072                    | 8192 tokens             | ~$130 / 1M tokens          | Best overall quality embedding ([Cloud Price][3])                          |
|                      | text-embedding-ada-002 (legacy)     | ~1536                    | 8192 tokens             | ~$100 / 1M tokens          | Still commonly used legacy model ([Cloud Price][4])                        |
|                      | **—** (smaller / internal variants) | ~768 / ~1536             | 8192 tokens             | —                          | Many frameworks provide mini versions for cost/perf ([docs.upsonic.ai][5]) |
|                      | **—** (fine-tuned variations)       | varies                   | 8192 tokens             | —                          | Provider-tuned for domains (search, code, etc.) ([docs.upsonic.ai][5])     |
|                      | **—** (experimental)                | varies                   | 8192 tokens             | —                          | Used by many third-party tools ([Reddit][6])                               |
| **Azure AI Foundry** | text-embedding-3-small              | ~1536                    | 8192 tokens             | ~same as OpenAI            | Same specs as OpenAI version via Foundry ([Microsoft Learn][7])            |
|                      | text-embedding-3-large              | ~3072                    | 8192 tokens             | ~same as OpenAI            | Same high-quality embed ([Azure AI][8])                                    |
|                      | **Voyage 3.5 Embedding**            | 256/512/1024/2048        | 32,000 tokens           | competitive but not public | Flexible dims + multilingual support ([Azure AI][9])                       |
|                      | **Voyage 3.5-lite**                 | 256/512/1024/2048        | 32,000 tokens           | lower than full version    | Cost-optimized variant ([Voyage AI][10])                                   |
|                      | Cohere Embed 4                      | up to 128k adjustable    | 128K tokens             | ~$120 / 1M tokens          | Multimodal + text embed, high context ([Cloud Price][11])                  |
|                      | Cohere Embed 4 (compressed dims)    | 256 / 512 / 1024 / 1536  | 128K tokens             | ~$120 / 1M tokens          | Adjustable dims with quantization ([Cloud Price][11])                      |
| **AWS Bedrock**      | amazon.titan-embed-text-v2          | 256 / 512 / 1024         | 8192 tokens             | —                          | Default Titan embeddings, configurable dims ([AWS Documentation][12])      |
|                      | amazon.titan-embed-text-v1          | 1536                     | 8192 tokens             | —                          | Older Titan embed version ([LiteLLM][13])                                  |
|                      | amazon.nova-embed-text-v2           | 256 / 384 / 1024 / 3072  | 8192 tokens             | —                          | Flexible dims + multimodal support ([AI SDK][14])                          |
|                      | cohere.embed-english-v3             | 1024                     | ~depends                | —                          | Standard Cohere English embed via Bedrock ([LiteLLM][13])                  |
|                      | cohere.embed-multilingual-v3        | 1024                     | ~depends                | —                          | Multilingual embed ([LiteLLM][13])                                         |
|                      | cohere.embed-v4                     | 256 / 512 / 1024 / 1536  | 128K tokens             | ~$120 / 1M tokens          | Large multimodal embed support ([Cloud Price][11])                         |
| **Google Vertex AI** | gemini-embedding-001                | ~3072                    | 2048 tokens             | ~$150 / 1M tokens (online) | Large high-quality embed ([Google Cloud][15])                              |
|                      | text-embedding-005                  | 768                      | 2048 tokens             | ~$100 / 1M tokens          | Balanced embed ([Google Cloud Documentation][16])                          |
|                      | text-multilingual-embedding-002     | 768                      | 2048 tokens             | ~$100 / 1M tokens          | Multilingual support ([Google Cloud Documentation][16])                    |
|                      | e5-large-v2 (OSS)                   | 1024                     | ~512 tokens             | —                          | Vertex supports E5 OSS embeddings ([Google Cloud][17])                     |
|                      | e5-base-v2 (OSS)                    | 768                      | ~512 tokens             | —                          | Lower footprint OSS embed ([Google Cloud][17])                             |
|                      | multilingual-e5-small (OSS)         | 384                      | ~512 tokens             | —                          | Compact multilingual embed ([Google Cloud][17])                            |

---

## 📊 **What These Specs Mean**

### 📍 **Embedding Dimensions**

* **Higher dimensions (e.g., 3072)** tend to capture finer semantic detail but cost more to store and compute.
* **Lower dims (e.g., 256–1024)** are faster, cheaper, and often sufficient for search + clustering. ([AI SDK][14])

### 📍 **Max Input / Context**

* Provider defaults are generally **8K tokens** for most current models, except some Azure Foundry and Cohere Embed 4 models which support **128K tokens**. ([Cloud Price][11])
* Vertex AI’s **gemini-embedding-001** supports **2K tokens** input. ([Google Cloud][15])

### 📍 **Pricing**

* **OpenAI text-embedding-3-small:** ~$0.00002 / 1K tokens → ~$20 / 1M tokens ([Cloud Price][2])
* **OpenAI text-embedding-3-large:** ~$0.00013 / 1K tokens → ~$130 / 1M tokens ([Cloud Price][3])
* **Azure Cohere Embed 4:** ~$0.12 / 1K tokens → ~$120 / 1M tokens ([Cloud Price][11])
* **Vertex AI embedding:** varies by model (Gemini: ~$150 / 1M online) ([Google Cloud][15])
* **AWS Bedrock pricing:** usually per inference, depends on model and region (not always publicly published). ([Amazon Web Services, Inc.][18])

---

💡 **Usage tips:**

* **Quality focus:** Large dims like 3072 (OpenAI text-embedding-3-large, Gemini) tend to rank better for semantic relevance.
* **Cost efficiency:** Smaller dims like 1536–1024 offer good balance for typical retrieval and clustering tasks.
* **Multilingual & multimodal:** Embed models that support multilingual or images (Cohere Embed 4, Nova) add flexibility for RAG systems.

[1]: https://openai.com/research/new-embedding-models-and-api-updates/?utm_source=chatgpt.com "New embedding models and API updates | OpenAI"
[2]: https://cloudprice.net/models/text-embedding-3-small?utm_source=chatgpt.com "Text Embedding 3 Small Pricing & Specs | AI Models | CloudPrice"
[3]: https://cloudprice.net/models/text-embedding-3-large?utm_source=chatgpt.com "Text Embedding 3 Large Pricing & Specs | AI Models | CloudPrice"
[4]: https://cloudprice.net/models/azure%2Ftext-embedding-ada-002?utm_source=chatgpt.com "Text Embedding Ada 002 Pricing & Specs | AI Models | CloudPrice"
[5]: https://docs.upsonic.ai/concepts/embeddings?utm_source=chatgpt.com "Embeddings - Upsonic AI"
[6]: https://www.reddit.com/r/n8n/comments/1qp7y3f/embedding_model_textembedding004_disappeared_need/?utm_source=chatgpt.com "Embedding model text-embedding-004 disappeared — need help choosing a stable alternative for Hybrid RAG"
[7]: https://learn.microsoft.com/en-us/azure/ai-foundry/model-inference/how-to/use-embeddings?utm_source=chatgpt.com "How to generate embeddings with Azure AI Foundry Models service - Azure AI Foundry | Microsoft Learn"
[8]: https://ai.azure.com/catalog/models/text-embedding-3-large?utm_source=chatgpt.com "AI Model Catalog | Microsoft Foundry Models"
[9]: https://ai.azure.com/catalog/models/voyage-3.5-embedding-model?utm_source=chatgpt.com "Voyage 3.5 Embedding Model - Azure AI Foundry"
[10]: https://blog.voyageai.com/2025/05/20/voyage-3-5/?utm_source=chatgpt.com "voyage-3.5 and voyage-3.5-lite: improved quality for a new retrieval ..."
[11]: https://cloudprice.net/models/azure_ai%2Fembed-v-4-0?utm_source=chatgpt.com "Embed V 4 0 Pricing & Specs | AI Models - Azure VM Comparison"
[12]: https://docs.aws.amazon.com/bedrock/latest/userguide/titan-embedding-models.html?utm_source=chatgpt.com "Amazon Titan Text Embeddings models - Amazon Bedrock"
[13]: https://docs.litellm.ai/docs/providers/bedrock_embedding?utm_source=chatgpt.com "Bedrock Embedding"
[14]: https://ai-sdk.dev/providers/ai-sdk-providers/amazon-bedrock?utm_source=chatgpt.com "Amazon Bedrock provider"
[15]: https://cloud.google.com/vertex-ai/generative-ai/docs/embeddings/get-text-embeddings?utm_source=chatgpt.com "Get text embeddings  |  Generative AI on Vertex AI  |  Google Cloud"
[16]: https://docs.cloud.google.com/vertex-ai/generative-ai/docs/embeddings/get-text-embeddings?utm_source=chatgpt.com "Get text embeddings  |  Generative AI on Vertex AI  |  Google Cloud Documentation"
[17]: https://cloud.google.com/vertex-ai/generative-ai/docs/use-embedding-models?utm_source=chatgpt.com "Use embedding models with Vertex AI RAG Engine  |  Generative AI on Vertex AI  |  Google Cloud"
[18]: https://aws.amazon.com/bedrock/pricing/?utm_source=chatgpt.com "Amazon Bedrock Pricing – AWS"
