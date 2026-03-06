# 📚 Comprehensive Exam Notes: Information Retrieval (Module 2.2)
> **Exam-Ready Consolidated Notes**
> **Topics:** Probabilistic Model · BIM · IR as Classification · BM25 · Language Models · LSI

---

## 1. The Probabilistic Model Foundation

The probabilistic model shifts the retrieval paradigm from geometric similarity to estimating the probability of relevance. It is built on the premise that retrieval is inherently uncertain.

### 1.1 The Probabilistic Ranking Principle (PRP)
* Developed by Robertson, the PRP provides the logical basis for this model.
* **Core Principle:** To maximize an IR system's effectiveness, documents should be ranked in decreasing order of their probability of relevance to the query.
* **Mathematical Goal:** Estimate `P(R=1 | D=d, Q=q)`, where `R` is a binary variable (`1` for relevant, `0` for non-relevant).


### 1.2 Derivation via Bayes' Theorem
* Applying Bayes' theorem: `P(R=1 | d, q) = P(d | R=1, q) × P(R=1 | q) / P(d | q)`.
* To make calculations efficient and avoid numeric underflow, this is transformed into a log-odds ratio: `log[ P(R=1 | d, q) / P(R=0 | d, q) ]`.
* By cancelling constants, the core task reduces to estimating: `log[ P(d | R=1, q) / P(d | R=0, q) ]`.

---

## 2. Binary Independence Model (BIM)

The BIM is the classic implementation of the probabilistic framework.

### 2.1 Key Assumptions
1. **Binary Assumption:** Documents and queries are represented as binary vectors (term presence/absence), ignoring raw term frequency.
2. **Independence Assumption:** Terms are assumed to occur independently in documents given their relevance status (similar to Naïve Bayes).
3. Only **query terms** contribute to relevance; non-query terms are ignored.

### 2.2 Term Weighting
* The log-odds form sums term-specific weights: `Score(d, q) = Σ_{t ∈ q} log[ P(t=1 | R=1) / P(t=1 | R=0) ]`.
* If no relevance judgments are available, assume `P(t=1|R=1) = 0.5` and estimate `P(t=1|R=0)` from document frequency (creating an IDF-like weight).
* If relevance judgments are available, use occurrence counts in relevant vs. non-relevant sets with Laplace smoothing to prevent zero probabilities.

---

## 3. Information Retrieval as Classification

In this framework, Information Retrieval is interpreted as a binary decision task where each document is assigned a label: Relevant (`R=1`) or Non-Relevant (`R=0`).


### 3.1 The Supervised Learning View
* **Task:** The system learns a decision function `f(d, q)` that predicts whether a document belongs to class `R=1` or `R=0`.
* **Feature Vectors:** Documents and queries are represented as vectors containing signals like term frequencies, TF-IDF weights, BM25 scores, or semantic similarity scores.
* **Probability Threshold:** A document is classified as relevant if the estimated probability exceeds a specific threshold.
* Standard classifiers used include Logistic Regression, SVMs, Decision Trees, and Neural Networks.

### 3.2 Retrieval vs. Standard Classification
* **Data Imbalance:** Non-relevant documents vastly outnumber relevant ones, leading to highly skewed class distributions.
* **Query Dependence:** Relevance is relational; a document’s "class" changes depending on the specific query issued.
* **Evaluation Focus:** IR aims to optimize ranking quality (e.g., Precision at k, MAP, NDCG) instead of overall error.
* **Soft Classification:** Ranking is a "soft" generalization of classification, ordering documents by probability instead of using a hard threshold.

---

## 4. The BM25 Ranking Algorithm

BM25 (Best Matching 25) evolved from the Okapi system and is the industry standard for keyword-based search. It is an improved version of TF-IDF.


### 4.1 Core Components
1. **Term Frequency (TF):** Measures how often a query term appears. BM25 uses *TF Saturation*, meaning beyond a certain point, more occurrences provide diminishing returns.
2. **Inverse Document Frequency (IDF):** Measures the rarity of a term across the corpus. Rare terms are weighted higher.
3. **Document Length Normalization:** Prevents longer documents from being unfairly favored by adjusting scores based on how the document's length compares to the average document length (`avgdl`).

### 4.2 The BM25 Formula
For a query `q` and document `d`, the score is the sum of contributions from each query term `t`:

`Score(q, d) = Σ_{t ∈ q} IDF(t) × [ freq(t, d) × (k1 + 1) ] / [ freq(t, d) + k1 × (1 - b + b × (|d| / avgdl)) ]`

* `k1` (1.2–2.0): Controls term frequency scaling (saturation).
* `b` (0.75): Controls the strength of document length normalization.
* `|d|`: Length of document `d`.
* `avgdl`: Average length of all documents in the corpus.

### 4.3 Why BM25 is Effective
* **Non-Linear TF:** Recognizes that a term appearing 100 times is not 100 times more relevant than a term appearing once.
* **Fairness:** Corrects for document length, ensuring concise documents can outrank long ones that just happen to mention a keyword more.

---

## 5. Language Models & Query Likelihood

### 5.1 Paradigm Shift
Instead of asking "Is this document relevant?", Language Models (LM) ask: *"How likely is it that this document's language model generated this query?"*. Rank documents by `P(q | d)`.

### 5.2 Query Likelihood Model
* Each document `d` is treated as a language model `M_d`.
* **Unigram assumption:** Query terms are generated independently: `P(q | M_d) = Π_{i} P(q_i | M_d)`.
* **Zero-Probability Problem:** If a term doesn't appear in `d`, `P(t | M_d) = 0`, making the whole product zero.

### 5.3 Smoothing Techniques
Smoothing mixes the document model with a background collection model `P(t | C)` to prevent zero probabilities and naturally down-weight common terms (IDF effect).
* **Jelinek–Mercer:** Linear interpolation; uses parameter `λ` to control the mix.
* **Dirichlet Prior:** Bayesian prior; uses parameter `μ` to control the influence of the collection.

---

## 6. Latent Semantic Indexing (LSI)

### 6.1 Motivation
Boolean and Vector Space models suffer from vocabulary problems:
* **Synonymy:** Different words mean the same thing.
* **Polysemy:** Same word has multiple meanings.
LSI addresses this by discovering latent semantic structure in term-document co-occurrence patterns.


### 6.2 Singular Value Decomposition (SVD)
LSI applies Truncated SVD to the term-document matrix `A`:
`A ≈ U_k × Σ_k × V_k^T`
* Reduces dimensionality to `k` latent dimensions (typically 50-300).
* Synonyms cluster in the same latent dimension, and ambiguous terms are separated by context.
* **Limitations:** SVD is computationally expensive, has no probabilistic interpretation, and is hard to update incrementally.

---

## 7. Model Comparison Table

| Model | Representation | Ranking Function | Handles TF? | Length Norm? | Theoretical Basis |
|---|---|---|---|---|---|
| **Boolean** | Set of terms | Exact match | ❌ | ❌ | Set theory |
| **Vector Space** | TF-IDF vectors | Cosine similarity | ✅ | Via cosine | Linear algebra |
| **BIM** | Binary term vectors | Log-odds ratio | ❌ | ❌ | Probability theory |
| **BM25** | Term counts | Saturated TF × IDF | ✅ | ✅ (param b) | Probabilistic IR |
| **Language Model**| Term probabilities | P(q \| M_d) | ✅ | Via smoothing | Statistical NLP |
| **LSI** | SVD latent vectors | Cosine in latent space | ✅ | Implicit | Linear algebra / SVD |

*(Table adapted from:)*