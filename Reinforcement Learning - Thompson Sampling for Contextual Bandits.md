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
   - For example, if Movie A had $$\( \alpha = 1 \)$$ and $$\( \beta = 1 \)$$ before, and the user clicked, it becomes $$\( \alpha = 2 \)$$ and $$\( \beta = 1 \)￥￥.

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
