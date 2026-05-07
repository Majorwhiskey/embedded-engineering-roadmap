# C Fundamentals

## Pointers

A pointer stores the memory address of another variable.

```c
int x = 10;
int *p = &x;      // p holds the address of x
printf("%d", *p); // dereference: prints 10
*p = 20;          // changes x to 20 through the pointer
```

**Pointer arithmetic** — moves by the size of the pointed-to type:
```c
int arr[3] = {10, 20, 30};
int *p = arr;     // points to arr[0]
p++;              // now points to arr[1] (advances by sizeof(int) = 4 bytes)
printf("%d", *p); // 20
```

**Pointers to pointers:**
```c
int x = 5;
int *p = &x;
int **pp = &p;    // pointer to pointer
printf("%d", **pp); // 5
```

**NULL pointer** — always initialize pointers you're not using yet:
```c
int *p = NULL;
if (p != NULL) { *p = 5; } // safe check before dereferencing
```

**Common mistakes:**
- Dereferencing NULL → segfault
- Dereferencing uninitialized pointer → undefined behavior
- Returning pointer to a local variable → dangling pointer (stack frame gone)

---

## Arrays

Arrays are contiguous blocks of memory. The array name decays to a pointer to its first element.

```c
int arr[5] = {1, 2, 3, 4, 5};
printf("%p", arr);      // address of arr[0]
printf("%p", &arr[0]);  // same address
```

**2D arrays** — stored in row-major order:
```c
int matrix[2][3] = {{1,2,3},{4,5,6}};
// memory: 1 2 3 4 5 6 (contiguous)
```

**Array vs pointer:**
```c
int arr[5];
int *p = arr;
sizeof(arr); // 20 (5 × 4 bytes) — actual array size
sizeof(p);   // 8  (pointer size on 64-bit) — just an address
```

---

## Structs

Group related data of different types under one name.

```c
typedef struct {
    char name[32];
    uint8_t age;
    float temperature;
} Sensor;

Sensor s = {"BMP280", 0, 25.3f};
printf("%s: %.1f°C\n", s.name, s.temperature);
```

**Struct padding** — compiler adds padding for alignment:
```c
struct Padded {
    char  a;   // 1 byte
               // 3 bytes padding
    int   b;   // 4 bytes
    char  c;   // 1 byte
               // 3 bytes padding
};             // total: 12 bytes (not 6)

struct Packed {
    char  a;
    int   b;
    char  c;
} __attribute__((packed)); // total: 6 bytes — disables padding
                            // use with caution: may cause unaligned access on MCUs
```

**Pointers to structs** — use `->` to access members:
```c
Sensor *sp = &s;
printf("%s", sp->name);     // arrow operator
printf("%s", (*sp).name);   // equivalent, but ugly
```

---

## Unions

All members share the same memory. Size = size of largest member. Only one member holds valid data at a time.

```c
typedef union {
    uint32_t raw;
    struct {
        uint8_t byte0;
        uint8_t byte1;
        uint8_t byte2;
        uint8_t byte3;
    } bytes;
} Register;

Register reg;
reg.raw = 0xDEADBEEF;
printf("%02X", reg.bytes.byte0); // EF (little-endian)
```

Useful in embedded for type-punning: interpret the same bytes as different types (e.g., float ↔ uint32_t for serialization).

---

## Enums

Named integer constants. Makes state machines and flags readable.

```c
typedef enum {
    STATE_IDLE = 0,
    STATE_RUNNING,
    STATE_ERROR,
    STATE_COUNT     // useful pattern: always know how many states
} SystemState;

SystemState state = STATE_IDLE;

switch (state) {
    case STATE_IDLE:    /* ... */ break;
    case STATE_RUNNING: /* ... */ break;
    case STATE_ERROR:   /* ... */ break;
    default: break;
}
```

---

## Typedefs

Create type aliases. In embedded, always use fixed-width types from `<stdint.h>`:

```c
#include <stdint.h>
#include <stdbool.h>

uint8_t   a;  // exactly 8-bit unsigned  (0–255)
int8_t    b;  // exactly 8-bit signed    (-128–127)
uint16_t  c;  // exactly 16-bit unsigned
int32_t   d;  // exactly 32-bit signed
uint64_t  e;  // exactly 64-bit unsigned
bool      f;  // true / false
```

Never use `int` or `long` for hardware registers — their sizes are platform-dependent. Always use `uint32_t`, `uint8_t`, etc.

---

## Key Rules to Remember

| Concept | Rule |
|---------|------|
| Pointer init | Always initialize to NULL or a valid address |
| Struct in embedded | Use `__attribute__((packed))` only when needed for protocol buffers |
| Integer types | Always use `<stdint.h>` fixed-width types for hardware |
| Array bounds | C does NOT check bounds — you will silently corrupt memory |
| `sizeof` on array | Only works if you have the actual array, not a pointer to it |
