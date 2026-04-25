# 하네스 백엔드 테스트 정책

> 백엔드 코드 변경 시 회귀 검출 게이트 4단. 미준수 = 머지 차단.
> 적용 대상: Spring Boot/JDBC/JPA 백엔드 보유 프로젝트 (현재 `museum-finder`).

## 1. 4단계 게이트

| # | 게이트 | 트리거 (해당하는 변경 포함 시 의무) | 도구 |
|:-:|---|---|---|
| 1 | **단위 / standalone 테스트** | **항상** (모든 backend PR) | JUnit5 + Mockito · 컨트롤러 standalone (`new C(svc)`) |
| 2 | **GET API 응답 테스트** | 읽기 endpoint 추가/변경/응답 스키마 수정 | standalone 컨트롤러 + 직렬화 검증 (Jackson `valueToTree`) |
| 3 | **DB 통합 트랜잭셔널 테스트** | SQL/JPA Specification/Repository 메서드/마이그레이션 추가/JdbcTemplate 호출 | `@SpringBootTest(NONE)` + `@Transactional` + `@EnabledIfEnvironmentVariable("POSTGRES_URL")` 자동 ROLLBACK |
| 4 | **로컬 bootRun + OAuth 수기 검증** | 컨트롤러·DI·Flyway·기동 시 인덱스/트리거 변경 | `./gradlew bootRun` + 헬스 + 변경 endpoint 인증 응답 캡처 |

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

### 2.3 DB 통합 트랜잭셔널

| 항목 | 기준 |
|---|---|
| 적용 | INSERT/UPDATE/DELETE SQL 추가·수정, JPA Specification path, JdbcTemplate `update()` 인자 매칭, Flyway 마이그레이션, 도메인 entity column 추가 |
| 어노테이션 | `@SpringBootTest(webEnvironment = NONE)` + `@Transactional` (자동 ROLLBACK) + `@EnabledIfEnvironmentVariable(named = "POSTGRES_URL", matches = ".+")` |
| Spring 설정 | `properties = {"spring.autoconfigure.exclude=", "spring.datasource.url=${POSTGRES_URL}", ...}` 으로 테스트 전용 application.yml 의 datasource 제외 우회 |
| 환경 전파 | `build.gradle.kts` 에 `tasks.withType<Test> { environment(...) }` 로 prod DB env 패스스루 (debug only — CI 에서는 `POSTGRES_URL` 미설정으로 자동 skip) |
| 비용 | SpringContext 1회 부팅 ~3초. 1 PR 당 1~2건 추가 권장 |
| **fail-closed 증거 의무** | CI skip 가능성을 보완하기 위해 **로컬에서 `POSTGRES_URL=...` 셋 한 후 `./gradlew test --tests "*IntegrationTest*"` 출력 (테스트 통과 라인 + 시각) 을 `qa-smoke.md` 에 캡처**. 미캡처 시 `rp-ship` 차단. 수기 누락 방지. |
| 회귀 가드 | "테스트 통과 → 의도 깨뜨림 → 빨간불 → 복원 → 다시 통과" 4-stroke 검증. 4 단계 모두 출력(또는 commit hash 차이) 을 `qa-smoke.md` 또는 PR description 한 곳에 1줄씩 4줄 이상 기록. 단순 "4-stroke 통과" 한 줄 금지 |

### 2.4 로컬 bootRun + OAuth 수기

| 항목 | 기준 |
|---|---|
| 적용 | 컨트롤러 추가/시그니처 변경, DI 배선 추가, Flyway V*.sql 추가, application.yml 변경 |
| 절차 | (a) `./gradlew bootRun` (b) `curl /actuator/health` 200 UP (c) Flyway 로그 `Successfully validated N migrations` + `Schema up to date` 또는 `Successfully applied` (d) 변경 endpoint OAuth 인증 cookie 통한 실 응답 1건 캡처 — PR description 또는 `qa-smoke.md` 첨부 |
| 5초 내 미통과 | 즉시 디버그. 머지 전 통과 필수 |

## 3. 예외 조항

| 변경 종류 | 게이트 면제 | 단축 한계 |
|---|---|---|
| docs-only (`*.md` 만) | 1·2·3·4 모두 면제 | — |
| 빌드 설정 (gradle plugin 버전 등) | 3·4 면제. 1·2 유지 | — |
| 의존성 업그레이드 — non-Spring | 3 면제. 1·2·4 유지 | — |
| 의존성 업그레이드 — **Spring Boot 마이너 포함** | 3 면제. 1·2·4 **모두 의무** | DI/AutoConfig 회귀 위험 |
| **test-only** (테스트 코드만 추가/리팩터, prod 무변경) | 3·4 면제. 1·2 유지 | — |
| 핫픽스 — `hotfix/*` 브랜치 + 회귀 가드 동시 추가 (4-stroke 검증 동반) | 4 단축 가능: **OAuth 응답 캡처만 면제**. health UP + Flyway 로그는 유지 | 4-stroke 증거는 `qa-smoke.md` 또는 PR description 둘 중 한 곳에 고정 저장 (둘 다 누락 금지) |

## 4. PRD/회고 증거

| 단계 | 저장 위치 |
|---|---|
| PRD §4 검증 표 | 4 게이트 적용 여부 표시 (해당/면제) |
| `qa-smoke.md` | bootRun 로그·OAuth 응답·통합 테스트 출력 캡처 |
| 회귀 가드 추가 시 | 4-stroke 검증 결과 (의도 깨뜨려 빨간불 확인) 1줄 |

## 5. 위반 처리

- PR 리뷰어가 본 정책 게이트 누락 발견 시 **즉시 차단 + 추가 PR 로 보강** (소급 금지 회고 r1 #4 와 동일 원칙)
- 회고에서 누락 사례 발견 시 다음 PR 의 의무 추가
- 정책 자체 개정은 본 파일 + 루트 CLAUDE.md 양쪽 동기 갱신 (하네스 메타 변경 단축 경로)

## 6. 도구·예시

- 통합 테스트 패턴: `repositories/museum-finder/backend/src/test/java/com/museumfinder/api/admin/AdminMuseumCreateIntegrationTest.java`
- 회귀 가드 4-stroke 사례: `museum-finder` PR #59 (JPA path) · #60 (INSERT placeholder) 회고
