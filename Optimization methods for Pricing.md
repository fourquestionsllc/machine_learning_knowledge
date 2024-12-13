There are several alternatives to **Thompson Sampling** and **Bayesian Optimization** for optimization, particularly in the context of pricing and decision-making. Here’s an overview of popular methods, their use cases, and libraries to implement them:

---

### **1. Epsilon-Greedy Algorithm**
#### **Description:**
- Balances exploration and exploitation by selecting a random action with a small probability (\(\epsilon\)) and the best-known action with a large probability (\(1-\epsilon\)).
- Simple and efficient for multi-armed bandit problems.

#### **Use Case:**
- When computational simplicity and ease of implementation are required.

#### **Implementation:**
```python
import numpy as np

def epsilon_greedy(prices, rewards, epsilon=0.1):
    n_prices = len(prices)
    counts = np.zeros(n_prices)  # Count of trials for each price
    total_rewards = np.zeros(n_prices)  # Total rewards for each price

    for t in range(1000):  # Number of rounds
        if np.random.rand() < epsilon:
            action = np.random.choice(n_prices)  # Explore
        else:
            action = np.argmax(total_rewards / (counts + 1e-10))  # Exploit

        reward = rewards[action]
        counts[action] += 1
        total_rewards[action] += reward

    return np.argmax(total_rewards / counts), total_rewards / counts

# Example usage
prices = [10, 15, 20]
true_rewards = [0.1, 0.08, 0.05]
optimal_price, avg_rewards = epsilon_greedy(prices, true_rewards)
print(f"Optimal Price: {prices[optimal_price]}, Average Rewards: {avg_rewards}")
```

---

### **2. Upper Confidence Bound (UCB)**
#### **Description:**
- UCB selects actions based on the principle of optimism in the face of uncertainty.
- Encourages exploration of less tried options by adding a confidence term to the estimated reward.

#### **Use Case:**
- Best suited for scenarios where the reward uncertainty decreases over time.

#### **Implementation:**
```python
import numpy as np

def ucb(prices, rewards, c=2):
    n_prices = len(prices)
    counts = np.zeros(n_prices)
    total_rewards = np.zeros(n_prices)

    for t in range(1000):  # Number of rounds
        if 0 in counts:
            action = np.argmin(counts)  # Explore untried actions first
        else:
            confidence = np.sqrt(c * np.log(t + 1) / counts)
            action = np.argmax(total_rewards / counts + confidence)  # UCB formula

        reward = rewards[action]
        counts[action] += 1
        total_rewards[action] += reward

    return np.argmax(total_rewards / counts), total_rewards / counts

# Example usage
prices = [10, 15, 20]
true_rewards = [0.1, 0.08, 0.05]
optimal_price, avg_rewards = ucb(prices, true_rewards)
print(f"Optimal Price: {prices[optimal_price]}, Average Rewards: {avg_rewards}")
```

---

### **3. Simulated Annealing**
#### **Description:**
- Probabilistic optimization algorithm inspired by the annealing process in metallurgy.
- Explores the solution space by accepting worse solutions with a probability that decreases over time.

#### **Use Case:**
- Global optimization problems where the function landscape has many local optima.

#### **Implementation:**
```python
from scipy.optimize import dual_annealing

# Simulated revenue function
def revenue_function(price):
    return -((price - 15) ** 2) + 100

# Simulated Annealing Optimization
result = dual_annealing(lambda x: -revenue_function(x[0]), bounds=[(5, 25)])
optimal_price = result.x[0]
print(f"Optimal Price: {optimal_price}")
```

---

### **4. Evolutionary Algorithms (e.g., Genetic Algorithms)**
#### **Description:**
- Population-based optimization inspired by natural selection.
- Evolves solutions over generations using mutation, crossover, and selection.

#### **Use Case:**
- Optimization problems with complex or non-convex search spaces.

#### **Implementation:**
```python
from deap import base, creator, tools, algorithms
import numpy as np

# Revenue function
def revenue_function(ind):
    price = ind[0]
    return -((price - 15) ** 2) + 100,

# Define GA components
creator.create("FitnessMax", base.Fitness, weights=(1.0,))
creator.create("Individual", list, fitness=creator.FitnessMax)
toolbox = base.Toolbox()
toolbox.register("attr_price", np.random.uniform, 5, 25)
toolbox.register("individual", tools.initRepeat, creator.Individual, toolbox.attr_price, n=1)
toolbox.register("population", tools.initRepeat, list, toolbox.individual)
toolbox.register("evaluate", revenue_function)
toolbox.register("mate", tools.cxBlend, alpha=0.5)
toolbox.register("mutate", tools.mutGaussian, mu=0, sigma=1, indpb=0.2)
toolbox.register("select", tools.selTournament, tournsize=3)

# Run Genetic Algorithm
population = toolbox.population(n=50)
result = algorithms.eaSimple(population, toolbox, cxpb=0.7, mutpb=0.2, ngen=50, verbose=False)
optimal_price = tools.selBest(population, k=1)[0][0]
print(f"Optimal Price: {optimal_price}")
```

---

### **5. Gradient-Free Optimization**
#### **Description:**
- Optimization methods like Nelder-Mead or Particle Swarm Optimization do not require gradient information.
- Suitable for black-box functions or noisy objective functions.

#### **Libraries:**
- `scipy.optimize` (e.g., Nelder-Mead)
- `pyswarms` for Particle Swarm Optimization.

#### **Example with Particle Swarm Optimization:**
```python
from pyswarms.single.global_best import GlobalBestPSO

# Revenue function
def revenue_function(x):
    return -((x[:, 0] - 15) ** 2) + 100

# Particle Swarm Optimization
bounds = (np.array([5]), np.array([25]))
optimizer = GlobalBestPSO(n_particles=10, dimensions=1, options={'c1': 1.5, 'c2': 1.5, 'w': 0.9}, bounds=bounds)
cost, pos = optimizer.optimize(revenue_function, iters=100)
print(f"Optimal Price: {pos[0]}")
```

---

### **Summary**
- **Epsilon-Greedy**: Simple, easy to implement.
- **UCB**: Effective for exploration-exploitation trade-offs.
- **Simulated Annealing**: Good for global optimization.
- **Genetic Algorithms**: Handles complex and non-convex problems.
- **Particle Swarm Optimization**: Fast, gradient-free global optimization.

Each method has its strengths depending on the problem complexity, the reward structure, and the need for computational efficiency.
