# PRD: 하네스 감사 픽스 PR-A (Critical 3건, 간소)

분류: 하네스 메타 변경 (단축 경로)
통합 브랜치: `feat/harness-audit-fixes`

## 1. 변경 이유

직전 감사에서 평균 6.7/10. Critical 3건은 에이전트 도달성·정확성에 직접 영향 — 워크플로우 중단 위험. 본 PR 3개 묶음의 첫 번째.

| ID | 이슈 | 영향 |
|----|------|------|
| C1 | `CLAUDE.md` `## Project Structure` 트리가 실제 파일과 불일치 (`harness-backend-test-policy.md`·`harness-db.md`·`docs/tasks.md`·`docs/prd/`·`docs/images/` 누락) | 트리를 SSOT로 신뢰하면 도달 불가 |
| C2 | 6개 핵심 스킬(`rp-task`·`rp-specify`·`rp-dev`·`rp-qa`·`rp-code-review`·`rp-amend`) 인바운드 마크다운 링크 0건 | 진입점에서 클릭 동선 부재 |
| C3 | `CLAUDE.md` 절대 규칙 3개 bullet (L131·L136·L150)이 600자+ 산문, 4~6개 사안 압축 | 파싱 정확도·가독성 저하 |

## 2. 영향 파일

| 파일 | 변경 내용 |
|------|----------|
| `CLAUDE.md` | (1) `## Project Structure` 트리 누락 5건 추가 (2) 절대 규칙 3개 bullet → 표/sub-bullet 분해 |
| `docs/harness-workflow.md` | 12단계 표 각 행에 `[/rp-xxx](skills/rp-xxx.md)` 마크다운 링크 보강 |

신규 파일·디렉터리 추가 없음. 스킬 본문 변경 없음.

## 3. 롤백

| 단계 | 조치 |
|------|------|
| 1 | `git revert <merge-commit>` 1회로 완전 복구 |
| 2 | 통합 브랜치(`feat/harness-audit-fixes`)에 머지된 상태 — main 도달 전이므로 main 영향 0 |

## 4. 검증

| 항목 | 결과 기준 |
|------|----------|
| CLAUDE.md 트리 ↔ 실제 일치 | diff 0 |
| 6개 스킬 인바운드 링크 ≥ 1 each | match 수 ≥ 6 |
| 절대 규칙 산문 → 표 분리 | 600자+ bullet 0개 |
| 200줄 한도 유지 | CLAUDE.md·harness-workflow.md ≤ 200 |
| 메타 리뷰 게이트 | Claude r1·r2 + Codex 1회 모두 통과 |

검증 명령:

```bash
# 트리 일치는 메타 코드 리뷰 단계 서브에이전트가 수동 비교 (find vs CLAUDE.md 트리 cell)
find docs -maxdepth 2 -type f -name '*.md' | sort
grep -cE 'skills/rp-(task|specify|dev|qa|code-review|amend)\.md' docs/harness-workflow.md
awk '/^- / && length>600 {print NR}' CLAUDE.md
wc -l CLAUDE.md docs/harness-workflow.md
```
