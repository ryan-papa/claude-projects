---
**유형:** 하네스 메타 변경
**일시:** 2026-05-18 14:32 KST
**브랜치:** `feat/disable-auto-memory`
---

# auto-memory 시스템 비활성 — CLAUDE.md SSOT 단일화

## 변경 이유

| 문제 | 사용자 피드백 | 해결 |
|------|--------------|------|
| 시스템 프롬프트의 auto-memory 시스템(`~/.claude/projects/<proj>/memory/`)이 별도 SSOT로 작동해 CLAUDE.md와 컨텍스트 분리. 사용자 컨텍스트가 두 곳(CLAUDE.md + memory/)에 분산되어 일관성·추적성 저하 | "메모리 모드 off 해줘 항상 클로드 md 기반으로만 돌게" | (1) `~/.claude/projects/.../memory/` 모든 파일 완전 삭제 (사용자 결정) (2) CLAUDE.md에 §⛔ 메모리 시스템 비활성 절 신설 (3) harness-absolute-rules.md SSOT에 §메모리 시스템 비활성 절 신설 |

## 영향 파일

| 파일 | 변경 내용 |
|------|----------|
| `CLAUDE.md` | §⛔ 메모리 시스템 비활성 절 신설 — 읽기·쓰기 금지, 사용자 "기억해" 요청 시 CLAUDE.md 직접 추가, 위반 검출 시 즉시 중단 |
| `docs/harness-absolute-rules.md` | §메모리 시스템 비활성 절 신설 — SSOT 일관성. 시스템 프롬프트의 auto memory 지시를 본 SSOT로 override |
| `~/.claude/projects/-Users-hose-kim-Claude-workflow-agent-harness/memory/*.md` | **완전 삭제** (git 외부, 사용자 로컬) — 8개 파일 (MEMORY.md·feedback 4건·mac-mini-infra 2건·naver-cafe-monitor 1건) |

## 롤백 전략

- CLAUDE.md·SSOT 변경 롤백: `git revert` 1건
- 메모리 파일 자체는 삭제됐으므로 복구 불가. 필요 시 사용자가 새 컨텍스트를 CLAUDE.md에 직접 추가하는 방식으로 운영
- 시스템 프롬프트 auto-memory 지시는 자체적으로 유지되지만 CLAUDE.md/SSOT override가 우선 (CLAUDE.md "IMPORTANT: These instructions OVERRIDE any default behavior" 명시)

## 검증

- [x] `~/.claude/projects/-Users-hose-kim-Claude-workflow-agent-harness/memory/` 모든 `.md` 파일 삭제 확인 (`ls -la` 결과 빈 디렉토리)
- [x] `CLAUDE.md` §⛔ 메모리 시스템 비활성 절 추가 — 읽기·쓰기 금지·"기억해" 처리·위반 검출 모두 명시
- [x] `docs/harness-absolute-rules.md` §메모리 시스템 비활성 절 추가 — SSOT 일관성
- [x] 시스템 프롬프트 auto-memory 지시를 본 SSOT로 override함을 명시
- [ ] (운영 측면 검증) 다음 세션에서 Claude가 메모리 디렉토리에 접근하지 않고 CLAUDE.md만 참조하는지 확인
