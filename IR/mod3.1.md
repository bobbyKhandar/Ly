# 📚 Comprehensive Exam Notes: Information Retrieval (Module 3.1)
> **Experimental Evaluation & Performance Metrics**
> **Topics:** Contingency Table · Precision · Recall · F-Measure · MAP · NDCG

---

## 1. Foundation of IR Evaluation

Evaluation in IR is an empirical discipline that uses standardized methods and test collections to assess retrieval quality. It ultimately answers the question: "How good is this system?".

### 1.1 The Evaluation Context (Cranfield Paradigm)
To perform an objective evaluation, an IR system requires three essential components:
* **Document Collection:** A static, fixed set of documents to be searched against.
* **Test Queries (Information Needs):** A set of topics or questions representing user search requirements.
* **Relevance Judgments:** A "gold standard" ground truth, usually human-labeled, marking which documents are relevant (R=1) or non-relevant (R=0) for each query. Relevance is assessed against the user's information need, not just keyword presence.

### 1.2 Retrieval as a Set-Based Task
Evaluation compares two primary sets of documents:
* **Set R (Relevant Documents):** The absolute set of documents in the collection that truly satisfy the information need.
* **Set A (Retrieved Documents):** The actual set of documents returned by the system in response to a query.

---

## 2. Core Terminology & The Contingency Table

Before calculating metrics, documents must be categorized into a 2x2 contingency table (Confusion Matrix) based on the system's output versus the ground truth. All classical IR metrics derive from this table.


| System Output | Relevant (Truth) | Non-Relevant (Truth) | Total |
| :--- | :--- | :--- | :--- |
| **Retrieved** | **True Positive (TP):** Relevant items correctly returned. | **False Positive (FP):** Irrelevant items incorrectly returned (noise / Type I Error). | A (total retrieved) |
| **Not Retrieved** | **False Negative (FN):** Relevant items missed by the system (Type II Error). | **True Negative (TN):** Irrelevant items correctly ignored. | N - A |
| **Total** | R (total relevant) | N - R | N (collection size) |

---

## 3. Precision: Measuring Accuracy

Precision evaluates the "purity" or quality of the retrieved set. It asks: "Of all the documents retrieved, what proportion is actually relevant?".


### 3.1 Formula and Interpretation
* **Formula:** $Precision = \frac{|R \cap A|}{|A|} = \frac{TP}{TP + FP}$
* **Focus:** Minimizing "noise" or false positives.
* **Application:** Critical for users who want a few high-quality results immediately (e.g., standard web search users who only read the top 5 results).
* **Range:** $0$ to $1$ (higher is better).

---

## 4. Recall: Measuring Completeness

Recall evaluates the "coverage" or completeness of the system. It asks: "Of all the relevant documents that exist, what proportion did the system find?".

### 4.1 Formula and Interpretation
* **Formula:** $Recall = \frac{|R \cap A|}{|R|} = \frac{TP}{TP + FN}$
* **Focus:** Minimizing misses or false negatives.
* **Application:** Crucial for legal discovery, medical research, or patent law, where missing even one relevant document is highly costly.
* **Range:** $0$ to $1$ (higher is better).

---

## 5. The Precision-Recall Trade-off

There is a fundamental inverse relationship between precision and recall in practice.


* **Broad Retrieval:** Retrieving more documents increases Recall (finding more relevant ones) but decreases Precision as the system starts including less relevant noise.
* **Narrow Retrieval:** Restricting results to only the most confident matches increases Precision but decreases Recall because many relevant items are left behind.

---

## 6. F-Measure: The Balanced Metric

The F-Measure (or F1-Score) provides a single value by computing the harmonic mean of Precision and Recall.

### 6.1 The F1 Formula
* **Formula:** $F1 = 2 \times \frac{Precision \times Recall}{Precision + Recall}$
* **Logic of the Harmonic Mean:** The harmonic mean is used because it penalizes extreme imbalances between P and R. If a system retrieves everything (Recall = 1.0) but includes massive noise (Precision = 0.01), the arithmetic mean (0.505) would be misleadingly high, whereas the F1 score (0.0198) correctly identifies the system as ineffective.

