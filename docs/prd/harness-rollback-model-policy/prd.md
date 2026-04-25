# PRD: 모델 선택 정책 롤백 (간소)

| 항목 | 값 |
|---|---|
| 분류 | 하네스 메타 변경 |
| 대상 | PR #31 + #32 전체 롤백 |
| 브랜치 | `feat/harness-rollback-model-policy` |
| base | `main` |

## 1. 변경 이유

| 사유 | 내용 |
|---|---|
| 동작 미작동 | skill frontmatter `model:` 필드 자동 전환이 실제로 적용되지 않음 (사용자 보고) |
| 효용 부재 | 자동 강제가 동작하지 않으면 정책 표·문구만 남아 SSOT 노이즈 증가 |
| 단순화 | "사용자가 `/model` 로 직접 전환" 단일 운영 모델 복귀, 단계별 권장 모델 표 제거 |
| 비용 회수 | 정책 유지 비용(문서 동기화·리뷰 분기) > 기대 이득 |

## 2. 영향 파일

| 파일 | 변경 |
|---|---|
| `CLAUDE.md` | "단계별 모델" 컬럼·"모델 선택 정책 (자동 적용)" 단락·Opus override 예외 단락 제거 |
| `docs/harness-workflow.md` | 단계별 권장 모델 컬럼·정책 절 제거 |
| `docs/skills/rp-*.md` (13개) | frontmatter `model:` 필드 제거 |
| `docs/prd/harness-model-policy/` | 디렉터리 삭제 (PRD + 리뷰 증거) |
| `docs/prd/harness-model-frontmatter/` | 디렉터리 삭제 (PRD + 리뷰 증거) |

실행: `git revert 3d19db4 2441d79` 두 커밋 되돌림으로 일괄 처리.

## 3. 롤백

| 시나리오 | 방법 |
|---|---|
| 본 PR 롤백 필요 | 본 PR(머지 후 부여될 번호) 의 머지 커밋을 `git revert -m 1 <merge-sha>` |
| 정책 재도입 | 기존 PR #31·#32 의 SHA(`2441d79`·`3d19db4`)를 cherry-pick 또는 새 PR로 재제안 (frontmatter 자동 전환 동작 검증 후) |

## 4. 검증

| 항목 | 방법 |
|---|---|
| frontmatter 잔존 여부 | `grep -l "^model:" docs/skills/rp-*.md` 결과 0건 |
| CLAUDE.md 정책 문구 잔존 | `grep -E "권장 모델\|모델 선택 정책\|Opus override" CLAUDE.md` 결과 0건 |
| workflow.md 동기화 | `docs/harness-workflow.md` 단계 표에 모델 컬럼 부재 확인 |
| PRD 폴더 삭제 | `ls docs/prd/harness-model-*` → No such file |
| Codex 스킬 동기화 | `rtk python3 scripts/sync-codex-skills.py --check` 통과 |
| 리뷰 게이트 | meta r1 Claude(서브에이전트) ≥ 8.0 + `/codex:review --wait` 1회 |
