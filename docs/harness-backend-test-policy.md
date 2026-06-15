# 하네스 백엔드 테스트 정책

> 백엔드 코드 변경 시 회귀 검출 게이트 2단. 미준수 = 머지 차단.
> 적용 대상: Spring Boot/JDBC/JPA 백엔드 보유 프로젝트 (현재 `museum-finder`).
> **실인프라(DB·Redis 등) 의존 테스트는 강제하지 않는다** — 모든 게이트는 mock/standalone 으로 인프라 없이 실행.

## 1. 2단계 게이트

| # | 게이트 | 트리거 (해당하는 변경 포함 시 의무) | 도구 |
|:-:|---|---|---|
| 1 | **단위 / standalone 테스트** | **항상** (모든 backend PR) | JUnit5 + Mockito · 컨트롤러 standalone (`new C(svc)`) |
| 2 | **GET API 응답 테스트** | 읽기 endpoint 추가/변경/응답 스키마 수정 | standalone 컨트롤러 + 직렬화 검증 (Jackson `valueToTree`) |

## 2. 게이트별 상세

### 2.1 단위 / standalone (항상)

| 항목 | 기준 |
|---|---|
| 서비스 단위 | Mockito + 순수 POJO. JdbcTemplate `verify()` 로 SQL 인자 매칭 가능 |
| 컨트롤러 standalone | `new Controller(mockedService)` 직접 호출. MockMvc 미사용 (프로젝트 정책) |
| 통과 기준 | `./gradlew test` 전체 PASS, 커버리지 신규 분기 모두 |

### 2.2 GET API 응답 테스트

| 항목 | 기준 |
|---|---|
| 적용 | `@GetMapping` 응답 형식 변경, DTO 필드 추가/제거, 직렬화 어노테이션 변경 |
| 검증 | 컨트롤러 standalone 테스트가 mock service 응답 → 실제 JSON 직렬화 확인 (Jackson `ObjectMapper.valueToTree(dto)`). 누락 필드, snake/camel, boolean `is*` prefix 같은 함정 차단 |
| 예시 | 본 정책 신설 사례: `boolean isActive` Java record 컴포넌트가 JSON `isActive`/`active` 어디로 직렬화되는지 contract test |

## 3. 예외 조항

| 변경 종류 | 게이트 면제 | 비고 |
|---|---|---|
| docs-only (`*.md` 만) | 1·2 모두 면제 | — |
| 빌드 설정 (gradle plugin 버전 등) | 2 면제. 1 유지 | — |
| 의존성 업그레이드 | 2 면제. 1 유지 | — |
| test-only (테스트 코드만 추가/리팩터, prod 무변경) | 2 면제. 1 유지 | — |

## 4. PRD/회고 증거

| 단계 | 저장 위치 |
|---|---|
| PRD §4 검증 표 | 2 게이트 적용 여부 표시 (해당/면제) |
| `qa-smoke.md` | 단위·GET API 테스트 출력 캡처 |

## 5. 위반 처리

- PR 리뷰어가 본 정책 게이트 누락 발견 시 **즉시 차단 + 추가 PR 로 보강** (소급 금지 회고 r1 #4 와 동일 원칙)
- 회고에서 누락 사례 발견 시 다음 PR 의 의무 추가
- 정책 자체 개정은 본 파일 + 루트 CLAUDE.md 양쪽 동기 갱신 (하네스 메타 변경 단축 경로)
