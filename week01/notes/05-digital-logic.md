# Day 5 — Digital Logic
**Date: May 11, 2026**

Topics: logic gates, Boolean algebra, De Morgan's laws, combinational vs sequential logic, flip-flops, number systems

---

## Why Digital Logic?

Everything in a computer — the CPU, your MCU's peripherals, every register — is built from transistors implementing logic gates. Understanding this layer helps you reason about timing, glitches, and why certain hardware constraints exist (setup/hold time, metastability).

---

## Logic Gates

### Basic Gates

```
AND gate: output is 1 only if ALL inputs are 1
  A ─┐
     ├─[AND]─── Y = A·B
  B ─┘

OR gate: output is 1 if ANY input is 1
  A ─┐
     ├─[OR]──── Y = A+B
  B ─┘

NOT gate: output is the inverse of input
  A ─[NOT]───── Y = Ā
```

### Full Truth Tables

```
AND          OR           XOR          NAND         NOR          XNOR
A B | Y      A B | Y      A B | Y      A B | Y      A B | Y      A B | Y
0 0 | 0      0 0 | 0      0 0 | 0      0 0 | 1      0 0 | 1      0 0 | 1
0 1 | 0      0 1 | 1      0 1 | 1      0 1 | 1      0 1 | 0      0 1 | 0
1 0 | 0      1 0 | 1      1 0 | 1      1 0 | 1      1 0 | 0      1 0 | 0
1 1 | 1      1 1 | 1      1 1 | 0      1 1 | 0      1 1 | 0      1 1 | 1
```

### C Implementation

```c
#include <stdint.h>

// 1-bit gates (using values 0 or 1)
uint8_t gate_and (uint8_t a, uint8_t b) { return  (a & b) & 1; }
uint8_t gate_or  (uint8_t a, uint8_t b) { return  (a | b) & 1; }
uint8_t gate_xor (uint8_t a, uint8_t b) { return  (a ^ b) & 1; }
uint8_t gate_not (uint8_t a)            { return  (~a)    & 1; }
uint8_t gate_nand(uint8_t a, uint8_t b) { return ~(a & b) & 1; }
uint8_t gate_nor (uint8_t a, uint8_t b) { return ~(a | b) & 1; }
uint8_t gate_xnor(uint8_t a, uint8_t b) { return ~(a ^ b) & 1; }
```

### Universal Gates

**NAND and NOR are universal** — you can build any logic function from only NAND gates (or only NOR gates). This is why most CMOS logic is actually implemented using NAND gates at the transistor level.

```
NOT from NAND:    A ─┬─[NAND]─ Ā       (tie both inputs together)
AND from NAND:    NAND followed by NOT
OR from NAND:     NOT both inputs, then NAND
```

---

## Boolean Algebra

Rules for simplifying logic expressions. Simpler expressions → fewer gates → less power, less area, faster.

### Identity Laws
```
A · 1 = A          A + 0 = A
A · 0 = 0          A + 1 = 1
```

### Idempotent & Complement
```
A · A = A          A + A = A
A · Ā = 0          A + Ā = 1    (complementation)
Ā̄ = A                           (double negation)
```

### Commutative & Associative
```
A · B = B · A       A + B = B + A
(A·B)·C = A·(B·C)  (A+B)+C = A+(B+C)
```

### Distributive & Absorption
```
A·(B+C) = A·B + A·C    (AND distributes over OR)
A+(B·C) = (A+B)·(A+C)  (OR distributes over AND)
A + A·B = A             (absorption)
A · (A+B) = A           (absorption)
```

### De Morgan's Laws — Most Important

```
~(A · B) = Ā + B̄      NAND = OR with inverted inputs
~(A + B) = Ā · B̄      NOR  = AND with inverted inputs
```

In C:
```c
// De Morgan's: ~(A & B) == (~A | ~B)
uint8_t a = 1, b = 0;
assert( !(a && b) == (!a || !b) );   // true for all a, b

// Useful for simplifying complex conditions:
// if (!(x > 5 && y < 10))  ←→  if (x <= 5 || y >= 10)
```

### Example Simplification

```
F = Ā·B + A·B
  = B·(Ā + A)      factor out B
  = B·1             complement law
  = B               simplified: F only depends on B
```

---

## Combinational vs Sequential Logic

| Property | Combinational | Sequential |
|----------|--------------|------------|
| Output depends on | Current inputs only | Current inputs + stored state |
| Memory | None | Has state (flip-flops) |
| Clock needed | No | Usually yes |
| Examples | Gates, adder, mux, decoder | Flip-flop, register, counter, FSM |

---

## Flip-Flops

The fundamental memory element. Stores 1 bit. The building block of all digital state machines, registers, and RAM.

### SR Latch (unclocked)

```
S (Set)   ─[NAND]─┬─ Q
                   └──[NAND]─ Q̄
R (Reset) ─[NAND]─┘

S=1, R=0 → Q=1   (Set)
S=0, R=1 → Q=0   (Reset)
S=0, R=0 → Q unchanged (Hold)
S=1, R=1 → Forbidden! (both outputs try to be 1 → unpredictable)
```

### D Flip-Flop (most important for you)

"Data" flip-flop. On the **rising edge** of CLK, captures the value of D into Q. Q holds until the next rising edge. This is how every register in an MCU works.

```
     D ──[D FF]── Q
    CLK ──↑ (edge triggered)

Timing diagram:
CLK  ─┐  ┌──┐  ┌──┐  ┌──┐
      └──┘  └──┘  └──┘  └──
D    ──1──────0─────1──────
Q    ───────1─────0─────1──  ← Q changes on CLK rising edge only
              ↑     ↑
          captured captured
```

