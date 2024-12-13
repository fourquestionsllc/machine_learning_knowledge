# Contextual Bandits: How It Works

Contextual Bandits are a type of reinforcement learning problem where an agent chooses actions (arms of a bandit) based on **context** (additional information about the environment) to maximize rewards over time.

---

## Core Steps of the Contextual Bandits Learning Algorithm

1. **Observe Context ($`x_t`$)**:  
   At each time step $`t`$, the agent observes a **context** $`x_t \in \mathbb{R}^d`$, a vector representing information about the environment.  
   - Example: For an ad system, the context could be user features like age and location.

2. **Select Action ($`a_t`$)**:  
   The agent uses a policy $`\pi(x_t)`$ to select an action $`a_t`$ from a set of available actions $`\mathcal{A}`$.  
   - Example: The agent decides which ad to show based on user features.

3. **Receive Reward ($`r_t`$)**:  
   After taking the action $`a_t`$, the agent receives a reward $`r_t \in \mathbb{R}`$, which depends on both the chosen action and the context.  
   - Example: Reward could be a binary click (1 for clicked, 0 for not clicked).

4. **Update the Policy**:  
   Using the observed reward $`r_t`$, the agent updates its policy or model to improve future decisions.  
   - This involves learning the relationship between context, actions, and rewards.

---

## Learning Techniques

### 1. Linear Models (Simplest Case)
   Assume that the reward $`r_t`$ is a linear function of the context for each action:

   ```math
   r_t = x_t^T \theta_a + \epsilon
   ```

   - $`x_t`$: Context vector.
   - $`\theta_a`$: Coefficients (weights) for action $`a`$ that need to be learned.
   - $`\epsilon`$: Noise.

   Use methods like **linear regression** to estimate $`\theta_a`$ for each action.

---

### 2. Exploration Strategies
   Balancing exploration (trying less-known actions) and exploitation (choosing the best-known action) is critical:
   - **Epsilon-Greedy**: Randomly explore with probability $`\epsilon`$, otherwise exploit the best action.
   - **Upper Confidence Bound (UCB)**: Choose actions that maximize the estimated reward plus an exploration bonus:

     ```math
     a_t = \arg\max_a \left( \hat{r}_a + \sqrt{\frac{\beta}{n_a}} \right)
     ```

     - $`\hat{r}_a`$: Estimated reward for action $`a`$.
     - $`n_a`$: Number of times action $`a`$ has been tried.
     - $`\beta`$: Controls exploration.

---

### 3. Model-Free Approaches (Neural Networks)
   - Use neural networks to approximate the reward function $`f(x, a)`$.
   - Train using gradient-based optimization methods with batches of past experiences.

---

## Algorithm Pseudocode

Here’s a simplified pseudocode for a basic Contextual Bandit algorithm:

1. **Initialize**:
   - $`\theta_a`$: Parameters for each action $`a \in \mathcal{A}`$.
   - Exploration parameter $`\epsilon`$.

2. **For each time step $`t`$**:
   1. **Observe context**: $`x_t`$.
   2. **Select action**:
      - With probability $`\epsilon`$, choose a random action (exploration).
      - Otherwise, choose:
        ```math
        a_t = \arg\max_a (x_t^T \theta_a)
        ```
        (exploitation).
   3. **Receive reward**: $`r_t`$ after taking action $`a_t`$.
   4. **Update model**:
      - Update $`\theta_{a_t}`$ using the reward $`r_t`$ and context $`x_t`$ (e.g., using regression or gradient descent).

3. Repeat.

---

## Example: Learning Ad Preferences

### Scenario
- **Context**: User features (e.g., age, location).
- **Actions**: Ads (e.g., Ad A, Ad B, Ad C).
- **Reward**: Whether the user clicks the ad (1 for click, 0 for no click).

### Learning Steps
1. Initialize weights ($`\theta_a`$) for each ad.
2. For each user:
   - Observe context (e.g., `[25 years old, New York]`).
   - Use $`\epsilon`$-greedy to select an ad:
     - **Explore**: Randomly select an ad.
     - **Exploit**: Choose the ad with the highest predicted click rate.
   - Receive reward (click or no click).
   - Update weights for the chosen ad based on the observed reward.

---

## Practical Example

In practice, algorithms like LinUCB (Linear Upper Confidence Bound) or neural network-based models are used for real-world problems like ad recommendation systems.
