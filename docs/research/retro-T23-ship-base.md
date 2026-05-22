# 회고 발췌 — rp-ship PR base 자동 결정

| 항목 | 값 |
|------|----|
| 출처 | 서비스 프로젝트 회고 (태스크 ID·PR 번호·브랜치명은 서비스 레포 회고 참조) |
| 시점 | 2026-04-22 |
| 발췌 목적 | 하네스 메타 변경 근거 — `rp-ship` PR base 자동 결정 로직 추가 |

## 발견된 일반 패턴

| # | 심각도 | 패턴 |
|:-:|:-----:|------|
| I1 | High | **rp-ship이 PR base 자동 결정 안 함** — `tasks.md` 또는 `CLAUDE.md`에 통합 브랜치 선언이 있음에도 `gh pr create` 기본값(레포 default branch = `main`)에 의존. 사용자 명시 지시 없으면 통합 브랜치 우회해 `main` 직행 가능 |
| I2 | High | **통합 브랜치 전략 유명무실화** — 통합 브랜치 선언 vs 실제 운영(`main` 직행) 괴리. 통합 브랜치가 `main`보다 뒤처진 stale 상태로 방치 → 규칙과 운영의 불일치 누적 |

## 채택된 개선 제안

| # | 유형 | 제안 |
|:-:|------|------|
| 1 | rp-ship 개선 | **PR base 자동 결정 로직** — `rp-ship` 스킬에 "`tasks.md` 또는 `CLAUDE.md`의 통합 브랜치 선언을 읽어 PR base 자동 지정" 단계 추가. 선언 없으면 `main`. 실수 방지 |
| 2 | 규칙 재정의 | **통합 브랜치 전략 유명무실화 해소** — 선택 (a) 통합 브랜치 폐지하고 task → `main` 직행을 공식화 (b) 통합 브랜치 강제 + `rp-ship`에서 base 자동 fast-forward. 실제 운영이 (a)이면 `tasks.md` 업데이트가 현실적 |

## 교차 분석

I1 (PR base 자동화 부재) ↔ I2 (통합 브랜치 유명무실화)는 **같은 뿌리**: 선언과 실제 운영의 괴리를 `rp-ship` 자동화가 감지·보정하지 못함. #1 + #2 함께 반영 권장.

→ 반영 위치: [`../skills/rp-ship.md`](../skills/rp-ship.md) PR base 감지 절차
→ 관련 PRD: [`../prd/harness-ship-auto-base/prd.md`](../prd/harness-ship-auto-base/prd.md)
