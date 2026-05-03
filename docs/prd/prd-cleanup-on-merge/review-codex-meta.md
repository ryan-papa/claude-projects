2026-05-03T07:27:14.000207Z ERROR codex_models_manager::manager: failed to refresh available models: timeout waiting for child process to exit
OpenAI Codex v0.128.0 (research preview)
--------
workdir: /Users/hose.kim/Claude/workflow-agent-harness
model: gpt-5.5
provider: openai
approval: never
sandbox: workspace-write [workdir, /tmp, $TMPDIR, /Users/hose.kim/.codex/memories]
reasoning effort: medium
reasoning summaries: none
session id: 019decbb-f450-70e0-833e-19fd003fc52d
--------
user
changes against 'main'
exec
/bin/zsh -lc 'rtk git diff 89f68d6375aa6fea967ad9dc21ca199888aac5e1' in /Users/hose.kim/Claude/workflow-agent-harness
 succeeded in 0ms:
.codex/skills/rp-ship/SKILL.md       | 16 +++++++-
 docs/harness-absolute-rules.md       |  1 +
 docs/harness-ship.md                 | 15 +++++++
 docs/prd/prd-cleanup-on-merge/prd.md | 80 ++++++++++++++++++++++++++++++++++++
 docs/skills/rp-ship.md               | 16 +++++++-
 5 files changed, 124 insertions(+), 4 deletions(-)

--- Changes ---

.codex/skills/rp-ship/SKILL.md
  @@ -78,8 +78,20 @@ ls "$FEATURE"/review-codex-meta.md >/dev/null 2>&1 || exit 1
  -8. **머지 실행**: `gh pr merge <num> --merge --delete-branch` (전략 `--merge` 고정)
  -9. **배포 확인**: 배포 워크플로우 완료 대기 + 결과 보고
  +
  +### PRD 정리 (머지 직전, 동일 PR)
  +8. **PRD 요약 PR 본문 임베드**:
  +   - 추출 섹션: `## 개요·목적`, `## 기능 요구사항`, PRD 하단 `## Review 결과`
  +   - 적용: `gh pr edit <num> --body "$(...)"` — 기존 본문 끝에 `<details><summary>PRD 요약</summary>...</details>` 추가
  +9. **PRD 디렉토리 삭제 커밋**:
  +   - `git rm -r <project-root>/docs/prd/[feature]/`
  +   - `git commit -m "chore(prd): merge 직전 PRD 정리"`
  +   - `git push`
  +10. **CI 재대기**: 정리 커밋 CI 통과만 확인 (`gh pr checks <num>`). 가드 (b) 재실행 면제 (정리 직전 통과 + 요약 임베드 완료)
  +
  +### 머지·배포
  +11. **머지 실행**: `gh pr merge <num> --merge --delete-branch` (전략 `--merge` 고정)
  +12. **배포 확인**: 배포 워크플로우 완료 대기 + 결과 보고
   
   ### 비상 탈출구
   
  +14 -2

docs/harness-absolute-rules.md
  @@ -48,6 +48,7 @@
  +- **머지 직전 PRD 정리 필수**: 자동 머지 가드 4종 통과 후 머지 실행 전, 동일 PR 내에서 PRD 디렉토리(`<project-root>/docs/prd/[feature]/`)를 `git rm -r` 통째 삭제 + PR 본문에 요약(개요·기능 요구사항·Review 결과) 임베드 + 정리 커밋 CI 재통과 확인 후 머지. Full PRD·간소 PRD 모두 적용. 가드 (b)는 정리 커밋 직전까지만 유효(정리 후 재실행 면제). 정리 단계 누락 시 머지 차단. 머지 후 PRD 참조는 **PR 본문 요약 + git history**로만 가능
   - **`rp-ship` PR base 자동 감지 게이트**: 통합 브랜치 선언 감지 → `--base` 주입
     - 감지 순서: (0) 메타 분기 선검사 (`review-claude-meta-r*.md` 또는 `review-codex-meta.md` 존재 → `--base main`) → (1) `docs/tasks.md` → (2) `CLAUDE.md` → (3) repo default
     - 매칭 정규식: `^[\s\-\*|]*통합 브랜치:\s*`?([A-Za-z0-9/_\-]+)`?` 정확히 1건만 채택
  +1 -0

docs/harness-ship.md
  @@ -42,6 +42,21 @@ QA + 코드리뷰(또는 콘텐츠 검수) 통과 후 사용자에게 결과를
  +## 머지 직전 PRD 정리 (정책)
  +
  +가드 4종 AND 통과 후, 머지 실행 전에 **동일 PR**에서 PRD 디렉토리를 통째 삭제한다. Full PRD·간소 PRD 모두 적용. 머지 후 PRD 참조는 **PR 본문 요약 + git history**로만 가능.
  +
  +| 단계 | 내용 |
  +|------|------|
  +| 요약 임베드 | PRD `## 개요·목적` + `## 기능 요구사항` + `## Review 결과` 추출 → PR 본문 `<details>` 블록 |
  +| 디렉토리 삭제 | `git rm -r <project-root>/docs/prd/[feature]/` (Full + 간소 동일) |
  +| 정리 커밋 | `chore(prd): merge 직전 PRD 정리` |
  +| CI 재통과 | 정리 커밋 CI SUCCESS 확인 (가드 b 재실행 면제) |
  +| 누락 시 | 머지 차단 |
  +
  +→ 실행 절차: [`skills/rp-ship.md`](skills/rp-ship.md) "PRD 정리" SSOT
  +→ 절대 규칙: [`harness-absolute-rules.md`](harness-absolute-rules.md) "머지 직전 PRD 정리 필수"
  +
   ## 머지 후 정책
   
   - 배포 완료 메시지 직후 `/rp-retro` 자동 진입 **금지** ([`harness-absolute-rules.md`](harness-absolute-rules.md): 회고는 사용자 명령 시에만)
  +15 -0

