# Teachable

학교 복도에서 태블릿 2대를 이용하여 학생의 현재 자세를 AI가 실시간으로 분류하고, 다른 태블릿에서 자신의 모습과 판정 결과를 확인하는 웹 기반 AI Pose 체험 프로젝트이다.

## 현재 상태

**Foundation / 설계 기준 확정 단계**

현재 저장소에는 구현에 앞서 필요한 아키텍처, 개발 지침, 모델 관리 규칙, 설정 정책, 개인정보 원칙, 테스트 계획을 먼저 고정하고 있다.

## 핵심 사용자 경험

```text
Tablet A : AI Camera
카메라
  -> Teachable Machine Pose
  -> 프레임별 prediction
  -> 판정 안정화
  -> 표준 Action ID
  -> Firebase Realtime Database
                         |
                         v
Tablet B : AI Mirror
자체 셀카 거울 화면
  + 현재 Action 수신
  -> 큰 동작명
  -> 선택적 신뢰도
  -> 시각적 피드백
```

### 중요한 원칙

- 영상과 사진은 태블릿 밖으로 보내지 않는다.
- Firebase에는 현재 동작명과 신뢰도 같은 비식별 상태만 보낸다.
- 학생 이름, 학번, 행동 이력을 저장하지 않는다.
- TTS는 사용하지 않는다.
- Teachable Machine 모델과 동작 유지시간/신뢰도/쿨다운 같은 판정 정책을 분리한다.
- Display 태블릿은 특정 AI 모델 구현에 종속되지 않는다.
- 초기에는 한 사람의 정적 Pose 분류에 집중한다.

## 확정 기술 방향

- Hosting: GitHub Pages
- Build: Vite + TypeScript
- AI v1: Teachable Machine Pose
- Model files: repository-local versioned exports
- Realtime bridge: Firebase Realtime Database
- Camera: browser MediaDevices API
- Tablet A: AI inference
- Tablet B: selfie mirror + result display
- Admin: model/runtime settings
- Internet/Wi-Fi: 학교 복도에서 정상 연결을 전제로 함

## 문서

개발 전에 관련 문서를 확인한다.

| 문서 | 역할 |
|---|---|
| [AGENTS.md](./AGENTS.md) | 사람/AI 개발자가 따라야 할 최상위 작업 지침 |
| [PROJECT_PLAN.md](./docs/PROJECT_PLAN.md) | 완성까지 단계별 개발 로드맵 |
| [ARCHITECTURE.md](./docs/ARCHITECTURE.md) | 구성 요소, 데이터 흐름, 인터페이스 |
| [DECISIONS.md](./docs/DECISIONS.md) | 이미 확정된 설계 결정 |
| [MODEL_GUIDE.md](./docs/MODEL_GUIDE.md) | Teachable Machine 모델 업로드·버전 관리 |
| [SETTINGS.md](./docs/SETTINGS.md) | 웹에서 조절할 판정/카메라/UI 설정 |
| [PRIVACY_SECURITY.md](./docs/PRIVACY_SECURITY.md) | 카메라·Firebase 개인정보 및 보안 원칙 |
| [TEST_PLAN.md](./docs/TEST_PLAN.md) | 태블릿/복도 실환경 검증 기준 |

## 목표 폴더 구조

구현이 진행되면 다음 구조를 기준으로 확장한다.

```text
teachable/
├─ AGENTS.md
├─ README.md
├─ docs/
│  ├─ PROJECT_PLAN.md
│  ├─ ARCHITECTURE.md
│  ├─ DECISIONS.md
│  ├─ MODEL_GUIDE.md
│  ├─ SETTINGS.md
│  ├─ PRIVACY_SECURITY.md
│  └─ TEST_PLAN.md
├─ public/
│  ├─ models/
│  │  ├─ registry.json
│  │  └─ tm-pose-v1/
│  │     ├─ model.json
│  │     ├─ metadata.json
│  │     ├─ weights.bin
│  │     └─ MODEL.md
│  └─ assets/
├─ src/
│  ├─ app/
│  ├─ camera/
│  ├─ ai/
│  │  ├─ adapters/
│  │  ├─ stabilizer/
│  │  └─ actions/
│  ├─ firebase/
│  ├─ display/
│  ├─ admin/
│  ├─ settings/
│  └─ styles/
├─ index.html
├─ package.json
├─ tsconfig.json
└─ vite.config.ts
```

폴더는 필요한 구현 단계에서 생성하며, 빈 구조를 위해 불필요한 placeholder를 남기지 않는다.

## URL 모드

초기 버전은 GitHub Pages 호환성을 위해 query parameter 기반 모드를 사용한다.

- `/?mode=camera` : AI Camera
- `/?mode=display` : AI Mirror
- `/?mode=admin` : 관리자 설정
- 파라미터 없음 : 역할 선택 화면

## 모델 업로드

Teachable Machine Pose에서 TensorFlow.js 모델을 다운로드한 뒤 다음처럼 새 버전 폴더로 추가한다.

```text
public/models/tm-pose-v1/
├─ model.json
├─ metadata.json
├─ weights.bin
└─ MODEL.md
```

모델을 업로드하기 전에 [MODEL_GUIDE.md](./docs/MODEL_GUIDE.md)를 확인한다.

모델의 클래스 예시는 다음과 같다.

- `neutral`
- `one_hand_up`
- `hands_up`
- `arms_open`
- `sitting`
- `bowing`

실제 클래스 구성은 첫 모델 학습 및 현장 검증 후 확정한다.

## 개발 단계

1. Foundation
2. Local AI Prototype
3. Prediction Stabilization
4. Firebase Bridge
5. AI Mirror UI
6. Admin & Model Management
7. Hallway Hardening

자세한 완료 조건은 [PROJECT_PLAN.md](./docs/PROJECT_PLAN.md)에 정의되어 있다.

## 초기 실기기 방향

- AI Camera: Galaxy Tab S9 FE 우선
- AI Mirror: Galaxy Tab S6 Lite 또는 보유 구형 태블릿
- 실제 기기 세대/브라우저 상태는 구현 후 테스트 문서에 기록

## 아직 하지 않는 것

- TTS
- 얼굴 인식
- 학생 식별
- 영상 스트리밍
- 학생별 행동 기록
- 복잡한 사용자 계정
- 별도 백엔드 서버
- 완전 오프라인 P2P 통신
- 동적 행동 인식의 조기 도입

기본 Pose 체험이 실제 복도에서 안정적으로 작동한 뒤 필요성을 검토한다.
