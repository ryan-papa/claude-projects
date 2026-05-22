# 회고 발췌 — AOP self-invocation + 동시성 체크리스트

| 항목 | 값 |
|------|----|
| 출처 | 서비스 프로젝트 회고 (태스크 ID·서비스명은 서비스 레포 회고 참조) |
| 시점 | 2026-04-22 |
| 발췌 목적 | 하네스 메타 변경 근거 — `harness-code-review.md` 정확성 섹션 강화 |

## 발견된 일반 패턴

| # | 심각도 | 패턴 |
|:-:|:-----:|------|
| C1 | Med | **AOP self-invocation 블라인드 스팟** — 같은 빈의 `@Transactional`·`@Cacheable`·`@CacheEvict` 메서드를 `this.xxx()` 로 호출하면 Spring AOP 프록시 우회. 정적 코드 리뷰 체크리스트 부재 시 1차 리뷰가 탐지 못함 |
| C2 | Med | **동시성 검증 테스트 공백** — `read-decide-write` 경로(집계 → 판정 → write)에서 두 트랜잭션이 서로의 insert 를 못 봐 결과 누락 레이스. 단위 테스트(mock) 만으로 검출 불가, 통합 테스트·명시적 락 패턴 필요 |

## 채택된 개선 제안

| # | 제안 | 유형 |
|:-:|------|:---:|
| 1 | [규칙 추가] **코드 리뷰 체크리스트에 "프록시 기반 self-invocation 금지" 항목 명시** — `@Transactional`·`@Cacheable`·`@CacheEvict` 메서드가 같은 빈의 다른 @-어드바이스 메서드를 `this.xxx()` 로 호출하면 FAIL 조건 고정 | 규칙 강화 |
| 2 | [규칙 추가] **동시성 테스트 또는 명시적 포기 메모 요구** — 쓰기 경로가 "read-decide-write" 구조일 때 레이스 방지 방식(락·atomic SQL·이벤트)을 PR body 에 명시. 단위 테스트로 대체 불가 영역 | 규칙 강화 |

## 핵심 학습

**Claude ⊕ Codex 이중 리뷰의 결정적 가치** — 1차 리뷰가 Medium 2건 제시했지만 추가 리뷰가 P1 2건(AOP 우회·동시성 레이스)을 새로 발견. "Codex 추가 리뷰 1회 필수" 규칙 실효성 입증. 향후 유사 패턴 사전 차단 위해 체크리스트 명문화 필요.

→ 반영 위치: [`../harness-code-review.md`](../harness-code-review.md) 정확성 섹션 (`1a. 동시성 필수 체크리스트` + `1b. 프록시 기반 self-invocation 금지`)
→ 관련 PRD: [`../prd/harness-review-aop-concurrency-checklist/prd.md`](../prd/harness-review-aop-concurrency-checklist/prd.md)