docs/prd/prd-cleanup-on-merge/prd.md
  @@ -0,0 +1,80 @@
  +# PRD — 머지 직전 PRD 자동 정리
  +
  +| 메타 | 값 |
  +|------|----|
  +| 유형 | 하네스 메타 변경 (간소 PRD) |
  +| 상태 | Draft |
  +| 통합 브랜치 | `main` (메타 분기 → `--base main` 자동) |
  +| 작업 브랜치 | `feat/prd-cleanup-on-merge` |
  +
  +## 변경 이유
  +
  +PRD·리뷰 증거 파일이 `repositories/[project]/docs/prd/[feature]/` 및 `docs/prd/[feature]/`에 누적되어 트리가 비대해진다. 머지 = 기능 출하 시점이므로 PRD는 의사결정 기록으로서 역할이 종료된다. 이 시점에 같은 PR diff 안에서 PRD 디렉토리를 통째 삭제하고, 핵심 요약(개요·기능 요구사항·Review 결과)을 PR 본문에 임베드하여 추후 참조성은 PR + git history로 보존한다.
  +
  +**해결 문제:**
  +- `docs/prd/` 트리 누적 → 탐색·리뷰 노이즈
  +- 사후 PRD 단일 출처 모호 (개발 중엔 파일, 머지 후엔 git history)
  +- 회고·문서 검색 시 outdated PRD 혼선
  +
  +**대안 비교:**
  +
  +| 방안 | 평가 |
  +|------|------|
  +| 즉시 삭제 + PR 본문 요약 임베드 (채택) | 트리 항상 깨끗, PR 본문이 사후 조회 anchor |
  +| `_archive/`로 이동 | 트리는 정리되나 archive 누적 → 동일 문제 재발 |
  +| 머지 후 별도 PR로 정리 | 자동화 어렵고 누락 위험, PR diff 분리되어 가독성 저하 |
  +| 상태만 `Shipped` 마킹 | 누적 미해결 |
  +
  +## 영향 파일
  +
  +| 파일 | 변경 내용 |
  +|------|----------|
  +| [`docs/harness-absolute-rules.md`](../../harness-absolute-rules.md) | "배포·머지·브랜치" 섹션에 "머지 직전 PRD 정리 필수" 절대 규칙 1줄 추가 |
  +| [`docs/skills/rp-ship.md`](../../skills/rp-ship.md) | 자동 머지 가드 통과 후 머지 직전 단계 8(요약 임베드)·9(`git rm -r` 정리 커밋)·10(CI 재대기) 신설, 머지·배포는 11·12로 재번호 |
  +| [`docs/harness-ship.md`](../../harness-ship.md) | "머지 직전 PRD 정리" 정책 섹션 신설 (단계별 표 + SSOT 링크) |
  +| `.codex/skills/rp-ship/SKILL.md` | `scripts/sync-codex-skills.py`로 자동 동기화 (수동 편집 아님) |
  +| `docs/prd/prd-cleanup-on-merge/` | 본 PRD + 리뷰 증거 (머지 직전 본 규칙에 의해 자체 삭제됨 — dogfooding) |
  +
  +**적용 범위:** Full PRD (`repositories/[project]/docs/prd/[feature]/`) + 간소 PRD (`docs/prd/[feature]/`) 둘 다.
  +
  +**핵심 동작 (`rp-ship` 절차):**
  +1. 가드 4종 AND 통과 (CI · 리뷰 증거 · base 정상 · MERGEABLE)
  +2. PRD `## 개요·목적` + `## 기능 요구사항` + `## Review 결과` 추출 → PR 본문 `<details>` 블록 임베드 (`gh pr edit --body`)
  +3. `git rm -r <project-root>/docs/prd/[feature]/`
  +4. 정리 커밋 `chore(prd): merge 직전 PRD 정리` + push
  +5. CI 재통과 확인 (가드 b 재실행 면제)
  +6. `gh pr merge --merge --delete-branch`
  +
  +## 롤백 전략
  +
  +| 조건 | 조치 |
  +|------|------|
  +| 정책 자체 철회 | 본 PR을 `git revert` → 절대 규칙·rp-ship·harness-ship 원복 |
  +| 정리 커밋만 실패 (CI 미통과 등) | OPEN 유지 + 사용자 보고 → 수동 조치 (자동 재시도 금지) |
  +| 머지 후 PRD 참조 필요 | PR 본문 `<details>` 요약 + `git log --all -- docs/prd/[feature]/` 로 git history 조회 |
  +| 본 dogfood PR 자체 정리 단계 실패 | OPEN 유지, PR 본문에 "self-host 첫 적용 — 문제 시 본 규칙 revert 검토" 명시 |
  +
  +## 검증
  +
  +| 항목 | 방법 |
  +|------|------|
  +| Claude 메타 리뷰 1회 | `/rp-plan-review` 서브에이전트 (5항목 메타 적용, 평균 ≥ 8.0 + 항목별 ≥ 7) → `review-claude-meta-r1.md` |
  +| Codex 메타 리뷰 1회 (병렬) | `codex review` → `review-codex-meta.md`. High/Critical 반영 |
  +| 자동 머지 가드 4종 | 정책 자체 변경이므로 가드 (b) 리뷰 증거 게이트는 본 PRD 디렉토리 메타 파일 2종 존재로 통과 |
  +| Dogfood 검증 | 본 PR 머지 시 신설 절차로 본 PRD 디렉토리 자체 삭제 (self-host) |
  +| CI lint-docs | docs 변경 통과 확인 |
  +
  +## Open Issues
  +
  +| 항목 | 상태 |
  +|------|------|
  +| 회고 시점 PRD 참조 경로 (PR 본문 vs git show) | 본 PRD에서 PR 본문 + git history로 명시 결정 |
  +| `<project-root>` 결정 (일반 기능 vs 메타) | rp-ship 사전 체크 게이트 로직 그대로 재사용 — 추가 설계 불필요 |
  +| 다중 PRD 디렉토리 동시 정리 (1 PR 다 feature) | 본 사이클 대상 외 — 등장 시 별도 PRD |
  +
  +## Review 결과
  +
  +| 단계 | 회차 | 결과 |
  +|------|:----:|------|
  +| Claude 메타 리뷰 | r1 | (대기) |
  +| Codex 메타 리뷰 | — | (대기) |
  +80 -0

docs/skills/rp-ship.md
  @@ -67,8 +67,20 @@ ls "$FEATURE"/review-claude-meta-r*.md >/dev/null 2>&1 || exit 1
  -8. **머지 실행**: `gh pr merge <num> --merge --delete-branch` (전략 `--merge` 고정)
  -9. **배포 확인**: 배포 워크플로우 완료 대기 + 결과 보고
  +
  +### PRD 정리 (머지 직전, 동일 PR)
  +8. **PRD 요약 PR 본문 임베드**:
  +   - 추출 섹션: `## 개요·목적`, `## 기능 요구사항`, PRD 하단 `## Review 결과`
  +   - 적용: `gh pr edit <num> --body "$(...)"` — 기존 본문 끝에 `<details><summary>PRD 요약</summary>...</details>` 추가
  +9. **PRD 디렉토리 삭제 커밋**:
  +   - `git rm -r <project-root>/docs/prd/[feature]/`
  +   - `git commit -m "chore(prd): merge 직전 PRD 정리"`
  +   - `git push`
  +10. **CI 재대기**: 정리 커밋 CI 통과만 확인 (`gh pr checks <num>`). 가드 (b) 재실행 면제 (정리 직전 통과 + 요약 임베드 완료)
  +
  +### 머지·배포
  +11. **머지 실행**: `gh pr merge <num> --merge --delete-branch` (전략 `--merge` 고정)
  +12. **배포 확인**: 배포 워크플로우 완료 대기 + 결과 보고
   
   ### 비상 탈출구
   
  +14 -2

