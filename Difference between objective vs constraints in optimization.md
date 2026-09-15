If you're asking about **optimization models** and the difference between an **objective function** and a **constraint**, the distinction is:

### Objective function vs. constraint

| Concept                | Purpose                                   | Example          |
| ---------------------- | ----------------------------------------- | ---------------- |
| **Objective function** | What you want to **maximize or minimize** | Minimize cost    |
| **Constraint**         | What the solution **must satisfy**        | Budget ≤ $10,000 |

For example, suppose you're optimizing delivery routes.

**Objective:**

$$
\min \sum_i cost_i
$$

You want to **minimize total delivery cost**.

**Constraints:**

$$
\sum_i cost_i \leq 10,000
$$

$$
delivery_i \leq vehicleCapacity
$$

$$
x_i \in \{0,1\}
$$

These define what solutions are **allowed**.

### Simple way to remember

```text
Optimization model

      What do I WANT?
             ↓
      Objective Function
      "Minimize cost"
             +
      What MUST I obey?
             ↓
         Constraints
      "Budget <= $10K"
```

### Example in Python

Using an optimization library such as OR-Tools:

```python
from ortools.linear_solver import pywraplp

solver = pywraplp.Solver.CreateSolver("SCIP")

x = solver.IntVar(0, 10, "x")
y = solver.IntVar(0, 10, "y")

# Constraint
solver.Add(x + y <= 10)

# Objective
solver.Maximize(3 * x + 2 * y)

solver.Solve()

print(x.solution_value())
print(y.solution_value())
```

Here:

```text
x + y <= 10
     ↑
 Constraint

3x + 2y
   ↑
 Objective
```

**In an interview:**

> The **objective function defines what we optimize**, while **constraints define the feasible solution space**. The optimizer searches for the best solution that satisfies all constraints.
