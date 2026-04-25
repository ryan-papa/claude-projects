# Codex Meta Review — 하네스 백엔드 테스트 정책

**상태:** PASS (필수 반영 후 재검토)

## r1 지적 (CRITICAL · HIGH · MEDIUM)

### CRITICAL
museum-finder/CLAUDE.md L208~ "DataJpaTest Specification 부재 = N/A" 조항이 그대로 남아 있어 신 루트 4-게이트와 충돌. #59 JPA path 회귀 재발 우회.
**반영**: 본 PR r2 — `repositories/museum-finder/CLAUDE.md` "테스트 정책" 섹션 정정. 루트 정책 우선 명시 + "DataJpaTest 부재 = N/A" 폐기 + DB 통합 트랜잭셔널 테스트 의무 추가.

### HIGH
DB 통합 게이트 fail-closed 미흡. POSTGRES_URL 없으면 skip 되고 CI 미설정이라 자동화 훅 부재.
**반영**: 본 PR r2 — 정책 §2.3 에 "fail-closed 증거 의무" 행 추가. 로컬 POSTGRES_URL 셋 후 통합 테스트 출력 캡처를 `qa-smoke.md` 에 의무, 미캡처 시 `rp-ship` 차단. 4-stroke 도 단순 "통과" 한 줄 금지·4 단계 출력 의무화.

### MEDIUM
4-stroke 증거 강제력 부족 + O-1 (다른 프로젝트 합류 자동화) 후속.
**반영**: 4-stroke 증거 의무는 r2 에서 강화. O-1 은 후속 PR 로 유지 (PRD 명시).

## Verdict
- r1: block
- r2 (필수 반영 후): pass
