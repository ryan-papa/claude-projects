# PRD: 하네스 감사 픽스 PR-B (High 5건, 간소)

분류: 하네스 메타 변경 (단축 경로)
통합 브랜치: `feat/harness-audit-fixes`

## 1. 변경 이유

감사 High 5건은 SSOT 위반·다책임 혼재 핵심. PR-A 후 표면 정리는 완료했지만 **중복된 규칙 본문**이 그대로라 한쪽만 수정하면 즉시 표류.

| ID | 이슈 | 현 상태 |
|----|------|--------|
| H1 | `CLAUDE.md` 186줄 + `harness-prd.md` 188줄 다책임 | 200줄 임박, 5책임/3책임 혼재 |
| H2 | "자동 머지 가드 4종 AND" 등 동일 규칙 3~5곳 평문 반복 | CLAUDE.md·harness-workflow.md·rp-ship.md·harness-ship.md 4곳 중복 |
| H3 | 토큰/기능 신호 분기 절차 4개 스킬 중복 | `harness-codex-review.md` SSOT 선언 무색 |
| H4 | 프로젝트 유형 QA 분기 표 3곳 중복 | `harness-qa.md`·`rp-qa.md`·`harness-workflow.md` 동일 |
| H5 | `harness-codex-review.md` 산문 1줄 ("본 표가 SSOT…") | 3개 사안 혼합 |

## 2. 영향 파일

| 파일 | 변경 |
|------|------|
| `CLAUDE.md` | (1) "절대 규칙" 섹션 → 별도 파일 분리(`harness-absolute-rules.md` 신설) + 1줄 링크. (2) "Project Structure" 트리에 신규 파일 1행 추가 |
| `docs/harness-absolute-rules.md` | **신설** — CLAUDE.md L130~150 절대 규칙 섹션 이전 (SSOT) |
| `docs/harness-workflow.md` | "절대 규칙" 섹션 중복 제거 → `harness-absolute-rules.md` 링크. "프로젝트 유형 QA 분기" 표 → `harness-qa.md` 링크 |
| `docs/harness-ship.md` | 자동 머지 가드 4종·base 감지 본문 중복 제거 → `harness-absolute-rules.md` + `rp-ship.md` 링크 |
| `docs/skills/rp-ship.md` | 자동 머지 가드·절대 규칙 본문 중복 제거 → `harness-absolute-rules.md` 링크 (실행 절차는 유지) |
| `docs/skills/rp-qa.md` | 프로젝트 유형 분기 표 제거 → `harness-qa.md` 링크 |
| `docs/skills/rp-plan-review.md`·`rp-eng-review.md`·`rp-code-review.md` | 토큰/기능 신호 분기 절차 본문 중복 제거 → `harness-codex-review.md` "토큰·기능 신호 패턴" SSOT 링크 |
| `docs/harness-codex-review.md` | L71 산문 1줄 → 헤더 메타 박스(SSOT/참조 규칙/정규식 옵션 3줄)로 분리 (H5) |
| `docs/harness-prd.md` | 변경 없음 (H1 의 CLAUDE.md 분리만으로 200줄 압박 완화. harness-prd 분할은 PR-C 로 이관) |
| `AGENTS.md` | 변경 없음 (codex 측 shorthand 절대 규칙은 다른 청중용 요약, 충돌 없음 확인) |

신규 파일 1개(`harness-absolute-rules.md`). 다른 신규 없음.

## 3. 롤백

| 단계 | 조치 |
|------|------|
| 1 | `git revert <merge-commit>` 1회로 완전 복구 |
| 2 | 통합 브랜치(`feat/harness-audit-fixes`)에 머지된 상태 — main 도달 전이므로 main 영향 0 |
| 3 | `harness-absolute-rules.md` 삭제는 revert 가 자동 수행 |

## 4. 검증

| 항목 | 결과 기준 |
|------|----------|
| CLAUDE.md ≤ 200줄 | 절대 규칙 분리 후 ~80줄 예상 |
| 모든 변경 파일 ≤ 200줄 | 각 wc -l |
| 절대 규칙 SSOT 단일화 | "자동 머지 가드 4종 AND" 본문은 `harness-absolute-rules.md` 1곳에만 (다른 파일은 링크만) |
| 토큰 신호 분기 SSOT | `harness-codex-review.md` 1곳에만, 4개 리뷰 스킬은 링크 |
| QA 분기 표 SSOT | `harness-qa.md` 1곳에만, `rp-qa.md`·`harness-workflow.md` 는 링크 |
| 죽은 링크 0건 | 모든 신규 링크 검증 |
| 메타 리뷰 게이트 | Claude `meta` 리뷰 r1(PRD 단계) → 편집 → r2(post-edit, 동일 `meta` 타입의 회차로 r{N} 패턴) + Codex r1·r2 통합 1파일. 모두 통과 |
| 죽은 링크 0건 | `bash -c 'find docs/ CLAUDE.md -name "*.md" \| while read f; do dir=$(dirname "$f"); grep -oP "\[.*?\]\(\K[^)]+" "$f" \| grep -v "^http\|^#" \| while read l; do t="$dir/${l%%#*}"; [ ! -e "$t" ] && echo "BROKEN $f -> $l"; done; done'` 출력 0줄 |

검증 명령:

```bash
grep -rn "자동 머지 가드 4종 AND" CLAUDE.md docs/ | grep -v 'review-' | grep -v 'absolute-rules'
grep -rn "토큰.*기능 신호 패턴" docs/skills/ | grep -v 'codex-review'
wc -l CLAUDE.md docs/harness-absolute-rules.md docs/harness-workflow.md docs/harness-ship.md docs/skills/rp-ship.md docs/skills/rp-qa.md docs/skills/rp-plan-review.md docs/skills/rp-eng-review.md docs/skills/rp-code-review.md docs/harness-codex-review.md
```
