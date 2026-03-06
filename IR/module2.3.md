# 📚 Comprehensive Exam Notes: Information Retrieval (Module 2.3)
> **Exam-Ready Consolidated Notes**
> **Topics:** Language Models · Query Likelihood · Smoothing · Relevance Models · Latent Semantic Indexing (LSI)

---

## 1. Language Models (LM) for IR: Foundations

Module 2.3 explores advanced retrieval frameworks that move beyond keyword matching, viewing retrieval as a generative process (LM) and utilizing linear algebra to capture hidden conceptual relationships (LSI).

### 1.1 The Paradigm Shift
* **Classical IR (VSM/BM25):** Discriminative models that ask *"How similar is document d to query q?"*.
* **Language Model IR:** Generative models that ask *"How likely is this document to have generated this query?"*.
* **The Generative Philosophy:** Each document ($d$) is viewed as having its own underlying Language Model ($M_d$).
* The system assumes a user has a "prototype" document in mind and generates a query by sampling words from that document's model.
* **Ranking Goal:** Documents are ranked by $P(q | M_d)$—the probability that the query $q$ was produced by the document model $M_d$.


### 1.2 Unigram Language Model
* The most widely used baseline is the **multinomial unigram model**.
* **Unigram Assumption:** Query terms are assumed to be generated independently from the document's word distribution.
* This approach is mathematically equivalent to a Multinomial Naïve Bayes classifier where documents are treated as the classes.

---

## 2. Query Likelihood Ranking

Query Likelihood is the most common implementation of the Language Model framework.

### 2.1 The Ranking Formula
* Under the unigram assumption, the probability of the query is the product of the probabilities of individual query terms ($q_i$) within the document:
    $$P(q | M_d) = \prod_{i=1}^{n} P(q_i | M_d)$$
* **Log-Space Computation:** To prevent numerical underflow (since probabilities are multiplied across many terms), calculations are done in log-space: $\sum_{i} \log P(q_i | M_d)$.

### 2.2 Bayesian Foundation
* Using Bayes' theorem: $P(d | q) = P(q | M_d) \times P(d) / P(q)$.
* $P(q)$ is constant across all documents, so it is ignored for ranking.
* The prior $P(d)$ is usually assumed uniform, meaning all documents are equally likely a priori.
* However, a genuine prior could encode external signals like Authority (PageRank), Recency, Document Length, or Read Count.

### 2.3 Maximum Likelihood Estimation (MLE)
* The simplest estimate for $P(q_i | M_d)$ is the Maximum Likelihood Estimate (MLE).
* This is the relative frequency of the term in the document: $P_{MLE}(t | M_d) = freq(t, d) / |d|$.

---

## 3. The Role of Smoothing

### 3.1 The Zero-Probability Problem
* A major technical challenge in LM is the data sparsity problem: if a single query term does not appear in a document, its probability $P(q_i | M_d)$ becomes zero.
* Because the formula relies on multiplication, a single zero causes the entire product to collapse to zero, completely excluding the document even if it is highly relevant overall.

### 3.2 Smoothing Mechanics
* **Purpose:** Smoothing solves the zero-probability problem by preventing zero probabilities and accounting for "unseen" terms.
* It does this by mixing the document model with a background collection model, $P(t | C)$.
* **Emergent IDF Effect:** Because terms common across the whole corpus have a high $P(t | C)$, smoothing naturally down-weights them, creating a formal, functional equivalent to IDF weighting.

### 3.3 Two Main Smoothing Techniques

| Technique | Formula & Mechanism | Key Effects |
| :--- | :--- | :--- |
| **Jelinek–Mercer (JM)** | Uses linear interpolation between the document probability and the collection-wide probability.<br> $P_{JM}(t \| d) = \lambda \times P_{MLE}(t \| d) + (1 - \lambda) \times P(t \| C)$. | $\lambda$ controls the mix. Close to 1 favors the document (good for verbose queries); close to 0 favors the collection (good for short queries). |
| **Dirichlet Prior** | Adds a Bayesian prior adjusting term counts based on collection frequency.<br> $P_{Dir}(t \| d) = (freq(t, d) + \mu \times P(t \| C)) / (\|d\| + \mu)$. | $\mu$ is the smoothing parameter. Automatically provides document-length normalization; short documents get pulled toward the collection model more strongly. |

