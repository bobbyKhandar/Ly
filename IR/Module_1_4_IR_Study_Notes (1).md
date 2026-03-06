# Module 1.4 — Document Parsing and Text Processing

> **Syllabus Coverage:** Document Parsing Pipeline · Text Normalization · Tokenization · Stop-word Removal · Stemming vs. Lemmatization · Phrases · N-grams · Pointwise Mutual Information (PMI)

---

## 1. Overview of the Parsing Pipeline

Raw text is unstructured, noisy, and redundant. Before any document can be indexed or any query can be matched, both must be parsed into a clean, normalized, and compact representation. This pipeline is called **Document Parsing** or **Text Preprocessing**.


```mermaid
flowchart LR
    A[Raw Document\nor Query] --> B[Normalization]
    B --> C[Tokenization]
    C --> D[Stop-word\nRemoval]
    D --> E[Stemming /\nLemmatization]
    E --> F[Phrase / N-gram\nDetection]
    F --> G[Index-Ready\nTerms]
```

> **The Rule of Symmetry (Critical Exam Point):** The **exact same pipeline** must be applied to documents during indexing AND to queries at search time. Any asymmetry causes the system to miss valid matches.

---

## 2. Text Normalization

Normalization standardizes textual data to ensure consistency across the corpus, preventing the system from treating identical concepts as different terms due to superficial formatting.

- **Format Conversion:** Transforming various file types (PDF, Word, HTML) into a uniform text structure by removing tags, scripts, and layout noise.
- **Case Folding:** Converting all text to lowercase (e.g., merging `"Data"` and `"data"` into the same token).
- **Punctuation Removal:** Stripping symbols, extra spaces, and marks unless they contribute directly to meaning.

---

## 3. Tokenization

**Definition:** The foundational process of breaking a continuous stream of text into individual, manageable atomic units called **tokens**.

### 3.1 Types of Tokenization

| Type | How it Works | Primary Use Case |
| :--- | :--- | :--- |
| **Word Tokenization** | Segments text by whitespace and delimiters into individual words. | Standard IR indexing. |
| **Sentence Tokenization** | Divides text based on punctuation (periods, exclamation marks). | Summarization and context analysis. |
| **Character Tokenization** | Breaks text into individual characters. | Fine-grained analysis; handling specific morphologically rich languages. |

### 3.2 Technical Challenges (The Hard Cases)

Real text contains edge cases that require deliberate parsing rules:
- **Hyphenated Words:** Determining if `"state-of-the-art"` should be indexed as one single token or four separate tokens.
- **Contractions & Apostrophes:** Deciding how to handle `"don't"` (split into `"do"` + `"not"`, or keep as `"dont"`) and names like `"O'Brien"`.
- **Special Strings:** Correctly identifying URLs, email addresses, numbers (`3.14`), and currency symbols so they are not inappropriately broken apart.

---

## 4. Stop-word Removal (Stopping)

**Definition:** Removing frequently occurring words that carry minimal semantic value (e.g., `"the"`, `"is"`, `"and"`, `"to"`).

- **Objective:** To drastically reduce index size (stopwords make up 30-50% of English text) and improve system efficiency by focusing only on "content" words.
- **Methodology:** Systems typically use a predefined, language-specific Stop List.
- **The Critical Trade-off:** While it saves space, removing stop-words can completely destroy phrase-based queries.
  - *Example:* The query `"to be or not to be"` consists entirely of stop-words. If removed, the query becomes empty.
  - *Modern Solution:* Rather than hard-deleting them, modern systems keep stopwords in a positional index for exact phrase matching, but rely on **IDF weighting** to naturally suppress their importance in standard keyword queries.

---

## 5. Stemming vs. Lemmatization

Both processes normalize word variations (inflections/derivations) to a single base form to improve **Recall** (ensuring `"running"` and `"runner"` both match a query for `"run"`).

### 5.1 Definitive Comparison

| Feature | Stemming | Lemmatization |
| :--- | :--- | :--- |
| **Approach** | **Heuristic:** Suffix stripping rules (chopping). | **Linguistic:** Dictionary-based morphological analysis. |
| **Accuracy** | Approximate; often produces non-words (e.g., `"computation"` -> `"comput"`). | High; always returns valid dictionary base words. |
| **Speed** | Very fast and computationally inexpensive. | Slower; requires context and part-of-speech knowledge. |
| **Example** | `"better"` -> `"better"` | `"better"` -> `"good"` |
| **Usage** | Standard in Search Engines (e.g., **Porter Stemmer**). | Standard in NLP pipelines. |