exec
/bin/zsh -lc "rtk sed -n '1,180p' docs/skills/rp-ship.md && printf '\\n---\\n' && rtk sed -n '1,120p' docs/harness-ship.md && printf '\\n---abs---\\n' && rtk sed -n '1,90p' docs/harness-absolute-rules.md" in /Users/hose.kim/Claude/workflow-agent-harness
 succeeded in 0ms:
---
description: '[11] 커밋·PR·CI·자동머지·배포. 수동 git/gh 우회 금지, 반드시 이 스킬 경유'
argument-hint: '[브랜치명] [변경 요약]'
---

# rp-ship

커밋 → PR → CI 확인 → **자동 머지(가드 4종 AND)** → 배포.

## 트리거

- 산출물 보고 후 자동 시작 (사용자 승인 단계 없음)
- `/rp-ship` 명령

## 절차

### ⛔ 사전 체크 게이트 (커밋 전 필수)

커밋 직전 `<project-root>/docs/prd/[feature]/` 디렉터리에서 리뷰 증거 파일 존재 검증. 누락 시 ship **중단** + 해당 리뷰 단계로 복귀.

| PRD 유형 | 필수 파일 |
|---------|----------|
| 일반 기능 | `review-claude-plan-r{N}.md` + `review-codex-plan.md` + `review-claude-eng-r{N}.md` + `review-codex-eng.md` + `review-claude-code-r{N}.md` + `review-codex-code.md` |
| 하네스 메타 변경 | `review-claude-meta-r{N}.md` + `review-codex-meta.md` |
| 회고 반영 사이클 | 상기 조건 + `retro-r{N}.md`(반영 근거) |

검증 방식: **PRD 유형별 필수 파일을 개별적으로 존재 확인**. 부분 존재는 통과 불가.

```bash
# 일반 기능 — 6개 전부 존재해야 통과 (단, review-claude-*-r*.md는 최소 1개 이상 회차)
FEATURE=docs/prd/[feature]
for f in \
  "$FEATURE"/review-codex-plan.md \
  "$FEATURE"/review-codex-eng.md \
  "$FEATURE"/review-codex-code.md; do
  [ -f "$f" ] || { echo "MISSING: $f"; exit 1; }
done
# Claude 회차 파일은 단계별로 최소 1개
for stage in plan eng code; do
  ls "$FEATURE"/review-claude-${stage}-r*.md >/dev/null 2>&1 \
    || { echo "MISSING: review-claude-${stage}-r*.md"; exit 1; }
done

# 하네스 메타 변경 — 2종 존재 필수
ls "$FEATURE"/review-claude-meta-r*.md >/dev/null 2>&1 || exit 1
[ -f "$FEATURE"/review-codex-meta.md ] || exit 1
```

하나라도 누락 시 "리뷰 증거 부족 — 해당 리뷰 단계로 복귀" 메시지 출력 후 ship 중단.

### 자동 수행 (게이트 통과 시)
0. **PR base 결정 (fail-closed)**: 아래 "PR base 자동 감지" 섹션에 따라 감지. 비정상 상태는 ship 중단 후 사용자 확인 요구.
1. **커밋**: 변경 파일만 `git add` + `git commit`
2. **README 검증**: 푸시 전 README.md 점검 (아래 참조)
3. **푸시**: `git push -u origin [branch]`
4. **PR 상태 확인**: `gh pr list --head [branch] --state all --json number,state,baseRefName`
   - OPEN PR 존재 + base 일치 → 해당 PR 재사용
   - OPEN PR 존재 + base 불일치 → `gh pr edit <num> --base <detected-base>` 로 보정 후 **CI 재실행 대기** (base 변경은 diff 범위 재정의. 자동 머지 가드는 재실행된 CI 통과를 다시 확인)
   - MERGED/CLOSED만 존재하거나 PR 없음 → 신규 PR 생성
5. **PR 생성/재사용**: `gh pr create --base <detected-base> ...` (제목 + 변경 요약 + 테스트 계획)
6. **CI 확인**: CI 통과 대기

### 자동 머지 (가드 4종 AND)
7. **자동 머지 안전 가드** — 모두 충족 시에만 진행:
   - (a) CI 모든 체크 SUCCESS (`gh pr checks <num>` exit 0)
   - (b) 리뷰 증거 파일 게이트 통과 (위 사전 체크 게이트 재확인)
   - (c) PR base 자동 감지 또는 메타 분기 결과 정상 (fail-closed 통과)
   - (d) `gh pr view <num> --json mergeable` 가 `MERGEABLE`
   하나라도 실패 → **자동 머지 중단 + PR 상태 OPEN 유지 + 사용자 즉시 보고**. `--admin`·`--no-verify` 우회 금지.

### PRD 정리 (머지 직전, 동일 PR)
8. **PRD 요약 PR 본문 임베드**:
   - 추출 섹션: `## 개요·목적`, `## 기능 요구사항`, PRD 하단 `## Review 결과`
   - 적용: `gh pr edit <num> --body "$(...)"` — 기존 본문 끝에 `<details><summary>PRD 요약</summary>...</details>` 추가
9. **PRD 디렉토리 삭제 커밋**:
   - `git rm -r <project-root>/docs/prd/[feature]/`
   - `git commit -m "chore(prd): merge 직전 PRD 정리"`
   - `git push`
10. **CI 재대기**: 정리 커밋 CI 통과만 확인 (`gh pr checks <num>`). 가드 (b) 재실행 면제 (정리 직전 통과 + 요약 임베드 완료)

### 머지·배포
11. **머지 실행**: `gh pr merge <num> --merge --delete-branch` (전략 `--merge` 고정)
12. **배포 확인**: 배포 워크플로우 완료 대기 + 결과 보고

### 비상 탈출구

환경변수 `RP_SHIP_MANUAL=1` 설정 시 자동 머지 비활성. CI 통과 후 PR URL 보고 → 사용자 승인 대기 → 승인 시 `gh pr merge --merge`. 위험 PR·실험적 변경 시 사용.

## PR base 자동 감지

PR 생성/리타깃 전 **통합 브랜치 선언을 감지**해 `--base` 에 주입. `gh pr create` 기본값(레포 default branch)에 의존 금지.

### 감지 순서 (엄격)

| # | 소스 | 규칙 | 실패 시 |
|:-:|------|------|--------|
| 0 | **메타 변경 분기 (선검사)** | 현재 ship 사이클의 PRD 폴더에 `review-claude-meta-r*.md` 또는 `review-codex-meta.md` 가 존재하면 **자동 감지 우회** + `--base main` 적용 | 메타 파일 부재 → 다음 소스로 진행 |
| 1 | 프로젝트 `docs/tasks.md` | 라인 시작 앵커 정규식 `^[\s\-\*|]*통합 브랜치:\s*`?([A-Za-z0-9/_\-]+)`?` 로 **정확히 1건** 매칭 | 2건 이상 → fail-closed / 0건 → 다음 소스 |
| 2 | 프로젝트 `CLAUDE.md` | 동일 정규식 정확히 1건 매칭 | 2건 이상 → fail-closed / 0건 → 다음 소스 |
| 3 | 레포 default branch | `gh repo view --json defaultBranchRef` | 결정 불가 → fail-closed |

