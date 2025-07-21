Here’s a **complete working example** of a **LangChain-based Retrieval-Augmented Generation (RAG) chatbot**, including the actual code, using:

* `LangChain`
* `OpenAI` LLM
* `FAISS` for vector database
* `TextLoader` for loading documents
* `Chroma` as an optional alternative to FAISS

---

### 🔧 **Install Dependencies**

```bash
pip install langchain openai faiss-cpu tiktoken chromadb
```

---

### 📁 **Directory Structure**

```
rag_chatbot/
├── app.py
├── docs/
│   └── example.txt
```

---

### 📄 **docs/example.txt**

```txt
LangChain is a framework for building applications powered by large language models (LLMs). It helps developers with components for retrieval, chains, and memory.
```

---

### 🧠 **app.py: LangChain RAG Chatbot**

```python
from langchain.document_loaders import TextLoader
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.chat_models import ChatOpenAI
from langchain.chains import RetrievalQA
import os

# Set your OpenAI key
os.environ["OPENAI_API_KEY"] = "your-api-key"

# 1. Load your documents
loader = TextLoader("docs/example.txt")
documents = loader.load()

# 2. Create embeddings from documents
embeddings = OpenAIEmbeddings()

# 3. Create vectorstore using FAISS
vectorstore = FAISS.from_documents(documents, embeddings)

# 4. Create retriever
retriever = vectorstore.as_retriever()

# 5. Initialize LLM
llm = ChatOpenAI(model_name="gpt-3.5-turbo")

# 6. Create RetrievalQA chain
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=retriever,
    return_source_documents=True
)

# 7. Ask a question
query = "What is LangChain used for?"
result = qa_chain(query)

# Print results
print("Answer:", result['result'])
print("\nSource Document:\n", result['source_documents'][0].page_content)
```

---

### ✅ **Sample Output**

```
Answer: LangChain is a framework for building applications powered by large language models (LLMs), offering components for retrieval, chains, and memory.

Source Document:
LangChain is a framework for building applications powered by large language models (LLMs). It helps developers with components for retrieval, chains, and memory.
```

---

### 🧠 Optional: Use ChromaDB instead of FAISS

```python
from langchain.vectorstores import Chroma

vectorstore = Chroma.from_documents(documents, embeddings, persist_directory="./chroma_db")
retriever = vectorstore.as_retriever()
```

---

### 🛠️ Customize With:

* More documents (PDFLoader, WebBaseLoader, etc.)
* Conversational memory (`ConversationBufferMemory`)
* UI (`Gradio`, `Streamlit`)

--------------

Here's a **custom LangChain chain** that follows your required flow for a **RAG-based chatbot**, step-by-step:

---

## 🔄 Custom Chain Flow

1. **Input Question + Chat History**
2. Decide: *Is retrieval needed?*
3. If needed:

   * Generate **standalone query** from chat history
   * Retrieve documents using **vector store**
4. Use retrieved docs + question → LLM generates answer

---

## 💡 Core Components Used

* `ChatOpenAI` for LLMs
* `PromptTemplate` for question classification + standalone query generation
* `RunnableSequence` (LCEL) for custom chains
* `FAISS` vector store
* `LangChain Expression Language (LCEL)` for orchestration

---

## 🧠 Code: Custom RAG Chain with Conditional Retrieval

```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts import PromptTemplate
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.schema.runnable import RunnableLambda, RunnableSequence
from langchain.document_loaders import TextLoader
import os

# --- Setup ---
os.environ["OPENAI_API_KEY"] = "your-api-key"

# Load documents & create vector store
loader = TextLoader("docs/example.txt")
docs = loader.load()
embeddings = OpenAIEmbeddings()
vectorstore = FAISS.from_documents(docs, embeddings)
retriever = vectorstore.as_retriever()

# --- LLM ---
llm = ChatOpenAI(temperature=0, model="gpt-3.5-turbo")

# --- Step 1: Decide if retrieval is needed ---
should_retrieve_prompt = PromptTemplate.from_template("""
You are an AI assistant. Decide whether the user query requires knowledge outside the chat history.
Chat history:
{chat_history}

User question:
{question}

Answer "yes" if retrieval is needed, otherwise "no".
""")

should_retrieve_chain = should_retrieve_prompt | llm | (lambda x: "yes" in x.content.lower())

# --- Step 2: Generate standalone search query ---
rephrase_prompt = PromptTemplate.from_template("""
Given the chat history and a follow-up question, rephrase it as a standalone question.

Chat history:
{chat_history}

Follow-up question:
{question}

Standalone question:
""")

rephrase_chain = rephrase_prompt | llm

# --- Step 3: Retrieval ---
retrieval_chain = RunnableLambda(lambda question: retriever.get_relevant_documents(question))

# --- Step 4: Answer generation using retrieved docs ---
answer_prompt = PromptTemplate.from_template("""
Use the following context to answer the question.

Context:
{context}

Question:
{question}

Answer:""")

final_answer_chain = answer_prompt | llm

# --- Full Chain Composition ---
def full_rag_chain(inputs: dict):
    chat_history = inputs["chat_history"]
    question = inputs["question"]

    should_retrieve = should_retrieve_chain.invoke({"chat_history": chat_history, "question": question})

    if should_retrieve:
        standalone_question = rephrase_chain.invoke({"chat_history": chat_history, "question": question}).content
        docs = retrieval_chain.invoke(standalone_question)
        context = "\n".join([doc.page_content for doc in docs])
        return final_answer_chain.invoke({"context": context, "question": question}).content
    else:
        return llm.invoke(question).content

# --- Run Example ---
chat_history = "User: What is LangChain?\nAssistant: It's a framework for building LLM-powered apps."
question = "What components does it provide?"

response = full_rag_chain({"chat_history": chat_history, "question": question})
print("Answer:\n", response)
```

---

## ✅ Output Example

```
Answer:
LangChain provides components for retrieval, chains, and memory that help developers build applications powered by LLMs.
```

---

## 🧪 Want to Go Further?

* Replace `FAISS` with `Chroma` or `Weaviate`
* Add `ConversationBufferMemory` to maintain chat context
* Serve this chain with FastAPI or Streamlit



