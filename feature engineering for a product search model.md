**Feature engineering for a product search model**—usually for **search ranking or relevance prediction** (e.g., given a user query and a product, predict how relevant it is). 

---

## **1. Types of Features**

Feature engineering for product search generally falls into **three categories**: textual, numeric/metadata, and behavioral.

---

### **A. Textual Features**

Used to measure **semantic and lexical similarity** between user query and product data.

1. **Query-Product Matching Features**

   * **TF-IDF similarity** (cosine similarity between query and product title/description)
   * **BM25 score** (classic search ranking feature)
   * **Jaccard similarity** (overlap between query and product tokens)
   * **Word embeddings / sentence embeddings similarity**

     * Use pre-trained embeddings like **Sentence-BERT**, **OpenAI embeddings**, or product-specific fine-tuned embeddings
   * **N-gram overlap**

     * Count of 1-gram, 2-gram overlaps between query and title/description

2. **Query-Product Keyword Features**

   * Query term in **title**, **description**, **brand**, **category**
   * Position of query term (title match is stronger than description match)
   * **Exact match indicator** (binary)

3. **Semantic Features**

   * Cosine similarity of embeddings for query vs. product text
   * Concept or category match (e.g., using product ontology)
   * Named entity match (brand, model number, color)

---

### **B. Product Metadata Features**

Leverage **structured data** about products:

1. **Popularity**

   * Sales count, click-through rate (CTR), purchase conversion rate
   * Rating score or review count

2. **Product Attributes**

   * Price (absolute, normalized, relative to category)
   * Discount, stock availability
   * Brand reputation

3. **Category & Hierarchy**

   * Category match with query
   * Depth in category tree
   * Category popularity

4. **Recency / Freshness**

   * New arrivals boost
   * Recently trending items

---

### **C. Query and User Interaction Features**

Features related to user behavior and context:

1. **Click-through / Purchase History**

   * CTR of product for similar queries
   * Personalized history features

2. **Query Popularity**

   * Frequency of query in search logs
   * Click distribution across products for query

3. **Session Context**

   * Previous queries in the same session
   * Products already viewed

---

### **D. Interaction Features (Cross Features)**

Capture relationships between query and product or between product features:

* Query length vs. product title length
* Query term match ratio = (# matched terms / # query terms)
* BM25 score normalized by category
* Embedding similarity weighted by product popularity

---

## **2. Feature Engineering Steps**

1. **Text Preprocessing**

   * Lowercasing, stopword removal, stemming/lemmatization
   * Special handling for numbers, brands, and model codes

2. **Feature Extraction**

   * Compute similarity scores (TF-IDF, BM25, embeddings)
   * Extract categorical and numeric metadata features

3. **Normalization & Encoding**

   * Normalize numeric features (e.g., MinMax or Z-score)
   * Encode categorical features (one-hot or embeddings)

4. **Feature Selection**

   * Check correlation, importance with model
   * Remove redundant or noisy features

5. **Optional: Feature Transformation**

   * Log-transform skewed features (e.g., sales, price)
   * Combine features (e.g., weighted embedding similarity × CTR)

---

### **3. Feature Examples Table**

| Feature Type | Feature Example          | Description                             |
| ------------ | ------------------------ | --------------------------------------- |
| Textual      | TF-IDF cosine similarity | Between query & product title           |
| Textual      | BM25 score               | Between query & description             |
| Textual      | Embedding similarity     | Using Sentence-BERT or OpenAI embedding |
| Metadata     | Price normalized         | Price scaled within category            |
| Metadata     | Product rating           | Average rating from reviews             |
| Behavioral   | CTR for query-product    | Click-through rate in logs              |
| Behavioral   | Query frequency          | Popularity of the query                 |
| Interaction  | Query term match ratio   | Fraction of query tokens in title       |

---

✅ **Tip:** Modern product search often combines **traditional features (BM25, TF-IDF)** with **dense embeddings** in a **learning-to-rank model** (like LambdaMART, XGBoost, or deep learning models) for better relevance.
