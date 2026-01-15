Here’s a **practical, engineering-focused solution set to reduce LLM hallucinations**, especially relevant for **RAG / agentic systems** like the ones you build at GEICO / CDC / J&J.

---

## 1. Ground the Model (Most Important)

**Hallucination = generation without evidence**

### ✅ Retrieval-Augmented Generation (RAG)

* Retrieve **only relevant documents**
* Pass retrieved text **verbatim** into the prompt
* Force citations

**Best practices**

* Chunk by **semantic meaning**, not size
* Use **hybrid search** (BM25 + embeddings)
* Re-rank with a cross-encoder

**Prompt constraint**

```
Answer ONLY using the provided context.
If the answer is not present, say "Not found in sources."
```

---

## 2. Add Confidence Gating (Refuse When Uncertain)

LLMs should **decline** instead of guessing.

### Techniques

* **Logprobs / confidence scoring**
* Ask model to output:

```json
{
  "answer": "...",
  "confidence": 0.00-1.00,
  "sources": [...]
}
```

Reject or escalate when confidence < threshold.

---

## 3. Use Multi-Step Verification (Agentic Pattern)

Instead of 1-shot answers:

### Pattern: **Generate → Verify → Answer**

1. Agent A drafts answer
2. Agent B checks against sources
3. Agent C decides:

   * ✅ answer
   * ❌ refuse
   * 🔄 re-retrieve

This reduces hallucinations **40–60%** in production systems.

---

## 4. Force Structured Outputs

Free-text = hallucination risk.

Use **schemas / JSON mode / function calling**

```json
{
  "claim": "string",
  "evidence": ["doc_id:line"],
  "confidence": "high|medium|low"
}
```

If evidence array is empty → block the answer.

---

## 5. Retrieval Quality > Model Size

Most hallucinations come from **bad retrieval**, not the LLM.

### Fix retrieval by:

* Increasing top-K recall
* Domain-specific embeddings
* Metadata filters (date, jurisdiction, patient group)
* Query rewriting (LLM → better search query)

---

## 6. Model Routing (Cost + Accuracy)

Use **cheap model first**, escalate only if needed.

Example:

```
Haiku → Sonnet → Opus
```

Escalate when:

* Retrieval confidence is low
* Question is complex / multi-hop
* High-risk domain (health, legal)

---

## 7. Explicit “I Don’t Know” Training

Add few-shot examples where **correct behavior = refusal**

Example:

```
Q: What caused patient X’s reaction?
Context: No mention
A: The information is not available in the provided sources.
```

---

## 8. Post-Answer Validation (Critical for Health / Finance)

Run checks **after generation**:

* Fact overlap with sources
* Named entity verification
* Rule-based constraints (no drug advice, no diagnosis)

If failed → suppress answer.

---

## 9. Evaluation & Monitoring (Often Missing)

Track hallucination explicitly.

### Metrics

* Groundedness score
* Faithfulness
* Answerable vs unanswerable accuracy

### Tools

* RAGAS
* TruLens
* LangSmith
* Custom “citation coverage %”

---

## 10. When Hallucinations Still Happen

**Fallback strategies**

* Ask clarifying question
* Return partial answer with citation
* Route to human review
* Provide source links only (no synthesis)

---

## Reference Architecture (Battle-Tested)

```
User Query
   ↓
Query Rewriter
   ↓
Hybrid Retrieval + Re-ranker
   ↓
Context Window (STRICT)
   ↓
LLM (JSON + Citations)
   ↓
Verifier Agent
   ↓
Confidence Gate
   ↓
Answer / Refusal / Escalation
```

