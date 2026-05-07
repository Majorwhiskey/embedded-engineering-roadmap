# Week 11 — Advanced Protocols, DSP & OTA Updates

**Month 3 · Advanced & Capstone** | Start: July 16, 2026

---

## Topics

🟡 CAN bus: standard (11-bit) vs extended (29-bit) ID, arbitration, error frames, error confinement (active/passive/bus-off), CAN FD basics  
🟡 OBD-II: PID format, request/response pattern, common PIDs (RPM, speed, coolant temp)  
🩷 CoAP: REST over UDP, GET/POST/PUT/DELETE, observe pattern, block-wise transfer, DTLS security  
🟡 DSP fundamentals: Nyquist theorem, aliasing, quantization noise  
🟡 FIR filters: impulse response, window design (Hamming, Blackman), fixed-point implementation  
🩷 IIR filters: biquad sections, direct form II, stability, coefficient quantization  
🩷 FFT: DFT concept, radix-2 Cooley-Tukey, frequency resolution, zero-padding  
🟡 Bootloader / DFU / OTA: dual-bank flash layout, download to inactive bank, SHA256 integrity check, swap on next boot, rollback on failure  
🩷 Cellular IoT: NB-IoT vs LTE-M, AT commands, PSM, eDRX, APN configuration  
🩷 Bluetooth LE: GAP roles (central/peripheral), GATT (services/characteristics/descriptors), Nordic nRF Connect  
🩷 LoRa/LoRaWAN: spreading factor, bandwidth, duty cycle limits, TTN setup  
⬜ AUTOSAR: software component model, RTE, BSW stack — conceptual overview  
⬜ Edge AI: TensorFlow Lite Micro, INT8 quantization, inference on STM32/ESP32  

---

## Projects

### 1. FIR Low-Pass Filter on ADC Signal
Inject a noisy signal (mix 100 Hz + 1 kHz tones) via ADC. Apply a 32-tap FIR low-pass filter in fixed-point C (cutoff 200 Hz).
- Plot raw vs filtered on serial plotter or oscilloscope

### 2. CoAP Resource Server
Expose `/sensors/temperature`, `/sensors/humidity`, `/actuators/led` as CoAP resources on ESP32.
- Use observe mode for live temperature updates
- Control LED from Linux host using `libcoap` CLI

### 3. OTA Firmware Update via MQTT
Dual-bank OTA on ESP32: Python script publishes firmware chunks over MQTT, ESP32 receives, writes to inactive OTA partition, verifies SHA256, sets boot partition, reboots.
- Rollback if first boot fails

---

## Submission Checklist

- [ ] Amogh — push work to `week11/amogh/`
- [ ] Vaishnavi — push work to `week11/vaishnavi/`
- [ ] Sujay — push work to `week11/sujay/`
- [ ] FIR filter: oscilloscope/plotter screenshot showing noise rejection
- [ ] OTA: rollback path tested and documented
