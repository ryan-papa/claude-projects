# PRD: Coding Principles 흡수 (간소)

분류: 하네스 메타 변경 (단축 경로)
출처 참고: forrestchang/andrej-karpathy-skills CLAUDE.md (Karpathy-style guidelines)

## 1. 변경 이유

LLM 코딩 어시스턴트가 자주 저지르는 3가지 실수(가정·과잉구현·범위 외 수정)를 방지하기 위한 행동 원칙을 하네스에 흡수한다. 기존 12단계 워크플로우·리뷰 게이트는 절차 강제에 강하지만, **개별 편집 단위에서의 자제력**을 직접 규정하지 않는다. 본 변경으로 그 공백을 메운다.

원문 4원칙 중 `Goal-Driven Execution`은 PRD·태스크·QA 게이트가 이미 더 강하게 강제하므로 제외하고, 나머지 3원칙(Think · Simplicity · Surgical)만 채택한다.

## 2. 영향 파일

| 파일 | 변경 내용 |
|------|----------|
| `CLAUDE.md` | `## Interaction` 다음에 `## Coding Principles` 섹션 신설 (3행 표) |
| `docs/harness-code-review.md` | "특수 규칙" 표에 Coding Principles 위반 시 감점 1줄 추가. 매핑: Think → 1 정확성, Simplicity → 2 설계, Surgical → 7 유지보수성 |

신규 파일·디렉터리 추가 없음. 스킬 변경 없음.

## 3. 롤백

| 단계 | 조치 |
|------|------|
| 1 | `git revert <merge-commit>` 1회로 완전 복구 |
| 2 | 별도 데이터·설정 변경 없음 → 추가 보정 불필요 |

## 4. 검증

| 항목 | 방법 |
|------|------|
| CLAUDE.md 200줄 이하 유지 | `wc -l CLAUDE.md` (신설 섹션 ≤ 8줄) |
| harness-code-review.md 200줄 이하 유지 | `wc -l docs/harness-code-review.md` |
| 링크 깨짐 없음 | grep 으로 신규 앵커 참조 없음 확인 (이번 변경은 외부 참조 0건) |
| 기획·엔지·코드 리뷰 게이트 통과 | 서브에이전트 + Codex 각 1회, 모두 통과 |
