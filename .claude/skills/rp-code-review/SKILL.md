---
name: rp-code-review
description: 하네스 코드 리뷰. 8개 항목으로 태스크 코드를 채점.
disable-model-invocation: true
allowed-tools: Read Grep Agent Bash
argument-hint: [task-branch or task-id]
---

# 코드 리뷰 (Code Review)

**대상:** $ARGUMENTS

## 전제 확인

태스크 개발이 완료되고 테스트가 통과했는지 확인.
미완료 시: "개발이 필요합니다. `/rp-dev`를 먼저 실행하세요." 안내.

## 절차

서브에이전트(Code Review Agent)에 위임. `docs/harness-code-review.md` 기준.

**시점:** 태스크 브랜치를 통합 브랜치에 머지하기 전.

**평가 항목 (8개, 10점 만점):**
정확성 / 보안 / 성능 / 설계 / 가독성 / 테스트 / 동시성 / 운영 가시성

**PR 유형별 포커스:**

| 유형 | 집중 |
|------|------|
| 버그 픽스 | 정확성 + 테스트 |
| 새 기능 | 설계 + 보안 + 성능 |
| 리팩토링 | 가독성 + 테스트 |

**판정:**
- 평균 ≥ 8.0 → 통과 → 통합 브랜치 머지 진행
- 평균 < 8.0 → Dev Agent 재투입 (최대 3회)
- 3회 후 < 7.0 → 사용자 확인
- 3회 후 7.0~8.0 → 통과 처리

**이슈 처리:**
- High/Mid → 자동 수정
- Low → 테이블로 제시, 사용자 선택

**머지 후:** `git log --oneline -3`으로 반영 확인

## 완료 조건

- [ ] 8개 항목 채점 완료
- [ ] 평균 ≥ 8.0 (또는 3회 후 7.0+)
- [ ] High/Mid 이슈 수정됨
- [ ] 통합 브랜치 머지 완료

완료 후: `✓ [8] 코드 리뷰 통과 (X.XX/10)` → 다음 태스크 `/rp-dev T-XX` 또는 전체 완료 시 `/rp-qa` 안내
