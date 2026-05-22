# Harness Engineering Framework

> AI 코딩 에이전트의 품질을 일정하게 유지하는 워크플로우 규칙 체계

---

## 이 프로젝트는 무엇을 하나?

AI 코딩 도구(Claude Code, Codex CLI 등)에게 "앱 하나 만들어줘"라고 시켰을 때, 같은 요청이어도 결과 품질이 매번 들쭉날쭉합니다. 이 프로젝트는 AI가 따라야 할 **일하는 순서(workflow)**와 **단계별 검토 규칙(review gates)**을 정리해서, 사람이 코드를 한 줄도 쓰지 않아도 일정한 품질의 결과물이 나오도록 보장합니다.

> **비전: 사람의 코딩 한 줄 없이 프로젝트를 완성한다.**

### 핵심 특징

- **12단계 워크플로우 (기획 → 회고)** — 새 프로젝트·기능 하나를 만들 때 AI가 어떤 순서로 무엇을 해야 하는지 단계별 명문화. 각 단계 완료 시 다음 단계로 **자동 진입**(사용자 승인 없이 진행). 단, 마지막 회고[12]는 사용자 명시 명령(`/rp-retro`) 시에만 실행.
- **3관문 리뷰** — 기획(PRD 작성 후), 엔지니어링(설계 검토), 코드(구현 검토) 3단계에서 **메인 AI가 아닌 별도 서브에이전트**가 독립 채점. 메인이 자기 작업을 셀프 채점하면 편향이 생기므로 강제 분리.
- **이중 리뷰 (Claude × Codex)** — Claude-led 모드(메인=Claude)에서는 위 3관문에 추가로 **OpenAI Codex의 독립 리뷰 1회**(`/codex:review`)를 병렬 실행. 한 AI가 놓친 문제(예: 동시성 버그, AOP 프록시 우회)를 다른 AI가 잡아내는 패턴. Codex-led 모드(메인=Codex)에서는 추가 리뷰 N/A.
- **자동 머지 가드 3종 AND** — PR(변경 묶음)의 자동 머지는 (a) CI(자동 검사) 통과 · (b) base 브랜치 정상 · (c) 충돌 없음(MERGEABLE) **3개 모두 충족** 시에만 진행. 하나라도 실패 시 PR을 OPEN 유지 + 사용자 즉시 보고.

---

## 프로젝트 구조

```
workflow-agent-harness/
├── docs/                              # 공통 규칙 문서
│   ├── harness-absolute-rules.md      # 절대 규칙 SSOT (예외 없음)
│   ├── harness-workflow.md            # 12단계 전체 플로우
│   ├── harness-prd.md                 # PRD 작성 + 2단계 리뷰
│   ├── harness-dev.md                 # 개발·태스크·테스트
│   ├── harness-qa.md                  # QA + 콘텐츠 검수
│   ├── harness-code-review.md         # 7항목 코드리뷰
│   ├── harness-codex-review.md        # Codex 추가 리뷰 (4·5·9, 1회)
│   ├── harness-ship.md                # 산출물 보고 + 배포
│   ├── harness-design.md              # UI 디자인 원칙
│   ├── harness-readme.md              # README 작성 규칙
│   ├── harness-db.md                  # DB 스타일 (테이블·컬럼·인덱스·ENUM)
│   ├── harness-backend-test-policy.md # 백엔드 4-게이트 테스트 정책
│   ├── security-guide.md              # 시크릿 관리 원칙 (sops+age)
│   ├── security/                      # 시크릿 운영 상세
│   ├── research/                      # 하네스 메타 변경 근거 발췌 (서비스 회고에서 추출)
│   ├── templates/                     # PRD·README·CI·시크릿 템플릿
│   └── skills/                        # 하네스 스킬 (rp-*)
├── .claude/commands/                  # Claude Code 슬래시 명령 (rp-* 심링크)
├── .codex/skills/                     # Codex CLI 로컬 스킬 변환본
├── scripts/sync-codex-skills.py       # 스킬 변환·동기화
├── AGENTS.md                          # Codex용 하네스 어댑터
├── CLAUDE.md                          # Claude Code 진입점
└── README.md
```

---

## 12단계 워크플로우

| # | 단계 | 핵심 | 추가 리뷰 |
|:-:|------|------|:--:|
| 1 | 초기화 | 새 프로젝트 골격 생성 | — |
| 2 | 구체화 | "무엇을 만들지" 질문·리서치로 결정 잠금 | — |
| 3 | PRD 작성 | 요구사항을 문서(PRD = Product Requirements Doc)로 정리 | — |
| 4 | 기획 리뷰 | PRD의 사용자 가치·경계 채점 | Codex 1회 (Claude-led) |
| 5 | 엔지니어링 리뷰 | 기술·확장성·운영성 채점 | Codex 1회 (Claude-led) |
| 6 | 태스크 분해 | 작업을 작은 단위로 쪼개기 | — |
| 7 | 개발 | 태스크별 코딩·테스트·커밋 | — |
| 8 | QA | 시나리오 테스트 + 콘텐츠 검수 | — |
| 9 | 코드 리뷰 | 7항목 채점 (정확성·설계·가독성·테스트·보안·성능·유지보수성) | Codex 1회 (Claude-led) |
| 10 | 산출물 보고 | 결과 정리 + 자동 진입 | — |
| 11 | 배포 (`rp-ship`) | PR 생성 → CI → 자동 머지 가드 3종 → **배포 승인 대기** → 배포 | — |
| 12 | 회고 | 절차 준수·효율·규칙 개선 평가 (**사용자 명령 시에만**) | — |

