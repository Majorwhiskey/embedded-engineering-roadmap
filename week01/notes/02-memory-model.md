# Day 2 — Memory Model
**Date: May 8, 2026**

Topics: stack, heap, BSS, data, text segments · memory-mapped I/O · `volatile` · GDB memory inspection

---

## The Five Memory Regions

Every C program's memory is split into distinct regions. Understanding where each variable lives is critical for embedded work where RAM is measured in kilobytes.

```
High address  ┌─────────────────────────┐
              │         Stack           │ ← grows downward on function call
              │           ↓             │
              │      (free space)       │
              │           ↑             │
              │          Heap           │ ← grows upward with malloc()
              ├─────────────────────────┤
              │       BSS Segment       │ ← uninit globals/statics, zeroed by startup
              ├─────────────────────────┤
              │      Data Segment       │ ← init globals/statics, copied from flash
              ├─────────────────────────┤
              │      Text Segment       │ ← compiled machine code, read-only
Low address   └─────────────────────────┘
```

On a **bare-metal MCU** (STM32F4 example):
```
Flash  0x0800 0000  Text + Data init values (read-only)
SRAM   0x2000 0000  BSS + Data (runtime) + Heap + Stack
```

At startup, the reset handler (before `main()`):
1. Copies the Data segment from flash to SRAM
2. Zeroes the BSS segment
3. Sets the stack pointer to the top of SRAM
4. Calls `main()`

---

## Stack

**What lives here:** local variables, function parameters, return addresses, saved registers.

**How it works:** the stack pointer (SP / R13 on ARM) moves down on function entry and back up on return. It's just a pointer increment — extremely fast.

```c
void foo(int x) {
    int a = 10;         // on stack
    char buf[64];       // 64 bytes on stack
    // stack frame: return addr | saved regs | x | a | buf[64]
}   // SP restored to caller's value — a, buf, x are GONE
```

**Stack frame layout (ARM Cortex-M):**
```
Before call:  SP → [caller's locals]
After call:   SP → [buf[64]] [a] [x] [saved LR] [saved regs]
                                                ↑ SP moved here
```

### Stack Overflow

Fixed at link time (linker script). On STM32, typically 1–4 KB. Exceeding it silently overwrites the heap or global variables.

```c
void recursive_bomb(int n) {
    char huge[4096];         // 4 KB per call
    recursive_bomb(n + 1);   // no base case → stack overflow → hardfault
}
```

**Detecting it:** use `uxTaskGetStackHighWaterMark()` in FreeRTOS, or initialize stack with a canary value (0xDEADBEEF) and check if overwritten.

### Why You Can't Return a Pointer to a Local Variable

```c
int* broken(void) {
    int local = 42;
    return &local;   // WRONG: local is gone when function returns
}                    // returned pointer is dangling — undefined behavior

int* correct(void) {
    static int value = 42;  // static: lives in BSS/Data, not stack
    return &value;           // safe
}
```

---

## Heap

**What lives here:** memory allocated with `malloc()`, `calloc()`, `realloc()`. You control the lifetime.

```c
#include <stdlib.h>

// Allocate
int *arr = malloc(10 * sizeof(int));
if (arr == NULL) {
    // malloc can fail — always check!
    handle_error();
}

// Use
for (int i = 0; i < 10; i++) arr[i] = i * 2;

// Free — must match every malloc
free(arr);
arr = NULL;   // prevents accidental use-after-free
```

### Heap Problems

| Problem | What happens |
|---------|-------------|
| Memory leak | `malloc` without `free` → heap grows → eventually runs out |
| Double free | `free(p)` twice → heap corruption → crash |
| Use after free | Reading `*p` after `free(p)` → garbage data or crash |
| Heap fragmentation | Many small alloc/free cycles → unusable gaps |

**On bare-metal MCUs: prefer static allocation.** Dynamic allocation risks:
- Non-deterministic timing (allocator scans free list)
- Fragmentation leading to `malloc` returning NULL after hours of uptime
- No OS to clean up leaks

