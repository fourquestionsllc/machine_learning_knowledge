### Built-in Functions for Q-Learning in Gym

OpenAI Gym provides an environment for reinforcement learning but does not include a built-in Q-learning implementation. Instead, you can use Gym to:
- **Interact with the environment** (e.g., get actions, states, and rewards).
- **Simulate episodes** for an agent to learn and update its Q-values.

For Q-learning, you typically need to implement:
1. **State space representation.**
2. **Action space interaction.**
3. **Reward collection and Q-value updates.**
4. **Policy selection (e.g., epsilon-greedy).**

---

### Example: Q-Learning with OpenAI Gym

Let's implement Q-learning for the **FrozenLake** environment. This environment is a grid world where the agent tries to reach a goal while avoiding holes.

---

#### Step-by-Step Implementation

```python
import numpy as np
import gym

# Create the FrozenLake environment
env = gym.make("FrozenLake-v1", is_slippery=True)  # is_slippery adds randomness
env.reset()

# Initialize Q-table
state_space = env.observation_space.n  # Number of states
action_space = env.action_space.n  # Number of actions
Q_table = np.zeros((state_space, action_space))

# Hyperparameters
learning_rate = 0.8
discount_factor = 0.95
epsilon = 1.0  # Exploration probability
epsilon_decay = 0.995
min_epsilon = 0.01
episodes = 10000
max_steps_per_episode = 100

# Q-Learning loop
for episode in range(episodes):
    state = env.reset()[0]
    done = False
    for step in range(max_steps_per_episode):
        # Epsilon-greedy policy
        if np.random.uniform(0, 1) < epsilon:
            action = env.action_space.sample()  # Explore
        else:
            action = np.argmax(Q_table[state, :])  # Exploit

        # Take action, observe next state and reward
        next_state, reward, done, _, _ = env.step(action)

        # Update Q-value using the Q-learning formula
        old_value = Q_table[state, action]
        next_max = np.max(Q_table[next_state, :])
        Q_table[state, action] = old_value + learning_rate * (reward + discount_factor * next_max - old_value)

        # Transition to the next state
        state = next_state

        if done:
            break

    # Decay epsilon
    epsilon = max(min_epsilon, epsilon * epsilon_decay)

# Test the learned policy
state = env.reset()[0]
done = False
print("Testing the policy:")
env.render()
for step in range(max_steps_per_episode):
    action = np.argmax(Q_table[state, :])  # Always exploit
    next_state, reward, done, _, _ = env.step(action)
    env.render()
    state = next_state
    if done:
        print("Reward:", reward)
        break
```

---

### Key Concepts in the Code

1. **Reward:**  
   - Collected from the environment after taking an action (`env.step(action)`).

2. **Action:**  
   - Selected using an **epsilon-greedy policy** (either randomly explore or exploit based on the Q-table).

3. **Policy:**  
   - Derived from the Q-table by always selecting the action with the highest Q-value for a state (`np.argmax(Q_table[state, :])`).

4. **Q-value Update:**
   - $`Q(s, a) \leftarrow Q(s, a) + \alpha \big( r + \gamma \max_{a'} Q(s', a') - Q(s, a) \big)`$
   - $`\alpha`$: learning rate, \(\gamma\): discount factor, \(r\): reward.

5. **State Transition:**
   - The agent transitions to the next state after taking an action (`next_state, reward, done, _, _ = env.step(action)`).

---

### Outputs

- During training, the Q-table is updated based on the agent's interactions.
- During testing, the agent uses the learned Q-table to follow the optimal policy and achieve rewards.

This example demonstrates how to use Gym for **state transitions**, **action selection**, and **reward collection**, while implementing Q-learning yourself.
