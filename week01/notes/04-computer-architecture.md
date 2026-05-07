# Day 4 — Computer Architecture
**Date: May 10, 2026**

Topics: von Neumann vs Harvard, CPU pipeline, memory hierarchy, buses, ARM registers, endianness

---

## Von Neumann vs Harvard Architecture

This distinction matters because the MCU you'll be programming (ARM Cortex-M) is based on Harvard principles.

### Von Neumann

Single shared bus for both instructions and data. The CPU fetches either an instruction OR reads/writes data — never both simultaneously. This creates the **von Neumann bottleneck**.

```
CPU ←───── Single Bus ─────→ Memory (code + data together)
```

Used by: x86 desktop CPUs (with caches that paper over the bottleneck), some older MCUs.

### Harvard

Separate buses for instructions and data. CPU can fetch an instruction AND read/write data simultaneously — no bottleneck.

```
CPU ←─── Instruction Bus ───→ Flash (code)
    ←─────  Data Bus  ──────→ SRAM  (data)
```

Used by: AVR, PIC, most DSPs. Classic AVR (ATmega328) is a pure Harvard machine.

### Modified Harvard (ARM Cortex-M)

ARM Cortex-M uses modified Harvard: separate instruction and data buses internally (and separate I-cache / D-cache on M7), but a **unified address space**. Code and data share the same 4 GB address range, just accessed via different internal buses.

```
Cortex-M Core
├─ I-Bus → Flash (code fetch)
├─ D-Bus → SRAM  (load/store)
└─ S-Bus → System peripherals (NVIC, SysTick)
    all mapped into one 0x00000000–0xFFFFFFFF address space
```

**Why it matters for you:** your `const` tables live in flash, your variables in SRAM. The core can fetch the next instruction from flash while loading sensor data from SRAM simultaneously.

---

## CPU Pipeline

A pipeline overlaps execution of multiple instructions so the CPU completes one instruction per clock cycle at steady state (throughput = 1 IPC).

### Classic 5-Stage Pipeline (RISC)

```
        Cycle: 1    2    3    4    5    6    7    8
Instruction 1: [IF] [ID] [EX] [MEM][WB]
Instruction 2:      [IF] [ID] [EX] [MEM][WB]
Instruction 3:           [IF] [ID] [EX] [MEM][WB]
Instruction 4:                [IF] [ID] [EX] [MEM][WB]
```

**Stages:**
| Stage | Name | What happens |
|-------|------|-------------|
| IF | Instruction Fetch | PC → memory → instruction register |
| ID | Instruction Decode | Decode opcode, read source registers from register file |
| EX | Execute | ALU operation, address calculation |
| MEM | Memory Access | Load from / store to data memory |
| WB | Write Back | Write ALU result or loaded value to destination register |

### ARM Cortex-M3/M4 Pipeline (3-stage)

ARM simplified to Fetch → Decode → Execute. The branch predictor reduces penalties.

```
Cycle: 1    2    3    4    5
I1:   [F]  [D]  [E]
I2:        [F]  [D]  [E]
I3:             [F]  [D]  [E]
```

**Pipeline hazards — why code sometimes slows down:**

1. **Data hazard:** instruction needs a result that's not written back yet.
   ```
   MOV R0, #5      ; R0 = 5 (written in WB at cycle 5)
   ADD R1, R0, #3  ; needs R0 at EX in cycle 4 → stall
   ```
   Modern CPUs use **forwarding** (passing result directly to the next stage) to reduce stalls.

2. **Control hazard (branch penalty):** on a branch, the CPU doesn't know which instruction comes next until EX.
   ```
   ; CPU fetches instructions after the branch speculatively
   ; If prediction is wrong → flush 1–3 stages → 1–3 wasted cycles
   ```
   Cortex-M4: 1–3 cycle branch penalty. This is why tight loops benefit from loop unrolling.

3. **Load-use hazard:** load followed immediately by use of the loaded value:
   ```
   LDR R0, [R1]    ; load takes extra cycle to access memory
   ADD R2, R0, #1  ; stall: R0 not ready yet
   ```

---

## Memory Hierarchy

Faster = smaller + more expensive + closer to the CPU. The hierarchy exploits **locality of reference**.

```
         Speed      Size        Location
────────────────────────────────────────────────
Registers  < 1 ns    16–32 × 32-bit   Inside CPU core
L1 Cache   1–4 ns    16–64 KB         On-chip per core
L2 Cache   4–12 ns   256 KB–4 MB      On-chip
L3 Cache   10–40 ns  4–64 MB          Shared on-chip
DRAM       50–100 ns GBs              Off-chip PCB
Flash/SSD  µs–ms     GBs–TBs          Off-chip
```

**Two types of locality:**
- **Temporal locality:** if you access address X now, you'll likely access X again soon → keep it in cache
- **Spatial locality:** if you access address X, you'll likely access X+1, X+2 → prefetch a whole cache line (typically 64 bytes)

**For MCUs (Cortex-M0/M3/M4):**
- No data cache (M0/M3/M4) — every load/store accesses SRAM directly
- Cortex-M7: 16/32 KB I-cache and D-cache
- Flash has wait states (typically 3–5 cycles at 168 MHz on STM32F4) — prefetch buffer compensates

**Practical implication:** on STM32F4, sequential instruction execution is fast (prefetch buffer feeds the pipeline), but random jumps (indirect function calls, switch tables) can cause cache misses and slow down significantly.

---

## Buses and the Bus Matrix

A bus is a set of shared wires connecting components. Three conceptual signals:
- **Address bus** (output from CPU): "I want to access this location"
- **Data bus** (bidirectional): the actual data being transferred
- **Control bus**: read/write select, clock, byte-enable signals

