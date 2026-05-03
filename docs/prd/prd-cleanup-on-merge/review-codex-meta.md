# Codex Meta Review — prd-cleanup-on-merge

실행: `codex review --base main` (codex-cli 0.128.0, exit=0, < 300s)
대상: `docs/prd/prd-cleanup-on-merge/prd.md` + diff vs `main` (5 files, +124/-4)
판정: P2 1건 (자발 반영) → ship 진행

## 지적 요약

| Priority | 항목 | 처리 |
|:--------:|------|------|
| P2 | Handle simplified PRDs without Full PRD headings (`docs/skills/rp-ship.md:73`) — 신설 단계 8 추출 섹션이 `## 개요·목적`/`## 기능 요구사항`만 가정. 간소 PRD(본 dogfood PR 포함)는 `## 변경 이유`/`## 영향 파일`/`## 검증` 섹션 사용. 간소 PRD ship 시 추출 실패 또는 불완전 임베드 위험. 권고: fallback 섹션 매핑 정의 또는 간소 PRD 템플릿에 Full PRD 헤딩 필수화 | **자발 반영** — P2(참고 등급)이나 dogfood 자체 영향(본 PR 머지 시 첫 적용)이라 즉시 반영. `rp-ship.md` 단계 8에 PRD 유형별 추출 매핑 추가 (Full = 개요·목적/기능 요구사항/Review 결과 / 간소 = 변경 이유/영향 파일/검증/Review 결과). 추출 누락·`gh pr edit` 1회 재시도 실패 시 ship 중단 분기 신설. `harness-ship.md` 정책 표 + PRD `## 영향 파일` 핵심 동작 동기화 |

High/Critical 0건. P2 1건 자발 반영 → 메타 리뷰 게이트 충족 → ship 진행.

## 마스킹 검증

raw stdout 내 `api[_-]?key|token|secret` 패턴 매칭값 0건 확인 (codex review가 환경 시크릿을 출력하지 않음).

## 비고

raw stdout 651줄은 200줄 lint 제한 초과로 본 요약본으로 대체 저장. 기존 PRD(`codex-cwd-restore` 등) 동일 패턴.

## 반영

| Codex 지적 | 등급 | 반영 |
|-----------|:----:|------|
| Handle simplified PRDs without Full PRD headings | P2 | rp-ship.md 단계 8 PRD 유형별 추출 매핑 + 임베드 실패 1회 재시도 후 ship 중단 분기. harness-ship.md 정책 표 + PRD 핵심 동작 동기화. 본 PR(dogfood) 머지 시 정상 추출 검증 |
