# Models

Teachable Machine Pose export 모델을 이 폴더 아래에 버전별로 저장한다.

예상 구조:

```text
models/
  registry.json
  tm-pose-v1/
    model.json
    metadata.json
    weights.bin
    MODEL.md
```

모델 업로드 전 반드시 `docs/MODEL_GUIDE.md`를 확인한다.

주의:
- 기존 모델 ID의 파일을 조용히 교체하지 않는다.
- 새 학습 모델은 새 버전 폴더를 만든다.
- 영상/학습 원본 이미지 파일은 이 저장소에 올리지 않는 것을 기본으로 한다.
