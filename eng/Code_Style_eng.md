# KevinCY-Kodex — Code Style Guide

**Version:** 2025.11  
**Scope:** Python · FastAPI · LangGraph · RAG · HTMX · Tailwind

---

## 1. General Coding Principles

-   Apply **100% `type hints`** to all code.
-   Functions must adhere to the **Single Responsibility Principle (SRP)**.
-   Use meaning-based naming for variables, functions, and files.
-   Prohibit magic numbers/strings (hard-coded values) → separate them into `settings`/`config`/constants.
-   Business logic flow: `routers` → `services` → `repositories`/`ai`.
-   Minimize side effects (aim for pure functions where possible).

---

## 2. Python Code Style

### 2.1 Naming Conventions

| Target | Convention | Example |
| :--- | :--- | :--- |
| Variable | `snake_case` | `user_id`, `file_path` |
| Function | `snake_case` | `load_document()`, `run_rag()` |
| Class | `PascalCase` | `DocumentParser`, `ChatService` |
| Constant | `UPPER_SNAKE_CASE` | `DEFAULT_CHUNK_SIZE = 800` |
| Filename | `snake_case` | `chat_service.py`, `rag_core.py` |

### 2.2 Formatting

-   Based on **PEP8**, with 4-space indentation.
-   Maximum line length: **100-120 characters** (wrap if exceeded).
-   **`import` order**:
    1.  Standard library
    2.  Third-party libraries
    3.  Local modules
    ```python
    # 1. Standard library
    import json
    import os
    from pathlib import Path
    
    # 2. Third-party libraries
    import numpy as np
    from fastapi import APIRouter
    
    # 3. Local modules
    from app.core.config import settings
    from app.services.chat_service import handle_chat
    ```
-   Prohibit `from x import *`.
-   Remove unused `imports` and variables.
-   Minimize `try/except` blocks and specify concrete exception types.

### 2.3 Docstring Style

-   Use a simple summary + `Args`/`Returns` format for public functions/methods.
    ```python
    from pathlib import Path

    def load_document(path: Path) -> str:
        """
        Loads text from the given file path.
    
        Args:
            path: The path to the text file.
    
        Returns:
            The entire text string read from the file.
        """
    ```

---

## 3. FastAPI Code Style

### 3.1 Router File Rules

-   **Filename**: `xxx_router.py` (e.g., `chat_router.py`, `voice_router.py`)
-   **Router instance name**: Always `router`.
    ```python
    from fastapi import APIRouter
    
    router = APIRouter(prefix="/chat", tags=["chat"])
    ```

### 3.2 Router Internal Principles

-   The Router is responsible for **input/output only** and does not contain business logic.
-   Follows the flow: `RequestModel` → `Service` call → `ResponseModel` return.
    ```python
    from app.models.chat_model import ChatRequest, ChatResponse
    from app.services.chat_service import handle_chat
    
    @router.post("/", response_model=ChatResponse)
    async def chat(request: ChatRequest) -> ChatResponse:
        # This is where the business logic is called.
        return await handle_chat(request)
    ```

### 3.3 Response Rules

-   Use Pydantic models for return types whenever possible.
-   Minimize returning `dict` directly; define a model and use it if necessary.

---

## 4. Pydantic Model Style

### 4.1 Basic Rules

-   **Filename**: `xxx_model.py`
-   **Class name**: `PascalCase`
-   Mainly used for I/O DTOs; do not include business logic.
    ```python
    from pydantic import BaseModel
    
    class ChatRequest(BaseModel):
        user_id: str | None = None
        query: str
    
    class ChatResponse(BaseModel):
        answer: str
        sources: list[str] | None = None
    ```

---

## 5. Services Layer Style

### 5.1 Rules

-   **Filename**: `xxx_service.py`
-   **Role**: Combination of business logic + workflow.
-   **Key Responsibilities**:
    -   Input validation (if additional validation is needed).
    -   Orchestration of multiple `repository`/`ai` calls.
    -   Exception handling and logging.
-   Maintain a structure where service functions are called directly from the Router.
    ```python
    from app.models.chat_model import ChatRequest, ChatResponse
    from app.ai.graph import run_chat_graph
    
    async def handle_chat(request: ChatRequest) -> ChatResponse:
        """Entry point for the service layer to handle chat requests."""
        state = {"query": request.query, "user_id": request.user_id}
        result = await run_chat_graph(state)
        return ChatResponse(answer=result["answer"], sources=result.get("sources"))
    ```

---

## 6. Repositories Style

### 6.1 Rules

-   **Filename**: `xxx_repository.py`
-   **Responsibility**: Dedicated to **IO** such as DB, files, Vector DB, external APIs.
-   Write as pure functions where possible, and do not include business rules.
-   Only perform roles like saving/retrieving/searching.
    ```python
    from typing import Sequence
    from app.models.chunk_model import Chunk
    
    def load_chunks(document_id: str) -> Sequence[Chunk]:
        """Loads a list of chunks corresponding to the document ID."""
        # ... DB or file system logic ...
        pass
    ```

---

## 7. AI / LangGraph Code Style

### 7.1 Folder Structure

The folder structure for AI and LangGraph related modules follows the project-wide standard.
> For a detailed AI-related folder structure, please refer to the `Folder_Standards_eng.md` document.

