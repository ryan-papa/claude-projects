# PRD (간소) — 하네스 백엔드 테스트 정책 신설

**날짜:** 2026-04-25
**트랙:** 하네스 메타 단축 경로 (Q4 #1)
**브랜치:** `feat/harness-backend-test-policy` → `main`

## 1. 변경 이유

| 사례 | 회귀 미검출 원인 |
|---|---|
| #51 V14 누락 의심 (4/24) | bootRun 검증 없어 운영에서 schema validation 가설 추정 |
| #59 JPA path `isActive` (4/24) | Specification mock 만으로 bind 경로 검증 불가 |
| #60 INSERT placeholder 4 누락 (4/25) | unit/standalone 으로 SQL bind 회귀 검증 불가 — 실 DB 통합 테스트 + bootRun 부재 |

**공통 원인**: 단위·standalone 테스트는 SQL bind, JPA path, Flyway 적용, DI 배선 같은 통합 회귀를 잡지 못함. 운영 배포 후 사용자 보고로 발견되는 패턴 반복.

**목표**: 백엔드 변경에 (a) 단위 항상 + (b) DB 건드리면 통합 트랜잭셔널 + (c) 컨트롤러/DI/Flyway 변경 시 bootRun 수기 검증 + (d) 읽기 endpoint API 응답 회귀 테스트 의무화.

## 2. 영향 파일

| 파일 | 변경 |
|---|---|
| `docs/harness-backend-test-policy.md` (신규) | 정책 본문 — 4단계 (단위 / GET API / DB 통합 / bootRun 수기) 적용 트리거·증거 저장 형식·예외 |
| `CLAUDE.md` (루트) | "Harness Engineering" 섹션에 신규 정책 링크 + 절대 규칙 1줄 추가 (백엔드 변경 시 본 정책 의무 적용) |
| `repositories/museum-finder/CLAUDE.md` | 기존 "백엔드 QA 게이트" 섹션을 신 정책 참조로 정정 (smoke-context + bootRun 항목 유지하되 정책 본문은 루트로 이관) |

## 3. 롤백 절차

| 시나리오 | 대응 |
|---|---|
| 신규 정책이 모든 PR 병목으로 작용 | (a) 정책 §3 "예외 조항" 명시: docs-only / 빌드 설정 / 의존 업그레이드는 통합 테스트 면제. (b) 미준수 시 회고에서 별도 PR 로 리팩터 (즉시 차단 X) |
| `@SpringBootTest` + 실 DB 가 CI 비용 폭증 | `@EnabledIfEnvironmentVariable` 로 CI 에서 skip, 로컬 머지 직전만 실행. 본 PR 의 `AdminMuseumCreateIntegrationTest` 패턴 그대로 재사용 |
| bootRun 수기 검증 노이즈 | 체크리스트 5초 내 완료(health UP + 변경 endpoint 1개 + Flyway 로그 + OAuth 응답 1건). 더 부담 시 정책 §4 단축 |

## 4. 검증

| 항목 | 기준 |
|---|---|
| 정책 문서 분량 | 200줄 이하 (CLAUDE.md `Document Style` 준수) |
| 루트 CLAUDE.md 링크 | "Harness Engineering" 표·"하네스 절대 규칙" 양쪽에 링크 |
| museum-finder 로컬 CLAUDE.md | 기존 게이트와 정책 본문 충돌 없는지 검증. smoke-context 1건은 이미 프로젝트 강제, 본 정책 흡수 |
| Claude meta 리뷰 | 서브에이전트 1회, FAIL 시 r2 재실행 |
| Codex 리뷰 | 가능 시 1회, 한도면 SKIP 사유 명시 |

## 5. 리뷰 계획

| 회차 | 주체 | 저장 |
|---|---|---|
| r1 | Claude 서브에이전트 (meta) | `prd/20260425_harness-backend-test-policy/review-claude-meta-r1.md` |
| — | Codex (시도) | `review-codex-meta.md` (성공/SKIP) |

## Open Issues

| ID | 내용 |
|---|---|
| O-1 | 다른 프로젝트(`repositories/[other]/`)에 같은 정책 적용 시기 — 본 PR 은 정책 본문만 루트에 두고 적용은 museum-finder 우선. 후속 프로젝트 합류는 그 시점 추가 |
| O-2 | 프런트엔드 동등 정책 — 이미 Playwright + axe 의무화 (CLAUDE.md). 본 PR 은 백엔드만 |
