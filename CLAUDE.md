# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What This Repo Is

A 3-person, 12-week embedded systems learning roadmap. Each week has its own folder (`week01/`–`week12/`), each containing a `README.md` with that week's topics and projects, and per-person subfolders (`amogh/`, `vaishnavi/`, `sujay/`) for individual work.

**Team:** Amogh (@Majorwhiskey), Vaishnavi, Sujay  
**Timeline:** May 7 – ~August 6, 2026  
**Repo:** https://github.com/Majorwhiskey/embedded-engineering-roadmap

---

## Build Commands

Projects in this repo are plain C compiled with GCC (host) or `arm-none-eabi-gcc` (MCU targets). There is no repo-wide build system — each project lives in a person's weekly subfolder and has its own Makefile.

### Host (Linux) C projects
```bash
gcc -Wall -Wextra -o output main.c
valgrind --leak-check=full ./output   # memory check
gdb ./output                          # debugging
```

### ARM bare-metal (Weeks 3–8)
```bash
arm-none-eabi-gcc -mcpu=cortex-m4 -mthumb -Wall -Wextra -o firmware.elf main.c
arm-none-eabi-objcopy -O binary firmware.elf firmware.bin
```

### Install core tools (Ubuntu/Debian)
```bash
sudo apt install build-essential valgrind gdb gcc-arm-none-eabi openocd
sudo apt install wireshark iperf3 nmap mosquitto mosquitto-clients
pip install scapy
```

---

## Repository Conventions

- **Commit format:** `weekNN: short description` — e.g. `week03: add PWM servo controller`
- **Work location:** each person commits only inside their own subfolder (`weekNN/<name>/`)
- **Shared code** (reusable headers, scripts, utilities) goes in `shared/`
- **No build artefacts** in commits — `.gitignore` covers `*.o`, `*.elf`, `*.bin`, `*.hex`, `*.map`, `build/`
- `.gitkeep` files exist solely to track empty directories in git — ignore them

---

## Roadmap Structure

| Weeks | Theme | Key skills added |
|-------|-------|-----------------|
| 1–4   | Month 1: Foundations | C, bare-metal MCU, GPIO/timers/ADC/DMA, GDB |
| 5–8   | Month 2: Systems & Networking | Serial protocols, TCP/IP, SSH, sockets, MQTT, FreeRTOS |
| 9–12  | Month 3: Advanced & Capstone | Embedded Linux, TLS/mTLS, OTA, DSP, CAN, capstone |

Each `weekNN/README.md` lists the exact topics (🟡 Required / 🩷 Recommended / ⬜ Optional), projects, and a submission checklist. Start there when helping with a specific week.

---

## Key Constraints to Know

- **Weeks 3–8 MCU code:** bare-metal only — no HAL, no vendor middleware, direct register writes
- **Week 10+ security:** mbedTLS for TLS on MCU; Mosquitto with port 8883 for secure MQTT
- **FreeRTOS (Week 8+):** always log `uxTaskGetStackHighWaterMark` for every task; never call blocking functions from ISR context
- **OTA (Week 11):** dual-bank flash only; always verify SHA256 before marking new partition bootable

---

## Reference Links

Full list in [`docs/references.md`](docs/references.md). Key ones:
- [Embedded Engineering Roadmap (source)](https://github.com/m3y54m/Embedded-Engineering-Roadmap)
- [Wokwi simulator](https://wokwi.com) — use when no physical MCU is available
- [FreeRTOS book](https://www.freertos.org/Documentation/RTOS_book.html)
- [Interrupt blog by Memfault](https://interrupt.memfault.com/blog)
