Below are examples of Q-learning implemented separately with **OpenAI Gym** and **TorchRL**. 

---

### **Example 1: Q-Learning with OpenAI Gym**
This uses a simple table-based Q-learning approach in a discrete environment like `FrozenLake-v1`.

---

### **Example 2: Q-Learning with TorchRL**
This uses a neural network to approximate the Q-values, suitable for environments with continuous state spaces like `CartPole-v1`.

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torchrl.envs import GymWrapper, TransformedEnv
from torchrl.envs.transforms import ToTensor
from collections import deque
import random

# Step 1: Define the environment
env = TransformedEnv(GymWrapper("CartPole-v1"), ToTensor())

# Step 2: Define the Q-Network
class QNetwork(nn.Module):
    def __init__(self, state_dim, action_dim):
        super(QNetwork, self).__init__()
        self.net = nn.Sequential(
            nn.Linear(state_dim, 128),
            nn.ReLU(),
            nn.Linear(128, 128),
            nn.ReLU(),
            nn.Linear(128, action_dim)
        )

    def forward(self, x):
        return self.net(x)

# Hyperparameters
state_dim = env.observation_spec.shape[0]
action_dim = env.action_spec.n

q_network = QNetwork(state_dim, action_dim)
optimizer = optim.Adam(q_network.parameters(), lr=1e-3)

# Replay buffer
replay_buffer = deque(maxlen=10000)

# Parameters
batch_size = 64
gamma = 0.99
epsilon = 1.0
epsilon_decay = 0.995
epsilon_min = 0.1

# Helper functions
def select_action(state):
    global epsilon
    if random.random() < epsilon:
        return random.randint(0, action_dim - 1)
    with torch.no_grad():
        state = torch.tensor(state, dtype=torch.float32).unsqueeze(0)
        q_values = q_network(state)
        return torch.argmax(q_values).item()

def train_step():
    if len(replay_buffer) < batch_size:
        return

    # Sample a batch from replay buffer
    batch = random.sample(replay_buffer, batch_size)
    states, actions, rewards, next_states, dones = zip(*batch)

    states = torch.tensor(states, dtype=torch.float32)
    actions = torch.tensor(actions, dtype=torch.long).unsqueeze(1)
    rewards = torch.tensor(rewards, dtype=torch.float32).unsqueeze(1)
    next_states = torch.tensor(next_states, dtype=torch.float32)
    dones = torch.tensor(dones, dtype=torch.float32).unsqueeze(1)

    # Compute target Q-values
    with torch.no_grad():
        target_q_values = rewards + gamma * (1 - dones) * torch.max(q_network(next_states), dim=1, keepdim=True)[0]

    # Compute predicted Q-values
    current_q_values = q_network(states).gather(1, actions)

    # Loss calculation
    loss = nn.MSELoss()(current_q_values, target_q_values)

    # Backpropagation
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()

# Training loop
num_episodes = 500

def train():
    global epsilon
    for episode in range(num_episodes):
        state = env.reset()
        done = False
        total_reward = 0

        while not done:
            action = select_action(state)
            next_state, reward, done, _ = env.step(action)

            # Store transition in replay buffer
            replay_buffer.append((state, action, reward, next_state, done))

            state = next_state
            total_reward += reward

            train_step()

        # Decay epsilon
        epsilon = max(epsilon_min, epsilon * epsilon_decay)

        print(f"Episode {episode + 1}, Total Reward: {total_reward}, Epsilon: {epsilon:.2f}")

# Run training
if __name__ == "__main__":
    train()
```

---

Each example demonstrates Q-learning applied differently:
1. **Gym**: Traditional Q-learning with a table for discrete environments.
2. **TorchRL**: Deep Q-learning for continuous state spaces.
