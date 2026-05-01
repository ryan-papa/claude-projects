# Codex Meta Review — PR-B

실행: `codex review --uncommitted` × 2 (codex-cli 0.128.0, 각 < 300s)
판정: High/Critical 반영 후 진행, P2 이하 참고

## R1: PRD 단계

| Priority | 항목 | 처리 |
|:--------:|------|------|
| P2 | `.claude/settings.json` knowledge-skills marketplace 미선언 | **defer (범위 외)** — 본 PR 시작 전 working tree |
| P2 | "Claude r1(PRD)·r2(코드)" 표현이 단일 meta 리뷰 규칙과 충돌 가능성 | **반영 완료** — `meta` 타입 r{N} 회차 패턴(r1=PRD, r2=post-edit)로 명시 |

## R2: 코드(문서) 단계

| Priority | 항목 | 처리 |
|:--------:|------|------|
| P2 | `.claude/settings.json` marketplace 미선언 (재지적) | **defer (범위 외)** |
| P2 | `harness-workflow.md` "프로젝트 유형 QA 분기" 표 → SSOT 링크 변경 후, SSOT(`harness-qa.md`) 표에 "코드리뷰" 컬럼 부재 → 콘텐츠 프로젝트가 7항목 코드리뷰로 잘못 라우팅될 위험 | **반영 완료** — `harness-qa.md` 표에 `코드리뷰` 컬럼 추가 (코드 프로젝트=7항목, 콘텐츠=빌드 검증만) |

High/Critical 0건. SSOT 보강 후 일관성 회복. 다음: ship.
