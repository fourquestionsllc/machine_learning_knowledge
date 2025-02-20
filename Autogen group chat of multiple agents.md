In this blog post, we'll explore how to build a collaborative environment using AutoGen's conversable agents to automate tasks through group chats. Specifically, we'll create a scenario where a **Coder Agent** and a **Visualization Critic Agent** work together to download a dataset, analyze the relationship between two variables, and generate a visualization.

## Prerequisites

Before diving into the implementation, ensure you have the following installed:

- **Python 3.7 or higher**
- **AutoGen AgentChat**: Install via pip:

  ```bash
  pip install autogen-agentchat~=0.2
  ```

- **Additional Libraries**: We'll use `matplotlib`, `pandas`, and `seaborn` for data handling and visualization. Install them using:

  ```bash
  pip install matplotlib pandas seaborn
  ```

## Setting Up the Environment

First, import the necessary libraries and configure your API endpoint:

```python
import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns
from IPython.display import Image

import autogen

# Load configuration for GPT-4
config_list_gpt4 = autogen.config_list_from_json("OAI_CONFIG_LIST")
```


The `config_list_from_json` function loads a list of configurations from an environment variable or a JSON file, setting up the Large Language Model (LLM) configurations for our agents.

## Defining the Agents

We'll define two agents:

1. **Coder Agent**: Responsible for writing code to download and visualize the dataset.
2. **Visualization Critic Agent**: Reviews the generated visualization and provides feedback.

```python
# Initialize the Coder Agent
coder = autogen.AssistantAgent(
    name="Coder",
    system_message="You are a coder capable of writing Python code to download datasets and generate visualizations.",
    llm_config=config_list_gpt4[0],
)

# Initialize the Visualization Critic Agent
visualization_critic = autogen.AssistantAgent(
    name="VisualizationCritic",
    system_message="You are a visualization critic. Your task is to review visualizations and provide constructive feedback.",
    llm_config=config_list_gpt4[0],
)
```


Each agent is initialized with a name, a system message defining its role, and an LLM configuration.

## Creating the Group Chat

To facilitate interaction between the agents, we'll set up a group chat managed by a `GroupChatManager`.

```python
# Create the Group Chat
groupchat = autogen.GroupChat(
    agents=[coder, visualization_critic],
    messages=[],
    max_round=5,
    speaker_selection_method="round_robin",
)

# Initialize the Group Chat Manager
manager = autogen.GroupChatManager(
    groupchat=groupchat,
    llm_config=config_list_gpt4[0],
)
```


The `GroupChat` includes both agents and specifies parameters like the maximum number of interaction rounds and the speaker selection method. The `GroupChatManager` oversees the conversation flow.

## Initiating the Conversation

We'll start the chat by instructing the `Coder` agent to download a dataset and create a visualization.

```python
# User's initial instruction
user_instruction = (
    "Download the dataset from "
    "https://raw.githubusercontent.com/uwdata/draco/master/data/cars.csv "
    "and create a scatter plot showing the relationship between weight and horsepower. "
    "Save the plot as 'weight_vs_horsepower.png'."
)

# Initiate the chat
coder.initiate_chat(
    manager,
    message=user_instruction,
)
```


The `Coder` agent will process this instruction, generate the necessary Python code, and execute it to produce the visualization.

## Agent Interaction and Feedback Loop

After the `Coder` agent generates the visualization, the `Visualization Critic` agent reviews it and provides feedback. This collaborative loop continues for the defined number of rounds or until the task is satisfactorily completed.

## Displaying the Visualization

Once the visualization is created and saved, we can display it within the notebook:

```python
# Display the generated plot
Image(filename="weight_vs_horsepower.png")
```


This will render the scatter plot directly in the notebook, allowing for immediate inspection and further analysis.

## Conclusion

By leveraging AutoGen's conversable agents, we've demonstrated how to automate a data analysis and visualization task through a collaborative group chat. This approach showcases the potential of multi-agent systems in streamlining complex workflows and enhancing productivity.

For more detailed information and advanced features, refer to the [AutoGen documentation](https://microsoft.github.io/autogen/0.2/docs/notebooks/agentchat_groupchat_vis/). 
