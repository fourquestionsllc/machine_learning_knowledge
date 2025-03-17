### 🧠 What is **DQN (Deep Q-Network)?**

**DQN** is a powerful reinforcement learning algorithm that combines:
- **Q-Learning** (model-free, off-policy RL)
- **Deep Neural Networks** (to approximate Q-values)

> Instead of using a Q-table (which doesn’t scale to large or continuous state spaces), DQN uses a **neural network to approximate Q(s, a)**.

It was first introduced by **DeepMind** and made famous by its success on **Atari games**.

---

### 📐 Core Idea of DQN

- Use a **neural network** \( Q(s, a; \theta) \) to approximate the Q-values.
- Learn the weights \( \theta \) of the network by minimizing a **loss function** based on the **Q-learning update rule**.

---

### 🧮 DQN Loss Function

$$\mathcal{L}(\theta) = \left( r + \gamma \cdot \max_{a'} Q(s', a'; \theta^-) - Q(s, a; \theta) \right)^2$$

Where:
- \( \theta \): current network weights
- \( \theta^- \): **target network weights** (a periodically updated copy of the main network)
- \( Q(s, a; \theta) \): predicted Q-value from the network
- $$\( r + \gamma \cdot \max Q(s', a') \)$$: target Q-value

---

### ⚙️ Training Algorithm for DQN

```text
1. Initialize:
    - Replay buffer D (for experience replay)
    - Q-network with random weights θ
    - Target network Q_target with weights θ⁻ = θ

2. For each episode:
    a. Initialize environment, get initial state s
    b. Repeat until episode ends:
        i.   Choose action a using ε-greedy policy from Q(s, a; θ)
        ii.  Take action a, observe reward r and next state s'
        iii. Store (s, a, r, s') in replay buffer D
        iv.  Sample a mini-batch of transitions from D
        v.   For each transition (s, a, r, s'):
                - Compute target: 
                  y = r + γ * max_a' Q_target(s', a'; θ⁻)
                - Compute loss: 
                  L = (y - Q(s, a; θ))²
        vi.  Perform gradient descent on L to update θ
        vii. Periodically update target network: θ⁻ ← θ
```

### 🧠 Key Techniques in DQN

| Technique              | Purpose |
|------------------------|--------|
| **Experience Replay**  | Store past experiences and sample randomly to break correlation and stabilize learning. |
| **Target Network**     | Use a separate, slowly-updated Q-network to compute target values for more stable learning. |
| **ε-Greedy Exploration** | Balance exploration and exploitation. |

---

### 🔍 Inference with DQN

After training, inference is simple:

```text
Given a state s:
    Choose action a = argmax_a Q(s, a; θ)
```

So the agent just passes the current state into the trained Q-network and selects the action with the highest predicted Q-value.

---

### 📊 Summary Table

| Component               | DQN Model |
|------------------------|------------------------------|
| Type                   | Model-free, off-policy RL |
| Q-function             | Approximated with neural network |
| Training goal          | Minimize difference between predicted and target Q-values |
| Replay Buffer          | Helps reduce data correlation |
| Target Network         | Stabilizes learning |
| Inference              | `a = argmax_a Q(s, a; θ)` |

---

### 🆚 DQN vs Q-Learning Recap

| Aspect            | Q-Learning         | DQN                         |
|-------------------|-------------------|-----------------------------|
| State space       | Small/discrete     | Large/continuous            |
| Q-function        | Table              | Neural Network              |
| Stability tricks  | None               | Experience replay + target net |
