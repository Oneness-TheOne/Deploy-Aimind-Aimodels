# AiMind AI Models (Docker)

그림 분석(HTP), 해석(RAG/LLM), 가이드/심리 챗봇을 제공하는 **FastAPI 기반 AI 모델 서비스**입니다.

## 기능

- **그림 분석** (`/analyze`): HTP(나무/집/여자/남자) 이미지 → YOLO 객체 감지 → JSON 결과
- **해석** (`/interpret`): 분석 JSON + ChromaDB RAG + LLM(Gemini) → 해석 문장
- **T-Score 산출**: 아동 정보와 분석 결과 기반 점수 계산
- **챗봇**: 웹 가이드 챗봇, 심리 분석 추가 질문 답변

## 기술 스택

- **Python 3.11**, FastAPI, Uvicorn
- **PyTorch (CPU)**, Ultralytics YOLO (이미지 분류)
- **ChromaDB**, LangChain, Google Gemini (RAG/LLM)
- **HuggingFace Embeddings** (벡터 임베딩)

## 디렉터리 구조

| 폴더 | 설명 |
|------|------|
| `chatbot/` | 가이드 챗봇, 심리분석 챗봇 (LangChain + Gemini) |
| `image_to_json/` | HTP 이미지 → JSON 변환 (YOLO 모델) |
| `jsonToLlm/` | JSON 해석 파이프라인, ChromaDB 저장, 배치 해석 |
| `chroma_db/` | 벡터 DB 저장 경로 (RAG용) |
| `data/` | 라벨 통계 등 참조 데이터 |
| `weights/` | YOLO 등 모델 가중치 |

## 빌드 및 실행

```bash
# 이미지 빌드
docker build -t aimind-aimodels .

# 실행 (포트 8080)
docker run -p 8080:8080 --env-file .env aimind-aimodels
```

## 환경 변수

- `AIMODELS_PORT`: 서버 포트 (기본 `8080`)
- `GEMINI_API_KEY` 또는 `GOOGLE_API_KEY`: Gemini API 키 (챗봇, 해석, RAG)

## API 예시

- `GET /health` — 헬스 체크
- `POST /analyze` — 그림 업로드 → 분석 JSON
- `POST /interpret` — 분석 결과 → LLM 해석
- `POST /chatbot/guide` — 가이드 챗봇 질의
- `POST /chatbot/psychological` — 심리 분석 추가 질문
