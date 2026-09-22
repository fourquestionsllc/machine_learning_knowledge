If your **AI agent is writing wrong values into a database**, don't try to fix it only by changing the prompt. Treat the database write as a **high-risk tool/action** and put validation between the agent and the DB.

A good architecture is:

```text
User
  ↓
AI Agent
  ↓
Structured output
  ↓
Validation layer       ← check here
  ↓
Business rules         ← check here
  ↓
DB write tool
  ↓
Database
```

### 1. Never let the agent directly generate SQL

Avoid:

```text
Agent → SQL → Database
```

Instead expose a constrained tool:

```python
update_customer(
    customer_id: str,
    email: str,
    status: str
)
```

The agent can choose the values, but **your application owns the actual database operation**.

---

### 2. Validate the values before writing

For example:

```python
def update_customer(customer_id, email, status):

    if not customer_id:
        raise ValueError("Missing customer_id")

    if status not in ["active", "inactive", "pending"]:
        raise ValueError("Invalid status")

    if not is_valid_email(email):
        raise ValueError("Invalid email")

    db.update_customer(
        customer_id=customer_id,
        email=email,
        status=status
    )
```

The important point is:

> **The LLM is allowed to propose a value; deterministic code decides whether that value is acceptable.**

---

### 3. Add database constraints

Don't rely exclusively on application validation.

For example:

```sql
ALTER TABLE customer
ADD CONSTRAINT valid_status
CHECK (status IN ('active', 'inactive', 'pending'));
```

And use:

* `NOT NULL`
* `CHECK`
* `UNIQUE`
* foreign keys
* appropriate data types
* length constraints

This gives you a second line of defense.

---

### 4. For important values, verify against the source of truth

Suppose the agent receives:

> "Change John's account balance to $5,000."

Don't simply trust the model.

Instead:

```text
Agent
 ↓
proposed balance = 5000
 ↓
Get account from DB
 ↓
Business-rule validation
 ↓
Authorization check
 ↓
Write
```

For example:

```python
current = get_account(account_id)

if proposed_balance < 0:
    reject()

if proposed_balance > current.credit_limit:
    reject()

update_balance(account_id, proposed_balance)
```

---

### 5. Add a confirmation step for dangerous writes

For high-impact operations:

```text
Agent proposes:

customer_id: 12345
status: inactive

        ↓

Validation

        ↓

"You're about to change customer 12345
from active → inactive."

        ↓

Approval

        ↓

DB write
```

You don't necessarily need human approval for every write—only **high-risk operations**.

---

### 6. Log the complete write decision

For every agent-generated DB modification, record:

```json
{
  "request": "Deactivate John's account",
  "agent_proposed": {
    "customer_id": "12345",
    "status": "inactive"
  },
  "validated": true,
  "written": true,
  "timestamp": "..."
}
```

This makes debugging much easier.

---

## For your regression testing

Since you were asking about regression testing earlier, I would specifically create **DB-write regression tests**.

Example:

```text
Test: UPDATE_CUSTOMER_001

Input:
"Set John's status to inactive"

Expected:
customer_id = 12345
status = inactive

Agent output:
customer_id = 12354    ← WRONG

Validation:
FAIL

Database:
NO WRITE
```

The critical rule should be:

> **If validation fails, the DB write must not happen.**

Then add tests for:

* wrong ID
* wrong field
* wrong value
* hallucinated value
* missing value
* wrong data type
* unauthorized record
* stale information
* conflicting instructions
* duplicate writes
* partial failure

### Best practical fix

If this is happening in a real agent system, I'd prioritize these **in this order**:

**1. Constrain the DB tool → 2. deterministic validation → 3. DB constraints → 4. audit logging → 5. regression tests → 6. prompt/model improvements.**

Prompting the agent to "be more careful" should be the **last line of defense**, not the primary protection.
