# 프로젝트 완성 계획

## 1. 프로젝트 목표

`teachable`은 학교 복도에 태블릿 2대를 설치하여 학생이 자신의 자세를 직접 확인하면서 AI의 실시간 분류 결과를 체험하는 웹앱이다.

핵심 경험은 다음과 같다.

1. 학생이 지정 위치에서 자세를 취한다.
2. 태블릿 A가 학생의 전신을 카메라로 촬영한다.
3. 브라우저에서 Teachable Machine Pose 모델을 실행한다.
4. 웹앱이 모델의 프레임별 prediction을 안정화한다.
5. 확정된 동작명과 신뢰도만 Firebase Realtime Database로 보낸다.
6. 태블릿 B는 자체 셀카 화면 위/아래에 현재 동작을 크고 명확하게 표시한다.
7. 학생은 AI가 자신의 자세를 어떻게 분류하는지 즉시 확인한다.

## 2. 확정 범위

### 포함

- GitHub Pages 정적 배포
- Android 태블릿 Chrome 지원
- 태블릿 A / 태블릿 B 역할 분리
- Teachable Machine Pose 모델 로딩
- 저장소 내부 모델 버전 관리
- 카메라 전면/후면 선택
- Pose 추론 및 클래스 확률 표시
- 판정 안정화
- Firebase Realtime Database 실시간 상태 전달
- 태블릿 B 셀카 거울 화면
- 동작명 및 선택적 신뢰도 표시
- 관리자 설정 화면
- 모델 선택
- 판정 파라미터 조정
- 시각적 성공/변경 피드백
- 추후 PWA 캐시를 붙일 수 있는 구조

### 제외

- TTS
- 얼굴 인식
- 사용자 식별
- 학생별 기록 저장
- 영상/사진 업로드
- 장기 행동 이력
- 별도 백엔드 서버
- 복잡한 계정 시스템
- 초기 버전의 다중 인물 행동 추적

## 3. 목표 사용자 흐름

### 태블릿 A: AI Camera

1. 사이트 접속
2. Camera 모드 선택 또는 고정 URL 접속
3. 모델 로드
4. 카메라 권한 허용
5. 카메라 프리뷰와 연결 상태 확인
6. Pose 추론 시작
7. 안정화된 동작을 Firebase에 갱신

운영자에게 필요한 정보:
- 카메라 연결 여부
- 모델 로딩 여부
- Firebase 연결 여부
- 현재 raw prediction
- 현재 확정 action
- FPS/추론 상태

### 태블릿 B: AI Mirror

1. 사이트 접속
2. Display 모드 선택 또는 고정 URL 접속
3. 자체 전면 카메라 권한 허용
4. 셀카 영상을 거울 형태로 크게 표시
5. Firebase의 현재 동작 구독
6. 동작명과 신뢰도를 시각적으로 표시
7. stale/연결 끊김 상태를 명확하게 구분

### 관리자

1. Admin 모드 접속
2. 현재 모델 선택
3. 신뢰도/유지시간/쿨다운 등 조절
4. 동작별 활성화 여부 조절
5. 화면 표시 옵션 조절
6. 설정 저장
7. 향후 필요 시 Firebase를 통한 원격 설정 동기화

## 4. 단계별 개발 로드맵

### Phase 0. Foundation

목표: 개발 기준 고정

- README
- AGENTS
- 프로젝트 계획
- 아키텍처
- 모델 관리 규칙
- 설정 정책
- 개인정보/보안 원칙
- 테스트 기준

완료 조건:
- 이후 개발자가 설계를 처음부터 다시 논의하지 않고 구현에 들어갈 수 있음

### Phase 1. Local AI Prototype

목표: 한 태블릿에서 모델을 안정적으로 구동

- Vite + TypeScript 초기화
- Camera API
- Teachable Machine Pose 라이브러리 통합
- 샘플/실제 모델 로더
- raw prediction 표시
- 모델 로드 실패 처리
- 모바일 화면 대응

