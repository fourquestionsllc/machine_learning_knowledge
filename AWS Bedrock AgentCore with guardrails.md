Below is an **actual example of Amazon Bedrock AgentCore agent code** that shows how to:

1. Build a simple agent using a Python agent framework (e.g., Strands Agents)
2. Integrate **RAG retrieval** (via a knowledge base or external retriever pattern)
3. Include **guardrails** to control safety/policy

I’ll include code you can run locally with the AgentCore starter toolkit and then explain how **RAG + guardrails** fit into it.

---

## ✅ 1. Basic AgentCore Agent (Strands + AgentCore)

This is the **baseline agent** that you’ll deploy in Bedrock AgentCore Runtime. It just takes a prompt and returns a response using a model behind Strands Agents.

```python
# agentcore_rag_guardrails_agent.py

from bedrock_agentcore.runtime import BedrockAgentCoreApp
from strands import Agent, Tool
from strands.models import BedrockModel

# 1) Initialize AgentCore entrypoint
app = BedrockAgentCoreApp()

# 2) Build your model with guardrail attached (optional)
bedrock_model = BedrockModel(
    model_id="us.anthropic.claude-3-7-sonnet-20250219-v1:0",
    # Example guardrail values from Bedrock console or API
    guardrail_id="your-guardrail-id",
    guardrail_version="your-guardrail-version"
)

# 3) Build Strands agent with the RAG retriever tool
# “retriever” below is a semantic search tool you implement (e.g., through S3/Opensearch)
class RetrieverTool(Tool):
    def run(self, query: str) -> str:
        # implement retrieval from your vectorstore here
        # return top-k docs or a context string
        return my_vector_search_function(query)

# Add the tool into the agent
tools = [
    RetrieverTool(name="RAGRetriever")
]

agent = Agent(
    model=bedrock_model,
    tools=tools,
    # Customize system prompt if needed
    system_prompt="You are a helpful assistant using RAG retrieval and safety guardrails.",
)

# 4) The AgentCore entrypoint
@app.entrypoint
def invoke(payload):
    user_prompt = payload.get("prompt", "")
    
    # Use retrieval first as context
    # E.g., you call your retriever tool to get grounding documents
    retrieved_context = agent.call_tool("RAGRetriever", user_prompt)

    # Build combined prompt
    prompt_with_context = f"""
    Context:
    {retrieved_context}

    User: {user_prompt}
    """

    # Ask your agent to respond with context
    result = agent(prompt_with_context)

    return {"response": result.message}

# 5) Start the agent runtime
if __name__ == "__main__":
    app.run()
```

### What’s Going On Here?

* **BedrockAgentCoreApp** is the entrypoint that AgentCore Runtime will host. ([AWS Documentation][1])
* **BedrockModel** wraps an LLM from Bedrock with attached guardrails (filtering/policy enforcement). ([DEV Community][2])
* **Strands Agents** orchestrates the RAG tool + model prompting logic. ([GitHub][3])

---

## 🎯 2. RAG Retriever “Tool”

This `RetrieverTool` can be backed by:

* **S3 + S3Vectors** (Amazon Bedrock KB adapters)
* **OpenSearch Serverless Vector Search**
* **Custom Vectorstore (FAISS, Pinecone, etc.)**

The idea is:

```python
def my_vector_search_function(query: str) -> str:
    """
    1) Compute embedding for query
    2) Search your vector database for top-k
    3) Return a combined context string
    """
    # This is a placeholder — plug in your real retrieval stack
    docs = vector_search(query)
    context_text = "\n\n".join(doc.text for doc in docs)
    return context_text
```

This function fetches relevant context so your agent is grounded in real RAG results — very helpful for large code repos or document corpora.

---

## 🛡 3. Attaching Guardrails

`guardrail_id` and `guardrail_version` in the `BedrockModel` constructor ensure that **Bedrock Guardrails policies** are applied to:

✔ Input moderation
✔ Output safety
✔ Domain-specific filtering

You create the guardrail (usually via the Bedrock console or API) and then pass the IDs here. Guardrails will then enforce your policy at **model invocation time**. ([AWS Documentation][4])

```python
bedrock_model = BedrockModel(
    model_id="us.anthropic.claude-3-7-sonnet-20250219-v1:0",
    guardrail_id="your-guardrail-id",
    guardrail_version="your-guardrail-version"
)
```

