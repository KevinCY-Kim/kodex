# KevinCY-Kodex — Architecture Guideline
Version: 2025.11  
Scope: FastAPI + LangGraph + RAG + STT/TTS + Web Frontend

---

## 1. System Overview

This system consists of the following main subsystems.

-   **API Backend**: FastAPI
-   **AI Engine**: LangGraph + LLM + RAG
-   **Voice Pipeline**: STT/TTS
-   **Web Frontend**: HTMX + Tailwind + Jinja2
-   **Infra & Ops**: Deployment, Monitoring, Environment Management

---

## 2. Backend Layered Architecture

### 2.1 Directory Standard Structure and Layer Responsibilities

The project's folder structure and the detailed responsibilities of each layer follow the **`Folder_Standards_eng.md`** document.

---

## 3. Request Lifecycle

### 3.1 Text-based Query (RAG / Chatbot)

1.  **Client**: `POST` request to `/api/chat`.
2.  **Router**: Validates input and calls the `Service`.
3.  **Service**:
    -   Retrieves user metadata.
    -   Calls the LangGraph/RAG pipeline.
    -   Transforms the result into a `ResponseModel`.
4.  **Router**: Returns the `ResponseModel` to the client.
5.  **Logger**: Records the request, response, and latency.

### 3.2 Voice-based Query (STT → RAG → TTS)

1.  **Client**: Uploads a voice file to `/api/voice-chat`.
2.  **Router**: Temporarily saves the file and calls the `Service`.
3.  **Service**:
    -   Converts audio to text via STT.
    -   Reuses the text-based RAG flow.
    -   Converts the generated answer to a voice file via TTS.
4.  **Router**: Returns both the text answer and the voice file URL.

---

## 4. AI / RAG / LangGraph Architecture

> For detailed implementation rules for each stage of the RAG pipeline, please refer to the **`RAG_eng.md`** document.

### 4.1 RAG Pipeline

#### Ingestion
-   Extract text from source documents like PDF, DOCX.
-   Remove unnecessary text and add metadata such as document type and section.

#### Chunking & Embedding
-   `chunk_size = 800`, `chunk_overlap = 200`
-   Embedding Model: `ko-sbert` or `e5-base`
-   Vector DB: `FAISS`, etc.

#### Retrieval
-   BM25 + Dense Hybrid search.
-   Apply a Reranker to the Top-N candidates to select the final candidates.

#### Answer Generation
-   Generate an answer based on the selected `chunk`.
-   Specify the source/evidence in the answer.
-   Can be converted to official document or report style if needed.

#### Post-processing
-   Filter prohibited words.
-   Organize the final response into JSON or Markdown format.

---

### 4.2 LangGraph Structure

-   Graph definition file location: `/app/ai/graph.py`

#### Example Nodes
-   `question_router_node`
-   `retrieval_node`
-   `rerank_node`
-   `generation_node`
-   `postprocess_node`

#### Invocation Method
-   Called from the `Service` layer in the form of `run_graph(input_state)`.

---

## 5. Voice Pipeline (STT / TTS)

### 5.1 STT
-   **Engine**: `Whisper` or `Faster-Whisper`
-   **Function**: Converts audio file to text.
-   **Options**: Automatic language detection, noise removal, etc.

### 5.2 TTS
-   **Engine**: `gTTS` or a commercial TTS API.
-   **Function**: Generates an audio file from text.
-   **File Management**: Uses a regular filename combining `timestamp` and `user_id`.

### 5.3 Pipeline Rules
-   The STT → RAG → TTS flow is managed as a single workflow in the `Service` layer.
-   The `Router` is only responsible for receiving files and calling the `Service`.

---

## 6. Frontend Architecture

### 6.1 Template Structure

```
/templates/
  ├── layout/
  │   ├── base.html
  │   ├── header.html
  │   └── footer.html
  ├── pages/
  │   ├── index.html
  │   ├── chat.html
  │   └── voice_chat.html
  └── partials/
      ├── chat_message.html
      └── loader.html
```

### 6.2 HTMX Rules
-   HTMX request API paths use the `/api/*` namespace.
-   **Example**: `<form hx-post="/api/chat" hx-target="#chat-window" hx-swap="beforeend">`
-   Must use `hx-indicator` to show a loading state to the user.

### 6.3 Design Principles
-   **Mobile First**
-   Card-based UI, ensure sufficient whitespace.
-   Provide user-friendly error messages.

---

## 7. Infra / Environment / Deployment

### 7.1 Environment Separation
-   Separate `local`, `staging`, and `prod` environments.
-   Manage settings for each environment via `.env` files.

### 7.2 GPU / LLM Server
-   The STT/LLM server can be physically separated from the web server.
-   Manage dependencies and environment versions via `conda` environment, `requirements.txt`, and `env.yml`.

### 7.3 Logging/Monitoring
-   **Key Metrics**: Request count, response time, error rate.
-   **Log Items**: `path`, `method`, `status_code`, `latency`, `user_id`, etc.

---

## 8. Cross-Cutting Concerns

### 8.1 Common Error Handling
-   Global exception handlers are defined in `/app/core/exception_handlers.py`.
-   **Error Response Format**:
    ```json
    {
      "error_code": "...",
      "message": "...",
      "detail": "..."
    }
    ```

### 8.2 Security
-   Apply a minimal `CORS` policy.
-   Prohibit logging of sensitive information (personal info, API keys, etc.).

### 8.3 Configuration Management
-   Manage settings in `/app/core/config.py` using Pydantic's `BaseSettings`.

---

## 9. Architectural Principles (Summary)

-   Prioritize **Clean Architecture**.
-   Business logic is concentrated in the **`services`** layer.
-   The AI engine is separated as an independent component.
-   Design a structure that is easily expandable to other domains.
-   Design reusable pipelines.

---

# End of KevinCY-Kodex Architecture