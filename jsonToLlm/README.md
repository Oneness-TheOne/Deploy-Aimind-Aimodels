# jsonToLlm (JSON → RAG/LLM 해석)

그림 분석 JSON을 **ChromaDB RAG + LLM(Gemini)**으로 해석하는 파이프라인입니다.

## 워크플로우

1. **ingest**: 논문 PDF → 청킹/지표 추출 → `results/` JSON → 벡터화 → ChromaDB 저장  
2. **interpret**: 원본 그림 JSON → 중간 분석 → ChromaDB 검색(RAG) → LLM → 해석 결과

## 주요 파일

| 파일 | 설명 |
|------|------|
| `main.py` | CLI 진입점 (`ingest`, `interpret` 명령) |
| `store_to_chroma.py` | JSON → ChromaDB 벡터 DB 적재 |
| `gemini_integration.py` | Gemini API 연동 |
| `htp_indicator_parser.py` | PDF → HTP 지표 JSON 변환 |
| `interpretation_prompts.py` | 해석용 프롬프트 |
| `tree_analyzer.py` 등 | 객체별 분석·해석 로직 |
| `run_batch_interpret.py` | 배치 해석 실행 |

## 디렉터리

- `htp_knowledge_base/` — ChromaDB 저장 경로 (RAG 지식 베이스)
- `thesis/` — 논문 PDF 소스
- `results/` — ingest 결과 JSON, chunks 등

## 사용

- **ingest**: PDF 또는 기존 JSON을 ChromaDB에 적재  
- **interpret**: `image_to_json` 결과 JSON 경로를 주어 해석 문장 생성  

환경 변수 `GEMINI_API_KEY` 필요.  
API에서는 AiModels 서비스의 `/interpret` 등이 이 파이프라인을 호출합니다.
