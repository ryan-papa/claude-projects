# PRD: 교차 런타임 추가 리뷰 제거 + `-Lead` 표기 통일 + [9] 재시도 5회 + 문서 300줄 한도

**유형:** 하네스 메타 변경
**브랜치:** feat/drop-cross-runtime-review
**작성일:** 2026-08-03

## 변경 이유

### 1) 외부 Codex 추가 리뷰 전면 제거 — "런타임 = 리뷰어"

현재 규칙은 비대칭이다. Claude-Lead 로 진입하면 Claude 서브에이전트 채점에 **더해** 외부 Codex(`/codex:review`)를 [4]·[5]·[9] 마다 1회씩 추가 호출한다. Codex-Lead 로 진입하면 추가 리뷰가 없다.

| 항목 | 현재 (비대칭) | 변경 후 (대칭) |
|------|-------------|--------------|
| Claude-Lead | Claude 서브에이전트 + **외부 Codex 1회** | Claude 서브에이전트만 |
| Codex-Lead | Codex 서브에이전트만 | Codex 서브에이전트만 |

**원칙:** 진입한 런타임이 채점 주체다. 클로드로 들어오면 클로드가, 코덱스로 들어오면 코덱스가 리뷰한다. 교차 호출 없음.

제거 근거:

| 사유 | 내용 | 근거 |
|------|------|------|
| 규칙 복잡도 | 스킬 3개가 모드별 2분기 + SKIP 6분기 매트릭스 + cwd 저장·복귀 절차를 이중 서술 | 삭제 대상 `harness-codex-review.md` 160줄 |
| 외부 의존 | ChatGPT 구독·`codex login`·Node 런타임·플러그인 설치가 리뷰 게이트 전제조건 | README 전제조건 표에 실재 |
| **수율** | 과거 Codex 추가 리뷰 산출물 **29건 중 10건이 "High/Critical 0건"** 명시. 실질 지적 대부분 P2 이하, High 지적 다수는 codex-review 기능 자체에 대한 메타 변경 건 | `docs/prd/*/review-codex-*.md` 집계 |
| 실패 표면 | 토큰 한도·rate limit·hang(300초)·네트워크 오류가 워크플로우 중단 사유로 유입 | **관측 기록 0건** — 잠재 리스크로만 계상 |
| 비대칭 | 동일 변경이 진입 런타임에 따라 다른 검증 강도를 받음 | 리뷰 매트릭스 |

**유지되는 것:** `Claude-Lead` / `Codex-Lead` 모드 구분 자체와 각 런타임의 서브에이전트 채점([4]·[5]·[9] 3관문)은 그대로다. 메인 셀프 채점 금지도 유지된다.

**포기하는 것 + 재평가 조건:** 이중 관점 검증을 잃는다. 이 리스크를 인상이 아닌 관측으로 다루기 위해 [`harness-absolute-rules.md`](../../harness-absolute-rules.md) §리뷰 구성에 상시 앵커를 넣는다.

> [9] 통과 후 머지된 변경이 **롤백·핫픽스를 유발한 사례 3건 누적 시** 교차 리뷰 재도입 여부 재검토. 발생 시 `docs/research/` 에 1줄 기록. 기록 없이 재도입 금지.

### 2) `-led` → `-Lead` 표기 통일

`Claude-led`·`Codex-led` 를 `Claude-Lead`·`Codex-Lead` 로 변경 (대문자 `L`). 활성 문서 12개 · 110건.

**판단 근거 유형:** 사용자 명시 지정. 대안은 (a) 현행 `-led` 유지 (b) `-Lead` (채택) (c) 한글 "클로드 주도"·"코덱스 주도" — (c)는 코드·프론트매터·영문 어댑터(`AGENTS.md`)와 혼용되어 기각, (a)는 사용자 지정으로 기각. 순수 표기 변경이라 동작 영향 0.

### 3) [9] 코드 리뷰 재시도 한도 3회 → 5회

코드 리뷰는 PRD 리뷰([4]·[5])와 달리 실제 코드 수정을 동반해 회차당 수렴 폭이 작다. [9]만 5회로 완화, [4]·[5]는 3회 유지.

