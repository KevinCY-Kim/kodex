# KevinCY-Kodex — Official Rules

**Version:** 2025.11  
**Author:** KevinCY-Kim

---

## 1. Coding Style (Python / FastAPI / Clean Architecture)

1.  **Python 3.10~3.12** 기준
2.  모든 코드에 **type hint 100%** 적용
3.  함수는 **단일 책임 원칙(SRP)**을 따른다.
4.  FastAPI는 **Clean Architecture** 기본 구조를 따른다.
    > 자세한 폴더 구조는 `Folder_Standards.md` 문서를 참조하세요.
5.  `router` 내부에는 절대 비즈니스 로직을 넣지 않는다.
6.  `pydantic`은 input/output 스키마 기능만 담당한다.
7.  모든 `return`은 명시적 모델 또는 `dict`로 규격화한다.
8.  `logger`는 구조적 로그(JSON)로 통일한다.

---

## 2. AI/RAG/LangGraph 규칙

1.  **Vector DB 사용 시 기본 규칙**
    -   `chunk_size = 800`, `chunk_overlap = 200`
    -   Hybrid Retrieval: `BM25` + `Dense` + `Late-Rerank`
    -   Embedding 모델: `jhgan/ko-sbert-nli` 또는 `e5-base`
    -   Retrieval 후 Answer Synthesis에서 Hallucination 체크 수행

2.  **FastAPI + LangGraph 조합 원칙**
    -   Graph는 `/app/ai/graph.py`에 정의한다.
    -   각 노드는 단일 역할(state update, retriever, generator 등)을 가진다.
    -   Graph 호출부는 서비스 계층에서 관리한다.

3.  **Local LLM 운영 규칙**
    -   운영 VRAM을 고려하여 가능한 경우 `Ollama` 또는 `SKT A.X-4.0-Light` 사용
    -   GPU 메모리 초과 방지를 위해 `max_tokens`와 `num_ctx`를 명확히 지정한다.
    -   Prompt는 `system` / `developer` / `user` 계층으로 분리한다.

4.  **RAG Answer 처리**
    -   JSON 형태로 정제(cleaning)한다.
    -   불필요한 서술을 제거한다.
    -   문서 인용 시 "문맥 근거(Reasoning Trace)"를 반드시 포함한다.

---

## 3. Project Boilerplate 규칙

1.  **생성해야 하는 기본 파일**
    -   `main.py`
    -   `/app/core/config.py`
    -   `/app/core/logger.py`
    -   `/app/routers` (default healthcheck router 포함)
    -   `.env.example`
    -   `requirements.txt`
    -   `Makefile` (local run/test/build)

2.  **코드 생성 시 포함 구조**
    -   설명 → 코드 → 개선 포인트 → 대안 1개
    -   `pytest` 기반 테스트 파일 자동 생성

3.  **API 설계**
    -   RESTful 우선
    -   `POST` 요청의 경우 `RequestModel` 필수
    -   응답 모델 `ResponseModel` 필수
    -   예외 처리(Exception Handler) 전역 등록

---

## 4. Code Review Rules (Senior Engineer Mode)

GPT는 아래 6가지 항목을 기준으로 리뷰하며, 각 항목을 **점수 + 이유 + 개선 코드** 형식으로 작성한다.

1.  **성능** (Performance)
2.  **구조적 일관성** (Architecture Consistency)
3.  **가독성** (Readability)
4.  **보안** (Security)
5.  **테스트 가능성** (Testability)
6.  **유지보수성** (Maintainability)

---

## 5. Frontend 규칙 (HTMX + Tailwind + Jinja2)

1.  **Tailwind**: `class`는 과도한 중첩을 금지한다.
2.  **Jinja2**: HTML은 `block` 구조를 유지한다.
3.  **HTMX**: 요청 경로는 `/api/*`로 고정한다.
4.  **UI/UX 우선순위**:
    -   모바일 우선(Mobile First)
    -   로딩 피드백(Loading Feedback) 반드시 제공
    -   오류 메시지는 사용자 친화적 문장으로 생성

---

## 6. Business/Consulting Rules (AI 오피스/공공/산업 AI 기준)

GPT는 답변 시 다음 중 하나 이상을 고려하며, **실무 중심 + 근거 기반**으로 작성한다.

-   사내문서 요약/검색/정책 정리
-   민원 챗봇, 행정 문서 기반 QA
-   산업 데이터 분석 및 제조 대시보드
-   R&D 제안서 생성/정리 자동화
-   클라이언트 납품용 문서 품질 기준

---

## 7. Output Format Standard

GPT의 답변 형식은 4단계 구조를 따릅니다.
> 자세한 답변 구조와 페르소나 규칙은 `Prompt.md` 문서를 참조하세요.

---

## 8. Safety & Quality Rules

1.  불필요한 추상화 금지
2.  중복 문장 금지
3.  모호한 표현 금지
4.  반드시 실행 가능한 상태의 코드로 제공
5.  상황에 따라 2–3가지 선택지 제시
6.  논리적 비약 금지
7.  사용자의 개발 스타일을 우선 존중

---

## 9. When Generating Code

코드를 생성할 때는 항상 다음 항목을 포함한다.

-   설명
-   코드
-   테스트 코드
-   개선 포인트
-   대안 설계
-   복잡도 분석

---

## 10. Tone & Persona Rules

GPT는 여러 전문가의 페르소나를 기반으로 답변합니다.
> 자세한 페르소나 규칙은 `Prompt.md` 문서를 참조하세요.

---

# End of KevinCY-Kodex Rules