```c
// Embedded-safe alternative: static memory pool
#define MAX_NODES 32
static Node_t node_pool[MAX_NODES];
static uint8_t node_used[MAX_NODES] = {0};

Node_t *alloc_node(void) {
    for (int i = 0; i < MAX_NODES; i++) {
        if (!node_used[i]) {
            node_used[i] = 1;
            return &node_pool[i];
        }
    }
    return NULL;  // pool exhausted
}
```

---

## BSS Segment

**What lives here:** uninitialized global variables and static variables. The C runtime zeroes this region before `main()` runs.

```c
int global_counter;          // BSS — starts at 0
static int error_count;      // BSS — starts at 0

void isr_handler(void) {
    static int call_count;   // BSS — persists between calls, starts at 0
    call_count++;
}
```

**On MCUs:** BSS lives in SRAM and is zeroed by the startup code (crt0 / reset handler). In your linker script you'll see symbols like `_sbss` and `_ebss` marking its bounds.

---

## Data Segment

**What lives here:** initialized global and static variables with non-zero values.

```c
int max_retries = 5;               // Data segment
static float pi = 3.14159f;        // Data segment
const char fw_version[] = "1.0.0"; // Likely in flash (read-only data)
```

**MCU detail:** The initial values are stored in flash. The startup code copies them to SRAM before `main()`. The flash area holding the init values is between symbols `_sidata` and `_edata`; the destination is `_sdata` to `_edata`.

---

## Text Segment

Contains the compiled machine code (instructions). Read-only. Lives in flash on MCUs.

On desktop: attempting to write to the text segment causes a segfault. On MCUs: writing to flash requires special erase/program sequences — you cannot modify code at runtime without a bootloader.

---

## `volatile` — The Most Important Keyword for Embedded

Without `volatile`, the compiler assumes memory only changes because your C code changes it. It will cache values in registers and eliminate "redundant" reads/writes — silently breaking hardware interaction.

```c
// WITHOUT volatile — BROKEN
uint32_t *reg = (uint32_t *)0x40020014;
*reg = 1;           // compiler may eliminate this (value never "read back")
while (*reg == 0);  // compiler may optimize to infinite loop or no loop

// WITH volatile — CORRECT
volatile uint32_t *reg = (volatile uint32_t *)0x40020014;
*reg = 1;           // write always happens
while (*reg == 0);  // read from hardware every iteration
```