### STM32F4 Bus Matrix

```
CPU Core ──┬── AHB (Advanced High-performance Bus) ──┬── DMA1 / DMA2
           │           (up to 168 MHz)                ├── Flash Interface
           │                                          ├── SRAM1 / SRAM2
           │                                          └── AHB→APB Bridge
           │                                                    │
           │                                          ┌─────────┴─────────┐
           │                                        APB1 (42 MHz)       APB2 (84 MHz)
           │                                        TIM2–7              TIM1/8
           │                                        USART2–5            USART1/6
           │                                        SPI2/3              SPI1/4/5/6
           │                                        I2C1/2/3            ADC1/2/3
           │                                        CAN1/2              SDIO
           │
GPIO ──────┴── AHB1 (direct, fast access)
```

**Key insight:** enabling a peripheral's clock (`RCC->APB1ENR |= ...`) connects the peripheral to the bus. Without the clock, reads return 0 and writes are ignored.

---

## ARM Cortex-M Registers

16 general-purpose registers, each 32 bits wide:

```
R0  – R3    : Argument/result registers (function call ABI)
R4  – R11   : Callee-saved (pushed/popped by called function)
R12  (IP)   : Intra-procedure scratch register
R13  (SP)   : Stack Pointer — points to top of stack
R14  (LR)   : Link Register — stores return address on function call
R15  (PC)   : Program Counter — address of current instruction + 4

Special registers:
xPSR        : Program Status Register
              [31] N — Negative flag
              [30] Z — Zero flag
              [29] C — Carry flag
              [28] V — Overflow flag
              [24] T — Thumb state (always 1 on Cortex-M)

PRIMASK     : [0] PM — masks all interrupts except NMI and HardFault
FAULTMASK   : [0] FM — masks all interrupts including faults
BASEPRI     : blocks interrupts at or below this priority level
CONTROL     : selects stack (MSP/PSP) and privilege level
```

**Calling convention (AAPCS — ARM Procedure Call Standard):**
- R0–R3: pass first 4 function arguments; R0 holds return value
- R4–R11: callee must save and restore if used
- R12, LR: caller-saved (may be trashed by called function)

```c
// int add(int a, int b) compiled to ARM:
// a in R0, b in R1
// result returned in R0
ADD R0, R0, R1
BX  LR           // return (branch to link register)
```

---

## Endianness

How multi-byte values are stored across consecutive memory addresses.

```
Value: 0xDEADBEEF (uint32_t)

Little-endian (ARM default, x86):
Address:  +0    +1    +2    +3
Content: 0xEF  0xBE  0xAD  0xDE   ← least significant byte at lowest address

Big-endian (network byte order, Motorola 68k):
Address:  +0    +1    +2    +3
Content: 0xDE  0xAD  0xBE  0xEF   ← most significant byte at lowest address
```

**Visualizing with a union:**
```c
union {
    uint32_t val;
    uint8_t  bytes[4];
} u;

u.val = 0xDEADBEEF;
printf("%02X %02X %02X %02X\n",
       u.bytes[0], u.bytes[1], u.bytes[2], u.bytes[3]);
// On ARM (little-endian): EF BE AD DE
```

**When it matters:**
- Sending integers over UART or network — both sides must agree
- Parsing binary protocols (e.g., sensor register values, network packets)
- `htonl()` / `ntohl()` convert between host and network (big-endian) byte order

ARM Cortex-M is little-endian by default. It can be configured for big-endian but rarely is in practice.

---

## Instruction Set — Thumb / Thumb-2

Cortex-M uses the **Thumb-2** instruction set:
- Mix of 16-bit and 32-bit instructions
- 16-bit instructions: most common ALU ops, loads/stores → better code density
- 32-bit instructions: wider immediates, more registers, DSP ops (MLA, SIMD)
- No 32-bit ARM mode on Cortex-M — always Thumb

```asm
; Cortex-M assembly example (ARM Unified Syntax)
    LDR  R0, =0x40020000   ; load address of GPIOA into R0
    LDR  R1, [R0]          ; load MODER register
    ORR  R1, R1, #(1<<10)  ; set bit 10
    STR  R1, [R0]          ; write back
    BX   LR                ; return
```

---

## Today's Mini-Exercises

1. Draw the pipeline execution diagram for 5 sequential instructions — show where hazards occur.
2. Write a program that demonstrates endianness: store `0xDEADBEEF` in a `uint32_t`, cast to `uint8_t *`, and print each byte.
3. Look up the STM32F401RE datasheet memory map. Find where USART2's base address is and which APB bus it's on.
4. Write a function that takes a 32-bit value and prints whether the system is little-endian or big-endian.

---

## YouTube Resources

| Topic | Video | Channel |
|-------|-------|---------|
| Crash Course Computer Science | [How CPUs Work](https://www.youtube.com/watch?v=cNN_tTXABUA) | CrashCourse |
| CPU Pipeline explained | [Pipelining in CPU](https://www.youtube.com/results?search_query=cpu+pipeline+stages+computer+architecture+explained) | Neso Academy |
| ARM architecture | [ARM Cortex-M Architecture](https://www.youtube.com/results?search_query=arm+cortex+m+architecture+explained) | Phil's Lab |
| Memory hierarchy | [Cache Memory Explained](https://www.youtube.com/watch?v=6JpLD3PUAZk) | CrashCourse |
| Von Neumann vs Harvard | [Von Neumann vs Harvard](https://www.youtube.com/results?search_query=von+neumann+vs+harvard+architecture+embedded) | Neso Academy |
