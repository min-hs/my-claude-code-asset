# Handoff 커맨드

세션 종료 전 HANDOFF.md 파일을 생성합니다.

## 실행 순서

1. 현재 상태 파악
```bash
git status
git log --oneline -5
```

2. HANDOFF.md 생성
```markdown
# Handoff Document

## 날짜
YYYY-MM-DD

## 완료된 작업
- [ ] 작업1 설명
- [ ] 작업2 설명

## 다음 해야 할 작업
1. 작업1
2. 작업2

## 주의사항
- 알려진 이슈
- 임시 코드 위치
- 의존성 문제

## 관련 파일
- `src/파일1.c` - 설명
- `include/파일2.h` - 설명

## 빌드 상태
- 빌드: ✅ 통과 / ❌ 실패
- 정적분석: ✅ 통과 / ❌ 실패

## 메모
추가 참고사항
```

3. 파일 저장 확인
```bash
cat HANDOFF.md
```

## 주의사항
- /clear 전에 반드시 실행
- 구체적으로 작성 (다음 세션의 나를 위해)
- 파일 경로는 정확하게 기록
