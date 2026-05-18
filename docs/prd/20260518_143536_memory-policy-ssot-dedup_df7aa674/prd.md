---
**유형:** 하네스 메타 변경
**일시:** 2026-05-18 14:35 KST
**브랜치:** `feat/memory-policy-ssot-dedup`
---

# 메모리 시스템 비활성 정책 SSOT 단일화 (중복 제거)

## 변경 이유

| 문제 | Codex stop-time review 지적 | 해결 |
|------|----------------------------|------|
| PR #47에서 §⛔ 메모리 시스템 비활성 본문이 `CLAUDE.md`와 `harness-absolute-rules.md` 두 곳에 모두 들어가 SSOT 룰 위반 (절대 규칙: "SSOT — 다른 문서·스킬에서 동일 규칙 본문 중복 정의 금지, 링크 참조만 허용") · 인프라 재기동 정책(SSOT 본문 + CLAUDE.md 1줄 가드) 패턴과도 일관성 결여 | "SSOT rule is duplicated and internally contradictory" | (1) CLAUDE.md §⛔ 메모리 시스템 비활성 본문 절 제거 (2) SSOT 링크 그룹에 1줄 가드만 남김 — 인프라 재기동 가드와 동일 패턴 (3) SSOT 본문에서 일자성 표현·중복 진술 정제 |

## 영향 파일

| 파일 | 변경 내용 |
|------|----------|
| `CLAUDE.md` | §⛔ 메모리 시스템 비활성 본문 절 제거 + SSOT 링크 그룹에 1줄 가드 추가 (인프라 재기동 가드 다음 줄) — `**⛔ auto-memory 시스템 비활성** — ... 상세: SSOT §메모리 시스템 비활성` |
| `docs/harness-absolute-rules.md` | §메모리 시스템 비활성 본문 정제 — 일자 표현("2026-05-18 완전 삭제 완료") 제거, override·CLAUDE.md 직접 추가 경로·위반 검출 규칙 유지 |

(메모리 디렉토리 자체는 PR #47에서 이미 삭제됨 — 본 PR 영향 무관)

## 롤백 전략

- `git revert` 1건 — 중복 본문 복원. 단 SSOT 룰 위반 상태 회귀라 추천하지 않음
- 코드·동작 영향 0 (문서 위치 이동만)

## 검증

- [x] CLAUDE.md에서 §⛔ 메모리 시스템 비활성 본문 절 제거 확인
- [x] CLAUDE.md SSOT 링크 그룹에 메모리 가드 1줄 추가 (인프라 재기동 가드 패턴과 일치)
- [x] harness-absolute-rules.md §메모리 시스템 비활성 본문 정제 (override·"기억해" 처리·위반 검출 규칙 유지)
- [x] SSOT 룰 위반 해소: 본문이 SSOT 1곳에만 존재
- [x] 두 정책(인프라 재기동·메모리 비활성)이 동일 패턴 (CLAUDE.md 1줄 가드 + SSOT 본문) 적용
