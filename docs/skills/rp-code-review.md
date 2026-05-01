---
description: '[9] 코드 리뷰. 7항목 최저 점수제 + Codex /codex:review 1회. High/Critical 반영'
argument-hint: '[대상 PR·브랜치]'
---

# rp-code-review

코드 리뷰 (7항목 평가).

## 트리거

- QA 통과 후 자동
- `/rp-code-review` 명령

## 유형별 분기

| 유형 | 리뷰 방식 |
|------|----------|
| 코드 프로젝트 | 7항목 전체 평가 + 민감 정보 스캔 |
| 콘텐츠 프로젝트 | 빌드 검증 + 민감 정보 스캔 |

## 민감 정보 스캔 (전 유형 필수)

CLAUDE.md "민감 정보 금지" 규칙 위반을 커밋 전 자동 검출.

```bash
# 스테이지된 파일에서 금지 패턴 검색
git diff --cached | grep -iE \
  'password[[:space:]]*=|api[_-]?key[[:space:]]*=|secret[[:space:]]*=|token[[:space:]]*=|@[a-z0-9]+\.(com|net|org|kr)|age1[a-z0-9]{50,}'
```

발견 시 머지 차단 → Dev Agent 수정 후 재스캔.
개인 식별 정보(실명·개인 이메일 등)는 정적 패턴으로 검출이 어려우므로, 커밋 메시지·문서 제목은 별도 수동 검토 필요.

## 평가 항목 (코드 프로젝트, 각 10점)

정확성 / 설계 및 구조 / 가독성 / 테스트 품질 / 보안 / 성능 및 효율성 / 유지보수성 및 컨벤션

각 항목에 점수 구간별 기준 + "8점 판단 질문" 적용.

## ⛔ 실행 주체 (서브에이전트 필수)

Claude 코드 리뷰는 **반드시 Agent 툴의 서브에이전트로 실행** (`subagent_type=general-purpose`). 메인 에이전트 셀프 채점 **금지**.

**서브에이전트 프롬프트** 4 필수 항목 (a)~(d) → SSOT: [`../harness-absolute-rules.md`](../harness-absolute-rules.md) "[리뷰 단계 서브에이전트 필수]" 절. 본 단계 적용값: (a) 리뷰 대상 diff·브랜치·파일 경로 (b) 7항목 + PR 유형별 포커스 (c)·(d) SSOT 그대로.

**증거 저장**: 메인 에이전트가 서브에이전트 결과를 `<project-root>/docs/prd/[feature]/review-claude-code-r{N}.md`로 저장 (N=회차, 덮어쓰기 금지).

> **하네스 메타 변경(간소 PRD)**: 코드 리뷰까지 내려올 경우 파일명은 `review-claude-meta-r{N}.md` 단일 리뷰로 대체. Codex 저장도 `review-codex-meta.md`.

**재시도**: 매 회차 새 서브에이전트.

**기술 실패 Fallback**: Agent 툴 오류·토큰 초과·형식 오류 시 최대 2회 재호출. 지속 실패 시 사용자에게 즉시 보고 + 중단. 메인 셀프 채점 우회 금지.

## PR 유형별 포커스

| PR 유형 | 집중 영역 |
|---------|-----------|
| 버그 픽스 | 정확성 + 테스트 품질 |
| 새 기능 | 설계 + 보안 + 성능 |
| 리팩토링 | 가독성 + 설계 + 테스트 품질 |
| 인프라·배포 | 보안 + 유지보수성 |

## 판정

**통과 조건 (모두 충족):**
- 평균 >= 8.0
- 각 항목 >= 7

최저 미달 시 평균 8.0 이상이어도 **미통과**.

**재시도:**
- < 8.0 → Dev Agent 재투입 (최대 3회)
- 3회 후 < 7.0 → 사용자에게 추가 사이클 여부 확인
- 3회 후 7.0~8.0 → 통과 처리

## Codex 추가 리뷰 (Claude와 1차 병렬 1회)

Claude 코드 리뷰 서브에이전트와 **동시** 발사 (메인이 동일 메시지에서 두 tool_use):

1. **사전 `SAVED_CWD=$(pwd)` 캡처** + PRD 루트로 `cd`
2. `codex review --base main` 실행 (wall-clock 300초 타임아웃). 종료 후 `cd "$SAVED_CWD"`
3. **매트릭스 판정**: [`../harness-codex-review.md`](../harness-codex-review.md) "1차 결과 매트릭스" SSOT 참조. 저장 경로는 단계 9 = `review-codex-code.md` (메타 변경은 `review-codex-meta.md`)
4. **통과**: Claude 점수제 통과 AND Codex High/Critical 반영 완료 → 산출물 보고[10] 진입
5. **미달**: 통합 반영 → Claude만 재실행 (Codex 재호출 금지, 최대 2회 추가)

## 이슈 처리

| 심각도 | 처리 |
|--------|------|
| High/Mid | Dev Agent가 자동 수정 |
| Low | 테이블로 제시, 사용자 선택 |

## 자동화 vs 판단

| 영역 | 처리 |
|------|------|
| 린트·포맷팅·빌드 | CI 자동화 (리뷰어 시간 소비 금지) |
| 설계·정확성·테스트 | 리뷰어 판단 (시간 집중) |

## 완료 조건

- 통과 → 산출물 보고 진입
- 입력: QA 결과 + 코드리뷰 점수를 산출물 보고에 전달

## ▶ 자동 전환

Claude 통과 + (Codex High/Critical 반영 완료 OR Codex 토큰/기능 SKIPPED) 시 `✓ [9] 코드리뷰 통과 (Claude+Codex)` (스킵 시 `(Claude+Codex SKIPPED)`) 출력 후 **산출물 보고[10] 자동 진입**.
산출물 보고 후 **커밋·PR까지 자동 진행**. 배포[11]에서 사용자 승인 대기.

→ 리뷰 기준 상세: [`../harness-code-review.md`](../harness-code-review.md)
→ Codex 리뷰 규칙: [`../harness-codex-review.md`](../harness-codex-review.md)
