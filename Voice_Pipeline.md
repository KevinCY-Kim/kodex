KevinCY-Kodex — Voice Pipeline Guideline

Version: 2025.11
Scope: STT · RAG · LangGraph · TTS · 공공/오피스 챗봇 음성 대화 흐름

1. Overview

음성 기반 질의응답 파이프라인은 다음 4단계로 구성된다.

Audio Input (음성 업로드)

STT (Speech-to-Text 변환)

RAG 또는 일반 LLM 기반 답변 생성

TTS (Text-to-Speech 변환)

이 흐름 전체를 FastAPI의 단일 서비스 함수로 묶어 제공하며, Router에서는 파일 수신과 응답 반환만 담당한다.

2. File Structure

권장 폴더 구조는 다음과 같다.

/app/ai
│
├── stt/
│   ├── stt_whisper.py
│   └── stt_utils.py
│
├── tts/
│   ├── tts_gtts.py
│   └── tts_utils.py
│
└── voice/
    ├── voice_pipeline.py
    └── voice_service.py

3. Audio Input Rules
3.1 업로드 규칙

사용자 음성은 UploadFile 형태로 수신한다.

허용 확장자: wav, mp3, m4a, ogg

파일 크기 제한(예: 5~10MB) 적용 가능

파일 저장 경로는 /tmp/{user_id}_{timestamp}.{ext} 형태로 생성한다.

3.2 보안 규칙

업로드된 파일은 처리 완료 후 자동 삭제한다.

업로드된 파일명을 그대로 시스템 경로로 사용하지 않는다.

MIME 타입 확인 후 검증된 오디오 파일만 처리한다.

4. STT (Speech-to-Text) Specification
4.1 엔진 선택

권장 모델:

Faster-Whisper (GPU 사용 가능한 경우)

Whisper Small/Medium (CPU 또는 제한된 GPU 환경)

4.2 STT 옵션 규칙

language: auto 또는 ko

noise_suppression: on (환경 따라 적용)

beam_size: 1~3

vad_filter: 필요 시 활성화

4.3 STT 출력 포맷

transcribed_text: 문자열

confidence(optional): 신뢰도

segments(optional): 시간 구간 정보

4.4 STT 품질 보정 규칙

연속단어/짤림 보정

“음 그…” 등 불필요한 잡음 제거

대명사(이거/저거/그분) → 질문 문맥에 맞게 재구성 가능

5. Query Reconstruction (Optional)

음성은 불명확한 문장이 많기 때문에 다음과 같은 재구성 규칙을 적용한다.

자연스러운 질문 형태로 자동 변환

누락된 단어를 문맥 기반으로 보완

TTS 친화적 문장 길이로 분할

예)
“출산지원금 어디서 받지?”
→ “파주시 출산지원금 신청 장소와 절차가 궁금합니다.”

6. RAG / LLM Answer Generation
6.1 선택 기준

공공문서/내부문서 기반 정보 → RAG 우선

잡담형·일상 대화 → 일반 LLM 답변

6.2 RAG 기준

Chunk retrieval

Rerank 적용

문서 기반 신뢰도 우선

근거 문장을 반드시 포함

6.3 답변 출력 형태

text_answer

source_chunks(optional)

structured_json(optional)

followup_questions(optional)

7. TTS (Text-to-Speech)
7.1 엔진

gTTS (간단한 MVP/PoC 용)

또는 상용/오픈소스 TTS 엔진 확장 가능

7.2 TTS 규칙

텍스트는 120~200자 단위로 자연스럽게 끊는다.

어르신 친화 옵션(slow)을 제공한다.

음질 강화 옵션(최대 bit rate 적용)

7.3 파일 출력 위치

/static/tts/{user_id}_{timestamp}.mp3

URL 형태 /static/tts/... 로 반환

7.4 TTS 파일 관리

일정 기간(예: 24~72시간) 후 삭제

mp3/wav 변환 오류 대비 예외 처리

8. FastAPI Router Rules
8.1 Router 역할

파일 수신

임시 파일 저장

서비스 호출

서비스 결과(JSON + TTS URL) 반환

8.2 Router 응답 형식
{
  "query": "사용자 질문",
  "answer": "생성된 답변",
  "tts_url": "/static/tts/xxx.mp3",
  "sources": [...]
}

9. Service Layer Workflow
9.1 전체 흐름

STT로 텍스트 변환

텍스트 정제/질문 재구성

LangGraph RAG 또는 일반 LLM으로 답변 생성

후처리(post-processing)

TTS 엔진으로 음성 생성

JSON + 오디오 파일 경로 반환

9.2 예외 상황 처리

STT 실패 → “음성이 인식되지 않았습니다.”

RAG/LLM 실패 → 기본 안내 메시지

TTS 실패 → 텍스트만 전달

10. Post-Processing Rules
10.1 텍스트 정제

문장을 너무 길게 만들지 않는다.

음성 출력 시 흐름이 자연스럽도록 pause(punctuation) 추가

공공문서 답변은 정중·안정적 톤 유지

10.2 파주시 특화

타 지자체 언급 감지 시 파주시 기준 재검사

변동성 있는 제도는 ‘최신 정보는 공식 사이트에서 재확인’ 문구 자동 추가

11. Logging Rules
필수 로그

user_id

STT 처리 시간

RAG/LLM 응답 시간

TTS 생성 시간

전체 latency

오류 코드

로그는 JSON 구조로 저장하는 것을 권장한다.

12. Quality Evaluation
STT 품질 지표

WER (Word Error Rate)

발화 길이 대비 인식 정확도

RAG 품질 지표

Recall@K

Faithfulness (근거 충실도)

사용자 경험 지표

응답 속도

음성 파일 재생 성공률

사용자 재질문 비율

13. 확장 전략

Faster-Whisper GPU acceleration

상업용 TTS(Neural TTS)로 교체

스트리밍 음성 입력(WebRTC)

음성 스트리밍 응답(TTS streaming)

감정/톤 조절 모델 연동

End of KevinCY-Kodex Voice Pipeline Guideline