# Retro R1 — Codex 스킵 규칙 PR 회고 반영 근거

## 출처
[PR #29](https://github.com/ryan-papa/workflow-agent-harness/pull/29) "feat(harness): Codex 추가 리뷰 토큰/기능 이슈 시 1회 스킵 허용" 배포 직후 회고에서 도출.

## 채택 항목

| # | 분류 | 내용 |
|---|------|------|
| 1 | 규칙 보강 | 메타 변경 단축 경로(rp-prd 간소 PRD)에서 PR base 자동 감지를 우회하고 `--base main`을 자동 적용 |
| 2 | 규칙 보강 | `.github/workflows/ci.yml` internal links 검사가 마크다운 anchor(`#fragment`)를 파일 경로로 해석하지 않도록 strip 처리 |
| 3 | 흐름 변경 | rp-ship 머지 단계를 사용자 승인 → CI+게이트+base+MERGEABLE AND 자동 머지로 전환 (Q1=1, Q2=2 결정) |
| 4 | 흐름 변경 | rp-ship 완료 → /rp-retro 자동 진입 제거. 회고는 사용자 명시 명령 시에만 실행 (Q3=2 결정) |

## 미채택 (회고에 보존, 차후 follow-up 가능)
- (3) R3 Claude 리뷰 개선 제안 일부 사후 반영 (Must/Should 라벨, 패턴 적중률)
- (4) Codex 리뷰 wrapper 구현
- (5) docs/tasks.md 통합 브랜치 단일성 강제
- (6) amend 사용 금지 + README 보강 별도 commit 분리 명시

## 적용 영향
| 항목 | 변경 |
|------|------|
| PR base 자동 감지 게이트 | 메타 변경 시 분기 추가. 일반 기능 경로는 영향 없음 |
| CI lint-docs | fragment strip 처리. 기존 valid 링크 검사 결과 동일 |
| Codex 추가 리뷰 규칙 | 변경 없음 (방금 머지된 규칙은 그대로 유지) |
