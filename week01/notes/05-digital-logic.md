# Digital Logic

## Logic Gates

Every digital circuit is built from these. In C, simulate them with bit operations.

| Gate | Symbol | Expression | Truth table |
|------|--------|-----------|-------------|
| AND  | `A & B` | `A · B` | 1 only when both inputs 1 |
| OR   | `A \| B` | `A + B` | 1 when any input is 1 |
| NOT  | `~A` | `Ā` | Inverts the input |
| NAND | `~(A & B)` | `A · B` overbar | AND then invert — universal gate |
| NOR  | `~(A \| B)` | `A + B` overbar | OR then invert — universal gate |
| XOR  | `A ^ B` | `A ⊕ B` | 1 when inputs differ |
| XNOR | `~(A ^ B)` | | 1 when inputs are equal |

**Universal gates:** NAND and NOR can each implement any logic function on their own. Most real CMOS logic is built from NAND gates.

### Truth Tables

```
AND        OR         XOR        NAND
A B │ Out  A B │ Out  A B │ Out  A B │ Out
0 0 │  0   0 0 │  0   0 0 │  0   0 0 │  1
0 1 │  0   0 1 │  1   0 1 │  1   0 1 │  1
1 0 │  0   1 0 │  1   1 0 │  1   1 0 │  1
1 1 │  1   1 1 │  1   1 1 │  0   1 1 │  0
```

---

## Boolean Algebra

Rules for simplifying logic expressions.

| Identity | Expression |
|----------|-----------|
| AND identity | `A · 1 = A` |
| OR identity | `A + 0 = A` |
| AND null | `A · 0 = 0` |
| OR null | `A + 1 = 1` |
| Idempotent | `A · A = A`, `A + A = A` |
| Complement | `A · Ā = 0`, `A + Ā = 1` |
| Double negation | `~~A = A` |
| De Morgan's | `~(A · B) = Ā + B̄` |
| De Morgan's | `~(A + B) = Ā · B̄` |

**De Morgan's Law** is the most important rule in digital design:
```
~(A & B) == (~A | ~B)   // NAND = NOT-AND = OR with inverted inputs
~(A | B) == (~A & ~B)   // NOR  = NOT-OR  = AND with inverted inputs
```

---

## Combinational vs Sequential Logic

| | Combinational | Sequential |
|---|---|---|
| Output depends on | Current inputs only | Current inputs + past state |
| Memory | None | Has memory (state) |
| Examples | Gates, mux, decoder, adder | Flip-flops, registers, counters |
| Clock | Not needed | Usually clocked |

---

## Flip-Flops

A flip-flop stores 1 bit. It's the basic memory element in digital systems.

### D Flip-Flop (most common)

- **D** — Data input
- **CLK** — Clock input
- **Q** — Output
- **Q̄** — Complemented output

**Behavior:** On the rising edge of CLK, Q captures the value of D. Q holds that value until the next rising edge.

```
     D ──┬──[D]──Q──
         │   ↑
        CLK  (rising edge triggered)
```

Truth table:
```
CLK edge  D  │  Q (next)
─────────────┼──────────
Rising    0  │     0
Rising    1  │     1
No edge   X  │  Q (unchanged)
```

**In C simulation:**
```c
typedef struct {
    uint8_t q;   // current output state
} DFlipFlop;

void d_ff_clock(DFlipFlop *ff, uint8_t d) {
    ff->q = d;   // on rising clock edge, capture D
}
```

### SR Latch (Set-Reset)

```
S=1, R=0 → Q=1 (Set)
S=0, R=1 → Q=0 (Reset)
S=0, R=0 → Q unchanged (Hold)
S=1, R=1 → Forbidden (undefined)
```

### JK Flip-Flop

Fixes the forbidden state of SR: when J=1 and K=1, output toggles.

---

## Half Adder & Full Adder

**Half adder** — adds two 1-bit inputs:
```c
uint8_t sum   = A ^ B;     // XOR
uint8_t carry = A & B;     // AND
```

**Full adder** — adds two bits plus carry-in:
```c
uint8_t sum   = A ^ B ^ Cin;
uint8_t carry = (A & B) | (B & Cin) | (A & Cin);
```

Chain 8 full adders to make an 8-bit ripple carry adder — this is how the ALU in a CPU adds numbers.

---

## Multiplexer (Mux)

Selects one of N inputs based on a select signal.

**2-to-1 mux:**
```c
uint8_t mux2(uint8_t a, uint8_t b, uint8_t sel) {
    return sel ? b : a;
}
// Boolean: out = (~sel & a) | (sel & b)
```

Used everywhere: bus switching, data routing, GPIO alternate function selection.

---

## Number Systems

| System | Base | Digits | C prefix |
|--------|------|--------|---------|
| Binary | 2 | 0–1 | `0b` (GCC extension) |
| Octal | 8 | 0–7 | `0` |
| Decimal | 10 | 0–9 | none |
| Hexadecimal | 16 | 0–9, A–F | `0x` |

Conversions:
```
0xFF   = 1111 1111 = 255
0x0F   = 0000 1111 = 15
0b1010 = 0x0A      = 10
```

**Two's complement** — how CPUs represent negative integers:
```
For 8-bit signed:
+5  = 0000 0101
-5  = 1111 1011   (invert all bits, then add 1)
-1  = 1111 1111
-128= 1000 0000   (most negative)
+127= 0111 1111   (most positive)
```

---

## Practical: Logic Gate Simulator in C

```c
#include <stdio.h>
#include <stdint.h>

uint8_t gate_and (uint8_t a, uint8_t b) { return a & b; }
uint8_t gate_or  (uint8_t a, uint8_t b) { return a | b; }
uint8_t gate_xor (uint8_t a, uint8_t b) { return a ^ b; }
uint8_t gate_nand(uint8_t a, uint8_t b) { return ~(a & b) & 1; }
uint8_t gate_nor (uint8_t a, uint8_t b) { return ~(a | b) & 1; }
uint8_t gate_xnor(uint8_t a, uint8_t b) { return ~(a ^ b) & 1; }
uint8_t gate_not (uint8_t a)            { return !a; }

void print_truth_table_2input(const char *name,
                               uint8_t (*gate)(uint8_t, uint8_t)) {
    printf("%-5s | A B | Out\n", name);
    printf("------+-----+----\n");
    for (uint8_t a = 0; a < 2; a++)
        for (uint8_t b = 0; b < 2; b++)
            printf("      | %d %d |  %d\n", a, b, gate(a, b));
    printf("\n");
}
```
