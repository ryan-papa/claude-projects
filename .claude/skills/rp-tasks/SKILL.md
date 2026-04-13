---
name: rp-tasks
description: 하네스 태스크 분해. PRD 기능 요구사항을 개발 태스크로 분해.
disable-model-invocation: true
allowed-tools: Read Write Glob
argument-hint: [prd-file-path]
---

# 태스크 분해

**대상 PRD:** $ARGUMENTS

## 전제 확인

PRD 상태가 `Approved`인지 확인.
미승인 시: "PRD 리뷰가 필요합니다. `/rp-eng-review`를 먼저 실행하세요." 안내.

## 절차

1. PRD의 기능 요구사항(F-XX)을 읽는다
2. 구현 단위로 태스크를 분해한다
3. `repositories/[project]/docs/tasks.md`에 기록한다
4. 통합 브랜치를 생성한다

**태스크 테이블 형식:**

| ID | 설명 | PRD | 우선순위 | 상태 | 브랜치 |
|----|------|-----|----------|------|--------|
| T-XX | 구현 내용 | F-XX | High/Mid/Low | Todo | `feat/T-XX-description` |

**분해 원칙:**
- 1태스크 = 1 PR 가능한 단위
- High 우선순위부터 정렬
- 의존 관계가 있으면 순서 반영
- 태스크 브랜치명: `feat/T-XX-description`
- 통합 브랜치명: `feat/[project-name]` 또는 `feat/[feature-name]`

## 완료 조건

- [ ] tasks.md 작성됨
- [ ] 통합 브랜치 생성됨
- [ ] 태스크 브랜치명 확정됨

완료 후: `✓ [6] 태스크 분해 완료 (N개 태스크)` → `/rp-dev T-01`로 개발 안내
