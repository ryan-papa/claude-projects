# PRD: Cross-runtime workflow drift fix

**유형:** 하네스 메타 변경
**브랜치:** feat/cross-runtime-drift-fix
**작성일:** 2026-05-22

## 변경 이유

Claude-led와 Codex-led가 같은 하네스 절차를 따르도록 작성 모드 매트릭스를 추가했지만, 일부 보조 문서와 오케스트레이터 표현은 과거 규칙을 남긴다.

| 드리프트 | 영향 |
|---|---|
| 산출물 보고 뒤 사용자 승인 문구 잔존 | 자동 머지 가드 통과 후 머지까지 진행하는 `rp-ship` 절차와 충돌 |
| PRD 리뷰 뒤 사용자 최종 확인 문구 잔존 | engineering review 통과 후 자동 task/dev 진입 계약과 충돌 |
| `rp-workflow` 멈춤 지점 표현 불일치 | 단계 [10] 보고와 단계 [11] ship 중 어디서 멈추는지 런타임별 해석 차이 |
| Lite track 예외가 SSOT 리뷰 매트릭스에 연결되지 않음 | 핫픽스의 통합 plan/engineering 리뷰가 Full flow 분리 리뷰 규칙과 충돌 |
| `rp-amend`가 Lite track을 제외만 함 | 기존 프로젝트 핫픽스 요청의 시작 절차가 문서상 약함 |
| 리뷰 서브에이전트 호출 시 사용자 확인 여부 미명시 | Codex-led review gate 진입에서 불필요한 사용자 round-trip 발생 가능 |

이번 변경은 자동 머지와 Lite track을 유지한다. 보조 문서를 현재 SSOT 의도에 맞추고, Lite track이 Full flow와 다른 명시적 예외임을 양 런타임에서 읽히게 한다. 리뷰 단계 서브에이전트 호출은 하네스 단계 실행 자체이며 추가 사용자 승인 지점이 아님도 명시한다.

### 대안 탐색

| 대안 | 요약 | 장점 | 단점 | 결과 |
|---|---|---|---|---|
| A | 보조 문서의 승인 문구와 Lite 라우팅만 수정 | 변경 폭 최소 | SSOT와 Codex adapter 충돌이 남음 | 기각 |
| B | SSOT에 자동 review subagent 호출과 Lite 예외를 연결하고 adapter 문구까지 정합 | 양 런타임 공통 권위와 진입 문구 동시 정리 | 문서 여러 곳 동기화 필요 | 채택 |
| C | Lite 전용 `rp-hotfix` 스킬 신설 | 진입점이 명시적 | 현재 Lite 정의를 확대하고 새 스킬 유지 비용 추가 | 보류 |
| D | Lite track을 Full flow로 흡수 | 리뷰 매트릭스 단순 | 사용자가 유지하라고 확인한 경량 트랙 제거 | 기각 |

**판단 근거 유형:** 사용자 의도 + 운영 정합성.
**선택 근거:** 자동 머지와 Lite track은 기존 설계로 유지하고, 현재 drift의 권위 위치와 양 런타임 adapter 해석만 닫는다.

### 수용 시나리오

| 시나리오 | 기대 동작 |
|---|---|
| Codex-led Full flow review gate 진입 | review subagent를 별도 사용자 확인 없이 단계 절차로 호출 |
| Full flow engineering review 통과 | 사용자 최종 확인 대기 없이 다음 단계로 자동 진입 |
| Claude-led 또는 Codex-led 산출물 보고 완료 | `rp-ship`이 기본 경로로 시작하고 자동 머지 가드 충족 시 머지까지 진행 |
| 기존 프로젝트 단일 핫픽스가 Lite 판별 통과 | `rp-amend`가 Full amend 대신 Lite track 진입을 안내하고 통합 plan/engineering 리뷰 예외를 적용 |
| Lite 판별 실패 | Full amend flow의 분리 plan/engineering 리뷰로 전환 |
| CI 워크플로우 없는 ship에서 CI 추가 거절 | CI 가드 (a)를 N/A로 두고 base·MERGEABLE 가드 통과 시 자동 머지 |
| 하네스 메타 변경 | 작성 모드 리뷰 매트릭스를 유지하고 Codex-led에서 Claude+Codex 병렬 리뷰로 되돌아가지 않음 |

## 영향 파일

