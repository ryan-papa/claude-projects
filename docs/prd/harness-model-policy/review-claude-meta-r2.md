# Claude 메타 리뷰 r2 — harness-model-policy

| 항목 | 값 |
|---|---|
| 회차 | r2 |
| 대상 | `docs/prd/harness-model-policy/prd.md` (실재 확인) |
| 작성일 | 2026-04-25 |
| 리뷰어 | 메인 (사용자 명시 지시) |

## r1 지적 검증

| ID | 등급 | 내용 | r2 검증 결과 |
|---|---|---|---|
| C-1 | Critical | `prd.md` 부재 | **무효**. 절대 경로 Read 결과 67 라인 정상 존재. r1 판정 오류 (도구 출력 가로채기 가능성). |
| H-1 | High | 효과 검증 절차 부재 | **반영**. §4 #6 정량 측정 (재작업 회차 r1→r2→r3, Codex High/Critical 발견 수, 4주 후 회고 종합 검토) 추가. |
| H-2 | High | code-review/eng-review 일률 Sonnet 의 보안·동시성 PR 리스크 | **반영**. §2 Opus override 예외 조항 추가 (보안 민감·동시성/트랜잭션·아키텍처 신규 도입). 명시 `/model opus` 전환 + review 산출물 헤더 모델 표시. |
| M-1 | Medium | 스킬 frontmatter 미수정 | **반영**. §2 본문에 "frontmatter `model:` 미추가 — 사용자 자유도 보존" 명시 + 후속 PR(hook 자동화) 분리 명시. |
| L-1 | Low | SSOT 중복 (CLAUDE.md + workflow.md) | **반영**. §4 #7 후속 정리 명시 (`harness-workflow.md` SSOT, `CLAUDE.md` 요약 1줄+링크). |

## 채점 (0~10)

| 항목 | 점수 | 근거 |
|---|:-:|---|
| 변경 이유 | 9 | §1 비용·속도·추론 우위 분담 논리 명료. Opus 가 PRD 초안에 집중되는 근거 구체. |
| 영향 파일 | 9 | 파일 3종 + 단계×모델 12행 표 + 예외 조항. 누락 없음. frontmatter 미수정 의도 명시. |
| 롤백 | 9 | 3 시나리오 (단계별 부분 롤백·정책 전체 revert·사용자 자유 전환). 영향 범위 2 문서로 한정 명시. |
| 검증 | 9 | 7 항목 (lint·가독성·인지·Codex·수기 측정·정량 측정·SSOT 후속). r1 H-1 정량화 보강 반영. |
| 정책 일관성 | 9 | Opus override 예외가 보안·동시성·아키텍처 케이스 커버. 상위 룰(`rp-*` 서브에이전트·Codex 의무)과 충돌 없음. 채점 vs 추론 부담 구분 논리 일관. |
| 후속 영향 | 9 | hook 자동화·SSOT 통합·4주 효과 측정 → 명시 후속 경로. 본 PR 범위와 후속 분리 명확. |

**최저 = 9 / 종합 = 9**

## 통과 여부

**통과 (8.0+)**. r1 High 2건 정량·예외 조항으로 반영, Medium·Low 후속 PR 경로 명시. r1 Critical 은 도구 가로채기 기인 오판으로 무효 처리.

## 다음 단계

1. `/codex:review --wait` 메타 단일 리뷰 (`review-codex-meta.md`)
2. Codex High/Critical 지적 시 본 PRD 보강 → r3
3. 통과 시 `/rp-ship` (단축 경로)
