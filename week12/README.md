# Week 12 — Capstone Project

**Month 3 · Advanced & Capstone** | Start: July 23, 2026 | End: ~August 6, 2026

---

## Consolidation Checklist Before Building

- [ ] Memory audit: stack high-water marks for every task, heap usage over 24-hour run, no fragmentation or leaks
- [ ] Full integration: TLS + OTA + watchdog + RTOS + networking all working together
- [ ] Power budget: measure current at each system state, calculate battery life estimate
- [ ] Performance profiling: DWT cycle counter for task execution time and ISR latency
- [ ] Documentation: architecture diagram, wiring diagram, API docs, setup instructions in README

---

## Choose One Capstone Option

---

### Option A — Secure IoT Gateway

**Stack:** STM32/ESP32 → FreeRTOS → TLS MQTT → Raspberry Pi → Node-RED dashboard

**Requirements:**
- Collect data from 2+ sensors via I2C and SPI
- FreeRTOS: 4 tasks — sensor, processing, MQTT publish, OTA listener
- TLS MQTT to Raspberry Pi running Mosquitto
- Pi exposes REST API (Python FastAPI) over TCP
- Node-RED dashboard displays live data
- OTA updates pushed from Pi to MCU over MQTT
- GitHub Actions CI builds and tests on every push

**Deliverable:** Full Wireshark capture showing TLS handshake and encrypted payloads

---

### Option B — Remote SSH Sensor Node

**Stack:** Battery-powered STM32 + ESP32 → LittleFS → SFTP sync → UDP dashboard

**Requirements:**
- Read sensors every 30 s, log to LittleFS on-device
- Sync to remote server via SFTP (SSH) when Wi-Fi available
- Server parses CSV, broadcasts UDP dashboard on LAN
- Watchdog, TLS for SFTP, STOP mode sleep between readings
- SSH reverse tunnel for remote debugging

**Deliverable:** Current measurement log showing <5 mA average draw

---

### Option C — Networked CAN Bus Monitor

**Stack:** RPi + MCP2515 → OBD-II parser → WebSocket dashboard → InfluxDB → MQTT alerts

**Requirements:**
- Read CAN frames from vehicle or CAN bus simulator
- Parse OBD-II PIDs: RPM, speed, coolant temp, throttle
- Browser dashboard via WebSocket (Chart.js)
- Raw frame logging to InfluxDB
- MQTT alerts on over-temp / speed threshold
- Remote access via SSH tunnel
- Entire stack as systemd services with watchdog integration

**Deliverable:** InfluxDB query showing 24-hour data retention and alerting log

---

## Final Submission Checklist

- [ ] Amogh — architecture diagram in `week12/amogh/`
- [ ] Vaishnavi — architecture diagram in `week12/vaishnavi/`
- [ ] Sujay — architecture diagram in `week12/sujay/`
- [ ] All source code committed, no build artefacts
- [ ] README in your folder covers: wiring, setup steps, how to flash/run, known limitations
- [ ] 24-hour stability run completed without crash or watchdog reset
