# Day 3 — Bit Manipulation
**Date: May 9, 2026**

Topics: bitwise operators, masks, shifts, bitfields, real MCU register programming

---

## Why Bit Manipulation?

Every hardware register on a microcontroller is a collection of bits. You don't write `UART.enable = true` — you set bit 13 of a 32-bit register. Bit manipulation is the language of hardware.

```
STM32 USART_CR1 register (32 bits):
Bit 13: UE   — UART Enable
Bit 12: M    — Word length
Bit 10: PCE  — Parity control enable
Bit  5: RXNEIE — RX interrupt enable
Bit  3: TE   — Transmitter enable
Bit  2: RE   — Receiver enable
...
```

You must set/clear these bits without touching others — that's what bit manipulation is for.

---

## The Six Bitwise Operators

```c
uint8_t a = 0b11001010;  // 0xCA = 202
uint8_t b = 0b10110110;  // 0xB6 = 182

a & b   // AND:  0b10000010 = 0x82  — 1 only where BOTH are 1
a | b   // OR:   0b11111110 = 0xFE  — 1 where EITHER is 1
a ^ b   // XOR:  0b01111100 = 0x7C  — 1 where inputs DIFFER
~a      // NOT:  0b00110101 = 0x35  — flip every bit
a << 2  // LEFT SHIFT:  0b00101000  — shift left by 2, fill 0s on right
a >> 2  // RIGHT SHIFT: 0b00110010  — shift right by 2 (logical for unsigned)
```

**Mental model for each:**
- AND: "keep only the bits I care about" (masking)
- OR: "force bits ON"
- XOR: "toggle bits"
- NOT: "invert everything"
- Left shift: multiply by 2ⁿ
- Right shift: divide by 2ⁿ (unsigned)

---

## The Four Register Operations

These four are the foundation of all bare-metal MCU code:

### 1. Set a Bit — force to 1, leave others unchanged
```c
// Formula: register |= (1 << bit_position)

uint32_t reg = 0b00000000;
reg |= (1 << 5);   // result: 0b00100000
reg |= (1 << 0);   // result: 0b00100001
reg |= (1 << 7);   // result: 0b10100001

// Real MCU example: enable GPIOA clock
RCC->AHB1ENR |= (1 << 0);   // bit 0 = GPIOAEN
```

### 2. Clear a Bit — force to 0, leave others unchanged
```c
// Formula: register &= ~(1 << bit_position)

// ~(1 << 5) = ~0b00100000 = 0b11011111  (mask with bit 5 = 0, rest = 1)
reg &= ~(1 << 5);   // clears bit 5, leaves all others

// Real MCU: configure PA5 as output (clear then set MODER bits)
GPIOA->MODER &= ~(0b11 << 10);   // clear bits [11:10]
GPIOA->MODER |=  (0b01 << 10);   // set to 0b01 = output
```

### 3. Toggle a Bit — flip without checking current value
```c
// Formula: register ^= (1 << bit_position)

reg ^= (1 << 5);   // if 0 → 1, if 1 → 0

// Real MCU: blink LED (toggle PA5 in a loop)
while (1) {
    GPIOA->ODR ^= (1 << 5);
    delay_ms(500);
}
```

### 4. Check a Bit — read its value
```c
// Formula: if (register & (1 << bit_position))

if (reg & (1 << 5)) {
    // bit 5 is SET
}

// Real MCU: poll until ADC conversion complete
while (!(ADC1->SR & (1 << 1)));  // wait for bit 1 (EOC) to be set

// Read a button on PC13 (Nucleo)
if (!(GPIOC->IDR & (1 << 13))) {
    // button pressed (active low)
}
```

---

## Masks

A mask is a constant that defines which bits you're working with. Using named masks makes code readable and maintainable.

```c
// Define masks at the top of your file
#define LED_PIN         5
#define LED_MASK        (1U << LED_PIN)      // 0x00000020

#define UART_UE_BIT     13
#define UART_TE_BIT     3
#define UART_RE_BIT     2
#define UART_UE_MASK    (1U << UART_UE_BIT)
#define UART_TE_MASK    (1U << UART_TE_BIT)
#define UART_RE_MASK    (1U << UART_RE_BIT)

// Usage
GPIOA->ODR     |=  LED_MASK;                 // LED on
GPIOA->ODR     &= ~LED_MASK;                 // LED off
USART1->CR1    |=  UART_UE_MASK | UART_TE_MASK | UART_RE_MASK;
```

