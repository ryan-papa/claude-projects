---
name: rp-ship
description: 하네스 최종 산출물. PR 생성, PRD Open Issues 업데이트, README 확인.
disable-model-invocation: true
allowed-tools: Read Write Edit Bash Glob
argument-hint: [project-name]
---

# 최종 산출물 (Ship)

**대상:** $ARGUMENTS

## 전제 확인

QA가 통과했는지 확인.
미통과 시: "QA가 필요합니다. `/rp-qa`를 먼저 실행하세요." 안내.

## 절차

### 1. 최종 검증

- [ ] 전체 테스트 통과 (`pytest` 또는 프로젝트 테스트 명령)
- [ ] 빌드 성공
- [ ] 통합 브랜치에 모든 커밋 포함

### 2. PRD Open Issues 업데이트

PRD 파일의 Open Issues 섹션을 현재 상태에 맞게 업데이트:
- 해결된 항목 → `Closed` 처리
- 남은 항목 → 상태 갱신

### 3. README 확인

`docs/harness-readme.md` 기준으로 README가 최신 상태인지 확인:
- 새 기능이 반영되었는지
- 설치/실행 명령이 유효한지
- 프로젝트 구조가 실제와 일치하는지

### 4. PR 생성

통합 브랜치 → main PR 생성:
- 제목: 70자 이하
- 본문: Summary (변경 내용) + Test plan (테스트 결과)
- `gh pr create` 사용

### 5. tasks.md 업데이트

전체 태스크 상태를 `Done`으로 확인.

## 완료 조건

- [ ] PR 생성됨 (URL 출력)
- [ ] PRD Open Issues 업데이트됨
- [ ] README 최신 상태
- [ ] tasks.md 전체 Done

완료 후: `✓ [10] 최종 산출물 전달 완료` + PR URL 출력
