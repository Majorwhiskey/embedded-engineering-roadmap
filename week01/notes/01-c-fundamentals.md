# Day 1 — C Fundamentals
**Date: May 7, 2026**

Topics: pointers, arrays, structs, unions, enums, typedefs, fixed-width types

---

## Pointers

A pointer is a variable that stores a memory address. It is the most important concept in C for embedded work — every peripheral register, every buffer, every data structure relies on pointers.

```c
int x = 10;
int *p = &x;       // & = "address of" operator
                   // p now holds the address of x (e.g. 0x7ffd1234)

printf("%d\n", x);  // 10  — value of x
printf("%p\n", &x); // 0x7ffd1234 — address of x
printf("%p\n", p);  // 0x7ffd1234 — what p stores
printf("%d\n", *p); // 10  — * = dereference: value at the address

*p = 42;            // write through pointer — changes x
printf("%d\n", x);  // 42
```

**Pointer size** depends on the platform:
- 64-bit Linux: `sizeof(int *) = 8`
- 32-bit ARM MCU: `sizeof(int *) = 4`

All pointer types are the same size — what differs is what type they *point to*.

### Pointer Arithmetic

When you do math on a pointer, it scales by the size of the pointed-to type:

```c
int arr[4] = {10, 20, 30, 40};
int *p = arr;          // points to arr[0]

printf("%d\n", *p);    // 10
p++;                   // advances by sizeof(int) = 4 bytes
printf("%d\n", *p);    // 20
printf("%d\n", *(p+1)); // 30  — p+1 points to arr[2], p unchanged

// arr[i] and *(arr+i) are EXACTLY equivalent
printf("%d\n", arr[2]);       // 30
printf("%d\n", *(arr + 2));   // 30
```

### Pointer to Pointer

```c
int x = 5;
int *p  = &x;    // pointer to int
int **pp = &p;   // pointer to pointer to int

printf("%d\n", **pp);    // 5  — dereference twice
**pp = 99;               // changes x through two levels of indirection
printf("%d\n", x);       // 99
```

Used in: linked lists, dynamic 2D arrays, `argv` in `main(int argc, char **argv)`.

### const and pointers — 4 combinations

```c
int x = 10, y = 20;

int *p1 = &x;                  // mutable pointer, mutable data
const int *p2 = &x;            // mutable pointer, CONST data (can't *p2 = ...)
int * const p3 = &x;           // CONST pointer, mutable data (can't p3 = &y)
const int * const p4 = &x;     // CONST pointer, CONST data
```

In embedded: peripheral registers are typically `volatile uint32_t * const` — the address is fixed (const pointer), but the value changes unpredictably (volatile).

### Function Pointers

```c
int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }

// Declare a function pointer: return_type (*name)(param_types)
int (*op)(int, int);

op = add;
printf("%d\n", op(3, 4));  // 7

op = sub;
printf("%d\n", op(3, 4));  // -1
```

Used extensively in embedded: jump tables, callback registration, RTOS task functions.

### Common Pointer Bugs

```c
// Bug 1: Uninitialized pointer (undefined behavior)
int *p;
*p = 5;   // writing to unknown address → crash or silent corruption

// Bug 2: Dangling pointer (use after free / after scope)
int *dangling;
{
    int local = 10;
    dangling = &local;
}  // local is gone — dangling now points to invalid stack memory
printf("%d\n", *dangling);  // undefined behavior

// Bug 3: NULL dereference
int *p = NULL;
*p = 5;  // segfault

// Fix: always initialize
int *p = NULL;
if (p != NULL) { *p = 5; }
```

---

## Arrays

Arrays are contiguous blocks of memory. The name of an array decays to a pointer to its first element in most contexts.

