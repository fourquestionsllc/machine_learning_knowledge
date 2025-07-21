Below is a **side-by-side comparison** of building the **same RAG-based chatbot** in two ways:

* ✅ **LangChain (manual chaining)**
* 🧠 **LangGraph (stateful, event-driven framework on top of LangChain)**

---

## 🧩 **Comparison Summary**

| Feature         | LangChain (Manual Chaining)           | LangGraph                                           |
| --------------- | ------------------------------------- | --------------------------------------------------- |
| Chain logic     | Imperative / functional composition   | Declarative, graph-based state transitions          |
| Best for        | Simple RAG flows or one-off pipelines | Complex, stateful, multi-turn conversational agents |
| Memory handling | External (`ConversationBufferMemory`) | Native state machine with memory transitions        |
| Extensibility   | Requires nested conditions & if-else  | Add nodes and transitions cleanly                   |
| Code length     | Shorter for simple use cases          | Scales better for complex agents                    |

---

## 🧠 Let's Build the Same Bot with **LangGraph**

### 🔧 Step 1: Install LangGraph

```bash
pip install langgraph
```

---

### 📄 **LangGraph RAG Chatbot Example**

```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts import PromptTemplate
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.document_loaders import TextLoader
from langgraph.graph import StateGraph, END
from langgraph.graph.state import GraphState
from typing import Annotated, TypedDict
import os

# 1. Setup
os.environ["OPENAI_API_KEY"] = "your-api-key"
llm = ChatOpenAI(model_name="gpt-3.5-turbo", temperature=0)

# 2. Load & Embed documents
loader = TextLoader("docs/example.txt")
docs = loader.load()
vectorstore = FAISS.from_documents(docs, OpenAIEmbeddings())
retriever = vectorstore.as_retriever()

# 3. Define Graph State
class RAGState(TypedDict):
    chat_history: str
    question: str
    standalone_question: str
    context: str
    answer: str
    should_retrieve: bool

# 4. Node: Decide if retrieval is needed
def decide_retrieve(state: RAGState):
    prompt = PromptTemplate.from_template("""
You are an AI assistant. Decide whether the user query needs external knowledge.
Chat history:
{chat_history}
User question:
{question}
Reply "yes" or "no"
""")
    output = llm(prompt.format_prompt(**state).to_messages())
    return {"should_retrieve": "yes" in output.content.lower()}

# 5. Node: Rephrase to standalone question
def rephrase_question(state: RAGState):
    prompt = PromptTemplate.from_template("""
Given the conversation, rephrase the question into a standalone form.

Chat:
{chat_history}
Follow-up:
{question}
Standalone question:
""")
    result = llm(prompt.format_prompt(**state).to_messages())
    return {"standalone_question": result.content.strip()}

# 6. Node: Retrieve documents
def retrieve_docs(state: RAGState):
    query = state["standalone_question"]
    docs = retriever.get_relevant_documents(query)
    return {"context": "\n".join([doc.page_content for doc in docs])}

# 7. Node: Final Answer Generation
def generate_answer(state: RAGState):
    prompt = PromptTemplate.from_template("""
Use this context to answer the user.

Context:
{context}

Question:
{question}

Answer:
""")
    output = llm(prompt.format_prompt(**state).to_messages())
    return {"answer": output.content.strip()}

# 8. Node: Direct LLM (no retrieval)
def no_retrieval_answer(state: RAGState):
    output = llm.invoke(state["question"])
    return {"answer": output.content.strip()}

# 9. Router Node
def router(state: RAGState):
    return "retrieval_path" if state["should_retrieve"] else "direct_path"

# 10. Build LangGraph
builder = StateGraph(RAGState)
builder.add_node("decide", decide_retrieve)
builder.add_node("rephrase", rephrase_question)
builder.add_node("retrieve", retrieve_docs)
builder.add_node("generate", generate_answer)
builder.add_node("direct_llm", no_retrieval_answer)

builder.set_entry_point("decide")
builder.add_conditional_edges("decide", router, {
    "retrieval_path": "rephrase",
    "direct_path": "direct_llm"
})
builder.add_edge("rephrase", "retrieve")
builder.add_edge("retrieve", "generate")
builder.add_edge("generate", END)
builder.add_edge("direct_llm", END)

graph = builder.compile()

# 11. Run the graph
chat_history = "User: What is LangChain?\nAssistant: It's a framework for LLMs."
question = "What components does it have?"

result = graph.invoke({
    "chat_history": chat_history,
    "question": question
})

print("Final Answer:\n", result['answer'])
```

---

## 🔍 **What LangGraph Makes Easier**

* Adding more **nodes** for things like feedback scoring, moderation, memory updates
* Routing logic is cleanly separated
* Built-in support for **looping**, **asynchronous steps**, and **event-based transitions**

---

## ✅ Conclusion: When to Use What

| Use LangChain if...                   | Use LangGraph if...                         |
| ------------------------------------- | ------------------------------------------- |
| You’re building a simple RAG pipeline | You want to model complex state transitions |
| You’re prototyping or scripting       | You’re building production-grade agents     |
| You want minimal external complexity  | You want full agent graphs with branching   |

---