### Multi-Bit Field Mask

For fields wider than 1 bit:

```c
// Extract bits [3:1] (3-bit field starting at bit 1)
#define FIELD_POS   1
#define FIELD_WIDTH 3
#define FIELD_MASK  (((1U << FIELD_WIDTH) - 1) << FIELD_POS)
//                = ((0b111) << 1) = 0b00001110

// Read the field
uint8_t value = (reg & FIELD_MASK) >> FIELD_POS;

// Write a new value into the field (read-modify-write)
uint8_t new_val = 0b101;
reg = (reg & ~FIELD_MASK) | ((new_val << FIELD_POS) & FIELD_MASK);
```

**Real example — GPIO MODER (2 bits per pin):**
```c
// Set pins PA5 to output mode (0b01) and PA6 to alternate function (0b10)
#define PIN5_POS  (5 * 2)    // 10
#define PIN6_POS  (6 * 2)    // 12

GPIOA->MODER &= ~((0b11 << PIN5_POS) | (0b11 << PIN6_POS));
GPIOA->MODER |=   (0b01 << PIN5_POS) | (0b10 << PIN6_POS);
```

---

## Bitfields in Structs

The compiler handles field extraction for you. Good for readability, but layout is implementation-defined — use only for internal code, not for wire formats.

```c
typedef union {
    uint32_t raw;
    struct {
        uint32_t PE    : 1;   // bit 0  — Parity error
        uint32_t FE    : 1;   // bit 1  — Framing error
        uint32_t NF    : 1;   // bit 2  — Noise flag
        uint32_t ORE   : 1;   // bit 3  — Overrun error
        uint32_t IDLE  : 1;   // bit 4  — Idle line detected
        uint32_t RXNE  : 1;   // bit 5  — Read data register not empty
        uint32_t TC    : 1;   // bit 6  — Transmission complete
        uint32_t TXE   : 1;   // bit 7  — Transmit data register empty
        uint32_t       : 24;  // bits [31:8] — reserved
    } bits;
} USART_SR_t;

volatile USART_SR_t *sr = (USART_SR_t *)&USART1->SR;

if (sr->bits.RXNE) {
    char c = USART1->DR;   // read received byte
}
```

---

## Shifts

```c
// Left shift = multiply by 2^n
uint32_t x = 3;
x << 1;   // 6   (3 × 2)
x << 3;   // 24  (3 × 8)

// Right shift on unsigned = divide by 2^n (logical: fills with 0)
uint32_t y = 128;
y >> 1;   // 64
y >> 3;   // 16

// Right shift on signed = arithmetic shift (fills with sign bit)
// Result is implementation-defined — avoid signed right shift
int8_t z = -8;      // 0b11111000
z >> 1;             // 0b11111100 = -4  (on most compilers)

// Common shift patterns
(1 << n)            // create a mask with only bit n set
~(1 << n)           // mask with only bit n CLEAR
((1 << n) - 1)      // mask for n lowest bits: n=4 → 0b00001111
```

---

## Essential Bit Tricks

