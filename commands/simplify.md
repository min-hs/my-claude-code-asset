# Simplify 커맨드

코드를 단순화하고 가독성을 개선합니다.

## 실행 순서

1. 대상 코드 분석
- 복잡도 확인
- 중복 코드 탐지
- 긴 함수 식별

2. 단순화 기법 적용

### 함수 분리
```c
// Before: 100줄 함수
void process_message(uint8_t* buf, size_t len) {
    // 파싱 로직 50줄
    // 처리 로직 30줄
    // 응답 로직 20줄
}

// After: 분리된 함수들
static int parse_message(uint8_t* buf, size_t len, Message* msg);
static int handle_message(Message* msg, Response* resp);
static int send_response(Response* resp);
```

### 조건문 단순화
```c
// Before
if (state == STATE_A) {
    if (event == EVENT_1) {
        // ...
    }
}

// After: 테이블 기반
typedef void (*Handler)(void);
Handler state_table[STATE_MAX][EVENT_MAX] = { ... };
state_table[state][event]();
```

### 매직넘버 제거
```c
// Before
if (timeout > 5000) { ... }

// After
#define CHARGE_TIMEOUT_MS  5000
if (timeout > CHARGE_TIMEOUT_MS) { ... }
```

3. 리팩토링 후 검증
```bash
make clean && make
cppcheck --enable=all src/
```

## 단순화 체크리스트
- [ ] 함수 50줄 이하
- [ ] 중첩 깊이 3단계 이하
- [ ] 매개변수 5개 이하
- [ ] 중복 코드 제거
- [ ] 명확한 변수명

## 주의사항
- 동작 변경 없이 구조만 개선
- 한 번에 하나씩 리팩토링
- 리팩토링 후 반드시 테스트
