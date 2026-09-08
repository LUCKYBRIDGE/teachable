# Architecture Decisions

이 문서는 이미 결정된 사항을 반복해서 재논의하지 않기 위한 기록이다.

## D-001. 2-Tablet 구조

**결정:** 한 태블릿에 촬영과 표시를 모두 몰지 않고 역할을 분리한다.

이유:
- 전신 인식에 적절한 촬영 거리와 결과 확인에 적절한 시청 거리가 다름
- 태블릿 A를 AI 인식 성능에 최적화할 수 있음
- 태블릿 B를 학생 경험에 최적화할 수 있음

## D-002. GitHub Pages 사용

**결정:** 웹앱은 GitHub Pages에 정적으로 배포한다.

별도 Node/Express 서버는 초기 범위에서 사용하지 않는다.

## D-003. Firebase Realtime Database 사용

**결정:** 태블릿 A -> B의 실시간 결과 전달에 Firebase Realtime Database를 사용한다.

Firebase의 역할:
- 현재 action state 중계
- 선택적으로 설정 동기화

Firebase의 역할이 아닌 것:
- 영상 저장
- 학생 데이터베이스
- 행동 history 저장

## D-004. 인터넷 연결 전제

**결정:** 학교 복도의 Wi-Fi/인터넷이 정상 동작한다고 전제한다.

완전 오프라인 P2P/WebRTC/로컬 AP는 초기 범위에서 제외한다.
PWA 캐싱은 향후 보조 기능으로만 검토한다.

## D-005. Teachable Machine Pose를 첫 모델로 사용

**결정:** 첫 구현은 Teachable Machine Pose export 모델을 사용한다.

단, 앱은 model adapter 구조로 만들어 다른 엔진으로 교체할 수 있어야 한다.

## D-006. 모델 파일 자체를 저장소에서 버전 관리

**결정:** Teachable Machine 공유 URL에만 의존하지 않고 export 파일을 `public/models/<model-id>/`에 저장한다.

장점:
- 모델과 코드 버전 동기화
- 롤백 가능
- 장기 운영 안정성
- 향후 PWA 캐시 가능

## D-007. Stabilizer는 모델 밖에서 구현

**결정:** hold time, threshold, cooldown 등의 시간 기반 정책을 모델에 포함시키지 않는다.

웹사이트 설정으로 조절한다.

## D-008. TTS 제외

**결정:** 동작명을 음성으로 읽지 않는다.

복도 소음과 반복 출력 문제를 피하고 시각적 피드백에 집중한다.

## D-009. 영상/사진 외부 전송 금지

**결정:** 카메라 데이터는 태블릿 브라우저 내부 AI 처리에만 사용한다.

태블릿 A와 B 사이에도 영상 스트림을 보내지 않는다.
태블릿 B의 자기 모습은 B 자체 카메라로 보여준다.

## D-010. 행동 이력 미저장

**결정:** Firebase는 current 값을 overwrite하고 학생별/시간별 행동 로그를 쌓지 않는다.

## D-011. 초기에는 단일 인물, 정적 Pose 중심

**결정:** 초기 모델은 한 사람의 비교적 정적인 자세 분류를 목표로 한다.

걷기, 뛰기, 박수, 손 흔들기처럼 시간축 자체가 핵심인 Action Recognition은 후속 범위이다.

## D-012. 초기 route는 query parameter

**결정:** `?mode=camera|display|admin`을 사용한다.

GitHub Pages의 직접 경로 새로고침 문제를 최소화한다.
