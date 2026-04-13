# Harness Engineering Workflow

## Full Flow

```
사용자 기능 전달
  ↓
▶ [1] 프로젝트 초기화
  ↓
▶ [2] 구체화 (도메인 → 기술, 질문·리서치)
  ↓
▶ [3] PRD 작성 (Doc Agent)
  ↓
▶ [4] 기획 리뷰 (Planning Review Agent) ← 최대 3회
  ↓
▶ [5] 엔지니어링 리뷰 (Engineering Review Agent) ← 최대 3회
  ↓ 점수 만족 시 바로 개발 진입 (사용자 승인 생략)
▶ [6] 태스크 분해
  ↓
▶ [7] 개발 (통합 브랜치 → 태스크별 반복)
  ↓ 태스크 머지 시마다 코드 리뷰 수행
▶ [8] 코드 리뷰 (8항목, 태스크 머지 시) → 개선 ← 최대 3회
  ↓ 전체 태스크 완료 후
▶ [9] QA (사전 검증 → 시나리오 테스트)
  ↓
▶ [10] 최종 산출물 전달
```

**상태 메시지:** 진입 `▶ [N] 단계명...`, 완료 `✓ [N] 단계명 완료`

**단계 전환 조건:**

| 전환 | 진입 조건 |
|------|----------|
| [1]→[2] | 프로젝트 디렉토리·CLAUDE.md·README 생성 완료 |
| [2]→[3] | 필수 항목(도메인+기술) 수집 완료 + 외부 API 인증 점검 완료(해당 시) |
| [3]→[4] | PRD 작성 완료 (모든 섹션 채움, Open Issues 정리) |
| [4]→[5] | 기획 리뷰 평균 ≥ 8.0 |
| [5]→[6] | 엔지니어링 리뷰 평균 ≥ 8.0 |
| [6]→[7] | tasks.md 작성 + 통합 브랜치 생성 |
| [7]→[8] | 각 태스크: 테스트 통과 후 코드 리뷰 대상 |
| [8]→[9] | 전체 태스크 코드 리뷰 통과 + 통합 브랜치 머지 완료 + QA 사전 검증 통과 |
| [9]→[10] | QA 시나리오 전체 통과 |

---

→ PRD 상세: [`harness-prd.md`](harness-prd.md)
→ 개발 상세: [`harness-dev.md`](harness-dev.md)
→ 코드리뷰 상세: [`harness-code-review.md`](harness-code-review.md)

## 공통 규칙

**프로젝트 초기화 (자동 생성):**

| 항목 | 경로 | 비고 |
|------|------|------|
| 프로젝트 설정 | `repositories/[project]/CLAUDE.md` | |
| 리드미 | `repositories/[project]/README.md` | 유형별 템플릿 적용 |
| PRD 디렉토리 | `repositories/[project]/docs/prd/` | |
| 리서치 디렉토리 | `repositories/[project]/docs/research/` | |

**레포 초기화 (신규 레포 생성 시, 기존 설정 있으면 스킵):**

| 항목 | 내용 |
|------|------|
| 브랜치 프로텍션 | main 직접 push 금지, PR 필수 |
| PR CI | 빌드 + 테스트 자동 실행, 실패 시 머지 차단 |
| 시크릿 스캔 | GitHub secret scanning + push protection |
| 머지 후 검증 | `git log --oneline -3`으로 커밋 반영 확인 |

**README 템플릿:** 프로젝트 유형에 따라 선택 적용
- 오픈소스/라이브러리 → [`templates/readme-opensource.md`](templates/readme-opensource.md)
- 사내 서비스/API → [`templates/readme-service.md`](templates/readme-service.md)
- 작성 규칙: [`harness-readme.md`](harness-readme.md)

**에이전트 역할 정의:**

| 에이전트 | 역할 | 행동 규칙 |
|----------|------|----------|
| Doc Agent | PRD 작성·수정 | 구체화 결과를 기반으로 PRD 템플릿에 맞춰 작성. 추측 금지, 미확정 사항은 Open Issues로. 200줄 이하 |
| Dev Agent | 태스크 구현 | 1태스크만 수행. 테스트 코드 필수 작성. 기존 코드 스타일 준수. 범위 밖 수정 금지 |
| Planning Review Agent | 기획 관점 검증 | 7개 항목 10점 채점. 사용자 관점에서 누락·모호함 지적. 감점 사유 구체적 명시 |
| Engineering Review Agent | 기술 관점 검증 | 6개 항목 10점 채점. "이걸 어떻게 테스트할 수 있는가" 원칙. 구현 리스크 지적 |
| Code Review Agent | 코드 품질 검증 | 8개 항목 10점 채점. PR 유형별 포커스 적용. 이슈 심각도별 처리 |
| QA Agent | 기능 검증 | PRD F-XX 기준 시나리오 테스트. 프로젝트 유형별 체크리스트 적용. 실패 시 재현 단계 명시 |

**에이전트 컨텍스트 표준 (필수 전달 항목):**

| 항목 | 내용 |
|------|------|
| 역할 정의 | 위 테이블의 역할·행동 규칙 |
| 파일 경로 | 읽기·쓰기 대상 경로 |
| 요구사항 요약 | 수집된 내용 |
| 작성 규칙 | 200줄 이하, 테이블/리스트 우선 |

**리서치 결과 저장:**
- 경로: `repositories/[project]/docs/research/YYYYMMDD_[topic].md`
- Doc Agent 호출 시 경로를 컨텍스트로 전달

**문서 정합성:**
- docs/ 파일 변경 시 CLAUDE.md 트리·링크 자동 동기화

**스펙 변경 시:** → [`harness-dev.md` §스펙 변경 시 플로우](harness-dev.md) 참조

**UI 프로젝트 시 디자인 원칙:**
- UI가 포함된 프로젝트는 [`harness-design.md`](harness-design.md)의 디자인 원칙을 준수
