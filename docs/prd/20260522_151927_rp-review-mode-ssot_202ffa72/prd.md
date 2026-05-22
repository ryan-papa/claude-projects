# PRD: 작성 모드 및 리뷰 매트릭스 SSOT 신설

**ID:** 20260522_151927_rp-review-mode-ssot_202ffa72
**유형:** 하네스 메타 변경
**브랜치:** feat/rp-review-mode-ssot
**작성일:** 2026-05-22

## 변경 이유

Codex 도입 후 리뷰 단계 운용에서 모호함 누적:

1. **리뷰 단계 채점 주체 불명확** — `harness-absolute-rules.md` "[리뷰 단계 서브에이전트 필수]" 절은 Claude 서브에이전트 가정. Codex CLI 진입 시 동일 절차가 어떻게 매핑되는지 SSOT 미정의
2. **메인 셀프 리뷰 가드 흩어짐** — 셀프 채점 금지 문구가 각 스킬에 산재. 양 런타임 공통 규칙으로 단일 출처화 필요
3. **3회 미달 시 동작 불일치** — 스킬별로 "사용자 보고", "Doc Agent 재실행", "추가 사이클 여부 확인" 등 표현 상이. 결정 권한 주체 모호
4. **Codex 추가 리뷰 적용 범위 불명** — `harness-codex-review.md`가 Claude-led 가정으로만 작성. Codex-led에서 추가 Codex 리뷰는 의미 없으나 명시 없음
5. **AGENTS.md ↔ docs/ 표현 비대칭** — "Claude-authored / Codex-authored" 어휘가 SSOT 매트릭스와 정합 미보장

본 PR로 작성 모드 = 메인 런타임 정의 + 리뷰 구성 매트릭스 + 메인 셀프 리뷰 절대 금지 + 단계당 3회 한도 + 3회 미달 사용자 결정 요청을 단일 SSOT에 명문화.

### 대안 비교 (메인 셀프 리뷰 가드 산재 해소 — 대안 3안)

| # | 대안 | 장점 | 단점 | 결과 |
|---|------|------|------|------|
| A | 분산 본문 유지 + lint 스크립트로 정합 강제 | 기존 스킬 파일 구조 보존 | lint 의존·CI 통과까지 가드 지연, SSOT 단일 검색 불가 | 기각 |
| B | 스킬별 본문 중복 + 자동 동기화 hook | 각 스킬이 자체 완결 (오프라인 가독성) | 동기화 누락 시 모순 잠복, Codex-led 변환본 중복 폭증 | 기각 |
| C | SSOT 단일화 (`harness-absolute-rules.md`) + 스킬은 참조만 | 단일 검색·단일 갱신·중복 0, 양 런타임 동일 진실원 | 스킬 단독 가독성 일부 손실 | **채택** |

판단 근거: 엔지니어 선호 (단일 진실원 + 중복 0이 운용 안전성 최상).

## 영향 파일

