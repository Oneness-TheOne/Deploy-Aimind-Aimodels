# image_to_json (HTP 이미지 → JSON)

HTP(나무/집/여자/남자) 그림 이미지를 **YOLO 모델로 분석**해 객체 감지·분류 결과를 JSON으로 출력하는 모듈입니다.

## 역할

- 업로드 이미지에 대해 학습된 HTP 모델(tree, house, woman, man)로 객체 감지
- 출력 형식:
  - **raw**: bbox/segment 등 원시 결과 (개발·검증용)
  - **rag**: RAG용 세분화 형식 (객체별 한글명, 비율, 위치, 존재 여부, 요약 문장)

## 주요 파일

- `image_to_json.py` — CLI 및 분석 로직 (Ultralytics YOLO)
- `*_weights/` — 객체별 학습 가중치 (tree, house, woman, man 등)
- `uploads/` — 업로드 이미지 저장
- `result/` — 분석 결과 JSON 저장

## 모델 구성

- `tree_mvp_14cls`, `house_15cls`, `woman_18cls`, `man_18cls` 등 설정 사용
- 클래스는 한글 매핑(`CLASS_EN_TO_KR`)으로 RAG 해석에 사용

## 실행 예 (CLI)

```bash
python image_to_json.py --image "경로/이미지.jpg" --object tree
python image_to_json.py --image "경로/이미지.jpg" --object tree --format rag --output result.json
```

API에서는 `main.py`의 `/analyze` 엔드포인트를 통해 이 모듈을 호출합니다.
