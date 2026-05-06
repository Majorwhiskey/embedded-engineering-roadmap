# Embedded Systems Engineering Roadmap

A 3-month collaborative learning roadmap covering C, electronics, bare-metal MCU programming, networking protocols, RTOS, Linux, and security.

## Team

| Person | GitHub Handle |
|--------|--------------|
| Person 1 | @TBD |
| Person 2 | @TBD |
| Person 3 | @TBD |

---

## Roadmap Overview

### Month 1 — Foundations (Weeks 1–4)
C language deep dive, electronics fundamentals, and bare-metal microcontroller programming.

| Week | Topics | Projects |
|------|--------|---------|
| 1 | C fundamentals, pointers, memory model, bit manipulation, computer architecture, digital logic, Git basics | C memory playground (linked list/stack/queue + Valgrind), bit manipulation library, logic gate simulator |
| 2 | MCU architecture, GPIO, timers, interrupts | LED blinky, button interrupt handler |
| 3 | ADC/DAC, PWM, UART | Sensor reader, PWM LED dimmer |
| 4 | I2C, SPI, bootloaders | I2C sensor driver, SPI display |

### Month 2 — Protocols & Connectivity (Weeks 5–8)
Networking, TCP/IP stack, SSH, MQTT, sockets, and RTOS.

| Week | Topics | Projects |
|------|--------|---------|
| 5 | TCP/IP fundamentals, packets, Wireshark | Packet capture analysis, raw socket ping |
| 6 | Sockets programming, MQTT | TCP echo server/client, MQTT pub/sub |
| 7 | SSH internals, cryptography basics | SSH tunnel setup, key-based auth automation |
| 8 | RTOS concepts, FreeRTOS tasks/queues/semaphores | Multitasked sensor pipeline |

### Month 3 — Linux, Security & Capstone (Weeks 9–12)
Embedded Linux, TLS, security hardening, and a final capstone project.

| Week | Topics | Projects |
|------|--------|---------|
| 9 | Embedded Linux, Yocto/Buildroot, device tree | Custom Linux image |
| 10 | TLS/mTLS, certificate management | TLS-secured MQTT client |
| 11 | Security hardening, secure boot, threat modeling | Hardened device configuration |
| 12 | Capstone project | End-to-end embedded + networked system |

---

## Repository Structure

```
embedded-engineering-roadmap/
├── README.md
├── .gitignore
├── week01/
│   ├── person1/      # Individual work
│   ├── person2/
│   └── person3/
├── shared/           # Shared utilities, libraries, and references
└── docs/             # Notes, datasheets, diagrams
```

Each week gets its own top-level folder (`week01/`, `week02/`, ...) with a subdirectory per person for individual work.

---

## Getting Started

### SSH Key Setup (one-time)
```bash
# Generate a key (skip if you already have one)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy the public key
cat ~/.ssh/id_ed25519.pub

# Add it to GitHub: Settings → SSH and GPG keys → New SSH key
```

### Clone and First Commit
```bash
git clone git@github.com:<org>/embedded-engineering-roadmap.git
cd embedded-engineering-roadmap/week01/<your-folder>
echo "# Week 1 — $(git config user.name)" > README.md
git add README.md
git commit -m "week01: initial commit for <your-name>"
git push
```

---

## Conventions

- Branch off `main` for experiments; merge via PR.
- Commit messages: `weekNN: short description` (e.g. `week01: add bit manipulation library`).
- Keep compiled binaries out of the repo — the `.gitignore` handles this.
- Drop shared reference material and utilities in `shared/` or `docs/`.
