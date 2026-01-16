### What is **Regression Testing**?

**Regression testing** is the process of re-running existing tests after a change (code, configuration, data, prompt, or model) to ensure that **previously working behavior has not broken**.

In short:

> **“Did anything that used to work stop working after the change?”**

---

## Why Regression Tests Matter

Regression testing helps catch:

* Bugs introduced by new features
* Side effects from refactoring
* Performance or accuracy degradation
* Behavior drift in ML/LLM systems

---

## Classic Software Regression Testing (Non-AI)

### Example 1: API Regression Test

**Change:** Add a new optional field to an API response

**Before**

```json
{
  "user_id": 123,
  "status": "active"
}
```

**After (expected)**

```json
{
  "user_id": 123,
  "status": "active",
  "last_login": "2026-01-10"
}
```

**Regression test checks**

* Existing fields (`user_id`, `status`) are unchanged
* API still returns HTTP 200
* Old clients don’t break

---

### Example 2: UI Regression Test

**Change:** Update CSS or layout

**Regression test**

* Login button still visible
* Submit form still works
* Error messages still appear

Tools:

* Playwright
* Cypress
* Selenium

---

## ML Model Regression Testing

Regression testing in ML is **not about exact equality**, but about **performance consistency**.

---

### Example 3: Classification Model Regression

**Change:** Retrain model with new data

**Baseline metrics**

| Metric   | Old Model |
| -------- | --------- |
| Accuracy | 0.84      |
| F1 score | 0.79      |

**Regression rule**

```text
Accuracy must not drop more than 2%
F1 must not drop more than 3%
```

**Regression test fails if**

* Accuracy < 0.82
* F1 < 0.76

---

### Example 4: Prediction Consistency Test

Input:

```json
{
  "monthly_charges": 85,
  "contract": "month-to-month"
}
```

Expected behavior:

* Churn probability ≈ 0.65 ± 0.05

Fail if:

* New model predicts 0.20 or 0.95

---

## LLM / GenAI Regression Testing (Very Important)

Since you’re working with **LLMs, RAG, agents**, regression testing is usually **behavior-based**, not exact-text-based.

---

### Example 5: Prompt Regression Test

**Change:** Modify system prompt

**Test case**

```text
User: What is the refund policy?
```

**Baseline expected behavior**

* Mentions 30-day refund
* No hallucinated policy
* Professional tone

**Regression checks**

* ❌ No new hallucinations
* ✅ Key facts still present
* ✅ Answer length within range

---

### Example 6: RAG Regression Test

**Change:** Update embedding model or chunking strategy

**Test**

* Ask 50 fixed questions
* Verify:

  * Relevant document is retrieved
  * Answer cites correct source
  * No irrelevant doc dominates

Example assertion:

```text
Top-3 retrieved chunks must contain doc_id = "refund_policy_v2"
```

---

### Example 7: Agent Workflow Regression

**Change:** Add a new agent (e.g., summarizer)

**Regression test**

* Existing tools still invoked
* No infinite loops
* Decision agent still routes correctly

Example:

```text
Input → Retrieval Agent → Answer Agent → Output
```

Fail if:

* Summarizer runs before retrieval
* Tool calls exceed threshold

---

## LLM Regression Metrics (Common)

| Type      | Metric                            |
| --------- | --------------------------------- |
| Accuracy  | Exact match / semantic similarity |
| Safety    | Hallucination rate                |
| Style     | Length, tone, formatting          |
| Latency   | p95 response time                 |
| Cost      | Token usage                       |
| Grounding | Citation correctness              |

---

## Simple Regression Test Example (LLM)

```python
test_cases = [
    {
        "input": "What is the refund policy?",
        "must_contain": ["30 days", "refund"],
        "must_not_contain": ["guaranteed profit"]
    }
]
```

---

## When Should You Run Regression Tests?

* After model retraining
* After prompt changes
* After RAG index updates
* After infra changes (GPU, container, model version)
* Before production deployment (CI/CD gate)

---

## Summary

**Regression testing = confidence that changes didn’t break existing behavior**

| System | Regression Focus            |
| ------ | --------------------------- |
| APIs   | Response shape & behavior   |
| UI     | User flows                  |
| ML     | Metric stability            |
| LLM    | Behavior, grounding, safety |
| Agents | Tool flow & decisions       |


* Show **how to automate this in CI/CD**
* Give **real examples for call-reason classification or banking transcripts** (based on your work)