**판단 근거 유형:** 사용자 명시 지정 — **관측 데이터 없음**. `docs/research/` 에 [9] 3회 소진 사례 기록 0건.

정량 근거 없는 SSOT 임계 완화이므로 **재평가 조건을 문서에 상시 앵커로 고정**한다. 회고[12]는 사용자 명시 명령 시에만 실행되므로(SSOT §단축 경로·예외) 재평가 트리거를 회고에 의존하면 영구 미실행될 수 있다. 따라서 [`harness-code-review.md`](../../harness-code-review.md) 재시도 절에 다음을 본 PR 범위로 포함한다.

> 5회는 **잠정값**. **4회 이상 소진 사례 3건 누적 시 적정성 재검토** — 소진 발생 시 회차·사유를 `docs/research/` 에 1줄 기록. 기록 없이 추가 완화 금지.

| 대안 | 요약 | 기각/채택 사유 |
|---|---|---|
| A | 4회 | 3회 대비 체감 차이가 작아 재조정 요청이 재발할 여지 — 기각 |
| B | **5회** | 사용자 지정값. 코드 수정 동반 수렴 특성에 여유 확보 — **채택** |
| C | 무제한 + wall-clock 한도 | 중단 조건이 시간에 종속돼 예측 불가, 토큰 폭증 — 기각 |
| D | 실패 유형별 차등 (테스트 실패 5회 / 설계 지적 3회) | 유형 판별이 주관적 → 게이트 우회 구실 제공 — 기각 |

**"왜 5인가":** 정량 근거 아님. "3회는 부족, 무제한은 위험" 사이의 실용 타협이며 위 재평가 앵커로 보정한다.

### 4) 문서 분량 한도 200줄 → 300줄

`CLAUDE.md` §Document Style 의 "파일당 200줄 이하" 를 300줄로 상향. 초과 시 대응(역할/책임 단위 파일 분리)은 그대로 유지.

**관찰 근거:** 직전 커밋 `08352aa` "docs(harness): rp-ship.md 200줄 한도 준수 (CI 분기 중복 제거·절차 압축)" — 한도를 맞추려 **내용을 압축한 실제 사례**. 현재 `docs/skills/rp-ship.md` 는 정확히 200줄로 한도에 붙어 있어, 다음 변경 때 또 압축 압력을 받는다. 200줄은 한 책임을 다루는 문서를 인위적으로 쪼개거나 압축하게 만든다.

| 대안 | 요약 | 기각/채택 사유 |
|---|---|---|
| A | 200줄 유지 | 위 압축 압력 지속 — 기각 |
| B | **300줄** | 여유를 주되 무한 비대화는 계속 차단. 사용자 지정값 — **채택** |
| C | 한도 폐지 | SSOT 문서 비대화 방치 → 탐색성 저하 — 기각 |
| D | 문서 종류별 차등 (SSOT 300 / 스킬 150) | 분류 경계 논쟁을 새로 만듦 — 기각 |

**판단 근거 유형:** 사용자 명시 지정 + 관측 사례 1건(`08352aa`).

### 4건을 한 PR로 묶는 근거

| 변경 | 성격 | 묶음 사유 |
|:---:|------|----------|
| 1 | 구조 변경 (주) | 본 PR 의 본체 |
| 2 | 리네이밍 | 변경 1이 `-led` 표기를 담은 문서 12개를 전부 건드림. 분리하면 동일 파일에 연속 충돌 발생 |
| 3 | 임계 완화 | 변경 1이 리뷰 스킬·SSOT 재시도 절을 이미 재작성 — 같은 문단을 두 번 고치는 것을 회피 |
| 4 | 임계 완화 | 변경 1~3이 문서 분량을 늘려 200줄 한도에 근접 (`harness-absolute-rules.md` 등). 한도 상향이 선행되지 않으면 본 PR 안에서 압축 압력 발생 |

**분리 가능성:** 3·4는 독립 revert 가능 (롤백 표 참조). 1·2는 동일 문단을 공유해 분리 revert 불가.

