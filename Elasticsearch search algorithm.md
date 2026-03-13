## **1. Elasticsearch (ES) Overview**

Elasticsearch is a **distributed, full-text search engine** built on **Apache Lucene**.
It is designed to **search, index, and analyze large amounts of data quickly**.

### **Key Concepts**

* **Document** – the basic unit of data (like a row in SQL).
* **Index** – a collection of documents (like a database table).
* **Shard** – a piece of an index for distribution and scaling.
* **Inverted Index** – core data structure for fast text search.

---

## **2. Search Algorithm in ES**

At a high level, the search works like this:

### **Step 1: Tokenization & Analysis**

* ES breaks text into **tokens** (words) and applies **analyzers** (lowercasing, stemming, stop words removal).
* Example: `"The quick brown fox"` → `["quick", "brown", "fox"]`

### **Step 2: Inverted Index Lookup**

* ES uses an **inverted index**, mapping terms → documents.
* Example:

```
"fox" → [doc1, doc3, doc5]
"quick" → [doc1, doc2]
```

* This allows **O(1) lookup** for documents containing a term.

### **Step 3: Query Execution**

* Different types of queries:

  * **Term Query** – exact match.
  * **Match Query** – analyzed text match (full-text search).
  * **Boolean Query** – AND/OR/NOT combinations.
  * **Range Query** – numerical/date ranges.

### **Step 4: Scoring & Ranking**

Elasticsearch ranks documents using **BM25 (Best Matching 25) algorithm**, which is a modern version of TF-IDF.

**BM25 Formula (simplified)**:
$$[ \text{score}(q,d) = \sum_{t \in q} \frac{f(t,d) \cdot (k+1)}{f(t,d) + k \cdot (1 - b + b \cdot \frac{|d|}{avgdl})} \cdot \log \frac{N - n_t + 0.5}{n_t + 0.5} ]$$

Where:

* (f(t,d)) = frequency of term (t) in document (d)
* (|d|) = length of document (d)
* (avgdl) = average document length
* (N) = total number of documents
* (n_t) = number of documents containing (t)
* (k, b) = hyperparameters

### **Step 5: Sorting & Pagination**

* Results are sorted by **score**, or other fields (like date).
* Supports **pagination** via `from` and `size`.

---

## **3. Optimizations**

* **Sharding & Replication** – splits index across nodes for performance & fault tolerance.
* **Caching** – query cache for repeated searches.
* **Filter Context** – for conditions without scoring (much faster).

---

### **Summary**

Elasticsearch search algorithm = **analyzed tokenization + inverted index lookup + BM25 scoring + distributed execution**.

It is highly optimized for **full-text search at scale**.
