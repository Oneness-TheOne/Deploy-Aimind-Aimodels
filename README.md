# Deploy-Aimind-Aimodels (아이마음 AI 모델)

**아이마음**은 보호자·상담사가 아동의 HTP(나무·집·사람) 그림을 업로드하면, AI가 분석·해석과 T-Score를 제공하는 **아동 심리 지원 플랫폼**입니다.  
이 저장소는 **HTP 그림 분석·해석·챗봇**을 담당하며, YOLO 객체 감지 → 구조화 분석 → ChromaDB RAG + LLM(Gemini) 해석을 한 번에 처리하는 FastAPI 기반 AI 모델 서비스입니다.

---

## 데모

- 웹에서 그림 분석·챗봇 사용: [http://43.203.135.230.nip.io:3000/](http://43.203.135.230.nip.io:3000/) (발표·시연용)

---

## 아이마음 프로젝트 구성

아이마음은 4개 저장소로 구성됩니다. 이 저장소(Aimodels)는 HTP 그림 분석·해석·T-Score·챗봇을 담당합니다.

```
[사용자] → Frontend(3000) → Backend(8000) → Aimodels(8080) / OCR(8090)
```

| 저장소                                 | 역할                                                                       | 기본 포트 |
| -------------------------------------- | -------------------------------------------------------------------------- | --------- |
| **Deploy-Aimind-Frontend**             | 웹 UI (로그인, 그림 분석, 그림일기 OCR, 커뮤니티, 마이페이지)              | 3000      |
| **Deploy-Aimind-Backend**              | REST API (인증, 사용자/아동, 분석·OCR 저장, 커뮤니티), AiModels·OCR 프록시 | 8000      |
| **Deploy-Aimind-Aimodels** (본 저장소) | HTP 그림 분석·해석·T-Score·챗봇 (YOLO + RAG + Gemini)                      | 8080      |
| **Deploy-Aimind-OCR**                  | 그림일기 이미지 → 텍스트 추출 (VLM + Gemini)                               | 8090      |

**전체 서비스 실행 순서 (로컬):** 1) Backend → 2) Aimodels → 3) OCR → 4) Frontend.  
Backend `.env`에 `AIMODELS_BASE_URL`, `OCR_BASE_URL` 설정. Frontend `NEXT_PUBLIC_*`를 같은 주소로 맞추면 됩니다.

**누구를 위한 서비스인가요?**

- **보호자**: 자녀의 HTP 그림 업로드 → AI 해석·T-Score·추천 사항 확인
- **상담사·교육기관**: 아동 심리 지원 참고 자료
- **개발자**: 그림 분석·RAG 파이프라인 참고 및 확장

---

## 파이프라인 개요

| 단계 | 입력                                              | 출력                                                   | 담당 모듈                                |
| ---- | ------------------------------------------------- | ------------------------------------------------------ | ---------------------------------------- |
| ①    | 아동 그림 4장(나무/집/남자/여자) + 이름·나이·성별 | 4개 그림별 **RAG 포맷 JSON** (ratio, center_x/y 등)    | `image_to_json` (YOLO)                   |
| ②    | ①의 RAG JSON                                      | **구조화 분석 JSON** + RAG 검색 쿼리                   | `jsonToLlm` (legacy 변환, tree_analyzer) |
| ③    | ②의 쿼리                                          | **HTP 전문 지식 검색** (ChromaDB k=10) → LLM 컨텍스트  | `jsonToLlm` (ChromaDB RAG)               |
| ④    | 구조화 분석 + RAG 컨텍스트                        | **그림별 심리 해석 JSON** + **종합 심리 결과** (4필드) | `jsonToLlm` (Gemini)                     |
| ⑤    | 분석 결과 + 아동 정보                             | **T-Score** (에너지/위치안정성/표현력), 또래 비교      | `drawing_score`, `analysis_metrics`      |

- **YOLO 출력(정규화 좌표)** ↔ **tree_analyzer(픽셀 bbox)** 형식 차이는 `legacy_converter`로 통일합니다.
- 수치만 LLM에 넘기면 해석 편차가 생기므로, **위치·요소를 언어화**한 뒤 RAG 쿼리로 사용해 일관된 해석을 유도합니다.
- **T-Score**: 손성희(2015) 모바일 HTP 표준화 연구 기준, 평균 50·표준편차 10 해석. 또래 평균 T는 정의상 50입니다.

---

## 기능

- **그림 분석** (`POST /analyze`): HTP 4장 업로드 → YOLO 객체 감지 → RAG 형식 JSON → 구조화 분석 → ChromaDB RAG + Gemini 해석 → 그림별 해석 + 종합 심리 결과 + T-Score·또래 비교·추천 사항
- **T-Score 산출** (`POST /analyze/score`): 이미 분석된 `results`와 아동 정보(나이·성별)로 에너지/위치안정성/표현력 T-Score 계산
- **챗봇** (`POST /chatbot`):
  - **가이드 챗봇**: 웹/서비스 이용 안내 (RAG + Gemini)
  - **심리 분석 챗봇**: `analysis_context`에 분석 결과를 넣으면, 그 결과 기반 추가 질문 답변

---

## 모델 구성 (YOLO)