```c
int arr[5] = {10, 20, 30, 40, 50};

// These are all equivalent:
printf("%d\n", arr[2]);       // 30 — index notation
printf("%d\n", *(arr + 2));   // 30 — pointer arithmetic
printf("%d\n", 2[arr]);       // 30 — legal in C (commutative) — never write this

// sizeof works on the actual array (not a pointer to it)
printf("%zu\n", sizeof(arr));        // 20 (5 × 4 bytes)
printf("%zu\n", sizeof(arr[0]));     // 4 (one int)
int len = sizeof(arr) / sizeof(arr[0]);  // 5 — length of array
```

**C does not check bounds.** Writing past the end of an array silently corrupts adjacent memory:
```c
int arr[3] = {1, 2, 3};
arr[5] = 99;   // compiles fine — corrupts memory → undefined behavior
```

### Passing Arrays to Functions

Arrays decay to pointers when passed to a function. The function loses the size information:

```c
void print_array(int *arr, int len) {  // must pass length separately
    for (int i = 0; i < len; i++) {
        printf("%d ", arr[i]);
    }
}

int data[5] = {1, 2, 3, 4, 5};
print_array(data, 5);
```

### 2D Arrays

Stored in row-major order — rows are contiguous in memory:

```c
int matrix[3][4] = {
    {1,  2,  3,  4},
    {5,  6,  7,  8},
    {9, 10, 11, 12}
};

// Memory layout: 1 2 3 4 5 6 7 8 9 10 11 12
printf("%d\n", matrix[1][2]);          // 7
printf("%d\n", *(*(matrix+1)+2));      // 7  (same thing, pointer form)
```

---

## Structs

A struct groups related variables of different types under one name. Essential for representing hardware registers, sensor data, protocol packets.

```c
#include <stdint.h>

typedef struct {
    char     name[32];
    uint8_t  address;     // I2C address
    float    temperature;
    float    pressure;
    uint8_t  is_ready;
} Sensor_t;

Sensor_t bmp280 = {"BMP280", 0x76, 0.0f, 0.0f, 0};
```

### Struct Padding and Alignment

The compiler inserts padding so each field is aligned to its natural alignment (usually its own size):

```c
struct Unoptimized {
    char   a;    // 1 byte at offset 0
                 // 3 bytes padding
    int    b;    // 4 bytes at offset 4
    char   c;    // 1 byte at offset 8
                 // 3 bytes padding
};               // total: 12 bytes

struct Optimized {
    int    b;    // 4 bytes at offset 0
    char   a;    // 1 byte at offset 4
    char   c;    // 1 byte at offset 5
                 // 2 bytes padding
};               // total: 8 bytes — same fields, better order

// Rule: place largest fields first to minimize padding
```

For protocol buffers or hardware register maps that must match an exact byte layout:
```c
struct __attribute__((packed)) UARTFrame {
    uint8_t  start;
    uint16_t length;
    uint8_t  data[64];
    uint16_t crc;
};
// No padding — but may cause unaligned access on some MCUs
```

### Accessing via Pointer

```c
Sensor_t *sp = &bmp280;

sp->temperature = 24.5f;   // arrow operator = (*sp).temperature
(*sp).pressure  = 1013.0f; // equivalent — but use -> instead

// Passing to a function (by pointer, not by value — avoids copying)
void update_sensor(Sensor_t *s) {
    s->is_ready = 1;
}
update_sensor(&bmp280);
```

### Nested Structs

```c
typedef struct {
    float x, y, z;
} Vec3_t;

typedef struct {
    Vec3_t  accel;
    Vec3_t  gyro;
    uint32_t timestamp_ms;
} IMUData_t;

IMUData_t imu;
imu.accel.x = 0.12f;
imu.gyro.z  = -0.04f;
```

---

## Unions

All members share the same memory location. Size = size of the largest member. Only one member is valid at a time.

