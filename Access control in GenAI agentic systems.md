Access control in **GenAI agentic systems** is critical because agents can call **tools/functions, APIs, databases, and external services**. You must ensure that **consumers (users, services, or agents)** only access the **functions and data they are authorized to use**.

Below is a **practical architecture and techniques** for implementing access control in GenAI agents.

---

# 1. What Needs Access Control in Agentic AI

In a GenAI agent system, access must be controlled for:

### User → Agent

Control which users can interact with the agent.

### Agent → Tools / Functions

Control which functions the agent is allowed to call.

### Agent → Data Sources

Control which datasets or knowledge bases the agent can retrieve.

### Agent → External APIs

Control access to external systems like CRM, billing, etc.

---

# 2. Typical Agentic Architecture with Access Control

```
User
  │
  ▼
Authentication Layer (OAuth / JWT / API Key)
  │
  ▼
Authorization Layer (RBAC / ABAC / Policy Engine)
  │
  ▼
Agent Runtime
  │
  ├── Tool Access Controller
  │        ├── function_1()
  │        ├── function_2()
  │        └── restricted_function()
  │
  ├── Data Access Layer
  │        ├── vector DB
  │        ├── SQL DB
  │        └── private docs
  │
  ▼
LLM
```

The **authorization layer decides what the agent is allowed to do.**

---

# 3. Role-Based Access Control (RBAC)

Assign roles to users and restrict functions.

Example:

| Role     | Allowed Functions |
| -------- | ----------------- |
| customer | view_orders       |
| analyst  | query_reports     |
| admin    | modify_data       |

Example implementation:

```python
ROLE_PERMISSIONS = {
    "customer": ["view_orders"],
    "analyst": ["query_reports"],
    "admin": ["view_orders", "query_reports", "modify_data"]
}

def check_permission(role, function_name):
    return function_name in ROLE_PERMISSIONS.get(role, [])
```

Before the agent calls a tool:

```python
if not check_permission(user_role, tool_name):
    raise PermissionError("Access denied")
```

---

# 4. Attribute-Based Access Control (ABAC)

More flexible than RBAC.

Policies depend on:

* user attributes
* data attributes
* environment context

Example:

Policy:

```
Allow access if:
user.department == data.department
```

Python example:

```python
def allow_access(user, document):
    return user["department"] == document["department"]
```

This is useful for **RAG systems with sensitive documents**.

---

# 5. Function-Level Access Control for Tools

Each tool should enforce authorization.

Example tool registry:

```python
TOOLS = {
    "get_customer_data": {
        "roles": ["admin", "support"]
    },
    "create_ticket": {
        "roles": ["customer", "support"]
    }
}
```

Check access before calling:

```python
def authorize_tool(user_role, tool):
    allowed = TOOLS[tool]["roles"]
    return user_role in allowed
```

---

# 6. Data-Level Access Control in RAG

For vector databases you must implement **row-level security**.

Example metadata filtering.

Stored document:

```
{
  text: "...",
  department: "finance"
}
```

Query filter:

```python
vector_db.query(
    query="revenue report",
    filter={"department": user.department}
)
```

This prevents **data leakage between departments**.

---

# 7. Policy Engine (Recommended for Production)

Use a **policy engine** instead of hardcoding rules.

Common tools:

| Tool              | Purpose                     |
| ----------------- | --------------------------- |
| Open Policy Agent | policy-based access control |
| Casbin            | RBAC / ABAC authorization   |
| Auth0             | identity and authorization  |
| Keycloak          | identity + role management  |

Example with **Casbin**:

```python
import casbin

e = casbin.Enforcer("model.conf", "policy.csv")

if not e.enforce(user, resource, action):
    raise PermissionError("Access denied")
```

---

# 8. Tool Authorization Layer for Agents

Agent frameworks should include a **tool gateway**.

Example flow:

```
Agent requests tool call
        │
        ▼
Tool Authorization Gateway
        │
        ├── check role
        ├── check policy
        └── check data scope
        │
        ▼
Execute tool
```

This prevents **LLM hallucinated tool calls**.

---

# 9. Secure Tool Invocation Pattern

Never allow the LLM to directly execute functions.

Instead:

```
LLM → proposed tool call
        ↓
Validation Layer
        ↓
Authorization Layer
        ↓
Tool Execution
```

Example:

```python
tool_call = agent_output["tool"]

if authorize_tool(user_role, tool_call):
    execute_tool(tool_call)
else:
    return "Not authorized"
```

---

# 10. Additional Security Controls

Important protections for GenAI agents:

### Prompt injection defense

Validate prompts before allowing tool calls.

### Tool sandboxing

Execute tools in restricted environments.

### Audit logs

Record:

* tool calls
* user identity
* accessed data

### Rate limiting

Prevent abuse.

---

# 11. Production Architecture Example

```
User
 │
 ▼
Identity Provider (OAuth / JWT)
 │
 ▼
API Gateway
 │
 ▼
Policy Engine (OPA / Casbin)
 │
 ▼
Agent Runtime
 │
 ├── Tool Gateway
 │
 ├── Vector DB (with metadata filters)
 │
 └── External APIs
```

---

✅ **Best Practice Summary**

1. Authenticate users (JWT / OAuth)
2. Use RBAC or ABAC policies
3. Restrict tool execution
4. Filter RAG retrieval by metadata
5. Use a policy engine
6. Log every agent action

