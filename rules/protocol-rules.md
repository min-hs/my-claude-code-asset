# 프로토콜 규칙

EV 충전 프로토콜 구현 규칙입니다.

## 지원 프로토콜

| 프로토콜 | 용도 | 표준 |
|----------|------|------|
| ISO 15118 | V2G 통신 | ISO 15118-2 |
| DIN 70121 | DC 충전 | DIN SPEC 70121 |
| OCPP | 백엔드 연동 | OCPP 1.6/2.0 |
| Modbus | 장비 제어 | Modbus RTU/TCP |

## 메시지 처리 규칙

### 파싱 안전성
```c
// 항상 길이 검증 먼저
int parse_message(uint8_t* buf, size_t len, Message* msg)
{
    // 1. NULL 체크
    if (buf == NULL || msg == NULL) {
        return ERR_NULL_PTR;
    }
    
    // 2. 최소 길이 체크
    if (len < MIN_MSG_LEN) {
        return ERR_INVALID_LENGTH;
    }
    
    // 3. 헤더 파싱
    msg->type = buf[0];
    msg->length = (buf[1] << 8) | buf[2];
    
    // 4. 페이로드 길이 검증
    if (msg->length > len - HEADER_SIZE) {
        return ERR_INVALID_LENGTH;
    }
    
    // 5. 페이로드 파싱
    return parse_payload(buf + HEADER_SIZE, msg);
}
```

### 상태 머신
```c
typedef enum {
    V2G_STATE_IDLE,
    V2G_STATE_SESSION_SETUP,
    V2G_STATE_SERVICE_DISCOVERY,
    V2G_STATE_CHARGE_PARAMETER,
    V2G_STATE_CHARGING,
    V2G_STATE_SESSION_STOP,
    V2G_STATE_ERROR
} V2GState;

// 상태 전이 테이블
typedef struct {
    V2GState current;
    V2GEvent event;
    V2GState next;
    StateHandler handler;
} StateTransition;

static const StateTransition transitions[] = {
    { V2G_STATE_IDLE,           EVT_SESSION_REQ,    V2G_STATE_SESSION_SETUP,    handle_session_req },
    { V2G_STATE_SESSION_SETUP,  EVT_DISCOVERY_REQ,  V2G_STATE_SERVICE_DISCOVERY, handle_discovery },
    // ...
};
```

## 타이밍 규칙

### ISO 15118 타이밍
```c
// 시퀀스 타이밍 (ms)
#define V2G_TIMING_SEQ              40      // 시퀀스 퍼포먼스 시간
#define V2G_TIMING_MSG_RESPONSE     2000    // 메시지 응답 대기
#define V2G_TIMING_SESSION          60000   // 세션 타임아웃

// 타이머 시작
void start_response_timer(void)
{
    timer_start(TIMER_MSG_RESPONSE, V2G_TIMING_MSG_RESPONSE, on_response_timeout);
}

// 타임아웃 처리
void on_response_timeout(void)
{
    LOG_WARN("Response timeout");
    retry_or_fail();
}
```

### 타임아웃 처리
```c
typedef struct {
    uint32_t start_tick;
    uint32_t timeout_ms;
    bool active;
} Timer;

bool is_timeout(Timer* t)
{
    if (!t->active) return false;
    return (get_tick() - t->start_tick) >= t->timeout_ms;
}
```

## 메시지 무결성

### CRC 검증
```c
uint16_t calc_crc16(uint8_t* data, size_t len)
{
    uint16_t crc = 0xFFFF;
    for (size_t i = 0; i < len; i++) {
        crc ^= data[i];
        for (int j = 0; j < 8; j++) {
            if (crc & 1) {
                crc = (crc >> 1) ^ 0xA001;
            } else {
                crc >>= 1;
            }
        }
    }
    return crc;
}

int verify_message(uint8_t* buf, size_t len)
{
    uint16_t received_crc = (buf[len-2] << 8) | buf[len-1];
    uint16_t calc_crc = calc_crc16(buf, len - 2);
    
    if (received_crc != calc_crc) {
        LOG_ERROR("CRC mismatch: recv=0x%04X, calc=0x%04X", 
                  received_crc, calc_crc);
        return ERR_CRC_MISMATCH;
    }
    return ERR_OK;
}
```

## OCPP 규칙

### 메시지 구조
```c
// OCPP Call: [MessageTypeId, UniqueId, Action, Payload]
// OCPP CallResult: [MessageTypeId, UniqueId, Payload]
// OCPP CallError: [MessageTypeId, UniqueId, ErrorCode, ErrorDesc, ErrorDetails]

typedef enum {
    OCPP_MSG_CALL        = 2,
    OCPP_MSG_CALLRESULT  = 3,
    OCPP_MSG_CALLERROR   = 4
} OcppMessageType;
```

### 트랜잭션 관리
```c
// 고유 ID 생성
void generate_unique_id(char* id, size_t size)
{
    snprintf(id, size, "%08X", get_unique_counter());
}

// 대기 중인 요청 관리
typedef struct {
    char unique_id[37];
    OcppAction action;
    uint32_t timestamp;
    bool pending;
} PendingRequest;
```

## 체크리스트

- [ ] 모든 수신 메시지 길이 검증
- [ ] CRC/체크섬 검증
- [ ] 상태 머신 전이 검증
- [ ] 타임아웃 처리
- [ ] 시퀀스 번호 관리
- [ ] 재전송 로직 구현
