# 간소 PRD: 인프라 리뷰 후속 정합 보정

**유형:** 하네스 메타 변경

## 변경 이유

PR #63(인프라 리뷰 신설 + 리뷰 병렬화) 머지 후 2축 리뷰(코드 관점 ∥ PRD↔구현 정합 감사)에서 발견된 결함을 보정한다. 판정 규칙 본체는 충실히 반영됐으나 일부 잔존 문구와 실행 우선순위가 정합하지 않다.

| # | 결함 | 영향 |
|:-:|------|------|
| P5 | 실인프라 접근 위반 시 "결과 폐기 + 재발사"에 **상한 없음** | 반복 위반 시 무한 재발사·토큰 소진이 규칙상 허용 |
| P6 | `AGENTS.md` 부분 동기화 — 관점 분리는 4축인데 "필수 워크플로우" 번호 리스트·"세 리뷰 단계" 표현은 3축 잔존 | **Codex-Lead 가 인프라 축을 건너뜀** (지원 런타임 2종 중 1종 게이트 실효 상실) |
| P7 | `CLAUDE.md`·`README.md` 의 "통과 판정은 **점수** 단일 기준" 잔존 | 무점수 인프라 축과 모순 (SSOT 는 "판정 단일 기준"으로 이미 갱신됨) |
| P8 | `harness-ship.md` 산출물 보고 포맷이 "QA + 코드리뷰"에 머묾 | 인프라 판정 미보고 |
| P9 | 잔존 단일축 표현 — `harness-absolute-rules.md` ship 자동 진입 문구, `rp-qa.md` 자동 전환, 리뷰 구성 표 | 병렬 모델과 불일치 |
| P10 | `rp-eng-review.md` 가 병렬 쌍인 `rp-plan-review.md` 대비 축약("실패 축 생략 통과 금지"·"재검증 미달 시 다음 사이클 전환" 누락) | 두 축의 규칙 강도 비대칭 |
| P11 | 발사 조건 교집합(메타 변경 ∩ Lite) 우선순위 미정의 / "미해결 ASK" 정의 부재 / 임계값 오버라이드 발견 경로 없음 | 판정 모호 |
| P12 | CI soft 300 경고가 평문 `echo` 라 로그에 묻힘, 링크 게이트·필수 파일 목록은 범위 미확대 | 경고 실효 없음, 신규 링크 CI 미검증 |
| P13 | PR #63 PRD 의 영향 파일 문구가 구현과 반대(`docs/skills/` 전용 예외), 줄 인용 1건 stale | 문서 이력 부정확 |

## 대안 탐색

| 대안 | 판단 | 이유 |
|------|------|------|
| A. 좁은 정합 보정 | **채택** | 인프라 리뷰 본체는 유지하고 잔존 3축 문구·skip 우선순위·재발사 상한만 맞춘다 |
| B. 현상 유지 | 기각 | Codex-Lead 경로와 README가 서로 다른 게이트를 안내한다 |
| C. 인프라 축 롤백 | 기각 | 이미 도입된 코드 프로젝트 게이트를 후속 문구 결함 때문에 되돌리는 것은 과하다 |
| D. 전체 워크플로우 재설계 | 기각 | 이번 결함은 문서 정합 보정으로 해결 가능하다 |

**판단 근거 유형:** 엔지니어 운영 판단. 변경 표면을 최소화하고, 새 절차를 추가하지 않는다.

## 영향 파일

| 파일 | 변경 |
|------|------|
| `docs/harness-infra-review.md` | 실인프라 접근 위반 **재발사 상한 2회** 후 사용자 보고 (P5). 발사 조건 우선순위(메타 skip > Lite 필수) + "미해결 ASK" 정의(사용자 강행 = 해결) (P11) |
| `docs/skills/rp-infra-review.md` | 위반 재발사 상한, Lite/meta 우선순위, 임계값 오버라이드 발견 경로 1줄 (P5·P11) |
| `docs/skills/rp-ship.md` | 인프라 축 정당한 skip 이수 확인만 유지 |
| `docs/harness-ship.md` | 산출물 보고 포맷에 인프라 판정(BLOCK/ASK/WARN 건수) 추가 (P8) |
| `AGENTS.md` | 필수 워크플로우 번호 리스트에 인프라 축 추가, "세 리뷰 단계"→"네 리뷰 축", 기준 문서 목록에 `harness-infra-review.md` 등재 (P6) |
| `CLAUDE.md` · `README.md` | "점수 단일 기준" → "판정 단일 기준(코드 점수 + 인프라 판정)" (P7) |
| `docs/harness-absolute-rules.md` | ship 자동 진입 문구 양축 표기, 리뷰 구성 표 축 구분 (P9) |
| `docs/skills/rp-qa.md` | 자동 전환에 `∥ /rp-infra-review` 표기 (P9) |
| `docs/skills/rp-eng-review.md` | `rp-plan-review.md` 와 동일 강도로 절차 5~7 보정 (P10) |
| `.codex/skills/rp-*/SKILL.md` | `docs/skills/` 원본 변경의 Codex 변환본 동기화 |
| `.github/workflows/ci.yml` | soft 경고를 `::warning file=` 어노테이션으로, 링크 게이트·필수 파일 목록 범위 확대 (P12) |
| `docs/prd/harness-infra-review-parallel/prd.md` | 영향 파일 문구를 구현과 일치하도록 정정 + stale 줄 인용 보정 (P13) |

## 롤백 전략

`git revert <merge-commit>` — 규칙 문서·스킬·CI 게이트 정합 변경을 한 번에 되돌린다. PR #63 의 규칙 본체는 건드리지 않는다.

## 검증

- [x] 불필요한 추가 기록 의무가 실행 규칙 문서와 스킬에 남아 있지 않음
- [x] 실인프라 접근 위반 재발사 상한이 3개 문서(SSOT·스킬·absolute-rules)에 일관 기재
- [x] `AGENTS.md` 전문에서 "세 리뷰"·3축 열거 0건, 필수 워크플로우에 인프라 축 포함
- [x] `CLAUDE.md`·`README.md` 에 "점수 단일 기준" 문구 0건
- [x] `rp-eng-review.md` 절차 5~7 이 `rp-plan-review.md` 와 동일 강도
- [x] `ci.yml` soft 경고가 `::warning file=` 형식, 링크 게이트 범위에 `AGENTS.md`·`README.md` 포함
- [x] `python3 scripts/sync-codex-skills.py --check` 통과 + 내부 링크 broken 0건 + 500줄 초과 0건

## 운영자 검증 게이트

- 다음 하네스 메타 PR 1회에서 운영자가 `README.md`·`AGENTS.md`·`rp-infra-review`만 보고 [9] 코드∥인프라 축, 메타 변경 skip, Lite 우선순위를 혼동 없이 판단 가능한지 확인한다.
- 실패 시 같은 후속 정합 보정 범위로 문구만 수정한다.
