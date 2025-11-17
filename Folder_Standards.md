KevinCY-Kodex — Folder Standards

Version: 2025.11
Scope: FastAPI · LangGraph · RAG · STT/TTS · Web(HTMX+Tailwind)

1. Overview

이 문서는 프로젝트 파일·폴더 구조의 표준 규칙을 정의한다.
프로젝트는 Clean Architecture 원칙에 따라 다음과 같은 7개 핵심 계층으로 구성한다.

core

routers

services

repositories

models

ai

utils

web (templates · static)

각 계층은 단일 책임(SRP)을 가져야 하며, 계층 간 의존 방향을 준수한다.

2. Root Folder Structure (Top-Level)

아래는 KevinCY 프로젝트의 최상위 구조이다.

project-root/
│
├── app/
│   ├── core/
│   ├── routers/
│   ├── services/
│   ├── repositories/
│   ├── models/
│   ├── ai/
│   ├── utils/
│   └── __init__.py
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

3. /app/core — Core Settings Layer

핵심 설정, 로거, 예외 처리 등 기반 시스템 구성 요소를 모아 둔다.

포함되는 파일

config.py

logger.py

exception_handlers.py

security.py (필요 시)

startup.py / shutdown.py

책임

환경 변수 로딩

공통 로거 설정

보안/인증 정책

FastAPI 전역 예외 처리

4. /app/routers — API Endpoint Layer

FastAPI 엔드포인트 정의.

규칙

파일명: xxx_router.py

라우터 변수명: router

역할: Request → Service → Response

예시 파일

chat_router.py

voice_router.py

admin_router.py

health_router.py

5. /app/services — Business Logic Layer

모든 비즈니스 로직 + 워크플로우 결합을 담당한다.

규칙

파일명: xxx_service.py

Router <-> Repository/AI 간의 조정자 역할

외부 시스템 호출 로직은 repository로 위임

예시 파일

chat_service.py

voice_service.py

rag_service.py

user_service.py

6. /app/repositories — IO (DB/File/API) Layer

DB, 파일 시스템, 외부 API, 벡터 DB 등의 IO 처리.

규칙

파일명: xxx_repository.py

가능한 한 순수 함수 형태 유지

비즈니스 로직 없음

예시 파일

document_repository.py

vector_repository.py

user_repository.py

external_api_repository.py

7. /app/models — Pydantic Models & DTOs

Request/Response/DTO에 사용되는 Pydantic 모델 정의.

규칙

파일명: xxx_model.py

비즈니스 로직 포함 금지

예시 파일

chat_model.py

voice_model.py

rag_model.py

user_model.py

8. /app/ai — LLM, RAG, LangGraph Engine Layer

AI 관련 모든 구성 요소를 담당하는 영역.

권장 세부 구조
/app/ai
│
├── graph.py
├── llm_client.py
│
├── ingest/
├── chunk/
├── embed/
├── retriever/
├── reranker/
├── generator/
├── postprocess/
│
├── stt/
└── tts/

역할

LangGraph 파이프라인

LLM 호출

RAG 파이프라인 구성요소

음성 처리(STT/TTS)

9. /app/utils — 공통 유틸 함수

작은 순수 함수들, 공통 로직, 형식 변환, 문자열 처리 등.

예시 파일

string_utils.py

time_utils.py

validation_utils.py

file_utils.py

10. /templates — Jinja2 HTML Templates

HTMX + Tailwind 기반 UI 템플릿.

구조 표준
/templates
│
├── layout/
│   ├── base.html
│   ├── header.html
│   └── footer.html
│
├── pages/
│   ├── index.html
│   ├── chat.html
│   └── voice_chat.html
│
└── partials/
    ├── chat_message.html
    ├── loader.html
    └── error.html

11. /static — CSS, JS, Assets

Tailwind CSS, JS, 이미지, 오디오 파일 등을 저장.

구성 예시
/static
│
├── css/
│   └── tailwind.css
│
├── js/
│   └── main.js
│
├── img/
│
└── tts/

12. /tests — Test Directory

pytest 기반 테스트 파일 저장.

규칙

파일명: test_xxx.py

단위 테스트 + 통합 테스트 분리 가능

13. Top-Level Files
.env / .env.example

환경 변수 파일.

requirements.txt

Python dependency list.

Makefile

실행/테스트/배포 스크립트.

README.md

프로젝트 개요 및 가이드.

main.py

FastAPI 엔트리 포인트.

14. 계층 간 의존성 규칙

아래 방향만 허용된다.

routers → services → repositories

routers → services → ai

services → repositories

services → ai

core, utils는 어디서든 import 가능

역방향(예: repository에서 service import) 금지.

15. 확장용 프로젝트 구조 (Large Scale)

대규모 프로젝트에서는 영역별 도메인 모듈 분리를 권장한다.

예시:

/app
│
├── domain_chat/
│   ├── chat_router.py
│   ├── chat_service.py
│   ├── chat_repository.py
│   ├── chat_model.py
│   └── __init__.py
│
├── domain_voice/
│   ├── voice_router.py
│   ├── voice_service.py
│   ├── voice_model.py
│   └── __init__.py
│
└── domain_rag/
    ├── rag_service.py
    ├── rag_model.py
    ├── rag_pipeline/
    └── __init__.py

16. 파주시·공공 프로젝트 특화 구조

공공 문서 프로젝트에서는 data/ 폴더를 포함한다.

project-root/
│
├── data/
│   ├── raw/        # 원문 PDF/문서
│   ├── cleaned/    # 전처리된 텍스트
│   ├── chunks/     # chunk 저장소
│   └── embeddings/ # 벡터 인덱스

17. 운영·배포에서의 폴더 관리 원칙

모든 환경(local/staging/prod)은 동일한 폴더 구조 유지

data·embedding·TTS 파일은 버전 규칙 적용

오래된 임시 파일, 로그, TTS 음성 파일은 주기적으로 정리

.env는 절대 버전 관리(Git)에 포함하지 않는다

End of KevinCY-Kodex Folder Standards