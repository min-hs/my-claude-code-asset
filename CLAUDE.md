# min-dev의 Claude Code 설정

## 개인 정보
- 이름: min-dev
- GitHub: min-hs
- 분야: 임베디드 소프트웨어 개발

## 핵심 마인드셋
**Claude Code는 시니어 개발자다.**
- 작업을 작게 쪼갤수록 결과물이 좋아진다
- "충전 기능 만들어줘" ❌
- "ISO 15118 메시지 파싱하고, V2G 세션 상태머신 구현하고, 타이머 핸들링 추가해줘" ✅

## 프롬프팅 베스트 프랙티스

### 1. Plan 모드 먼저 (가장 중요!)
```
Shift+Tab → Plan 모드 토글
복잡한 작업은 Plan 모드에서 계획 → 확정 후 구현
```

### 2. 구체적인 프롬프트
```
❌ "함수 만들어줘"
✅ "SECC에서 ChargeParameterDiscoveryReq 메시지를 파싱하는 함수 만들어줘.
   입력은 uint8_t 버퍼, 출력은 구조체 포인터. 에러 시 음수 반환."
```

### 3. 에이전트 체이닝
```
복잡한 작업 → /plan → 구현 → /review → /verify
```

## 컨텍스트 관리 (핵심!)
> 컨텍스트는 신선한 우유. 시간이 지나면 상한다.

### 규칙
- 토큰 80-100k 넘기 전에 리셋 (200k 가능하지만 품질 저하)
- 3-5개 작업마다 컨텍스트 정리
- `/compact` 3번 후 `/clear`

### 컨텍스트 관리 패턴
```
작업 → /compact → 작업 → /compact → 작업 → /compact
→ /handoff (HANDOFF.md 생성) → /clear → 새 세션
```

### HANDOFF.md 필수!
컨텍스트 리셋 전에 반드시 HANDOFF.md 생성
- 지금까지 뭐 했는지
- 다음에 뭐 해야 하는지
- 주의할 점

## 사용 가능한 커맨드

| 커맨드 | 용도 |
|--------|------|
| /plan | 작업 계획 수립 |
| /commit-push-pr | 커밋→푸시→PR 한 번에 |
| /verify | 빌드, 정적분석 검증 |
| /review | 코드 리뷰 |
| /simplify | 코드 단순화 |
| /build-fix | 빌드 에러 수정 |
| /handoff | HANDOFF.md 생성 |
| /compact-guide | 컨텍스트 관리 가이드 |

## 사용 가능한 에이전트

| 에이전트 | 용도 |
|----------|------|
| planner | 복잡한 기능 계획 |
| code-reviewer | 코드 품질/보안 리뷰 |
| architect | 아키텍처 설계 |
| security-reviewer | 보안 취약점 분석 |

## 프로젝트 정보

### SECC (Smart Electric Vehicle Communication Controller)
- EV 충전 프로토콜 구현 (ISO 15118, DIN 70121)
- V2G 통신 스택
- 위치: D:\01_Work\EVSE

### GridWiz 모니터링 시스템
- 충전 인프라 모니터링
- OCPP 프로토콜 연동
- 위치: D:\01_Work\GW_app

## 자주 사용하는 명령어
```bash
make            # 빌드
make build      # 빌드 (명시적)
make clean      # 클린 빌드 준비
make clean all  # 전체 클린 후 빌드
cmake ..        # CMake 설정
cppcheck --enable=all src/  # 정적 분석
```

## 금지 사항
- main/master 브랜치에 직접 push 금지
- 하드코딩된 매직넘버 금지 (#define 또는 const 사용)
- 동적 메모리 할당 후 해제 누락 금지
- 전역 변수 남용 금지
- printf 디버깅 커밋 금지

## 선호하는 기술 스택
- Language: C, C++
- Build: CMake, Make, GCC
- Debug: GDB, Valgrind
- Static Analysis: cppcheck, clang-tidy
- Protocol: ISO 15118, OCPP, Modbus
- OS : Linux

## 커밋 메시지 형식
```
[타입] 제목

본문 (선택)

Co-Authored-By: Claude <noreply@anthropic.com>
```
타입: feat, fix, docs, style, refactor, test, chore

## 작업 완료 후 체크리스트
- [ ] 빌드 통과 (make clean && make)
- [ ] 정적 분석 통과 (cppcheck)
- [ ] 메모리 누수 체크
- [ ] 매직넘버 제거
- [ ] 주석 작성 (한국어)

## 코딩 스타일
- 함수명: snake_case (예: parse_message)
- 구조체/열거형: PascalCase (예: ChargeParameter)
- 매크로: UPPER_SNAKE_CASE (예: MAX_BUFFER_SIZE)
- 들여쓰기: 4 spaces
- 함수 50줄 이하, 파일 800줄 이하
- 한국어 주석 권장

## Claude가 자주 실수하는 것 (여기에 추가)
<!-- Claude가 실수할 때마다 여기에 규칙 추가 -->