---

## 🚀 4. Deploy to AWS Bedrock AgentCore Runtime

### Local Dev

```bash
pip install bedrock-agentcore strands-agents bedrock-agentcore-starter-toolkit
agentcore create  # choose Strands Agents when prompted
agentcore dev
```

### Deploy to Production Runtime

```bash
agentcore configure -e agentcore_rag_guardrails_agent.py
agentcore launch
agentcore invoke '{"prompt":"Explain how to refactor this code..."}'
```

AgentCore packages your code, assigns a runtime, and sets up CloudWatch logs — all managed and secure. ([AWS Documentation][5])

---

## 📌 Notes & Best Practices

✅ **Keep RAG context size limited** before sending to the model.
✅ **Monitor metrics and guardrail violations** in CloudWatch for safety insights.
✅ **Use vector stores with metadata filters** to narrow your retrieval scope.
✅ **Test locally with `agentcore dev`** extensively before production deploy. ([AWS Documentation][5])

---

## 📌 Summary

| Feature              | How It’s Done                            |
| -------------------- | ---------------------------------------- |
| AgentCore entrypoint | `BedrockAgentCoreApp()`                  |
| RAG retrieval        | Custom retriever tool + vector search    |
| Guardrails           | Attached to model via `guardrail_id`     |
| Deployment           | `agentcore launch` + AWS managed runtime |

---

