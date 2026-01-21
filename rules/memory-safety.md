# 메모리 안전성 규칙

임베디드 환경에서의 메모리 관리 규칙입니다.

## 기본 원칙

1. **정적 할당 우선**
2. **동적 할당 시 반드시 해제**
3. **버퍼 크기 항상 검증**

## 정적 할당
```c
// 권장: 정적 할당
static uint8_t rx_buffer[MAX_BUFFER_SIZE];
static Message msg_queue[QUEUE_SIZE];

// 지양: 동적 할당
uint8_t* rx_buffer = malloc(size);  // 필요한 경우만
```

## 동적 할당 규칙

### 할당 후 NULL 체크
```c
uint8_t* buf = malloc(size);
if (buf == NULL) {
    return -1;  // 에러 처리
}
```

### 사용 후 반드시 해제
```c
void process_data(void)
{
    uint8_t* buf = malloc(size);
    if (buf == NULL) return;
    
    // 사용
    
    free(buf);      // 반드시 해제
    buf = NULL;     // 댕글링 포인터 방지
}
```

### 에러 경로에서도 해제
```c
int process(void)
{
    uint8_t* buf1 = malloc(SIZE1);
    if (buf1 == NULL) return -1;
    
    uint8_t* buf2 = malloc(SIZE2);
    if (buf2 == NULL) {
        free(buf1);     // buf1 해제 필수!
        return -1;
    }
    
    // 처리
    
    free(buf2);
    free(buf1);
    return 0;
}
```

## 버퍼 오버플로우 방지

### 크기 검증
```c
// 위험
void copy_data(uint8_t* dst, uint8_t* src, size_t len)
{
    memcpy(dst, src, len);  // dst 크기 체크 없음
}

// 안전
void copy_data(uint8_t* dst, size_t dst_size, uint8_t* src, size_t len)
{
    if (len > dst_size) {
        len = dst_size;  // 또는 에러 반환
    }
    memcpy(dst, src, len);
}
```

### 안전한 문자열 함수
```c
// 금지
strcpy(dst, src);
sprintf(buf, "%s", str);

// 권장
strncpy(dst, src, sizeof(dst) - 1);
dst[sizeof(dst) - 1] = '\0';
snprintf(buf, sizeof(buf), "%s", str);
```

## 포인터 안전성

### NULL 체크
```c
int process_message(Message* msg)
{
    if (msg == NULL) {
        return -1;
    }
    // 처리
}
```

### 댕글링 포인터 방지
```c
free(ptr);
ptr = NULL;  // 해제 후 NULL 설정
```

### 이중 해제 방지
```c
if (ptr != NULL) {
    free(ptr);
    ptr = NULL;
}
```

## 스택 오버플로우 방지
```c
// 위험: 큰 로컬 배열
void bad_function(void)
{
    uint8_t buffer[10000];  // 스택 오버플로우 위험
}

// 안전: static 또는 동적 할당
static uint8_t buffer[10000];  // BSS 영역

void good_function(void)
{
    // buffer 사용
}
```

## 체크리스트

- [ ] 모든 malloc에 NULL 체크
- [ ] 모든 malloc에 대응하는 free
- [ ] 에러 경로에서 메모리 해제
- [ ] 버퍼 복사 전 크기 검증
- [ ] 포인터 해제 후 NULL 설정
- [ ] 큰 배열은 static 또는 heap 사용
