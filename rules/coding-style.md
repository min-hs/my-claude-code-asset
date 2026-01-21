# 코딩 스타일 규칙

임베디드 C/C++ 코딩 스타일 가이드입니다.

## 네이밍 규칙

### 함수명
```c
// snake_case 사용
int parse_message(uint8_t* buf, size_t len);
static void handle_timeout(void);
```

### 구조체/열거형
```c
// PascalCase 사용
typedef struct {
    uint8_t type;
    uint16_t length;
} MessageHeader;

typedef enum {
    STATE_IDLE,
    STATE_CHARGING,
    STATE_ERROR
} ChargeState;
```

### 매크로/상수
```c
// UPPER_SNAKE_CASE 사용
#define MAX_BUFFER_SIZE     1024
#define CHARGE_TIMEOUT_MS   5000
```

### 변수명
```c
// snake_case, 의미 있는 이름
uint8_t rx_buffer[MAX_BUFFER_SIZE];
int32_t current_temperature;
bool is_charging;
```

## 포맷팅

### 들여쓰기
- 4 spaces (탭 금지)

### 중괄호
```c
// K&R 스타일
if (condition) {
    do_something();
} else {
    do_other();
}

// 함수는 새 줄
void function_name(void)
{
    // body
}
```

### 라인 길이
- 최대 100자

## 코드 구조

### 함수 길이
- 최대 50줄
- 초과 시 분리

### 파일 길이
- 최대 800줄
- 초과 시 모듈 분리

### 중첩 깊이
- 최대 3단계
```c
// 나쁜 예
if (a) {
    if (b) {
        if (c) {
            if (d) {  // 4단계 - 금지
            }
        }
    }
}

// 좋은 예: early return
if (!a) return;
if (!b) return;
if (!c) return;
// 로직 수행
```

## 주석

### 파일 헤더
```c
/**
 * @file    message_parser.c
 * @brief   V2G 메시지 파싱 모듈
 * @author  min-dev
 * @date    2024-01-01
 */
```

### 함수 주석
```c
/**
 * @brief   메시지를 파싱한다
 * @param   buf     입력 버퍼
 * @param   len     버퍼 길이
 * @param   msg     결과 메시지 구조체
 * @return  0: 성공, 음수: 에러
 */
int parse_message(uint8_t* buf, size_t len, Message* msg);
```

### 인라인 주석
```c
// 한국어 주석 권장
uint32_t timeout = 5000;  // 충전 타임아웃 (ms)
```

## 금지 사항
- 매직넘버 하드코딩
- goto 문 (에러 처리 제외)
- 전역 변수 남용
- 암묵적 타입 변환
