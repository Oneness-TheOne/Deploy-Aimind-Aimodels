# Chatbot (가이드 / 심리분석)

AiMind AI 모델 서비스에서 사용하는 **챗봇 모듈**입니다.

## 구성

| 파일 | 역할 |
|------|------|
| `guideChatbot.py` | 웹/서비스 **가이드 챗봇** — LangChain + ChromaDB RAG + Gemini |
| `psychologicalAnalysisChatbot.py` | **심리 분석 추가 질문** 답변 — 분석 결과 컨텍스트 기반 LLM 응답 |

## 기술

- **LangChain**: ChromaDB 검색, ChatGoogleGenerativeAI(Gemini)
- **HuggingFace Embeddings**: RAG용 문서 임베딩
- **guides/** — 가이드 문서(마크다운 등) RAG 소스

## 사용

`main.py`에서 다음처럼 import하여 API 라우트에서 호출합니다.

- `ask_to_website_guide_chatbot`: 가이드 질의
- `get_answer_for_more_question_about_analysis`: 심리 분석 추가 질문

환경 변수 `GEMINI_API_KEY`(또는 `GOOGLE_API_KEY`) 필요.
