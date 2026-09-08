# 개인정보·보안 원칙

## 1. 기본 원칙

이 시스템은 학교 복도에서 카메라를 사용하므로 기능보다 먼저 데이터 최소화를 지킨다.

핵심 원칙:
**카메라 영상은 해당 태블릿의 브라우저 안에서만 처리하고 외부로 보내지 않는다.**

## 2. 수집하지 않는 데이터

초기 시스템은 다음을 수집/저장하지 않는다.

- 얼굴 이미지
- 사진
- 동영상
- 음성
- 학생 이름
- 학번
- 반
- 계정
- 생체정보
- 개인별 행동 기록
- 방문 시간 기록
- 개인별 통계

## 3. Firebase로 보낼 수 있는 데이터

현재 장면의 비식별 상태만 허용한다.

예:
```json
{
  "actionId": "arms_open",
  "confidence": 0.91,
  "updatedAt": 1788825600000,
  "source": "camera-a",
  "modelId": "tm-pose-v1"
}
```

이 값은 다음 상태로 overwrite한다.

## 4. History 금지

초기 설계에서는 다음 구조를 만들지 않는다.

```text
/history/<timestamp>
/students/<studentId>
/sessions/<sessionId>/events
```

제품 요구가 명시적으로 바뀌기 전까지 장기 로그 기능을 추가하지 않는다.

## 5. 영상 스트리밍 금지

태블릿 A의 영상을 태블릿 B로 보내지 않는다.

태블릿 B가 학생의 모습을 보여줄 필요가 있을 경우 B 자체의 전면 카메라를 사용한다.

따라서:
- WebRTC 영상 스트림 없음
- Firebase Storage 없음
- frame snapshot upload 없음

## 6. Firebase Web config

브라우저용 Firebase config는 클라이언트 애플리케이션에서 사용되는 설정이다.
보안을 config 문자열 은닉에 의존하지 않는다.

실제 보호는 다음에 둔다.

- Firebase Security Rules
- 허용 데이터 구조 validation
- 필요 이상으로 넓은 read/write path 금지
- 별도 프로젝트 사용
- 저장 데이터 최소화

서비스 계정 private key나 Admin SDK 자격증명은 GitHub 저장소에 절대 커밋하지 않는다.

## 7. Security Rules 요구

배포 전 rules를 반드시 별도 검토한다.

최소 요구:
- 허용된 path만 write
- actionId 타입/길이 검증
- confidence 0~1 검증
- timestamp 타입 검증
- 예상하지 않은 대용량 객체 차단
- history 생성 경로 없음

초기 개발 편의를 위한 `.read: true, .write: true`를 운영 rules로 남기지 않는다.

## 8. 화면 안내

실제 복도 배치 시 카메라 사용 목적을 화면 또는 주변 안내물에 명확히 표시한다.

권장 안내 취지:
- AI가 현재 자세를 분류함
- 영상/사진을 저장하지 않음
- 화면 속 모습은 실시간 체험을 위한 것임

정확한 학교 내부 안내 문구는 실제 운영 단계에서 확정한다.

## 9. 디버깅

개발 중에도 개인정보를 console이나 Firebase에 남기지 않는다.

허용되는 로그:
- model loaded
- camera started
- actionId
- confidence
- FPS
- connection state

금지:
- image base64
- frame blob
- 사용자 식별 정보

## 10. 실패 안전

Firebase 연결이 끊기면 Display는 마지막 학생의 자세를 무기한 현재 값으로 표시하지 않는다.
stale timeout을 넘기면 연결 확인/대기 상태로 전환한다.
