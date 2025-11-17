# KevinCY-Kodex — Architecture Guideline
Version: 2025.11  
Scope: FastAPI + LangGraph + RAG + STT/TTS + Web Frontend

---

## 1. System Overview

본 시스템은 다음의 주요 서브시스템으로 구성됩니다.

-   **API Backend**: FastAPI
-   **AI Engine**: LangGraph + LLM + RAG
-   **Voice Pipeline**: STT/TTS
-   **Web Frontend**: HTMX + Tailwind + Jinja2
-   **Infra & Ops**: 배포, 모니터링, 환경 관리

---

## 2. Backend Layered Architecture

### 2.1 디렉토리 표준 구조 및 계층별 책임

프로젝트의 폴더 구조와 각 계층의 상세 책임은 **`Folder_Standards.md`** 문서를 따릅니다.

---

## 3. Request Lifecycle

### 3.1 텍스트 기반 질의 (RAG / 챗봇)

1.  **Client**: `/api/chat`으로 `POST` 요청
2.  **Router**: 입력 검증 후 `Service` 호출
3.  **Service**:
    -   사용자 메타데이터 조회
    -   LangGraph/RAG 파이프라인 호출
    -   결과를 `ResponseModel`로 변환
4.  **Router**: `ResponseModel`을 클라이언트에 반환
5.  **Logger**: 요청, 응답, 지연 시간 기록

### 3.2 음성 기반 질의 (STT → RAG → TTS)

1.  **Client**: `/api/voice-chat`으로 음성 파일 업로드
2.  **Router**: 파일을 임시 저장 후 `Service` 호출
3.  **Service**:
    -   STT를 통해 텍스트로 변환
    -   텍스트 기반 RAG 흐름 재사용
    -   생성된 답변을 TTS로 음성 파일 변환
4.  **Router**: 텍스트 답변과 음성 파일 URL을 함께 반환

---

## 4. AI / RAG / LangGraph Architecture

> RAG 파이프라인의 각 단계별 상세 구현 규칙은 **`RAG.md`** 문서를 참조하세요.

### 4.1 RAG Pipeline

#### Ingestion
-   PDF, DOCX 등 원문서 → 텍스트 추출
-   불필요 문구 제거, 문서 종류/섹션 등 메타데이터 추가

#### Chunking & Embedding
-   `chunk_size = 800`, `chunk_overlap = 200`
-   Embedding 모델: `ko-sbert` 또는 `e5-base`
-   Vector DB: `FAISS` 등

#### Retrieval
-   BM25 + Dense Hybrid 검색
-   Top-N 후보군에 Reranker 적용하여 최종 후보 선정

#### Answer Generation
-   선택된 `chunk`를 근거로 답변 생성
-   답변에 출처/근거 명시
-   필요 시 공문, 보고서 스타일로 변환

#### Post-processing
-   금칙어 필터링
-   최종 응답을 JSON 또는 Markdown 형식으로 정리

---

### 4.2 LangGraph Structure

-   Graph 정의 파일 위치: `/app/ai/graph.py`

#### 노드 예시
-   `question_router_node`
-   `retrieval_node`
-   `rerank_node`
-   `generation_node`
-   `postprocess_node`

#### 호출 방식
-   `Service` 계층에서 `run_graph(input_state)` 형태로 호출

---

## 5. Voice Pipeline (STT / TTS)

### 5.1 STT
-   **엔진**: `Whisper` 또는 `Faster-Whisper`
-   **기능**: 오디오 파일 → 텍스트 변환
-   **옵션**: 언어 자동 감지, 노이즈 제거 등

### 5.2 TTS
-   **엔진**: `gTTS` 또는 상용 TTS API
-   **기능**: 텍스트 → 오디오 파일 생성
-   **파일 관리**: `타임스탬프`와 `user_id`를 조합한 규칙적 파일명 사용

### 5.3 Pipeline 규칙
-   STT → RAG → TTS 흐름은 `Service` 계층에서 단일 워크플로우로 관리합니다.
-   `Router`는 파일 수신과 `Service` 호출 역할만 담당합니다.

---

## 6. Frontend Architecture

### 6.1 템플릿 구조

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

### 6.2 HTMX 규칙
-   HTMX 요청 API 경로는 `/api/*` 네임스페이스를 사용합니다.
-   **예시**: `<form hx-post="/api/chat" hx-target="#chat-window" hx-swap="beforeend">`
-   `hx-indicator`를 사용하여 사용자에게 로딩 상태를 반드시 표시합니다.

### 6.3 디자인 원칙
-   **모바일 우선** (Mobile First)
-   카드형 UI, 충분한 여백 확보
-   사용자 친화적인 오류 메시지 제공

---

## 7. Infra / Environment / Deployment

### 7.1 환경 분리
-   `local`, `staging`, `prod` 환경 분리
-   각 환경별로 `.env` 파일을 통해 설정 관리

### 7.2 GPU / LLM 서버
-   STT/LLM 서버는 웹 서버와 물리적으로 분리하여 구성할 수 있습니다.
-   `conda` 환경, `requirements.txt`, `env.yml`을 통해 의존성 및 환경 버전을 관리합니다.

### 7.3 로깅/모니터링
-   **주요 지표**: 요청 수, 응답 시간, 오류 비율
-   **로그 항목**: `path`, `method`, `status_code`, `latency`, `user_id` 등

---

## 8. Cross-Cutting Concerns

### 8.1 공통 에러 처리
-   전역 예외 처리기는 `/app/core/exception_handlers.py`에 정의합니다.
-   **오류 응답 형식**:
    ```json
    {
      "error_code": "...",
      "message": "...",
      "detail": "..."
    }
    ```

### 8.2 보안
-   최소한의 `CORS` 정책 적용
-   로그에 민감 정보(개인정보, API 키 등) 기록 금지

### 8.3 설정 관리
-   `/app/core/config.py`에서 `Pydantic`의 `BaseSettings`를 사용하여 설정을 관리합니다.

---

## 9. Architectural Principles (요약)

-   **Clean Architecture** 우선
-   비즈니스 로직은 **`services`** 계층에 집중
-   AI 엔진은 독립된 컴포넌트로 분리
-   타 도메인으로 확장이 용이한 구조 설계
-   재사용 가능한 파이프라인 설계

---

# End of KevinCY-Kodex Architecture
