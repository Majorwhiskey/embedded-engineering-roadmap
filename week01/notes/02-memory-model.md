# Memory Model

## The Four Segments

Every C program's memory is divided into four regions:

```
High address
┌─────────────────┐
│      Stack      │  ← grows downward
│        ↓        │
│   (free space)  │
│        ↑        │
│      Heap       │  ← grows upward (malloc/free)
├─────────────────┤
│  BSS Segment    │  ← uninitialized globals & statics (zeroed at startup)
├─────────────────┤
│  Data Segment   │  ← initialized globals & statics
├─────────────────┤
│  Text Segment   │  ← compiled machine code (read-only)
└─────────────────┘
Low address
```

---

## Stack

- Stores: local variables, function parameters, return addresses
- Managed automatically — grows on function call, shrinks on return
- Fast: just a pointer increment/decrement
- **Fixed size** — overflow = crash (stack smashing)

```c
void foo(void) {
    int x = 10;       // on stack
    char buf[64];     // on stack — 64 bytes consumed
}                     // x and buf gone when foo() returns
```

**Stack frame layout (simplified):**
```
foo() stack frame:
┌──────────────┐ ← SP (stack pointer) before call
│ return addr  │
│ saved regs   │
│ local vars   │
│ buf[64]      │
└──────────────┘ ← SP after entering foo()
```

On MCUs, stack size is defined in the linker script — typically 1–8 KB. Exceeding it silently overwrites adjacent memory.

---

## Heap

- Stores: dynamically allocated memory (`malloc`, `calloc`, `realloc`, `free`)
- You manage it — leaks and corruption are your problem
- Slower than stack: allocator must find a free block
- Fragmentation: many small alloc/free cycles leave unusable gaps

```c
int *arr = malloc(10 * sizeof(int));  // allocate 40 bytes
if (arr == NULL) { /* handle failure */ }

arr[0] = 42;
free(arr);      // must free — or it leaks
arr = NULL;     // good habit: avoid dangling pointer
```

**Avoid heap on bare-metal MCUs** — use static allocation instead. Dynamic allocation on MCUs risks fragmentation, non-deterministic timing, and heap exhaustion.

---

## BSS Segment

Holds **uninitialized** (or zero-initialized) global and static variables.  
The C runtime zeroes this region before `main()` runs.

```c
int global_counter;        // BSS — zero at startup
static int call_count;     // BSS — zero at startup

void foo(void) {
    static int times = 0;  // BSS — persists across calls, starts at 0
    times++;
}
```

---

## Data Segment

Holds **initialized** global and static variables. Values are stored in flash and copied to RAM at startup.

```c
int max_retries = 5;              // Data segment
static const char version[] = "1.0"; // may be in flash (read-only data)
```

On MCUs: the startup code (in the vector table / reset handler) copies the data segment from flash to RAM before calling `main()`.

---

## Text Segment

Contains the compiled machine code. Read-only. Lives in flash on MCUs.

---

## Memory-Mapped I/O

On microcontrollers, **peripheral registers are mapped into the memory address space**. Reading or writing to a specific address controls hardware directly.

```c
// STM32 example — toggle an LED by writing to GPIOA ODR register
#define GPIOA_ODR  (*(volatile uint32_t *)0x40020014)

GPIOA_ODR |= (1 << 5);   // set bit 5 → PA5 HIGH
GPIOA_ODR &= ~(1 << 5);  // clear bit 5 → PA5 LOW
```

**`volatile` is mandatory** for memory-mapped registers:
- Tells the compiler: this value can change outside the program's control
- Prevents the compiler from caching the value in a register or optimizing away the read/write
- Without `volatile`: the compiler may eliminate "redundant" reads/writes to hardware registers — causing silent bugs

```c
volatile uint32_t *reg = (volatile uint32_t *)0x40020014;
```

---

## Inspecting Memory with GDB

```bash
# Start GDB
gdb ./program

# Examine memory: x/<count><format><size> <address>
# formats: x=hex, d=decimal, c=char, s=string, i=instruction
# sizes:   b=byte, h=halfword(2), w=word(4), g=giant(8)

(gdb) x/4xw 0x40020014    # show 4 words in hex at address
(gdb) x/8xb &my_struct    # show 8 bytes of a struct
(gdb) info registers       # show all CPU registers
(gdb) p &variable          # print address of a variable
(gdb) p *pointer           # dereference and print
```

---

## MCU Memory Map (STM32F4 example)

| Region | Address Range | Contents |
|--------|--------------|----------|
| Flash  | 0x0800 0000 – 0x080F FFFF | Code + const data |
| SRAM   | 0x2000 0000 – 0x2001 FFFF | Stack + heap + globals |
| APB1   | 0x4000 0000 – 0x4000 77FF | UART, SPI, I2C, TIM2–7 |
| APB2   | 0x4001 0000 – 0x4001 57FF | USART1, SPI1, TIM1 |
| AHB1   | 0x4002 0000 – 0x4007 FFFF | GPIO, DMA, RCC |
| Core   | 0xE000 0000 – 0xE00F FFFF | NVIC, SysTick, DWT |
