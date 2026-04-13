---
name: rp-init
description: 하네스 프로젝트 초기화. 디렉토리 구조, CLAUDE.md, README, PRD 폴더 생성.
disable-model-invocation: true
allowed-tools: Read Write Bash Glob
argument-hint: [project-name]
---

# 프로젝트 초기화

**대상 프로젝트:** $ARGUMENTS

## 절차

`docs/harness-workflow.md`의 프로젝트 초기화 + 레포 초기화 규칙을 따른다.

1. `repositories/$ARGUMENTS/` 디렉토리 생성
2. 아래 파일 자동 생성:

| 파일 | 내용 |
|------|------|
| `CLAUDE.md` | 프로젝트별 설정 (공통 docs/ 참조) |
| `README.md` | 유형별 템플릿 적용 (`templates/readme-opensource.md` 또는 `templates/readme-service.md`) |
| `docs/prd/` | PRD 디렉토리 |
| `docs/research/` | 리서치 디렉토리 |

3. 레포 초기화 (신규 레포 시):
   - 브랜치 프로텍션, PR CI, 시크릿 스캔 설정
   - 기존 설정 있으면 스킵

4. 프로젝트 유형 질문 → README 템플릿 선택

## 완료 조건

- [ ] 디렉토리 구조 생성됨
- [ ] CLAUDE.md 작성됨
- [ ] README.md 템플릿 적용됨

완료 후: `✓ [1] 프로젝트 초기화 완료` 출력 → `/rp-spec $ARGUMENTS`로 구체화 진행 안내
