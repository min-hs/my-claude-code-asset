# TDD 커맨드

테스트 주도 개발 방식으로 코드를 작성합니다.

## TDD 사이클
```
Red → Green → Refactor
(실패 테스트) → (통과 코드) → (리팩토링)
```

## 실행 순서

1. 테스트 먼저 작성
```c
// test_message_parser.c
#include "unity.h"
#include "message_parser.h"

void test_parse_valid_message(void) {
    uint8_t buf[] = {0x01, 0x02, 0x03};
    Message msg;
    
    int result = parse_message(buf, sizeof(buf), &msg);
    
    TEST_ASSERT_EQUAL(0, result);
    TEST_ASSERT_EQUAL(0x01, msg.type);
}

void test_parse_null_buffer(void) {
    Message msg;
    
    int result = parse_message(NULL, 0, &msg);
    
    TEST_ASSERT_EQUAL(-1, result);
}
```

2. 테스트 실행 (실패 확인)
```bash
make test
```

3. 최소한의 코드 구현
```c
int parse_message(uint8_t* buf, size_t len, Message* msg) {
    if (buf == NULL || msg == NULL) {
        return -1;
    }
    msg->type = buf[0];
    return 0;
}
```

4. 테스트 실행 (통과 확인)
```bash
make test
```

5. 리팩토링
- 코드 정리
- 중복 제거
- 테스트 재실행

## 임베디드 TDD 팁

### 하드웨어 의존성 분리
```c
// 인터페이스 (테스트 가능)
typedef struct {
    int (*read)(uint8_t* buf, size_t len);
    int (*write)(uint8_t* buf, size_t len);
} CommInterface;

// 실제 구현
CommInterface uart_comm = { uart_read, uart_write };

// 테스트용 Mock
CommInterface mock_comm = { mock_read, mock_write };
```

### 테스트 더블
- Stub: 고정된 값 반환
- Mock: 호출 검증
- Fake: 간단한 구현

## 테스트 체크리스트
- [ ] 정상 케이스
- [ ] 경계값 (0, MAX, overflow)
- [ ] 에러 케이스 (NULL, invalid)
- [ ] 타이밍 관련 케이스

## 주의사항
- 테스트 없이 코드 작성 금지
- 한 번에 하나의 테스트만
- 테스트 코드도 깔끔하게 유지