**예시 (museum-finder `docs/tasks.md`)**: `- 통합 브랜치: \`feat/mvp-v1\` · 태스크 브랜치 \`feat/T-NN-{slug}\`` → `feat/mvp-v1` 1건 매칭 → `--base feat/mvp-v1`.

### 수동 오버라이드

사용자가 `rp-ship --base <X>` 로 명시 전달 시 자동 감지 **비활성화**하고 `<X>` 사용. 감지 로직을 거치지 않는 유일한 우회 경로.

### Fail-closed 조건 (ship 중단, 사용자 확인 요구)

- 감지된 브랜치 값에 공백/`\n` 포함
- 감지 정규식이 2건 이상 매칭 (모호성)
- 감지된 브랜치가 원격에 부재 (`git ls-remote --exit-code origin refs/heads/<base>` 실패)
- `HEAD` 가 detached 상태
- 프로젝트 루트(`.git` 디렉터리)를 찾지 못함
- 느슨한 브랜치명 추론(`feat/mvp-*` 등) 금지 — 반드시 `통합 브랜치:` 앵커 필드만 권위

## README 검증

커밋 후·푸시 전 README.md가 현재 프로젝트와 일치하는지 검증.

**검증 항목** (각 10점):

| # | 항목 |
|---|------|
| 1 | 프로젝트 개요 — 목적/기능 설명 일치 |
| 2 | 디렉토리 구조 — 주요 폴더/파일 반영 |
| 3 | 설치/실행 방법 — 의존성·스크립트 일치 |
| 4 | 기능 목록 — 구현된 주요 기능 기재 |
| 5 | 기술 스택 — 실제 사용 기술 명시 |

**통과:** 평균 8.0점 이상
**실패 시:** README 수정 → 재검증 (최대 3회) → 3회 실패 시 사용자 판단 위임

## CI 분기

```
PR 생성 시점에 .github/workflows/ 확인
  ├── CI 있음 → gh pr merge --merge --auto (CI 통과 대기)
  └── CI 없음 → "CI가 없습니다. 추가할까요?" 사용자에게 질문
       ├── 추가 원함 → CI 워크플로우 생성 후 재푸시
       └── 추가 안 함 → 사용자에게 수동 머지 안내
```

## 절대 규칙

→ **SSOT**: [`../harness-absolute-rules.md`](../harness-absolute-rules.md) — CI 통과 전 머지 금지, 자동 머지 가드 4종 AND, 리뷰 증거 게이트, PR base 자동 감지(fail-closed), feat 브랜치 직행 배포 금지, rp-ship 필수 호출, 비상 탈출구 `RP_SHIP_MANUAL=1`. 본문 중복 금지.

**본 스킬 한정 보조 규칙:**
- **머지 전략**: `--merge` 고정 (squash/rebase 금지)
- **동일 브랜치 재PR**: MERGED/CLOSED된 PR이 있어도 신규 PR 생성 (OPEN PR만 재사용)
- **QA·코드리뷰 이수 확인**: PR 생성 직전 `rp-qa`·`rp-code-review` 완료 상태 체크. 미완이면 ship 중단 후 해당 단계 복귀 (메타 단축 경로는 본 규칙 적용 외)

## 머지 후 검증

- `git log [target-branch] --oneline -3`으로 커밋 반영 확인
- 배포 워크플로우 상태 확인: `gh run list --limit 1`
- 배포 완료 시 라이브 URL 안내

## 완료 조건

- PR 머지 완료
- 배포 워크플로우 성공
- 라이브 URL 사용자에게 전달

## ▶ 자동 전환

배포 완료 즉시 `✓ [11] 배포 완료. 회고가 필요하면 /rp-retro` 출력. 회고는 사용자 명시 명령 시에만 실행 (자동 진입 없음).

→ 배포 상세: [`../harness-ship.md`](../harness-ship.md)

---
# Harness: 산출물 보고 + 배포

본 문서는 **정책·이유** SSOT. 실행 절차(커밋·PR·CI·머지·README 검증·머지 후 검증)는 [`skills/rp-ship.md`](skills/rp-ship.md) SSOT.

## 산출물 보고 → 사용자 승인

QA + 코드리뷰(또는 콘텐츠 검수) 통과 후 사용자에게 결과를 보고한다.

**보고 포맷:**
- QA 결과 요약 (통과/실패 항목)
- 코드리뷰 점수 (7항목 또는 빌드 검증)
- 변경 파일 목록 + 줄 수 변화
- "승인 시 커밋→PR→배포를 진행합니다" 안내

**⛔ 사용자 승인 없이 배포(머지) 진행 금지** — 커밋·PR은 산출물 보고 후 자동 수행

## 커밋 → PR → 배포 (실행 절차)

→ [`skills/rp-ship.md`](skills/rp-ship.md) "절차" 섹션 SSOT (사전 체크 게이트, PR base 결정, 자동 머지 가드 4종 AND, 비상 탈출구 포함)

## CI 정책

| 규칙 | 내용 |
|------|------|
| CI 없을 때 | PR 생성 시점에 "CI가 없습니다. 추가할까요?" 추천 |
| 추가 여부 | 사용자 선택 시에만 추가 |
| CI 없이 머지 | 사용자에게 수동 머지 안내 |
| **동일 브랜치 재작업** | 기존 PR이 MERGED/CLOSED면 **신규 PR 생성** (OPEN PR만 재사용) |

> CI 통과 전 머지 금지 · `--admin`/`--no-verify` 우회 금지 · 자동 머지 가드 4종 AND · 리뷰 증거 게이트 · PR base 자동 감지(fail-closed) → 본문은 [`harness-absolute-rules.md`](harness-absolute-rules.md) SSOT 참조

## 푸시 전 README 검증 (정책)

모든 push 전 README.md 가 프로젝트 현 상태와 일치해야 한다.

| 항목 | 정책 |
|------|------|
| 검증 5항목 | 프로젝트 개요·디렉토리 구조·설치/실행·기능 목록·기술 스택 |
| 통과 기준 | 평균 ≥ 8.0 |
| 실패 시 | README 수정 → 재검증 (최대 3회) → 3회 실패 시 사용자 판단 위임 |
| 미통과 효과 | **push 차단** |

→ 실행 절차: [`skills/rp-ship.md`](skills/rp-ship.md) "README 검증" SSOT

## 머지 직전 PRD 정리 (정책)

