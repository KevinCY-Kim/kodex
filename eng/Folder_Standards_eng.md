# KevinCY-Kodex — Folder Standards

**Version:** 2025.11  
**Scope:** FastAPI · LangGraph · RAG · STT/TTS · Web(HTMX+Tailwind)

---

## 1. Overview

This document defines the standard rules for the project's file and folder structure.
The project is composed of the following core layers, based on Clean Architecture principles.

-   `core`: Core settings, logger, security, etc.
-   `routers`: API endpoints
-   `services`: Business logic
-   `repositories`: Database, files, external APIs, etc. (IO)
-   `models`: Pydantic schemas
-   `ai`: LLM, RAG, LangGraph, etc. (AI Engine)
-   `utils`: Common utilities
-   `templates` & `static`: Web frontend

Each layer must have a **Single Responsibility (SRP)** and adhere to the inter-layer dependency direction.

---

## 2. Root Folder Structure (Top-Level)

The following is the top-level structure of a `KevinCY-Kodex` project.

```
project-root/
│
├── app/
│   ├── core/
│   ├── routers/
│   ├── services/
│   ├── repositories/
│   ├── models/
│   ├── ai/
│   └── utils/
│
├── templates/
├── static/
│
├── tests/
│
├── .env
├── .env.example
├── requirements.txt
├── README.md
├── Makefile
└── main.py
```

---

## 3. /app/core — Core Settings Layer

Manages base system components like core settings, logger, and exception handling.

-   **Included Files**: `config.py`, `logger.py`, `exception_handlers.py`, `security.py`, etc.
-   **Responsibilities**:
    -   Loading environment variables.
    -   Configuring a common logger.
    -   Security/authentication policies.
    -   Global FastAPI exception handling.

---

## 4. /app/routers — API Endpoint Layer

Defines FastAPI endpoints.

-   **Rules**:
    -   Filename: `xxx_router.py`
    -   Router variable name: `router`
    -   Role: `Request` → Call `Service` → Return `Response`
-   **Examples**: `chat_router.py`, `voice_router.py`, `health_router.py`

---

## 5. /app/services — Business Logic Layer

Responsible for all business logic and workflow orchestration.

-   **Rules**:
    -   Filename: `xxx_service.py`
    -   Acts as an orchestrator between `Router` and `Repository`/`AI` layers.
    -   Delegates external system call logic to the `Repository`.
-   **Examples**: `chat_service.py`, `voice_service.py`, `rag_service.py`

---

## 6. /app/repositories — IO (DB/File/API) Layer

Handles all IO, including DB, file system, external APIs, and vector DBs.

-   **Rules**:
    -   Filename: `xxx_repository.py`
    -   Maintain a pure function style as much as possible.
    -   No business logic.
-   **Examples**: `document_repository.py`, `vector_repository.py`, `user_repository.py`

---

## 7. /app/models — Pydantic Models & DTOs

Defines Pydantic models used for `Request`/`Response`/`DTO`.

-   **Rules**:
    -   Filename: `xxx_model.py`
    -   Prohibit inclusion of business logic.
-   **Examples**: `chat_model.py`, `voice_model.py`, `rag_model.py`

---

## 8. /app/ai — LLM, RAG, LangGraph Engine Layer

An area responsible for all AI-related components.

-   **Recommended Sub-structure**:
    ```
    /app/ai/
      ├── graph.py
      ├── llm_client.py
      ├── ingest/
      ├── chunk/
      ├── embed/
      ├── retriever/
      ├── reranker/
      ├── generator/
      ├── postprocess/
      ├── stt/
      └── tts/
    ```
-   **Roles**: LangGraph pipelines, LLM calls, RAG components, voice processing (STT/TTS).

---

## 9. /app/utils — Common Utility Functions

Handles small pure functions, common logic, format conversions, string processing, etc.

-   **Examples**: `string_utils.py`, `time_utils.py`, `file_utils.py`

---

## 10. /templates — Jinja2 HTML Templates

Manages UI templates based on HTMX and Tailwind.

-   **Standard Structure**:
    ```
    /templates/
      ├── layout/ (base.html, header.html, footer.html)
      ├── pages/ (index.html, chat.html, voice_chat.html)
      └── partials/ (chat_message.html, loader.html, error.html)
    ```

---

## 11. /static — CSS, JS, Assets

Stores compiled CSS, JS, images, audio files, etc.

-   **Example Configuration**:
    ```
    /static/
      ├── css/ (tailwind.css)
      ├── js/ (main.js)
      ├── img/
      └── tts/
    ```

---

## 12. /tests — Test Directory

Stores `pytest`-based test code.

-   **Rules**:
    -   Filename: `test_xxx.py`
    -   Unit tests and integration tests can be separated.

---

## 13. Top-Level Files

-   `.env` / `.env.example`: Environment variable files.
-   `requirements.txt`: Python dependency list.
-   `Makefile`: Scripts for run/test/deploy.
-   `README.md`: Project overview and guide.
-   `main.py`: FastAPI application entry point.

---

## 14. Inter-Layer Dependency Rules

Dependencies should always flow from the outside in.

-   **Allowed**:
    -   `routers` → `services`
    -   `services` → `repositories` / `ai`
-   **Globally Usable**: `core`, `utils`, `models`
-   **Prohibited**: Reverse dependencies (e.g., `repositories` → `services`)

---

## 15. Project Structure for Expansion (Large Scale)

For large-scale projects, it is recommended to separate modules by functional domain.

-   **Example**:
    ```
    /app/
      ├── domain_chat/
      │   ├── chat_router.py
      │   ├── chat_service.py
      │   └── chat_model.py
      │
      ├── domain_voice/
      │   ├── voice_router.py
      │   ├── voice_service.py
      │   └── voice_model.py
      │
      └── ...
    ```

---

## 16. Paju City · Public Project Specific Structure

For public document projects, a `data/` folder can be included to manage original and processed data.

```
project-root/
  ├── data/
  │   ├── raw/        # Original PDF/documents
  │   ├── cleaned/    # Preprocessed text
  │   ├── chunks/     # Chunk storage
  │   └── embeddings/ # Vector index
  └── ...
```

---

## 17. Folder Management Principles in Operations & Deployment

-   Maintain the same folder structure across all environments (`local`, `staging`, `prod`).
-   Apply versioning rules to `data`, `embedding`, and `TTS` files.
-   Periodically clean up old temporary files, logs, and TTS voice files.
-   Never include the `.env` file in version control (Git).

---

End of KevinCY-Kodex Folder Standards