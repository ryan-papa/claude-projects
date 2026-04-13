---
name: rp-spec
description: 하네스 구체화. 도메인→기술→외부API인증 순서로 대화형 질문 진행.
disable-model-invocation: true
allowed-tools: Read Grep WebSearch WebFetch
argument-hint: [project-name or feature-description]
---

# 구체화 (Specification)

**대상:** $ARGUMENTS

## 전제 확인

`repositories/*/CLAUDE.md` 또는 프로젝트 컨텍스트가 존재하는지 확인.
없으면: "프로젝트가 초기화되지 않았습니다. `/rp-init`을 먼저 실행하세요." 안내.

## 절차

`docs/harness-prd.md`의 구체화 프로세스를 따른다.

**질문 규칙 (CLAUDE.md 기준):**
- 1개씩 대화형 진행
- 객관식 4개 + 추천 점수 + 주관식
- 최소 2개, 최대 15개

**질문 순서:**

### 1단계: 도메인 정보
- 대상 특정, 데이터 구분, 인증·인가, 입력 데이터, 결과 전달 대상

### 2단계: 기술 정보
- 핵심 기능, 개발 언어·런타임, 실행 환경, 외부 연동

### 3단계: 외부 API 인증 점검 (외부 API 연동 1개 이상 시)
- `docs/harness-prd.md` §외부 API 인증 점검 체크리스트 순회
- 토큰 라이프사이클 / 초기 셋업+장애 시나리오 / 보안

### 4단계: UI/UX (웹/앱 프로젝트 시)
- 대상 사용자, 디자인 톤, 레이아웃, 인터랙션, 참고 사이트

### 5단계: 구현 조건 분해·검증
- 수집된 요구사항에서 구현 조건 분해 → 각 조건 검증
- 미충족 조건 발견 시 사용자에게 제약 + 대안 제시

**기술 스택 불명확 시:** 자동 리서치 후 결과를 `docs/research/`에 저장

## 완료 조건

- [ ] 도메인 필수 항목 수집 완료
- [ ] 기술 필수 항목 수집 완료
- [ ] 외부 API 인증 점검 완료 (해당 시)
- [ ] 구현 조건 검증 완료

완료 후: `✓ [2] 구체화 완료` → `/rp-prd`로 PRD 작성 안내
