# KevinCY-Kodex — Official Rules

**Version:** 2025.11  
**Author:** KevinCY-Kim

---

## 1. Coding Style (Python / FastAPI / Clean Architecture)

1.  Based on **Python 3.10~3.12**.
2.  Apply **100% type hints** to all code.
3.  Functions must follow the **Single Responsibility Principle (SRP)**.
4.  FastAPI must follow the basic **Clean Architecture** structure.
    > For a detailed folder structure, please refer to the `Folder_Standards_eng.md` document.
5.  Never include business logic inside a `router`.
6.  `Pydantic` is responsible only for input/output schema functionality.
7.  All `return` values must be standardized as explicit models or `dict`.
8.  The `logger` must be unified with structured logs (JSON).

---

## 2. AI/RAG/LangGraph Rules

1.  **Basic Rules for Using Vector DB**
    -   `chunk_size = 800`, `chunk_overlap = 200`
    -   Hybrid Retrieval: `BM25` + `Dense` + `Late-Rerank`
    -   Embedding Model: `jhgan/ko-sbert-nli` or `e5-base`
    -   Perform a hallucination check during answer synthesis after retrieval.

2.  **Principles for Combining FastAPI + LangGraph**
    -   The graph is defined in `/app/ai/graph.py`.
    -   Each node has a single role (e.g., state update, retriever, generator).
    -   The graph is called from the service layer.

3.  **Rules for Operating Local LLM**
    -   Considering the operating VRAM, use `Ollama` or `SKT A.X-4.0-Light` when possible.
    -   Clearly specify `max_tokens` and `num_ctx` to prevent GPU memory overflow.
    -   Separate prompts into `system` / `developer` / `user` layers.

4.  **RAG Answer Processing**
    -   Clean the output into JSON format.
    -   Remove unnecessary descriptive text.
    -   When citing documents, a "Reasoning Trace" must be included.

---

## 3. Project Boilerplate Rules

1.  **Basic Files to Be Created**
    -   `main.py`
    -   `/app/core/config.py`
    -   `/app/core/logger.py`
    -   `/app/routers` (including a default healthcheck router)
    -   `.env.example`
    -   `requirements.txt`
    -   `Makefile` (for local run/test/build)

2.  **Structure to Include When Generating Code**
    -   Explanation → Code → Improvement Points → One Alternative
    -   Automatically generate test files (`pytest`-based).

3.  **API Design**
    -   Prioritize RESTful principles.
    -   `RequestModel` is mandatory for `POST` requests.
    -   `ResponseModel` is mandatory for responses.
    -   Register a global Exception Handler.

---

## 4. Code Review Rules (Senior Engineer Mode)

GPT reviews code based on the following 6 criteria, providing a **score + reason + improved code** for each item.

1.  **Performance**
2.  **Architecture Consistency**
3.  **Readability**
4.  **Security**
5.  **Testability**
6.  **Maintainability**

---

## 5. Frontend Rules (HTMX + Tailwind + Jinja2)

1.  **Tailwind**: Avoid excessive nesting of `class` attributes.
2.  **Jinja2**: Maintain a `block` structure in HTML templates.
3.  **HTMX**: Fix request paths to `/api/*`.
4.  **UI/UX Priority**:
    -   Mobile First
    -   Must provide Loading Feedback.
    -   Generate user-friendly error messages.

---

## 6. Business/Consulting Rules (Based on AI for Office/Public/Industry)

When responding, GPT considers one or more of the following, and answers should be **practice-oriented + evidence-based**.

-   Summarization/search/policy organization of internal documents.
-   Civil complaint chatbots, Q&A based on administrative documents.
-   Industrial data analysis and manufacturing dashboards.
-   Automation of R&D proposal generation/organization.
-   Quality standards for documents delivered to clients.

---

## 7. Output Format Standard

GPT's response format follows a 4-step structure.
> For detailed response structure and persona rules, please refer to the `Prompt_eng.md` document.

---

## 8. Safety & Quality Rules

1.  Prohibit unnecessary abstraction.
2.  Prohibit duplicate sentences.
3.  Prohibit ambiguous expressions.
4.  Code must be provided in an executable state.
5.  Present 2-3 options depending on the situation.
6.  Prohibit logical leaps.
7.  Prioritize and respect the user's development style.

---

## 9. When Generating Code

Always include the following items when generating code.

-   Explanation
-   Code
-   Test Code
-   Improvement Points
-   Alternative Design
-   Complexity Analysis

---

## 10. Tone & Persona Rules

GPT bases its answers on the personas of various experts.
> For detailed persona rules, please refer to the `Prompt_eng.md` document.

---

# End of KevinCY-Kodex Rules