# Computer Architecture

## Von Neumann vs Harvard

| | Von Neumann | Harvard |
|---|---|---|
| Memory | Single shared bus for code + data | Separate buses for code and data |
| Speed | Bottleneck: can't fetch instruction and data simultaneously | Faster: parallel access |
| Examples | x86 desktop CPUs | AVR, PIC, most DSPs |
| Used in | General-purpose computers | Microcontrollers, DSPs |

**Modified Harvard** (ARM Cortex-M) — separate instruction and data caches internally, but unified external memory address space. Best of both worlds.

---

## CPU Pipeline

A pipeline overlaps the stages of multiple instructions so the CPU does useful work every clock cycle.

**Classic 5-stage pipeline:**
```
Cycle:    1    2    3    4    5    6    7
Inst 1:  [IF] [ID] [EX] [MEM][WB]
Inst 2:       [IF] [ID] [EX] [MEM][WB]
Inst 3:            [IF] [ID] [EX] [MEM][WB]
```

Stages:
- **IF** — Instruction Fetch: read next instruction from memory
- **ID** — Instruction Decode: decode opcode, read registers
- **EX** — Execute: ALU operation
- **MEM** — Memory Access: load/store
- **WB** — Write Back: write result to register file

**Pipeline hazards:**
- **Data hazard** — instruction needs result from previous instruction not yet written back → stall
- **Control hazard** — branch: don't know which instruction comes next until EX stage → flush pipeline (branch penalty)
- **Structural hazard** — two instructions need the same hardware unit simultaneously

ARM Cortex-M3/M4 has a 3-stage pipeline (Fetch, Decode, Execute) with a branch predictor.

---

## Memory Hierarchy

Faster memory is smaller and more expensive. The hierarchy exploits **locality** — programs tend to reuse the same data and instructions.

```
Registers     │ < 1 ns  │  < 1 KB   │ Inside CPU core
──────────────┼─────────┼───────────┤
L1 Cache      │  1–4 ns │  32–64 KB │ On-chip, per core
──────────────┼─────────┼───────────┤
L2 Cache      │  4–12 ns│  256 KB–4 MB │ On-chip
──────────────┼─────────┼───────────┤
L3 Cache      │ 10–40 ns│  4–64 MB  │ Shared on-chip
──────────────┼─────────┼───────────┤
DRAM (RAM)    │ 50–100 ns│ GBs       │ Off-chip
──────────────┼─────────┼───────────┤
Flash / SSD   │ µs–ms   │ GBs–TBs   │ Non-volatile
```

**For MCUs (STM32):**
- Flash: code lives here (read-only during execution)
- SRAM: stack, heap, globals (fast, volatile)
- No cache on Cortex-M0/M3; Cortex-M4/M7 may have I-cache and D-cache

---

## Buses

A bus is a shared communication channel between components.

**Three main buses:**
- **Address bus** — CPU → memory: "I want to access address 0x20000100" (unidirectional)
- **Data bus** — bidirectional: carries the actual data being read/written
- **Control bus** — read/write signals, clock, interrupt lines

**AHB / APB in STM32:**
```
CPU Core
   │
  AHB (Advanced High-performance Bus) — fast, for DMA, GPIO, Flash
   │
  APB bridge
   │
  APB1 (36 MHz max) — UART, SPI, I2C, TIM2–7
  APB2 (72 MHz max) — USART1, SPI1, TIM1, ADC
```

Peripheral clock speed is set by the APB prescaler in RCC. This is why you must enable the peripheral clock (`RCC->APB1ENR |= ...`) before using a peripheral.

---

## CPU Registers (ARM Cortex-M)

```
General purpose:  R0–R12   (32-bit each)
Stack Pointer:    R13 (SP) — points to top of stack
Link Register:    R14 (LR) — stores return address when calling a function
Program Counter:  R15 (PC) — address of next instruction to execute

Special:
  xPSR  — Program Status Register (flags: N, Z, C, V, thumb state)
  PRIMASK  — disable/enable all interrupts
  CONTROL  — thread/handler mode, stack select
```

**Calling convention (AAPCS):** R0–R3 pass function arguments and return values. R4–R11 are callee-saved (function must restore them). R12 (IP) is scratch.

---

## Endianness

The byte order in which multi-byte values are stored in memory.

```
Value: 0xDEADBEEF

Little-endian (x86, ARM default):
Address: 0x00  0x01  0x02  0x03
Value:   0xEF  0xBE  0xAD  0xDE   ← least significant byte first

Big-endian (network byte order, some older MCUs):
Address: 0x00  0x01  0x02  0x03
Value:   0xDE  0xAD  0xBE  0xEF   ← most significant byte first
```

**Why it matters:** when serializing data over a network or UART, both sides must agree on byte order. Use `htonl()`/`ntohl()` when writing network code.

---

## Key Numbers to Know

| Item | Value |
|------|-------|
| ARM Cortex-M4 max clock | 168 MHz (STM32F4) |
| STM32F4 Flash | 1 MB |
| STM32F4 SRAM | 192 KB |
| sizeof(int) on 32-bit ARM | 4 bytes |
| sizeof(pointer) on 32-bit ARM | 4 bytes |
| Default stack size (STM32) | 1–4 KB (linker script) |
| Interrupt latency (Cortex-M) | 12 cycles (no FPU) |
