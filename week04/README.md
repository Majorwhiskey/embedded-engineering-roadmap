# Week 4 — Interrupts, DMA, Watchdog & Power Management

**Month 1 · Foundations** | Start: May 28, 2026

---

## Topics

🟡 Interrupts: NVIC, ISR rules (keep short, no blocking), priority levels, nested interrupts, EXTI  
🟡 DMA: mem-to-periph, periph-to-mem, mem-to-mem, circular mode, half-transfer interrupt  
🟡 Watchdog timers: IWDG (independent), WWDG (window), kick strategy, reset detection  
🩷 Power modes: sleep, stop, standby, wakeup sources (RTC, pin, UART)  
🩷 Memory management: static vs dynamic, stack overflow detection, heap fragmentation  
🩷 Debugging with SWD + GDB: breakpoints, watchpoints, inspect registers, step through ISR  
⬜ Bootloader / DFU: firmware update over UART, memory map, jump to application  

---

## Projects

### 1. DMA-Driven UART Logger
DMA receives UART data into a circular buffer — zero CPU during transfer.  
- Main loop parses complete commands when idle-line interrupt fires  

### 2. Watchdog-Protected Task Monitor
Three software tasks each kick their own watchdog token within a deadline.  
- Simulate a stuck task, verify system resets, log reboot reason to a register  

### 3. Low-Power Wakeup Node
MCU enters STOP mode, wakes on RTC alarm every 30 s, takes ADC reading, sends 1 UART packet, returns to sleep.  
- Target: **under 10 µA** average current draw  
- Measure with multimeter in µA mode  

---

## Submission Checklist

- [ ] Amogh — push work to `week04/amogh/`
- [ ] Vaishnavi — push work to `week04/vaishnavi/`
- [ ] Sujay — push work to `week04/sujay/`
- [ ] UART DMA project: verify with logic analyser that CPU doesn't toggle during transfer
- [ ] Power project: current measurement screenshot/notes included
