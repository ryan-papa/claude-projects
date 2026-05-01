# PRD: 하네스 감사 픽스 통합 (최종 PR, 간소)

분류: 하네스 메타 변경 (단축 경로) — 통합 브랜치 → main

## 1. 변경 이유

PR-A·B·C 3건이 통합 브랜치 `feat/harness-audit-fixes` 에 순차 머지 완료. 본 PR 은 통합 결과를 main 으로 합치는 최종 단계. 추가 신규 변경 없음 (3개 PR 의 sum).

## 2. 영향 파일

| 그룹 | 파일 |
|------|------|
| PR-A 산물 | CLAUDE.md (트리·산문 분해), docs/harness-workflow.md (스킬 인덱스), audit-A-critical/ |
| PR-B 산물 | docs/harness-absolute-rules.md 신설, CLAUDE.md·harness-workflow·harness-ship·rp-ship·rp-qa·rp-{plan,eng,code}-review·harness-codex-review·harness-qa, audit-B-high/ |
| PR-C 산물 | harness-ship·harness-prd·harness-absolute-rules·rp-amend·rp-{plan,eng,code}-review·harness-{qa,code-review}·harness-backend-test-policy, security-guide-init/ 이전, harness-codex-review/prd.md 신설, tasks.md 정정, audit-C-mid-low/ |

자세한 내역은 각 PR (#37·#38·#39) 본문 참조.

## 3. 롤백

| 단계 | 조치 |
|------|------|
| 1 | 통합 PR `git revert <merge-commit>` → main 에서 PR-A·B·C 모두 한번에 제거 |
| 2 | 또는 개별 PR 별 revert (PR-A·B·C 각각 squash 1 커밋) |

## 4. 검증

| 항목 | 결과 기준 |
|------|----------|
| CI lint-docs | 통과 (PR base=main 이므로 정식 트리거됨) |
| 모든 변경 파일 ≤ 200줄 | `wc -l` 검증 (max harness-code-review.md 184) |
| 죽은 링크 0건 | bash 검증 (PR-A·B·C 각각 통과 확인됨) |
| 절대 규칙 SSOT | `harness-absolute-rules.md` 1곳 |
| (a)~(d) 프롬프트 SSOT | `harness-absolute-rules.md` 1곳 |
| QA 분기 SSOT | `harness-qa.md` 1곳 |
| 토큰 신호 SSOT | `harness-codex-review.md` 1곳 |
| 평가 항목 SSOT | `rp-{plan,eng,code}-review.md` |
| 메타 리뷰 게이트 | (각 sub-PR 누적: PR-A·B·C 모두 r1·r2 + Codex r1·r2 통과) + 본 통합 PR Claude `meta` r1 + Codex 1회 통과. r2 는 본 PR 에 신규 편집 없으므로 적용 외 (PR-A·B·C 가 이미 post-edit 검증 완료). |