| 파일 | 변경 |
|---|---|
| `docs/harness-absolute-rules.md` | Lite track이 Full flow 리뷰 단계 분리 규칙의 명시적 예외임을 연결. review subagent 호출은 리뷰 단계 자동 실행이며 추가 사용자 승인 지점이 아님을 SSOT에 명시. 메타 단축 경로 리뷰 범위는 작성 모드 매트릭스로 정합 |
| `docs/harness-workflow.md` | Lite track 진입 경로와 자동 머지 흐름을 현재 절차와 맞춤 |
| `docs/harness-prd.md` | engineering review 뒤 자동 전환 계약으로 사용자 최종 확인 문구 정합 |
| `docs/harness-ship.md` | 산출물 보고 뒤 자동 ship/자동 머지 정책으로 승인 문구 정합 |
| `README.md` | 공개 진입 문서의 ship 멈춤 지점 설명을 자동 머지 정책으로 정합 |
| `CLAUDE.md` | Claude 진입 문서의 `rp-amend` 요약을 Full/Lite 분기로 정합 |
| `docs/skills/rp-workflow.md` | 멈춤 지점과 ship 설명을 자동 머지 정책으로 정리 |
| `docs/skills/rp-amend.md` | 기존 프로젝트 핫픽스가 Lite track으로 라우팅되는 절차와 진입 메타데이터 명시 |
| `docs/skills/rp-code-review.md` | 산출물 보고 뒤 ship 자동 진입 문구를 자동 머지 정책으로 정합 |
| `docs/skills/rp-ship.md` | CI 없는 ship도 SSOT의 N/A 가드 분기로 자동 머지 절차 정합 |
| `AGENTS.md` | Codex-led 리뷰 서브에이전트 호출을 단계 자동 실행으로 명시. Full flow 분리 리뷰 규칙이 Lite 통합 리뷰 예외와 충돌하지 않도록 정합 |
| `.codex/skills/rp-*/SKILL.md` | skill 원본 변경 후 변환본 재생성 |

## 롤백 전략

본 PR을 revert한다. 자동 머지와 Lite track 자체 구현은 바뀌지 않으므로 롤백 영향은 문서 라우팅 설명 복귀에 한정된다.

## 검증

| 항목 | 방법 | 기준 |
|---|---|---|
| 자동 머지 문구 정합 | `rtk rg -n "승인|자동 머지|산출물 보고" docs/harness-ship.md docs/skills/rp-workflow.md docs/harness-workflow.md` | ship 기본 경로가 사용자 승인 대기라고 남지 않음 |
| Lite track 연결 | `rtk rg -n "Lite|핫픽스" AGENTS.md docs/harness-absolute-rules.md docs/harness-workflow.md docs/skills/rp-amend.md` | SSOT 예외와 amend 라우팅이 함께 보이고 AGENTS가 Lite 예외를 보존 |
| 리뷰 자동 호출 | `rtk rg -n "추가 사용자 승인|자동 호출|사용자 확인" AGENTS.md docs/harness-absolute-rules.md` | review subagent 호출이 별도 승인 지점이 아님을 SSOT와 Codex adapter에 명시 |
| 충돌 문구 제거 | `rtk rg -n "산출물 보고.*사용자 승인|승인 시 커밋|배포 \\[11\\].*사용자 승인|머지 승인 대기" README.md docs/harness-ship.md docs/skills/rp-workflow.md docs/skills/rp-amend.md docs/skills/rp-code-review.md` | 기본 ship 경로를 승인 대기로 표현하는 문구 0건 |
| PRD 자동 전환 정합 | `rtk rg -n "사용자 최종 확인|Approved.*바로 개발 진입" docs/harness-prd.md` | engineering review 뒤 사용자 승인 대기 문구 0건, 자동 전환 문구 유지 |
| no-CI ship 정합 | `rtk rg -n "CI 없이 머지|CI 없음" docs/harness-ship.md docs/skills/rp-ship.md` | no-CI 경로가 수동 머지 대기가 아니라 CI 가드 (a) N/A 자동 머지 분기를 설명 |
| 메타/Lite 리뷰 정합 | `rtk rg -n "Claude\\+Codex 병렬|^## PRD 리뷰 \\(2단계 순차 실행\\)|회고.*필수 유지" AGENTS.md docs/harness-absolute-rules.md docs/harness-prd.md docs/harness-workflow.md` | 작성 모드 매트릭스와 Lite 예외에 반대되는 고정 표현 0건 |
| 분기 내부 시범 사용 | 대표 요청 4건을 수정 문서 기준으로 재판단: Codex-led Full review, 산출물 보고 뒤 ship, Lite 판별 통과, Lite 판별 실패 | 수용 시나리오 표의 기대 동작과 일치 |
| Codex skill 동기화 | `rtk python3 scripts/sync-codex-skills.py --check` | 변환본 sync 통과 |
