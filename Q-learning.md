### 🧠 What is **Q-Learning**?

**Q-Learning** is a type of **model-free reinforcement learning algorithm** used to learn an optimal **action-selection policy** for an agent interacting with an environment.

- The goal is to **learn a function** called the **Q-function** or **action-value function**, denoted as **Q(s, a)**, which estimates the **expected cumulative reward** (called "return") of taking an action `a` in a state `s`, and then following the optimal policy thereafter.

- It doesn’t need a model of the environment (like transition probabilities), which is why it's **model-free**.

---

### 💡 Core Idea

The **Q-function** is updated iteratively using the **Bellman Equation**:

$$Q(s, a) \leftarrow Q(s, a) + \alpha \left[ r + \gamma \cdot \max_{a'} Q(s', a') - Q(s, a) \right]$$

Where:
- \( s \): current state  
- \( a \): action taken  
- \( r \): reward received after taking action \( a \)  
- \( s' \): next state after taking action  
- \( \alpha \): learning rate (how fast Q-values are updated)  
- \( \gamma \): discount factor (how much future rewards are valued)

---

### 🏋️‍♂️ Q-Learning Training Algorithm (Step-by-step)

```text
1. Initialize Q(s, a) arbitrarily for all states and actions (e.g., Q-table with zeros).
2. Repeat for each episode (or until convergence):
    a. Initialize the starting state s.
    b. Repeat for each step in the episode:
        i.   Choose action a using an exploration strategy (e.g., ε-greedy).
        ii.  Take action a → observe reward r and next state s'.
        iii. Update Q(s, a) using:
             Q(s, a) ← Q(s, a) + α [r + γ * max_a' Q(s', a') - Q(s, a)]
        iv.  Set s ← s'.
    c. (Optional) Reduce ε over time to reduce exploration.
```

> **ε-greedy strategy**: With probability ε choose a random action (exploration), otherwise choose the best action from Q-table (exploitation).

---

### 🔍 Q-Learning Inference (Using the Learned Policy)

Once training is complete, inference is simple:

```text
For a given state s:
    Choose the action a = argmax_a Q(s, a)
    (i.e., pick the action with the highest Q-value in that state)
```

This defines the **optimal policy**:  
\[
\pi^*(s) = \arg\max_a Q(s, a)
\]

---

### 📌 Summary
| Component        | Description |
|------------------|-------------|
| Q(s, a)          | Expected return of taking action `a` in state `s` |
| Learning Goal    | Find Q* such that following argmax(Q*) is optimal |
| Training         | Update Q-values using Bellman update rule |
| Inference        | Pick action with highest Q(s, a) for current state |
