---
description: '[4] 기획 리뷰. Claude 9항목 점수제 + Codex /codex:review 1회. AND 진행'
argument-hint: '[대상 PRD 경로]'
---

# rp-plan-review

기획 리뷰. PRD의 기능 완전성과 사용자 관점 적합성 검증. **메인 셀프 채점 절대 금지** — 해당 런타임 서브에이전트가 **기획 관점**으로 채점.

## 작성 모드

| 모드 | 채점 주체 | 외부 추가 리뷰 |
|------|---------|-------------|
| Claude-led (Claude Code) | Claude Agent 툴 서브에이전트 | Codex 1회 병렬 |
| Codex-led (Codex CLI) | Codex `spawn_agent` 서브에이전트 | **N/A** — 메인=Codex이므로 외부 추가 리뷰 없음 |

SSOT: [`../harness-absolute-rules.md`](../harness-absolute-rules.md) "작성 모드 및 리뷰 매트릭스".

## 트리거

- PRD 작성 완료 후
- `/rp-plan-review` 명령

## 절차 (양 모드 공통)

> Claude-led는 단계 1 (a)·(b) 모두 발사 / 매트릭스 판정. Codex-led는 (b) Bash `codex review` 생략 / 판정 = 서브에이전트 결과만. 다른 단계는 동일.

1. **1차 발사**: Claude-led는 메인이 동일 메시지에서 두 tool_use 동시 호출. Codex-led는 (a)만 호출
   - (a) **서브에이전트**(Claude-led=Agent 툴 `subagent_type=general-purpose` / Codex-led=`spawn_agent`): 메인 셀프 채점 **금지**. 프롬프트 4 필수 항목 (a)~(d) → SSOT: [`../harness-absolute-rules.md`](../harness-absolute-rules.md) "[리뷰 단계 서브에이전트 필수]" 절. 본 단계 적용값: (a) PRD 경로 (b) 9항목 (c)·(d) SSOT 그대로
   - (b) **[Claude-led 전용] Bash `codex review`**: 사전 `SAVED_CWD=$(pwd)` 캡처 + PRD 루트로 `cd` (일반 기능 `repositories/[project]/`, 메타 `workflow-agent-harness/`). 300초 타임아웃. 종료 후 `cd "$SAVED_CWD"`. Codex-led는 본 항목 생략 (메인이 이미 Codex)
2. **판정**: Claude-led는 SSOT [`../harness-codex-review.md`](../harness-codex-review.md) "1차 결과 매트릭스" 적용. Codex-led는 서브에이전트 결과만으로 판정
3. **결과 처리**: 별도 파일 저장 없음. 서브에이전트 점수·지적과 (Claude-led 시) Codex High/Critical은 인-메모리에서 PRD 본문에 반영
4. **통과**: 서브에이전트 평균 ≥8.0 + 각 항목 ≥7. Claude-led는 추가로 Codex High/Critical 반영 완료 (Codex SKIP 시 서브에이전트 점수만으로) → 다음 단계
5. **미달**: 지적 반영 (Claude-led=Codex High/Critical+서브에이전트 통합, Codex-led=서브에이전트만) → PRD 본문 갱신 → 새 서브에이전트로 재실행 (Claude-led의 외부 Codex 재호출 금지). 최대 2회 추가, **총 3회**. 3회 미달 시 자동 중단 + 사용자 결정 요청 (강행/재설계/중단)
6. **기술 실패 Fallback**: SSOT [`../harness-absolute-rules.md`](../harness-absolute-rules.md) "재시도 한도" 적용. 메인 셀프 채점 우회 금지
7. **[Claude-led 전용] Codex 비-스킵 비정상 종료**(네트워크·hang·매칭 0건): 워크플로우 중단 + 사용자 보고

## 평가 항목

| 항목 | 설명 |
|------|------|
| 문제 정의 | 해결하려는 문제가 명확하고 관찰 가능한 근거가 있는가 |
| 사용자 가치 | 대상 사용자에게 실질적 가치가 있는가 |
| 기능 완전성 | 누락된 기능이 없는가 |
| 우선순위 | 기능 간 우선순위가 합리적인가 |
| 실현 가능성 | 주어진 조건으로 구현 가능한가 |
| 경계 명확성 | 스코프/비-스코프가 분명하고 포함·비포함 판단이 가능한가 |
| 분기 충분성 | 예외·에러·빈 상태·대안 흐름이 고려되어 있는가 |
| 사용자 검증 게이트 | 실사용자 반응 검증 방법이 있는가 (프로토타입 / 인터뷰 / A·B 테스트 / 내부 시범 사용 중 명시) |
| 대안 탐색 | 대안 A/B/C/D 중 선택 근거 + 판단 근거 유형(창업자 직감 / 엔지니어 선호 / 제품 가설)이 명시되어 있는가 |

## 판정

- 평균 >= 8.0 + 각 항목 >= 7 → 통과
- 평균 미달 또는 항목별 최저 < 7 → Doc Agent가 PRD 본문 갱신 후 재실행 (인-메모리 최대 3회)
- 3회 미달 → 자동 중단 + **사용자 결정 요청** (강행/재설계/중단)

## ▶ 자동 전환

- Claude-led: Claude 서브에이전트 통과 + (Codex High/Critical 반영 완료 OR Codex 토큰/기능 SKIPPED) 시 `✓ [4] 기획 리뷰 통과 (Claude+Codex)` (스킵 시 `(Claude+Codex SKIPPED)`) 출력 후 **`/rp-eng-review` 자동 진입**.
- Codex-led: Codex 서브에이전트 통과 시 `✓ [4] 기획 리뷰 통과 (Codex-led)` 출력 후 **`/rp-eng-review` 자동 진입** (외부 Codex 추가 리뷰 N/A).

→ PRD 상세: [`../harness-prd.md`](../harness-prd.md)
→ Codex 리뷰 규칙: [`../harness-codex-review.md`](../harness-codex-review.md)
