# Harness: 산출물 보고 + 배포

본 문서는 **정책·이유** SSOT. 실행 절차(커밋·PR·CI·머지·README 검증·머지 후 검증)는 [`skills/rp-ship.md`](skills/rp-ship.md) SSOT.

## 산출물 보고 → 사용자 승인

QA + 코드리뷰(또는 콘텐츠 검수) 통과 후 사용자에게 결과를 보고한다.

**보고 포맷:**
- QA 결과 요약 (통과/실패 항목)
- 코드리뷰 점수 (7항목 또는 빌드 검증)
- 변경 파일 목록 + 줄 수 변화
- "승인 시 커밋→PR→배포를 진행합니다" 안내

**⛔ 사용자 승인 없이 배포(머지) 진행 금지** — 커밋·PR은 산출물 보고 후 자동 수행

## 커밋 → PR → 배포 (실행 절차)

→ [`skills/rp-ship.md`](skills/rp-ship.md) "절차" 섹션 SSOT (사전 체크 게이트, PR base 결정, 자동 머지 가드 4종 AND, 비상 탈출구 포함)

## CI 정책

| 규칙 | 내용 |
|------|------|
| CI 없을 때 | PR 생성 시점에 "CI가 없습니다. 추가할까요?" 추천 |
| 추가 여부 | 사용자 선택 시에만 추가 |
| CI 없이 머지 | 사용자에게 수동 머지 안내 |
| **동일 브랜치 재작업** | 기존 PR이 MERGED/CLOSED면 **신규 PR 생성** (OPEN PR만 재사용) |

> CI 통과 전 머지 금지 · `--admin`/`--no-verify` 우회 금지 · 자동 머지 가드 4종 AND · 리뷰 증거 게이트 · PR base 자동 감지(fail-closed) → 본문은 [`harness-absolute-rules.md`](harness-absolute-rules.md) SSOT 참조

## 푸시 전 README 검증 (정책)

모든 push 전 README.md 가 프로젝트 현 상태와 일치해야 한다.

| 항목 | 정책 |
|------|------|
| 검증 5항목 | 프로젝트 개요·디렉토리 구조·설치/실행·기능 목록·기술 스택 |
| 통과 기준 | 평균 ≥ 8.0 |
| 실패 시 | README 수정 → 재검증 (최대 3회) → 3회 실패 시 사용자 판단 위임 |
| 미통과 효과 | **push 차단** |

→ 실행 절차: [`skills/rp-ship.md`](skills/rp-ship.md) "README 검증" SSOT

## 머지 후 정책

- 배포 완료 메시지 직후 `/rp-retro` 자동 진입 **금지** ([`harness-absolute-rules.md`](harness-absolute-rules.md): 회고는 사용자 명령 시에만)
- 배포 완료 시 라이브 URL 사용자에게 전달

→ 실행 절차: [`skills/rp-ship.md`](skills/rp-ship.md) "머지 후 검증" SSOT

## 최종 산출물

| 항목 | 기준 |
|------|------|
| 코드 | 실행 가능, 테스트 통과 |
| README | 간결 명료, 링크 참조 방식, **구조 변경 시 동기화 필수** |
| PRD | Open Issues 업데이트 |

→ 스킬 상세: [`skills/rp-ship.md`](skills/rp-ship.md)
