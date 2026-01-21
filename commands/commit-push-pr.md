# Commit-Push-PR 커맨드

커밋, 푸시, PR 생성을 한 번에 수행합니다.

## 실행 순서

1. 변경사항 확인
```bash
git status
git diff --stat
```

2. 스테이징
```bash
git add -A
```

3. 커밋 (형식 준수)
```bash
git commit -m "[타입] 제목

본문 (선택)

Co-Authored-By: Claude <noreply@anthropic.com>"
```

4. 푸시
```bash
git push origin <branch-name>
```

5. PR 생성
```bash
gh pr create --title "[타입] 제목" --body "## 변경사항
- 항목1
- 항목2

## 테스트
- [ ] 빌드 통과
- [ ] 정적 분석 통과"
```

## 커밋 타입
- feat: 새 기능
- fix: 버그 수정
- docs: 문서 수정
- style: 코드 포맷팅
- refactor: 리팩토링
- test: 테스트 추가
- chore: 빌드, 설정 변경

## 주의사항
- main/master 직접 푸시 금지
- PR 전 빌드 확인 필수
