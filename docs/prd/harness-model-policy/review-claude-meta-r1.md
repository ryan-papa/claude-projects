# review-claude-meta-r1 — harness-model-policy

리뷰어: Claude (서브에이전트 역할 수행, 메인 컨텍스트 분리)
대상 브랜치: `feat/harness-model-policy`
대상 파일:
- `docs/prd/harness-model-policy/prd.md` — **부재 (디렉터리만 존재, 0 파일)**
- `CLAUDE.md` (라인 94~115 변경)
- `docs/harness-workflow.md` (라인 5~42 변경)

## 채점

| # | 항목 | 점수 | 근거 |
|:-:|------|:--:|------|
| 1 | 변경 이유 명료성 | **3/10** | 간소 PRD(`prd.md`)가 **존재하지 않음**. 4섹션(이유·영향 파일·롤백·검증) 중 어느 것도 문서화 안 됨. CLAUDE.md 본문에 "비용·속도 절감 + Opus 의 추론 우위" 한 줄 근거만 있으나, 왜 specify·prd 만 Opus 인지(예: 코드리뷰는 왜 Sonnet 으로 충분한지) 정량·정성 근거 없음. 하네스 메타 변경 단축 경로 정의상 PRD 누락은 **High** 결함 |
| 2 | 영향 파일 정확성 | **8/10** | 실제 수정은 `CLAUDE.md` + `docs/harness-workflow.md` 2개로 명세대로. 단, `docs/skills/rp-*.md` frontmatter `model:` 필드에 영향이 발생할 수 있는데 본 PR 은 명시적으로 "스킬 frontmatter 미수정"으로 두었음 — 이 결정의 영향 평가가 PRD 부재로 누락 |
| 3 | 롤백 단순성 | **9/10** | 두 파일 모두 표·다이어그램 텍스트 추가뿐, 외부 의존성·코드·CI 변경 없음. 단일 revert commit 으로 원복 가능. 데이터/스키마 영향 0 |
| 4 | 검증 절차 | **5/10** | CI `lint-docs` 가 마크다운 링크·표 검증을 수행하므로 자동 게이트는 통과 가능. 그러나 **본 정책의 효과 검증 절차**(예: 다음 N개 이터레이션에서 모델 사용 로그·비용 절감률·품질 회귀 모니터링) 가 PRD 에 없음. "권장이며 강제 전환 자동화는 별도 후속"은 검증을 사실상 수동·주관에 위임 |
| 5 | 정책 일관성 | **7/10** | specify/prd→Opus, 나머지→Sonnet 매핑은 **합리적**. 신규 도메인 추론 구간에 강모델 집중은 표준적 trade-off. 다만 (a) `rp-code-review` 는 7항목 채점 + 보안·성능 추론을 요구하는데 Sonnet 단일 권장은 일부 PR(보안·동시성 비중 큰 변경) 에서 품질 저하 리스크. (b) `rp-eng-review` 도 아키텍처 추론 비중이 높아 Sonnet 일률 권장은 specify 와 비대칭 — PRD 에 예외 조항(중대 보안/아키텍처 PR 시 Opus override) 명문화 필요 |
| 6 | 후속 영향 | **6/10** | 스킬 `frontmatter` 미수정은 사용자 자유도(수동 `/model` 전환) 보장 측면에서 의도적이나, **자동화 부재로 정책이 사실상 가이드 수준**. CLAUDE.md `자동 전환` 절(라인 120) 과 본 정책의 "수동 모델 전환" 사이 인지부조화 — 전 단계 자동 진입을 약속하면서 모델만 수동? 사용자 흐름 끊김. 후속 작업(자동 모델 스위칭 hook, settings.json `model` per-skill) 의 **티켓·기한 명문화**가 PRD 에 있어야 함 |

## 종합 (최저 점수제)

**최저 = #1 변경 이유 명료성 = 3/10 → 종합 3/10 → 미통과 (8.0+ 필요)**

## High / Critical

| 등급 | 항목 | 조치 |
|------|------|------|
| **Critical** | `prd.md` 부재 | 4섹션(변경 이유·영향 파일·롤백·검증) 간소 PRD 작성 후 재리뷰. 하네스 메타 변경 단축 경로 정의상 필수 |
| **High** | 효과 검증 절차 부재 | "권장 모델 적용 후 비용·품질 변동 회고 점검" 항목을 PRD §검증에 명문화. 최소 1회 회고에서 모델 분포·재작업률 측정 |
| **High** | code-review/eng-review 일률 Sonnet 권장의 위험 | 보안·동시성·아키텍처 비중 큰 PR 시 Opus override 예외 조항 명문화 |
| **Medium** | 스킬 frontmatter 미수정으로 자동화 단절 | 후속 티켓 명시 (예: `T-harness-auto-model-switch`) 및 본 정책의 권장 → 강제 승격 조건 정의 |
| **Low** | CLAUDE.md 본문 표와 harness-workflow.md 표 중복 | SSOT 를 harness-workflow.md 로 두고 CLAUDE.md 는 링크만 — 향후 두 곳 동기 업데이트 누락 리스크 |

## 결론

PRD(`prd.md`) 누락이 단일 Critical. 작성 후 본 리뷰를 r2 로 재실행할 것. 정책 자체의 방향성은 합리적이나, 검증·예외·후속 티켓 명문화 없이는 "권장"이 사문화될 가능성이 큼.
