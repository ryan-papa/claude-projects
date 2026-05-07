# Harness Codex Review

Claude 리뷰와 Codex 플러그인(`openai/codex-plugin-cc`) 추가 리뷰를 **1차 병렬**로 실행한다. Codex는 1회만 호출되며, Claude는 미달 시 최대 2회 추가 재실행(총 3회).

## 적용 단계

| 단계 | Claude 리뷰 | Codex 명령 | 결과 처리 |
|:---:|-------------|-----------|---------|
| 4 | 기획 리뷰 | `/codex:review --wait` | High/Critical 지적을 PRD 본문에 반영 |
| 5 | 엔지니어링 리뷰 | `/codex:review --wait` | High/Critical 지적을 PRD 본문에 반영 |
| 9 | 코드 리뷰 | `/codex:review --wait --base main` | High/Critical 지적을 코드·PRD 본문에 반영 |
| meta | 하네스 메타 변경(간소 PRD 단일 리뷰) | `/codex:review --wait` | High/Critical 지적을 간소 PRD 본문에 반영 |

cwd 정의:
- 일반 기능 변경 → `repositories/[project]/`
- 하네스 메타 변경(본 규칙·docs/·CLAUDE.md 등) → `claude-projects/` 루트

## 실행 규칙

| 항목 | 규칙 |
|------|------|
| 실행 순서 | Claude 리뷰 서브에이전트와 Codex를 **1차 병렬** 실행 (메인이 동일 메시지에서 Agent 툴 + Bash `codex review`를 두 tool_use로 동시 발사). Claude 미달 시 2·3차 재실행은 Claude만 (Codex 재호출 금지) |
| 모드 | `--wait` 포그라운드 또는 `codex review --uncommitted`, 완료까지 동기 대기 |
| 회차 | Codex는 **1차 1회만** 실행 (재시도 없음). Claude는 최대 3회 |
| 점수화 | **없음** — Codex는 점수 산출·통과 판정 안 함 |
| 반영 규칙 | 지적 심각도 **High / Critical**만 수동 반영. Medium 이하는 참고용 |
| 결과 보존 | **별도 파일 저장 없음** — High/Critical 지적은 PRD 본문에 반영. Codex stdout은 인-메모리 처리 후 폐기 |
| cwd | **해당 기능의 PRD가 있는 프로젝트 루트**에서 실행. 일반 기능은 `repositories/[project]/`, 하네스 메타 변경은 `claude-projects/` 루트. cwd 오인 시 Codex가 상위 레포를 리뷰해 결과 무효 |
| 실행 전 cwd 저장 | `/codex:review` 진입 **직전** `SAVED_CWD=$(pwd)`로 시작 cwd 캡처 후 PRD 프로젝트 루트로 `cd`. 이미 일치 시 `cd` 생략 가능 |
| 실행 전 체크 | `pwd` 출력이 PRD 프로젝트 루트와 일치하는지 검증. 불일치 시 `cd` 후 재확인. 체크 생략 금지 |
| 실행 후 cwd 복귀 | Codex 종료 직후(정상/SKIPPED/중단 무관) `cd "$SAVED_CWD"` + `pwd` 검증. 복귀 실패(원래 디렉터리 삭제 등) 시 중단 + 사용자 보고. 누락 시 다음 단계 진행 금지 |
| Hang 타임아웃 | wall-clock **300초** 초과 시 SIGTERM → 비-스킵 = 중단 (사용자 보고). 회고에서 조정 가능 |

## 병렬 실행 패턴

```
[단계 N 진입]
  ↓
SAVED_CWD=$(pwd)  ← 시작 cwd 캡처
  ↓
PRD 프로젝트 루트로 cd (이미 일치 시 생략)
  ↓
1차 병렬 발사 (메인이 동일 메시지에서 두 tool_use 동시 호출):
  ├─ Agent 툴: Claude 서브에이전트 (기존 점수제 채점)
  └─ Bash: codex review --uncommitted (300초 타임아웃)
  ↓
양쪽 결과 수신 → cd "$SAVED_CWD"
  ↓
1차 결과 매트릭스 판정 (아래 표)
  ├─ 통과 → 다음 단계
  └─ 미달 → 통합 반영(Codex High/Critical + Claude 지적) → Claude 재실행 (Codex 재호출 금지, 최대 2회 추가)
```

### 1차 결과 매트릭스 (6분기)

| Claude | Codex 정상 | Codex SKIP (토큰/기능) | Codex 비-스킵 비정상 |
|--------|-----------|----------------------|--------------------|
| **통과** (평균 ≥8.0 / 최저 ≥7) | High/Critical 반영 → 다음 단계 | SKIPPED 보고 → 다음 단계 | 워크플로우 중단 + 사용자 보고 |
| **미달** | Codex High/Critical + Claude 지적 통합 반영 → Claude 재실행 (Codex 재호출 금지) | Claude 지적만 반영 → Claude 재실행 | 워크플로우 중단 + 사용자 보고 |

통과 조건: **Claude 점수 통과 AND Codex High/Critical 반영 완료** (Codex SKIP 시 Claude 점수만으로 판정).

2·3차 재실행 원칙: Codex는 **1차 1회만**. 메인은 재실행 PRD/코드에 1차 Codex High/Critical 반영이 포함되었는지 검증 후 새 서브에이전트 호출.

## High / Critical 반영 절차

1. Codex 결과에서 `[high]` / `[critical]` 표기 라인만 추출
2. 각 지적을 PRD(단계 4·5·메타) 또는 코드(단계 9)에 반영
3. 반영 결과는 PRD 본문 자체로 흡수 (별도 기록 파일 없음)
4. Claude 리뷰는 재실행하지 않음 (Codex 1회 원칙 유지, 반영만 수행)