If you want a **full reference starter repository** example that includes **vector search + tools + deployment scripts**, the **Amazon Bedrock AgentCore Samples repo** on GitHub is the best place to explore next — it includes RAG and tool integration patterns: [https://github.com/awslabs/amazon-bedrock-agentcore-samples](https://github.com/awslabs/amazon-bedrock-agentcore-samples) ([GitHub][6])

---

[1]: https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/agentcore-get-started-toolkit.html?utm_source=chatgpt.com "Get started with Amazon Bedrock AgentCore - Amazon Bedrock AgentCore"
[2]: https://dev.to/aws-builders/add-guardrails-to-your-strands-agent-in-zero-time-with-amazon-bedrock-guardrails-1gam?utm_source=chatgpt.com "Add Guardrails to Strands Agent w/ Amazon Bedrock"
[3]: https://github.com/aws-samples/sample-strands-agentcore-starter?utm_source=chatgpt.com "aws-samples/sample-strands-agentcore-starter"
[4]: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails.html?utm_source=chatgpt.com "Detect & Filter Harmful Content: Amazon Bedrock Guardrails"
[5]: https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-get-started-toolkit.html?utm_source=chatgpt.com "Get started with the Amazon Bedrock AgentCore starter toolkit - Amazon Bedrock AgentCore"
[6]: https://github.com/awslabs/amazon-bedrock-agentcore-samples?utm_source=chatgpt.com "awslabs/amazon-bedrock-agentcore-samples"



Good question — this is a **core Bedrock concept** and it’s easy to gloss over when you just see `"your-guardrail-id"` in code.

Below is a **clear, end-to-end explanation** of **what a Bedrock Guardrail is, how it’s implemented, how the ID is created, and how it’s enforced at deploy/runtime**.

---

# 1️⃣ What is a Bedrock Guardrail (Conceptually)?

A **Bedrock Guardrail** is a **managed policy object** in Amazon Bedrock that enforces **input and output controls** on model invocations.

Think of it as:

> **A policy layer that wraps the model call**
> (not code, not prompts, not LangChain logic)

It is enforced **inside Bedrock’s inference service**, *before* and *after* the LLM generates text.

---

# 2️⃣ What Is `"your-guardrail-id"`?

`your-guardrail-id` is **not something you define in code**.

It is:

* An **AWS resource**
* Created in **Amazon Bedrock → Guardrails**
* Identified by:

  * `guardrailId`
  * `guardrailVersion`

Example:

```text
Guardrail ID: gr-0a1b2c3d4e5f
Version: 1
```

These values are then **referenced** by your agent or model invocation.

---

# 3️⃣ How Guardrails Are Created (Actual Implementation)

## Option A — Bedrock Console (Most Common)

### Steps:

1. Go to **AWS Console → Amazon Bedrock**
2. Open **Guardrails**
3. Click **Create guardrail**
4. Configure policies:

   * 🔒 Content filters (violence, hate, sexual, etc.)
   * 🧠 Topic restrictions (allowed / denied topics)
   * 🧾 PII detection (block or redact)
   * ✍️ Output formatting / grounding rules
5. Save → **Version is created**
6. You get:

   * `guardrailId`
   * `guardrailVersion`

No code yet — this is **infrastructure-level configuration**.

---

## Option B — Create via API / IaC (Advanced)

You can create guardrails via API (or Terraform / CloudFormation).

Conceptually:

```python
bedrock.create_guardrail(
    name="clinical-safety-guardrail",
    blocked_input_topics=["self-harm", "illegal-drugs"],
    pii_entities=["SSN", "MedicalRecordNumber"],
    blocked_output_topics=["medical advice"]
)
```

AWS returns:

```json
{
  "guardrailId": "gr-0a1b2c3d4e5f",
  "version": "1"
}
```

---

# 4️⃣ How Guardrails Are Enforced (Critical Part)

### 🔑 Guardrails are enforced **by Bedrock itself**

Not:

* LangChain
* Strands
* AgentCore
* Your application code

### Enforcement Flow

```
User Input
   ↓
Bedrock Guardrail (INPUT CHECK)
   ↓
LLM Inference
   ↓
Bedrock Guardrail (OUTPUT CHECK)
   ↓
Response or Blocked/Error
```

If the request violates a rule:

* The model **never executes**, or
* The output is **blocked, redacted, or replaced**

---

# 5️⃣ How `"your-guardrail-id"` Is Used in Code

## Plain Bedrock Model Invocation

```python
response = bedrock_runtime.invoke_model(
    modelId="anthropic.claude-3-sonnet",
    guardrailIdentifier="gr-0a1b2c3d4e5f",
    guardrailVersion="1",
    body=payload
)
```

At this point:

* Guardrail is **already enforced**
* Your code **cannot bypass it**

---

## AgentCore / Strands Agent (What You Saw)

```python
bedrock_model = BedrockModel(
    model_id="anthropic.claude-3-sonnet",
    guardrail_id="gr-0a1b2c3d4e5f",
    guardrail_version="1"
)
```

### Important:

AgentCore simply **passes these values to Bedrock Runtime**.
It does **not implement guardrails itself**.

---

# 6️⃣ How This Works at **AgentCore Deploy Time**

When you run:

```bash
agentcore launch
```

AgentCore:

1. Packages your agent code
2. Deploys a managed runtime
3. Stores the **guardrail ID/version as part of model config**
4. Every invocation uses that guardrail

You **cannot remove or override** the guardrail at runtime unless you redeploy.

---

# 7️⃣ What Happens on Violation? (Actual Behavior)

### Example: PII Block

User input:

```
What is John Doe's SSN?
```

Result:

```json
{
  "errorType": "GuardrailViolation",
  "message": "Input violates PII policy"
}
```

### Example: Output Block

Model tries to generate disallowed content → Bedrock:

* Stops output
* Returns a safe fallback message

---

# 8️⃣ Guardrails vs Prompt-Based Safety (Important Distinction)

| Feature                  | Guardrails         | Prompt Rules     |
| ------------------------ | ------------------ | ---------------- |
| Enforced location        | Bedrock service    | Inside model     |
| Can be bypassed          | ❌ No               | ✅ Yes            |
| Auditable                | ✅ Yes (CloudWatch) | ❌ No             |
| Consistent across agents | ✅ Yes              | ❌ No             |
| Regulated workloads      | ✅ Required         | ❌ Not sufficient |

For **health, finance, gov, clinical**, guardrails are **mandatory**.

---

# 9️⃣ Best Practice for AgentCore + RAG + Guardrails

**Use all three layers:**

1. **Guardrails (Bedrock)**
   → Hard safety boundary
2. **RAG grounding**
   → Reduce hallucinations
3. **Agent logic**
   → Business rules, tool usage

This is exactly how **AWS positions AgentCore for enterprise AI**.

---

# 10️⃣ Mental Model (One Sentence)

> `"your-guardrail-id"` is a **Bedrock-managed safety policy**, created outside your code, enforced by AWS infrastructure, and attached to every model invocation your agent makes.

