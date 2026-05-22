---
description: '[5] 엔지 리뷰. Claude 5항목 점수제 + Codex /codex:review 1회. AND 진행'
argument-hint: '[대상 PRD 경로]'
---

# rp-eng-review

엔지니어링 리뷰. PRD의 기술적 실현 가능성과 아키텍처 적합성 검증. **메인 셀프 채점 절대 금지** — 해당 런타임 서브에이전트가 **기술 관점**으로 채점.

## 작성 모드

| 모드 | 채점 주체 | 외부 추가 리뷰 |
|------|---------|-------------|
| Claude-led (Claude Code) | Claude Agent 툴 서브에이전트 | Codex 1회 병렬 |
| Codex-led (Codex CLI) | Codex `spawn_agent` 서브에이전트 | 없음 |

SSOT: [`../harness-absolute-rules.md`](../harness-absolute-rules.md) "작성 모드 및 리뷰 매트릭스".

## 트리거

- 기획 리뷰 통과 후
- `/rp-eng-review` 명령

## 절차 (양 모드 공통)

> Claude-led는 단계 1 (a)·(b) 모두 발사 / 매트릭스 판정. Codex-led는 (b) Bash `codex review` 생략 / 판정 = 서브에이전트 결과만. 다른 단계는 동일.

1. **1차 발사**: Claude-led는 메인이 동일 메시지에서 두 tool_use 동시 호출. Codex-led는 (a)만 호출
   - (a) **서브에이전트**(Claude-led=Agent 툴 `subagent_type=general-purpose` / Codex-led=`spawn_agent`): 메인 셀프 채점 **금지**. 프롬프트 4 필수 항목 (a)~(d) → SSOT: [`../harness-absolute-rules.md`](../harness-absolute-rules.md) "[리뷰 단계 서브에이전트 필수]" 절. 본 단계 적용값: (a) PRD + CLAUDE.md·harness-db.md 참고 (b) 5항목 (c)·(d) SSOT 그대로
   - (b) **[Claude-led 전용] Bash `codex review`**: 사전 `SAVED_CWD=$(pwd)` 캡처 + PRD 루트로 `cd`. 300초 타임아웃. 종료 후 `cd "$SAVED_CWD"`. Codex-led는 본 항목 생략
2. **판정**: Claude-led는 SSOT [`../harness-codex-review.md`](../harness-codex-review.md) "1차 결과 매트릭스" 적용. Codex-led는 서브에이전트 결과만으로 판정
3. **결과 처리**: 별도 파일 저장 없음. 서브에이전트 점수·지적과 (Claude-led 시) Codex High/Critical은 인-메모리에서 PRD 본문에 반영
4. **통과**: 서브에이전트 평균 ≥8.0 + 각 항목 ≥7. Claude-led는 추가로 Codex High/Critical 반영 완료 → 다음 단계
5. **미달**: 지적 반영 (Claude-led=Codex High/Critical+서브에이전트 통합, Codex-led=서브에이전트만) → PRD 본문 갱신 → 새 서브에이전트로 재실행 (Claude-led의 외부 Codex 재호출 금지). 최대 2회 추가, **총 3회**. 3회 미달 시 자동 중단 + 사용자 결정 요청
6. **기술 실패 Fallback**: 동일 SSOT 적용 (서브에이전트 오류 시 최대 2회 재호출, 지속 실패 시 사용자 보고)
7. **[Claude-led 전용] Codex 비-스킵 비정상 종료**: 워크플로우 중단 + 사용자 보고

## 평가 항목

| 항목 | 설명 |
|------|------|
| 아키텍처 | 기술 스택, 구조 적합성 |
| 확장성 | 트래픽/데이터 증가 대응 |
| 보안 | 인증, 데이터 보호, 취약점 |
| 성능 | 응답 시간, 리소스 효율 |
| 운영성 | 배포, 모니터링, 장애 대응 |

## 판정

- 평균 >= 8.0 + 각 항목 >= 7 → 통과
- 평균 미달 또는 항목별 최저 < 7 → Doc Agent가 PRD 본문 갱신 후 재실행 (인-메모리 최대 3회)
- 3회 미달 → 자동 중단 + **사용자 결정 요청** (강행/재설계/중단)

## ▶ 자동 전환

- Claude-led: Claude 서브에이전트 통과 + (Codex High/Critical 반영 완료 OR Codex 토큰/기능 SKIPPED) 시 `✓ [5] 엔지니어링 리뷰 통과 (Claude+Codex)` (스킵 시 `(Claude+Codex SKIPPED)`) 출력 후 **`/rp-task` 자동 진입**.
- Codex-led: Codex 서브에이전트 통과 시 `✓ [5] 엔지니어링 리뷰 통과 (Codex-led)` 출력 후 **`/rp-task` 자동 진입** (외부 Codex 추가 리뷰 N/A).

→ PRD 상세: [`../harness-prd.md`](../harness-prd.md)
→ Codex 리뷰 규칙: [`../harness-codex-review.md`](../harness-codex-review.md)
