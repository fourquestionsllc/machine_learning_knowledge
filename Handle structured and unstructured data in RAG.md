## ✅ **Short interview answer**

“Structured data already has schema, so ingestion focuses on converting tables into natural-language or JSON chunks with metadata. Unstructured data requires extraction, cleaning, chunking, and embedding. In RAG, the challenge is normalizing both so the retriever returns semantically relevant information, regardless of source type.”

---

## ✅ **Full strong interview answer**

“In RAG systems we handle **structured and unstructured data differently** during ingestion because they have different formats and retrieval patterns:

---

### **1. Structured Data (tables, SQL, spreadsheets, APIs)**

Structured data has a defined schema — rows, columns, keys.

**How I ingest it:**

* Map tables into **semantic text representations** (“row-to-text”, “table summaries”, “column descriptions”).
* Optionally keep it as **JSON** chunks if the LLM consumes structured formats well.
* Add metadata like table name, primary keys, source system, last-update timestamp.
* Sometimes I generate **generated summaries** (e.g., “sales by region 2023”) to improve retrieval.
* For large tables, I don’t embed every row — instead:

  * embed column descriptions
  * embed aggregated slices
  * use hybrid retrieval (keyword + vector)
  * or route through a text-to-SQL agent instead of RAG

**Goal:** Make structured data semantically searchable and linkable to SQL or downstream reasoning.

---

### **2. Unstructured Data (PDF, docs, emails, HTML, slides)**

This has no schema and requires preprocessing.

**How I ingest it:**

* Extract clean text via loaders (PDFMiner, PyMuPDF, OCR if needed).
* Remove junk (page numbers, headers, watermarks).
* Chunk into 200–1000 token segments with overlap.
* Add metadata: document name, headings, author, section, etc.
* Generate embeddings per chunk.
* Store chunks + embeddings in the vector store for semantic search.

**Goal:** Convert messy text into clean, chunked, semantically searchable documents.

---

### **3. Hybrid Retrieval (Best Practice)**

RAG systems often combine both:

* **Structured data → SQL agent / semantic SQL retrieval**
* **Unstructured data → vector store retrieval**
* **Ranking → Hybrid reranker (BM25 + LLM reranking)**

This ensures high recall from tables and high semantic relevance from documents.


Give me the next keyword.
