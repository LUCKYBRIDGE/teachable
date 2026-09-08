# Teachable Machine Pose 모델 관리 지침

## 1. 목적

AI 모델 파일을 애플리케이션 코드와 독립적으로 버전 관리하고, 복도 현장 테스트에서 모델을 빠르게 교체하거나 롤백하기 위한 규칙이다.

## 2. 모델의 책임

Teachable Machine Pose 모델은 현재 프레임의 자세에 대해 클래스별 확률을 반환한다.

예:
```text
neutral       0.03
one_hand_up   0.04
hands_up      0.91
arms_open     0.02
```

모델이 담당하지 않는 것:
- 0.8초 유지 여부
- 최근 10프레임 평균
- 쿨다운
- Neutral 복귀
- Firebase 전송 주기
- 화면 표시 방식

위 항목은 웹앱의 Stabilizer/Settings 계층에서 처리한다.

## 3. Export 방식

Teachable Machine에서 Pose Project를 학습한 뒤 TensorFlow.js 형식으로 다운로드한다.

모델 한 버전은 최소 다음 파일 묶음을 그대로 보존한다.

```text
public/models/tm-pose-v1/
  model.json
  metadata.json
  weights.bin
```

실제 export 파일명이 다르면 원본 참조 관계를 깨뜨리지 않는다.

## 4. 권장 모델 폴더

```text
public/
  models/
    registry.json
    tm-pose-v1/
      model.json
      metadata.json
      weights.bin
      MODEL.md
    tm-pose-v2/
      ...
```

모델 폴더 이름은 변경되지 않는 stable ID로 사용한다.

권장:
- `tm-pose-v1`
- `tm-pose-v2`
- `tm-pose-hall-v3`

비권장:
- `new-model`
- `final`
- `final2`
- `real-final`

## 5. 모델 registry

앱이 폴더를 직접 추측하지 않도록 registry를 둔다.

예시:
```json
{
  "models": [
    {
      "id": "tm-pose-v1",
      "name": "복도 기본 포즈 v1",
      "type": "teachable-machine-pose",
      "path": "models/tm-pose-v1/",
      "enabled": true
    }
  ]
}
```

실제 schema는 구현 단계에서 TypeScript 타입과 함께 확정한다.

## 6. 클래스 이름 규칙

가능하면 Teachable Machine 학습 단계부터 영문 stable class ID를 사용한다.

권장:
- `neutral`
- `one_hand_up`
- `hands_up`
- `arms_open`
- `sitting`
- `bowing`

한국어 화면명은 별도 Action Mapper에 둔다.

예:
```text
model class  : hands_up
actionId     : hands_up
display label: 만세
```

이렇게 해야 모델 교체와 UI 문구 변경이 서로 영향을 덜 준다.

## 7. Neutral 클래스

Neutral은 필수로 둔다.

Neutral 데이터에는 특정 목표 자세를 취하지 않는 자연스러운 기본 상태를 충분히 포함한다.

이유:
목표 클래스만 있으면 AI는 어떤 입력이든 그중 하나로 억지 분류할 수 있다.

## 8. 학습 데이터 수집 원칙

초기 권장 원칙:

- 클래스별 샘플 수를 가능한 한 균형 있게 맞춘다.
- 한 사람만으로 학습하지 않는다.
- 키와 체형이 다른 사람을 포함한다.
- 팔의 각도와 자세 변형을 적절히 포함한다.
- 실제 설치 장소의 조명/배경 조건을 포함한다.
- 너무 비슷한 연속 프레임만 대량으로 채우지 않는다.
- 카메라와 사람 사이 실제 설치 거리를 학습/검증에 반영한다.
- 역광, 지나가는 사람, 부분 가림 등 실패 조건을 별도 검증한다.

## 9. 모델 버전 기록

각 모델 폴더의 `MODEL.md`에 다음을 기록한다.

- 모델 ID
- 생성일
- 클래스 목록
- 학습 목적
- 기존 모델 대비 변경점
- 학습 환경
- 알려진 혼동 클래스
- 권장 threshold
- 테스트 결과
- 폐기/활성 여부

모델 파일을 바꾸면서 같은 ID를 재사용하지 않는다.
동일 ID는 동일 모델을 의미해야 한다.

## 10. 모델 선택과 롤백

Admin에서 registry의 모델을 선택할 수 있게 한다.

예:
```text
복도 기본 포즈 v1
복도 조명 보완 v2
앉기 개선 v3
```

새 모델이 실제 현장에서 나쁘면 이전 모델로 즉시 복귀 가능해야 한다.

## 11. 모델 품질 평가

모델 선택은 학습 화면의 training accuracy만으로 결정하지 않는다.

실제 설치 환경에서 다음을 확인한다.

- 기본 자세 오탐
- 손 들기 vs 만세 혼동
- 양팔 벌리기 vs 한 손 들기 혼동
- 앉기 진입/복귀
- 학생 키 차이
- 옷 색/형태 차이
- 조명 변화
- 복도 배경
- 카메라 거리
- 옆 학생 통과

## 12. 모델 파일 변경 시 주의

- `model.json`과 `weights.bin` 관계를 깨뜨리지 않는다.
- 모델 바이너리를 텍스트 편집하지 않는다.
- GitHub Pages base path를 고려해 URL을 생성한다.
- 캐시 때문에 이전 모델이 남는 경우 버전 ID/경로를 바꾼다.
- 모델 로드 실패는 사용자에게 명시적 오류 화면으로 보여준다.

## 13. 후속 모델 교체

Teachable Machine의 한계를 넘는 요구가 생기면 model adapter만 교체한다.

후보:
- MediaPipe Pose + 규칙/분류기
- TensorFlow.js custom classifier
- 시간축 Action Recognition 모델

단, Firebase payload와 Display UI 계약은 가능한 한 유지한다.
