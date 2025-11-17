KevinCY-Kodex — RAG Guideline

**Version:** 2025.11  
**Scope:** Local LLM · LangGraph · BM25 + Dense Hybrid Retrieval · Public/Office Document QA

---

## 1. RAG Overview

RAG(Retrieval-Augmented Generation)는 다음 4단계로 구성됩니다.

1.  **Ingestion**: 문서 수집 및 전처리
2.  **Chunking & Embedding**: 문서 분할 및 벡터화
3.  **Retrieval**: 검색 (BM25 + Dense + Rerank)
4.  **Answer Generation & Post-processing**: LLM 기반 응답 생성 및 후처리

모든 단계는 독립적인 모듈로 설계하며, `Service` 계층에서 조합하여 사용합니다.

---

## 2. RAG Architecture Structure

RAG 관련 모듈의 폴더 구조는 프로젝트 전체 표준을 따릅니다. 각 단계는 **단일 책임 원칙(SRP)**을 준수해야 합니다.
> 자세한 AI 관련 폴더 구조는 `Folder_Standards.md` 문서를 참조하세요.

---

## 3. Ingestion Rules

### 3.1 문서 종류
-   PDF, HWP(파싱 API 기반), DOCX
-   HTML, JSON, CSV
-   공공기관 문서

### 3.2 Ingestion 기본 규칙
-   텍스트 추출 후 반드시 **정제(cleaning)** 과정을 수행합니다.
    -   불필요한 항목 제거: 목차, 페이지 번호, 반복 헤더/푸터, 광고 등
-   문서 **메타데이터**를 추가합니다.
    -   `source`: 출처
    -   `type`: 문서 유형
    -   `section`: 섹션 정보
    -   `date`: 작성일

### 3.3 파주시 프로젝트 특화 전처리
-   "타 지자체" 언급은 감지 후 주석 처리하거나 별도 필드에 저장합니다.
-   전화번호, 부서명 등은 `meta={"contact": "...", "department": "..."}` 형태로 저장해 활용합니다.

---

## 4. Chunking Strategy

### 4.1 기본 chunk 규칙
-   기본 `chunk_size`: 800
-   `overlap`: 200
-   문서 크기에 따라 탄력적으로 조절 가능합니다.

### 4.2 Chunk 생성 원칙
-   의미 단위 기반 분절(문단·표·섹션 중심)을 우선합니다.
-   매우 긴 표나 법령은 별도 청크 타입으로 관리합니다.
-   `Whisper` 또는 `OCR`로부터 온 텍스트는 노이즈 제거를 우선 적용합니다.

### 4.3 Chunk 저장 형식
-   하나의 `chunk`는 다음 필드를 가집니다: `id`, `text`, `source`, `section`, `page`(optional), `embedding`(벡터)

---

## 5. Embedding Rules

### 5.1 기본 모델
-   `jhgan/ko-sbert-nli` (한국어 문서 RAG 최적)
-   `intfloat/e5-base`
-   다국어 필요 시: `multilingual-e5-base`

### 5.2 Embedding 정책
-   모든 `chunk`는 문자열 전처리 후 임베딩합니다.
-   길이가 너무 짧거나 긴 `chunk`는 제거하거나 분할 처리합니다.

### 5.3 저장소
-   Local `FAISS` (GPU/CPU) 사용을 기본으로 합니다.
-   ID와 메타데이터 매핑은 별도 DB나 JSONL 파일로 관리합니다.

---

## 6. Retrieval Rules

### 6.1 Hybrid Retrieval
1.  **1단계**: `BM25` 키워드 검색
2.  **2단계**: `Dense` 벡터 검색
3.  **3단계**: 두 결과를 합쳐 상위 N개 후보 생성

### 6.2 Reranking
-   `Cross-Encoder` 또는 LLM 기반 `rerank`를 적용합니다.
-   Top-N (예: 20개) → Top-K (3~5개)로 최종 정제합니다.
-   Rerank 점수는 `score` 필드로 포함합니다.

