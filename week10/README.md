# Week 10 — Debugging, Embedded Security & TLS

**Month 3 · Advanced & Capstone** | Start: July 9, 2026

---

## Topics

🟡 JTAG/SWD: interface differences, OpenOCD setup, probe types (ST-Link, J-Link, CMSIS-DAP)  
🟡 GDB remote debugging: `target remote`, `load`, `break`, `watch`, `info registers`, `backtrace`, `x/`, `disassemble`, `set variable`  
🟡 Hardfault debugging: reading CFSR/HFSR/MMFAR registers, decoding fault type, walking corrupted stack  
🟡 Core dumps on Linux: `ulimit -c`, `gdb executable core`, `backtrace full`  
🟡 Embedded security fundamentals: attack surface mapping, threat modeling (STRIDE), secure coding  
🟡 Common vulnerabilities: buffer overflow, format string, integer overflow, stack smashing — with MCU examples  
🟡 TLS/DTLS: TLS 1.3 handshake, certificate chain, mutual auth (mTLS), cipher suites  
🟡 mbedTLS on MCUs: `mbedtls_config.h`, RAM/flash footprint, `ssl_context` setup, certificate loading  
🩷 Secure MQTT: TLS on port 8883, CA cert + client cert + client key, Mosquitto TLS config  
🩷 SSH hardening: ed25519 only, disable root/password auth, `AllowUsers`, port change, fail2ban, ufw  
🩷 Secure boot: root of trust, code signing, flash encryption (ESP32 eFuse, STM32 RDP levels)  
⬜ CI/CD for firmware: GitHub Actions, build + test on push, QEMU unit tests, artifact upload  

---

## Projects

### 1. GDB Hardfault Crash Investigation
Write firmware with a deliberate null-pointer dereference. Catch in `HardFault_Handler`, read CFSR register, decode fault type in GDB.
- Fix without reflashing by patching memory live

### 2. Secure MQTT with mTLS
Generate CA, server cert, and client cert using `openssl`. Configure Mosquitto with TLS. Deploy client cert to ESP32 with mbedTLS.
- Verify in Wireshark: zero plaintext visible, TLS handshake visible

### 3. Firmware CI Pipeline
GitHub Actions workflow: checkout → build with `arm-none-eabi-gcc` → run Unity unit tests in QEMU → upload `.elf` artifact.
- Add build status badge to README

---

## Submission Checklist

- [ ] Amogh — push work to `week10/amogh/`
- [ ] Vaishnavi — push work to `week10/vaishnavi/`
- [ ] Sujay — push work to `week10/sujay/`
- [ ] Wireshark `.pcap` showing TLS handshake saved
- [ ] GitHub Actions badge visible in your folder README
