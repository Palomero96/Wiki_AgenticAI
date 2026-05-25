# Tokens, Embeddings, Vectors, and Transfer Learning in NLP

This document explains fundamental concepts to understand how language models process text, represent information, and adapt to new tasks.

---

## 1. Tokens

* **Definition**: Minimum unit of text that a model processes. It can be a word, subword, or character.
* **Types**:
  * Whole words: "cat", "dog"
  * Subwords (BPE, WordPiece): "c", "##at", "d", "##og"
  * Characters: "c", "a", "t"
* **Example**:

```text
Text: "Artificial intelligence"
Tokens (WordPiece): ["Art", "##ificial", "intelligence"]
```

* **Importance**: Tokenization defines how embeddings and the input to the model are constructed.

---

## 2. Embeddings

* **Definition**: Dense vector representation of tokens or entities that captures semantic meaning.
* **Properties**:
  * Close vectors → similar words
  * Allow for semantic operations:

$$v_{king} - v_{man} + v_{woman} \approx v_{queen}$$

* **Typical dimension**: 50-1024 depending on the model.
* **Conceptual diagram**:

```text
Tokens --> Embedding Layer --> Dense vectors
```

---

## 3. Vectors and Vector Search

### 3.1 Vectors vs Embeddings

* **Vector**: A list of numbers in an n-dimensional space, can represent any data.
* **Embedding**: A special vector that captures semantic or contextual relationships.
* **Visual summary**:

```text
All embeddings are vectors
But not all vectors are embeddings
```

### 3.2 Vector Search

* **Goal**: Find similar items in a vector space.
* **Uses**: Information retrieval, recommendation, semantic search.
* **Similarity metrics**:
  * Cosine similarity:

$$\text{sim}(u,v) = \frac{u \cdot v}{\|u\| \|v\|}$$

  * Euclidean distance:

$$d(u,v) = \sqrt{\sum_i (u_i - v_i)^2}$$

### 3.3 Conceptual search example

```text
Embeddings:
v_cat = [0.21, -0.11, 0.56]
v_dog = [0.19, -0.09, 0.53]
v_lion = [0.40, 0.12, 0.80]

Query: "cat"
Similarities:
cos(v_cat, v_dog) = 0.98
cos(v_cat, v_lion) = 0.75

Result: "dog" (most semantically similar)
```

* **Conceptual search diagram**:

```text
Query (embedding) --> Compare with embeddings in the database --> Ranking by similarity --> Closest result
```

### 3.4 Vector Databases and Indexes

* **Definition**: Specialized systems to store and search embeddings efficiently.
* **Goal**: Enable fast and scalable vector search in high dimensions.
* **Examples of vector databases**:
  * FAISS (Facebook AI Similarity Search)
  * Milvus
  * Pinecone
* **Common indexes and details**:
  * **HNSW (Hierarchical Navigable Small World)**: A graph structure that allows approximate nearest neighbor search with high efficiency and accuracy.
  * **IVF (Inverted File Index)**: Divides the vector space into clusters and searches first in the most relevant cluster, accelerating retrieval.
  * **Annoy (Approximate Nearest Neighbors Oh Yeah)**: Randomized trees that allow fast searches with approximations in high-dimensional spaces.
  * **PQ (Product Quantization)**: Reduces the memory needed to store large vectors, allowing faster searches with compression.

* **Conceptual diagram**:

```text
Embeddings --> Indexing (HNSW / IVF / Annoy / PQ) --> Approximate Search --> Most similar results
```

---

## 4. Fine-Tuning

* **Definition**: Adjusting a pre-trained model to a specific task.
* **Process**:
  1. Take a pre-trained model (BERT, GPT)
  2. Add task-specific layer(s)
  3. Train with task data
* **Conceptual example**:

```text
Pre-trained BERT --> Sentiment classifier --> Review dataset
```

* **Advantage**: Reduces the need for large amounts of data and training time.

---

## 5. Transfer Learning

* **Definition**: Reusing knowledge learned in one task for another related task.
* **Types in NLP**:
  * Feature-based: using embeddings as features
  * Fine-tuning: adjusting the entire model to the new task
* **Conceptual example**:

```text
Model pre-trained on Wikipedia --> News classification --> Tweet classification
```

* **Benefit**: Allows leveraging large pre-trained models without training from scratch.

---

## 6. Complete Conceptual Flow

```text
Raw text 
     ↓
Tokenization 
     ↓
Embeddings 
     ↓
Vector search / Comparison (Vector DB and indexes)
     ↓
Pre-trained model 
     ↓
Fine-tuning / Transfer Learning 
     ↓
Specific task
```

---

## 📌 Conclusion

* **Tokens**: The foundation of text processing.
* **Embeddings**: Dense vectors that capture semantic meaning.
* **Vector search and databases**: Allow finding semantic similarities quickly and scalably; indexes optimize approximate search.
* **Fine-tuning**: Specific adaptation of large models.
* **Transfer learning**: Reusing pre-trained knowledge.

> The combination of these concepts is the basis for building robust and efficient modern NLP systems.