---

## 4. Relevance Models & Pseudo-Relevance Feedback (PRF)

### 4.1 Motivation
* Queries are often short, and users frequently do not know the precise vocabulary used in the relevant documents.
* Relevance Models extend language modeling by estimating an expanded "query model" from the top-retrieved documents.

### 4.2 PRF Mechanism
* **Assumption:** The top-$k$ returned documents are assumed to be relevant.
* A joint distribution over terms and queries is estimated, assigning high weight to terms that frequently co-occur with query terms in those top documents.
* **Rocchio Algorithm (LM Perspective):** Expands the query by moving the query vector towards relevant documents and away from non-relevant ones using the formula: $q' = \alpha \times q + \beta \times \text{centroid}(Rel) - \gamma \times \text{centroid}(NRel)$.

---

## 5. Latent Semantic Indexing (LSI)

Latent Semantic Indexing (or Latent Semantic Analysis) addresses the fundamental vocabulary limitations of keyword-based matching models like the Vector Space Model (VSM).

### 5.1 The Vocabulary Mismatch Problem
VSM fails to capture deeper semantic patterns due to two main issues:
* **Synonymy:** Multiple words express the same concept (e.g., "car" and "automobile"). VSM treats them as separate dimensions, failing to find matches.
* **Polysemy:** One word has multiple meanings (e.g., "bank" as financial vs. riverbank). VSM overestimates similarity when documents use the same word in completely different contexts.


### 5.2 Singular Value Decomposition (SVD) Mechanics
LSI solves these problems by projecting both terms and documents into a lower-dimensional "concept space".
* **The Matrix:** It starts with a high-dimensional, sparse Term-Document Matrix ($A$ or $C$), where values are typically TF-IDF weights.
* **SVD Math:** Any matrix can be decomposed into $A = U \times \Sigma \times V^T$.
    * $U$: Term vectors in concept space.
    * $\Sigma$: Diagonal matrix of singular values (importance of each concept).
    * $V^T$: Document vectors in concept space.
* **Truncation:** LSI uses a rank-$k$ approximation ($A_k = U_k \times \Sigma_k \times V_k^T$), keeping only the top $k$ singular values (usually 50–300 dimensions).


### 5.3 LSI Outcomes
* **Concept Matching:** Instead of matching exact keywords, the system computes cosine similarity between the query and documents in this new $k$-dimensional latent concept space.
* **Synonymy Resolution:** Synonyms map to nearby points because they co-occur in similar contexts.
* **Improved Recall:** Documents sharing similar co-occurrence patterns are brought closer together, significantly improving Recall.
* **Weaknesses:** SVD is highly computationally expensive ($O(m \cdot n \cdot k)$), it lacks a formal probabilistic interpretation, and it is difficult to update incrementally when new documents arrive.

---

## 6. Model Comparison Summary

| Feature | Language Model (LM) | Latent Semantic Indexing (LSI) |
| :--- | :--- | :--- |
| **Foundation** | Probability Theory / Generative NLP | Linear Algebra / SVD |
| **Logic** | Ranks by likelihood of query generation | Ranks by similarity in latent concept space |
| **Handles Synonymy** | ❌ (Only with expansion/PRF) | ✅ (Solves synonymy inherently) |
| **Handles Polysemy** | ❌ | ✅ (Partial disambiguation) |
| **Key Advantage** | Principled statistical framework; handles IDF via smoothing | Discovers hidden semantic structure and noise reduction |
| **Complexity / Cost** | High (requires smoothing/estimation), but update cost is low | Very High (SVD is computationally expensive); high update cost requiring full re-SVD |

---

> **Exam Tips for Module 2.3:**
> * Understand that **Dirichlet smoothing automatically normalizes for document length**, while Jelinek-Mercer relies on a fixed linear mix.
> * Be prepared to explain the **zero-probability problem** caused by MLE and how smoothing fixes it by incorporating corpus-wide probabilities ($P(t|C)$).
> * Know the mathematical steps for LSI: generate term-document matrix, apply **SVD**, and **truncate to rank k**.
> * Be able to define **Synonymy** and **Polysemy** and explain how the reduced $k$-dimensional space groups contextually similar words together to solve these issues.