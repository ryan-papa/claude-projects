# Meta PRD Review (Claude, r1) — PR-A Critical

| 항목 | 점수 | 근거 |
|------|:----:|------|
| 1. 변경 이유 명확성 | 9 | C1·C2·C3 이슈 ID·증거(누락 파일 5건, 인바운드 링크 0건, 600자+ bullet)·영향까지 표로 명시. "직전 감사 평균 6.7" 출처 링크는 미첨부 |
| 2. 영향 파일 정확성 | 7.5 | 2개 파일 정확 식별, "신규 파일·스킬 본문 변경 없음" 명시. 단 `harness-workflow.md` 어느 영역에 마크다운 링크 보강(L8~30 본문 vs L41~47 보조 링크 블록)인지 위치 미지정 |
| 3. 롤백 안전성 | 9 | `git revert <merge-commit>` 단일 명령, 통합 브랜치 격리로 main 영향 0. 부분 롤백 시나리오는 생략됐으나 단축 경로 범위에서 적정 |
| 4. 검증 방법 구체성 | 7.5 | find·grep·awk·wc 모두 구체화. 단 `grep -c "skills/rp-{task,specify,...}.md"`는 brace expansion이 grep에서 동작 안 함 — `grep -cE`+pipe 또는 `-e` 다중화 필요. "awk 길이 검증" 명령 미기재 |
| 5. 기존 하네스 규칙 정합성 | 9 | 단축 경로(`rp-init`·`rp-specify`·`rp-task`·`rp-dev` 스킵 + feat 브랜치 + 간소 PRD + 리뷰 + `rp-ship`) 준수, 200줄 한도(현 165/142) 보호, 4섹션 구조 일치, 통합 브랜치 표기 규칙 일치 |
| 6. 범위 적절성 | 9 | Critical 3건만 묶음, 신규 기능 0건, 스킬 본문 미변경. 3개 PR 묶음 중 1번이라는 분리 명시로 메타 단축 경로에 부합 |
| 평균 | 8.5 | |
| 최저 | 7.5 | |

## 판정
PASS

## 지적사항
- 검증 §4 grep 명령 brace expansion 무동작: `grep -cE 'skills/rp-(task|specify|dev|qa|code-review|amend)\.md' docs/harness-workflow.md` 형태로 수정 필요 (실행 시 0 반환되어 검증 실패 오인 위험)
- 영향 §2 `harness-workflow.md` 링크 보강 위치 명확화: 12단계 본문 표(L8~30 ASCII 다이어그램은 코드 블록이라 마크다운 링크 미렌더) vs 보조 링크 블록(L41~47 이미 존재) 중 어디에 추가하는지 PRD에 1줄 추가 권장
- 변경 이유 §1 "감사 평균 6.7" 근거 파일 경로(예: `docs/audits/...`) 링크 보강 권장 — 추후 회고 추적성
- 검증 §4 "awk 길이 검증" 실제 명령 1줄 예시 추가 권장 (`awk '/^- / && length>600' CLAUDE.md | wc -l` 등)
- 롤백 §3에 "C1·C2·C3 중 일부만 문제 시" 부분 롤백 절차는 생략됐으나 단일 PR 범위에서 허용
