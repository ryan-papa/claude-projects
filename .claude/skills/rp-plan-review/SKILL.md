---
name: rp-plan-review
description: 하네스 기획 리뷰. PRD를 7개 항목으로 채점하고 미달 시 재작성.
disable-model-invocation: true
allowed-tools: Read Write Agent
argument-hint: [prd-file-path]
---

# 기획 리뷰 (Planning Review)

**대상 PRD:** $ARGUMENTS

## 전제 확인

PRD 파일이 존재하고 상태가 `Draft` 또는 `Planning Review`인지 확인.
없으면: "PRD가 필요합니다. `/rp-prd`를 먼저 실행하세요." 안내.

## 절차

서브에이전트(Planning Review Agent)에 위임하여 채점한다.

**평가 항목 (10점 만점):**

| 항목 | 설명 | 감점 예시 |
|------|------|----------|
| 명확성 | 요구사항이 모호하지 않은가 | "적절히 처리" 같은 비정량적 표현 |
| 완성도 | 누락된 섹션·내용 없는가 | 에러 케이스 미정의 |
| 실현가능성 | 현실적으로 구현 가능한가 | 검증 없는 기술적 가정 |
| 일관성 | 섹션 간 모순 없는가 | 개요와 기능 요구사항 범위 불일치 |
| 측정가능성 | 성공 기준이 정량적인가 | 수치 없는 목표 |
| 간결성 | 핵심만 담겼는가 | 구현 디테일이 PRD에 포함 |
| 적정크기 | 분량이 과도하지 않은가 | 200줄 초과 |

**판정:**
- 평균 ≥ 8.0 → 통과 → PRD 상태를 `Planning Review` 완료로 기록
- 평균 < 8.0 → 피드백 반영하여 재작성 (최대 3회)
- 3회 후 < 8.0 → `Max retry reached`, 사용자 판단

**결과 기록:** PRD 파일 하단 `## Review 결과 > 기획 리뷰`에 점수·피드백 기입

## 완료 조건

- [ ] 채점 완료 + PRD에 기록됨
- [ ] 평균 ≥ 8.0 통과

완료 후: `✓ [4] 기획 리뷰 통과 (X.XX/10)` → `/rp-eng-review`로 엔지니어링 리뷰 안내
