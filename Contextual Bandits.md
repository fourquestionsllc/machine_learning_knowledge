### **Contextual Bandits: What Are They?**

Contextual Bandits are a type of reinforcement learning problem where an agent chooses actions (arms of a bandit) based on **context** (additional information about the environment) to maximize rewards over time.

- **Bandits**: Refer to the multi-armed bandit problem, where each "arm" of a slot machine gives a different reward.
- **Contextual**: The agent gets some extra information (context) about the environment before choosing an arm.

### **How It Works**

1. **Context**: At each time step, the agent observes a **context** \(x\), which describes the situation.
2. **Action**: The agent chooses an action \(a\) (an arm of the bandit) based on the context \(x\).
3. **Reward**: The agent receives a reward \(r\), which depends on both the chosen action \(a\) and the context \(x\).
4. **Goal**: The agent learns a policy $` \( \pi(x) \) `$ to choose actions that maximize expected rewards over time.

---

### **Key Concepts**

1. **Exploration vs. Exploitation**:  
   - The agent must balance:
     - **Exploration**: Trying new actions to learn their rewards.
     - **Exploitation**: Choosing the best-known action to maximize reward.

2. **Context Features**:  
   - The context \(x\) could be a vector of features, such as user demographics in a recommendation system or the weather in a delivery system.

3. **Reward Model**:  
   - The agent learns a reward function \(r = f(x, a)\) that predicts rewards for each action given the context.

4. **Regret**:  
   - Measures how much reward is lost by not always choosing the best action.

---

### **Example: Personalized Ad Recommendations**

#### Scenario
- **Context**: A user visits a webpage, and their features (e.g., age, location, interests) form the context.
- **Actions**: The ads the platform can show.
- **Reward**: Whether the user clicks on the ad (click = 1, no click = 0).

The agent learns over time which ads perform best for different types of users.

---

### **Code Example: Contextual Bandits**

Here’s a simple implementation:

```python
import numpy as np
from sklearn.linear_model import LinearRegression

# Simulated environment
class AdEnvironment:
    def __init__(self):
        self.true_coefficients = {
            0: [1.5, 2.0],  # Ad 0
            1: [-1.0, 1.0], # Ad 1
            2: [0.5, -1.5]  # Ad 2
        }
    
    def get_reward(self, context, action):
        noise = np.random.normal(0, 0.1)
        reward = np.dot(context, self.true_coefficients[action]) + noise
        return reward

# Contextual Bandit Agent
class ContextualBandit:
    def __init__(self, n_actions, n_features):
        self.n_actions = n_actions
        self.models = [LinearRegression() for _ in range(n_actions)]
        self.data = {a: {"X": [], "y": []} for a in range(n_actions)}
    
    def select_action(self, context, epsilon=0.1):
        if np.random.rand() < epsilon:
            return np.random.randint(self.n_actions)  # Explore
        else:
            # Predict reward for each action
            predictions = [
                self.models[a].predict(context.reshape(1, -1))[0]
                if len(self.data[a]["X"]) > 0 else 0
                for a in range(self.n_actions)
            ]
            return np.argmax(predictions)  # Exploit
    
    def update(self, context, action, reward):
        self.data[action]["X"].append(context)
        self.data[action]["y"].append(reward)
        # Train model for the action
        X = np.array(self.data[action]["X"])
        y = np.array(self.data[action]["y"])
        self.models[action].fit(X, y)

# Main simulation
n_actions = 3
n_features = 2
n_rounds = 1000

env = AdEnvironment()
agent = ContextualBandit(n_actions, n_features)

total_reward = 0
for t in range(n_rounds):
    # Generate random context
    context = np.random.uniform(-1, 1, size=n_features)
    
    # Agent selects action
    action = agent.select_action(context)
    
    # Get reward from the environment
    reward = env.get_reward(context, action)
    
    # Update the agent
    agent.update(context, action, reward)
    
    # Track total reward
    total_reward += reward

print(f"Total Reward: {total_reward:.2f}")
```

---

### **How the Example Works**

1. **AdEnvironment**:
   - Simulates the reward for each ad based on a linear function of the context.

2. **ContextualBandit**:
   - Maintains a separate linear regression model for each action.
   - Uses ε-greedy to explore and exploit.

3. **Simulation**:
   - Runs multiple rounds where the agent interacts with the environment, learns, and improves.

---

### **Applications**
- Personalized recommendations (e.g., ads, news).
- Healthcare (e.g., choosing treatments based on patient features).
- Financial services (e.g., deciding loan offers based on applicant data).