### 대안 탐색 — 변경 1 (Codex 리뷰 처리)

| 대안 | 요약 | 장점 | 단점 | 결과 |
|---|---|---|---|---|
| A | 현행 유지 | 이중 검증 유지 | 위 4개 사유 그대로 | 기각 |
| B | 로컬 설정 토글로 on/off | 되돌리기 쉬움 | 두 경로를 모두 문서화·유지해야 해 복잡도가 오히려 증가. 기본 OFF면 죽은 규칙이 상시 잔존 | 기각 |
| C | **전면 제거 (런타임=리뷰어)** | 규칙 대칭·외부 의존 제거·실패 표면 축소 | 이중 관점 검증 상실 | **채택** |
| D | Codex/Codex-Lead 개념까지 전부 삭제 | 최대 단순화 | Codex CLI 진입 경로 자체가 사라짐 — 사용자 의도와 불일치 | 기각 |

**판단 근거 유형:** 사용자 명시 지정 ("클로드로 하면 클로드 리뷰, 코덱스로 하면 코덱스 리뷰").
**선택 근거:** D 가 아니라 C 인 이유 — 사용자는 Codex **진입 경로**는 유지하고 **교차 호출**만 없애기를 요구했다.

### 비-스코프

| 항목 | 사유 |
|------|------|
| `Codex-Lead` 모드 · `.codex/skills/` · `AGENTS.md` · `sync-codex-skills.py` | 존치 — Codex CLI 진입 경로는 유지 |
| [4]·[5] 재시도 한도 | 3회 유지 |
| 7항목 채점 기준·통과 임계(평균 8.0 / 최저 7) | 변경 없음 |
| 과거 PRD 산출물(`docs/prd/*/review-codex-*.md`) | 이력 보존 — 삭제하지 않음 |
| **기존 설치된 Codex 플러그인·CLI·인증 잔존물** | **제거 불필요** — `settings.json` 선언만 제거. 로컬에 남은 플러그인·`codex` CLI·login 은 하네스 게이트 밖 수동 사용 자유 |
| **`settings.json` 의 선반영 로컬 변경** | 본 PR 착수 **이전부터** 워킹트리에 있던 미커밋 변경 — (a) `enabledPlugins` 에 `knowledge-skills@apps-in-toss-skills` 추가 (b) 최상위 키 순서 재배치. **커밋에 싣지 않는다** — 인덱스에는 HEAD 기준 codex 2블록 삭제만 스테이징하고 워킹트리 로컬 상태는 그대로 보존 (되돌리면 사용자가 쓰던 플러그인이 꺼짐). `knowledge-skills` 는 대응 `extraKnownMarketplaces` 선언이 레포에 없어 그대로 커밋하면 dangling 참조가 배포되므로 분리가 필수 |

### 수용 시나리오

| 시나리오 | 기대 동작 |
|---|---|
| Claude Code 진입 + [4] 리뷰 | Claude 서브에이전트 1개만 발사. `codex` 명령 미호출. 확인 질문 없음 |
| Claude Code 진입 + [9] 리뷰 | 위와 동일. 통과 시 `✓ [9] 코드리뷰 통과` 출력 후 산출물 보고[10] 자동 진입 |
| Codex CLI 진입 + [4]·[5]·[9] | `spawn_agent` 서브에이전트만. 추가 리뷰는 기존과 동일(원래 없음). **단 [9] 재시도 한도는 Claude-Lead 와 동일하게 3→5회 적용** |
| [9] 리뷰 4회차 미달 | 중단 없이 5회차 재투입 (기존 3회에서는 중단) |
| [9] 리뷰 5회차도 미달 | 자동 중단 + 사용자 결정 요청 (강행/재설계/중단). 임계 완화 금지 |
| 사용자가 수동으로 Codex 리뷰를 원함 | 하네스 게이트 밖에서 직접 실행. 워크플로우 규칙과 무관 |

## 영향 파일

