# PRD — 머지 직전 PRD 자동 정리

| 메타 | 값 |
|------|----|
| 유형 | 하네스 메타 변경 (간소 PRD) |
| 상태 | Draft |
| 통합 브랜치 | `main` (메타 분기 → `--base main` 자동) |
| 작업 브랜치 | `feat/prd-cleanup-on-merge` |

## 변경 이유

PRD·리뷰 증거 파일이 `repositories/[project]/docs/prd/[feature]/` 및 `docs/prd/[feature]/`에 누적되어 트리가 비대해진다. 머지 = 기능 출하 시점이므로 PRD는 의사결정 기록으로서 역할이 종료된다. 이 시점에 같은 PR diff 안에서 PRD 디렉토리를 통째 삭제하고, 핵심 요약(개요·기능 요구사항·Review 결과)을 PR 본문에 임베드하여 추후 참조성은 PR + git history로 보존한다.

**해결 문제:**
- `docs/prd/` 트리 누적 → 탐색·리뷰 노이즈
- 사후 PRD 단일 출처 모호 (개발 중엔 파일, 머지 후엔 git history)
- 회고·문서 검색 시 outdated PRD 혼선

**대안 비교:**

| 방안 | 평가 |
|------|------|
| 즉시 삭제 + PR 본문 요약 임베드 (채택) | 트리 항상 깨끗, PR 본문이 사후 조회 anchor |
| `_archive/`로 이동 | 트리는 정리되나 archive 누적 → 동일 문제 재발 |
| 머지 후 별도 PR로 정리 | 자동화 어렵고 누락 위험, PR diff 분리되어 가독성 저하 |
| 상태만 `Shipped` 마킹 | 누적 미해결 |

## 영향 파일

| 파일 | 변경 내용 |
|------|----------|
| [`docs/harness-absolute-rules.md`](../../harness-absolute-rules.md) | "배포·머지·브랜치" 섹션에 "머지 직전 PRD 정리 필수" 절대 규칙 1줄 추가 |
| [`docs/skills/rp-ship.md`](../../skills/rp-ship.md) | 자동 머지 가드 통과 후 머지 직전 단계 8(요약 임베드)·9(`git rm -r` 정리 커밋)·10(CI 재대기) 신설, 머지·배포는 11·12로 재번호 |
| [`docs/harness-ship.md`](../../harness-ship.md) | "머지 직전 PRD 정리" 정책 섹션 신설 (단계별 표 + SSOT 링크) |
| `.codex/skills/rp-ship/SKILL.md` | `scripts/sync-codex-skills.py`로 자동 동기화 (수동 편집 아님) |
| `docs/prd/prd-cleanup-on-merge/` | 본 PRD + 리뷰 증거 (머지 직전 본 규칙에 의해 자체 삭제됨 — dogfooding) |

**적용 범위:** Full PRD (`repositories/[project]/docs/prd/[feature]/`) + 간소 PRD (`docs/prd/[feature]/`) 둘 다.

**핵심 동작 (`rp-ship` 절차):**
1. 가드 4종 AND 통과 (CI · 리뷰 증거 · base 정상 · MERGEABLE)
2. PRD 유형별 요약 추출 → PR 본문 `<details>` 블록 임베드 (`gh pr edit --body`)
   - Full PRD: `## 개요·목적` + `## 기능 요구사항` + `## Review 결과`
   - 간소 PRD (메타): `## 변경 이유` + `## 영향 파일` + `## 검증` + `## Review 결과`
   - 추출 누락 / `gh pr edit` 1회 재시도 실패 시 ship 중단 + OPEN 유지
3. `git rm -r <project-root>/docs/prd/[feature]/`
4. 정리 커밋 `chore(prd): merge 직전 PRD 정리` + push
5. CI 재통과 확인 (가드 b 재실행 면제)
6. `gh pr merge --merge --delete-branch`

## 롤백 전략

| 조건 | 조치 |
|------|------|
| 정책 자체 철회 | 본 PR을 `git revert` → 절대 규칙·rp-ship·harness-ship 원복 |
| 정리 커밋만 실패 (CI 미통과 등) | OPEN 유지 + 사용자 보고 → 수동 조치 (자동 재시도 금지) |
| 머지 후 PRD 참조 필요 | PR 본문 `<details>` 요약 + `git log --all -- docs/prd/[feature]/` 로 git history 조회 |
| 본 dogfood PR 자체 정리 단계 실패 | OPEN 유지, PR 본문에 "self-host 첫 적용 — 문제 시 본 규칙 revert 검토" 명시 |

## 검증

| 항목 | 방법 |
|------|------|
| Claude 메타 리뷰 1회 | `/rp-plan-review` 서브에이전트 (5항목 메타 적용, 평균 ≥ 8.0 + 항목별 ≥ 7) → `review-claude-meta-r1.md` |
| Codex 메타 리뷰 1회 (병렬) | `codex review` → `review-codex-meta.md`. High/Critical 반영 |
| 자동 머지 가드 4종 | 정책 자체 변경이므로 가드 (b) 리뷰 증거 게이트는 본 PRD 디렉토리 메타 파일 2종 존재로 통과 |
| Dogfood 검증 | 본 PR 머지 시 신설 절차로 본 PRD 디렉토리 자체 삭제 (self-host) |
| CI lint-docs | docs 변경 통과 확인 |

## Open Issues

| 항목 | 상태 |
|------|------|
| 회고 시점 PRD 참조 경로 (PR 본문 vs git show) | 본 PRD에서 PR 본문 + git history로 명시 결정 |
| `<project-root>` 결정 (일반 기능 vs 메타) | rp-ship 사전 체크 게이트 로직 그대로 재사용 — 추가 설계 불필요 |
| 다중 PRD 디렉토리 동시 정리 (1 PR 다 feature) | 본 사이클 대상 외 — 등장 시 별도 PRD |

## Review 결과

| 단계 | 회차 | 결과 |
|------|:----:|------|
| Claude 메타 리뷰 | r1 | **통과** — 평균 8.1, 최저 7 (항목 6 경계 명확성·항목 7 분기 충분성). 지적 (간소 PRD 추출 섹션 매핑·임베드 실패 분기·임베드 완전성 검증) → r1 통합 반영 |
| Codex 메타 리뷰 | — | 정상 — P2 1건 (간소 PRD 추출 매핑 불일치). dogfood 영향 회피 위해 자발 반영 (등급 무관) |
| 통합 반영 | r1 | rp-ship 단계 8 PRD 유형별 추출 분기 + 임베드 실패 1회 재시도 후 ship 중단 + harness-ship 정책 표 + 핵심 동작 동기화 |
