# rp-ship

커밋 → PR → CI 확인 → 머지 → 배포.

## 트리거

- 산출물 보고 후 **사용자 승인** 시
- `/rp-ship` 명령

## 절차

1. **커밋**: 변경 파일만 `git add` + `git commit`
2. **푸시**: `git push -u origin [branch]`
3. **PR 생성**: `gh pr create` (제목 + 변경 요약 + 테스트 계획)
4. **CI 확인**: 아래 분기 참조
5. **머지**: CI 통과 후 `gh pr merge --merge --auto`
6. **배포 확인**: 배포 워크플로우 완료 대기 + 결과 보고

## CI 분기

```
PR 생성 시점에 .github/workflows/ 확인
  ├── CI 있음 → gh pr merge --merge --auto (CI 통과 대기)
  └── CI 없음 → "CI가 없습니다. 추가할까요?" 사용자에게 질문
       ├── 추가 원함 → CI 워크플로우 생성 후 재푸시
       └── 추가 안 함 → 사용자에게 수동 머지 안내
```

## 절대 규칙

- **CI 통과 전 머지 금지** (예외 없음)
- `--admin` 플래그로 강제 머지 **금지**
- 사용자 승인 없이 진입 **금지**

## 머지 후 검증

- `git log [target-branch] --oneline -3`으로 커밋 반영 확인
- 배포 워크플로우 상태 확인: `gh run list --limit 1`
- 배포 완료 시 라이브 URL 안내

## 완료 조건

- PR 머지 완료
- 배포 워크플로우 성공
- 라이브 URL 사용자에게 전달

→ 배포 상세: [`../harness-ship.md`](../harness-ship.md)