| 파일 | 변경 |
|------|------|
| `docs/harness-codex-review.md` | **삭제** (외부 Codex 리뷰 SSOT 전체) |
| `docs/harness-absolute-rules.md` | 리뷰 매트릭스 외부 추가 리뷰 열 제거, `[4][5][9] Codex 추가 리뷰` 절 삭제, 재시도 한도 표([4][5]=3 / [9]=5), `-Lead` |
| `docs/skills/rp-plan-review.md`·`rp-eng-review.md` | Codex 병렬 발사·매트릭스 판정·비정상 종료 분기 제거, 절차 단일화, `-Lead` |
| `docs/skills/rp-code-review.md` | 위 + 재시도 5회 |
| `docs/skills/rp-workflow.md`·`rp-retro.md` | Codex 추가 리뷰 항목·스킵 트렌드 항목 제거 |
| `docs/harness-workflow.md` | 플로우 다이어그램·자동 진입 규칙에서 Codex 제거, [9] 5회 |
| `docs/harness-code-review.md` | 재시도 5회 + **잠정값 재평가 앵커 주석**, `-Lead` |
| `scripts/sync-codex-skills.py` | 제거된 기능을 참조하는 죽은 치환 규칙 정리 — **총 45→8 (37개 삭제)**: `replacements` dict 35→8 · `adapted.replace()` 블록 5→1 · `codex_description()` 6→0. 어댑터 노트에 교차 호출 금지 1줄 추가 |
| `CLAUDE.md`·`AGENTS.md`·`docs/harness-workflow.md`·`docs/tasks.md` | 문서 분량 한도 **200줄 → 300줄** |
| `docs/harness-prd.md` | 간소 PRD 검증 항목에서 Codex 제거, `-Lead` |
| `CLAUDE.md` | 구조 트리에서 `harness-codex-review.md` 제거, 리뷰 표·링크·재시도 갱신 |
| `AGENTS.md` | 외부 추가 리뷰 제거, 기준 문서 링크 제거, 재시도·`-Lead` |
| `README.md` | "이중 리뷰" 특징 → "런타임 = 리뷰어" 로 교체, 전제조건에서 ChatGPT 구독·Node.js **제거** + `Python 3` **추가**(스킬 동기화 스크립트 실행에 필요), `Codex 플러그인` 절 삭제, 구조 트리·12단계 표·작성 모드 표 갱신 |
| `.claude/settings.json` | `extraKnownMarketplaces.openai-codex` + `enabledPlugins.codex@openai-codex` 선언 제거 (**11줄 삭제만**). 선행 로컬 변경은 **부분 스테이징으로 분리** — 아래 비-스코프 |
| `.codex/skills/rp-*/SKILL.md` | `sync-codex-skills.py` 자동 변환본 |

## 롤백 전략

| 범위 | 방법 |
|------|------|
| 전체 | `git revert` — 문서·설정 변경만이라 코드 동작 영향 없음 |
| Codex 리뷰만 복구 | (1) revert 로 `harness-codex-review.md` 복원 (2) `settings.json` 플러그인 재선언 (3) **`sync-codex-skills.py` 를 파일 단위로 revert** — 삭제분이 dict 27 + 블록 4 + description 6 으로 3곳에 흩어져 있어 개수를 세어 복원하면 누락 위험. 부분 복원 시 `.codex/skills/` 재생성 결과가 깨짐 (4) `/codex:setup` 재실행 (5) 완료 조건: `python3 scripts/sync-codex-skills.py --check` PASS |
| 재시도 한도만 원복 | `[9] 5회` 를 서술하는 **원본 8개 파일** 전부 5→3 동기화 — `harness-absolute-rules.md`·`harness-code-review.md`·`harness-workflow.md`·`skills/rp-code-review.md`·`skills/rp-workflow.md`·`CLAUDE.md`·`AGENTS.md`·`README.md`. 이후 `sync-codex-skills.py` 재실행으로 `.codex/skills/` 2개 자동 반영. 완료 조건: `--check` PASS + `grep -rnE "\[9\].*5회|재시도.*5회"` 잔존 0건. **주의**: 무맥락 `grep "5회"` 는 `rp-ship.md` 의 `mergeStateStatus` 재폴링 최대 5회(본 변경 무관)에 오탐 — 그 규칙은 건드리지 말 것 |

