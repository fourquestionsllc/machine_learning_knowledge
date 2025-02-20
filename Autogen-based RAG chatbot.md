# Building a RAG-Based Chatbot with AutoGen: A Multi-Agent Approach

## Introduction
Retrieval-Augmented Generation (RAG) enhances chatbots by incorporating **external knowledge retrieval** into the response generation process. In this article, we'll implement a **multi-agent RAG chatbot using AutoGen**, leveraging agents for different tasks such as:
- **Deciding whether retrieval is needed**
- **Rewriting questions into a stand-alone format**
- **Retrieving relevant documents from a vector database**
- **Generating responses based on retrieved documents**
- **Orchestrating agent interactions**

## Architecture Overview
We'll use **AutoGen's agent-based framework** to implement the following agents:
1. **Retrieval Decision Agent** - Determines if a query requires document retrieval.
2. **Question Rewriting Agent** - Converts user queries into a stand-alone format.
3. **Retrieval Agent** - Queries a vector database for relevant document chunks.
4. **Answer Generation Agent** - Generates responses based on retrieved information.
5. **Orchestration Agent** - Coordinates the agents and manages the workflow.

---

## Implementation
### Step 1: Install Dependencies
```bash
pip install autogen langchain chromadb openai
```

---

### Step 2: Define Agent Prompts
#### 1. Retrieval Decision Agent Prompt
This agent determines if a document search is required.
```python
retrieval_decision_prompt = """
You are a decision-making agent. Your task is to determine whether the user’s question requires document retrieval.
If the question is general knowledge or common sense, respond with 'NO'.
If the answer requires external document retrieval, respond with 'YES'.
User Question: {question}
Decision (YES or NO):
"""
```

#### 2. Question Rewriting Agent Prompt
This agent reformulates questions into a stand-alone format.
```python
question_rewrite_prompt = """
You are a question rewriter. Your task is to rewrite the user's query into a self-contained question.
Make sure the question can be understood without chat history.
Chat History:
{history}
User Question: {question}
Rewritten Question:
"""
```

#### 3. Retrieval Agent
Retrieves relevant chunks from a **ChromaDB** vector database.
```python
retrieval_prompt = """
You are a retrieval agent. Given a question, retrieve the top 3 relevant chunks from the vector database.
User Question: {question}
Relevant Chunks:
"""
```

#### 4. Answer Generation Agent
This agent generates an answer using retrieved document chunks.
```python
answer_generation_prompt = """
You are an AI assistant. Use the following retrieved documents to generate an informative answer.

Retrieved Chunks:
{documents}

User Question: {question}
Answer:
"""
```

---

### Step 3: Implement AutoGen Agents

#### 1. Import Required Libraries
```python
from autogen import Agent, UserProxyAgent, GroupChat, GroupChatManager
from langchain.vectorstores import Chroma
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.document_loaders import TextLoader

# Load and prepare vector database
embedding_function = OpenAIEmbeddings()
vector_db = Chroma(persist_directory="./chroma_db", embedding_function=embedding_function)
```

#### 2. Define Individual Agents
```python
retrieval_decision_agent = Agent(
    name="retrieval_decision",
    system_message=retrieval_decision_prompt
)

question_rewrite_agent = Agent(
    name="question_rewriter",
    system_message=question_rewrite_prompt
)

retrieval_agent = Agent(
    name="retrieval",
    system_message=retrieval_prompt,
    function=lambda question: vector_db.similarity_search(question, k=3)
)

answer_generation_agent = Agent(
    name="answer_generator",
    system_message=answer_generation_prompt
)
```

#### 3. Implement the Orchestration Agent
```python
def orchestrate_chat(user_query, chat_history):
    # Step 1: Decide if retrieval is needed
    retrieval_needed = retrieval_decision_agent.run(question=user_query)
    if retrieval_needed.strip().upper() == "NO":
        return answer_generation_agent.run(documents="", question=user_query)
    
    # Step 2: Rewrite the question
    rewritten_question = question_rewrite_agent.run(history=chat_history, question=user_query)
    
    # Step 3: Retrieve relevant documents
    retrieved_docs = retrieval_agent.run(question=rewritten_question)
    
    # Step 4: Generate the final answer
    response = answer_generation_agent.run(documents=retrieved_docs, question=user_query)
    return response
```

---

### Step 4: Running the Chatbot
```python
chat_history = []
while True:
    user_input = input("User: ")
    if user_input.lower() in ["exit", "quit"]:
        break
    
    response = orchestrate_chat(user_input, chat_history)
    chat_history.append(("User", user_input))
    chat_history.append(("AI", response))
    print("AI:", response)
```

---

## Conclusion
In this article, we built a **multi-agent RAG chatbot using AutoGen**. The chatbot:
- **Decides if document retrieval is necessary**
- **Rewrites user queries into a self-contained format**
- **Retrieves relevant document chunks from a vector database**
- **Generates high-quality responses**
- **Uses an orchestration agent to manage interactions**

This modular approach ensures **scalability, adaptability, and efficient knowledge retrieval**, making it ideal for enterprise chatbots and knowledge-based assistants.