가드 4종 AND 통과 후, 머지 실행 전에 **동일 PR**에서 PRD 디렉토리를 통째 삭제한다. Full PRD·간소 PRD 모두 적용. 머지 후 PRD 참조는 **PR 본문 요약 + git history**로만 가능.

| 단계 | 내용 |
|------|------|
| 요약 임베드 | PRD `## 개요·목적` + `## 기능 요구사항` + `## Review 결과` 추출 → PR 본문 `<details>` 블록 |
| 디렉토리 삭제 | `git rm -r <project-root>/docs/prd/[feature]/` (Full + 간소 동일) |
| 정리 커밋 | `chore(prd): merge 직전 PRD 정리` |
| CI 재통과 | 정리 커밋 CI SUCCESS 확인 (가드 b 재실행 면제) |
| 누락 시 | 머지 차단 |

→ 실행 절차: [`skills/rp-ship.md`](skills/rp-ship.md) "PRD 정리" SSOT
→ 절대 규칙: [`harness-absolute-rules.md`](harness-absolute-rules.md) "머지 직전 PRD 정리 필수"

## 머지 후 정책

- 배포 완료 메시지 직후 `/rp-retro` 자동 진입 **금지** ([`harness-absolute-rules.md`](harness-absolute-rules.md): 회고는 사용자 명령 시에만)
- 배포 완료 시 라이브 URL 사용자에게 전달

→ 실행 절차: [`skills/rp-ship.md`](skills/rp-ship.md) "머지 후 검증" SSOT

## 최종 산출물

| 항목 | 기준 |
|------|------|
| 코드 | 실행 가능, 테스트 통과 |
| README | 간결 명료, 링크 참조 방식, **구조 변경 시 동기화 필수** |
| PRD | Open Issues 업데이트 |

→ 스킬 상세: [`skills/rp-ship.md`](skills/rp-ship.md)

---abs---
# Harness 절대 규칙 (SSOT)

본 문서가 **단일 출처(SSOT)**. 다른 문서·스킬에서 동일 규칙 본문 중복 정의 금지, 링크 참조만 허용.
[`../CLAUDE.md`](../CLAUDE.md) 진입점에서 본 파일을 참조.

## ⛔ 적용 범위

예외 없음. "급해서", "간단해서", "나중에" 등 어떤 이유로도 단계 스킵 불가.

## 문서·리뷰 절차

- **[리뷰 단계 서브에이전트 필수]** `/rp-plan-review`·`/rp-eng-review`·`/rp-code-review` Claude 리뷰 적용:
  - 실행 주체: Agent 툴 분리 서브에이전트(`subagent_type=general-purpose`). 메인 셀프 채점 **전면 금지** (이해충돌·관성 편향 방지)
  - 역할 분리: 서브 = Claude 채점, 메인 = `/codex:review` 실행·결과 저장
  - 서브에이전트 프롬프트 4 필수 항목(SSOT): (a) 대상 파일 경로 + 참고 파일, (b) 채점 기준(단계별 항목 수: 4=9, 5=5, 9=7), (c) 독립 판정 지시(메인 컨텍스트 미공유), (d) 역할 경계: 본 단계 채점만 수행 — **Claude 측 적용 시 Codex 실행·저장 금지 / Codex 측 적용 시 `review-claude-*.md` 합성 금지** (`.codex/skills/` 어댑터 노트 동시 참조). **스킬 파일은 본 (a)~(d) 본문 중복 금지, 링크 참조만 허용**
  - Fallback: 기술적 실패 최대 2회 재호출 → 지속 시 사용자 즉시 보고. 셀프 채점 우회 금지
  - 평가 미달: PRD 재작성 후 새 서브에이전트 재실행 (기술 실패와 구분)
  - 증거 저장: `<project-root>/docs/prd/[feature]/review-claude-{plan,eng,code,meta}-r{N}.md`. `{meta}` = 간소 PRD 단일 리뷰 전용. 회차별 새 파일 (덮어쓰기 금지)
- 기능 변경 시 코드 전에 PRD 문서 먼저 업데이트 + 리뷰
- **[4][5][9] Codex 추가 리뷰 필수**: Claude 리뷰와 **1차 병렬 실행** (메인이 Agent 툴 + Bash `codex review`를 동일 메시지에서 동시 호출). Codex 1회만, Claude 미달 시 Claude만 최대 2회 추가 재실행 (총 3회). 통과 조건: Claude 점수 통과 AND Codex High/Critical 반영 완료 (Codex SKIP 시 Claude 점수만으로 판정)
  - 타임아웃: wall-clock 300초
  - cwd: 일반 기능 `repositories/[project]/`, 하네스 메타 변경 `claude-projects/`
  - **실행 종료 후 시작 cwd 복귀 필수** (정상/SKIPPED/중단 무관). `SAVED_CWD=$(pwd)` 캡처 → 종료 직후 `cd "$SAVED_CWD"`. 누락 시 다음 단계 진행 금지
  - 점수화 없음. High/Critical 지적 반영 후 다음 단계 진입
  - 결과 저장: `<project-root>/docs/prd/[feature]/review-codex-{plan,eng,code,meta}.md`
  - 토큰/기능 신호 패턴 명시 출력 시 1회 스킵 + SKIPPED 헤더 + 7항목 증거 저장 ([`harness-codex-review.md`](harness-codex-review.md) "토큰·기능 신호 패턴" SSOT)
  - 그 외 비정상 종료(네트워크·login·플러그인·hang·매칭 0건): 중단 + 사용자 보고. 자동 재시도 금지

## 테스트·QA·게이트

- 테스트 코드 없이 커밋/머지 절대 금지
- 테스트·빌드 미통과 시 다음 단계 진행 금지
- **QA·코드 리뷰 단계 생략 금지 (게이트 강화)**: 두 단계 모두 완료 여부 배포 전 체크. 하나라도 미수행이면 배포 차단. "단위 테스트 통과 = QA 대체 불가"
- **프런트엔드 변경 시 Playwright E2E + axe(접근성) 검사 필수**: `rp-qa` 단계에서 둘 다 실행, 실패 시 진행 차단. E2E 테스트가 없는 UI 태스크는 완료 불가
- **백엔드 변경 시 4-게이트 의무**: [`harness-backend-test-policy.md`](harness-backend-test-policy.md) — (1) 단위 항상 (2) 읽기 endpoint API 응답 테스트 (3) SQL/JPA/마이그레이션 변경 시 `@SpringBootTest+@Transactional` ROLLBACK 통합 테스트 (4) 컨트롤러·DI·Flyway 변경 시 로컬 `bootRun` + 헬스 + Flyway 로그 + OAuth 인증 endpoint 응답 캡처. 미준수 머지 차단
- **push 전 README 검증 필수** (5항목 평균 8.0+, 최대 3회 재시도, 미통과 시 push 차단)

## 배포·머지·브랜치

