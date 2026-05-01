# PRD: 리뷰 단계 병렬화 (Claude + Codex 1차 동시)

분류: 하네스 메타 변경 (단축 경로)

## 1. 변경 이유

현재 단계 4·5·9 + 메타 단축 경로 리뷰는 **직렬**: Claude(최대 3회) → 통과 후 Codex(1회). Claude 3회 모두 끝나야 Codex가 시작되어 wall-clock·토큰 비효율. 1차에서 Claude·Codex 병렬 시:
- 1차에서 양쪽 지적을 함께 받아 통합 반영 가능 → 회차 단축
- Codex 지적이 일찍 노출되어 PRD/코드 수정 방향이 더 정확해짐
- 토큰 이슈 SKIP 분기는 기존 SSOT 그대로 유지

**비용/편익 균형점**: Claude 1차 통과 케이스에서도 Codex 1회는 무조건 실행되어 비용 +1회. 그러나 Claude 미달 케이스(현재 통계상 절반 이상, audit-A/B/C·codex-cwd-restore 등 r1 미달 빈번)에선 Codex 결과를 1차에서 받아 통합 반영하므로 회차 단축이 비용을 상쇄. **순이득 조건**: Claude 1차 미달률이 50% 이상일 때 wall-clock·토큰 모두 이득. 통과 케이스는 Codex 1회 비용 추가지만 절대 시간은 직렬 대비 동일 또는 단축(병렬 실행이라 wall-clock max 기준).

베이스라인 측정값은 본 PR 적용 전 미보유 → 머지 후 첫 사이클에서 wall-clock 1회 기록·후속 회고에서 비교.

## 2. 대안 비교

| 대안 | 장점 | 단점 | 채택 |
|------|------|------|------|
| (A) 1차 병렬 + Claude 미달 시 Claude만 재실행 (총 3회) | wall-clock 단축·Codex 1회 원칙 유지·기존 점수제·반영 규칙 그대로 | 통과 케이스에서 Codex 1회 비용 +1 | ★ |
| (B) 매 회차 Claude·Codex 둘 다 병렬 (총 3회씩) | 모든 회차에 Codex 시야 반영 | 토큰·시간 낭비, Codex 1회 원칙 위배 | × |
| (C) 직렬 유지 (현행) | 변경 없음·Codex 1회 원칙 자연 충족 | wall-clock 길고 1차 Codex 시야 늦게 들어옴 | × |

(A) 채택: 사용자 명시 요구 + Codex 1회·점수제·High/Critical 반영 규칙 유지하면서 wall-clock 단축.

## 3. 영향 파일

| 파일 | 변경 내용 |
|------|-----------|
| `docs/harness-codex-review.md` | "직렬 실행 패턴" → "병렬 실행 패턴" 교체. 1차 결과 6분기 매트릭스(아래) + 병렬 실행 메커니즘 명시 |
| `docs/harness-absolute-rules.md` | `[4][5][9] Codex 추가 리뷰 필수` 첫 줄 "Claude 리뷰 통과 후" → "Claude 리뷰와 1차 병렬 실행". Codex 1회·점수화 없음·High/Critical 반영 필수·SKIP 규칙·cwd 복귀는 유지 |
| `docs/skills/rp-plan-review.md` | 절차 재정렬: (1) Claude 서브에이전트 + Codex 동시 호출 (2) 결과 합치기 (3) 매트릭스 판정 (4) 미달 시 통합 반영 + Claude 재실행(2·3차는 Codex 재호출 금지) |
| `docs/skills/rp-eng-review.md` | 동일 |
| `docs/skills/rp-code-review.md` | 동일 |

### 1차 병렬 결과 매트릭스 (6분기)

| Claude | Codex 정상 | Codex SKIP | Codex 비-스킵 비정상 |
|--------|-----------|-----------|--------------------|
| **통과** | High/Critical 반영 → 다음 단계 | SKIP 헤더 저장 → 다음 단계 | 워크플로우 중단 + 사용자 보고 |
| **미달** | Codex High/Critical + Claude 지적 통합 반영 → Claude 재실행 (Codex 재호출 금지) | Claude 지적만 반영 → Claude 재실행 | 워크플로우 중단 + 사용자 보고 |

2·3차 재실행 원칙: Codex는 **1차 1회만**. Claude 재실행 PRD/코드에 1차 Codex High/Critical 반영이 포함되었는지 메인이 검증 후 새 서브에이전트 호출.

### 병렬 실행 메커니즘

`/codex:review --wait`는 동기 블로킹이라 단일 셸에서 진정한 병렬 불가. 메인 에이전트가 (a) Agent 툴로 Claude 서브에이전트 호출 + (b) Bash로 `codex review --uncommitted` 호출을 **동일 메시지에서 두 tool_use로 동시 발사** → 두 결과가 독립적으로 수신될 때까지 대기. 메인이 완료 후 매트릭스 판정.

## 4. 롤백

| 단계 | 조치 |
|------|------|
| 1 | `git revert <merge-commit>` → main에서 본 PR 제거 |
| 2 | 각 파일 직전 커밋 상태로 복원. 후속 PR은 다시 직렬로 회귀 |

## 5. 검증

| 항목 | 결과 기준 |
|------|----------|
| 변경 파일 ≤ 200줄 | 5개 파일 모두 200줄 이하 유지 |
| SSOT 1곳 유지 | 병렬 패턴 본문은 `harness-codex-review.md` 1곳, 매트릭스도 동일 |
| 죽은 링크 0건 | 신규 링크 없음 |
| 자기참조 회피 | 본 PR은 머지 전이라 기존 직렬 절차로 메타 리뷰 진행. 머지 후 다음 PR부터 병렬 절차 적용 |
| 6분기 매트릭스 명시 | `harness-codex-review.md`에 6분기 표 존재 확인 |
| 메커니즘 명시 | "메인 에이전트가 Agent + Bash 동시 호출" 문구 존재 확인 |
| wall-clock 측정 | 머지 후 첫 사이클에서 1차 병렬 wall-clock 기록 → 다음 회고에서 직렬 대비 비교 |
| 메타 리뷰 게이트 | Claude `review-claude-meta-r{N}.md` + Codex `review-codex-meta.md` 통과 (직렬, 본 PR 한정) |
| CI lint-docs | 통과 |