## 토큰/기능 이슈 스킵 (1회)

Codex CLI가 토큰 한도·rate limit·기능 미지원 신호를 명시적으로 출력한 경우에 한해 **1회 스킵 후 다음 단계 진입**. 그 외 모든 비정상 종료는 **기존대로 중단 + 사용자 보고**.

### 스킵 판정 (AND 조건)

다음 두 조건을 **모두** 충족해야 스킵:

1. exit code ≠ 0 **또는** exit code = 0이라도 stderr/stdout에 후술 패턴 매칭
2. stderr/stdout이 [토큰·기능 신호 패턴](#토큰기능-신호-패턴) 표 중 **1개 이상** 매칭

매칭 0건 → 비-스킵 = 중단. 재시도 없음.

### 토큰·기능 신호 패턴

> **SSOT**: 본 표가 단일 출처
> **참조 규칙**: 다른 스킬·문서는 본 섹션 링크 참조, 중복 정의 금지
> **정규식 옵션**: 전체 case-insensitive

| # | 정규식 | 매칭 신호 | 실제 Codex CLI 출력 예시 |
|---|--------|-----------|---------------------------|
| 1 | `rate.?limit` | rate limit | `Error: rate limit exceeded` |
| 2 | `usage.?limit` | usage cap (ChatGPT Pro 등) | `ERROR: You've hit your usage limit. Upgrade to Pro ...` |
| 3 | `token[^\n]*(limit\|exceed\|exhaust)` | 토큰 한도 | `token limit exceeded for this request` |
| 4 | `context.{0,3}(length\|window).*(exceed\|limit)` | 컨텍스트 한도 | `context length exceeded` |
| 5 | `quota` | quota 소진 | `monthly quota exhausted` |
| 6 | `(purchase\|upgrade).*(credit\|plan\|pro)` | 결제 안내 동반 한도 | `visit ... to purchase more credits` |
| 7 | `not (yet )?supported` | 기능 미지원 | `model not yet supported` |
| 8 | `unsupported (model\|feature)` | 기능 미지원 | `unsupported feature: --base on detached HEAD` |
| 9 | `model.*not.*available` | 모델 미가용 | `model gpt-5.5 not available in this region` |

신규 패턴 추가 시 본 섹션 갱신 + [동작 시뮬레이션 양방향 검증](#스킵-규칙-검증) 통과 후 머지.

### 스킵 시 보고 형식

증거 파일 저장 없음. 사용자에게 응답 텍스트로 `SKIPPED — codex token/feature signal` 헤더 + 매칭 패턴 # + 판정 사유 한 줄만 보고 후 다음 단계 진입. stdout/stderr 인용 시 `(?i)(api[_-]?key\|token\|secret)\s*[:=]\s*\S+` 매칭 값은 `***REDACTED***`로 치환.

### 비-스킵 사유 (기존대로 중단)

- 네트워크 오류, `codex login` 미완료, 플러그인 미설치
- 패턴 매칭 0건 stderr (예측 불가 메시지)
- wall-clock 300초 초과 (hang)
- exit code != 0 + 패턴 매칭 0건

이 경우 워크플로우 즉시 중단 + 사용자 보고. 자동 재시도 금지.

### 스킵 규칙 검증

| 검증 항목 | 방법 |
|-----------|------|
| 양방향 동작 시뮬레이션 | (1) 가짜 stderr `rate limit exceeded`/`hit your usage limit` 주입 → SKIPPED 보고 출력 + 다음 단계 진입 PASS. (2) 가짜 stderr `network unreachable` → 매칭 0건 → 워크플로우 중단 + 사용자 보고 PASS. 두 케이스 모두 통과해야 검증 완료 |
| 마스킹 검증 | 가짜 stderr `api_key=sk-abc123def` 주입 → 사용자 보고 텍스트에 `***REDACTED***` 치환 확인 |

## 플러그인 선언

`.claude/settings.json` 루트에 선언:

```json
{
  "extraKnownMarketplaces": {
    "openai-codex": {
      "source": { "source": "github", "repo": "openai/codex-plugin-cc" }
    }
  },
  "enabledPlugins": { "codex@openai-codex": true }
}
```

- 요구사항: Node.js 18.18+, ChatGPT 구독 또는 OpenAI API 키, `codex login` 완료
- clone 후 폴더 trust 시 플러그인 설치 프롬프트 자동 노출
- 최초 1회 `/codex:setup` + `!codex login` 필요

## ⛔ 절대 규칙

- 단계 4 · 5 · 9 및 **메타 변경 단일 리뷰**에서 Codex 리뷰 **수행 필수**. **단**, [토큰/기능 이슈 스킵](#토큰기능-이슈-스킵-1회) 조건(AND 충족) 시에만 1회 스킵 허용. 임의 생략 금지
- Codex 결과 **별도 파일 저장 금지** (인-메모리 처리 후 PRD 본문 반영)
- High / Critical 지적은 반영 **필수**. 미반영 시 다음 단계 진입 금지
- Codex는 점수화·재시도 **하지 않음** (1회 원칙)
- 플러그인 미설치·login 미완료 등 환경 오류 시 워크플로우 중단, 사용자 보고
- 토큰/기능 신호 외 비정상 종료(네트워크·hang·매칭 0건)는 **기존대로 중단**

→ 워크플로우: [`harness-workflow.md`](harness-workflow.md)
→ 기획 리뷰: [`skills/rp-plan-review.md`](skills/rp-plan-review.md)
→ 엔지 리뷰: [`skills/rp-eng-review.md`](skills/rp-eng-review.md)
→ 코드 리뷰: [`harness-code-review.md`](harness-code-review.md)
