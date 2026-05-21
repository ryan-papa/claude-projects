# PRD: rp-ship 가드 (a) 동기 검증 의무화 + `--auto` 위임 금지

**ID:** 20260521_130000_rp-ship-guard-a-strict_1896631f
**유형:** 하네스 메타 변경
**작성일:** 2026-05-21

## 변경 이유

2026-05-21 PR #274 (museum-finder 라이브 핫픽스) 가 rp-ship 절차로 진행되었으나 **PR Check 의 backend/frontend job 이 pending 상태에서 머지됨**. 원인 3중:

1. **`gh pr merge --auto` 동작 오인**: `--auto` 는 GitHub branch protection 의 required status checks 통과 시 머지로, rp-ship 가드 (a) "CI 모든 체크 SUCCESS" 를 보장하지 않고 인프라 정책에 종속
2. **Branch protection required checks 결손**: paths-filter / version-guard 만 required, backend / frontend / build 는 not-required → paths-filter + version-guard pass 시점 (10초) 에 즉시 머지 트리거
3. **rp-ship 절차의 `--auto` 위임 묵시**: 절차 99줄 "CI 있음 → gh pr merge --merge --auto" 가 가드 (a) 의 코드 강제 검증을 GitHub 에 위임

코드 강제 검증 부재로 인프라 결손 시 가드 무력화. 본 PR 은 절차 측 결손 (#3) 만 메움. 인프라 측 결손 (#2) 는 별도 운영자 작업.

## 영향 파일

| 파일 | 변경 |
|---|---|
| `docs/skills/rp-ship.md` | 절차 7번 가드 (a) 명시화 (`gh pr checks <num> --watch` 종결 대기 + exit 0 + pending/queued/in_progress/fail 0건 검증) + 절차 8번 `--auto` 금지 명시 + "CI 분기" 도식 갱신 + 금지 패턴 섹션 추가 |
| `docs/harness-absolute-rules.md` | 가드 (a) SSOT 문구 동기 — 동기 검증 의무 + `--auto` 위임 금지 + CI 없음 시 N/A + 우회 금지 옵션에 `--auto` 추가 |
| `.codex/skills/rp-ship/SKILL.md` | `sync-codex-skills.py --install-user` 자동 갱신 (Codex 변환본 동기화) |

repositories/ 산하 0 변경. 코드 동작 영향 0.

## 롤백 전략

본 PR 자체 revert. 절차가 기존 (`--auto` 위임) 으로 복귀하지만 라이브 영향 없음 — 절차 정의 변경일 뿐. 단, branch protection 인프라 작업 (별도 트랙) 이 본 PR 이전에 완료되면 `--auto` 도 안전. 본 PR 이후에 인프라 작업이 완료되면 본 절차 + 인프라 둘 다 가드 — 더 안전. revert 우선순위 낮음.

## 검증

| 게이트 | 결과 |
|---|---|
| `rg "gh pr merge --auto" docs/skills/rp-ship.md docs/harness-absolute-rules.md` | 매칭 0건 (금지 패턴 섹션의 인용 1건 제외) — 정책 일관성 확인 |
| `sync-codex-skills.py --check` | "Codex skills in sync: 13" 통과 |
| **본 PR 자체 dogfooding** | 본 PR 머지 시 새 가드 (a) 동기 검증 절차로 ship — `gh pr checks <num> --watch` 실제 호출 → 모든 체크 SUCCESS 확인 후 머지. 절차가 self-validating |
| 향후 회귀 차단 | 다음 PR 부터 `--auto` 단독 호출 시 코드 리뷰 [9] 자동 차단 (rg 가드) |

## 리뷰 반영 (Codex High)

Codex 가 본 PR 의 "CI 없음 → (a) N/A 분기" 가 절대 규칙 41줄 "CI 통과 전 머지 금지 (예외 없음)" 와 충돌한다고 지적 (1건 High). 정합성 메우기 위해 절대 규칙 41줄 자체를 정밀화: "워크플로우 1건 이상 존재 시 통과 전 머지 금지. 워크플로우 부재 시 본 조항 N/A — 가드 (a) N/A 분기와 정합". 사용자 결정 ("CI 없으면 통과") 반영 + 절대 규칙 정합성 유지.

Claude 서브에이전트 리뷰: 최저점 8/10, High/Critical 없음, 통과.

## 추가 작업 (별도 PR / 운영자)

- 각 프로젝트 레포 main branch protection 의 required status checks 에 `backend`, `frontend`, `build` 추가 (museum-finder 등). 인프라 측 결손 동시 메움
- rp-retro 회고 시 본 PR + PR #274 묶음 r1 분석
