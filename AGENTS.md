# AGENTS.md

이 문서는 사람과 AI 개발 도구가 `teachable` 저장소에서 일할 때 따라야 하는 최상위 개발 지침이다.

## 1. 작업 시작 전 필수 확인

모든 구현 작업은 다음 순서로 시작한다.

1. `README.md`를 읽는다.
2. `docs/PROJECT_PLAN.md`를 읽는다.
3. `docs/ARCHITECTURE.md`를 읽는다.
4. 작업이 AI 모델과 관련되면 `docs/MODEL_GUIDE.md`를 읽는다.
5. 설정을 건드리면 `docs/SETTINGS.md`를 읽는다.
6. 데이터·Firebase·카메라 처리와 관련되면 `docs/PRIVACY_SECURITY.md`를 읽는다.
7. 구현 후 `docs/TEST_PLAN.md`의 관련 항목을 검증한다.

문서와 코드가 충돌하면 문서의 확정 결정을 우선하되, 실제 구현상 문제가 발견되면 문서를 함께 수정한다.

## 2. 확정 아키텍처

이 프로젝트의 기본 구조는 다음과 같다.

- 정적 배포: GitHub Pages
- 프런트엔드: Vite + TypeScript
- 태블릿 A: 카메라 + Pose 모델 + 동작 분류 + 판정 안정화 + 결과 전송
- 태블릿 B: 셀카 거울 화면 + 판정 결과 수신 + 시각적 피드백
- 실시간 전달: Firebase Realtime Database
- 초기 AI: Teachable Machine Pose export 모델
- 모델 파일: 저장소의 `public/models/<model-id>/`에 버전별 보관
- TTS: 사용하지 않는다.
- 영상·사진 전송/저장: 하지 않는다.
- 학생 이름·식별정보·행동 이력 저장: 하지 않는다.

## 3. 핵심 설계 원칙

### 3.1 모델과 판정 정책을 분리한다

AI 모델은 각 프레임의 클래스 확률을 반환하는 역할만 한다.

다음 항목은 모델 파일에 넣지 않고 웹앱의 판정 안정화 계층에서 처리한다.

- 최소 신뢰도
- 동작 유지시간
- 최근 프레임 투표/평균
- 동일 판정 비율
- Neutral 복귀시간
- 쿨다운
- 동작별 활성화 여부
- 동작별 임계값 override

모델을 다시 학습하지 않고 현장에서 조정할 수 있어야 한다.

### 3.2 표시 태블릿은 모델에 종속되지 않는다

태블릿 B는 Teachable Machine의 클래스명을 직접 해석하지 않는다.
태블릿 A에서 표준 Action ID로 변환한 뒤 Firebase로 보낸다.

예:
`HandsUp -> hands_up -> "손 들기"`

이 원칙을 지켜 향후 Teachable Machine을 다른 Pose/Action 모델로 교체해도 태블릿 B와 Firebase 계약을 유지한다.

### 3.3 영상은 장치 밖으로 보내지 않는다

카메라 프레임은 브라우저 내부 AI 추론에만 사용한다.
Firebase에는 확정된 동작 상태만 전송한다.

허용되는 기본 payload 예:
`{ actionId, confidence, updatedAt, source }`

금지:
- 이미지
- 영상
- 얼굴 crop
- 학생 이름
- 학생 번호
- 개인 식별자
- 지속적인 행동 로그

### 3.4 GitHub Pages 경로를 안전하게 처리한다

Repository Pages의 base path가 `/teachable/`임을 고려한다.
절대경로 `/models/...`를 무분별하게 사용하지 않는다.
Vite의 `base` 설정 또는 `import.meta.env.BASE_URL`을 사용한다.

### 3.5 태블릿 성능을 우선한다

- 불필요한 프레임마다 Firebase write를 하지 않는다.
- AI 추론 FPS와 화면 렌더 FPS를 분리할 수 있게 한다.
- TensorFlow/모델 인스턴스를 반복 생성하지 않는다.
- 카메라 stream을 중복 생성하지 않는다.
- 화면이 숨겨졌을 때 불필요한 연산을 줄인다.
- 초기 목표는 한 사람의 안정적 포즈 인식이다.

## 4. URL/모드 규칙

GitHub Pages의 새로고침 404 문제를 피하기 위해 초기 버전은 SPA path route보다 query parameter를 우선한다.

- `/?mode=camera` : 태블릿 A
- `/?mode=display` : 태블릿 B
- `/?mode=admin` : 관리자 설정
- 파라미터가 없으면 역할 선택 화면

필요하면 이후 라우터를 도입하되 Pages fallback을 함께 설계한다.

## 5. 브랜치·변경 원칙

- 기능 개발은 작업 브랜치에서 한다.
- 안정화된 변경은 PR로 main에 합친다.
- 한 PR에 서로 무관한 대규모 변경을 섞지 않는다.
- 모델 바이너리 변경과 애플리케이션 코드 변경을 가능한 한 구분한다.
- 생성 파일(`dist/`)은 저장소에 직접 커밋하지 않는 것을 기본으로 한다.
- Firebase 비밀키나 서비스 계정 키를 저장소에 커밋하지 않는다.

## 6. 구현 금지/보류 항목

명시적 요구가 생기기 전까지 다음 기능을 추가하지 않는다.

- TTS/음성 출력
- 얼굴 인식
- 사람 식별
- 사용자 로그인
- 학생별 기록 저장
- 행동 통계/분석용 장기 로그
- 별도 Node/Express 백엔드
- SQL 데이터베이스
- 영상 스트리밍 A -> B
- 다중 인물 추적을 위한 과도한 복잡도

## 7. Definition of Done

기능 완료는 코드가 동작하는 것만을 의미하지 않는다.

최소한 다음을 충족해야 한다.

1. TypeScript 빌드 성공
2. GitHub Pages base path에서 정적 자산/모델 로딩 성공
3. Android Chrome 태블릿에서 카메라 권한 및 방향 정상
4. 태블릿 A의 모델 추론과 안정화 정상
5. Firebase가 끊기거나 늦을 때 UI가 오작동하지 않음
6. 태블릿 B에서 stale 상태를 현재 상태처럼 표시하지 않음
7. 영상/사진이 네트워크로 전송되지 않음
8. 관련 문서와 설정 스키마가 구현과 일치