- 산출물 보고 없이 배포 진행 금지 (커밋·PR은 산출물 보고 후 자동)
- **CI 통과 전 머지 금지** (예외 없음)
- **feat 브랜치 직행 배포 금지**: 모든 배포는 `rp-ship` 경유 (PR → CI → main 머지 → 배포). feat/통합 브랜치 상태로 프로덕션 프로세스 기동·노출 금지. 단, 로컬 개발 서버(`uvicorn --reload`)는 예외
- **`main` 직접 수정 금지**: `main` 브랜치에서 docs·CLAUDE.md·스킬·settings 수정 감지 시 즉시 중단 + feat 브랜치 전환 요구
- **`rp-ship` 스킬 호출 필수**: 커밋·PR·머지·배포는 수동 `git`/`gh` 우회 없이 `rp-ship` 스킬 경유. 단, `rp-ship` 스킬 내부 절차로 명시된 명령은 예외
- **`rp-ship` 자동 머지 가드 4종 AND**: 모두 충족 시에만 자동 머지
  - (a) CI 모든 체크 SUCCESS
  - (b) 리뷰 증거 게이트 통과
  - (c) PR base 정상 감지
  - (d) `gh pr view --json mergeable` = `MERGEABLE`
  - 하나라도 실패 → 중단 + OPEN 유지 + 사용자 보고. `--admin`·`--no-verify` 우회 금지. 비상 탈출구 `RP_SHIP_MANUAL=1` 환경변수만 자동 머지 비활성 허용
- **머지 직전 PRD 정리 필수**: 자동 머지 가드 4종 통과 후 머지 실행 전, 동일 PR 내에서 PRD 디렉토리(`<project-root>/docs/prd/[feature]/`)를 `git rm -r` 통째 삭제 + PR 본문에 요약(개요·기능 요구사항·Review 결과) 임베드 + 정리 커밋 CI 재통과 확인 후 머지. Full PRD·간소 PRD 모두 적용. 가드 (b)는 정리 커밋 직전까지만 유효(정리 후 재실행 면제). 정리 단계 누락 시 머지 차단. 머지 후 PRD 참조는 **PR 본문 요약 + git history**로만 가능
- **`rp-ship` PR base 자동 감지 게이트**: 통합 브랜치 선언 감지 → `--base` 주입
  - 감지 순서: (0) 메타 분기 선검사 (`review-claude-meta-r*.md` 또는 `review-codex-meta.md` 존재 → `--base main`) → (1) `docs/tasks.md` → (2) `CLAUDE.md` → (3) repo default
  - 매칭 정규식: `^[\s\-\*|]*통합 브랜치:\s*`?([A-Za-z0-9/_\-]+)`?` 정확히 1건만 채택
  - Fail-closed: 2건+ 매칭·공백 포함·원격 부재·detached HEAD·프로젝트 루트 미확인 → 중단. 느슨한 `feat/*` 추론 금지
  - 우회 허용: 수동 `--base <X>`, 메타 분기 자동
  - base 리타깃 시 CI 재실행 후 자동 머지 가드 재확인

## 단축 경로·예외

- **하네스 메타 변경 단축 경로**: `rp-init`·`rp-specify`·`rp-task`·`rp-dev` 스킵 + feat 브랜치 + `rp-prd` 간소(변경 이유·영향 파일·롤백·검증 4섹션) + 리뷰 + `rp-ship`. 완전 생략은 금지
- **회고(`/rp-retro`)는 사용자 명시 명령 시에만 실행** — 자동 진입 없음. 필요 시 사용자가 직접 `/rp-retro` 호출
- 워크플로우 위반 발견 시 즉시 중단하고 빠진 단계부터 재진행

