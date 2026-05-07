---
description: '[5] 엔지 리뷰. Claude 5항목 점수제 + Codex /codex:review 1회. AND 진행'
argument-hint: '[대상 PRD 경로]'
---

# rp-eng-review

엔지니어링 리뷰. PRD의 기술적 실현 가능성과 아키텍처 적합성 검증.

## 트리거

- 기획 리뷰 통과 후
- `/rp-eng-review` 명령

## 절차

1. **1차 병렬 발사**: 메인이 동일 메시지에서 두 tool_use 동시 호출
   - **Agent 툴**(Claude 서브에이전트, `subagent_type=general-purpose`): 메인 셀프 채점 **금지**. 프롬프트 4 필수 항목 (a)~(d) → SSOT: [`../harness-absolute-rules.md`](../harness-absolute-rules.md) "[리뷰 단계 서브에이전트 필수]" 절. 본 단계 적용값: (a) PRD + CLAUDE.md·harness-db.md 참고 (b) 5항목 (c)·(d) SSOT 그대로
   - **Bash `codex review`**: 사전 `SAVED_CWD=$(pwd)` 캡처 + PRD 루트로 `cd`. 300초 타임아웃. 종료 후 `cd "$SAVED_CWD"`
2. 양쪽 결과 수신 후 메인이 매트릭스 판정 → SSOT [`../harness-codex-review.md`](../harness-codex-review.md) "1차 결과 매트릭스" 참조
3. **결과 처리**: 별도 파일 저장 없음. Claude 점수·지적과 Codex High/Critical은 인-메모리에서 PRD 본문에 반영
4. **통과**: Claude 평균 ≥8.0 + 각 항목 ≥7 AND Codex High/Critical 반영 완료 → 다음 단계
5. **미달**: 통합 반영을 PRD 본문에 갱신 → 새 서브에이전트로 Claude만 재실행 (Codex 재호출 금지). 최대 2회 추가, 총 3회. 회차 추적 없음
6. **기술 실패 Fallback**: 동일 SSOT 적용 (Agent 오류 시 최대 2회 재호출, 지속 실패 시 사용자 보고)
7. **Codex 비-스킵 비정상 종료**: 워크플로우 중단 + 사용자 보고

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
- 3회 실패 → 사용자에게 보고

## ▶ 자동 전환

Claude 통과 + (Codex High/Critical 반영 완료 OR Codex 토큰/기능 SKIPPED) 시 `✓ [5] 엔지니어링 리뷰 통과 (Claude+Codex)` (스킵 시 `(Claude+Codex SKIPPED)`) 출력 후 **`/rp-task` 자동 진입**.

→ PRD 상세: [`../harness-prd.md`](../harness-prd.md)
→ Codex 리뷰 규칙: [`../harness-codex-review.md`](../harness-codex-review.md)
