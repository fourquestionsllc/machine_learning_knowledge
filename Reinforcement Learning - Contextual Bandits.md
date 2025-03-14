## 🎯 What is a Contextual Bandit? (Recap)
A **Contextual Bandit** is a reinforcement learning setting where:
- At each time step:
  1. The agent sees some **context \( x \)** (aka state/features).
  2. It chooses an **action \( a \)**.
  3. It receives a **reward \( r(x, a) \)** — **but only for the chosen action**, not others.
  4. There is **no state transition** or future impact — it's a single-shot decision problem.

### 🔁 The goal:
> Learn a **policy** \( \pi(x) \rightarrow a \) that selects actions to **maximize expected reward** for each context.

---

## 🧠 Types of Contextual Bandit Algorithms (High-Level)

There are several popular algorithms, each with its own training & inference approach:

| Algorithm | Model Type | Training Style | Inference Style |
|----------|-------------|----------------|-----------------|
| **ε-Greedy** | Regression/Neural Net | Supervised learning | Predict rewards, pick best |
| **Thompson Sampling** | Probabilistic | Bayesian updates | Sample reward estimate, pick best |
| **UCB** | Regression/Uncertainty | Confidence-based updates | Exploit + explore via confidence |
| **Policy Gradient (Bandit version)** | Probabilistic policy | Gradient ascent | Sample or argmax from policy |

Let’s now walk through each algorithm’s **training and inference process** in detail, especially the commonly used **model-based (supervised) approaches**.

---

## 🧠 **1. ε-Greedy Contextual Bandits (Model-based)**

### ✍️ **Training Algorithm**
You train a **separate reward model per action**, e.g., a regression or neural net.

#### Steps:
1. Initialize reward predictors \( Q_a(x) \) for each action \( a \in A \)
2. Loop over episodes/time steps:
   - Observe context \( x_t \)
   - **Action Selection** (ε-greedy):
     - With prob ε: **Random action**
     - With prob 1-ε: Choose action with highest predicted reward:  
       \( a_t = \arg\max_a Q_a(x_t) \)
   - Observe reward \( r_t \) for selected action \( a_t \)
   - **Update model \( Q_{a_t} \)** using supervised learning:
     - Train with sample \( (x_t, r_t) \)

> 🔎 Think of each model \( Q_a \) as learning: "If I take action `a` in context `x`, how much reward do I expect?"

---

### 🔮 **Inference Algorithm (After Training)**

```text
1. Observe new context x
2. Predict reward for each action: Q_a(x)
3. Choose action a = argmax Q_a(x)
```

> Simple and effective. It's just inference via **reward prediction per action**.

---

## 🧠 **2. Thompson Sampling (Bayesian Model)**

- Maintain a **probability distribution over model parameters or reward estimates**
- Use **posterior sampling** to choose actions

### ✍️ Training Algorithm
1. For each action \( a \), maintain a Bayesian model for \( P(r|x, a) \)
2. At each step:
   - Observe context \( x_t \)
   - **Sample reward estimate** \( \tilde{r}_a \sim P(r|x_t, a) \)
   - Select action \( a_t = \arg\max_a \tilde{r}_a \)
   - Observe reward \( r_t \)
   - **Update posterior** for action \( a_t \)

> Often implemented using **Bayesian linear regression** or **Bayesian neural nets**

### 🔮 Inference
- Sample reward prediction for each action
- Pick action with highest sampled reward

---

## 🧠 **3. Upper Confidence Bound (UCB)**

- Adds an **exploration bonus** based on **uncertainty or visit count**

### ✍️ Training Algorithm
1. For each action \( a \), learn mean reward \( \hat{r}_a(x) \) and uncertainty \( u_a(x) \)
2. At each step:
   - Observe context \( x \)
   - Select action:
     \[
     a = \arg\max_a \left[ \hat{r}_a(x) + \text{uncertainty bonus}_a(x) \right]
     \]
   - Update model for \( a \) with observed reward

> Uncertainty can be computed via **variance in reward predictions**, **count-based**, or **confidence bounds**

### 🔮 Inference
- Predict reward + uncertainty bonus
- Pick action with highest **upper confidence bound**

---

## 🧠 **4. Policy-Based (e.g., Policy Gradient Bandits)**

### ✍️ Training Algorithm
- Parameterize a policy: \( \pi(a | x; \theta) \)
- Train the policy directly using **REINFORCE**-style gradient updates

Steps:
1. Sample action from \( \pi(a | x) \)
2. Observe reward \( r \)
3. Update policy:
   \[
   \nabla_\theta J(\theta) = \nabla_\theta \log \pi(a | x) \cdot r
   \]

### 🔮 Inference
- Choose action:
  - Either **sample from** \( \pi(a|x) \) or **argmax** \( \pi(a|x) \)

---

## 📌 Summary: Algorithm Comparison (Training & Inference)

| Algorithm | Training Style | Inference |
|----------|------------------|------------|
| ε-Greedy Regression | Supervised reward prediction | Predict all rewards, pick best |
| Thompson Sampling | Bayesian posterior updates | Sample from reward model |
| UCB | Supervised + uncertainty modeling | Predict reward + uncertainty |
| Policy Gradient | Policy directly optimized | Sample or argmax from policy |

---