### 5.2 Stemming Errors

Algorithms like the industry-standard **Porter Stemmer** are fast but prone to errors:
- **Over-stemming:** Merging completely unrelated words (e.g., `"university"` and `"universe"` both becoming `"univers"`). *This negatively impacts Precision.*
- **Under-stemming:** Failing to merge related words (e.g., `"alumnus"` and `"alumni"` remaining separate). *This negatively impacts Recall.*

---

## 6. Phrase Processing and N-grams

Retrieval systems perform vastly better when they recognize that a sequence of words constitutes a single meaningful unit. Treating `"New York"` as a single phrase prevents false matches for `"New"` and `"York"` independently.

### 6.1 N-grams

An **N-gram** is a contiguous sequence of `n` consecutive items (characters or words) extracted from a text.

- **Unigram (1-gram):** `"information"`
- **Bigram (2-gram):** `"data mining"`
- **Trigram (3-gram):** `"artificial neural network"`

**Mathematical Formula:** The maximum number of unique character-level N-grams for an alphabet of size `λ` is: **λ^n**

### 6.2 Applications of N-grams

| N-gram Type | Primary Applications |
| :--- | :--- |
| **Word N-grams** | Phrase indexing, predictive text, and language modeling. |
| **Character N-grams** | Operating within a word (e.g., trigrams of `"retrieval"` -> `"ret"`, `"etr"`, `"tri"`). Used heavily for **spelling error detection**, fuzzy matching, and wildcard queries. |

### 6.3 Statistical Phrase Detection: Pointwise Mutual Information (PMI)

To determine if two words form a meaningful phrase rather than just appearing next to each other by chance, systems calculate the Pointwise Mutual Information (PMI).

**PMI Formula:**
`PMI(w1, w2) = log2 ( P(w1, w2) / (P(w1) * P(w2)) )`

- **PMI > 0:** Strong association. The words co-occur more than expected by chance (likely a meaningful phrase like `"machine learning"`).
- **PMI ≈ 0:** Words occur independently.
- **PMI < 0:** Weak association (unlikely to be a phrase).

---

## 7. Full Pipeline — Worked End-to-End Example

**Raw Document:** `"Dr. Smith's Information Retrieval systems are running efficiently."`

1. **Normalization:** Remove formatting, lowercasing.
   `"dr. smith's information retrieval systems are running efficiently."`
2. **Tokenization:** Split on whitespace/punctuation, handle apostrophes.
   `["dr", "smith", "s", "information", "retrieval", "systems", "are", "running", "efficiently"]`
3. **Stopword Removal:** Remove `"s"` and `"are"`.
   `["dr", "smith", "information", "retrieval", "systems", "running", "efficiently"]`
4. **Stemming (Porter Stemmer):** Strip suffixes.
   `["dr", "smith", "inform", "retriev", "system", "run", "effici"]`
5. **N-gram / Phrase Extraction (Bigrams):** `["dr smith", "smith inform", "inform retriev", "retriev system", "system run", "run effici"]`

*Result: Both the individual stemmed tokens and the bigrams are sent to the index.*

---

## 8. 🎯 Exam-Focused Summary

- **Tokenization Challenges:** Be prepared to list the hard edge cases (hyphens, apostrophes, URLs, numbers).
- **Stop-words Trade-off:** Removing them saves index space and speeds up searches, but risks breaking phrase queries like `"The Who"`. 
- **Stemming vs. Lemmatization:** Stemming is fast/rule-based (Porter Stemmer) but creates non-words; Lemmatization is slow/dictionary-based but linguistically accurate.
- **Stemming Errors:** Over-stemming hurts precision; under-stemming hurts recall.
- **PMI (Pointwise Mutual Information):** Understand that PMI > 0 indicates two words are statistically linked enough to be considered a phrase.
- **Character N-grams:** Memorize that character-level N-grams (not word-level) are the primary tool for spelling correction and fuzzy matching.
- **Symmetry:** Document and query preprocessing must be perfectly symmetrical.