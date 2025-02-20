# Building a Multi-Agent RAG-Based Chatbot with LangChain

Retrieval-Augmented Generation (RAG) is a powerful approach to building intelligent chatbots that can query documents and generate informed responses. In this blog, we'll implement a RAG-based chatbot using LangChain with multiple agents, each responsible for specific tasks.

## **Architecture Overview**
Our chatbot will use the following agents:
1. **Question Classifier Agent**: Determines if a question requires document retrieval.
2. **Rewriter Agent**: Rewrites the question into a stand-alone query.
3. **Retriever Agent**: Fetches relevant document chunks from a vector database.
4. **Answer Generator Agent**: Generates a response based on retrieved documents.
5. **Orchestrator Agent**: Manages the interaction between agents.

### **Tech Stack**
- Python
- LangChain
- FAISS (as a vector database)
- OpenAI API (for LLM)
- ChromaDB or Pinecone (alternative vector store)

## **Implementation**
### **Step 1: Install Dependencies**
```bash
pip install langchain openai faiss-cpu chromadb
```

### **Step 2: Initialize LangChain and Agents**

#### **Load OpenAI LLM**
```python
from langchain.chat_models import ChatOpenAI
from langchain.schema import SystemMessage

llm = ChatOpenAI(model_name="gpt-4", temperature=0.7)
```

#### **Define Vector Store (FAISS)**
```python
from langchain.vectorstores import FAISS
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.document_loaders import TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter

# Load and process documents
loader = TextLoader("documents/sample.txt")
documents = loader.load()
text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)
chunks = text_splitter.split_documents(documents)

# Create vector store
embeddings = OpenAIEmbeddings()
vector_store = FAISS.from_documents(chunks, embeddings)
```

### **Step 3: Implement Agents**

#### **1. Question Classifier Agent**
```python
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate

question_classifier_prompt = PromptTemplate(
    input_variables=["question"],
    template="""Classify if the given question requires document retrieval.
    Question: {question}
    Respond with 'yes' or 'no'."""
)
question_classifier_chain = LLMChain(llm=llm, prompt=question_classifier_prompt)

def should_retrieve(question):
    response = question_classifier_chain.run(question)
    return response.strip().lower() == "yes"
```

#### **2. Rewriter Agent**
```python
question_rewriter_prompt = PromptTemplate(
    input_variables=["chat_history", "question"],
    template="""Rewrite the given question into a self-contained query, considering the chat history.
    Chat History: {chat_history}
    Question: {question}
    Standalone Question:"""
)
question_rewriter_chain = LLMChain(llm=llm, prompt=question_rewriter_prompt)

def rewrite_question(chat_history, question):
    return question_rewriter_chain.run(chat_history=chat_history, question=question)
```

#### **3. Retriever Agent**
```python
retriever = vector_store.as_retriever()

def retrieve_documents(query):
    return retriever.get_relevant_documents(query)
```

#### **4. Answer Generator Agent**
```python
answer_generator_prompt = PromptTemplate(
    input_variables=["question", "context"],
    template="""Given the following context, answer the question.
    Context: {context}
    Question: {question}
    Answer:"""
)
answer_generator_chain = LLMChain(llm=llm, prompt=answer_generator_prompt)

def generate_answer(question, docs):
    context = "\n".join([doc.page_content for doc in docs])
    return answer_generator_chain.run(question=question, context=context)
```

### **Step 4: Implement the Orchestrator**
```python
def orchestrate_chat(chat_history, question):
    if should_retrieve(question):
        standalone_question = rewrite_question(chat_history, question)
        documents = retrieve_documents(standalone_question)
        answer = generate_answer(standalone_question, documents)
    else:
        answer = llm.predict(question)
    
    chat_history.append((question, answer))
    return answer
```

### **Step 5: Run the Chatbot**
```python
chat_history = []
while True:
    user_input = input("You: ")
    if user_input.lower() in ["exit", "quit"]:
        break
    response = orchestrate_chat(chat_history, user_input)
    print("Bot:", response)
```

## **Conclusion**
In this tutorial, we built a multi-agent RAG-based chatbot using LangChain. The chatbot efficiently classifies questions, rewrites queries, retrieves relevant documents, and generates informative answers. This modular approach allows for scalable and adaptable conversational AI applications.

