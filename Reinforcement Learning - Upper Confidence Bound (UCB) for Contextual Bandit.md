## 📚 **Overview: What is LinUCB?**

- **Goal**: Balance **exploitation** (choose arm with highest estimated reward) and **exploration** (choose uncertain arms to learn more).
- **Approach**: Estimate expected reward **+ a confidence interval** for each arm, then select the arm with the highest **upper confidence bound**.

### Reward Model:
We assume a **linear reward model**:

$$r_{t,a} = x_{t,a}^\top \theta_a + \epsilon, \quad \epsilon \sim \mathcal{N}(0, \sigma^2)$$

Each arm has a separate parameter vector $$\( \theta_a \in \mathbb{R}^d \)$$, or in a **shared model**, there is a global $$\( \theta \)$$.

---

## 📌 **Algorithm Details: LinUCB**

### 🔧 **Hyperparameter**
- $$\( \alpha > 0 \)$$: Exploration parameter (controls how much exploration you do)

---

## ✅ **Initialization (Training setup)**

For each arm $$\( a \in \{1, ..., K\} \)$$:
- $$\( A_a = I_d \in \mathbb{R}^{d \times d} \)$$: design matrix (feature covariance)
- $$\( b_a = 0_d \in \mathbb{R}^d \)$$: response vector (feature-weighted rewards)

---

## 🔁 **For each timestep $$\( t = 1, 2, ..., T \)$$:**

### 1. **Observe context vector** $$\( x_{t,a} \in \mathbb{R}^d \)$$ for each arm $$\( a \)$$

### 2. **Estimate model parameters** for each arm:
$$\hat{\theta}_a = A_a^{-1} b_a$$

### 3. **Compute UCB score for each arm:**
$$UCB_{t,a} = x_{t,a}^\top \hat{\theta}_a$$
+
$$+ \alpha \cdot \sqrt{x_{t,a}^\top A_a^{-1} x_{t,a}}$$

- The first term is the estimated reward (exploitation).
- The second term is the **confidence width** (exploration bonus).

### 4. **Select the arm with the highest UCB:**
$$a_t = \arg\max_a \text{UCB}_{t,a}$$

### 5. **Observe reward $$\( r_t \)$$** for chosen arm $$\( a_t \)$$

### 6. **Update statistics for selected arm $$\( a_t \)$$:**
$$A_{a_t} \leftarrow A_{a_t} + x_{t,a_t} x_{t,a_t}^\top$$

$$b_{a_t} \leftarrow b_{a_t} + r_t x_{t,a_t}$$

---

## 📎 Summary Table

| Step | Description |
|------|-------------|
| \( A_a \) | Feature covariance matrix |
| \( b_a \) | Weighted rewards |
| \( \hat{\theta}_a \) | Estimated reward model |
| \( \alpha \) | Exploration-exploitation balance |
| \( \text{UCB}_{t,a} \) | Reward + uncertainty |

---

## 🔍 **Inference Phase (Once Training is Done)**

Technically, LinUCB learns **online**, so it's always learning. But if you want to stop training and just **use the learned parameters** for inference:
- For each arm $$\( a \)$$, use $$\( \hat{\theta}_a = A_a^{-1} b_a \)$$
- For a new context $$\( x \)$$, you can either:
  - Choose **argmax $$\( x^\top \hat{\theta}_a \)$$** (pure exploitation)
  - Or keep using $$\( x^\top \hat{\theta}_a + \alpha \cdot \sqrt{x^\top A_a^{-1} x} \)$$ (continued exploration)

---

## ✍️ Optional: **Shared Linear Model across Arms**

Sometimes we assume a **single global \( \theta \)** instead of per-arm \( \theta_a \). Then:
- \( A = \sum x_{t,a_t} x_{t,a_t}^\top \)
- \( b = \sum r_{t} x_{t,a_t} \)
- Same procedure, just one set of \( A \), \( b \)

---

## 💡 Summary Comparison: LinUCB vs Thompson Sampling

| Feature | LinUCB | Thompson Sampling |
|--------|--------|-------------------|
| Decision Basis | Upper confidence bound | Posterior sampling |
| Exploration | Deterministic (via α) | Probabilistic |
| Posterior Model | Point estimate + uncertainty | Full distribution |
| Complexity | Simple | More Bayesian machinery |
