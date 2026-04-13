---
name: rp-eng-review
description: 하네스 엔지니어링 리뷰. PRD를 6개 기술 항목으로 채점.
disable-model-invocation: true
allowed-tools: Read Write Agent Grep
argument-hint: [prd-file-path]
---

# 엔지니어링 리뷰 (Engineering Review)

**대상 PRD:** $ARGUMENTS

## 전제 확인

기획 리뷰를 통과했는지 확인 (PRD 하단 기획 리뷰 점수 ≥ 8.0).
미통과 시: "기획 리뷰가 필요합니다. `/rp-plan-review`를 먼저 실행하세요." 안내.

## 절차

서브에이전트(Engineering Review Agent)에 위임. 기존 코드베이스도 함께 분석.

**평가 항목 (10점 만점):**

| 항목 | 설명 | 감점 예시 |
|------|------|----------|
| 요구사항 명확성 | AC 존재, 엣지 케이스 명시 | AC 없음, HTTP 에러별 처리 미구분 |
| 기술적 실현가능성 | 스택 정합성, 외부 의존성 | rate limit 미확인, 미검증 라이브러리 |
| 범위·공수 적정성 | 숨겨진 작업, 연쇄 변경 | 마이그레이션 순서 부재 |
| NFR 충족도 | 성능, 보안, 관측가능성 | 로그 레벨 미정의, 동시 쓰기 미고려 |
| 의존성·리스크 | 기술 부채, 롤백 계획 | API 스키마 변경 대응 없음 |
| 테스트 가능성 | 테스트 전략, mock 설계 | 단위/통합 구분 없음 |

**리뷰 원칙:** 각 항목에 "이걸 어떻게 테스트할 수 있는가?" 질문 적용

**판정:**
- 평균 ≥ 8.0 → 통과 → PRD 상태를 `Approved`로 변경
- 평균 < 8.0 → 피드백 반영 재작성 (최대 3회)
- 3회 후 < 8.0 → `Max retry reached`, 사용자 판단

**결과 기록:** PRD 파일 하단 `## Review 결과 > 엔지니어링 리뷰`에 기입

## 완료 조건

- [ ] 채점 완료 + PRD에 기록됨
- [ ] 평균 ≥ 8.0 통과
- [ ] PRD 상태 `Approved`

완료 후: `✓ [5] 엔지니어링 리뷰 통과 (X.XX/10)` → `/rp-tasks`로 태스크 분해 안내
