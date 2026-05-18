---
**유형:** 하네스 메타 변경
**일시:** 2026-05-18 14:22 KST
**브랜치:** `feat/infra-restart-permission-rule`
---

# 인프라 재기동 사용자 허락 절대 규칙 — SSOT 반영

## 변경 이유

| 문제 | 사용자 피드백 | 해결 |
|------|--------------|------|
| AI 어시스턴트(Claude·Codex)·자동화 스크립트가 일반 결정 위임 신호("ㄱㄱ"·"진행해")를 인프라 재기동(colima·k3s·노드 reboot 등) 동의로 해석할 위험 | "인프라 재시작은 반드시 나의 허락을 받고 하도록 강하게 readme, claude.md 이런데 다 넣어줘" | 하네스 절대 규칙 SSOT(`harness-absolute-rules.md`)에 §인프라 재기동·다운타임 작업 절 신설 + 진입점 CLAUDE.md에서 SSOT 링크 보강 |

레포별 상세는 해당 레포 CLAUDE.md·README.md(예: `repositories/mac-mini-infra/`)에 별도 PRD로 반영됨 (mac-mini-infra PR #56). 본 PRD는 하네스 SSOT 측 1줄짜리 가드 명시.

## 영향 파일

| 파일 | 변경 내용 |
|------|----------|
| `docs/harness-absolute-rules.md` | §인프라 재기동·다운타임 작업 절 신설 — 대상 작업 5종 + 일반 결정 위임 신호 불충분 + 영속성 검증 colima restart도 동일 규칙 + 리뷰[4·5·9] 즉시 fail + 레포별 상세 위치 안내 |
| `CLAUDE.md` | "⛔ 하네스 절대 규칙" 줄 다음에 인프라 재기동 가드 1줄 추가 — SSOT §인프라 재기동·다운타임 작업 + `repositories/mac-mini-infra/CLAUDE.md` §⛔ 인프라 재기동 결정 링크 |

(코드 영향 0 — 스킬·스크립트·하네스 동작 변경 없음. 정책 문서 신설만)

## 롤백 전략

- 정책 자체 롤백 필요 시 `git revert` 1건 — 본 정책이 운영 사고를 유발한 경우(예: 자동 재기동이 필요했던 시점에 차단되어 SLO 위반)에만 검토
- 문서 변경이라 즉시 적용·즉시 롤백 모두 코드 영향 0

## 검증

- [x] `harness-absolute-rules.md` §인프라 재기동·다운타임 작업 절 추가 — 대상 작업 + 적용 원칙 + 리뷰 게이트
- [x] `CLAUDE.md` 절대 규칙 줄 다음 인프라 재기동 가드 1줄 추가 — SSOT + 레포별 정책 링크
- [x] "ㄱㄱ" 같은 일반 결정 위임 신호로는 불충분함을 SSOT에 명시
- [x] AI 어시스턴트(Claude·Codex)·자동화 스크립트가 동일 규칙 적용 대상임을 SSOT에 명시
- [x] 영속성 검증을 위한 `colima restart`도 동일 규칙임을 SSOT에 명시
- [x] mac-mini-infra PR #56과 일관성 유지 — 두 레포 정책 문서가 서로 참조
- [ ] (운영 측면 검증) 다음 사이클에서 재기동성 작업 요청 시 본 정책에 따라 사용자 명시 동의 후 진행되는지 확인