| 파일 | 변경 |
|---|---|
| `docs/harness-absolute-rules.md` | **신설** "작성 모드 및 리뷰 매트릭스" SSOT 절 + 기존 "[리뷰 단계 서브에이전트 필수]"·"[4][5][9] Codex 추가 리뷰" 절을 SSOT 참조 형태로 정합 |
| `docs/harness-workflow.md` | Flow 다이어그램에 Claude-led 분기 표기, "자동 진행 규칙" 표를 모드 일반화 + 3회 미달 사용자 결정 요청 |
| `docs/harness-codex-review.md` | 적용 조건 = Claude-led 모드 한정 명시 (Codex-led는 본 문서 N/A) + 절대 규칙 첫 줄 동일 |
| `docs/skills/rp-plan-review.md` | 작성 모드 표 + 절차 헤더 "Claude-led 기준" + Codex-led 시 외부 Codex 호출 생략 안내 + 3회 미달 사용자 결정 |
| `docs/skills/rp-eng-review.md` | 동일 패턴 적용 |
| `docs/skills/rp-code-review.md` | 동일 + Codex 추가 리뷰 절 헤더에 "Claude-led 전용" 명시 + Codex-led N/A 안내 |
| `AGENTS.md` | 기본 운영 표를 모드 매트릭스로 교체 + 작성 모드 절(Claude-led / Codex-led) 정합 + 기본 판단 절 갱신 |
| `CLAUDE.md` | Harness Engineering 절에 작성 모드 한 줄 + 표 갱신 + "메인 셀프 리뷰 절대 금지" 보조 문장 |
| `scripts/sync-codex-skills.py` | `Agent 툴+Codex 서브에이전트` 모순 한 줄 해소용 매핑 추가 (1줄) |
| `.codex/skills/rp-{plan,eng,code}-review/SKILL.md` | sync-codex-skills.py 재생성 변환본 (수동 편집 X) |
| `docs/harness-prd.md` | [4][5] 리뷰 실행 주체 표현을 "Agent 툴" 강제 → "메인 런타임의 서브에이전트(Claude-led=Agent 툴 / Codex-led=`spawn_agent`)"로 일반화 + 3회 미달 사용자 결정 요청 명시 — 1차 리뷰 Codex P2 #3 반영 |
| `docs/harness-code-review.md` | 재시도 분기 "3회 후 7.0~8.0 → 통과 처리"를 SSOT 정합 — 3회 미달 시 자동 중단 + 사용자 결정 요청, 임계 완화 금지 — 1차 리뷰 Codex P2 #1 반영 |
| `docs/skills/rp-code-review.md` | 동일 분기 정합 (3회 후 7.0~8.0 통과 처리 제거) — 1차 리뷰 Codex P2 #1 반영 |
| `docs/skills/rp-plan-review.md`·`rp-eng-review.md` | 절차 본문을 "양 모드 공통"으로 재작성 — 단계 1(a) 서브에이전트는 모드별 호출 도구 명시 / 단계 1(b) Bash `codex review`는 Claude-led 전용 명시 / 단계 5 미달 시 지적 반영 분기를 모드별 분리 / 단계 7 Codex 비-스킵 종료는 Claude-led 한정. 자동 전환 절도 모드별 분기로 재작성 — 1차·2차 리뷰 Codex P2 #2 반영 |
| `docs/skills/rp-workflow.md` | [4][5][9] Codex 추가 리뷰가 Claude-led 한정임을 본문 + 단계 표에 명시 — 2차 엔지 리뷰 Codex P2 #1 반영 |
| `docs/skills/rp-code-review.md` "⛔ 실행 주체 (서브에이전트 필수)" 절 | Claude 한정 표현 → 양 모드 공통(Claude-led=Agent 툴 / Codex-led=`spawn_agent`)으로 재작성. 자동 전환 절도 모드별 분기 — 2차 엔지 리뷰 Claude High #1 반영 |
| `docs/harness-code-review.md` "⛔ 실행 주체" 절 | "Agent 툴" 강제 → 메인 런타임 서브에이전트로 일반화 + SSOT 참조 — 2차 엔지 리뷰 Codex P2 #3 반영 |
| `scripts/sync-codex-skills.py` 매핑 | `"Doc Agent": "문서 수정"` 단순 치환을 다단계 정밀 매핑("문서 작성자 재투입"/"문서 작성자 재작성"/"문서 작성자")으로 교체. `Dev Agent`도 동일 정밀화. 변환본 비문("문서 수정가 PRD 본문 갱신") 해소 — 2차 엔지 리뷰 Claude High #2 반영. 추가: `"subagent_type=general-purpose"` 일률 치환 + `"Agent 툴**(Claude 서브에이전트, agent_type="` 보조 매핑 + `"Claude+Codex": "Codex-led"` 일률 치환 3개 **제거** — 양 모드 공통 표현(`Claude-led=Agent 툴 subagent_type=general-purpose / Codex-led=spawn_agent`)이 Codex 변환본에서도 SSOT 매트릭스 그대로 보존되도록. 1차 코드 리뷰 Codex P2 #1 + Claude Mid #1 반영 |

`repositories/[project]/` 산하 0 변경. 코드 동작 영향 0. 도구·런타임 동작 변경 0 (문서·스킬 정합만).

## 롤백 전략

본 PR 자체 revert. SSOT 절·매트릭스가 사라지고 기존 분산된 표현으로 복귀 — 운용은 가능하나 (a) Codex-led 진입자가 SSOT 매트릭스 부재로 재차 모호함 인지, (b) 메인 셀프 리뷰 가드 분산. 머지된 후속 PR이 SSOT를 참조하기 전까지 revert 안전 — 라이브 영향 0 (런타임 동작 미변경).

## 검증

| 항목 | 방법 | 통과 기준 |
|------|------|----------|
| 문서 동기화 | `rtk python3 scripts/sync-codex-skills.py --check` | `Codex skills in sync: 13` |
| 양 모드 매트릭스 보존 | `rtk grep -nE "Claude-led=Agent 툴 \\\`subagent_type=general-purpose\\\`" .codex/skills/rp-{plan,eng,code}-review/SKILL.md` | 3건 매칭 (변환본도 SSOT 양 모드 표현 보존) |
| SSOT 단일 출처 | `rtk grep -nE "^## 작성 모드 및 리뷰 매트릭스" docs/harness-absolute-rules.md` | 1건 매칭 |
| 셀프 리뷰 금지 | `rtk grep -n "메인 셀프" docs/harness-absolute-rules.md docs/skills/rp-{plan,eng,code}-review.md` | 4파일 모두 매칭 |
| 3회 미달 사용자 결정 | `rtk grep -n "사용자 결정 요청" docs/harness-absolute-rules.md docs/harness-workflow.md docs/skills/rp-*-review.md` | 모든 리뷰 스킬 + SSOT + workflow 매칭 |
| Claude-led 한정 명시 | `rtk grep -n "Claude-led 모드 한정\|Claude-led 전용" docs/harness-codex-review.md docs/skills/rp-code-review.md` | 양쪽 매칭 |