```c
// Check if power of 2 (single bit set)
bool is_power_of_two(uint32_t n) {
    return n && !(n & (n - 1));
    // n=8 (0b1000): n-1=7 (0b0111): 8&7 = 0 → true
}

// Count set bits (Kernighan's method)
int popcount(uint32_t n) {
    int count = 0;
    while (n) {
        n &= (n - 1);   // clear the lowest set bit
        count++;
    }
    return count;
}
// Or: __builtin_popcount(n) — compiles to a single CPU instruction on ARM

// Reverse bits in a byte (3-step butterfly)
uint8_t reverse_bits(uint8_t b) {
    b = (b & 0xF0) >> 4 | (b & 0x0F) << 4;  // swap nibbles
    b = (b & 0xCC) >> 2 | (b & 0x33) << 2;  // swap pairs
    b = (b & 0xAA) >> 1 | (b & 0x55) << 1;  // swap adjacent bits
    return b;
}

// Rotate left (barrel roll — no standard C operator)
uint32_t rol32(uint32_t val, uint8_t n) {
    return (val << n) | (val >> (32 - n));
}

// Rotate right
uint32_t ror32(uint32_t val, uint8_t n) {
    return (val >> n) | (val << (32 - n));
}

// Extract a nibble
uint8_t high = (byte >> 4) & 0x0F;
uint8_t low  =  byte       & 0x0F;

// Swap two variables without a temp (using XOR)
a ^= b;   // a = a XOR b
b ^= a;   // b = b XOR (a XOR b) = original a
a ^= b;   // a = (a XOR b) XOR a = original b

// Endian swap 16-bit
uint16_t bswap16(uint16_t x) { return (x >> 8) | (x << 8); }

// Endian swap 32-bit
uint32_t bswap32(uint32_t x) {
    return ((x & 0xFF000000) >> 24) |
           ((x & 0x00FF0000) >>  8) |
           ((x & 0x0000FF00) <<  8) |
           ((x & 0x000000FF) << 24);
}
// Or: __builtin_bswap32(x) on GCC
```

---

## Real Full Example — UART Init (STM32, bare-metal)

```c
#define RCC_BASE    0x40023800
#define USART2_BASE 0x40004400

// Enable APB1 clock for USART2 (bit 17)
*(volatile uint32_t *)(RCC_BASE + 0x40) |= (1 << 17);

// Enable APB1 clock for GPIOA (bit 0 of AHB1ENR)
*(volatile uint32_t *)(RCC_BASE + 0x30) |= (1 << 0);

// Set PA2 (TX) and PA3 (RX) to alternate function mode (0b10)
volatile uint32_t *GPIOA_MODER = (volatile uint32_t *)0x40020000;
*GPIOA_MODER &= ~((0b11 << 4) | (0b11 << 6));   // clear PA2, PA3
*GPIOA_MODER |=   (0b10 << 4) | (0b10 << 6);    // set to AF mode

// Set BRR for 115200 baud @ 16 MHz: BRR = 16000000 / 115200 = 139 (0x8B)
*(volatile uint32_t *)(USART2_BASE + 0x08) = 0x8B;

// Enable USART2: UE (bit13), TE (bit3), RE (bit2)
*(volatile uint32_t *)(USART2_BASE + 0x0C) |= (1 << 13) | (1 << 3) | (1 << 2);
```

This is the raw form — production code wraps these in CMSIS macros, but under the hood it's identical.

---

## Today's Mini-Exercises

1. Write `set_bit`, `clear_bit`, `toggle_bit`, `check_bit` as macros and as inline functions. Test with `assert()`.
2. Write a function that takes a 32-bit register value and a pin number (0–31) and returns a string showing which bits are set.
3. Implement `reverse_bits` and `rotate_left` without using any loops — pure bitwise operations.
4. Simulate setting up a UART: start with `reg = 0`, then set the UE, TE, and RE bits. Print the final hex value and verify it's `0x000020C`.

---

## YouTube Resources

| Topic | Video | Channel |
|-------|-------|---------|
| Bit manipulation intro | [Bit Manipulation](https://www.youtube.com/watch?v=NLKQEOgBAnw) | CS50 Shorts |
| Bitwise operators in C | [Bitwise Operators in C](https://www.youtube.com/results?search_query=bitwise+operators+c+programming+neso+academy) | Neso Academy |
| Embedded register manipulation | [Bare Metal STM32 GPIO](https://www.youtube.com/results?search_query=bare+metal+stm32+gpio+register+bit+manipulation) | Fastbit Embedded |
| Bit tricks | [Bit Hacks](https://www.youtube.com/results?search_query=bit+manipulation+tricks+embedded+c) | Low Level Learning |
| How registers work | [MCU Peripheral Registers Explained](https://www.youtube.com/results?search_query=microcontroller+peripheral+registers+explained) | Phil's Lab |
