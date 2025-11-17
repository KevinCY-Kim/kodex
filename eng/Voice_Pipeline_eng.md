# KevinCY-Kodex — Voice Pipeline Guideline

**Version:** 2025.11  
**Scope:** STT · RAG · LangGraph · TTS · Public/Office Chatbot Voice Conversation Flow

---

## 1. Overview

The voice-based Q&A pipeline consists of the following 4 stages.

1.  **Audio Input**: Voice file upload.
2.  **STT (Speech-to-Text)**: Conversion of speech to text.
3.  **Answer Generation**: RAG or general LLM-based answer generation.
4.  **TTS (Text-to-Speech)**: Conversion of text to speech.

This entire flow is bundled into a single `Service` function in `FastAPI`, with the `Router` only responsible for file reception and response return.

---

## 2. File Structure

Voice processing related files are placed according to the `KevinCY-Kodex` standard folder structure.

-   **STT/TTS Modules**: `/app/ai/stt/`, `/app/ai/tts/`
    -   As part of the AI tech stack, they are located within the `ai` directory.
-   **Voice Pipeline Workflow**: `/app/ai/voice_pipeline.py`
    -   As an AI workflow connecting STT, RAG, and TTS, it is located in the `ai` directory.
-   **Voice Service**: `/app/services/voice_service.py`
    -   As business logic connecting the router and the AI pipeline, it is located in the `services` directory.

> For a detailed overall folder structure, please refer to the `Folder_Standards_eng.md` document.

---

## 3. Audio Input Rules

### 3.1 Upload Rules
-   Receive user's voice as an `UploadFile`.
-   Allowed extensions: `wav`, `mp3`, `m4a`, `ogg`.
-   A file size limit (e.g., 5-10MB) can be applied.
-   Generate file storage path in the format `/tmp/{user_id}_{timestamp}.{ext}`.

### 3.2 Security Rules
-   Automatically delete uploaded files after processing is complete.
-   Do not use the uploaded filename directly as a system path.
-   Process only verified audio files after checking the `MIME` type.

---

## 4. STT (Speech-to-Text) Specification

### 4.1 Engine Selection
-   **Recommended Models**:
    -   `Faster-Whisper` (if GPU is available).
    -   `Whisper Small/Medium` (for CPU or limited GPU environments).

### 4.2 STT Option Rules
-   `language`: `auto` or `ko`.
-   `noise_suppression`: `on` (apply depending on the environment).
-   `beam_size`: 1-3.
-   `vad_filter`: Activate if necessary.

### 4.3 STT Output Format
-   `transcribed_text`: string.
-   `confidence`(optional): Confidence score.
-   `segments`(optional): Time segment information.

### 4.4 STT Quality Correction Rules
-   Correct for repeated words/cut-offs.
-   Remove unnecessary noise like "um, uh...".
-   Pronouns (this/that/he) can be rephrased to fit the question context.

---

## 5. Query Reconstruction (Optional)

Since spoken language can be unclear, the following reconstruction rules can be applied.

-   Automatically convert to a natural question form.
-   Supplement missing words based on context.
-   Split into TTS-friendly sentence lengths.
-   **Example**: "Where get maternity grant?" → "I would like to know the location and procedure for applying for the Paju City maternity grant."

---

## 6. RAG / LLM Answer Generation

### 6.1 Selection Criteria
-   **Prioritize RAG**: For information based on public/internal documents.
-   **General LLM**: For conversational/daily-life chat.

### 6.2 RAG Criteria
-   Apply `Chunk` retrieval and `Rerank`.
-   Prioritize document-based reliability.
-   Must include supporting sentences.

### 6.3 Answer Output Format
-   `text_answer`
-   `source_chunks`(optional)
-   `structured_json`(optional)
-   `followup_questions`(optional)

---

## 7. TTS (Text-to-Speech)

### 7.1 Engine
-   `gTTS` (for simple MVP/PoC).
-   Expandable to commercial/open-source TTS engines.

### 7.2 TTS Rules
-   Naturally break text into 120-200 character units.
-   Provide an elderly-friendly option (`slow`).
-   Consider a sound quality enhancement option (apply max bit rate).

### 7.3 File Output
-   **Path**: `/static/tts/{user_id}_{timestamp}.mp3`
-   **Return**: URL format (`/static/tts/...`)

### 7.4 TTS File Management
-   Delete after a certain period (e.g., 24-72 hours).
-   Implement exception handling for `mp3`/`wav` conversion errors.

---

## 8. FastAPI Router Rules

### 8.1 Router Role
-   File reception and temporary storage.
-   Service invocation.
-   Return of service result (JSON + TTS URL).

### 8.2 Router Response Format
```json
{
  "query": "User's question",
  "answer": "Generated answer",
  "tts_url": "/static/tts/xxx.mp3",
  "sources": [...]
}
```

---

## 9. Service Layer Workflow

### 9.1 Overall Flow
1.  Convert to text with STT.
2.  Refine text/reconstruct question.
3.  Generate answer with LangGraph RAG or general LLM.
4.  Post-processing.
5.  Generate speech with TTS engine.
6.  Return JSON and audio file path.

### 9.2 Exception Handling
-   **STT Failure**: "Speech was not recognized."
-   **RAG/LLM Failure**: Default guidance message.
-   **TTS Failure**: Deliver text only.

---

## 10. Post-Processing Rules

### 10.1 Text Refinement
-   Do not make sentences too long.
-   Add `pause`(punctuation) for a natural flow in speech output.
-   Maintain a polite and stable tone for public document answers.

### 10.2 Paju City Specifics
-   If mentions of other municipalities are detected, re-check against Paju City standards.
-   For volatile systems, automatically add 're-verify the latest information on the official site'.

---

## 11. Logging Rules

-   **Required Log Items**: `user_id`, STT processing time, RAG/LLM response time, TTS generation time, total `latency`, error code.
-   It is recommended to store logs in a JSON structure.

---

## 12. Quality Evaluation

-   **STT Quality Metrics**: `WER` (Word Error Rate), recognition accuracy relative to utterance length.
-   **RAG Quality Metrics**: `Recall@K`, `Faithfulness` (adherence to evidence).
-   **User Experience Metrics**: Response speed, audio file playback success rate, user re-query rate.

---

## 13. Expansion Strategy

-   `Faster-Whisper` GPU acceleration.
-   Replace with commercial TTS (`Neural TTS`).
-   Streaming voice input (`WebRTC`).
-   Streaming voice response (TTS streaming).
-   Integration with emotion/tone adjustment models.

End of KevinCY-Kodex Voice Pipeline Guideline