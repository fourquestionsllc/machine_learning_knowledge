Below is a **practical, battle-tested guide** to **regression testing for LLMs** (prompts, RAG, agents). This is the exact approach used in production to prevent **silent quality regressions** when prompts/models/retrievers change.

I’ll keep it **implementation-level**, not theory.

---

# 1. What LLM Regression Testing Really Is

LLM regression ≠ “same text output”

Instead, you assert:

* **Behavior stays within acceptable bounds**
* **Failure modes do not increase**
* **Critical signals don’t degrade**

Think of it as **contract testing for AI behavior**.

---

# 2. What Changes Require Regression Tests

Run regression tests when **ANY** of these change:

✅ Prompt wording
✅ System / developer message
✅ Model version
✅ Temperature / top-p
✅ Retriever logic / chunking
✅ Tools / agent graph
✅ Safety filters

---

# 3. Regression Test Types (You Need All 4)

## Type 1: **Golden Test Cases (Behavioral)**

### What to Store

Each test case includes:

```json
{
  "id": "call_reason_dispute_fee",
  "input": "Customer is upset about a late fee",
  "context": ["...retrieved chunks..."],
  "expected": {
    "labels": ["Billing > Fees > Dispute"],
    "must_not": ["Fraud"],
    "confidence_min": 0.7
  }
}
```

**Important:**
You are NOT storing exact outputs — only **constraints**.

---

## Type 2: **Schema & Format Regression**

For structured outputs:

```json
{
  "label": "string",
  "confidence": "float",
  "reasoning": "string"
}
```

### Automated Assertions

```python
assert output["confidence"] >= 0.7
assert output["label"] in ALLOWED_LABELS
```

Fail fast if schema breaks.

---

## Type 3: **Semantic Regression (LLM-as-Judge)**

Use a **judge model** to compare *baseline vs candidate*.

### Judge Prompt (Key Pattern)

```text
You are an evaluator.

Question:
{input}

Baseline Answer:
{baseline}

Candidate Answer:
{candidate}

Score from 1–5:
- Correctness
- Completeness
- Hallucination Risk

Explain briefly.
```

### Fail Criteria

```python
assert candidate_score >= baseline_score - 1
assert hallucination <= 2
```

This is **the most powerful regression signal**.

---

## Type 4: **RAG Regression (Critical)**

Track these **over time**:

| Metric           | Regression Signal    |
| ---------------- | -------------------- |
| Context Recall   | ↓ = retriever broken |
| Faithfulness     | ↓ = hallucination    |
| Answer Relevance | ↓ = prompt issue     |

Use **RAGAS** or **TruLens**.

---

# 4. Baseline vs Candidate Comparison Strategy

### Golden Rule

> Always compare against a **known-good baseline**

### Workflow

```text
Baseline = main branch prompt
Candidate = PR prompt

Same inputs → both systems
Compare metrics → block if degraded
```

---

# 5. Regression Test Execution Flow

### CI Pipeline (Example)

```text
PR opened
 ↓
Run unit tests
 ↓
Run LLM regression suite (100–300 cases)
 ↓
Score comparison
 ↓
FAIL if regression detected
```

---

# 6. Minimal Pytest Example (Realistic)

```python
def test_prompt_regression(test_case):
    baseline = run_llm(BASELINE_PROMPT, test_case.input)
    candidate = run_llm(CANDIDATE_PROMPT, test_case.input)

    judge_score = judge_llm(
        question=test_case.input,
        baseline=baseline,
        candidate=candidate
    )

    assert judge_score["correctness"] >= 4
    assert judge_score["hallucination"] <= 2
```

---

# 7. Regression Thresholds (Use These Defaults)

| Metric             | Fail If  |
| ------------------ | -------- |
| Hallucination rate | +2%      |
| Schema failure     | >1%      |
| Faithfulness       | −5%      |
| Judge correctness  | −1 point |
| Tool error         | +1%      |

---

# 8. Handling Non-Determinism (Very Important)

### Techniques

* Temperature ≤ 0.3
* Run **N=3** samples → majority vote
* Judge average score
* Seed if model supports it

---

# 9. Dataset Design (This Matters Most)

### Recommended Split

| Type        | %   |
| ----------- | --- |
| Happy path  | 40% |
| Edge cases  | 30% |
| Adversarial | 20% |
| Safety      | 10% |

**Include**

* Ambiguous questions
* Missing context
* Conflicting documents
* Long inputs

---

# 10. Agent & Tool Regression

### Assert

```python
assert tool_called == "search_db"
assert num_steps < 6
assert agent_terminated is True
```

Track:

* Infinite loops
* Wrong tool usage
* Premature termination

---

# 11. Storage & Observability

Store for each run:

* Prompt version hash
* Model version
* Retrieval config
* Scores
* Diff explanation

This lets you **explain regressions** later.

---

# 12. What NOT to Do

❌ Exact string comparison
❌ BLEU / ROUGE only
❌ Manual QA only
❌ No baseline comparison
❌ No CI gate

---

# 13. Tooling Stack (Recommended)

| Need            | Tool                  |
| --------------- | --------------------- |
| CI regression   | pytest                |
| Prompt tracking | LangSmith             |
| RAG metrics     | RAGAS                 |
| LLM judge       | GPT-4.1 / GPT-4o-mini |
| Dashboards      | Prometheus / Grafana  |

---

# 14. Minimal Starter Setup (1 Week Plan)

**Day 1–2**

* Create 150 gold test cases
* Lock baseline prompt

**Day 3**

* Implement schema + judge tests

**Day 4**

* Add RAG metrics

**Day 5**

* Wire into CI/CD