### 7.2 LangGraph Node Function Style

-   All nodes maintain the `state: dict -> dict` format.
    ```python
    async def retrieval_node(state: dict) -> dict:
        query: str = state["query"]
        docs = retrieve_documents(query)
        return {**state, "retrieved_docs": docs}
    ```

### 7.3 LLM Call Style

```python
async def generate_answer(context: str, question: str) -> str:
    prompt = f"""
You are an expert who answers based on official documents.
You must answer based on evidence, and if you don't know, say you don't know.

[Context]
{context}

[Question]
{question}
"""
    return await llm_client.chat(prompt)
```

---

## 8. RAG Code Style

### 8.1 Chunking

-   **Default settings**: `chunk_size = 800`, `chunk_overlap = 200`
    ```python
    def chunk_text(content: str, size: int = 800, overlap: int = 200) -> list[str]:
        chunks: list[str] = []
        start = 0
        length = len(content)
    
        while start < length:
            end = min(start + size, length)
            chunk = content[start:end]
            chunks.append(chunk)
            start += size - overlap
    
        return chunks
    ```

### 8.2 Retrieval

-   **Hybrid**: Combination of keyword (`BM25`) + vector search.
-   Separate `Rerank` into a dedicated function/module.
    ```python
    def hybrid_search(query: str, k: int = 10) -> list[dict]:
        keyword_docs = bm25_search(query, k=k)
        vector_docs = dense_search(query, k=k)
        merged = merge_results(keyword_docs, vector_docs)
        return rerank(query, merged)
    ```

### 8.3 Answer Generation

-   Use structured output (JSON-like `dict`) whenever possible.
    ```python
    def build_answer_payload(answer: str, sources: list[str]) -> dict:
        return {
            "answer": answer,
            "sources": sources,
        }
    ```

---

## 9. Error Handling Style

### 9.1 Rules

-   Global FastAPI exception handlers are managed in `/app/core/exception_handlers.py`.
-   Convert exceptions to appropriate `HTTPException` in the `Service` layer.
    ```python
    from fastapi import HTTPException, status
    
    def ensure_document_exists(doc_id: str) -> None:
        if not document_exists(doc_id):
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail="The corresponding document could not be found.",
            )
    ```
-   Do not log sensitive information (e.g., social security numbers, API tokens).

---

## 10. Logging Style

-   Use structured logs (JSON format).
    ```python
    import logging
    
    logger = logging.getLogger(__name__)
    
    def log_chat_request(user_id: str | None, query: str) -> None:
        logger.info({
            "event": "chat_request",
            "user_id": user_id,
            "query": query[:100],  # Consider sensitive info and length limits
        })
    ```

---

## 11. HTMX Code Style

### 11.1 Basic Principles

-   HTMX request paths should always use the `/api/*` namespace.
-   Explicitly use `hx-target`, `hx-swap`, and `hx-indicator`.
    ```html
    <form
      hx-post="/api/chat"
      hx-target="#chat-window"
      hx-swap="beforeend"
      hx-indicator="#chat-loading"
    >
      <input type="text" name="query" />
      <button type="submit">Send</button>
    </form>
    ```

---

## 12. Tailwind Code Style

-   **Class order** should maintain the following group sequence:
    1.  Layout (`flex`, `grid`, etc.)
    2.  Alignment/Spacing (`items-center`, `gap`, `p`/`m`)
    3.  Typography (`text-sm`, `font-medium`)
    4.  Color/Background (`bg-white`)
    5.  Other (`rounded-lg`, `shadow`, `transition`)
    ```html
    <div class="flex items-center gap-2 px-4 py-2 text-sm font-medium text-gray-900 bg-white rounded-lg shadow">
      ...
    </div>
    ```
-   Prioritize using the Tailwind default palette over meaningless custom colors.

---

## 13. File Naming Rules

| File Type | Naming Example |
| :--- | :--- |
| Router | `chat_router.py` |
| Service | `chat_service.py` |
| Repository | `chat_repository.py` |
| Model | `chat_model.py` |
| Utility | `string_utils.py` |
| AI Module | `retriever.py` |
| Test Code | `test_chat_service.py` |

---

## 14. Test Code Style (pytest)

### 14.1 Rules

-   **Filename**: `test_xxx.py`
-   **Test function name**: Use `test_` prefix.
-   **Given–When–Then** pattern is recommended.
    ```python
    import pytest
    from app.models.chat_model import ChatRequest
    from app.services.chat_service import handle_chat
    
    @pytest.mark.asyncio
    async def test_handle_chat_returns_answer():
        # Given
        request = ChatRequest(query="Hello")
    
        # When
        response = await handle_chat(request)
    
        # Then
        assert response.answer
        assert isinstance(response.answer, str)
    ```

---

## 15. Git Commit Message Style

-   Use English-based `prefix`.
    -   `feat`: Add a new feature (e.g., `feat: add voice chat STT pipeline`)
    -   `fix`: Fix a bug (e.g., `fix: handle empty query in chat service`)
    -   `refactor`: Refactor code (e.g., `refactor: simplify rag retrieval flow`)
    -   `docs`: Modify documentation (e.g., `docs: update architecture and code-style`)
    -   `chore`: Other changes like build, dependencies (e.g., `chore: bump dependency versions`)

---

End of KevinCY-Kodex Code Style Guide