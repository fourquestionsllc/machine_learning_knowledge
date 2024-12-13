The **`mabwiser`** library is a Python package that simplifies Multi-Armed Bandit (MAB) implementations, including contextual bandit algorithms. Here's how to use it for price optimization where the context, actions, and rewards are as described.

---

### 1. **Setup**
- Install the `mabwiser` library if not already installed:
  ```bash
  pip install mabwiser
  ```

- Define your **context**, **actions**, and **rewards**:
  - **Context:** Product features, store information, historical sales, seasonal index.
  - **Actions:** Prices you are optimizing.
  - **Rewards:** Revenue generated for the chosen price.

---

### 2. **Steps to Implement Contextual Bandit**

#### a. **Prepare Data**
The dataset should include:
- Context features: e.g., `['product_id', 'store_id', 'seasonal_index', 'historical_sales']`
- Actions: e.g., a discrete list of price options.
- Rewards: Revenue corresponding to the action taken.

#### b. **Train and Use the Contextual Bandit**
`mabwiser` supports contextual algorithms like LinUCB and LinTS (Linear Upper Confidence Bound and Linear Thompson Sampling). These methods work well for contextual bandits.

---

### Example Code

Here’s an example of using `mabwiser` for price optimization:

```python
import pandas as pd
from mabwiser.mab import MAB, LearningPolicy, NeighborhoodPolicy

# Step 1: Load and Prepare Data
# Example: historical data containing contexts, actions (prices), and rewards (revenue)
data = pd.read_csv("pricing_data.csv")  # Replace with your dataset

contexts = data[['product_id', 'store_id', 'seasonal_index', 'historical_sales']].values
actions = data['price'].values
rewards = data['revenue'].values

# Step 2: Split Data into Training and Testing
train_size = int(0.8 * len(data))
contexts_train, contexts_test = contexts[:train_size], contexts[train_size:]
actions_train, actions_test = actions[:train_size], actions[train_size:]
rewards_train, rewards_test = rewards[:train_size], rewards[train_size:]

# Step 3: Initialize Contextual Bandit (LinUCB)
mab = MAB(
    arms=list(set(actions)),  # Unique price options
    learning_policy=LearningPolicy.LinUCB(alpha=0.1),  # Exploration-exploitation tradeoff
    neighborhood_policy=NeighborhoodPolicy.KNearest(k=3)  # Optional: Localize decision-making
)

# Step 4: Train the Model
mab.fit(contexts=contexts_train, actions=actions_train, rewards=rewards_train)

# Step 5: Predict the Best Action (Price) for New Contexts
predicted_prices = []
for context in contexts_test:
    predicted_price = mab.predict(context)
    predicted_prices.append(predicted_price)

# Step 6: Evaluate Model
from sklearn.metrics import mean_squared_error

actual_rewards = rewards_test
predicted_rewards = [data[data['price'] == price]['revenue'].mean() for price in predicted_prices]

mse = mean_squared_error(actual_rewards, predicted_rewards)
print(f"Mean Squared Error of Revenue Prediction: {mse}")

# Step 7: Deploy for Real-Time Pricing
# Use mab.predict(context) in real-time to suggest optimal prices for new contexts.
```

---

### 3. **Explanation of Key Steps**

#### a. **Learning Policy**
- `LinUCB`: Linear Upper Confidence Bound (UCB) algorithm estimates reward as a linear combination of context features.
- `LinTS`: Linear Thompson Sampling balances exploration and exploitation probabilistically.

#### b. **Neighborhood Policy**
- `KNearest`: Incorporates neighborhood learning to make decisions specific to localized contexts (e.g., store-wise).

#### c. **Real-Time Prediction**
Once trained, the `mab.predict(context)` method can suggest prices dynamically for new contexts.

---

### 4. **Deployment**
1. **Wrap the Model in an API**: Use frameworks like Flask or FastAPI to expose `mab.predict(context)` as an API endpoint.
2. **Real-Time Data Collection**: Store context, action, and reward data for continuous model updates.
3. **Periodic Retraining**: Regularly update the model with new data to refine pricing recommendations.

---

### 5. **Benefits**
- **Dynamic Pricing**: Adjusts based on real-time context.
- **Efficient Exploration**: Optimizes prices without requiring exhaustive experimentation.
- **Scalability**: Can handle large contexts and multiple products.

This implementation with `mabwiser` ensures that the pricing system adapts dynamically, maximizing revenue while balancing exploration and exploitation.
