# 시스템 아키텍처

## 1. 전체 구조

```text
GitHub Pages
  |
  +-- Tablet A: AI Camera
  |     Camera
  |       -> Pose Model
  |       -> Raw Predictions
  |       -> Stabilizer
  |       -> Action Mapper
  |       -> Firebase Sender
  |
  +-- Tablet B: AI Mirror
  |     Selfie Camera
  |       + Firebase Listener
  |       -> Display State
  |
  +-- Admin
        Model / Threshold / Hold / UI Settings

                 |
        Firebase Realtime Database
        current state only
```

## 2. 기술 기준

초기 구현 기준:

- Vite
- TypeScript
- 브라우저 MediaDevices API
- Teachable Machine Pose
- TensorFlow.js 계열 런타임
- Firebase modular Web SDK
- GitHub Pages

UI 프레임워크는 초기 필수 조건이 아니다. 단순한 구조를 우선하고, 실제 UI 복잡도가 커질 때만 도입한다.

## 3. 런타임 역할

### Camera App

책임:
- 카메라 device 선택 및 stream 생명주기
- Teachable Machine 모델 로드
- 프레임 추론
- raw prediction 보관
- prediction stabilization
- Action ID 매핑
- Firebase 상태 발행

책임 아님:
- 학생 데이터 기록
- 영상 업로드
- Display UI 세부 표현

### Display App

책임:
- 자체 전면 카메라를 거울처럼 표시
- Firebase current state 구독
- stale 상태 판정
- 확정 action을 큰 UI로 표시
- 선택적으로 confidence/후보 확률 표시

책임 아님:
- Pose 모델 추론
- Teachable Machine 클래스 해석
- AI 판정 안정화

### Admin App

책임:
- 모델 선택
- 판정 설정
- 표시 옵션
- 필요 시 Firebase 설정 동기화

## 4. 모델 Adapter

AI 엔진과 애플리케이션을 분리한다.

권장 인터페이스 개념:

```ts
type Prediction = {
  className: string;
  probability: number;
};

interface PoseClassifier {
  load(modelId: string): Promise<void>;
  predict(frame: CanvasImageSource): Promise<Prediction[]>;
  dispose(): Promise<void> | void;
}
```

초기 구현은 Teachable Machine Adapter이다.
향후 MediaPipe/custom model adapter를 추가할 수 있다.

Display/Firebase 계층은 adapter 변경을 몰라야 한다.

## 5. Action Mapper

모델의 클래스 이름을 표준 Action ID로 변환한다.

예:

```text
Teachable class: "Hands Up"
        |
        v
Action ID: "hands_up"
        |
        +-- label: "만세"
        +-- enabled: true
        +-- threshold: 0.85
```

Firebase에는 모델 클래스명이 아니라 표준 Action ID를 보내는 것을 기본으로 한다.

## 6. Stabilizer

raw prediction을 즉시 결과로 보내지 않는다.

개념 흐름:

```text
frame predictions
   |
minimum confidence
   |
sliding time/window
   |
same-class ratio
   |
hold duration
   |
neutral/reset policy
   |
cooldown
   |
confirmed action
```

정확한 기본값은 `docs/SETTINGS.md`를 따른다.

## 7. Firebase 데이터 계약

초기 권장 구조:

```text
/hall/current
  actionId
  confidence
  updatedAt
  source
  modelId
```

예:

```json
{
  "actionId": "hands_up",
  "confidence": 0.94,
  "updatedAt": 1788825600000,
  "source": "camera-a",
  "modelId": "tm-pose-v1"
}
```

원칙:
- current node는 overwrite한다.
- push 기반 history를 만들지 않는다.
- 이미지를 넣지 않는다.
- 학생 식별 필드를 만들지 않는다.

## 8. Stale 상태

태블릿 B는 마지막 값이 오래되었다면 이를 현재 동작처럼 보여주지 않는다.

예:
- 0~2초: 정상
- 2초 이상 갱신 없음: "AI 카메라 연결 확인" 상태
- 실제 값은 설정 가능

이 기준은 네트워크 장애 시 이전 학생의 결과가 계속 남는 문제를 막는다.

## 9. Firebase write 정책

모델 추론 FPS와 Firebase write FPS를 분리한다.

예:
- 카메라 렌더: 30fps
- AI 추론: 10~15fps
- Firebase: 확정 action 변경 시 즉시 + 필요한 경우 낮은 주기의 heartbeat

매 inference마다 write하지 않는다.

## 10. URL 전략

초기:
- `?mode=camera`
- `?mode=display`
- `?mode=admin`

이유:
- GitHub Repository Pages에서 path routing의 404 리스크를 줄임
- 한 빌드로 세 역할 제공
- 태블릿별 북마크/홈 화면 등록이 쉬움

## 11. 정적 자산/모델 경로

Vite에서는 GitHub Pages base를 고려한다.

권장:
- `base: '/teachable/'`
- 모델/manifest는 `public/models/`
- 런타임에서는 base URL helper를 사용

절대 루트 `/models/...`를 하드코딩하지 않는다.

## 12. 오류 상태

반드시 UI 상태로 구분한다.

- app booting
- camera permission required
- camera unavailable
- model loading
- model load failed
- ai running
- firebase connecting
- firebase disconnected
- no person / neutral
- evaluating
- confirmed action
- stale remote state

오류를 단순 console log로만 남기지 않는다.

## 13. 향후 PWA

초기 개발 완료 후 다음을 추가할 수 있다.

- app shell cache
- 모델 파일 cache
- 홈 화면 설치
- standalone display

Firebase 실시간 동기화는 인터넷 연결을 전제로 한다.
PWA는 페이지/모델 로딩 장애 완화용이며 완전한 오프라인 2-device 통신을 목표로 하지 않는다.