**Three cases where `volatile` is required:**
1. Memory-mapped peripheral registers
2. Variables modified by an interrupt service routine (ISR)
3. Variables shared between threads/tasks (though `volatile` alone isn't sufficient for thread safety — use mutexes too)

```c
// ISR example
volatile uint8_t data_ready = 0;   // modified by ISR

void USART1_IRQHandler(void) {
    received_byte = USART1->DR;
    data_ready = 1;                // ISR sets flag
}

int main(void) {
    while (!data_ready);           // main polls flag — must be volatile
    process(received_byte);
}
```

---

## Memory-Mapped I/O

On microcontrollers, **peripheral registers appear in the normal address space**. Reading or writing a specific address directly controls hardware.

```c
// STM32F4 — enable GPIOA clock and toggle PA5 (LD2 on Nucleo)

// Step 1: Enable clock to GPIOA (bit 0 of RCC_AHB1ENR)
#define RCC_AHB1ENR   (*(volatile uint32_t *)0x40023830)
RCC_AHB1ENR |= (1 << 0);

// Step 2: Set PA5 to output mode (bits [11:10] of GPIOA_MODER = 0b01)
#define GPIOA_MODER   (*(volatile uint32_t *)0x40020000)
GPIOA_MODER &= ~(0b11 << 10);
GPIOA_MODER |=  (0b01 << 10);

// Step 3: Toggle PA5 by writing to GPIOA_ODR
#define GPIOA_ODR     (*(volatile uint32_t *)0x40020014)
GPIOA_ODR ^= (1 << 5);
```

In practice you use the CMSIS struct headers which give you nice names:
```c
RCC->AHB1ENR   |= RCC_AHB1ENR_GPIOAEN;
GPIOA->MODER   &= ~GPIO_MODER_MODE5;
GPIOA->MODER   |=  GPIO_MODER_MODE5_0;
GPIOA->ODR     ^=  GPIO_ODR_OD5;
```

Both compile to identical machine code. The CMSIS macros expand to the same volatile pointer casts.

---

## GDB Memory Inspection

```bash
gcc -g -o program main.c    # compile with debug symbols
gdb ./program

# Run to a breakpoint
(gdb) break main
(gdb) run

# Print variable value and address
(gdb) print x               # value
(gdb) print &x              # address
(gdb) print *ptr            # dereference pointer

# Examine raw memory: x/<count><format><size> <address>
# format: x=hex, d=decimal, u=unsigned, c=char, s=string, i=instruction
# size:   b=byte(1), h=halfword(2), w=word(4), g=giant(8)

(gdb) x/4xw &arr            # 4 words in hex starting at arr
(gdb) x/8xb &my_struct      # 8 bytes of a struct in hex
(gdb) x/1xw 0x40020014      # read a hardware register address
(gdb) x/20i main            # disassemble 20 instructions from main

# Show all CPU registers
(gdb) info registers

# Show call stack
(gdb) backtrace

# Show where on the stack we are
(gdb) info frame
```

---

## STM32F4 Memory Map Reference

| Region | Start | End | What's There |
|--------|-------|-----|-------------|
| Flash | 0x0800 0000 | 0x080F FFFF | Code, const data, init values |
| SRAM1 | 0x2000 0000 | 0x2001 BFFF | Stack, heap, globals (112 KB) |
| SRAM2 | 0x2001 C000 | 0x2001 FFFF | 16 KB |
| APB1  | 0x4000 0000 | 0x4000 77FF | TIM2–7, UART2–5, SPI2/3, I2C |
| APB2  | 0x4001 0000 | 0x4001 57FF | TIM1, USART1, SPI1, ADC |
| AHB1  | 0x4002 0000 | 0x4007 FFFF | GPIO A–I, DMA1/2, RCC |
| NVIC  | 0xE000 E000 | 0xE000 EFFF | Interrupt controller |

---

## Today's Mini-Exercises

1. Declare one variable in each segment (stack, heap, BSS, Data). Print each address — observe how they cluster in different regions.
2. Write a function with `static` local variable. Call it 5 times and print the variable each time — confirm it persists.
3. Intentionally leak memory in a loop (don't free). Watch memory usage grow with `valgrind --tool=massif`.
4. Write a `volatile` flag updated in a simulated ISR (just a function). Show that removing `volatile` causes GCC `-O2` to optimize away the flag check (use `objdump -d`).

---

## YouTube Resources

| Topic | Video | Channel |
|-------|-------|---------|
| Stack vs Heap | [Stack vs Heap Memory in C](https://www.youtube.com/watch?v=_8-ht2AKyH4) | Alex Hyett |
| Memory segments deep dive | [C Memory Model](https://www.youtube.com/results?search_query=c+memory+segments+stack+heap+bss+text) | Low Level Learning |
| volatile keyword | [The volatile keyword in C](https://www.youtube.com/results?search_query=volatile+keyword+c+embedded+systems) | Jacob Sorber |
| GDB tutorial | [GDB Debugger Tutorial](https://www.youtube.com/watch?v=bWH-nL7v5F4) | CS50 / freeCodeCamp |
| Memory-mapped I/O | [Bare Metal STM32 Programming](https://www.youtube.com/results?search_query=bare+metal+stm32+memory+mapped+io+registers) | Fastbit Embedded |
