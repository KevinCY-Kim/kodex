# KevinCY-Kodex — Architecture Guideline
Version: 2025.11  
Scope: FastAPI + LangGraph + RAG + STT/TTS + Web Frontend

---

## 1. System Overview

본 시스템은 다음 서브시스템으로 구성된다:

- API Backend (FastAPI)
- AI Engine (LangGraph + LLM + RAG)
- Voice Pipeline (STT/TTS)
- Web Frontend (HTMX + Tailwind + Jinja2)
- Infra & Ops

---

## 2. Backend Layered Architecture

### 2.1 디렉토리 표준 구조
/app
/core # 설정, 초기화, 로거, 의존성
/routers # HTTP 엔드포인트 (입·출력만)
/services # 비즈니스 로직
/repositories # DB, 파일, 외부 API
/models # Pydantic I/O 모델
/ai # LangGraph, LLM, RAG
/utils # 공통 유틸
main.py # 엔트리포인트


### 2.2 계층별 책임

#### routers
- HTTP 요청/응답 변환
- RequestModel → Service 계층 호출 → ResponseModel 반환
- 비즈니스 로직, DB 접근, LLM 직접 호출 금지

#### services
- 도메인 비즈니스 로직 담당
- repository · ai 모듈 조합
- 트랜잭션/워크플로우 단위의 함수 제공

#### repositories
- DB, 파일 시스템, 외부 API, Vector DB 등 IO 담당
- 내부는 순수 Python 함수 + type hint
- 서비스 계층 외 직접 호출 금지

#### models
- Pydantic Request/Response/DTO 정의
- 검증/직렬화만 담당

#### core
- 설정 로딩(BaseSettings)
- 로거 설정
- 공통 예외 처리

#### ai
- LangGraph 워크플로우
- LLM 클라이언트
- RAG 컴포넌트(retriever, reranker, generator)

#### utils
- 공통 유틸 함수 모음

---

## 3. Request Lifecycle

### 3.1 텍스트 기반 질의 (RAG / 챗봇)

1. client → `/api/chat` POST
2. router: 입력 검증 후 service 호출
3. service:  
   - 사용자 메타 조회  
   - LangGraph/RAG 호출  
   - 모델 변환
4. router: ResponseModel 반환
5. logger: 요청/응답/지연시간 기록

### 3.2 음성 기반 질의 (STT → RAG → TTS)

1. client → `/api/voice-chat` 파일 업로드
2. router: 파일 임시 저장 후 service 호출
3. service:  
   - STT → 텍스트 변환  
   - 일반 RAG 흐름 재사용  
   - LLM 답변 → TTS 변환
4. router: 텍스트 + 음성 URL 반환

---

## 4. AI / RAG / LangGraph Architecture

### 4.1 RAG Pipeline

#### Ingestion
- PDF → Text
- 불필요 문구 제거, 문서 종류/섹션 메타데이터 추가

#### Chunking & Embedding
- chunk_size = 750, overlap = 100
- embedding model: ko-sbert 또는 e5-base
- Vector DB(Faiss 등)

#### Retrieval
- BM25 + Dense Hybrid
- Top-N 후보 → Reranker 적용

#### Answer Generation
- 선택 chunk 기반 생성
- 출처/근거 포함
- 공문·보고서 스타일 변환 가능

#### Post-processing
- 금칙어 필터링
- JSON/Markdown 정리

---

### 4.2 LangGraph Structure

- graph 파일 위치: `/app/ai/graph.py`

#### 노드 예시
- question_router_node  
- retrieval_node  
- rerank_node  
- generation_node  
- postprocess_node  

#### 호출 방식
- service 계층: `run_graph(input_state)`

---

## 5. Voice Pipeline (STT / TTS)

### 5.1 STT
- Whisper 또는 Faster-Whisper
- 파일 → 텍스트
- 언어 감지 / 옵션 파라미터

### 5.2 TTS
- gTTS 또는 기타 TTS API
- 텍스트 → 오디오 파일 생성
- 규칙적 파일명(타임스탬프+user_id)

### 5.3 Pipeline 규칙
- STT → RAG → TTS는 service 계층에서 하나의 플로우로 묶는다
- router는 파일 수신 + service 호출만 담당

---

## 6. Frontend Architecture

### 6.1 템플릿 구조

/templates
/layout
base.html
header.html
footer.html
/pages
index.html
chat.html
voice_chat.html
/partials
chat_message.html
loader.html


### 6.2 HTMX 규칙
- HTMX 요청 경로는 `/api/*`
- 예: `<form hx-post="/api/chat" hx-target="#chat-window" hx-swap="beforeend">`
- `hx-indicator`로 로딩 표시 필수

### 6.3 디자인 원칙
- 모바일 first
- 카드형 UI, 충분한 여백
- 사용자 친화적 에러 메시지

---

## 7. Infra / Environment / Deployment

### 7.1 환경 분리
- local / staging / prod  
- 각 환경별 `.env` 파일 관리

### 7.2 GPU / LLM 서버
- STT/LLM 서버를 Web 서버와 분리 가능
- conda env + requirements + env.yml 버전 관리

### 7.3 로깅/모니터링
- 요청수, 응답시간, 에러비율 모니터링
- 로그: path, method, status_code, latency, user_id

---

## 8. Cross-Cutting Concerns

### 8.1 공통 에러 처리
- `/app/core/exception_handlers.py`에 정의
- 오류 응답 형식:

{
"error_code": "...",
"message": "...",
"detail": ...
}

### 8.2 보안
- 최소 허용 CORS
- 민감정보 로그 금지

### 8.3 설정 관리
- `/app/core/config.py`에서 BaseSettings 사용

---

## 9. Architectural Principles (요약)

- Clean Architecture 우선  
- Business Logic = services  
- AI 엔진은 분리된 컴포넌트  
- 타 지자체/타 도메인 확장 가능 구조  
- 재사용 가능한 파이프라인 설계  

---

# End of KevinCY-Kodex Architecture