## 검증

대상 = 활성 문서(`docs/skills/`·`docs/harness-*.md`·`CLAUDE.md`·`AGENTS.md`·`README.md`·`.codex/`·`scripts/`). `docs/prd/`·`docs/research/` 는 이력 보존 대상이라 제외.

| 항목 | 방법 | 결과 |
|------|------|------|
| Codex 리뷰 규칙 잔존 | `grep -rn "codex:review\|codex review\|harness-codex-review"` | **0건 PASS** |
| `-led` 잔존 | `grep -rn "Claude-led\|Codex-led"` (`.py` 포함) | **0건 PASS** — 110건 전부 `-Lead` 치환 |
| 링크 무결성 | 삭제된 `harness-codex-review.md` 참조 링크 전수 검색 | **0건 PASS** (`docs/prd/` 이력 언급만 잔존) |
| 회차 표기 정합성 | `grep -rn "최대 3회\|3회 미달" docs/skills/rp-code-review.md docs/harness-code-review.md` | **0건 PASS** ([9] 문맥) |
| Codex 스킬 동기화 | `python3 scripts/sync-codex-skills.py --check` | **PASS** (13개 in sync) |
| settings.json 범위 격리 | `git diff --cached --stat -- .claude/settings.json` | **PASS** — 인덱스 `11 deletions(-)` 단독, 추가 0줄. codex 2블록 외 유입 없음. `permissions`·`hooks` 는 HEAD 와 내용 동일 |
| 문서 **300줄** 한도 | `wc -l` 전 활성 문서 (한도 상향 반영) | **PASS** (초과 0건) |
| 한도 표기 동기화 | `grep -rn "200줄"` (활성 문서) | **0건 PASS** — 4곳 전부 300줄 |
| 서브레포 링크 금지 | `grep -rnE "\]\(.*repositories/[a-z-]+/[^)]+\)" docs/ CLAUDE.md AGENTS.md README.md` | **0건 PASS** |
| 변환 규칙 사장 방지 | `sync-codex-skills.py` 치환 규칙을 `docs/skills/rp-*.md` 13개와 전수 대조 | **PASS(조건부)** — 본 PR 이 제거한 37개는 전부 0매칭(오삭제 없음). 잔존 8개 중 4개(`Agent 툴의 서브에이전트`·`Agent 툴 오류`·`Doc Agent 재투입`·`Doc Agent 재작성`)는 **HEAD 시점부터 이미 0매칭** — 본 PR 유입분이 아니므로 Surgical 원칙상 존치 |

### 행동 게이트

| # | 대상 변경 | 기준 | 결과 |
|:-:|:--------:|------|------|
| G1 | 1 | 본 PRD 의 [4]·[5]·[9] 리뷰 실행 시 `codex` 명령 호출 **0건** | 리뷰 단계에서 실측 |
| G2 | 1 | 리뷰 관련 사용자 확인 질문 **0건** | 리뷰 단계에서 실측 |
| G3 | 2 | 완료 출력이 `(Claude-Lead)` 형식 | 리뷰 단계에서 실측 |
| G4 | **3** | [9] 4회차 미달 시 중단 없이 5회차 재투입 (미발생 시 `관측 없음` 으로 기록 — 통과 조건 아님) | 리뷰 단계에서 실측 |
| G5 | **4** | 본 PR 종료 시점 활성 문서 최대 줄수가 **200 초과 ~ 300 이하** 구간에 진입했는지. 진입 = 한도 상향이 실제 구속력을 가짐 / 미진입 = `현시점 무효과` 로 기록 (통과 조건 아님) | 커밋 직전 `wc -l` 실측 |

| 항목 | 방법 | 결과 |
|------|------|------|
| 리뷰 | [4] 기획 · [5] 엔지니어링 · [9] 코드 — Claude 서브에이전트 (외부 추가 리뷰 없음) | — |
