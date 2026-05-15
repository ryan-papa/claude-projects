---
**유형:** 하네스 메타 변경
**일시:** 2026-05-16 08:11 KST
**브랜치:** `feat/harness-prd-cleanup-drop-and-test-scope`
---

# 하네스 PRD 정리 단계 폐기 + 테스트 변경 영향 범위 강제

## 변경 이유

| 문제 | 사용자 피드백 | 해결 |
|------|--------------|------|
| 머지 직전 PRD 디렉토리 삭제 commit 이 CI 를 한 번 더 돌려 ship 지연 | "테스트 또 도는것도 느리고" | 디렉토리 삭제 단계(rp-ship 9·10) 폐기. PR 본문 임베드만 PR 생성 시점(단계 5)에 처음부터 포함 |
| 머지 후 PRD 가 main 에 없어 추적성 약화 | "PRD 하나니까 올려도 될듯" | PRD 디렉토리 main 영구 보존 |
| 단위 테스트가 요청 기능만 커버해 인접 회귀 검출 못 함 (안드 5초 가드 회귀 사례) | "수정된 다른 영역도 테스트 코드에 추가하도록 해" | rp-dev 테스트 규칙 + rp-code-review 게이트에 "변경 영향 범위 회귀 테스트 필수" 명시 |

## 영향 파일

| 파일 | 변경 내용 |
|------|----------|
| `docs/harness-absolute-rules.md` | (1) L50 — "머지 직전 PRD 정리 필수" → "PR 생성 시 PRD 요약 본문 포함 필수" 로 갱신. 디렉토리 삭제·정리 commit·CI 재통과 항목 제거. (2) "하네스 메타 변경 단축 경로" 에 `docs/skills/` 전용 메타 변경의 [9] 단독 적용 예외 조항 추가 |
| `docs/harness-ship.md` | L45-58 "머지 직전 PRD 정리 (정책)" 섹션 → "PR 생성 시 PRD 요약 본문 포함 (정책)" 로 갱신. 사후 정리 = 없음 |
| `docs/skills/rp-ship.md` | 단계 8 (본문 임베드) 을 단계 5 (PR 생성) 안에 통합. 단계 9 (디렉토리 삭제 commit) · 10 (CI 재대기) 제거. 단계 11·12 → 8·9 로 번호 갱신 |
| `docs/skills/rp-dev.md` | "테스트 규칙" 섹션에 "변경 영향 범위 테스트 필수" 1줄 추가 |
| `docs/skills/rp-code-review.md` | "PR 유형별 포커스" 다음에 "변경 영향 범위 테스트 검증 (필수)" 짧은 섹션 추가. 누락 시 항목 4 (테스트 품질) -1 감점 |
| `.codex/skills/rp-{ship,dev,code-review}/SKILL.md` | docs/skills 변환본 — sync 스크립트가 자동 동기화 |

## 롤백 전략

| 단계 | 조치 |
|------|------|
| 머지 전 | PR close 만으로 회복. 추가 commit 불필요 |
| 머지 후 | `git revert <merge-sha>` 단일 commit. 변경 범위가 docs/skills 만이라 코드 동작 영향 0 |

## 검증

| 게이트 | 결과 |
|--------|------|
| codex 스킬 동기화 (`python3 scripts/sync-codex-skills.py --check`) | ✓ in sync: 13 |
| 변경 파일 grep — 잔존 "PRD 정리"·"머지 직전 PRD"·"디렉토리 삭제" commit 메시지 패턴 참조 | ✓ 0건 (수정한 3개 파일 외) |
| 후속 rp-ship 동작 | 다음 ship 사이클에서 PR 본문 PRD 요약 임베드가 PR 생성 시점에 들어가는지 + PRD 디렉토리 삭제 commit 이 생성되지 않는지 실측 |