```c
typedef struct {
    uint8_t q;        // current output
    uint8_t q_bar;    // inverted output
} DFlipFlop_t;

// Call this on every simulated rising clock edge
void d_ff_rising_edge(DFlipFlop_t *ff, uint8_t d) {
    ff->q     = d & 1;
    ff->q_bar = (~d) & 1;
}
```

### JK Flip-Flop

Resolves the forbidden SR state. When J=1, K=1 → Q toggles.

```
J=0, K=0 → Q unchanged (Hold)
J=1, K=0 → Q=1         (Set)
J=0, K=1 → Q=0         (Reset)
J=1, K=1 → Q toggles   (Toggle) ← no forbidden state
```

### T (Toggle) Flip-Flop

Simplest: when T=1, Q toggles on each clock edge. The basis for binary counters.

```c
void t_ff_rising_edge(DFlipFlop_t *ff, uint8_t t) {
    if (t) ff->q ^= 1;   // toggle
}
```

---

## Combinational Circuits

### Half Adder

Adds two 1-bit numbers. Produces Sum and Carry.

```
Sum   = A XOR B
Carry = A AND B
```

```c
void half_adder(uint8_t a, uint8_t b, uint8_t *sum, uint8_t *carry) {
    *sum   = a ^ b;
    *carry = a & b;
}
```

### Full Adder

Adds two 1-bit numbers plus a carry-in.

```
Sum   = A XOR B XOR Cin
Carry = (A AND B) OR (B AND Cin) OR (A AND Cin)
```

```c
void full_adder(uint8_t a, uint8_t b, uint8_t cin,
                uint8_t *sum, uint8_t *cout) {
    *sum  = a ^ b ^ cin;
    *cout = (a & b) | (b & cin) | (a & cin);
}
```

Chain 8 full adders to build an 8-bit ripple carry adder — this is how the ALU computes addition.

### Multiplexer (MUX)

Selects one of N inputs based on a select signal. Like a rotary switch in hardware.

```
2-to-1 MUX:  Y = (SEL=0) ? A : B
             Y = (~SEL & A) | (SEL & B)

4-to-1 MUX: uses 2 select bits
```

```c
uint8_t mux2(uint8_t a, uint8_t b, uint8_t sel) {
    return sel ? b : a;
}

uint8_t mux4(uint8_t in[4], uint8_t sel) {
    return in[sel & 0b11];
}
```

### Decoder

Takes an n-bit binary input and activates exactly one of 2ⁿ outputs.

```
2-to-4 decoder (sel=0b10 → activate output 2):
  sel  | out3 out2 out1 out0
  0b00 |  0    0    0    1
  0b01 |  0    0    1    0
  0b10 |  0    1    0    0
  0b11 |  1    0    0    0
```

Used in: memory address decoding, GPIO pin selection.

---

## Number Systems

### Conversion Table

| Decimal | Binary | Hex |
|---------|--------|-----|
| 0  | 0000 0000 | 0x00 |
| 10 | 0000 1010 | 0x0A |
| 15 | 0000 1111 | 0x0F |
| 16 | 0001 0000 | 0x10 |
| 255| 1111 1111 | 0xFF |

**Hex ↔ Binary:** each hex digit = exactly 4 bits:
```
0xAB = 0b1010 1011
         A      B
```

### Two's Complement (How CPUs Store Negative Numbers)

For an n-bit signed integer:
- Range: -2^(n-1) to 2^(n-1) - 1
- For 8-bit: -128 to +127

```
To negate a value: invert all bits, then add 1

+5  = 0000 0101
Step 1 (invert): 1111 1010
Step 2 (add 1):  1111 1011  = -5

Verify: 5 + (-5) = 0000 0101 + 1111 1011 = 0000 0000  ✓ (carry out discarded)

Special cases:
 0 = 0000 0000
-1 = 1111 1111  (all ones)
-128 = 1000 0000  (most negative, no positive counterpart)
+127 = 0111 1111  (most positive)
```

**Sign extension:** when widening a signed value (int8 → int32), fill the upper bits with the sign bit:
```c
int8_t  a = -5;          // 0xFB = 1111 1011
int32_t b = (int32_t)a;  // 0xFFFFFFFB = 1111...1111 1011
                          // upper 24 bits filled with 1 (sign bit)
```

---

## Today's Mini-Exercises

1. Build a `print_truth_table` function in C. Pass a function pointer to each gate and print all input combinations.
2. Implement a D flip-flop struct with a `clock()` function. Chain 4 of them together as a 4-bit shift register.
3. Build a full 8-bit ripple carry adder using 8 `full_adder()` calls. Add 170 + 85 and verify the result.
4. Prove De Morgan's law in code: for all 2-bit combinations of A and B, verify that `!(A && B) == (!A || !B)`.

---

## YouTube Resources

| Topic | Video | Channel |
|-------|-------|---------|
| Logic gates intro | [Logic Gates](https://www.youtube.com/watch?v=gI-qXk7XojA) | CrashCourse |
| Boolean algebra | [Boolean Algebra](https://www.youtube.com/results?search_query=boolean+algebra+simplification+neso+academy) | Neso Academy |
| Flip flops explained | [Flip Flops](https://www.youtube.com/results?search_query=d+flip+flop+explained+digital+electronics) | Neso Academy |
| How computers add | [Adding Numbers in Binary](https://www.youtube.com/watch?v=VBDoT8o4q00) | CrashCourse |
| Two's complement | [Two's Complement](https://www.youtube.com/results?search_query=twos+complement+binary+explained) | Ben Eater |
