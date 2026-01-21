# 빌드 및 디버깅 규칙

임베디드 프로젝트 빌드 및 디버깅 규칙입니다.

## 빌드 규칙

### Makefile 구조
```makefile
# 프로젝트 설정
TARGET = firmware
BUILD_DIR = build

# 컴파일러 설정
CC = arm-none-eabi-gcc
CFLAGS = -Wall -Wextra -Werror
CFLAGS += -std=c99
CFLAGS += -O2

# 디버그 빌드
debug: CFLAGS += -g -DDEBUG
debug: all

# 릴리즈 빌드
release: CFLAGS += -DNDEBUG
release: all
```

### 컴파일 경고 설정
```makefile
# 필수 경고 옵션
WARNINGS = -Wall              # 기본 경고
WARNINGS += -Wextra           # 추가 경고
WARNINGS += -Werror           # 경고를 에러로
WARNINGS += -Wshadow          # 변수 섀도잉
WARNINGS += -Wformat=2        # printf 포맷
WARNINGS += -Wconversion      # 암묵적 변환
WARNINGS += -Wunused          # 미사용 변수/함수
```

### 빌드 명령어
```bash
# 기본 빌드
make

# 클린 빌드
make clean && make

# 디버그 빌드
make debug

# 릴리즈 빌드
make release

# 정적 분석
make analyze
```

## 디버그 매크로

### 조건부 디버그 출력
```c
#ifdef DEBUG
    #define DBG_PRINT(fmt, ...) \
        printf("[DBG][%s:%d] " fmt "\n", __func__, __LINE__, ##__VA_ARGS__)
    #define DBG_HEX(buf, len)   debug_hex_dump(buf, len)
#else
    #define DBG_PRINT(fmt, ...)
    #define DBG_HEX(buf, len)
#endif
```

### ASSERT 매크로
```c
#ifdef DEBUG
    #define ASSERT(expr) \
        do { \
            if (!(expr)) { \
                printf("ASSERT FAILED: %s, %s:%d\n", \
                       #expr, __FILE__, __LINE__); \
                while(1);  /* 무한 루프로 정지 */ \
            } \
        } while(0)
#else
    #define ASSERT(expr)  ((void)0)
#endif

// 사용 예
ASSERT(ptr != NULL);
ASSERT(len <= MAX_SIZE);
```

### 성능 측정
```c
#ifdef DEBUG
    #define PERF_START()    uint32_t _perf_start = get_tick()
    #define PERF_END(name)  printf("[PERF] %s: %lu ms\n", name, get_tick() - _perf_start)
#else
    #define PERF_START()
    #define PERF_END(name)
#endif

// 사용 예
void process_data(void)
{
    PERF_START();
    // 처리 로직
    PERF_END("process_data");
}
```

## 디버깅 도구

### GDB 명령어
```bash
# GDB 시작
arm-none-eabi-gdb build/firmware.elf

# 자주 쓰는 명령어
(gdb) target remote :3333     # OpenOCD 연결
(gdb) monitor reset halt      # 리셋 후 정지
(gdb) load                    # 펌웨어 로드
(gdb) break main              # 브레이크포인트
(gdb) continue                # 실행 계속
(gdb) next                    # 다음 줄 (step over)
(gdb) step                    # 다음 줄 (step into)
(gdb) print variable          # 변수 출력
(gdb) x/10xb 0x20000000       # 메모리 덤프
(gdb) bt                      # 백트레이스
```

### Valgrind (호스트 테스트용)
```bash
# 메모리 누수 체크
valgrind --leak-check=full ./test_app

# 상세 출력
valgrind --leak-check=full --show-leak-kinds=all ./test_app

# 초기화되지 않은 메모리 사용
valgrind --track-origins=yes ./test_app
```

### 정적 분석
```bash
# cppcheck
cppcheck --enable=all --inconclusive src/
cppcheck --enable=all --xml 2> report.xml src/

# clang-tidy
clang-tidy src/*.c -- -Iinclude
```

## 디버그 출력 포맷

### 로그 포맷
```c
// 타임스탬프 포함
#define LOG(level, fmt, ...) \
    printf("[%lu][%s] " fmt "\n", get_tick(), level, ##__VA_ARGS__)

#define LOG_I(fmt, ...)  LOG("INFO ", fmt, ##__VA_ARGS__)
#define LOG_W(fmt, ...)  LOG("WARN ", fmt, ##__VA_ARGS__)
#define LOG_E(fmt, ...)  LOG("ERROR", fmt, ##__VA_ARGS__)
```

### HEX 덤프
```c
void debug_hex_dump(const uint8_t* buf, size_t len)
{
    printf("HEX DUMP (%zu bytes):\n", len);
    for (size_t i = 0; i < len; i++) {
        printf("%02X ", buf[i]);
        if ((i + 1) % 16 == 0) printf("\n");
    }
    if (len % 16 != 0) printf("\n");
}
```

## 릴리즈 빌드 체크리스트

- [ ] DEBUG 매크로 비활성화
- [ ] printf 디버깅 제거
- [ ] ASSERT 비활성화 확인
- [ ] 최적화 레벨 적절히 설정 (-O2)
- [ ] 불필요한 심볼 제거 (-s)
- [ ] 바이너리 크기 확인
- [ ] 스택 사용량 분석

## 체크리스트

- [ ] 모든 경고 해결 (-Werror)
- [ ] 정적 분석 통과 (cppcheck)
- [ ] 디버그 코드 릴리즈에서 제거
- [ ] 메모리 누수 체크 (Valgrind)
- [ ] 스택 오버플로우 체크
