# PRD: Codex 리뷰 종료 후 시작 cwd 복귀 강제

분류: 하네스 메타 변경 (단축 경로)

## 1. 변경 이유

`/codex:review` 실행 시 PRD 프로젝트 루트로 `cd` 이동하지만, 종료 후 시작 cwd로 복귀하는 규칙이 없어 다음 단계에서 cwd 오인이 발생함. `harness-codex-review.md` 실행 규칙 표는 "실행 전 체크"만 명시(L29), 종료 후 처리는 누락. Codex 리뷰는 단계 4·5·9 + 메타 변경 단일 리뷰에서 모두 호출되므로 누락 시 후속 단계(단계 5 시작 cwd, ship 단계 PR base 자동 감지 등) 모두 영향.

## 2. 대안 비교

| 대안 | 장점 | 단점 | 채택 |
|------|------|------|------|
| (A) SSOT 문서에 규칙 명시 + 스킬은 링크 참조 | 1곳 정의·유지보수 단순·기존 패턴과 일관 | 사람·LLM이 지킬 책임 | ★ |
| (B) Stop 훅으로 자동 `cd` 복귀 | 강제력 | Bash 도구 cwd는 도구 내부 상태라 훅이 별도 셸로 실행되면 복귀 불확실·디버깅 난이도↑ | × |
| (C) 래퍼 스크립트(pushd/popd) | 자동·문서 무관 | 신규 파일·codex 플러그인 호출 경로 우회 | × |

(A) 채택 사유: 사용자 요청 범위가 "codex 리뷰 명시적 이동 시점만 복귀". 무차별 자동화는 의도된 디렉토리 이동도 막을 위험. SSOT 패턴은 기존 `harness-codex-review.md` "토큰·기능 신호 패턴" 등에서 이미 검증된 형태.

## 3. 영향 파일

| 파일 | 변경 내용 |
|------|-----------|
| `docs/harness-codex-review.md` | 실행 규칙 표(L20-30)에 "실행 전 cwd 저장(`SAVED_CWD=$(pwd)`)"·"실행 후 cwd 복귀(`cd "$SAVED_CWD"` + `pwd` 검증)" 2행 추가. 직렬 실행 패턴(L32-47) 정상/SKIPPED/중단 3분기 모두에 `cd "$SAVED_CWD"` 단계 추가. 복귀 실패(원래 디렉토리 삭제 등) 시 중단 + 사용자 보고 1줄 추가 |
| `docs/harness-absolute-rules.md` | `[4][5][9] Codex 추가 리뷰 필수` 하위 불릿(L20-26)에 "실행 종료 후 시작 cwd 복귀 필수 (정상/SKIPPED/중단 무관). 누락 시 다음 단계 진행 금지" 1줄 추가 |

`SAVED_CWD` 라이프사이클: `/codex:review` 진입 **직전** `SAVED_CWD=$(pwd)` 캡처 → PRD 루트로 `cd` → Codex 실행 → 종료 직후(3분기 무관) `cd "$SAVED_CWD"` 복귀.

스킬 파일(`rp-plan-review.md`·`rp-eng-review.md`·`rp-code-review.md`)은 이미 SSOT 링크 참조 패턴이라 별도 수정 불필요.

## 4. 롤백

| 단계 | 조치 |
|------|------|
| 1 | `git revert <merge-commit>` → main 에서 본 PR 제거 |
| 2 | 두 파일 직전 커밋 상태로 복원, 스킬 파일 변경 없음 → 부수 효과 없음 |

## 5. 검증

| 항목 | 결과 기준 |
|------|----------|
| 변경 파일 ≤ 200줄 | `harness-codex-review.md`·`harness-absolute-rules.md` 모두 200줄 이하 유지 |
| SSOT 1곳 유지 | cwd 복귀 규칙 본문은 `harness-codex-review.md` 실행 규칙 표 1곳, 절대 규칙은 한 줄 요약만 |
| 죽은 링크 0건 | 신규 링크 없음 (기존 링크 유지) |
| 3분기 동작 시뮬레이션 | 정상 종료/SKIPPED/중단 각 케이스에서 종료 직후 `pwd` 출력이 `SAVED_CWD`와 일치하는지 패턴 검증 (`harness-codex-review.md` "스킵 규칙 검증" 양방향 시뮬레이션과 동형) |
| 복귀 실패 분기 명시 | 표·패턴에 "`cd "$SAVED_CWD"` 실패 시 중단 + 사용자 보고" 문구 존재 확인 |
| 메타 리뷰 게이트 | Claude `review-claude-meta-r{N}.md` + Codex `review-codex-meta.md` 통과 |
| CI lint-docs | 통과 |
