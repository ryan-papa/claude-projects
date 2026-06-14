# PRD: 베이스 브랜치 origin 최신화 + 워크트리 격리 (간소)

**유형:** 하네스 메타 변경

개발 진입 시 origin 기준 최신 코드에서 통합 브랜치를 생성하고, 통합 브랜치별 git worktree로 작업 공간을 격리한다.

## 변경 이유

| 문제 | 영향 |
|------|------|
| 로컬 stale 브랜치에서 통합 브랜치 생성 | 최종 머지 시 충돌·깨짐 |
| 베이스 브랜치 감지 규칙 부재 | 레포마다 main/develop 혼용, 수동 판단 |
| 통합 브랜치(=기능/프로젝트) 간 작업 공간 공유 | 서로 다른 작업의 파일 충돌, 동시 진행 불가 |

**격리 범위 명확화:** 본 변경의 격리 대상은 **통합 브랜치(서로 다른 기능/프로젝트) 간**이다. 동일 통합 브랜치 내 태스크는 에이전트당 1태스크 규칙([`harness-dev.md`](../../harness-dev.md))에 따라 단일 워크트리 안에서 순차 진행한다(태스크 병렬 격리 아님).

## 변경 사양

| 항목 | 규칙 |
|------|------|
| 베이스 감지 | 유저 미지정 시 `origin/develop → origin/main → origin/master` 순. `git ls-remote --heads origin <b>`로 원격 존재 확인, 첫 존재 채택 |
| origin 미존재·베이스 전무 | origin 리모트 없거나 3개 후보 모두 부재 시 **중단 + 사용자 질의**(빈/신규 레포는 사용자가 베이스 명시). 임의 로컬 fallback 금지 |
| 최신화 | 통합 브랜치 생성 전 `git fetch origin`. 실패 시 중단·보고. origin/[base] 기준 생성, 로컬 stale 사용 금지 |
| 메인 클론 브랜치 정책 | 메인 클론(`repositories/[project]/`)은 **베이스 브랜치에 유지**. 통합 브랜치는 **워크트리에서만 생성·체크아웃** → 동일 브랜치 동시 체크아웃 충돌(git 제약) 회피 |
| 격리 단위 | 통합 브랜치(`feat/[project]`) 1개당 워크트리 1개. 태스크 브랜치 생성·체크아웃·통합 브랜치 머지 **모두 해당 워크트리 안에서** 수행 |
| 워크트리 경로 | `repositories/[project]/worktrees/feat-[project]/` (서브레포 내부, leaf는 브랜치 슬래시→대시). 생성 명령이 `-C repositories/[project]` 기준 상대경로라 서브레포 내부에 생성 → 서브레포 자체 `.gitignore`에 `worktrees/` 추가(하네스 `.gitignore`는 `repositories/` 전체 제외로 이미 커버) |
| 생성 명령 | `git -C repositories/[project] worktree add worktrees/feat-[project] -b feat/[project] origin/[base]` |
| 수행 시점 | `rp-dev` 진입 시 1회 (베이스 감지 → fetch → 워크트리+통합 브랜치 생성). 이후 모든 개발 작업은 워크트리 디렉터리에서 진행 |
| 재진입 | 워크트리 존재 시 재사용 + `git fetch`. 워킹트리 clean이면 origin/[base] 변경분 rebase 안내, **미커밋 변경 있으면 rebase 강행 금지 → 경고 후 사용자 선택(중단/stash)** |
| 정리 | PR 머지 후 `rp-ship`이 `git worktree remove` 자동 제거. 미커밋·미푸시 변경 감지 시 **보류 + 사용자 알림**, 다음 `rp-dev` 진입 시 재확인 |
| rp-ship 호환 | 워크트리에서 `.git`은 **파일**(gitdir 포인터). rp-ship의 PR base 감지·fail-closed 조건이 `.git` 파일도 정상 인식하도록 보장 |
| 적용 범위 | 서브 레포(`repositories/[project]/`)만. 하네스 메타 레포 제외 |

## 영향 파일

| 파일 | 변경 |
|------|------|
| `docs/harness-dev.md` | 브랜치 전략 섹션: "main에서 생성" → origin 베이스 감지·fetch·워크트리·메인 클론 정책으로 갱신 |
| `docs/harness-absolute-rules.md` | §배포·머지·브랜치에 베이스 감지 순서·origin 최신화 한 줄 SSOT 추가 |
| `docs/skills/rp-dev.md` | 진입 시 베이스 감지·fetch·워크트리 생성·재진입 절차 추가 |
| `docs/skills/rp-ship.md` | 머지 후 워크트리 정리 + `.git` 파일 호환 절차 추가 |
| `.gitignore` | `worktrees/` 제외 추가 |
| `CLAUDE.md` | Project Structure에 `worktrees/` 명시 |
| `.codex/skills/rp-dev/`, `rp-ship/` | sync 스크립트로 변환본 갱신 |

## 롤백 전략

feat 브랜치 미머지 시 브랜치 폐기. 머지 후 문제 시 해당 PR revert (문서·스킬 변경뿐, 런타임 영향 없음).

## 검증

| 게이트 | 방법 |
|--------|------|
| 문서 정합성 | CLAUDE.md 트리·링크 동기화 확인 |
| Codex 스킬 동기화 | `python3 scripts/sync-codex-skills.py --check` |
| 동작 드라이런 | 서브 레포 1곳 대상: 베이스 감지 → 워크트리 생성 → 태스크 브랜치 머지 → rp-ship 정리까지 충돌 없이 수행되는지 1회 확인 |
