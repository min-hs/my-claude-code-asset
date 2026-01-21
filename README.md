# min-dev의 Claude Code 설정

임베디드 소프트웨어 개발자를 위한 Claude Code 설정 파일입니다.

## 빠른 설치

다른 컴퓨터에서 Claude Code에게 이렇게 말하세요:
```
https://github.com/min-hs/my-claude-code-asset 이 레포의 설정 파일들을 ~/.claude/에 복사해서 적용해줘
```

## 수동 설치
```bash
# 1. 레포 클론
git clone https://github.com/min-hs/my-claude-code-asset.git
cd my-claude-code-asset

# 2. 설정 파일 복사
cp CLAUDE.md ~/.claude/
cp settings.json ~/.claude/
cp -r agents ~/.claude/
cp -r commands ~/.claude/
cp -r rules ~/.claude/

# 3. (선택) 터미널 alias 추가
echo 'alias c="claude"' >> ~/.zshrc
echo 'alias cc="claude code"' >> ~/.zshrc
source ~/.zshrc
```

## 포함된 기능

### 에이전트 (4개)

| 에이전트 | 용도 |
|---------|------|
| planner | 복잡한 기능 계획 수립 |
| code-reviewer | 코드 품질/보안 리뷰 |
| architect | 시스템 아키텍처 설계 |
| security-reviewer | 보안 취약점 분석 |

### 슬래시 커맨드 (9개)

| 커맨드 | 용도 |
|--------|------|
| /plan | 작업 계획 수립 |
| /commit-push-pr | 커밋 → 푸시 → PR 한 번에 |
| /verify | 빌드, 린트, 테스트 검증 |
| /review | 코드 리뷰 |
| /simplify | 코드 단순화 |
| /tdd | 테스트 주도 개발 |
| /build-fix | 빌드 에러 수정 |
| /handoff | HANDOFF.md 생성 (세션 인계) |
| /compact-guide | 컨텍스트 관리 가이드 |

### 규칙 (6개)

| 규칙 | 내용 |
|------|------|
| coding-style | 네이밍, 포맷팅, 한글 주석 규칙 |
| memory-safety | 메모리 할당/해제, 버퍼 오버플로 방지 |
| interrupt-safety | ISR 규칙, 공유 자원 보호 |
| error-handling | 에러 코드, 리소스 정리, 재시도 전략 |
| protocol-rules | ISO 15118, OCPP, Modbus 프로토콜 규칙 |
| build-debug | Makefile, 컴파일러 경고, 디버그 매크로 |

## 주요 특징

- **임베디드 C/C++ 개발 최적화**: make, gcc, gdb, valgrind 등 지원
- **프로토콜 지원**: ISO 15118, DIN 70121, OCPP, Modbus
- **메모리 안전**: 정적 할당 우선, 동적 할당 규칙
- **인터럽트 안전**: ISR 규칙, 크리티컬 섹션
- **한국어 주석**: 복잡한 로직에 한글 설명

## 라이선스

MIT License