| 모델명           | 클래스 수 | 주요 탐지 클래스                          |
| ---------------- | --------- | ----------------------------------------- |
| `tree_mvp_14cls` | 14        | 기둥, 수관, 가지, 나뭇잎, 열매, 꽃, 새 등 |
| `house_15cls`    | 15        | 지붕, 집벽, 문, 창문, 굴뚝, 울타리, 길 등 |
| `man_18cls`      | 18        | 머리, 얼굴, 눈, 코, 입, 목, 팔, 다리 등   |
| `woman_18cls`    | 18        | 머리, 얼굴, 눈, 코, 입, 목, 팔, 다리 등   |

가중치 경로: `image_to_json/{object_type}_weights/{gender}/best.pt` (예: `tree_weights/male/best.pt`).

---

## 기술 스택

- **Python 3.11**, FastAPI, Uvicorn
- **PyTorch (CPU)**, Ultralytics YOLO (세그멘테이션/객체 감지)
- **ChromaDB**, LangChain, **Google Gemini** (RAG·해석·챗봇)
- **HuggingFace Embeddings** (가이드 챗봇 RAG용 벡터 임베딩)
- **Google Generative AI Embeddings** (`models/embedding-001`, jsonToLlm ChromaDB RAG용)
- **drawing_norm_dist_stats.csv** (T-Score 기준표), **data/label_stats_by_group.json** (또래 통계, 선택)

---

## 디렉터리 구조

| 경로                                      | 설명                                                                                   |
| ----------------------------------------- | -------------------------------------------------------------------------------------- |
| `main.py`                                 | FastAPI 앱: `/health`, `/analyze`, `/analyze/score`, `/chatbot`                        |
| `drawing_score.py`                        | T-Score 산출 (에너지/위치안정성/표현력)                                                |
| `analysis_metrics.py`                     | 이미지 메트릭·또래 비교 (label_stats 기반)                                             |
| `drawing_norm_dist_stats.csv`             | T-Score 기준표 (연령·성별·그림 유형별)                                                 |
| `chatbot/`                                | 가이드 챗봇, 심리분석 챗봇 (LangChain + ChromaDB + Gemini)                             |
| `image_to_json/`                          | HTP 이미지 → YOLO → RAG 포맷 JSON (`uploads/`, `result/` 생성)                         |
| `jsonToLlm/`                              | JSON → 구조화 분석, ChromaDB 저장, RAG 검색, Gemini 해석 (legacy 변환, 배치 해석 포함) |
| `jsonToLlm/htp_knowledge_base/`           | ChromaDB 벡터 DB (RAG용 HTP 지식 베이스)                                               |
| `jsonToLlm/results/`, `jsonToLlm/thesis/` | ingest 결과 JSON, 논문 PDF 소스                                                        |
| `data/`                                   | `label_stats_by_group.json` 등 (또래 통계, 없으면 비교 생략)                           |
| `weights/`                                | 공용 모델 가중치 (선택). 실제 YOLO 가중치는 `image_to_json/*_weights/` 사용            |

---

## 사전 요구사항

- **Python 3.11+**
- (선택) Docker — 이미지로 배포할 경우

---

## 빌드 및 실행

### 로컬 실행

```bash
# 의존성 설치
pip install -r requirements.txt

# 환경 변수 설정 (.env에 GEMINI_API_KEY 또는 GOOGLE_API_KEY)
# AIMODELS_PORT=8080 (기본값)

# 서버 실행 (기본 포트 8080)
python main.py
# 또는
uvicorn main:app --host 0.0.0.0 --port 8080
```

### Docker (Dockerfile 제공 시)

```bash
docker build -t aimind-aimodels .
docker run -p 8080:8080 --env-file .env aimind-aimodels
```

---

## 환경 변수

| 변수                                   | 설명                                            |
| -------------------------------------- | ----------------------------------------------- |
| `AIMODELS_PORT`                        | 서버 포트 (기본 `8080`)                         |
| `GEMINI_API_KEY` 또는 `GOOGLE_API_KEY` | Gemini API 키 (해석·챗봇·RAG)                   |
| `GEMINI_API_KEYS`                      | 복수 키 (쉼표 구분) 시 503 등 에러 시 순환 사용 |

---

## API 요약

| 메서드 | 경로             | 설명                                                                          |
| ------ | ---------------- | ----------------------------------------------------------------------------- |
| GET    | `/health`        | 헬스 체크                                                                     |
| POST   | `/analyze`       | 그림 4장 + 아동 정보 → 분석·해석·T-Score·종합 심리 결과·추천 사항             |
| POST   | `/analyze/score` | 기존 분석 결과(`results`) + 나이·성별 → T-Score만 산출                        |
| POST   | `/chatbot`       | `question` 필수. `analysis_context` 있으면 심리 분석 챗봇, 없으면 가이드 챗봇 |

- `/analyze`: `tree`, `house`, `man`, `woman` (파일), `child_name`, `child_age`, `child_gender` (Form).
- `/chatbot`: JSON body `{ "question": "...", "analysis_context": { ... } }` (analysis_context 선택).

---

## 참고

- **그림일기 OCR**은 본 저장소 범위 밖이며, 발표에서 언급한 LLAVA-ONEVISION 등 OCR 파이프라인은 별도 구성입니다.
- CORS는 `main.py`에서 localhost/127.0.0.1 기준으로 설정되어 있으므로, 배포 시 허용 오리진을 필요에 따라 수정하세요.
