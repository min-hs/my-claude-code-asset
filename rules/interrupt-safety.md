# 인터럽트 안전성 규칙

임베디드 환경에서 인터럽트 처리 규칙입니다.

## 기본 원칙

1. **ISR은 최대한 짧게**
2. **공유 자원 보호**
3. **블로킹 함수 금지**

## ISR 작성 규칙

### 짧고 빠르게
```c
// 좋은 예: 플래그만 설정
volatile bool rx_flag = false;

void UART_IRQHandler(void)
{
    rx_buffer[rx_index++] = UART->DR;
    rx_flag = true;  // 플래그 설정만
}

// 메인 루프에서 처리
void main_loop(void)
{
    if (rx_flag) {
        rx_flag = false;
        process_rx_data();  // 실제 처리
    }
}
```

### ISR에서 금지 사항
```c
void BAD_IRQHandler(void)
{
    printf("debug");        // ❌ 금지: 블로킹 함수
    malloc(size);           // ❌ 금지: 동적 할당
    process_message();      // ❌ 금지: 긴 처리
    wait_for_flag();        // ❌ 금지: 대기
}
```

## 공유 자원 보호

### volatile 키워드
```c
// ISR과 메인 간 공유 변수
volatile uint32_t tick_count = 0;
volatile bool data_ready = false;
```

### Critical Section
```c
// 인터럽트 비활성화로 보호
void safe_update(void)
{
    uint32_t primask = __get_PRIMASK();
    __disable_irq();
    
    // 공유 자원 접근
    shared_counter++;
    
    __set_PRIMASK(primask);  // 이전 상태 복원
}
```

### 원자적 접근
```c
// 32비트 단일 접근은 원자적 (ARM)
volatile uint32_t flag;  // OK

// 64비트나 구조체는 보호 필요
typedef struct {
    uint32_t a;
    uint32_t b;
} Data;

volatile Data shared_data;  // Critical section 필요
```

## 링 버퍼 패턴
```c
#define BUF_SIZE 256  // 2의 거듭제곱

typedef struct {
    volatile uint8_t buffer[BUF_SIZE];
    volatile uint32_t head;  // ISR이 씀
    volatile uint32_t tail;  // 메인이 읽음
} RingBuffer;

// ISR에서 쓰기
void ISR_write(RingBuffer* rb, uint8_t data)
{
    uint32_t next = (rb->head + 1) & (BUF_SIZE - 1);
    if (next != rb->tail) {
        rb->buffer[rb->head] = data;
        rb->head = next;
    }
}

// 메인에서 읽기
bool main_read(RingBuffer* rb, uint8_t* data)
{
    if (rb->head == rb->tail) {
        return false;  // 비어있음
    }
    *data = rb->buffer[rb->tail];
    rb->tail = (rb->tail + 1) & (BUF_SIZE - 1);
    return true;
}
```

## 인터럽트 우선순위
```c
// 우선순위 설정 (숫자 낮을수록 높은 우선순위)
NVIC_SetPriority(UART_IRQn, 2);   // 높음
NVIC_SetPriority(TIMER_IRQn, 3);  // 중간
NVIC_SetPriority(ADC_IRQn, 4);    // 낮음

// 우선순위 역전 주의
// 높은 우선순위 ISR에서 낮은 우선순위 자원 접근 시 주의
```

## 체크리스트

- [ ] ISR 실행 시간 최소화
- [ ] ISR에서 블로킹 함수 미사용
- [ ] 공유 변수에 volatile
- [ ] 복합 자원 접근 시 critical section
- [ ] 인터럽트 우선순위 적절히 설정
- [ ] 링 버퍼로 데이터 전달
