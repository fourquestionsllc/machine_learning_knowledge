# Regression Testing for an Agentic AI Product

## 1. What is regression testing for an agentic AI product?

Regression testing verifies that changes to an agentic AI system do not break behavior that previously worked. For an agentic product, this is broader than testing a conventional ML model because the system may reason, call tools, access data, maintain state, and take actions.

The goal is to detect regressions in:

- **Task outcomes:** Does the agent still complete the intended task?
- **Reasoning and planning:** Does it choose an appropriate sequence of steps?
- **Tool use:** Does it select the right tool and provide valid arguments?
- **Safety:** Does it avoid unauthorized, unsafe, or destructive actions?
- **Reliability:** Does it handle failures, retries, timeouts, and unexpected inputs?
- **Grounding:** Does it use the correct information and avoid unsupported claims?
- **Conversation state:** Does it preserve relevant context without leaking or mixing state?
- **Cost and latency:** Does a change cause excessive tool calls, tokens, or execution time?

## 2. Build a regression test suite

Create a version-controlled set of representative scenarios. Each test should contain:

1. **Initial state** – user request, available data, permissions, and environment.
2. **Expected behavior** – what the agent should accomplish.
3. **Allowed actions** – tools/actions the agent may use.
4. **Forbidden actions** – actions that must never occur.
5. **Expected final result** – facts, state changes, or output requirements.
6. **Evaluation criteria** – rules or metrics used to determine pass/fail.

Include several categories:

| Category | Example |
|---|---|
| Happy path | Agent completes a normal customer-support task |
| Edge case | Missing or ambiguous information |
| Tool failure | API returns an error or times out |
| Safety | User requests an unauthorized operation |
| Multi-step | Agent must plan and execute several actions |
| State | Agent must use information from earlier turns |
| Adversarial | Prompt injection or conflicting instructions |
| Recovery | Agent detects failure and retries or changes strategy |

## 3. Test at multiple levels

### A. Component tests

Test individual tools, prompts, policies, retrieval components, and parsers independently. These tests should be deterministic where possible.

### B. Agent trajectory tests

Run the complete agent loop and inspect the trajectory:

**Input → Plan → Tool calls → Observations → Next steps → Final answer/action**

Check not only the final answer but also whether the agent took an acceptable path.

### C. End-to-end tests

Run realistic user scenarios against an environment that closely resembles production. Verify the final response **and** resulting system state.

For actions that can modify real data, use mocks, sandboxes, or reversible test environments.

## 4. Use deterministic and LLM-based evaluation

Traditional assertions are useful for objective requirements:

- HTTP status
- database state
- tool selected
- required field present
- number of retries
- permission checks
- exact structured output

For subjective behavior, use an evaluator model ("LLM-as-a-judge") with a fixed rubric. For example, score:

- Task completion
- Correctness
- Relevance
- Safety
- Instruction following
- Tool-use quality

Do not rely exclusively on an LLM judge. Combine it with deterministic checks and, for critical workflows, human review.

## 5. Establish regression metrics

Track metrics across releases, such as:

- **Task success rate**
- **Tool-call accuracy**
- **Safety violation rate**
- **Hallucination/grounding error rate**
- **Recovery success rate**
- **Average steps/tool calls**
- **Latency**
- **Token/API cost**

Compare the current release with a known-good baseline. Define thresholds that block a release when important metrics degrade.

Example:

```text
Task success       >= 95%
Safety violations  = 0 for critical scenarios
Grounding score    >= baseline - 2%
P95 latency        <= baseline + 10%
Cost per task      <= baseline + 15%
```

Thresholds should reflect the risk and business impact of the product.

## 6. Run regression tests continuously

A practical CI/CD flow is:

```text
Code / Prompt / Model change
          ↓
Unit & component tests
          ↓
Agent regression suite
          ↓
Safety & adversarial tests
          ↓
Compare against baseline
          ↓
Human review for critical changes
          ↓
Staging / canary deployment
          ↓
Production monitoring
```

Run a small, fast suite on every pull request and a larger suite nightly or before production releases.

## 7. Handle nondeterminism

Agent outputs can vary between runs. Avoid tests that require one exact response unless the output is intentionally deterministic.

Instead, test **properties and outcomes**. For example:

> The agent must obtain user confirmation before sending an email.

rather than:

> The agent must produce exactly this sentence.

Run important scenarios multiple times and measure the pass rate. This helps identify flaky behavior and changes in reliability.

## 8. Maintain the regression suite

Every production incident should ideally become a regression test. Also add tests when:

- A new tool is introduced.
- The model or prompt changes.
- Permissions or policies change.
- Retrieval/data sources change.
- A new agent capability is released.
- A previously observed failure mode is discovered.

Keep tests representative and remove redundant cases periodically.

## 9. Key principle

For an agentic AI product, regression testing should evaluate **the entire behavior loop, not just the final text**.

A strong regression strategy combines:

**deterministic assertions + trajectory evaluation + safety tests + outcome-based evaluation + baseline comparison + production monitoring.**

The most important question is not simply *“Did the model give the same answer?”* but:

> **“Did the agent still achieve the intended outcome safely, reliably, and within acceptable cost and latency?”**
