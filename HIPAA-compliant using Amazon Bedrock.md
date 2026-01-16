Below is a **concrete, end-to-end HIPAA-compliant example using Amazon Bedrock** in an **agentic AI setup**. I’ll keep it practical and aligned with how teams actually pass security reviews.

---

# HIPAA-Compliant Agentic AI Using **Amazon Bedrock**

> **Assumption**
> You are a **Covered Entity or Business Associate**, and AWS is used under the **AWS HIPAA Eligible Services** program (AWS signs a BAA).

Amazon Bedrock **is HIPAA-eligible** *when used correctly*, but **HIPAA compliance is architectural, not automatic**.

---

## 1. Reference Architecture (HIPAA-Safe)

```
Client (Clinician App)
  |
  v
API Gateway (Auth + WAF)
  |
  v
PHI Gatekeeper (Lambda)
  |
  v
De-Identification Agent (Lambda)
  |
  v
Agent Orchestrator (Step Functions / LangGraph)
  |
  +--> Task Agent (Bedrock LLM – de-ID only)
  |
  +--> RAG Retriever (Encrypted Vector Store)
  |
  v
Human Review (Optional)
  |
  v
Output Filter / Re-ID Agent
  |
  v
Client
```

---

## 2. Why Bedrock Works for HIPAA

| Requirement              | How Bedrock Meets It                  |
| ------------------------ | ------------------------------------- |
| BAA                      | AWS provides BAA                      |
| Data isolation           | Your VPC, your IAM                    |
| No training on your data | Bedrock models don’t train on prompts |
| Encryption               | KMS + TLS                             |
| Model choice             | Claude, Titan, etc.                   |

> ⚠️ **Critical**: Only **HIPAA-eligible AWS services** may touch PHI.

---

## 3. Agent Roles (HIPAA Enforcement)

### Agent 1️⃣ PHI Gatekeeper (NO LLM)

**Purpose**

* Detect PHI
* Decide allowed workflow

**Implementation**

* AWS Comprehend Medical
* Regex + rules
* No Bedrock calls here

```python
def gatekeeper(text):
    if detect_phi(text):
        return "DEIDENTIFY"
    return "GENERAL"
```

---

### Agent 2️⃣ De-Identification Agent (NON-Bedrock)

Use **AWS Comprehend Medical** or custom NLP.

```python
import boto3

comprehend = boto3.client("comprehendmedical")

def deidentify(text):
    entities = comprehend.detect_phi(Text=text)["Entities"]
    for e in entities:
        text = text.replace(e["Text"], "[REDACTED]")
    return text
```

✔ Raw PHI **never** reaches Bedrock
✔ Reversible tokens optional (stored in DynamoDB with KMS)

---

### Agent 3️⃣ Task Agent (Amazon Bedrock LLM)

**Allowed**

* Summarization
* Classification
* Reasoning on **de-identified text only**

**Disallowed**

* Diagnosis
* Treatment decisions
* Re-identification

---

## 4. Calling Bedrock Safely (Python Example)

### IAM Policy (Minimum Required)

```json
{
  "Effect": "Allow",
  "Action": ["bedrock:InvokeModel"],
  "Resource": "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-sonnet"
}
```

---

### Bedrock Invoke Example (Claude)

```python
import boto3
import json

bedrock = boto3.client(
    service_name="bedrock-runtime",
    region_name="us-east-1"
)

def call_bedrock(deid_text):
    prompt = f"""
You are a clinical documentation assistant.
The text is fully de-identified.
Do NOT attempt re-identification.

Summarize key clinical issues:

{deid_text}
"""

    response = bedrock.invoke_model(
        modelId="anthropic.claude-3-sonnet-20240229-v1:0",
        contentType="application/json",
        accept="application/json",
        body=json.dumps({
            "anthropic_version": "bedrock-2023-05-31",
            "max_tokens": 500,
            "messages": [
                {"role": "user", "content": prompt}
            ]
        })
    )

    result = json.loads(response["body"].read())
    return result["content"][0]["text"]
```

✔ No PHI
✔ No storage
✔ No training

---

## 5. HIPAA-Safe RAG with Bedrock

### ❌ Dangerous

* Storing PHI embeddings
* Shared OpenSearch index

### ✅ Safe Pattern

| Layer            | Rule                               |
| ---------------- | ---------------------------------- |
| Vector DB        | Amazon OpenSearch (HIPAA eligible) |
| Embeddings       | **De-identified only**             |
| Encryption       | KMS                                |
| Tenant Isolation | Index per tenant                   |
| TTL              | Auto-expiry                        |

```python
# Pseudocode
deid_docs = deidentify(raw_docs)
embeddings = titan_embed(deid_docs)
store_encrypted(embeddings)
```

---

## 6. Human-in-the-Loop (Strongly Recommended)

HIPAA + FDA guidance → **LLMs are assistive**

Use:

* Amazon A2I
* Manual approval UI
* Step Functions wait state

```
LLM Output
   ↓
Human Review
   ↓
Approve / Reject
```

---

## 7. Logging & Auditing (HIPAA Requirement)

### What to Log (CloudWatch)

```json
{
  "agent": "task_agent_v1",
  "action": "summarization",
  "data_class": "PHI_DEIDENTIFIED",
  "request_id": "abc-123",
  "timestamp": "2026-01-15T21:10:00Z"
}
```

### What NOT to Log

❌ Prompts
❌ Responses
❌ Embeddings

---

## 8. Network & Security Controls

| Control        | AWS Service               |
| -------------- | ------------------------- |
| Private access | VPC Endpoints for Bedrock |
| Key mgmt       | AWS KMS                   |
| Secrets        | AWS Secrets Manager       |
| Auth           | IAM + Cognito             |
| Monitoring     | GuardDuty + CloudTrail    |

---

## 9. Step Functions = HIPAA-Safe Agent Orchestration

```json
Gatekeeper
  → DeIdentify
     → BedrockTask
        → HumanReview
           → OutputFilter
```

✔ Deterministic
✔ Auditable
✔ No autonomous behavior

---

## 10. Common Bedrock HIPAA Mistakes

❌ Sending PHI directly to Bedrock
❌ Storing prompts in S3 without encryption
❌ Using shared vector indices
❌ Debug logging LLM inputs
❌ Autonomous agents taking actions

---

## 11. Security Review Talking Point (Use This)

> “All PHI is detected and de-identified before any LLM invocation.
> Amazon Bedrock receives only anonymized data, does not store prompts,
> does not train on customer data, and is accessed via private VPC endpoints
> under an AWS BAA.”

---

## Final Takeaway

**HIPAA + Bedrock works when:**

* Bedrock is **not trusted with PHI**
* Agents are **specialized and constrained**
* Humans remain in control
* Everything is auditable
