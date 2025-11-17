KevinCY-Kodex — RAG Guideline

Version: 2025.11
Scope: Local LLM · LangGraph · BM25 + Dense Hybrid Retrieval · Public/Office Document QA

1. RAG Overview

RAG(Retrieval-Augmented Generation)는 다음 4단계로 구성된다.

Ingestion (문서 수집 및 전처리)

Chunking & Embedding (문서 분할 + 벡터화)

Retrieval (검색: BM25 + Dense + Rerank)

Answer Generation & Post-processing (LLM 기반 응답 생성)

모든 단계는 독립적인 모듈로 설계하며, 서비스 계층에서 조합하여 사용한다.

2. RAG Architecture Structure

폴더 구조는 다음을 기본으로 한다.

/app/ai/ingest

/app/ai/chunk

/app/ai/embed

/app/ai/retriever

/app/ai/reranker

/app/ai/generator

/app/ai/postprocess

/app/ai/graph.py (LangGraph Workflow)

각 단계는 단일 책임 원칙(SRP)을 따른다.

3. Ingestion Rules
3.1 문서 종류

PDF

HWP(파싱 API 기반)

DOCX

HTML

JSON/CSV

공공기관 문서

3.2 Ingestion 기본 규칙

텍스트 추출 후 반드시 정제(cleaning) 과정을 수행한다.

문서 내 불필요한 항목 제거

목차, 페이지 번호, 반복 헤더/푸터

광고, 시스템 메시지

문서 메타데이터 추가

출처(source)

문서 유형(type)

섹션 정보(section)

작성일(date)

3.3 파주시 프로젝트 특화 전처리

“타 지자체” 언급은 감지 후 주석 처리하거나 별도 필드에 저장한다.

전화번호, 부서명 등은 meta={"contact": "...", "department": "..."} 형태로 저장해 활용한다.

4. Chunking Strategy
4.1 기본 chunk 규칙

기본 chunk_size: 800

overlap: 200

문서 크기에 따라 탄력 조절 가능

4.2 Chunk 생성 원칙

의미 단위 기반 분절(문단·표·섹션 중심)

매우 긴 표/법령은 별도 청크 타입으로 관리

Whisper 또는 OCR로부터 온 텍스트는 노이즈 제거 우선 적용

4.3 Chunk 저장 형식

하나의 chunk는 다음 필드를 가진다.

id

text

source

section

page(optional)

embedding (벡터)

5. Embedding Rules
5.1 기본 모델

jhgan/ko-sbert-nli (한국어 문서 RAG 최적)

intfloat/e5-base

multilang이 필요하면 multilingual-e5-base

5.2 Embedding 정책

모든 chunk는 문자열 전처리 후 임베딩

길이가 너무 짧은 chunk는 제거

길이 제한 초과 chunk는 split 후 처리

5.3 저장소

Local FAISS (GPU/CPU)

필요 시 memory-mapped index 사용

ID → Metadata 매핑은 별도 DB/JSONL로 관리

6. Retrieval Rules
6.1 Hybrid Retrieval

1단계: BM25 키워드 검색

2단계: Dense 벡터 검색

3단계: 두 결과를 합쳐 상위 N개 후보 생성

6.2 Reranking

Cross-Encoder 또는 LLM 기반 rerank

Top-N (예: 20개) → Top-K(3~5개)로 최종 정제

Rerank 점수는 score 필드로 포함

6.3 Retrieval 품질 규칙

검색에서 타 지자체 문서가 섞일 경우 조정 규칙 적용

keyword weight 조절

“파주시”, “복지”, “지원”, “조례” 등 키워드 우대

동일 chunk 반복 방지

출처가 불분명한 chunk 제외

7. Answer Generation Rules
7.1 Prompt 정책

System Prompt: 문서 기반 전문가

사용자 질문 + 선택된 chunk를 컨텍스트로 넣어 생성

컨텍스트 외 정보 추측 금지

근거 문장을 반드시 포함

7.2 출력 형태

answer: 최종 응답

sources: 참조한 chunk ID 리스트

reasoning(optional): 선택 근거(내부용)

followup_questions(optional): 사용자가 추가로 물어볼 만한 질문

7.3 정부/공공 문서 응답 규칙

문서 기반 답변 우선

최신성 보장 불가능한 경우 주의 문구 포함

어려운 행정 용어는 예시와 함께 설명

8. Post-Processing Rules
8.1 텍스트 정제

중복 문장 제거

불필요한 접속사 축소

“사용자에게 읽히는 문장” 기준으로 자연스럽게 재구성

8.2 파주시 특화 후처리

다른 지자체 문장 포함 시 필터링

불완전한 조항을 보완하지 않고 문서 기반으로 그대로 유지

제도/지원금 문의시 “공식 문의처”를 자동으로 추가

8.3 JSON 포맷 예시

최종 응답은 다음과 같은 구조를 따라야 한다

{
  "answer": "...",
  "sources": ["paju_2024_welfare_01#chunk_021", "paju_guideline#chunk_004"],
  "followup": ["관련 서류는 무엇인가요?", "신청 가능한 온라인 페이지는 있나요?"]
}

9. LangGraph Workflow Rules
9.1 RAG Pipeline Graph 구성

필수 노드:

preprocess_node

retrieval_node

rerank_node

generation_node

postprocess_node

9.2 State 구조

query

user_meta (optional)

retrieved_docs

reranked_docs

answer

sources

9.3 에러 처리

retrieval 실패 → 빈 리스트 반환

chunk 없는 경우 → “문서 내 근거 없음” 메시지

시스템 오류 → fallback 응답 생성

10. Evaluation Rules
10.1 품질 지표

Recall@K

Precision@K

Rerank 성능

Answer Faithfulness (문서 근거 충실도)

Hallucination score

10.2 자동 테스트

known answer baseline 대비 비교

critical keyword coverage 검사

chunk alignment 검증

11. 운영(Production) 규칙
11.1 Index Versioning

매 ingest마다 index_version=YYYYMMDD_vX 규칙 적용

오래된 인덱스는 제거

문서 변경 시 동일 문서 전부 재임베딩

11.2 Monitoring

Retrieval hit ratio

LLM latency

STT/TTS pipeline latency (음성 RAG의 경우)

사용자 섹션별 질문량

11.3 데이터 보안

민감정보 있는 chunk는 embedding 제외

전화번호/주민번호는 masking 후 저장

End of KevinCY-Kodex RAG Guideline