### 6.2 Generalized F-beta ($F_\beta$)
Systems can be tuned to prioritize one metric over the other using a $\beta$ weight:
* **Formula:** $F_\beta = \frac{(1 + \beta^2) \times P \times R}{(\beta^2 \times P) + R}$
* **$\beta = 1$:** Precision and Recall are equally important (Standard F1).
* **$\beta > 1$ (e.g., $\beta = 2$):** Recall is more important (e.g., security screening, legal/medical search).
* **$\beta < 1$ (e.g., $\beta = 0.5$):** Precision is more important (e.g., auto-complete suggestions, web search).

---

## 7. Advanced Ranking Metrics (Ranked Retrieval)

### 7.1 Precision at K (P@K)
* Measures precision considering only the top-K ranked documents.
* **Formula:** $P@K = \frac{\text{Relevant documents in top-K positions}}{K}$.
* It is a user-centric metric that does not penalize the system for failing to retrieve relevant documents outside the top K.

### 7.2 Average Precision (AP) & Mean Average Precision (MAP)
* **Average Precision (AP):** Evaluates a ranked list by computing precision at every rank position where a relevant document is retrieved, then averaging those values.
* **Mean Average Precision (MAP):** The mean of AP scores over a set of queries. It is the standard single-number summary of ranked retrieval performance across multiple queries.

### 7.3 11-Point Interpolated Precision
* Evaluates precision at 11 standard recall levels: 0.0, 0.1, 0.2, ..., 1.0.
* **Interpolation Formula:** $P_{interp}(r) = \max\{P(r') : r' \geq r\}$.
* This "right-filling" ensures the Precision-Recall curve is monotonically non-increasing, smoothing out the jagged saw-tooth pattern of a raw P-R curve.

### 7.4 Normalized Discounted Cumulative Gain (NDCG)
* **Motivation:** Handles graded relevance (e.g., highly relevant vs. marginally relevant) rather than just binary relevance, and applies a logarithmic discount to reflect the diminishing value of documents at lower ranks.
* **Discounted Cumulative Gain (DCG@k):** $\sum_{i=1}^{k} \frac{rel_i}{\log_2(i + 1)}$.
* **Ideal DCG (IDCG@k):** The DCG obtained by a perfect ranking (all documents sorted by decreasing relevance grade).
* **NDCG@k:** $\frac{DCG@k}{IDCG@k}$ (Produces a score between 0.0 and 1.0).

---

## 8. Evaluation Approaches: Offline vs. Online

| Aspect | Offline Evaluation | Online Evaluation |
| :--- | :--- | :--- |
| **Method** | Test collections + relevance judgments | Live user interaction data |
| **Example** | TREC, CLEF, NTCIR benchmarks | A/B testing, click-through rates, session time |
| **Metrics** | Precision, Recall, MAP, NDCG | CTR, dwell time, abandonment rate, task success |
| **Pros** | Reproducible, controlled, no user needed | Measures real-world user satisfaction |
| **Cons** | Relevance judgments are expensive; pool bias | Noisy signals; ethical concerns; slow feedback |

---

## 9. Exam Practice Scenario (Numerical)

**Problem Setup:** A database has 20 relevant documents in total ($R=20$). A search returns 12 results ($A=12$), of which 8 are relevant ($TP=8$).

**Contingency Matrix Breakdown:**
* **True Positives (TP):** $8$ (Relevant and Retrieved).
* **False Positives (FP):** $12 - 8 = 4$ (Retrieved but Not Relevant).
* **False Negatives (FN):** $20 - 8 = 12$ (Relevant but Not Retrieved).

**Calculations:**
* **Precision:** $8 / 12 = 0.667$ (66.7%).
* **Recall:** $8 / 20 = 0.40$ (40%).
* **F1 Score:** $2 \times \frac{0.667 \times 0.40}{0.667 + 0.40} = \frac{0.5336}{1.067} \approx 0.50$ (50%).