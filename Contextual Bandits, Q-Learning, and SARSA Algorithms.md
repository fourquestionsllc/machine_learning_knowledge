### **Contextual Bandits, Q-Learning, and SARSA Algorithms**

#### **1. Contextual Bandits**
Contextual Bandits are a variant of multi-armed bandits where decisions (actions) are made based on some context (state). In pricing, the context could include features like product demand, competitor prices, and seasonality. The goal is to select the best action (price) for each context to maximize rewards (e.g., revenue or profit).

#### **2. Q-Learning Algorithm**
Q-Learning is a model-free reinforcement learning algorithm that estimates the value (Q-value) of taking an action in a given state. It updates the Q-value using the Bellman equation:

$
\[
Q(s, a) \leftarrow Q(s, a) + \alpha [r + \gamma \max_{a'} Q(s', a') - Q(s, a)]
\]

- \(s, a\): Current state and action.
- \(r\): Reward received.
- \(\alpha\): Learning rate.
- \(\gamma\): Discount factor.
- \(s', a'\): Next state and action.
- $

#### **3. SARSA Algorithm**
SARSA (State-Action-Reward-State-Action) is another model-free RL algorithm. Unlike Q-Learning, SARSA updates the Q-value based on the actual action taken in the next state:

\[
Q(s, a) \leftarrow Q(s, a) + \alpha [r + \gamma Q(s', a') - Q(s, a)]
\]

- SARSA is on-policy, meaning it considers the policy used to generate actions during training.
- Q-Learning is off-policy, meaning it optimizes a target policy independent of the policy used to generate actions.

---

### **Using Contextual Bandits for Price Optimization**

In pricing, Contextual Bandits can determine the optimal price for a product based on contextual features. Q-Learning or SARSA can be applied if the problem is framed as a sequential decision-making task where each action impacts future rewards (e.g., long-term customer retention).

---

### **Libraries for Implementation**
1. **Contextual Bandits:**
   - `Vowpal Wabbit` (VW): Efficient for large-scale bandit problems.
   - `MABWiser`: Lightweight library for multi-armed bandits.
   - `Contextual`: R-based library for contextual bandits.
   
2. **Q-Learning and SARSA:**
   - `gym` and `stable-baselines3` (Python): RL environments and pre-built algorithms.
   - `TF-Agents`: TensorFlow-based RL library.
   - `PyTorch RL`: Custom implementation with PyTorch.

---

### **Example: Contextual Bandit for Pricing**

```python
import numpy as np
from sklearn.linear_model import LinearRegression

# Simulate environment
n_products = 3
n_contexts = 1000
context_features = np.random.rand(n_contexts, 5)  # Context: [demand, competitor_price, ...]
rewards = np.random.rand(n_contexts, n_products)  # Reward: Revenue for each product

# Bandit Model
class ContextualBandit:
    def __init__(self, n_products, n_features):
        self.n_products = n_products
        self.models = [LinearRegression() for _ in range(n_products)]
        self.data = {i: ([], []) for i in range(n_products)}  # Store (contexts, rewards) for each product

    def predict(self, context):
        predictions = [model.predict(context.reshape(1, -1))[0] if len(self.data[i][0]) > 0 else 0
                       for i, model in enumerate(self.models)]
        return np.argmax(predictions)  # Select the product with the highest predicted reward

    def update(self, context, product, reward):
        self.data[product][0].append(context)
        self.data[product][1].append(reward)
        X, y = np.array(self.data[product][0]), np.array(self.data[product][1])
        self.models[product].fit(X, y)

# Train bandit
bandit = ContextualBandit(n_products, context_features.shape[1])
total_reward = 0

for i in range(n_contexts):
    context = context_features[i]
    product = bandit.predict(context)
    reward = rewards[i, product]
    bandit.update(context, product, reward)
    total_reward += reward

print(f"Total Reward: {total_reward}")
```

---

### **Example: Q-Learning for Pricing**

```python
import numpy as np
import gym
from collections import defaultdict

# Environment: Simple pricing environment
class PricingEnv(gym.Env):
    def __init__(self):
        self.action_space = gym.spaces.Discrete(3)  # Prices: Low, Medium, High
        self.observation_space = gym.spaces.Box(low=0, high=1, shape=(3,), dtype=np.float32)  # [demand, competitor_price, seasonality]
        self.state = None

    def reset(self):
        self.state = np.random.rand(3)
        return self.state

    def step(self, action):
        price = [1.0, 1.5, 2.0][action]  # Price levels
        demand = self.state[0] * (2.5 - price)  # Simulated demand
        reward = price * demand
        self.state = np.random.rand(3)
        done = False
        return self.state, reward, done, {}

# Q-Learning Implementation
env = PricingEnv()
q_table = defaultdict(lambda: np.zeros(env.action_space.n))
alpha, gamma, epsilon = 0.1, 0.9, 0.1

for episode in range(1000):
    state = env.reset()
    total_reward = 0
    for t in range(100):
        if np.random.rand() < epsilon:  # Exploration
            action = env.action_space.sample()
        else:  # Exploitation
            action = np.argmax(q_table[tuple(state)])

        next_state, reward, done, _ = env.step(action)
        total_reward += reward

        # Q-Value Update
        best_next_action = np.argmax(q_table[tuple(next_state)])
        q_table[tuple(state)][action] += alpha * (
            reward + gamma * q_table[tuple(next_state)][best_next_action] - q_table[tuple(state)][action]
        )

        state = next_state
        if done:
            break

    print(f"Episode {episode}: Total Reward = {total_reward}")
```

---

### **When to Use Q-Learning, SARSA, or Contextual Bandits**
- **Contextual Bandits:** Ideal for one-shot pricing optimization with a given context.\n- **Q-Learning/SARSA:** Use for sequential pricing where current prices affect future rewards (e.g., customer loyalty).\n\nThese algorithms, combined with libraries like `Vowpal Wabbit`, `gym`, and `stable-baselines3`, empower effective pricing strategies.
