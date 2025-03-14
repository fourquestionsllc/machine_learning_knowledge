Great — let's dive into the **detailed training algorithm of Thompson Sampling for contextual bandits**, often referred to as **Bayesian Linear Regression Thompson Sampling** when used in a linear model setup.

---

## 🎯 **Contextual Bandit Problem Recap**

In a **contextual bandit** setting:
- At each timestep \( t \):
  1. You observe a **context vector** \( x_{t,a} \in \mathbb{R}^d \) for each arm \( a \in \{1, ..., K\} \)
  2. You choose an **arm \( a_t \)** based on the context
  3. You observe a **reward \( r_{t,a_t} \)** for the selected arm (not for others)

The goal is to **maximize cumulative reward** (or minimize regret).

---

## 🎲 Thompson Sampling (TS) for Contextual Bandits

TS maintains a **posterior distribution over the reward model parameters**, and **samples from it to make decisions**, balancing **exploration and exploitation naturally**.

### ➕ Common Assumptions:
- **Linear reward model:**  
  \( r_{t,a} = x_{t,a}^\top \theta_a + \epsilon_t \), where \( \epsilon_t \sim \mathcal{N}(0, \sigma^2) \)
- **Prior over parameters:** \( \theta_a \sim \mathcal{N}(\mu_{a,0}, \Sigma_{a,0}) \)

---

## 📚 Detailed Algorithm (Bayesian Linear Regression Version)

Let’s go step by step for **each arm \( a \)**:

### **Initialization:**
For each arm \( a \):
- Set prior: \( \mu_{a,0} = 0 \), \( \Sigma_{a,0} = \lambda I_d \) (Gaussian prior)
- Set \( A_a = \lambda I_d \) (covariance matrix inverse)
- Set \( b_a = 0_d \) (weighted rewards vector)

---

### **For each round \( t = 1, 2, ..., T \):**

1. **Observe context vectors \( x_{t,a} \in \mathbb{R}^d \)** for all arms \( a \)

2. **Sample model parameters for each arm:**
   - For arm \( a \), compute posterior:
     - \( \hat{\Sigma}_a = A_a^{-1} \)
     - \( \hat{\mu}_a = A_a^{-1} b_a \)
   - **Sample \( \tilde{\theta}_a \sim \mathcal{N}(\hat{\mu}_a, \hat{\Sigma}_a) \)**

3. **Choose arm \( a_t \)** that maximizes estimated reward:
   \[
   a_t = \arg\max_a x_{t,a}^\top \tilde{\theta}_a
   \]

4. **Observe reward \( r_t \)** for the selected arm \( a_t \)

5. **Update posterior for selected arm \( a_t \):**
   - \( A_{a_t} \leftarrow A_{a_t} + x_{t,a_t} x_{t,a_t}^\top \)
   - \( b_{a_t} \leftarrow b_{a_t} + r_t x_{t,a_t} \)

(Other arms remain unchanged.)

---

## 🧠 Intuition

- TS samples plausible model parameters from the posterior.
- Sometimes, it samples “optimistic” parameters → **exploration**.
- Over time, the posterior becomes sharper, and actions focus on the **best arm** → **exploitation**.

---

## 📌 Optional Enhancements:
- **Shared parameter model** (single \( \theta \) across arms) when arms have shared features.
- **Nonlinear models**: TS can also work with Bayesian Neural Networks or via bootstrapped approximations.
- **Thompson Sampling with Logistic Regression** — applicable for binary rewards, but more complex due to non-conjugacy.

---

## 📎 Summary of Key Variables

| Variable | Description |
|---------|-------------|
| $$\( x_{t,a} \)$$ | Context vector for arm \( a \) |
| $$\( A_a \)$$ | Covariance matrix of the posterior |
| $$\( b_a \)$$ | Weighted reward sum |
| $$\( \hat{\mu}_a \)$$ | Posterior mean |
| $$\( \tilde{\theta}_a \)$$ | Sampled parameters from posterior |


### Understanding Thompson Sampling for Contextual Bandits

**Thompson Sampling** is a method used to balance exploration (trying out less-known options) and exploitation (choosing the best-known option) when making decisions. For **contextual bandits**, it means making decisions based on the current context while learning which options (or "arms") are best over time.

---

### Step-by-Step Explanation with an Example

#### Problem Setup
Imagine you're running a **movie streaming service** and want to recommend movies to users. For each user, you want to maximize the chance that they click on a recommended movie (reward).

1. **Context:** Each user has some features, e.g., 
   - Age: 25
   - Preferred Genre: Action
   - Time of day: Evening

2. **Actions (Arms):** Different movies you can recommend, e.g., 
   - Movie A: Action
   - Movie B: Comedy
   - Movie C: Drama

3. **Reward:** Whether the user clicks on the movie or not (1 for a click, 0 for no click).

---

### How Thompson Sampling Works

1. **Start with Uncertainty (Prior):**
   - You don’t know which movie the user will prefer initially.
   - Assume a **probabilistic model** for each movie, e.g., a **Beta distribution** with parameters $$\( \alpha = 1 \) (successes)$$ and $$\( \beta = 1 \) (failures)$$. This represents your initial uncertainty about each movie's reward probability.

2. **Observe the Context:**
   - For the current user, their features (age, genre preference, etc.) are your context.

3. **Generate Reward Probabilities:**
   - For each movie (action), use the context and past rewards to update your model.
   - Sample a reward probability from each movie's Beta distribution. For example:
     - Movie A: Sampled reward probability = 0.7
     - Movie B: Sampled reward probability = 0.4
     - Movie C: Sampled reward probability = 0.6

4. **Choose the Action with the Highest Sampled Probability:**
   - In this case, recommend **Movie A** because it has the highest sampled reward probability (0.7).

5. **Update Beliefs (Posterior):**
   - After observing whether the user clicked or not:
     - If the user clicked on Movie A, increase $$\( \alpha \)$$ for Movie A.
     - If they didn’t click, increase $$\( \beta \)$$ for Movie A.
   - For example, if Movie A had $$\( \alpha = 1 \)$$ and $$\( \beta = 1 \)$$ before, and the user clicked, it becomes $$\( \alpha = 2 \)$$ and $$\( \beta = 1 \)$$.

6. **Repeat:**
   - Over time, this process learns the best recommendations for different contexts.

---

### Why is Thompson Sampling Effective?

- **Exploration:** By sampling probabilities, it sometimes chooses less-favored movies to gather more data.
- **Exploitation:** It mostly picks the movie with the highest estimated reward based on the current data.

---

### Example in Action

- **Day 1:** You recommend movies randomly and get these clicks:
  - Movie A: 3 clicks out of 5
  - Movie B: 1 click out of 5
  - Movie C: 2 clicks out of 5
  - Initial beliefs: Beta(4, 3), Beta(2, 5), Beta(3, 4)

- **Day 2:** For a 25-year-old who likes action, you sample reward probabilities:
  - Movie A: Sample = 0.65
  - Movie B: Sample = 0.35
  - Movie C: Sample = 0.55
  - You recommend **Movie A**.

- **Day 3 and Beyond:** As you collect more data, your recommendations get better for specific user contexts.

This process balances learning about new movies while also recommending ones likely to perform well.


---

