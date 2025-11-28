## Multilingual Document Summarization Engine

The application will be built on a **multilingual deep learning pipeline** designed to handle high-quality abstractive summarization for both English and Malay documents, focusing on robustness and semantic accuracy.

---

### 1. Preprocessing and Normalization

Preprocessing is tailored to address the unique linguistic characteristics of both languages, ensuring clean input for the downstream model.

<table>
<tr>
<td>

**Step**
</td>
<td>

**Rationale**
</td>
<td>

**Technical Implementation**
</td>
<td>

**Notes**
</td>
</tr>
<tr>
<td>

**Input Cleaning**
</td>
<td>Standardize text and remove noise.</td>
<td>Regex for punctuation, special character removal. HTML tag stripping.</td>
<td>Crucial for documents scraped from web sources.</td>
</tr>
<tr>
<td>

**Multilingual Tokenization**
</td>
<td>Split text into consistent subword units.</td>
<td>

Use **Byte Pair Encoding (BPE)** or **WordPiece** tokenizer from a multilingual model (e.g., XLM-R).
</td>
<td>Shared vocabulary handles both languages efficiently and minimizes out-of-vocabulary (OOV) tokens.</td>
</tr>
<tr>
<td>

**Language-Specific Standardization**
</td>
<td>Reduce morphological variation.</td>
<td>

**English:** Standard NLTK or spaCy Lemmatization. **Malay:** Use a dedicated **Indonesian/Malay Stemmer** (e.g., Nazief-Andi algorithm) due to its agglutinative nature.
</td>
<td>This improves the representation of base forms, particularly for Extractive methods, but is still beneficial for Abstractive models.</td>
</tr>
<tr>
<td>

**Document Truncation/Chunking**
</td>
<td>Manage long inputs for transformer limits.</td>
<td>

Documents exceeding the max sequence length ($L\_{max}$, typically 512 or 1024) must be truncated or processed via hierarchical summarization.
</td>
<td>Use a sliding window or a two-stage approach (Extractive pre-summary followed by Abstractive).</td>
</tr>
</table>

---

### 2. Multilingual Architecture & Code-Switching

The core of the system relies on a **Massively Multilingual Transformer** architecture.

#### Model Selection

* **Architecture:** **Encoder-Decoder (Sequence-to-Sequence)** model, like **mBART** or a fine-tuned version of **mT5 (Multilingual T5)**.
* **Rationale:** These models are pre-trained on huge multilingual corpora (e.g., **Common Crawl**) spanning over 50 languages, including English and Malay, allowing them to:
  * **Shared Representation:** Map linguistic features of both languages into a unified semantic space.
  * **Abstractive Capability:** Designed inherently for generative tasks like translation and summarization.

#### Code-Switching (CS) Strategy

CS is a major challenge in Malaysian text, mixing English and Malay within sentences (e.g., "The plan is very **_komprehensif_**").

* **Tolerance via Subwords:** The chosen BPE/WordPiece tokenizers are inherently **robust** to CS. Mixed-language words are broken down into known subword units (e.g., `komprehensif` might break into `kompreh`, `ensif`), which the pre-trained model can process contextually.
* **Fine-Tuning on CS Data:** For optimal performance, the model should be fine-tuned on a small, curated or synthetically generated **Code-Switched Malay-English corpus**. This explicitly teaches the model to handle the language shift in context.
* **Language Identification (Optional):** For complex cases, an initial fast language identification layer can tag sentences or chunks, which can be passed to the model as an explicit input feature or a prompt instruction.

---

### 3. Summarization Approach and Implementation

The selected approach is **Abstractive Summarization** to produce human-quality, novel summaries.

<table>
<tr>
<td>

**Implementation Detail**
</td>
<td>

**Technical Action**
</td>
<td>

**Justification**
</td>
</tr>
<tr>
<td>

**Core Model**
</td>
<td>

**Fine-Tuned mT5-large or XLM-R based Seq2Seq model.**
</td>
<td>Provides state-of-the-art abstractive quality and native multilingual support.</td>
</tr>
<tr>
<td>

**Training Data**
</td>
<td>Parallel corpus of (Source, Summary) pairs for both English (e.g., CNN/DailyMail) and Malay (e.g., local news archives).</td>
<td>Ensures the model learns high-quality summarization specific to the domain and language register.</td>
</tr>
<tr>
<td>

**Inference/Decoding**
</td>
<td>

Use **Beam Search** with **Length Penalty** and **Coverage Penalty**.
</td>
<td>

**Beam Search** finds the most probable sequence. Penalties control summary length and ensure diverse token generation, preventing repetition.
</td>
</tr>
<tr>
<td>

**Implementation Platform**
</td>
<td>

**Hugging Face Transformers library** on a **PyTorch** or **TensorFlow** backend.
</td>
<td>Provides standardized, high-performance implementations of all required architectures and easy deployment.</td>
</tr>
</table>

---

### 4. Evaluation Methodology

A comprehensive evaluation involves both automatic metrics for scalability and human assessment for quality validation.

#### Automatic Evaluation Metrics

<table>
<tr>
<td>

**Metric**
</td>
<td>

**Calculation Basis**
</td>
<td>

**Purpose**
</td>
</tr>
<tr>
<td>

**ROUGE-L**
</td>
<td>Longest Common Subsequence (LCS)</td>
<td>

Primary metric. Measures summary _informativeness_ and _fluency_ by capturing sentence-level structure overlap.
</td>
</tr>
<tr>
<td>

**BERTScore**
</td>
<td>Cosine Similarity of BERT Embeddings</td>
<td>

Measures **semantic similarity** (meaning) rather than simple word overlap. Crucial for abstractive summaries where wording differs from the reference.
</td>
</tr>
<tr>
<td>

**CHRF**
</td>
<td>Character n-gram F-score</td>
<td>Good metric for agglutinative languages like Malay as it focuses on character overlap, making it more resilient to differing word forms.</td>
</tr>
</table>


#### Human Evaluation Protocol

A panel of human evaluators (native/fluent in both languages) will rate a statistically significant sample of summaries across three key dimensions using a 1-5 Likert scale:

1. **Factuality:** Is the summary consistent with the source document (no hallucinated information)?
2. **Fluency/Grammar:** Is the language smooth and grammatically correct in English or Malay?
3. **Coherence/Focus:** Does the summary flow logically and cover the main theme of the document?