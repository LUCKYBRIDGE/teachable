# 웹사이트 설정 정책

## 1. 원칙

현장에서 자주 바뀔 값은 모델 파일이나 소스코드에 하드코딩하지 않는다.
가능하면 Admin 화면에서 조절할 수 있게 한다.

설정은 세 층으로 구분한다.

1. AI 판정 설정
2. 카메라/화면 설정
3. 연결/운영 설정

초기 기본값은 가설이며 복도 실측 후 조정한다.

## 2. AI 판정 설정

| 항목 | 초기 제안 | 설명 |
|---|---:|---|
| modelId | tm-pose-v1 | 사용할 모델 |
| confidenceThreshold | 0.80 | 최소 신뢰도 |
| holdTimeMs | 800 | 같은 후보 동작 유지시간 |
| windowMs | 1000 | 최근 판정 관찰 범위 |
| sameClassRatio | 0.70 | 관찰 범위 내 동일 클래스 비율 |
| neutralResetMs | 500 | Neutral로 복귀하기 위한 유지시간 |
| cooldownMs | 1200 | 확정 직후 과도한 재전환 방지 |
| inferenceFps | 12 | AI 추론 목표 FPS |
| staleMs | 2500 | Display가 remote 상태를 오래된 것으로 판단하는 시간 |

이 값은 성능 검증 전 임시 기본값이다.

## 3. 동작별 override

global 설정 위에 필요한 동작만 개별 설정을 덮을 수 있게 한다.

예:
```json
{
  "hands_up": {
    "threshold": 0.85,
    "holdTimeMs": 700,
    "enabled": true
  },
  "sitting": {
    "threshold": 0.78,
    "holdTimeMs": 1000,
    "enabled": true
  }
}
```

필요한 이유:
- 모든 포즈가 동일 난이도가 아님
- 특정 클래스만 오탐이 많을 수 있음
- 모델 재학습 전 현장 튜닝이 가능함

## 4. 동작 활성화

모델에 클래스가 있어도 사이트에서 일시 비활성화할 수 있어야 한다.

예:
- neutral: 항상 활성
- hands_up: 활성
- one_hand_up: 활성
- arms_open: 활성
- sitting: 비활성

비활성 클래스는 최종 확정 action으로 사용하지 않는다.

## 5. 카메라 설정

초기 제공 후보:
- camera facing: front / rear
- mirror preview: on/off
- preferred resolution
- preview fit: contain / cover
- skeleton overlay: on/off
- debug prediction overlay: on/off
- inference FPS

카메라 해상도는 과도하게 높이지 않는다.
표시용 해상도와 AI 입력 해상도는 같을 필요가 없다.

## 6. Display 설정

- 셀카 거울 on/off
- 거울 좌우반전 on/off
- confidence 표시 on/off
- confidence progress bar on/off
- 전체 클래스 확률 표시 on/off
- 일반 모드 / AI 학습 모드
- 결과 영역 크기
- 상태 변경 애니메이션 on/off
- fullscreen 안내

TTS 설정은 두지 않는다.

## 7. 연결/운영 설정

- channel/room ID
- camera source ID
- stale timeout
- heartbeat 주기
- 연결 상태 표시
- debug mode

학교 복도 고정 설치에서는 기본 channel을 하나로 둘 수 있지만 코드에서 수정하기보다 설정으로 관리한다.

## 8. 설정 저장 위치

### 1단계

브라우저 `localStorage`를 사용한다.

장점:
- 구현 단순
- Firebase 설정과 독립
- 각 태블릿별 카메라/UI 설정에 적합

### 후속

필요하면 Firebase에 공유 설정을 둔다.

예:
```text
/hall/config
```

공유하기 좋은 설정:
- active model
- threshold
- hold time
- enabled actions

기기별로 남겨야 할 설정:
- camera device
- preview mirror
- display UI preference

## 9. 기본 설정과 고급 설정

Admin UI는 모든 값을 한 화면에 노출하지 않는다.

기본:
- 모델
- 최소 신뢰도
- 유지시간
- 동작 ON/OFF
- 신뢰도 표시
- 관절 표시

고급:
- window
- same class ratio
- neutral reset
- cooldown
- inference FPS
- stale timeout
- heartbeat
- debug overlay

## 10. 설정 검증

모든 입력은 범위를 검증한다.

예:
- threshold: 0.50 ~ 0.99
- holdTimeMs: 100 ~ 5000
- sameClassRatio: 0.50 ~ 1.00
- inferenceFps: 1 ~ 30

비정상 값이 저장되어 앱이 멈추지 않도록 schema validation을 둔다.

## 11. 추천 preset

현장 튜닝을 쉽게 하기 위해 후속으로 preset을 제공할 수 있다.

- 빠른 반응
- 균형
- 안정 우선

preset은 개별 설정의 묶음일 뿐 모델을 바꾸지 않는다.