각 단계 완료 시 다음 단계 자동 진입. 멈춤 지점은 배포[11] 머지 승인 대기와 회고[12](자동 진입 없음) 둘뿐.

**진입 스킬:**
- `/rp-workflow` — 신규 프로젝트·기능 (init부터 전 단계)
- `/rp-amend` — 기존 프로젝트 기능 수정·추가 (init 스킵, specify부터)

---

## 규칙 문서

| 문서 | 역할 |
|------|------|
| [`harness-absolute-rules.md`](docs/harness-absolute-rules.md) | 절대 규칙 SSOT(Single Source of Truth = 유일한 정답 문서) |
| [`harness-workflow.md`](docs/harness-workflow.md) | 12단계 전체 플로우 |
| [`harness-prd.md`](docs/harness-prd.md) | PRD 작성 + 기획·엔지니어링 2단계 리뷰 |
| [`harness-dev.md`](docs/harness-dev.md) | 개발·태스크·테스트 |
| [`harness-qa.md`](docs/harness-qa.md) | QA + 콘텐츠 검수 |
| [`harness-code-review.md`](docs/harness-code-review.md) | 7항목 코드리뷰 |
| [`harness-codex-review.md`](docs/harness-codex-review.md) | Codex 추가 리뷰 (4·5·9, 1회) |
| [`harness-ship.md`](docs/harness-ship.md) | 산출물 보고 + 배포 |
| [`security-guide.md`](docs/security-guide.md) | 시크릿·자격증명 (sops+age) |

---

## 빠른 시작

### 전제조건

| 도구 | 용도 | 확인 |
|------|------|-----|
| [Claude Code](https://claude.com/claude-code) | AI 코딩 에이전트 (메인) | `claude --version` |
| Node.js 18.18+ | Codex 플러그인 런타임 | `node --version` |
| Git | 버전 관리 | `git --version` |
| ChatGPT 구독 또는 OpenAI API 키 | Codex 리뷰 인증 | — |

### 사용법

```bash
git clone https://github.com/ryan-papa/workflow-agent-harness.git
cd workflow-agent-harness
claude  # Claude Code 실행 — CLAUDE.md + Codex 플러그인 자동 제안
```

1. Claude Code가 `CLAUDE.md`를 자동 로드해 하네스 규칙을 에이전트에 적용
2. `.claude/settings.json`에 선언된 Codex 플러그인 설치 프롬프트 승인
3. 최초 1회 `/codex:setup` + `!codex login`으로 Codex CLI 설치·인증
4. 신규 프로젝트는 `/rp-workflow [기능 설명]`, 기존 프로젝트 수정은 `/rp-amend [변경 설명]`
5. 서비스 레포는 `repositories/{프로젝트명}/`에 위치 — **하네스 문서에서 서브레포 내부 파일을 직접 링크 참조 금지** (필요 시 발췌본을 `docs/research/`에 작성)

**Codex CLI에서 직접 진입할 경우** 루트 [`AGENTS.md`](AGENTS.md)가 Claude 중심 규칙을 Codex 실행 규칙으로 번역합니다. 로컬 Codex 스킬 설치는 `rtk python3 scripts/sync-codex-skills.py --install-user`.

### Codex 플러그인

`.claude/settings.json`에 선언되어 clone 후 폴더 trust 시 자동 설치 프롬프트가 뜹니다.

```json
{
  "extraKnownMarketplaces": {
    "openai-codex": { "source": { "source": "github", "repo": "openai/codex-plugin-cc" } }
  },
  "enabledPlugins": { "codex@openai-codex": true }
}
```

제공 명령: `/codex:review`, `/codex:rescue`, `/codex:status`, `/codex:result` 등.
**Claude-led 모드에서만** 4·5·9 단계의 Claude 서브에이전트 완료 후 `/codex:review --wait` 1회 실행, High/Critical 지적 반영.

---

## 작성 모드 (Claude-led vs Codex-led)

진입 런타임에 따라 리뷰 구성이 달라집니다 (SSOT: [`harness-absolute-rules.md`](docs/harness-absolute-rules.md) "작성 모드 및 리뷰 매트릭스"):

| 모드 | 메인 런타임 | 서브에이전트 채점 | 외부 추가 리뷰 |
|------|-----------|---------------|-------------|
| Claude-led | Claude Code | Claude Agent 툴 | Codex 1회 (4·5·9) |
| Codex-led | Codex CLI | Codex `spawn_agent` | N/A — 메인=Codex |

---

## 기여 가이드

1. Fork
2. `git checkout -b feat/{기능명}`
3. 커밋 (Conventional Commits)
4. Push → PR 생성

하네스 규칙·문서·스킬 변경은 **메타 변경 단축 경로** 적용: `rp-init`·`rp-specify`·`rp-task`·`rp-dev` 스킵 + `rp-prd` 간소(변경 이유·영향 파일·롤백 전략·검증 4섹션) + 리뷰[4·5·9] + `rp-ship`. 상세: [`harness-absolute-rules.md`](docs/harness-absolute-rules.md) "단축 경로·예외".

---

## 라이선스

MIT License
