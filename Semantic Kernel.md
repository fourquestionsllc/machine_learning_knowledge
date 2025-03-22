# Unlocking the Power of Semantic Kernel in Generative AI: A Practical Guide with Code

Generative AI is evolving rapidly, and one of the most powerful tools helping developers build intelligent AI workflows is **Semantic Kernel (SK)**. Developed by Microsoft, Semantic Kernel is an open-source SDK designed to seamlessly integrate AI models like OpenAI and Azure OpenAI with conventional programming logic and memory. It provides a flexible framework to build, compose, and orchestrate complex AI pipelines.

In this blog, we'll explore:
- What Semantic Kernel is
- Its key functions in Gen AI
- How to use it with actual code examples

---

## What is Semantic Kernel?

**Semantic Kernel (SK)** is a lightweight SDK that enables you to create plugins and pipelines using natural language prompts, semantic functions, and traditional programming code. It's especially effective in building:
- AI Agents
- Copilot experiences
- Prompt chaining
- Knowledge-based systems (like RAG pipelines)

---

## Key Functions of Semantic Kernel for Generative AI

1. **Semantic Functions** – Natural language instructions backed by prompts and LLMs.
2. **Native Functions** – C#/Python code functions that integrate external logic or systems.
3. **Planners** – Automatically create plans (sequences of steps) to achieve a goal.
4. **Memory Integration** – Store and retrieve data using vector embeddings (for Retrieval-Augmented Generation).
5. **Chaining & Orchestration** – Seamlessly combine multiple functions (semantic + native) to build complex flows.

---

## Getting Started with Semantic Kernel (Python Example)

### Step 1: Install Semantic Kernel
```bash
pip install semantic-kernel
```

### Step 2: Basic Setup
```python
import semantic_kernel as sk
from semantic_kernel.connectors.ai.open_ai import OpenAIChatCompletion

kernel = sk.Kernel()
kernel.add_chat_service("openai", OpenAIChatCompletion(
    model_id="gpt-3.5-turbo",
    api_key="YOUR_OPENAI_API_KEY"
))
```

### Step 3: Create a Semantic Function (Prompt Template)
```python
from semantic_kernel.semantic_functions import SemanticFunctionConfig

prompt_template = """
You are a creative storyteller. Write a short story based on the theme: {{$input}}
"""

story_function = kernel.create_semantic_function(
    prompt_template,
    function_name="CreativeStory",
    description="Generates a creative story based on input theme."
)

output = story_function("a lonely astronaut exploring Mars")
print(output)
```

### Step 4: Add Native Functions (Custom Python Code)
```python
def word_count_function(text: str) -> int:
    return len(text.split())

kernel.add_function("Tools", "WordCounter", word_count_function)
result = kernel.run("Tools.WordCounter", input=output)
print("Word Count:", result)
```

### Step 5: Combine and Chain Functions
You can chain semantic + native functions in a workflow:
```python
from semantic_kernel.orchestration.kernel_arguments import KernelArguments

args = KernelArguments(input="a wizard lost in a futuristic world")
result = kernel.run_async(["CreativeStory", "Tools.WordCounter"], input_args=args)
print(result)
```

---

## Real-world Use Case: RAG Pipeline Example
```python
from semantic_kernel.memory import VolatileMemoryStore
from semantic_kernel.connectors.memory.embedding.openai import OpenAITextEmbedding

kernel.add_text_embedding_generation_service("embedding", OpenAITextEmbedding(
    model_id="text-embedding-ada-002",
    api_key="YOUR_OPENAI_API_KEY"
))

memory = VolatileMemoryStore()
kernel.register_memory(memory)

# Save and retrieve a knowledge snippet
await kernel.memory.save_information("facts", id="fact1", text="The sun is a star.")
results = await kernel.memory.search("facts", "What is the sun?", limit=1)
for r in results:
    print(r.text)
```

---

## Conclusion
Semantic Kernel is a powerful abstraction that bridges the gap between prompt engineering and software engineering. With SK, you can:
- Build scalable Gen AI applications
- Integrate LLMs with business logic
- Leverage planning and memory for autonomous agents

**Explore more**: [https://github.com/microsoft/semantic-kernel](https://github.com/microsoft/semantic-kernel)

Whether you're building an AI copilot, a smart chatbot, or an autonomous agent—Semantic Kernel makes it modular, composable, and developer-friendly.