```c
typedef union {
    uint32_t raw;          // access all 32 bits at once
    struct {
        uint8_t byte0;     // bits  7:0
        uint8_t byte1;     // bits 15:8
        uint8_t byte2;     // bits 23:16
        uint8_t byte3;     // bits 31:24
    };
} Word_t;

Word_t w;
w.raw = 0xDEADBEEF;
printf("%02X\n", w.byte0);  // EF  (little-endian: LSB first)
printf("%02X\n", w.byte3);  // DE
```

**Real embedded use case** — packing a float into bytes for UART transmission:
```c
typedef union {
    float    f;
    uint8_t  bytes[4];
} FloatBytes_t;

FloatBytes_t fb;
fb.f = 3.14159f;
// Now fb.bytes[0..3] holds the raw IEEE-754 bytes — send over UART
uart_send(fb.bytes, 4);
```

---

## Enums

Named integer constants. Vastly improve readability of state machines and flags.

```c
typedef enum {
    GPIO_MODE_INPUT     = 0b00,
    GPIO_MODE_OUTPUT    = 0b01,
    GPIO_MODE_ALTFUNC   = 0b10,
    GPIO_MODE_ANALOG    = 0b11,
} GPIO_Mode_t;

typedef enum {
    SENSOR_OK      = 0,
    SENSOR_TIMEOUT = 1,
    SENSOR_CRC_ERR = 2,
    SENSOR_NOT_RDY = 3,
} SensorStatus_t;

SensorStatus_t read_sensor(Sensor_t *s) {
    if (!s->is_ready) return SENSOR_NOT_RDY;
    // ... read ...
    return SENSOR_OK;
}

SensorStatus_t result = read_sensor(&bmp280);
switch (result) {
    case SENSOR_OK:      printf("OK\n");      break;
    case SENSOR_TIMEOUT: printf("Timeout\n"); break;
    default:             printf("Error\n");   break;
}
```

---

## Fixed-Width Types — `<stdint.h>`

Never use `int`, `long`, or `short` for hardware-level code. Their sizes are platform-dependent.

```c
#include <stdint.h>
#include <stdbool.h>

uint8_t   a;   // 0 to 255
int8_t    b;   // -128 to 127
uint16_t  c;   // 0 to 65535
int16_t   d;   // -32768 to 32767
uint32_t  e;   // 0 to 4,294,967,295
int32_t   f;   // -2,147,483,648 to 2,147,483,647
uint64_t  g;   // 0 to 18,446,744,073,709,551,615

bool      h;   // true (1) or false (0)
```

**Why it matters:** on a 16-bit AVR MCU, `int` is 16 bits. On a 32-bit ARM, `int` is 32 bits. Code that assumes `int` is 32 bits will silently break. `uint32_t` is always 32 bits, everywhere.

---

## Today's Mini-Exercises

1. Write a `swap(int *a, int *b)` function that swaps two integers using pointers.
2. Write a function that reverses an array in-place using pointer arithmetic (no index `[]`).
3. Create a `Packet_t` struct with fields: `uint8_t id`, `uint16_t length`, `uint8_t data[32]`, `uint16_t crc`. Print its `sizeof` and compare to the sum of its fields' sizes — spot the padding.
4. Write a union that lets you read a `uint16_t` as two separate `uint8_t` bytes.

---

## YouTube Resources

| Topic | Video | Channel |
|-------|-------|---------|
| Pointers deep dive | [Pointers in C / C++](https://www.youtube.com/watch?v=zuegQmMdy8M) | freeCodeCamp |
| C structs & memory | [CS50 Week 1 – C](https://www.youtube.com/watch?v=4IrgnBVMtGQ) | Harvard CS50 |
| Pointer arithmetic | [C Programming – Pointers](https://www.youtube.com/results?search_query=neso+academy+pointers+in+c) | Neso Academy |
| Function pointers | [Function Pointers in C](https://www.youtube.com/results?search_query=function+pointers+in+c+low+level+learning) | Low Level Learning |
| Unions and bitfields | [C Unions Explained](https://www.youtube.com/results?search_query=unions+in+c+embedded+systems) | Jacob Sorber |
