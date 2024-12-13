### **Thompson Sampling and Bayesian Optimization for Pricing and Optimization**

#### **1. Thompson Sampling**
Thompson Sampling is a probabilistic algorithm used for decision-making under uncertainty, particularly for multi-armed bandit problems. It models the reward distribution for each action and selects actions based on samples drawn from the posterior distribution. It balances exploration and exploitation efficiently.

In pricing, Thompson Sampling can be used to dynamically adjust prices based on observed rewards (e.g., revenue or profit), balancing the need to explore new pricing strategies and exploit known successful ones.

#### **2. Bayesian Optimization**
Bayesian Optimization is a probabilistic model-based optimization technique designed for black-box functions that are expensive to evaluate. It uses a surrogate model (usually a Gaussian Process) to model the objective function and selects the next point to evaluate using an acquisition function (e.g., Expected Improvement).

In pricing, Bayesian Optimization can help determine the optimal price point by modeling the relationship between price and revenue/profit, even with limited data.

---

### **Libraries for Implementation**
1. **Thompson Sampling:** 
   - `MABWiser`: A Python library for multi-armed bandits.
2. **Bayesian Optimization:**
   - `GPyOpt`: A library for Bayesian optimization using Gaussian processes.
   - `scikit-optimize`: Lightweight library for Bayesian optimization.
   - `BoTorch`: PyTorch-based Bayesian optimization framework.

---

### **Example: Thompson Sampling for Pricing**

```python
import numpy as np
from mabwiser.mab import MAB, Thompsonsampling

# Simulated data
prices = [10, 15, 20]  # Different price levels
true_conversion_rates = [0.1, 0.08, 0.05]  # True probabilities of purchase
n_trials = 1000

# Simulate customer interactions
np.random.seed(42)
rewards = np.zeros((n_trials, len(prices)))
for t in range(n_trials):
    rewards[t] = np.random.binomial(1, true_conversion_rates)

# Initialize Thompson Sampling
mab = MAB(arms=prices, learning_policy=Thompsonsampling())
mab.fit(decisions=np.repeat(prices, n_trials // len(prices)), rewards=rewards.flatten())

# Simulate pricing decisions
selected_prices = []
for t in range(n_trials):
    selected_price = mab.predict()  # Choose a price
    selected_prices.append(selected_price)

# Analyze results
print(f"Selected Prices: {np.unique(selected_prices, return_counts=True)}")
```

---

### **Example: Bayesian Optimization for Pricing**

```python
import numpy as np
from skopt import gp_minimize
from skopt.space import Real
import matplotlib.pyplot as plt

# Simulated revenue function
def revenue_function(price):
    return -((price - 15) ** 2) + 100  # Revenue is maximized at price = 15

# Define search space
space = [Real(5.0, 25.0, name="price")]

# Bayesian Optimization
res = gp_minimize(
    func=lambda x: -revenue_function(x[0]),  # Minimize negative revenue
    dimensions=space,
    n_calls=20,  # Number of evaluations
    random_state=42
)

# Results
optimal_price = res.x[0]
print(f"Optimal Price: {optimal_price}")

# Plot the optimization process
plt.plot(res.func_vals, marker="o")
plt.xlabel("Iteration")
plt.ylabel("Negative Revenue")
plt.title("Optimization Process")
plt.show()
```

---

### **Key Differences and Applications**
- **Thompson Sampling** is effective for dynamic pricing when feedback (rewards) is observed iteratively over time.
- **Bayesian Optimization** is better suited for static price optimization when evaluating a price is expensive or slow.

Both methods provide powerful tools for solving pricing and optimization challenges efficiently!
