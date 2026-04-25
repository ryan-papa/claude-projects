# Meta Review (r1) — 하네스 백엔드 테스트 정책

**리뷰어:** 독립 서브에이전트
**대상:** PRD + harness-backend-test-policy.md + 루트 CLAUDE.md + museum-finder/CLAUDE.md

## 점수

| # | 축 | 점수 | 근거 |
|:-:|---|:-:|---|
| 1 | 문제 정의 | 8 | #51/#59/#60 매핑 정확. #51 가설 표현 약함 |
| 2 | 4-게이트 명료성 | 9 | 트리거·도구·통과 기준 표 1:1 |
| 3 | 적용 범위 | 8 | Spring 한정 명시. 신규 프로젝트 자동 합류 트리거 부재 (-1) |
| 4 | 비용 vs 차단력 | 8 | EnabledIfEnvironmentVariable + 3초 부팅. CI skip 시 로컬 실행 보장 권고 누락 |
| 5 | 회귀 가드 4-stroke | 7 | 핫픽스 증거 저장 위치 모호 → r1 반영 |
| 6 | 예외 조항 | 7 | test-only/Spring 마이너/단축 한계 모호 → r1 반영 |
| 7 | 문서 정합성 | 9 | 루트·프로젝트 정합 강. 사례 범위 살짝 어긋남 |
| 8 | 운영 위반 처리 | 8 | 자동 게이트 봇 부재 명시 권고 |

**최저 7 — PASS.**

## 필수 반영 (r1 처리 완료)

- ✅ 핫픽스 4-stroke 증거 저장 위치 — qa-smoke.md OR PR description 한 곳 고정
- ✅ 핫픽스 단축 한계 — OAuth 응답 캡처만 면제, health/Flyway 유지
- ✅ test-only 변경 면제 행 추가
- ✅ Spring Boot 마이너 업그레이드 게이트 4 의무 명시

## 권고 (후속 PR)

- PRD §1 #51 회고 인용 강화
- 정책 §2.3 에 CI skip 시 로컬 실행 캡처 PR 체크리스트
- §6 사례 목록·PRD §1 사례 범위 일치
- 운영 위반 처리 자동화 훅 명시
- 신규 백엔드 프로젝트 자동 합류 (rp-init build.gradle.kts 검출)