exec
/bin/zsh -lc "rtk nl -ba docs/skills/rp-ship.md | sed -n '74,88p'; rtk nl -ba docs/prd/prd-cleanup-on-merge/prd.md | sed -n '1,45p'; rtk nl -ba docs/harness-ship.md | sed -n '38,56p'" in /Users/hose.kim/Claude/workflow-agent-harness
 succeeded in 0ms:
    74	   - 적용: `gh pr edit <num> --body "$(...)"` — 기존 본문 끝에 `<details><summary>PRD 요약</summary>...</details>` 추가
    75	9. **PRD 디렉토리 삭제 커밋**:
    76	   - `git rm -r <project-root>/docs/prd/[feature]/`
    77	   - `git commit -m "chore(prd): merge 직전 PRD 정리"`
    78	   - `git push`
    79	10. **CI 재대기**: 정리 커밋 CI 통과만 확인 (`gh pr checks <num>`). 가드 (b) 재실행 면제 (정리 직전 통과 + 요약 임베드 완료)
    80	
    81	### 머지·배포
    82	11. **머지 실행**: `gh pr merge <num> --merge --delete-branch` (전략 `--merge` 고정)
    83	12. **배포 확인**: 배포 워크플로우 완료 대기 + 결과 보고
    84	
    85	### 비상 탈출구
    86	
    87	환경변수 `RP_SHIP_MANUAL=1` 설정 시 자동 머지 비활성. CI 통과 후 PR URL 보고 → 사용자 승인 대기 → 승인 시 `gh pr merge --merge`. 위험 PR·실험적 변경 시 사용.
    88	
     1	# PRD — 머지 직전 PRD 자동 정리
     2	
     3	| 메타 | 값 |
     4	|------|----|
     5	| 유형 | 하네스 메타 변경 (간소 PRD) |
     6	| 상태 | Draft |
     7	| 통합 브랜치 | `main` (메타 분기 → `--base main` 자동) |
     8	| 작업 브랜치 | `feat/prd-cleanup-on-merge` |
     9	
    10	## 변경 이유
    11	
    12	PRD·리뷰 증거 파일이 `repositories/[project]/docs/prd/[feature]/` 및 `docs/prd/[feature]/`에 누적되어 트리가 비대해진다. 머지 = 기능 출하 시점이므로 PRD는 의사결정 기록으로서 역할이 종료된다. 이 시점에 같은 PR diff 안에서 PRD 디렉토리를 통째 삭제하고, 핵심 요약(개요·기능 요구사항·Review 결과)을 PR 본문에 임베드하여 추후 참조성은 PR + git history로 보존한다.
    13	
    14	**해결 문제:**
    15	- `docs/prd/` 트리 누적 → 탐색·리뷰 노이즈
    16	- 사후 PRD 단일 출처 모호 (개발 중엔 파일, 머지 후엔 git history)
    17	- 회고·문서 검색 시 outdated PRD 혼선
    18	
    19	**대안 비교:**
    20	
    21	| 방안 | 평가 |
    22	|------|------|
    23	| 즉시 삭제 + PR 본문 요약 임베드 (채택) | 트리 항상 깨끗, PR 본문이 사후 조회 anchor |
    24	| `_archive/`로 이동 | 트리는 정리되나 archive 누적 → 동일 문제 재발 |
    25	| 머지 후 별도 PR로 정리 | 자동화 어렵고 누락 위험, PR diff 분리되어 가독성 저하 |
    26	| 상태만 `Shipped` 마킹 | 누적 미해결 |
    27	
    28	## 영향 파일
    29	
    30	| 파일 | 변경 내용 |
    31	|------|----------|
    32	| [`docs/harness-absolute-rules.md`](../../harness-absolute-rules.md) | "배포·머지·브랜치" 섹션에 "머지 직전 PRD 정리 필수" 절대 규칙 1줄 추가 |
    33	| [`docs/skills/rp-ship.md`](../../skills/rp-ship.md) | 자동 머지 가드 통과 후 머지 직전 단계 8(요약 임베드)·9(`git rm -r` 정리 커밋)·10(CI 재대기) 신설, 머지·배포는 11·12로 재번호 |
    34	| [`docs/harness-ship.md`](../../harness-ship.md) | "머지 직전 PRD 정리" 정책 섹션 신설 (단계별 표 + SSOT 링크) |
    35	| `.codex/skills/rp-ship/SKILL.md` | `scripts/sync-codex-skills.py`로 자동 동기화 (수동 편집 아님) |
    36	| `docs/prd/prd-cleanup-on-merge/` | 본 PRD + 리뷰 증거 (머지 직전 본 규칙에 의해 자체 삭제됨 — dogfooding) |
    37	
    38	**적용 범위:** Full PRD (`repositories/[project]/docs/prd/[feature]/`) + 간소 PRD (`docs/prd/[feature]/`) 둘 다.
    39	
    40	**핵심 동작 (`rp-ship` 절차):**
    41	1. 가드 4종 AND 통과 (CI · 리뷰 증거 · base 정상 · MERGEABLE)
    42	2. PRD `## 개요·목적` + `## 기능 요구사항` + `## Review 결과` 추출 → PR 본문 `<details>` 블록 임베드 (`gh pr edit --body`)
    43	3. `git rm -r <project-root>/docs/prd/[feature]/`
    44	4. 정리 커밋 `chore(prd): merge 직전 PRD 정리` + push
    45	5. CI 재통과 확인 (가드 b 재실행 면제)
    38	| 검증 5항목 | 프로젝트 개요·디렉토리 구조·설치/실행·기능 목록·기술 스택 |
    39	| 통과 기준 | 평균 ≥ 8.0 |
    40	| 실패 시 | README 수정 → 재검증 (최대 3회) → 3회 실패 시 사용자 판단 위임 |
    41	| 미통과 효과 | **push 차단** |
    42	
    43	→ 실행 절차: [`skills/rp-ship.md`](skills/rp-ship.md) "README 검증" SSOT
    44	
    45	## 머지 직전 PRD 정리 (정책)
    46	
    47	가드 4종 AND 통과 후, 머지 실행 전에 **동일 PR**에서 PRD 디렉토리를 통째 삭제한다. Full PRD·간소 PRD 모두 적용. 머지 후 PRD 참조는 **PR 본문 요약 + git history**로만 가능.
    48	
    49	| 단계 | 내용 |
    50	|------|------|
    51	| 요약 임베드 | PRD `## 개요·목적` + `## 기능 요구사항` + `## Review 결과` 추출 → PR 본문 `<details>` 블록 |
    52	| 디렉토리 삭제 | `git rm -r <project-root>/docs/prd/[feature]/` (Full + 간소 동일) |
    53	| 정리 커밋 | `chore(prd): merge 직전 PRD 정리` |
    54	| CI 재통과 | 정리 커밋 CI SUCCESS 확인 (가드 b 재실행 면제) |
    55	| 누락 시 | 머지 차단 |
    56	

exec
/bin/zsh -lc "rtk nl -ba docs/skills/rp-ship.md | sed -n '68,75p'" in /Users/hose.kim/Claude/workflow-agent-harness
 succeeded in 0ms:
    68	   - (d) `gh pr view <num> --json mergeable` 가 `MERGEABLE`
    69	   하나라도 실패 → **자동 머지 중단 + PR 상태 OPEN 유지 + 사용자 즉시 보고**. `--admin`·`--no-verify` 우회 금지.
    70	
    71	### PRD 정리 (머지 직전, 동일 PR)
    72	8. **PRD 요약 PR 본문 임베드**:
    73	   - 추출 섹션: `## 개요·목적`, `## 기능 요구사항`, PRD 하단 `## Review 결과`
    74	   - 적용: `gh pr edit <num> --body "$(...)"` — 기존 본문 끝에 `<details><summary>PRD 요약</summary>...</details>` 추가
    75	9. **PRD 디렉토리 삭제 커밋**:

exec
/bin/zsh -lc 'rtk rg "통합 브랜치|task|rp-ship|docs/prd" docs/harness-workflow.md docs/harness-dev.md docs/harness-ship.md docs/skills/rp-dev.md docs/skills/rp-ship.md docs/harness-prd.md -n' in /Users/hose.kim/Claude/workflow-agent-harness
 succeeded in 0ms:
