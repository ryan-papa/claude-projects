# PRD: 하네스 감사 픽스 PR-C (Mid 8 + Low 1, 간소)

분류: 하네스 메타 변경 (단축 경로)
통합 브랜치: `feat/harness-audit-fixes`

## 1. 변경 이유

PR-A·B 후 잔여 Mid·Low 항목 정리. 우선순위는 SSOT 단일화·진입성·정합성. PR-A/B 진행 중 자연 해소된 Low 항목은 검증만.

| ID | 이슈 | 액션 |
|----|------|------|
| M1 | `harness-ship.md`(110) vs `rp-ship.md`(157) 책임 모호, README/CI/머지 후 검증 본문 중복 | harness-ship.md = 정책, rp-ship.md = 실행 절차로 분리. 중복 본문 → rp-ship.md SSOT, harness-ship.md 는 링크 |
| M2 | `harness-prd.md` L134~163 평가 항목 표 vs `rp-{plan,eng}-review.md` 표 중복 | rp-스킬 = 실행 SSOT. harness-prd.md 는 한 줄 링크 |
| M3 | `harness-prd.md` L11~29 코드블록 + L33~39 표 5단계 중복 | 코드블록 제거, 표 + 후속 항목 bullet 유지 |
| M4 | `rp-amend.md` L29~41 vs `rp-workflow.md` L31~43 12단계 표 중복 | rp-amend = "rp-workflow와 동일, 차이만" 표 유지, 풀 12단계 표 제거 |
| M5 | `rp-{plan,eng,code}-review.md` 의 (a)(b)(c)(d) 프롬프트 템플릿 중복 | `harness-absolute-rules.md` "리뷰 단계 서브에이전트 필수" 절에 (a)~(d) SSOT 추가, 스킬은 1줄 링크 |
| M6 | `harness-{prd,qa,code-review}.md` 하단 `skills/rp-*` 역링크 누락 | 각 파일 말미에 "관련 스킬" 1줄 추가 |
| M7 | `docs/prd/20260417_182747_security-guide_478d1984.md` stale (명명 규약 위반) | `docs/prd/security-guide-init/prd.md` 로 이전 |
| M8 | `docs/prd/harness-codex-review/` 에 prd.md 부재, review만 단독 | placeholder `prd.md` 추가 (감사 추적 복구) 또는 archive 노트 |
| L2 | `harness-backend-test-policy.md` L42·50 표 셀 200자+ | sub-table 또는 번호 리스트 분해 |
| L1 | (사전 해소) PR-A/B 후 CLAUDE.md ⛔ 사용 ≥10 → 3건으로 자연 감소 | 검증만 |
| L3 | (사전 해소) 죽은 링크 0건, 무의미 앵커 0건 | 검증만 |

## 2. 영향 파일

| 파일 | 변경 |
|------|------|
| `docs/harness-ship.md` | README/CI/머지 후 검증 본문 → rp-ship.md 링크. 정책 본문만 유지 |
| `docs/skills/rp-ship.md` | 변경 없음 (이미 실행 SSOT 위치) |
| `docs/harness-prd.md` | L11~29 코드블록 제거, L131~163 평가 항목 표 → rp-스킬 링크. 5단계 흐름은 표만 유지 |
| `docs/skills/rp-plan-review.md`·`rp-eng-review.md`·`rp-code-review.md` | 프롬프트 (a)~(d) 본문 → harness-absolute-rules.md 링크 (M5). 평가 항목 표는 유지(SSOT) |
| `docs/skills/rp-amend.md` | 12단계 표 → "rp-workflow와 동일, 차이 1행만" 형태로 단축 |
| `docs/harness-absolute-rules.md` | "[리뷰 단계 서브에이전트 필수]" 절에 프롬프트 (a)~(d) 표 SSOT 추가 (M5) |
| `docs/harness-qa.md`·`docs/harness-code-review.md`·`docs/harness-prd.md` | 말미에 "관련 스킬: skills/rp-{qa,code-review,prd|plan-review|eng-review}.md" 역링크 추가 (M6) |
| `docs/prd/security-guide-init/prd.md` | **신설** — 기존 stale 파일 이전 (M7) |
| `docs/prd/20260417_182747_security-guide_478d1984.md` | 삭제 (M7 의 이전 후) |
| `docs/prd/harness-codex-review/prd.md` | **신설** — placeholder + 회고 노트 (M8) |
| `docs/harness-backend-test-policy.md` | L42·L50 셀 sub-table 분해 (L2) |

신규 파일 2개(`security-guide-init/prd.md`, `harness-codex-review/prd.md`). 삭제 1개. CLAUDE.md 트리는 변경 없음 (prd/ 디렉터리만 트리에 있음, 내부 파일은 비-등재).

## 3. 롤백

| 단계 | 조치 |
|------|------|
| 1 | `git revert <merge-commit>` 1회로 완전 복구. 신설 파일 자동 삭제, 이전된 파일 자동 복원 |
| 2 | 통합 브랜치(`feat/harness-audit-fixes`)에 머지된 상태 — main 영향 0 |

## 4. 검증

| 항목 | 결과 기준 |
|------|----------|
| 모든 변경 파일 ≤ 200줄 | `wc -l` 검증 |
| 평가 항목 표 SSOT 단일화 | `harness-prd.md` 에 평가 항목 표 본문 부재, rp-스킬에만 존재 |
| (a)~(d) 프롬프트 템플릿 SSOT | `harness-absolute-rules.md` 1곳에만 본문, 스킬은 링크 |
| QA·코드리뷰 역링크 | `harness-{qa,code-review}.md` 말미에 `skills/rp-*` 마크다운 링크 |
| stale 파일 정리 | `docs/prd/20260417_*` 파일 부재, `security-guide-init/prd.md` 존재 |
| harness-codex-review/ 폴더 정리 | `prd.md` 존재 |
| L1 사전 해소 검증 | `grep -c "⛔" CLAUDE.md` 결과 ≤ 5 |
| L3 사전 해소 검증 | bash 죽은 링크 검사 0건 |
| 메타 리뷰 게이트 | Claude `meta` 리뷰 (회차 r{N} 패턴: r1 = PRD 단계 사전 검증, r2 = post-edit 검증. 둘 다 동일 `meta` 타입의 회차) + Codex r1·r2 통합 1파일. 모두 통과 |
