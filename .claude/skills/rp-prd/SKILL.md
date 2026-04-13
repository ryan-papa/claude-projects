---
name: rp-prd
description: 하네스 PRD 작성. 구체화 결과를 기반으로 PRD 템플릿에 맞춰 작성.
disable-model-invocation: true
allowed-tools: Read Write Glob
argument-hint: [project-name or feature-name]
---

# PRD 작성

**대상:** $ARGUMENTS

## 전제 확인

구체화가 완료되었는지 확인 (도메인+기술 정보 수집 여부).
미완료 시: "구체화가 필요합니다. `/rp-spec`을 먼저 실행하세요." 안내.

## 절차

`docs/prd-template.md` 템플릿에 맞춰 PRD를 작성한다.

**파일명:** `repositories/[project]/docs/prd/YYYYMMDD_HHMMSS_[feature]_[8자리랜덤].md`
- 타임스탬프: `date +%Y%m%d_%H%M%S` 실행값
- 랜덤: 세션당 1회 생성 후 유지

**필수 섹션:**

| 섹션 | 내용 |
|------|------|
| 개요·목적 | 배경, 목표, 범위 |
| 사용자 스토리 | As a / I want / So that |
| 기능 요구사항 | F-XX ID, 요구사항, 우선순위, AC(완료 조건) |
| 비기능 요구사항 | 성능, 보안, 확장성 |
| 제약사항 | 기술·비즈니스 |
| 성공 지표 | 정량적 목표 |
| Open Issues | TBD 추적 |
| Review 결과 | 기획/엔지니어링 리뷰 테이블 (빈 상태) |

**Doc Agent 행동 규칙:**
- 구체화 결과만 기반으로 작성. 추측 금지
- 미확정 사항은 Open Issues로
- 200줄 이하
- 테이블/리스트 우선, 산문 지양

## 완료 조건

- [ ] PRD 파일 생성됨
- [ ] 모든 필수 섹션 채움
- [ ] Open Issues 정리됨
- [ ] 상태: `Draft`

완료 후: `✓ [3] PRD 작성 완료` → `/rp-plan-review`로 기획 리뷰 안내