docs/harness-dev.md:7:  └── feat/[project-name]          ← 통합 브랜치
docs/harness-dev.md:16:| 통합 브랜치 | `feat/[project-name]`, main에서 생성 |
docs/harness-dev.md:17:| 태스크 브랜치 | `feat/T-XX-description`, 통합 브랜치에서 생성 |
docs/harness-dev.md:18:| 태스크 머지 | 태스크 → 통합 브랜치로 머지 |
docs/harness-dev.md:19:| 최종 머지 | 통합 브랜치 → main (PR) |
docs/harness-dev.md:44:- 태스크 목록 파일: `repositories/[project]/docs/tasks.md`
docs/harness-dev.md:62:브랜치 생성 → 개발(1태스크/에이전트) → 빌드+테스트 검증 → 유닛테스트 작성+실행 → 통합 브랜치 머지+검증
docs/harness-dev.md:83:- [ ] 통합 브랜치 머지 + git log 검증
docs/skills/rp-ship.md:6:# rp-ship
docs/skills/rp-ship.md:13:- `/rp-ship` 명령
docs/skills/rp-ship.md:19:커밋 직전 `<project-root>/docs/prd/[feature]/` 디렉터리에서 리뷰 증거 파일 존재 검증. 누락 시 ship **중단** + 해당 리뷰 단계로 복귀.
docs/skills/rp-ship.md:31:FEATURE=docs/prd/[feature]
docs/skills/rp-ship.md:76:   - `git rm -r <project-root>/docs/prd/[feature]/`
docs/skills/rp-ship.md:91:PR 생성/리타깃 전 **통합 브랜치 선언을 감지**해 `--base` 에 주입. `gh pr create` 기본값(레포 default branch)에 의존 금지.
docs/skills/rp-ship.md:98:| 1 | 프로젝트 `docs/tasks.md` | 라인 시작 앵커 정규식 `^[\s\-\*|]*통합 브랜치:\s*`?([A-Za-z0-9/_\-]+)`?` 로 **정확히 1건** 매칭 | 2건 이상 → fail-closed / 0건 → 다음 소스 |
docs/skills/rp-ship.md:102:**예시 (museum-finder `docs/tasks.md`)**: `- 통합 브랜치: \`feat/mvp-v1\` · 태스크 브랜치 \`feat/T-NN-{slug}\`` → `feat/mvp-v1` 1건 매칭 → `--base feat/mvp-v1`.
docs/skills/rp-ship.md:106:사용자가 `rp-ship --base <X>` 로 명시 전달 시 자동 감지 **비활성화**하고 `<X>` 사용. 감지 로직을 거치지 않는 유일한 우회 경로.
docs/skills/rp-ship.md:115:- 느슨한 브랜치명 추론(`feat/mvp-*` 등) 금지 — 반드시 `통합 브랜치:` 앵커 필드만 권위
docs/skills/rp-ship.md:146:→ **SSOT**: [`../harness-absolute-rules.md`](../harness-absolute-rules.md) — CI 통과 전 머지 금지, 자동 머지 가드 4종 AND, 리뷰 증거 게이트, PR base 자동 감지(fail-closed), feat 브랜치 직행 배포 금지, rp-ship 필수 호출, 비상 탈출구 `RP_SHIP_MANUAL=1`. 본문 중복 금지.
docs/harness-prd.md:75:repositories/[project]/docs/prd/YYYYMMDD_HHMMSS_[feature]_[8자리랜덤].md
docs/harness-prd.md:91:**파일명:** `docs/prd/[feature]/prd.md` (하네스 루트 기준)
docs/harness-workflow.md:18:▶ [6] 태스크 분해                   → /rp-task
docs/harness-workflow.md:28:▶ [11] 커밋 → PR → CI → **자동 머지(가드 4종 AND)** → 배포 → /rp-ship
docs/harness-workflow.md:44:| 6 | 태스크 분해 | [`skills/rp-task.md`](skills/rp-task.md) |
docs/harness-workflow.md:48:| 11 | 커밋·PR·머지·배포 | [`skills/rp-ship.md`](skills/rp-ship.md) |
docs/harness-workflow.md:77:**⛔ 절대 규칙 SSOT:** [`harness-absolute-rules.md`](harness-absolute-rules.md) — QA/코드리뷰 생략 불가, 자동 머지 가드 4종 AND, 메타 단축 경로, rp-ship 필수 호출, main 직접 수정 금지 등 본문은 SSOT 참조
docs/harness-workflow.md:118:| PRD 디렉토리 | `repositories/[project]/docs/prd/` | |
docs/skills/rp-dev.md:2:description: '[7] 개발. 태스크별 반복 사이클. 통합 브랜치에서 태스크 단위 구현·테스트'
docs/skills/rp-dev.md:17:1. 통합 브랜치에서 태스크 브랜치 생성
docs/skills/rp-dev.md:21:5. 통합 브랜치로 머지 + `git log` 검증
docs/skills/rp-dev.md:42:- [ ] 통합 브랜치 머지 + git log 검증
docs/harness-ship.md:3:본 문서는 **정책·이유** SSOT. 실행 절차(커밋·PR·CI·머지·README 검증·머지 후 검증)는 [`skills/rp-ship.md`](skills/rp-ship.md) SSOT.
docs/harness-ship.md:19:→ [`skills/rp-ship.md`](skills/rp-ship.md) "절차" 섹션 SSOT (사전 체크 게이트, PR base 결정, 자동 머지 가드 4종 AND, 비상 탈출구 포함)
docs/harness-ship.md:43:→ 실행 절차: [`skills/rp-ship.md`](skills/rp-ship.md) "README 검증" SSOT
docs/harness-ship.md:52:| 디렉토리 삭제 | `git rm -r <project-root>/docs/prd/[feature]/` (Full + 간소 동일) |
docs/harness-ship.md:57:→ 실행 절차: [`skills/rp-ship.md`](skills/rp-ship.md) "PRD 정리" SSOT
docs/harness-ship.md:65:→ 실행 절차: [`skills/rp-ship.md`](skills/rp-ship.md) "머지 후 검증" SSOT
docs/harness-ship.md:75:→ 스킬 상세: [`skills/rp-ship.md`](skills/rp-ship.md)

codex
The new merge-time cleanup procedure is internally inconsistent for simplified PRDs, which the policy explicitly includes and this PR uses. That makes the documented ship flow unreliable for meta changes until the extraction rule is adjusted.

Review comment:

- [P2] Handle simplified PRDs without Full PRD headings — /Users/hose.kim/Claude/workflow-agent-harness/docs/skills/rp-ship.md:73-73
  This cleanup step applies to both Full and simplified PRDs, but it requires extracting `## 개요·목적` and `## 기능 요구사항`. Simplified/meta PRDs in this repo, including the new dogfood PRD, use sections such as `## 변경 이유`, `## 영향 파일`, and `## 검증` instead, so the first simplified PRD ship will either embed an incomplete summary or fail the extraction just before deleting the PRD directory. Please either define fallback sections for simplified PRDs or make those headings mandatory in the simplified PRD template.
The new merge-time cleanup procedure is internally inconsistent for simplified PRDs, which the policy explicitly includes and this PR uses. That makes the documented ship flow unreliable for meta changes until the extraction rule is adjusted.

Review comment:

- [P2] Handle simplified PRDs without Full PRD headings — /Users/hose.kim/Claude/workflow-agent-harness/docs/skills/rp-ship.md:73-73
  This cleanup step applies to both Full and simplified PRDs, but it requires extracting `## 개요·목적` and `## 기능 요구사항`. Simplified/meta PRDs in this repo, including the new dogfood PRD, use sections such as `## 변경 이유`, `## 영향 파일`, and `## 검증` instead, so the first simplified PRD ship will either embed an incomplete summary or fail the extraction just before deleting the PRD directory. Please either define fallback sections for simplified PRDs or make those headings mandatory in the simplified PRD template.

## 반영

| Codex 지적 | 등급 | 반영 |
|-----------|:----:|------|
| [P2] Handle simplified PRDs without Full PRD headings (rp-ship.md:73) — 간소 PRD는 `## 변경 이유`/`## 영향 파일`/`## 검증` 섹션으로 구성되어 추출 매핑 불일치 | P2 (참고용 등급, dogfood 자체 영향으로 자발 반영) | rp-ship.md 단계 8에 PRD 유형별 추출 매핑 추가 (Full / 간소 분리). 추출 누락·`gh pr edit` 1회 재시도 실패 시 ship 중단 분기 신설. harness-ship.md 정책 표 + PRD 핵심 동작 동기화 |
