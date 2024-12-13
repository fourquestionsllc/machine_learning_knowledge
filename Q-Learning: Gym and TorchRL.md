Below are two implementations of Q-learning: one using **Gym** and plain Python, and the other using **TorchRL** for enhanced functionality.

---

### Example 1: Q-Learning with Gym (Plain Python)

```python
import gym
import numpy as np
import random

# Create the environment
env = gym.make('FrozenLake-v1', is_slippery=False, map_name="4x4")  # FrozenLake example
state_space = env.observation_space.n
action_space = env.action_space.n

# Initialize Q-table
q_table = np.zeros((state_space, action_space))

# Hyperparameters
alpha = 0.1  # Learning rate
gamma = 0.99  # Discount factor
epsilon = 1.0  # Exploration rate
epsilon_decay = 0.995
min_epsilon = 0.01
episodes = 1000
max_steps = 100

# Training loop
for episode in range(episodes):
    state = env.reset()[0]  # Reset environment
    done = False
    for _ in range(max_steps):
        # Choose action (ε-greedy)
        if random.uniform(0, 1) < epsilon:
            action = env.action_space.sample()  # Explore
        else:
            action = np.argmax(q_table[state])  # Exploit

        # Take action
        next_state, reward, done, _, _ = env.step(action)
        
        # Update Q-value
        q_table[state, action] = q_table[state, action] + alpha * (
            reward + gamma * np.max(q_table[next_state]) - q_table[state, action]
        )
        
        state = next_state
        if done:
            break
    
    # Decay epsilon
    epsilon = max(min_epsilon, epsilon * epsilon_decay)

# Test the policy
state = env.reset()[0]
env.render()
done = False
while not done:
    action = np.argmax(q_table[state])
    state, reward, done, _, _ = env.step(action)
    env.render()
```

---

### Example 2: Q-Learning with TorchRL

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torchrl.envs import GymWrapper, TransformedEnv, Compose, ToTensor
from torchrl.data import ReplayBuffer, LazyTensorStorage
from torchrl.objectives import DQNLoss
from torchrl.modules import QValueModule

# Environment setup
from gymnasium.envs.classic_control import CartPoleEnv
env = TransformedEnv(
    GymWrapper(CartPoleEnv()),
    Compose(ToTensor())
)

# Q-Network
class QNetwork(nn.Module):
    def __init__(self, input_dim, output_dim):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(input_dim, 128),
            nn.ReLU(),
            nn.Linear(128, output_dim)
        )

    def forward(self, x):
        return self.net(x)

q_net = QNetwork(input_dim=env.observation_spec.shape[-1], output_dim=env.action_spec.shape[-1])
optimizer = optim.Adam(q_net.parameters(), lr=1e-3)

# Replay buffer
buffer_size = 10000
batch_size = 64
replay_buffer = ReplayBuffer(storage=LazyTensorStorage(size=buffer_size))

# Training parameters
gamma = 0.99
epsilon = 1.0
epsilon_decay = 0.995
min_epsilon = 0.01
episodes = 1000

# Training loop
for episode in range(episodes):
    state = env.reset()
    done = False
    while not done:
        # Select action (ε-greedy)
        if torch.rand(1).item() < epsilon:
            action = env.action_spec.rand()
        else:
            action = torch.argmax(q_net(state), dim=-1)

        # Step environment
        next_state, reward, done, _ = env.step(action)
        transition = {"state": state, "action": action, "next_state": next_state, "reward": reward}
        replay_buffer.append(transition)
        state = next_state

        # Sample and train
        if len(replay_buffer) > batch_size:
            batch = replay_buffer.sample(batch_size)
            loss = DQNLoss(q_net, gamma).forward(batch)
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()

    # Decay epsilon
    epsilon = max(min_epsilon, epsilon * epsilon_decay)
```

---

### Key Differences:
1. **Gym Example**:
   - Uses a simple tabular Q-learning approach.
   - Best for discrete state and action spaces.

2. **TorchRL Example**:
   - Uses a neural network to approximate Q-values, making it suitable for continuous or large state/action spaces.
   - Includes advanced utilities like replay buffers for better training efficiency.

Would you like further clarification or extensions on either of these implementations?
