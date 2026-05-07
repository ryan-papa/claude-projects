# PRD (간소): 리뷰 증거 파일 폐기

**ID:** 20260507_114639_drop-review-evidence-files
**날짜:** 2026-05-07
**상태:** Draft
**유형:** 하네스 메타 변경 (간소 PRD)

---

## 변경 이유

| 항목 | 내용 |
|------|------|
| 현 상태 모순 | `harness-prd.md` L129 = "PRD 하단 흡수" / `harness-absolute-rules.md` L18·L25 = "별도 `review-claude-*-r{N}.md`·`review-codex-*.md` 파일 강제" — 같은 규칙이 두 갈래로 정의됨 |
| 산출물 의미 중복 | 리뷰 지적은 PRD 본문에 반영되어 PRD 자체가 리뷰 결과물인데, 별도 증거 파일·`## Review 결과` 섹션·Codex 원문 파일까지 추가 보존 → 동일 정보 3중 보관 |
| 머지 후 무가치 | 머지 직전 `git rm -r`로 PRD 디렉토리 통째 삭제(`harness-absolute-rules.md` L51) → 보존 비용만 발생, 회수 가치는 0 |
| 단순화 효과 | 리뷰 = "PRD 본문 갱신" 단일 액션으로 정의 → 자동 머지 가드 (b) 검증도 PR body 임베드 단일 소스로 수렴 |

**결정:** 작성 시점부터 리뷰 증거 파일을 만들지 않는다. 점수표·회차 추적·Codex 요약·원문 별도 보존 모두 폐기. 리뷰 지적의 흔적은 PRD 본문 자체에 반영된 결과물로만 남긴다.

## 부수 결정 (리뷰 지적 반영)

| 항목 | 결정 | 근거 |
|------|------|------|
| 자동 머지 가드 (b) "리뷰 증거 게이트" | **폐기** — 가드 4종 → 3종 ((a) CI SUCCESS · (c) PR base 정상 · (d) MERGEABLE) | 가드 (b)를 "PR body 임베드 PRD 요약" 검증으로 재정의 시 임베드 시점이 가드 이후라 순환 의존. 사용자 정신("리뷰 보존 X, PRD에 반영")과 일관 → 인-메모리 리뷰 통과 신뢰 |
| 메타 PRD 식별자 | PRD 본문 frontmatter `**유형:** 하네스 메타 변경` + 간소 4섹션(`## 변경 이유`·`## 영향 파일`·`## 롤백 전략`·`## 검증`) 존재 | 기존 `review-claude-meta-r*.md`/`review-codex-meta.md` 존재 기반 식별이 폐기되므로, PRD 본문 자체가 식별 단일 소스. `rp-ship.md` 메타 분기 선검사·`harness-absolute-rules.md` L53 갱신 |
| 재시도 회차(r{N}) 카운팅 | **폐기** — 재시도 = "PRD 본문 갱신 반복"으로 단순화. 회차 추적·기록 X | 회차별 새 파일이 폐기되면 카운팅 신호가 사라짐. "최대 3회"는 인-메모리 시도 한도로만 유지. 통과 시점의 PRD가 단일 결과물 |

## 영향 파일

