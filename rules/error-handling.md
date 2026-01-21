# 에러 핸들링 규칙

임베디드 환경에서의 에러 처리 규칙입니다.

## 기본 원칙

1. **모든 함수는 에러를 반환할 수 있다**
2. **에러는 무시하지 않는다**
3. **에러 복구 또는 안전한 상태로 전환**

## 반환값 규칙

### 표준 반환 패턴
```c
// 성공: 0, 실패: 음수 에러코드
int function_name(params)
{
    if (error_condition) {
        return -ERROR_CODE;
    }
    return 0;  // 성공
}
```

### 에러 코드 정의
```c
// 프로젝트 공통 에러 코드
typedef enum {
    ERR_OK              =  0,
    ERR_NULL_PTR        = -1,
    ERR_INVALID_PARAM   = -2,
    ERR_BUFFER_FULL     = -3,
    ERR_TIMEOUT         = -4,
    ERR_COMM_FAIL       = -5,
    ERR_CRC_MISMATCH    = -6,
    ERR_OUT_OF_MEMORY   = -7,
    ERR_STATE_INVALID   = -8,
} ErrorCode;
```

### 에러 메시지 매핑
```c
const char* get_error_string(int err)
{
    switch (err) {
        case ERR_OK:            return "Success";
        case ERR_NULL_PTR:      return "Null pointer";
        case ERR_INVALID_PARAM: return "Invalid parameter";
        case ERR_TIMEOUT:       return "Timeout";
        default:                return "Unknown error";
    }
}
```

## 에러 체크 패턴

### 모든 반환값 체크
```c
int result;

result = init_uart();
if (result != ERR_OK) {
    LOG_ERROR("UART init failed: %d", result);
    return result;
}

result = init_timer();
if (result != ERR_OK) {
    LOG_ERROR("Timer init failed: %d", result);
    return result;
}
```

### Early Return 패턴
```c
int process_message(Message* msg)
{
    // 파라미터 검증
    if (msg == NULL) {
        return ERR_NULL_PTR;
    }
    
    if (msg->length > MAX_MSG_LEN) {
        return ERR_INVALID_PARAM;
    }
    
    if (!is_valid_type(msg->type)) {
        return ERR_INVALID_PARAM;
    }
    
    // 정상 처리
    return do_process(msg);
}
```

### 리소스 정리 패턴
```c
int complex_operation(void)
{
    int result = ERR_OK;
    uint8_t* buf1 = NULL;
    uint8_t* buf2 = NULL;
    
    buf1 = malloc(SIZE1);
    if (buf1 == NULL) {
        result = ERR_OUT_OF_MEMORY;
        goto cleanup;
    }
    
    buf2 = malloc(SIZE2);
    if (buf2 == NULL) {
        result = ERR_OUT_OF_MEMORY;
        goto cleanup;
    }
    
    result = do_work(buf1, buf2);
    
cleanup:
    free(buf2);  // NULL이어도 안전 (C99)
    free(buf1);
    return result;
}
```

## 에러 로깅

### 로그 레벨
```c
typedef enum {
    LOG_LEVEL_ERROR,    // 에러 (복구 불가)
    LOG_LEVEL_WARN,     // 경고 (복구 가능)
    LOG_LEVEL_INFO,     // 정보
    LOG_LEVEL_DEBUG     // 디버그
} LogLevel;

#define LOG_ERROR(fmt, ...)  log_print(LOG_LEVEL_ERROR, fmt, ##__VA_ARGS__)
#define LOG_WARN(fmt, ...)   log_print(LOG_LEVEL_WARN, fmt, ##__VA_ARGS__)
#define LOG_INFO(fmt, ...)   log_print(LOG_LEVEL_INFO, fmt, ##__VA_ARGS__)
#define LOG_DEBUG(fmt, ...)  log_print(LOG_LEVEL_DEBUG, fmt, ##__VA_ARGS__)
```

### 에러 로그 포맷
```c
// 파일명, 라인, 함수명 포함
LOG_ERROR("[%s:%d] %s() - Error: %d (%s)", 
          __FILE__, __LINE__, __func__, 
          err, get_error_string(err));
```

## 에러 복구 전략

### 재시도
```c
#define MAX_RETRY  3

int send_with_retry(uint8_t* data, size_t len)
{
    int retry;
    int result;
    
    for (retry = 0; retry < MAX_RETRY; retry++) {
        result = send_data(data, len);
        if (result == ERR_OK) {
            return ERR_OK;
        }
        LOG_WARN("Send failed, retry %d/%d", retry + 1, MAX_RETRY);
        delay_ms(100);
    }
    
    LOG_ERROR("Send failed after %d retries", MAX_RETRY);
    return result;
}
```

### 안전 상태 전환
```c
void handle_critical_error(int err)
{
    LOG_ERROR("Critical error: %d", err);
    
    // 안전 상태로 전환
    stop_charging();
    open_relay();
    set_state(STATE_ERROR);
    
    // 에러 표시
    set_error_led(true);
}
```

## 체크리스트

- [ ] 모든 함수 반환값 체크
- [ ] 에러 코드 정의 및 문서화
- [ ] 적절한 에러 로깅
- [ ] 리소스 정리 (메모리, 핸들)
- [ ] 에러 복구 또는 안전 상태 전환