### 6.3 Retrieval 품질 규칙
-   검색 결과에 타 지자체 문서가 섞일 경우, "파주시", "복지" 등 핵심 키워드에 가중치를 부여하여 조정합니다.
-   동일하거나 출처가 불분명한 `chunk`는 후순위로 처리하거나 제외합니다.

---

## 7. Answer Generation Rules

### 7.1 Prompt 정책
-   **System Prompt**: "문서 기반 전문가" 역할 부여
-   사용자 질문과 선택된 `chunk`를 컨텍스트로 넣어 답변을 생성합니다.
-   컨텍스트 외 정보 추측을 금지하고, 근거 문장을 반드시 포함하도록 지시합니다.

### 7.2 출력 형태
-   `answer`: 최종 응답
-   `sources`: 참조한 `chunk` ID 리스트
-   `reasoning`(optional): 선택 근거 (내부용)
-   `followup_questions`(optional): 사용자가 추가로 물어볼 만한 질문

### 7.3 정부/공공 문서 응답 규칙
-   문서 기반 답변을 우선하며, 최신성 보장이 불가능한 경우 주의 문구를 포함합니다.
-   어려운 행정 용어는 예시와 함께 설명합니다.

---

## 8. Post-Processing Rules

### 8.1 텍스트 정제
-   중복 문장을 제거하고 불필요한 접속사를 축소하여 간결하게 만듭니다.
-   "사용자에게 읽히는 문장" 기준으로 자연스럽게 재구성합니다.

### 8.2 파주시 특화 후처리
-   다른 지자체 관련 문장이 포함된 경우 필터링합니다.
-   제도/지원금 문의 시 "공식 문의처" 정보를 자동으로 추가합니다.

### 8.3 JSON 포맷 예시

최종 응답은 다음과 같은 구조를 따르는 것을 권장합니다.
```json
{
  "answer": "...",
  "sources": ["paju_2024_welfare_01#chunk_021", "paju_guideline#chunk_004"],
  "followup": ["관련 서류는 무엇인가요?", "신청 가능한 온라인 페이지는 있나요?"]
}
```

---

## 9. LangGraph Workflow Rules

### 9.1 RAG Pipeline Graph 구성
-   **필수 노드**: `preprocess_node`, `retrieval_node`, `rerank_node`, `generation_node`, `postprocess_node`

### 9.2 State 구조
-   `query`, `user_meta`(optional), `retrieved_docs`, `reranked_docs`, `answer`, `sources`

### 9.3 에러 처리
-   `retrieval` 실패 시 빈 리스트를 반환합니다.
-   `chunk`가 없는 경우 "문서 내 근거 없음" 메시지를 생성합니다.
-   시스템 오류 시 `fallback` 응답을 생성합니다.

---

## 10. Evaluation Rules

### 10.1 품질 지표
-   `Recall@K`, `Precision@K`
-   Rerank 성능
-   `Answer Faithfulness` (문서 근거 충실도)
-   `Hallucination score`

### 10.2 자동 테스트
-   `Known-answer` 베이스라인과 비교합니다.
-   `Critical keyword` 커버리지를 검사합니다.
-   `Chunk alignment`를 검증합니다.

---

## 11. 운영(Production) 규칙

### 11.1 Index Versioning
-   매 `ingest`마다 `index_version=YYYYMMDD_vX` 규칙을 적용합니다.
-   오래된 인덱스는 제거하고, 문서 변경 시 해당 문서는 전부 재임베딩합니다.

### 11.2 Monitoring
-   `Retrieval hit ratio`
-   `LLM latency`
-   `STT/TTS pipeline latency` (음성 RAG의 경우)
-   사용자 섹션별 질문량

### 11.3 데이터 보안
-   민감 정보가 포함된 `chunk`는 임베딩에서 제외합니다.
-   전화번호, 주민번호 등은 마스킹 후 저장합니다.

End of KevinCY-Kodex RAG Guideline