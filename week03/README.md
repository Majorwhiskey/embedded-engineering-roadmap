# Week 3 — MCU Peripherals I: GPIO, Timers, ADC, DAC, PWM

**Month 1 · Foundations** | Start: May 21, 2026

> All bare-metal — no HAL, no vendor middleware. Direct register writes only.

---

## Topics

🟡 GPIO: input/output modes, pull-up/pull-down, open-drain, speed, alternate function  
🟡 Clock management: HSI/HSE, PLL, AHB/APB prescalers, clock tree  
🟡 Timers: basic, general-purpose, input capture, output compare, counter modes  
🟡 PWM: frequency calculation, duty cycle, dead time, complementary outputs  
🟡 ADC: resolution (8/10/12-bit), sampling time, Vref, DMA-driven continuous mode  
🩷 DAC: output voltage, waveform generation with timer trigger  
🩷 Build system: GCC ARM toolchain, Makefile, CMake basics, linker scripts  
⬜ Bootloader basics: reset vector, initial stack pointer, vector table  

---

## Projects

### 1. Timer-Only LED Blinker
Blink LED at exactly **2 Hz** using only a hardware timer interrupt. Zero HAL delays.  
- Verify exact frequency with oscilloscope or logic analyser  

### 2. PWM + ADC Servo Controller
Read potentiometer via ADC (0–4095), map → 1–2 ms pulse width, drive servo via PWM timer.  
- All bare-metal register writes  

### 3. Sine Wave DAC Output
Generate a 1 kHz sine wave using a 256-point lookup table, DAC, and timer interrupt.  
- View clean waveform on oscilloscope  

---

## Recommended Hardware

- STM32 Nucleo-F401RE or Nucleo-F103RB (~$15)  
- OR ESP32 DevKit (~$5)  
- OR use [Wokwi](https://wokwi.com) simulator (free, no hardware needed)  

---

## Submission Checklist

- [ ] Amogh — push work to `week03/amogh/`
- [ ] Vaishnavi — push work to `week03/vaishnavi/`
- [ ] Sujay — push work to `week03/sujay/`
- [ ] No HAL calls — raw register access only
- [ ] Linker script included in build
