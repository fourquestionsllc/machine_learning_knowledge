**Columns included**

* Model
* Params (if public)
* Context Window
* Modality
* Strengths / Notes
* Typical Use

I also grouped them by platform for easier scanning.

---

# 🔵 OpenAI — Top Models

| Model         | Params                     | Context | Modality              | Strengths                 | Typical Use                 |
| ------------- | -------------------------- | ------- | --------------------- | ------------------------- | --------------------------- |
| GPT-4o        | undisclosed (~GPT-4 class) | 128K    | Text + Vision + Audio | Fast + cheap + multimodal | General assistant, chatbots |
| GPT-4.1       | undisclosed                | 1M      | Text + Vision         | Very long context         | RAG, document QA            |
| GPT-4.1-mini  | undisclosed                | 1M      | Text                  | Lower cost, long context  | Batch jobs                  |
| GPT-4.1-nano  | undisclosed                | 1M      | Text                  | Ultra cheap/fast          | High-volume tasks           |
| o4            | undisclosed                | 200K+   | Text                  | Strong reasoning          | Agents, planning            |
| o4-mini       | undisclosed                | 200K+   | Text                  | Cheap reasoning           | Tool-calling                |
| GPT-3.5-turbo | ~175B class                | 16K     | Text                  | Budget legacy             | Simple chat                 |
| gpt-oss-20B   | 20B                        | 128K    | Text                  | Open-weight               | Self-hosting/fine-tune      |

---

# 🟦 Microsoft Azure AI Foundry — Top Hosted Models

*(Azure hosts both OpenAI + partner models)*

| Model             | Params         | Context | Modality    | Strengths                 | Typical Use         |
| ----------------- | -------------- | ------- | ----------- | ------------------------- | ------------------- |
| GPT-4o (Azure)    | same as OpenAI | 128K    | Multimodal  | Enterprise hosted         | Copilots            |
| GPT-4.1           | same           | 1M      | Text/Vision | Long docs                 | Knowledge bots      |
| Claude 3.5 Sonnet | undisclosed    | 200K    | Text/Vision | Strong reasoning + safety | Enterprise agents   |
| Claude 3 Opus     | undisclosed    | 200K    | Text/Vision | Deep reasoning            | Research tasks      |
| Llama 3.1 70B     | 70B            | 128K    | Text        | Open weight               | Fine-tuning         |
| Llama 3.1 405B    | 405B           | 128K    | Text        | Very strong open model    | Advanced agents     |
| Cohere Command-R+ | undisclosed    | 128K    | Text        | RAG optimized             | Enterprise search   |
| Mistral Large     | ~123B          | 128K    | Text        | Efficient                 | Cost-sensitive apps |

---

# 🟧 Amazon Web Services Amazon Bedrock — Top Models

| Model              | Params            | Context | Modality    | Strengths            | Typical Use        |
| ------------------ | ----------------- | ------- | ----------- | -------------------- | ------------------ |
| Titan Text Premier | ~65B              | 32K     | Text        | Native AWS           | Secure enterprise  |
| Titan Text Express | ~8–20B            | 8–32K   | Text        | Fast/cheap           | High throughput    |
| Claude 3.5 Sonnet  | undisclosed       | 200K    | Text/Vision | Strong reasoning     | Agents             |
| Claude 3 Haiku     | small             | 200K    | Text/Vision | Very fast            | Real-time          |
| Llama 3.1 70B      | 70B               | 128K    | Text        | Open weight          | Fine-tuning        |
| Llama 3.1 405B     | 405B              | 128K    | Text        | SOTA open            | Complex tasks      |
| Mistral Large      | ~123B             | 128K    | Text        | Good price/perf      | Production         |
| Mixtral 8×7B       | MoE (~46B active) | 32K     | Text        | Efficient sparse MoE | Low cost inference |

---

# 🟩 Google Cloud Vertex AI — Top Models

| Model                      | Params      | Context | Modality    | Strengths            | Typical Use      |
| -------------------------- | ----------- | ------- | ----------- | -------------------- | ---------------- |
| Gemini 1.5 Pro             | undisclosed | 1M–2M   | Multimodal  | Ultra-long context   | Large docs/video |
| Gemini 1.5 Flash           | undisclosed | 1M      | Multimodal  | Fast/cheap           | Realtime apps    |
| Gemini 1.0 Ultra           | undisclosed | 128K    | Multimodal  | Deep reasoning       | Complex tasks    |
| Gemini 1.0 Pro             | undisclosed | 128K    | Multimodal  | Balanced             | General chat     |
| Gemma 27B                  | 27B         | 128K    | Text        | Open weight          | Fine-tune        |
| Gemma 12B                  | 12B         | 128K    | Text        | Mid size             | Private deploy   |
| Gemma 7B                   | 7B          | 32–128K | Text        | Edge friendly        | Local inference  |
| Claude 3.5 Sonnet (hosted) | undisclosed | 200K    | Text/Vision | Enterprise reasoning | Agents           |

---

# ✅ Quick Takeaways

If you’re choosing for engineering work:

* **Longest context** → Gemini 1.5 Pro, GPT-4.1
* **Best reasoning** → Claude 3.5 Sonnet, o4, GPT-4.1
* **Cheapest scale** → Flash / mini / Haiku / Titan Express
* **Open-weight fine-tuning** → Llama 3.1, Gemma, Mixtral
* **Enterprise compliance** → Azure or Bedrock