## 🛠️ Example (ε-Greedy Training Pseudocode)
```python
# For 3 actions (prices), each with its own model
models = [NeuralNet(), NeuralNet(), NeuralNet()]

for t in range(T):
    x = get_context()
    
    # ε-greedy
    if np.random.rand() < epsilon:
        a = np.random.randint(0, 3)
    else:
        preds = [models[i](x) for i in range(3)]
        a = np.argmax(preds)
    
    r = get_reward(x, a)
    models[a].train_on(x, r)  # Train only selected action’s model
```

---
---

### 🎯 What is a **Contextual Bandits Model**?

A **Contextual Bandit** (also called a **Multi-Armed Bandit with Context**) is a **simplified version of reinforcement learning** where:

- The agent observes **context** (features/state) at each time step.
- It chooses an **action (arm)** from a set of discrete actions.
- It receives a **reward** for the selected action **only**, not for the others.
- **No environment transition or next state is considered.**

> 🧠 It’s like Q-learning without state transitions — you make a decision and get immediate feedback, but you don’t care about future consequences.

---

### 📊 Real-World Examples
| Domain | Context (state) | Action | Reward |
|--------|------------------|--------|--------|
| News recommendation | User profile | Article shown | Click or not |
| E-commerce | User/session features | Product/price shown | Purchase made |
| Ads | Viewer/device/context | Ad shown | Click-through |

---

### 🧠 Contextual Bandit vs Full RL
| Feature               | Contextual Bandits              | Full RL (e.g., Q-Learning) |
|-----------------------|----------------------------------|-----------------------------|
| Future states         | ❌ No state transitions         | ✅ Yes, state transitions |
| Long-term planning    | ❌ Not considered               | ✅ Yes, long-term reward |
| Feedback              | ✅ Reward for chosen action only | ✅ Reward + next state |

---

### 🔢 Mathematical Setup

- Let $$\( x \in \mathcal{X} \)$$ be the **context vector**
- Action $$\( a \in \{1, 2, ..., K\} \)$$
- Reward $$\( r(a, x) \)$$ is **only observed for selected action**

The goal is to learn a **policy $$\( \pi(x) \rightarrow a \)$$** that chooses actions to **maximize expected reward**.

---

### ✅ Training Algorithms for Contextual Bandits

Several learning approaches exist:

#### 1. **ε-Greedy Linear Bandit**
- Fit a separate **regression model** for each action.
- At each step:
  - With probability ε, choose a random action (exploration)
  - With probability 1−ε, choose the action with the **highest predicted reward** (exploitation)

#### 2. **Thompson Sampling**
- Maintain a probabilistic belief about reward distributions.
- Sample parameters from posterior and choose action with highest sample reward.
- Update beliefs with observed reward.

#### 3. **Upper Confidence Bound (UCB)**
- Choose action \( a \) that maximizes:
  \[
  \hat{r}(a, x) + \text{uncertainty}(a)
  \]
- Encourages trying actions with **high uncertainty**, balances exploration & exploitation.

#### 4. **Policy Gradient Bandits**
- Train a policy \( \pi(a|x; \theta) \) via gradient ascent on expected reward.

#### 5. **Neural Contextual Bandits (Deep CB)**
- Use a **neural network** \( Q(x, a; \theta) \) to model reward prediction.
- Trained via MSE loss between predicted and observed rewards.
- Similar to one-step DQN (but no bootstrapping or state transitions).

---

### 📐 General Training Algorithm (Model-based Bandit):

```text
1. Initialize model(s) for each action
2. For each time step t:
   a. Observe context x_t
   b. Select action a_t using exploration strategy (ε-greedy, TS, UCB)
   c. Observe reward r_t
   d. Update model for action a_t using (x_t, r_t)
```

---

### 🔍 Inference Algorithm (After Training)

```text
1. Observe context x
2. Predict reward for each action a: r̂(x, a)
3. Select action with highest r̂(x, a)
```

For Neural Contextual Bandits, this is just:
```python
action = argmax(Q(x, a))  # for all a
```

---

### 📊 Summary Table

| Component        | Contextual Bandits |
|------------------|--------------------|
| Input            | Context / features |
| Output           | Discrete action    |
| Learning Signal  | Reward only for chosen action |
| Goal             | Learn a policy to maximize expected reward |
| Model Options    | Linear, Bayesian, UCB, NN |

---

### 💡 Bonus Tip:
Contextual Bandits are ideal when:
- Feedback is **immediate**
- **No long-term impact** of decisions
- Fast training + low complexity is desired

---

### **Contextual Bandits: What Are They?**

Contextual Bandits are a type of reinforcement learning problem where an agent chooses actions (arms of a bandit) based on **context** (additional information about the environment) to maximize rewards over time.

- **Bandits**: Refer to the multi-armed bandit problem, where each "arm" of a slot machine gives a different reward.
- **Contextual**: The agent gets some extra information (context) about the environment before choosing an arm.

### **How It Works**

1. **Context**: At each time step, the agent observes a **context** \(x\), which describes the situation.
2. **Action**: The agent chooses an action \(a\) (an arm of the bandit) based on the context \(x\).
3. **Reward**: The agent receives a reward \(r\), which depends on both the chosen action \(a\) and the context \(x\).
4. **Goal**: The agent learns a policy  \( $`\pi(x)`$ \)  to choose actions that maximize expected rewards over time.

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


---

