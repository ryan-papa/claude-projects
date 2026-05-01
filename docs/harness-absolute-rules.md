# Harness 절대 규칙 (SSOT)

본 문서가 **단일 출처(SSOT)**. 다른 문서·스킬에서 동일 규칙 본문 중복 정의 금지, 링크 참조만 허용.
[`../CLAUDE.md`](../CLAUDE.md) 진입점에서 본 파일을 참조.

## ⛔ 적용 범위

예외 없음. "급해서", "간단해서", "나중에" 등 어떤 이유로도 단계 스킵 불가.

## 문서·리뷰 절차

- **[리뷰 단계 서브에이전트 필수]** `/rp-plan-review`·`/rp-eng-review`·`/rp-code-review` Claude 리뷰 적용:
  - 실행 주체: Agent 툴 분리 서브에이전트(`subagent_type=general-purpose`). 메인 셀프 채점 **전면 금지** (이해충돌·관성 편향 방지)
  - 역할 분리: 서브 = Claude 채점, 메인 = `/codex:review` 실행·결과 저장
  - 서브에이전트 프롬프트 4 필수 항목(SSOT): (a) 대상 파일 경로 + 참고 파일, (b) 채점 기준(단계별 항목 수: 4=9, 5=5, 9=7), (c) 독립 판정 지시(메인 컨텍스트 미공유), (d) 역할 경계: 본 단계 채점만 수행 — **Claude 측 적용 시 Codex 실행·저장 금지 / Codex 측 적용 시 `review-claude-*.md` 합성 금지** (`.codex/skills/` 어댑터 노트 동시 참조). **스킬 파일은 본 (a)~(d) 본문 중복 금지, 링크 참조만 허용**
  - Fallback: 기술적 실패 최대 2회 재호출 → 지속 시 사용자 즉시 보고. 셀프 채점 우회 금지
  - 평가 미달: PRD 재작성 후 새 서브에이전트 재실행 (기술 실패와 구분)
  - 증거 저장: `<project-root>/docs/prd/[feature]/review-claude-{plan,eng,code,meta}-r{N}.md`. `{meta}` = 간소 PRD 단일 리뷰 전용. 회차별 새 파일 (덮어쓰기 금지)
- 기능 변경 시 코드 전에 PRD 문서 먼저 업데이트 + 리뷰
- **[4][5][9] Codex 추가 리뷰 필수**: Claude 리뷰 통과 후 `/codex:review --wait` 1회 포그라운드 실행
  - 타임아웃: wall-clock 300초
  - cwd: 일반 기능 `repositories/[project]/`, 하네스 메타 변경 `claude-projects/`
  - 점수화 없음. High/Critical 지적 반영 후 다음 단계 진입
  - 결과 저장: `<project-root>/docs/prd/[feature]/review-codex-{plan,eng,code,meta}.md`
  - 토큰/기능 신호 패턴 명시 출력 시 1회 스킵 + SKIPPED 헤더 + 7항목 증거 저장 ([`harness-codex-review.md`](harness-codex-review.md) "토큰·기능 신호 패턴" SSOT)
  - 그 외 비정상 종료(네트워크·login·플러그인·hang·매칭 0건): 중단 + 사용자 보고. 자동 재시도 금지

## 테스트·QA·게이트

- 테스트 코드 없이 커밋/머지 절대 금지
- 테스트·빌드 미통과 시 다음 단계 진행 금지
- **QA·코드 리뷰 단계 생략 금지 (게이트 강화)**: 두 단계 모두 완료 여부 배포 전 체크. 하나라도 미수행이면 배포 차단. "단위 테스트 통과 = QA 대체 불가"
- **프런트엔드 변경 시 Playwright E2E + axe(접근성) 검사 필수**: `rp-qa` 단계에서 둘 다 실행, 실패 시 진행 차단. E2E 테스트가 없는 UI 태스크는 완료 불가
- **백엔드 변경 시 4-게이트 의무**: [`harness-backend-test-policy.md`](harness-backend-test-policy.md) — (1) 단위 항상 (2) 읽기 endpoint API 응답 테스트 (3) SQL/JPA/마이그레이션 변경 시 `@SpringBootTest+@Transactional` ROLLBACK 통합 테스트 (4) 컨트롤러·DI·Flyway 변경 시 로컬 `bootRun` + 헬스 + Flyway 로그 + OAuth 인증 endpoint 응답 캡처. 미준수 머지 차단
- **push 전 README 검증 필수** (5항목 평균 8.0+, 최대 3회 재시도, 미통과 시 push 차단)

## 배포·머지·브랜치

- 산출물 보고 없이 배포 진행 금지 (커밋·PR은 산출물 보고 후 자동)
- **CI 통과 전 머지 금지** (예외 없음)
- **feat 브랜치 직행 배포 금지**: 모든 배포는 `rp-ship` 경유 (PR → CI → main 머지 → 배포). feat/통합 브랜치 상태로 프로덕션 프로세스 기동·노출 금지. 단, 로컬 개발 서버(`uvicorn --reload`)는 예외
- **`main` 직접 수정 금지**: `main` 브랜치에서 docs·CLAUDE.md·스킬·settings 수정 감지 시 즉시 중단 + feat 브랜치 전환 요구
- **`rp-ship` 스킬 호출 필수**: 커밋·PR·머지·배포는 수동 `git`/`gh` 우회 없이 `rp-ship` 스킬 경유. 단, `rp-ship` 스킬 내부 절차로 명시된 명령은 예외
- **`rp-ship` 자동 머지 가드 4종 AND**: 모두 충족 시에만 자동 머지
  - (a) CI 모든 체크 SUCCESS
  - (b) 리뷰 증거 게이트 통과
  - (c) PR base 정상 감지
  - (d) `gh pr view --json mergeable` = `MERGEABLE`
  - 하나라도 실패 → 중단 + OPEN 유지 + 사용자 보고. `--admin`·`--no-verify` 우회 금지. 비상 탈출구 `RP_SHIP_MANUAL=1` 환경변수만 자동 머지 비활성 허용
- **`rp-ship` PR base 자동 감지 게이트**: 통합 브랜치 선언 감지 → `--base` 주입
  - 감지 순서: (0) 메타 분기 선검사 (`review-claude-meta-r*.md` 또는 `review-codex-meta.md` 존재 → `--base main`) → (1) `docs/tasks.md` → (2) `CLAUDE.md` → (3) repo default
  - 매칭 정규식: `^[\s\-\*|]*통합 브랜치:\s*`?([A-Za-z0-9/_\-]+)`?` 정확히 1건만 채택
  - Fail-closed: 2건+ 매칭·공백 포함·원격 부재·detached HEAD·프로젝트 루트 미확인 → 중단. 느슨한 `feat/*` 추론 금지
  - 우회 허용: 수동 `--base <X>`, 메타 분기 자동
  - base 리타깃 시 CI 재실행 후 자동 머지 가드 재확인

## 단축 경로·예외

- **하네스 메타 변경 단축 경로**: `rp-init`·`rp-specify`·`rp-task`·`rp-dev` 스킵 + feat 브랜치 + `rp-prd` 간소(변경 이유·영향 파일·롤백·검증 4섹션) + 리뷰 + `rp-ship`. 완전 생략은 금지
- **회고(`/rp-retro`)는 사용자 명시 명령 시에만 실행** — 자동 진입 없음. 필요 시 사용자가 직접 `/rp-retro` 호출
- 워크플로우 위반 발견 시 즉시 중단하고 빠진 단계부터 재진행
