---
name: rp-qa
description: 하네스 QA. PRD 기능 요구사항 기준 시나리오 테스트. 프로젝트 유형별 체크리스트.
disable-model-invocation: true
allowed-tools: Read Bash Glob Grep Agent
argument-hint: [project-name]
---

# QA (Quality Assurance)

**대상:** $ARGUMENTS

## 전제 확인 (QA 사전 검증)

- [ ] 전체 태스크 브랜치가 통합 브랜치에 머지됨 (`git log`)
- [ ] 주요 소스 파일 존재 (`ls src/`)
- [ ] 전체 테스트 통과

미충족 시: 미완료 항목 안내 + "해당 태스크의 `/rp-dev`, `/rp-code-review`를 먼저 완료하세요."

## 절차

PRD 기능 요구사항(F-XX) 기준으로 시나리오 테스트 수행.

### 배치/CLI/API 프로젝트

| 항목 | 확인 내용 |
|------|----------|
| 정상 실행 | 메인 명령어 → 기대 출력 |
| 에러 핸들링 | 잘못된 입력, 네트워크 단절, 파일 없음 |
| 인증 실패 | 토큰 만료/잘못된 키 → 명확한 에러 |
| 중복 실행 | 멱등성 확인 |
| 로그 출력 | INFO/WARNING/ERROR 적절, 민감 정보 미출력 |
| 빈 데이터 | 처리할 데이터 없을 때 정상 종료 |
| 설정 누락 | 필수 환경변수 없을 때 명확한 에러 |

### 웹 프로젝트

- Playwright E2E 필수: 모든 페이지 접근, 네비게이션 링크 클릭, 주요 플로우
- UI 프로젝트: 디자인 QA → `docs/harness-design.md` §7 기준 (평균 ≥ 8.0)

### QA 실패 시

```
실패 항목 발견 → Dev Agent 재투입 → 수정 → 테스트 → 재실행 (최대 3회)
3회 후 실패 → 사용자 보고
```

## 완료 조건

- [ ] F-XX 전체 시나리오 통과
- [ ] 프로젝트 유형별 체크리스트 통과
- [ ] 디자인 QA 통과 (UI 프로젝트 시)

완료 후: `✓ [9] QA 통과` → `/rp-ship`으로 최종 산출물 안내
