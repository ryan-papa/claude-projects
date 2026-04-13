---
name: rp-dev
description: 하네스 개발. 지정 태스크 1개를 구현하고 테스트 작성.
disable-model-invocation: true
allowed-tools: Read Write Edit Bash Glob Grep Agent
argument-hint: [task-id, e.g. T-01]
---

# 개발 (Development)

**대상 태스크:** $ARGUMENTS

## 전제 확인

1. tasks.md가 존재하고 해당 태스크가 `Todo` 상태인지 확인
2. 통합 브랜치가 존재하는지 확인

미충족 시: "태스크 분해가 필요합니다. `/rp-tasks`를 먼저 실행하세요." 안내.

## 절차

`docs/harness-dev.md`의 태스크별 개발 사이클을 따른다.

```
통합 브랜치에서 태스크 브랜치 생성
  ↓
Dev Agent에 1태스크 위임
  ↓
빌드 + 테스트 직접 실행하여 검증
  ↓ 실패 시 재투입 (최대 3회)
테스트 통과
```

**Dev Agent 행동 규칙:**
- 1태스크만 수행
- 테스트 코드 필수 작성 (비즈니스 로직 중심)
- 기존 코드 스타일 준수
- 범위 밖 수정 금지

**⛔ 절대 규칙:**
- 테스트 코드 없이 커밋 금지
- 테스트 미통과 상태에서 다음 태스크 금지
- 빌드 실패 상태에서 다음 태스크 금지
- 에이전트 보고만 믿지 않음 → 직접 빌드+테스트 실행

## 완료 조건

- [ ] 태스크 브랜치 생성됨
- [ ] 코드 구현 완료
- [ ] 테스트 코드 작성됨
- [ ] 테스트 전체 통과
- [ ] 빌드 성공
- [ ] tasks.md 상태 `Done` 업데이트

완료 후: `✓ [7] T-XX 개발 완료` → `/rp-code-review`로 코드 리뷰 안내
