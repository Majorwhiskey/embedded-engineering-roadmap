# Bit Manipulation

The single most important skill in embedded systems. Every peripheral register is controlled by individual bits.

## Operators

| Operator | Symbol | Example | Result |
|----------|--------|---------|--------|
| AND | `&` | `0b1100 & 0b1010` | `0b1000` |
| OR | `\|` | `0b1100 \| 0b1010` | `0b1110` |
| XOR | `^` | `0b1100 ^ 0b1010` | `0b0110` |
| NOT | `~` | `~0b1100` | `0b...0011` |
| Left shift | `<<` | `1 << 3` | `0b1000` (= 8) |
| Right shift | `>>` | `0b1000 >> 2` | `0b0010` (= 2) |

---

## The Four Core Operations

### Set a bit
Force a specific bit to 1, leave all others unchanged.
```c
register |= (1 << n);   // set bit n

// Example: set bit 5 of GPIOA_ODR
GPIOA_ODR |= (1 << 5);
```

### Clear a bit
Force a specific bit to 0, leave all others unchanged.
```c
register &= ~(1 << n);  // clear bit n

// Example: clear bit 5
GPIOA_ODR &= ~(1 << 5);
```

### Toggle a bit
Flip a bit: 0 → 1, 1 → 0.
```c
register ^= (1 << n);   // toggle bit n

// Example: toggle bit 5 (blink LED)
GPIOA_ODR ^= (1 << 5);
```

### Check a bit
Test whether a specific bit is 1.
```c
if (register & (1 << n)) { /* bit n is set */ }

// Example: check if button (bit 0 of GPIOB_IDR) is pressed
if (GPIOB_IDR & (1 << 0)) {
    // button pressed
}
```

---

## Masks

A mask is a value used to isolate specific bits.

```c
#define LED_PIN     5
#define LED_MASK    (1 << LED_PIN)   // 0b00100000 = 0x20

// Using the mask:
GPIOA_ODR |=  LED_MASK;   // set
GPIOA_ODR &= ~LED_MASK;   // clear
GPIOA_ODR ^=  LED_MASK;   // toggle
```

**Multi-bit mask** — isolate a field of bits:
```c
// Extract bits [3:1] (a 3-bit field starting at bit 1)
#define FIELD_MASK  0b00001110   // = 0x0E
#define FIELD_POS   1

uint8_t value = (register & FIELD_MASK) >> FIELD_POS;

// Write a 3-bit value into bits [3:1]
register = (register & ~FIELD_MASK) | ((new_val << FIELD_POS) & FIELD_MASK);
```

---

## Bitfields in Structs

Let the compiler manage the bit positions for you:

```c
typedef struct {
    uint32_t mode  : 2;   // bits [1:0]  — 2 bits wide
    uint32_t otype : 1;   // bit  [2]    — 1 bit
    uint32_t ospeed: 2;   // bits [4:3]
    uint32_t pupd  : 2;   // bits [6:5]
    uint32_t       : 25;  // reserved — padding to 32 bits
} GPIO_MODER_t;

volatile GPIO_MODER_t *moder = (GPIO_MODER_t *)0x40020000;
moder->mode = 0b01;   // output mode
```

**Warning:** bitfield layout is implementation-defined. Only use for non-portable internal code, not for network protocol parsing.

---

## Shifts

```c
// Left shift = multiply by 2^n (fast)
int x = 3;
x << 2;   // 3 × 4 = 12

// Right shift = divide by 2^n
// For unsigned: logical shift (fills with 0)
// For signed:   arithmetic shift (fills with sign bit — implementation-defined)
uint32_t a = 16;
a >> 2;   // 16 / 4 = 4

// Build a value from individual bits
uint8_t byte = (bit7 << 7) | (bit6 << 6) | (bit5 << 5) | low_nibble;
```

---

## Useful Patterns

```c
// Check if a number is a power of 2
bool is_power_of_two(uint32_t n) {
    return n && !(n & (n - 1));
}

// Count set bits (popcount)
int count_set_bits(uint32_t n) {
    int count = 0;
    while (n) {
        count += n & 1;
        n >>= 1;
    }
    return count;
}
// or: __builtin_popcount(n) on GCC

// Reverse bits in a byte
uint8_t reverse_bits(uint8_t b) {
    b = (b & 0xF0) >> 4 | (b & 0x0F) << 4;
    b = (b & 0xCC) >> 2 | (b & 0x33) << 2;
    b = (b & 0xAA) >> 1 | (b & 0x55) << 1;
    return b;
}

// Rotate left (no standard operator in C)
uint8_t rotate_left(uint8_t val, uint8_t n) {
    return (val << n) | (val >> (8 - n));
}

// Rotate right
uint8_t rotate_right(uint8_t val, uint8_t n) {
    return (val >> n) | (val << (8 - n));
}

// Extract a nibble
uint8_t high_nibble = (byte >> 4) & 0x0F;
uint8_t low_nibble  =  byte       & 0x0F;

// Swap bytes in a 16-bit value (endian swap)
uint16_t swap16(uint16_t x) {
    return (x >> 8) | (x << 8);
}
```

---

## Real Embedded Example — Configuring STM32 GPIO

```c
// Enable GPIOA clock (bit 0 of RCC_AHB1ENR)
RCC->AHB1ENR |= (1 << 0);

// Set PA5 as output (bits [11:10] of GPIOA_MODER = 0b01)
GPIOA->MODER &= ~(0b11 << 10);   // clear bits 11:10
GPIOA->MODER |=  (0b01 << 10);   // set to output mode

// Set PA5 high
GPIOA->ODR |= (1 << 5);

// Set PA5 low
GPIOA->ODR &= ~(1 << 5);
```

This is exactly how you write bare-metal MCU code — manipulating hardware registers with the four core bit operations.
