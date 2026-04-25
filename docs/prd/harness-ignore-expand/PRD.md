# PRD — 하네스 ignore 패턴 확장 + .codexignore 신설

| 항목 | 값 |
|------|----|
| 유형 | 하네스 메타 변경 (간소 PRD, 4섹션) |
| 브랜치 | `feat/harness-ignore-expand` |
| 영향 범위 | 루트 `.claudeignore`, 루트 `.codexignore` (신규) |

## 1. 변경 이유

- Claude/Codex 세션이 `repositories/*/` 하위 대용량 디렉토리·바이너리를 반복 스캔해 토큰 소모가 큼.
- 현재 `.claudeignore`는 표준 빌드 산출물 위주이며, 실제 레포에서 발견되는 다음 패턴이 누락됨.
  - 프레임워크 캐시: `.astro/`(naver-cafe-monitor/web), `.docusaurus/`, `.svelte-kit/` 등
  - sops 산출물: `.env.enc`(museum-finder, naver-cafe-monitor 실재) — 암호문 읽어도 무가치
  - 탐색용: `spike/`(museum-finder 11M 실재)
  - 바이너리/데이터: `*.pdf`, `*.sqlite`, `*.parquet` 등
- Codex CLI도 동일 어댑터(AGENTS.md)를 사용하나 `.codexignore`가 부재 — 동일 규칙 미러 필요.

## 2. 영향 파일

| 파일 | 변경 |
|------|------|
| `.claudeignore` | 섹션 4종 추가: 프레임워크 캐시 / IaC 상태 / sops 산출물 / 탐색용 + 미디어·데이터 패턴 보강 |
| `.codexignore` | 신규 (`.claudeignore` 동일 규칙, 헤더만 변경) |

기존 규칙 삭제·수정 없음. 추가만 발생.

## 3. 롤백

- `git revert <commit>` 단발 커밋 — 즉시 원복 가능.
- 기능적 의존성 없음 (스킬·CI 미참조). 롤백 시 부수 효과는 토큰 절감 효과 소실뿐.

## 4. 검증

| # | 항목 | 방법 |
|---|------|------|
| 1 | 패턴 문법 유효성 | `git check-ignore -v` 로 샘플 경로 검증 (예: `repositories/museum-finder/.env.enc`, `repositories/naver-cafe-monitor/web/.astro`) |
| 2 | 기존 규칙 보존 | `diff` 로 추가-only 확인, 제거된 라인 0건 |
| 3 | `.codexignore` 미러 | 두 파일 본문(헤더 제외) `diff` 0 라인 |
| 4 | 의도 외 무력화 없음 | `repositories/**/src/`·`docs/`·`*.md` 등 코드/문서 경로가 무시되지 않는지 `git check-ignore` 샘플 |
