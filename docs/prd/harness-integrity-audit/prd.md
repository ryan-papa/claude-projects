# 간소 PRD — 하네스 정합성 검사 일괄 반영

**유형:** 하네스 메타 변경

## 변경 이유

3개 Explore 서브에이전트 병렬 검사로 발견된 정합성 결함을 일괄 해소. 동시에 (1) 레포명 교체(`claude-projects` → `workflow-agent-harness`), (2) README 최신화(170→164줄, 12단계·작성 모드·서브레포 참조 금지 반영), (3) 무관 임시 파일 정리.

| 심각도 | 건수 | 핵심 해결 |
|--------|:--:|----------|
| Critical | 3 | 회고 자동 진입 정합(5개 문서) + 원본 부재 5종 예외 명기 + 스킬 인덱스 [10] 추가 |
| High | 7 | ⛔ 서브레포 참조 금지 정책 신설 + 6건 모순 해소 |
| Medium | 6 | 간소 PRD 추출 의도 명문화 + 재시도 한도 "총 3회" 명시 + 3개 스킬 SSOT 링크 참조로 정리 등 |
| Low | 2 | 표시 텍스트·용어 일관성 |

추가 정리:
- 무관 PRD 1건 삭제 (`docs/prd/pg-mcp-server-setup`)
- 루트 임시 파일 10건 삭제 (SESSION-RESUME × 8, mockup × 2 — 다른 서비스 세션 흔적)
- 빈 `docs/images/` 디렉터리 + CLAUDE.md 트리 항목 제거
- git remote URL 정정 (`claude-projects.git` → `workflow-agent-harness.git`)

신규 정책 (SSOT 등록):
- 하네스 회고 발췌본은 `docs/research/`, 서비스 회고는 `repositories/[project]/docs/research/`로 위치 분리
- 하네스 문서에서 서브레포 내부 파일 마크다운 링크 직접 참조 금지 — 위반 시 리뷰[9] 정합성 fail
- 메타 변경 단축 경로 = [4][5][9] 전부 적용 (rp-eng-review 누락 보정). **단**, `docs/skills/` 전용 메타 변경은 SSOT 예외에 따라 [9]만
- 기술 실패 재시도 = "1차 + 추가 2회 = 총 3회" 명시
- 스킬-단독 운영 5종(`rp-workflow·rp-amend·rp-init·rp-specify·rp-retro`) — 원본 `harness-*.md` 없음, **SSOT `harness-absolute-rules.md` "단축 경로·예외"에 명문화**
- 회고 자동 진입 정책 정정 — `rp-amend.md`까지 일관 반영

## 영향 파일

| 분류 | 파일 |
|------|------|
| 진입점·트리 | `CLAUDE.md`, `README.md` |
| SSOT·정책 | `docs/harness-absolute-rules.md`, `docs/harness-workflow.md`, `docs/harness-ship.md`, `docs/harness-codex-review.md` |
| 스킬 | `docs/skills/rp-workflow.md`, `rp-ship.md`, `rp-plan-review.md`, `rp-eng-review.md`, `rp-code-review.md`, `rp-init.md` |
| 보안 문서 | `docs/security-guide.md`, `docs/security/secrets-management.md` |
| 템플릿 | `docs/templates/readme-opensource.md` |
| 기존 PRD 참조 정정 | `docs/prd/harness-review-aop-concurrency-checklist/prd.md`, `docs/prd/harness-ship-auto-base/prd.md` |
| 신규 (발췌본) | `docs/research/retro-T22-aop-concurrency.md`, `docs/research/retro-T23-ship-base.md` |
| 삭제 | `docs/prd/pg-mcp-server-setup/` + 루트 임시 10건 + `docs/images/` |
| 스킬 (추가) | `docs/skills/rp-amend.md` (회고 자동 진입 정책 일관 반영) |
| Codex 동기화 | `.codex/skills/rp-*/SKILL.md` 7건 (스크립트 자동 갱신) |

**본 PR 범위 외 (사전 상태, 후속 PR로 분리):**
- `.claude/settings.json` (87줄 키 재배치 + 신규 플러그인 enable — `extraKnownMarketplaces` 출처 선언 동반 필요, 보안 신뢰 체인 별도 검토)
- `.gitignore` (+`.gstack/`)

코드 동작 영향 0건. `repositories/[project]/` 산하 0 변경.

## 롤백 전략

- 단일 PR 단위 `git revert` 가능 — 27+개 파일 변경 + 신규 2 + 삭제 11(+디렉터리 1)이 한 머지 커밋에 묶임
- **부분 revert 가이드**: 신규 정책 추가 chunk(`harness-absolute-rules.md` "⛔ 서브레포 참조 금지 / 회고 위치 정책" 절 + "단축 경로·예외" 절 5종 스킬-단독 행 + `harness-codex-review.md` "적용 단계" 절 보충, `docs/research/*`)와 단순 치환 chunk(claude-projects→workflow-agent-harness)는 별도 보정 PR로 분리 revert 가능
- git remote URL 변경은 로컬 .git/config만 영향 — `git remote set-url origin <구 URL>`로 즉시 원복
- Codex 변환본은 `rtk python3 scripts/sync-codex-skills.py`로 항상 재생성 가능
- 삭제된 임시 파일은 git 미추적이었으므로 복구 불필요. `pg-mcp-server-setup` PRD는 revert 시 자동 복원

## 검증

- ✅ 사전 검증 (검색 범위: `docs/` `CLAUDE.md` `AGENTS.md` `README.md` `.codex/` — `settings.json`·`.gitignore`는 본 PR 범위 외, `settings.local.json`은 `.gitignore` 추적 외): `grep -rE "\]\(.*repositories/[a-z-]+/[^)]+\)" docs/ CLAUDE.md AGENTS.md` 마크다운 서브레포 링크 0건 / `grep "claude-projects" docs/ CLAUDE.md AGENTS.md README.md .codex/` 잔존 0건 (리뷰[5] 지적 반영: `.codex/skills/rp-init/SKILL.md` 잔존 1건은 sync 재실행으로 해결)
- ✅ Codex 스킬 동기화: `rtk python3 scripts/sync-codex-skills.py --check` → "in sync: 13"
- ⏳ **후속 PR 권고**: 신규 정책 회귀 감지 CI 가드 추가 (`.github/workflows/ci.yml`에 `grep -rn "claude-projects"` + `grep -rnE "\]\(.*repositories/[a-z-]+/[^)]+\)"` 두 가드)
- ⏳ 리뷰[4] 기획: Claude 서브에이전트 + Codex 1회
- ⏳ 리뷰[5] 엔지니어링: Claude 서브에이전트 + Codex 1회
- ⏳ 리뷰[9] 코드: Claude 서브에이전트 + Codex 1회
- ⏳ CI: 자동 머지 가드 3종 AND (SUCCESS · base 정상 · MERGEABLE) 충족 시 자동 머지