| 파일 | 변경 내용 |
|------|----------|
| `docs/harness-absolute-rules.md` | L15 역할 경계의 `review-claude-*.md` 합성 금지 표현 일반화("Claude 채점 결과 합성 금지") / L18 증거 저장 경로 단락 삭제 / L25 Codex 결과 저장 단락 삭제 / L45-50 가드 4종 → 3종((b) 폐기, 번호 재부여) / L51 PRD 정리 단락에서 `Review 결과` 임베드 항목 제거 + 머지 직전 정리 절차는 유지(가드 통과 후 단순 PR body 요약 임베드만) / L53 메타 분기 선검사 = PRD frontmatter `유형: 하네스 메타 변경` + 4섹션 존재로 재정의 |
| `docs/harness-prd.md` | L129 "PRD 하단 `## Review 결과` 기입" 단락 삭제 + 리뷰 결과 흔적은 본문 반영으로 명시 |
| `docs/harness-codex-review.md` | L9-12 결과 저장 경로 표 삭제 / L69 "review-codex-*.md 하단에 `## 반영` 기록" 단락 삭제 / 리뷰 결과는 PRD 본문에 반영 |
| `docs/harness-ship.md` | L30 SSOT 참조에서 "리뷰 증거 게이트" 문구 → "PR body 게이트"로 갱신 / L51 임베드 규칙에서 `## Review 결과` 추출 항목 제거 |
| `docs/prd-template.md` | L127-153 `## Review 결과` 섹션 전체 삭제 |
| `docs/skills/rp-prd.md` | L34 메타 리뷰 파일명 규약 단락 삭제 |
| `docs/skills/rp-plan-review.md` | L22-23 Claude/Codex 저장 경로 단락 삭제 + 재시도는 "PRD 본문 갱신 반복"(회차 카운팅 폐기) 명시 |
| `docs/skills/rp-eng-review.md` | L22-23 Claude/Codex 저장 경로 단락 삭제 + 재시도 단순화 |
| `docs/skills/rp-code-review.md` | L47 메인 에이전트 증거 저장 단락 삭제 / L49 메타 변경 파일명 단락 삭제 / L83 저장 경로 분기 단락 삭제 + 재시도 단순화 |
| `docs/skills/rp-ship.md` | L19-49 사전 체크 게이트(`review-*.md` 존재 검증) 전체 폐기 / L66 가드 (b) 항목 삭제, 가드 3종으로 재서술 / L74-75 임베드 추출 항목에서 `## Review 결과` 제거 / L101 메타 분기 선검사 = PRD frontmatter `유형: 하네스 메타 변경` + 4섹션 존재 / L150 SSOT 링크 문구에서 "리뷰 증거 게이트" 삭제 |
| `CLAUDE.md` | L69 트리에서 "리뷰 증거" 문구 삭제 |
| `AGENTS.md` | L9 게이트 항목에서 "리뷰 증거" 삭제 / L121 "리뷰 증거 수집 필수" 단락 삭제 / L135-146 Codex/Claude 리뷰 파일명 규칙 블록 전체 삭제 / L157 검토 우선순위에서 "리뷰 증거 오류" 삭제 / L176 "리뷰 증거 파일 보존" 항목 삭제 |
| `.codex/skills/rp-plan-review/SKILL.md`, `rp-eng-review/SKILL.md`, `rp-code-review/SKILL.md`, `rp-prd/SKILL.md`, `rp-ship/SKILL.md` | 원본 동기화 (`scripts/sync-codex-skills.py --install-user`) |

## 롤백 전략

| 조건 | 절차 |
|------|------|
| 머지 후 회귀 발견 | `git revert <merge-commit>` 1회로 단일 PR 복원 (모든 영향 파일이 단일 PR에 묶여 있음) |
| 리뷰 추적성 부족 판명 | 후속 PR로 PRD 하단 `## Review 결과` 섹션만 부분 복원 (가드 (b)는 PR body 단일 소스 유지) |
| Codex 원문 회수 필요 | git history에서 본 PR 머지 직전 시점의 PR body·PRD 본문 추출 (이후 모든 리뷰 흔적은 PRD 본문 반영으로만 존재) |

## 검증

| 단계 | 방식 |
|------|------|
| Claude 자체 점검 | 영향 파일 12종 + Codex 어댑터 5종 변경 후 `grep -rn "review-claude\|review-codex\|리뷰 증거\|## Review 결과"` 결과가 본 PRD 디렉토리 외에 0건임을 확인 |
| 스킬 동기화 | `rtk python3 scripts/sync-codex-skills.py --check` 무결성 확인 |
| Claude 서브에이전트 리뷰 | Agent 툴 `general-purpose`로 본 PRD 채점 — 결과는 PRD 본문 갱신으로만 반영 (별도 파일 X) |
| Codex 추가 리뷰 | `/codex:review` 1회 — High/Critical 지적은 PRD 본문에 반영 (별도 파일 X) |
| CI | `.github/workflows/` 기존 lint-docs 통과 |
| 머지 가드 | (a) CI SUCCESS / (b) PR base = main / (c) `gh pr view --json mergeable` = MERGEABLE — 가드 3종으로 단순화 |