완료 조건:
- Galaxy Tab S9 FE에서 카메라 영상과 prediction이 지속적으로 동작

### Phase 2. Prediction Stabilization

목표: 프레임별 흔들림을 실제 동작 판정으로 변환

- 최소 신뢰도
- hold time
- sliding window
- 동일 클래스 비율
- Neutral 복귀
- cooldown
- action mapping
- 동작별 override

완료 조건:
- 짧은 오분류로 결과가 반복해서 뒤집히지 않음
- 설정 변경만으로 반응속도와 안정성을 조절 가능

### Phase 3. Firebase Bridge

목표: A -> B 실시간 상태 전달

- Firebase modular SDK
- 현재 상태 1개만 overwrite
- action schema validation
- write throttle
- stale timestamp
- 연결 상태 표시
- 오류/재연결 처리

완료 조건:
- 두 태블릿이 동일 Wi-Fi/인터넷 환경에서 안정적으로 동기화
- Firebase에 영상, 사진, 개인 식별 데이터가 없음

### Phase 4. AI Mirror UI

목표: 학생이 멀리서도 즉시 이해할 수 있는 디스플레이

- 태블릿 B 셀카 거울
- 동작명 대형 표시
- 선택적 confidence
- 상태 변경 애니메이션
- "판단 중", "사람을 찾는 중", "연결 끊김" 상태
- 일반 모드 / AI 학습 모드

완료 조건:
- 복도 거리에서 동작명이 명확히 읽힘
- 학생이 카메라에 잡힌 자신의 모습을 확인할 수 있음

### Phase 5. Admin & Model Management

목표: 재배포 없이 현장 튜닝 가능

- 모델 registry
- 모델 선택
- global threshold
- per-action threshold
- hold time
- neutral reset
- cooldown
- AI FPS
- confidence 표시
- skeleton 표시
- 동작 활성화/비활성화

완료 조건:
- 모델 재학습 없이 대부분의 현장 반응 문제를 설정으로 해결 가능

### Phase 6. Hallway Hardening

목표: 실제 복도 상설 사용

- S9 FE / S6 Lite / 보유 태블릿 실기기 테스트
- 전신이 잡히는 거리 확정
- 바닥 standing zone 안내
- 역광/형광등/혼잡 조건 테스트
- 장시간 발열/메모리 테스트
- 화면 꺼짐/복귀 대응
- Firebase 일시 장애 처리
- GitHub Pages 배포 자동화
- 필요 시 PWA 설치/캐싱

완료 조건:
- 수업 시간 사이의 실제 복도 환경에서 반복 사용해도 운영자가 수동으로 계속 복구하지 않아도 됨

## 5. 초기 성공 기준

첫 공개 가능한 버전은 다음 조건을 만족해야 한다.

- 최소 5개 정도의 정적 자세를 분류할 수 있음
- Neutral 클래스가 있음
- 태블릿 A와 B가 1초 이내 체감 수준으로 상태 동기화
- 결과 변경이 과도하게 깜빡이지 않음
- 학생 영상이 외부로 전송되지 않음
- 동작명은 멀리서 읽을 수 있는 크기로 표시
- 모델/임계값/유지시간을 코드 재작성 없이 바꿀 수 있음

## 6. 초기 권장 동작 집합

모델 확정 전 후보이며 실제 학습 결과에 따라 조정한다.

- neutral
- one_hand_up
- hands_up
- arms_open
- sitting
- bowing

동작명은 모델 클래스명과 사용자 표시명을 분리한다.

## 7. 향후 확장 후보

기본 시스템 안정화 뒤 검토한다.

- 랜덤 포즈 도전 모드
- AI 학습 모드에서 전체 클래스 확률 표시
- 관절점/스켈레톤 설명
- 여러 모델 프리셋
- PWA 홈 화면 설치
- MediaPipe 기반 커스텀 분류기로 모델 adapter 교체
- 동적인 Action Recognition 연구

이 항목들은 초기 완성 범위보다 우선하지 않는다.
