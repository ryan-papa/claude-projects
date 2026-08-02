---
description: '[5] 엔지 리뷰. 진입 런타임 서브에이전트가 5항목 채점. 교차 런타임 리뷰 없음'
argument-hint: '[대상 PRD 경로]'
---

# rp-eng-review

엔지니어링 리뷰. PRD의 기술적 실현 가능성과 아키텍처 적합성 검증. **메인 셀프 채점 절대 금지** — 해당 런타임 서브에이전트가 **기술 관점**으로 채점.

## 작성 모드

| 모드 | 채점 주체 | 외부 추가 리뷰 |
|------|---------|-------------|
| Claude-Lead (Claude Code) | Claude Agent 툴 서브에이전트 | **없음** — Codex 호출 금지 |
| Codex-Lead (Codex CLI) | Codex `spawn_agent` 서브에이전트 | **없음** — Claude 호출 금지 |

**런타임 = 리뷰어.** 교차 런타임 추가 리뷰 전면 금지. SSOT: [`../harness-absolute-rules.md`](../harness-absolute-rules.md) "작성 모드 및 리뷰 매트릭스".

## 트리거

- 기획 리뷰 통과 후
- `/rp-eng-review` 명령

## 절차 (양 모드 공통)

1. **서브에이전트 발사** (Claude-Lead=Agent 툴 `subagent_type=general-purpose` / Codex-Lead=`spawn_agent`): 메인 셀프 채점 **금지**. 프롬프트 4 필수 항목 (a)~(d) → SSOT: [`../harness-absolute-rules.md`](../harness-absolute-rules.md) "[리뷰 단계 서브에이전트 필수]" 절. 본 단계 적용값: (a) PRD + CLAUDE.md·harness-db.md 참고 (b) 5항목 (c)·(d) SSOT 그대로
2. **판정**: 서브에이전트 결과만으로 판정
3. **결과 처리**: 별도 파일 저장 없음. 점수·지적은 인-메모리에서 PRD 본문에 반영
4. **통과**: 평균 ≥8.0 + 각 항목 ≥7 → 다음 단계
5. **미달**: 지적 반영 → PRD 본문 갱신 → 새 서브에이전트로 재실행. 최대 2회 추가, **총 3회**. 3회 미달 시 자동 중단 + 사용자 결정 요청
6. **기술 실패 Fallback**: SSOT [`../harness-absolute-rules.md`](../harness-absolute-rules.md) "재시도 한도" 적용

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

서브에이전트 통과 시 `✓ [5] 엔지니어링 리뷰 통과 (Claude-Lead)` 또는 `✓ [5] 엔지니어링 리뷰 통과 (Codex-Lead)` 출력 후 **`/rp-task` 자동 진입**.

→ PRD 상세: [`../harness-prd.md`](../harness-prd.md)
