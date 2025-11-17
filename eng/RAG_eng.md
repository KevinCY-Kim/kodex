# KevinCY-Kodex — RAG Guideline

**Version:** 2025.11  
**Scope:** Local LLM · LangGraph · BM25 + Dense Hybrid Retrieval · Public/Office Document QA

---

## 1. RAG Overview

RAG (Retrieval-Augmented Generation) consists of the following 4 stages.

1.  **Ingestion**: Document collection and preprocessing.
2.  **Chunking & Embedding**: Document splitting and vectorization.
3.  **Retrieval**: Searching (BM25 + Dense + Rerank).
4.  **Answer Generation & Post-processing**: LLM-based response generation and post-processing.

All stages are designed as independent modules and are orchestrated in the `Service` layer.

---

## 2. RAG Architecture Structure

The folder structure for RAG-related modules follows the project-wide standard. Each stage must adhere to the **Single Responsibility Principle (SRP)**.
> For a detailed AI-related folder structure, please refer to the `Folder_Standards_eng.md` document.

---

## 3. Ingestion Rules

### 3.1 Document Types
-   PDF, HWP (based on parsing API), DOCX
-   HTML, JSON, CSV
-   Public institution documents

### 3.2 Basic Ingestion Rules
-   A **cleaning** process must be performed after text extraction.
    -   Remove unnecessary items: table of contents, page numbers, repeating headers/footers, ads, etc.
-   Add document **metadata**.
    -   `source`: The origin of the document.
    -   `type`: The type of the document.
    -   `section`: Section information.
    -   `date`: Creation date.

### 3.3 Paju City Project Specific Preprocessing
-   Mentions of "other municipalities" are detected and commented out or stored in a separate field.
-   Phone numbers, department names, etc., are stored in the format `meta={"contact": "...", "department": "..."}` for utilization.

---

## 4. Chunking Strategy

### 4.1 Basic Chunk Rules
-   Default `chunk_size`: 800
-   `overlap`: 200
-   Can be flexibly adjusted according to document size.

### 4.2 Chunk Creation Principles
-   Prioritize semantic unit-based segmentation (paragraph, table, section-centric).
-   Manage very long tables/laws as a separate chunk type.
-   Prioritize noise removal for text from `Whisper` or `OCR`.

### 4.3 Chunk Storage Format
-   A single `chunk` has the following fields: `id`, `text`, `source`, `section`, `page`(optional), `embedding`(vector).

---

## 5. Embedding Rules

### 5.1 Default Models
-   `jhgan/ko-sbert-nli` (Optimized for Korean document RAG)
-   `intfloat/e5-base`
-   If multilingual support is needed: `multilingual-e5-base`

### 5.2 Embedding Policy
-   All `chunks` are embedded after string preprocessing.
-   Remove or split `chunks` that are too short or too long.

### 5.3 Storage
-   Use local `FAISS` (GPU/CPU) by default.
-   Manage ID-to-metadata mapping in a separate DB or JSONL file.

---

## 6. Retrieval Rules

### 6.1 Hybrid Retrieval
1.  **Stage 1**: `BM25` keyword search.
2.  **Stage 2**: `Dense` vector search.
3.  **Stage 3**: Combine the two results to generate Top-N candidates.

### 6.2 Reranking
-   Apply `Cross-Encoder` or LLM-based `rerank`.
-   Refine from Top-N (e.g., 20) to Top-K (3-5) final candidates.
-   Include the Rerank score in a `score` field.

### 6.3 Retrieval Quality Rules
-   If documents from other municipalities are mixed in the search results, apply adjustment rules by giving more weight to keywords like "Paju City", "welfare", etc.
-   Deprioritize or exclude duplicate or unclearly sourced `chunks`.

---

## 7. Answer Generation Rules

### 7.1 Prompt Policy
-   **System Prompt**: Assign the role of a "Document-based Expert".
-   Generate the answer by feeding the user's question and selected `chunks` as context.
-   Prohibit speculation on information outside the context and instruct to include supporting sentences.

### 7.2 Output Format
-   `answer`: The final response.
-   `sources`: A list of referenced `chunk` IDs.
-   `reasoning`(optional): The basis for selection (for internal use).
-   `followup_questions`(optional): Questions the user might ask next.

### 7.3 Government/Public Document Response Rules
-   Prioritize document-based answers and include a warning if timeliness cannot be guaranteed.
-   Explain difficult administrative terms with examples.

---

## 8. Post-Processing Rules

### 8.1 Text Refinement
-   Make sentences concise by removing duplicate sentences and reducing unnecessary conjunctions.
-   Reconstruct naturally based on "sentences to be read by the user."

### 8.2 Paju City Specific Post-processing
-   Filter out sentences related to other municipalities.
-   Automatically add "official contact information" for inquiries about systems/subsidies.

### 8.3 Example JSON Format

The final response is recommended to follow a structure like this:
```json
{
  "answer": "...",
  "sources": ["paju_2024_welfare_01#chunk_021", "paju_guideline#chunk_004"],
  "followup": ["What are the related documents?", "Is there an online page where I can apply?"]
}
```

---

## 9. LangGraph Workflow Rules

### 9.1 RAG Pipeline Graph Configuration
-   **Required Nodes**: `preprocess_node`, `retrieval_node`, `rerank_node`, `generation_node`, `postprocess_node`

### 9.2 State Structure
-   `query`, `user_meta`(optional), `retrieved_docs`, `reranked_docs`, `answer`, `sources`

### 9.3 Error Handling
-   If `retrieval` fails, return an empty list.
-   If no `chunks` are found, generate a "No evidence in documents" message.
-   In case of a system error, generate a `fallback` response.

---

## 10. Evaluation Rules

### 10.1 Quality Metrics
-   `Recall@K`, `Precision@K`
-   Rerank performance
-   `Answer Faithfulness` (adherence to document evidence)
-   `Hallucination score`

### 10.2 Automated Testing
-   Compare against a `Known-answer` baseline.
-   Check for `Critical keyword` coverage.
-   Verify `Chunk alignment`.

---

## 11. Production Rules

### 11.1 Index Versioning
-   Apply the `index_version=YYYYMMDD_vX` rule for each `ingest`.
-   Remove old indexes, and re-embed the entire document upon modification.

### 11.2 Monitoring
-   `Retrieval hit ratio`
-   `LLM latency`
-   `STT/TTS pipeline latency` (for voice RAG)
-   Question volume per user section.

### 11.3 Data Security
-   Exclude `chunks` with sensitive information from embedding.
-   Store phone numbers, social security numbers, etc., after masking.

End of KevinCY-Kodex RAG Guideline