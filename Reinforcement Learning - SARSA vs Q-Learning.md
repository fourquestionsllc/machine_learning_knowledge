### 🧠 What is **SARSA**?

**SARSA** stands for:  
> **S**tate - **A**ction - **R**eward - **S**tate - **A**ction

It's a **model-free, on-policy reinforcement learning algorithm** used to learn a policy that maximizes expected rewards. Like Q-learning, it tries to estimate the **action-value function Q(s, a)**, but it **learns from the actions it actually takes**, not from the optimal ones (as in Q-learning).

---

### ✅ Key Difference: SARSA vs Q-Learning

|               | **SARSA (On-Policy)**                            | **Q-Learning (Off-Policy)**                           |
|---------------|--------------------------------------------------|-------------------------------------------------------|
| Learning from | The **action you actually take** (current policy) | The **best possible action** (greedy action)          |
| Update rule   | Uses **Q(s', a')**                                | Uses **max_a Q(s', a)**                               |

---

### 📐 SARSA Update Rule (Bellman equation):

\[
Q(s, a) \leftarrow Q(s, a) + \alpha \left[ r + \gamma \cdot Q(s', a') - Q(s, a) \right]
\]

Where:
- \(s\): current state  
- \(a\): action taken  
- \(r\): reward received  
- \(s'\): next state  
- \(a'\): next action (chosen by **same policy**, e.g., ε-greedy)  
- \(\alpha\): learning rate  
- \(\gamma\): discount factor

---

### 🏋️‍♂️ SARSA Training Algorithm (Step-by-step)

```text
1. Initialize Q(s, a) arbitrarily (e.g., all zeros)
2. Repeat for each episode:
    a. Initialize starting state s
    b. Choose action a using policy derived from Q (e.g., ε-greedy)
    c. Repeat until episode ends:
        i.   Take action a, observe reward r and next state s'
        ii.  Choose next action a' from s' (using ε-greedy again)
        iii. Update Q(s, a) using:
             Q(s, a) ← Q(s, a) + α [r + γ * Q(s', a') - Q(s, a)]
        iv.  Set s ← s', a ← a'
```

> Notice: In SARSA, you **pick the next action a' before updating** the Q-value — because the learning is based on the actual path you follow under your policy.

---

### 🔍 Inference (Using the Learned Policy)

Once training is done:

```text
For a given state s:
    Choose action a = argmax_a Q(s, a)
```

This is the **exploitation phase**, where you use the Q-table to follow the best-known policy.

---

### 📌 Summary Table:

| Concept        | Description |
|----------------|-------------|
| Q(s, a)         | Expected return from state `s` and action `a` |
| Learning Type  | **On-policy**: learns from the current policy's behavior |
| Update Formula | `Q(s,a) ← Q(s,a) + α [r + γ * Q(s’,a’) - Q(s,a)]` |
| Difference     | Uses the action **actually chosen** in next state, not necessarily the best |

---

### 🆚 SARSA vs Q-Learning — Analogy:

Imagine driving through a city:
- **Q-Learning**: Always assumes you'll drive **perfectly from next step onward**.
- **SARSA**: Knows you **might still make suboptimal turns**, so it learns from **what you actually do**.

---

Would you like to see a **SARSA code example in Python**, or how SARSA performs differently than Q-learning in certain environments (e.g., cliff walking)?
