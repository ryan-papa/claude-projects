# PRD — 하네스 모델 선택 정책 (간소 PRD)

| 항목 | 값 |
|---|---|
| 작성일 | 2026-04-25 |
| 종류 | 하네스 메타 변경 (단축 경로) |
| 통합 브랜치 | main |
| 작업 브랜치 | feat/harness-model-policy |

## 1. 변경 이유

`/rp-*` 단계마다 모델 비용·속도가 차이 나는데 정책이 부재. 사용자 결정: **rp-specify 와 rp-prd 만 Opus, 그 외 모든 단계는 Sonnet**. 비용 절감 + 사용 한도 보호 + Opus 의 추론 우위는 PRD 초안 작성에 집중. 리뷰 단계는 Sonnet 으로도 충분 (객관 채점 위주, 새 설계 추론 부담 낮음). 개발(`/rp-dev`) 은 코드 패턴 따르기·테스트 보강 위주라 Sonnet 에 적합.

## 2. 영향 파일

| 파일 | 변경 |
|---|---|
| `CLAUDE.md` (루트) | "## 모델 선택 정책" 섹션 신설. 단계별 권장 모델 표 + 사용 가이드 한 줄 |
| `docs/harness-workflow.md` | 12 단계 표에 "모델" 컬럼 추가 (단계별 권장값) |
| 각 `docs/skills/rp-*.md` | frontmatter `model:` 필드는 추가하지 않음 — Skill 파일이 모델을 강제하면 사용자 자유도 낮아짐. 가이드만 |

권장 매핑:

| 단계 | 스킬 | 모델 | 이유 |
|:-:|---|---|---|
| 1 | rp-init | Sonnet | 정형 셋업 |
| 2 | rp-specify | **Opus** | 신규 도메인 추론 + 질문 설계 |
| 3 | rp-prd | **Opus** | 비목표·리스크·테스트 매트릭스 등 종합 사고 |
| 4 | rp-plan-review | Sonnet | 채점 (서브에이전트 룰은 별개) |
| 5 | rp-eng-review | Sonnet | 채점 |
| 6 | rp-task | Sonnet | 분해 |
| 7 | rp-dev | Sonnet | 코드 작성 |
| 8 | rp-qa | Sonnet | 검증 |
| 9 | rp-code-review | Sonnet | 채점 |
| 10 | (산출물 보고) | Sonnet | 텍스트 정리 |
| 11 | rp-ship | Sonnet | gh/git 자동화 |
| 12 | rp-retro | Sonnet | 회고 작성 |

사용자가 `/model` 슬래시로 즉시 전환. 이번 정책은 **권장**이며, 강제 전환 자동화는 별도 PR (예: hook 으로 단계 진입 시 모델 비교).

**Opus override 예외 조항** (메인 룰을 깨지 않는 명시 예외):
- 보안 민감 PR (CSO/security 라벨, OAuth/암호화/CSRF 변경) — 9·5 단계 Opus 권장
- 동시성·트랜잭션·DB 락·분산 시스템 패턴 — 5·9 단계 Opus 권장
- 아키텍처 신규 도입 (모듈 신설·인프라 전환) — 5 단계 Opus 권장

해당 PR 의 plan/eng/code 리뷰 진입 시 사용자 또는 메인 에이전트가 명시 `/model opus` 로 전환 후 진행. 전환 시 review 산출물 헤더에 모델 표시.

## 3. 롤백

| 상황 | 조치 |
|---|---|
| 정책 적용 후 Sonnet 으로 리뷰 품질 저하 발생 | `CLAUDE.md` 의 모델 표에서 해당 단계만 Opus 로 되돌리는 PR |
| 정책 자체 무효화 | 단일 commit revert. 영향 = `CLAUDE.md` + `harness-workflow.md` 두 문서뿐 |
| 사용자가 임의 모델 선호 | 본 정책은 권장이라 `/model` 로 자유 전환 가능 — 차단 메커니즘 없음 |

## 4. 검증

| # | 절차 |
|:-:|---|
| 1 | `CLAUDE.md` 에 새 섹션 추가 — 마크다운 lint (`docs/lint-docs` CI 통과) |
| 2 | `harness-workflow.md` 12 단계 표 컬럼 추가 후 가독성 확인 |
| 3 | 본 PRD 머지 후 다음 `/rp-amend` 진입 시 사용자가 단계별 모델 가이드를 인지하는지 — 회고 r1 에서 점검 |
| 4 | Codex 리뷰 (메타 단일 — `review-codex-meta.md`) 통과 |
| 5 | 정책 적용 후 첫 `/rp-dev` 단계가 Sonnet 으로 진행되었는지 다음 회고 r1 에서 측정 (수기 메모) |
| 6 | **효과 측정 (정량)** — 적용 후 머지된 PR 5건의 (a) 단계별 사용 모델 (b) 재작업 회차(r1→r2→r3) (c) Codex High/Critical 발견 수 — `/rp-retro` 의 평가 표에 별도 컬럼 추가. 4주 후 회고에서 종합 검토하여 정책 유지·조정 결정 |
| 7 | **SSOT 단일화** — 모델 매핑 표는 `harness-workflow.md` 가 SSOT, `CLAUDE.md` 는 요약 1줄 + 링크. 본 PRD 는 두 곳 모두 명시했으므로 후속 PR 에서 SSOT 정리 (Low) |
