# Week 5 — Serial Protocols: UART, I2C, SPI & Sensors

**Month 2 · Systems & Networking** | Start: June 4, 2026

---

## Topics

🟡 UART: frame format (start/data/parity/stop), baud rate calculation, RS-232 vs RS-485, RTS/CTS  
🟡 I2C: 7-bit and 10-bit addressing, START/STOP, ACK/NACK, clock stretching, multi-master arbitration  
🟡 SPI: MOSI/MISO/SCK/CS, CPOL/CPHA modes 0–3, CS management for multiple devices, full-duplex  
🟡 Sensor datasheets: reading register maps, initialization sequences, calibration data  
🩷 Logic analyser: capture and decode live I2C/SPI/UART, timing diagrams  
🩷 1-Wire protocol: DS18B20 temperature sensor, parasite power mode  
⬜ I2S audio protocol: word select, bit clock, data format  

---

## Projects

### 1. I2C Multi-Sensor Dashboard
Read **BMP280** (pressure + temperature) and **MPU6050** (accelerometer + gyroscope) simultaneously via I2C.  
- Format as JSON, stream to PC via UART at 10 Hz  

### 2. SPI Display Driver
Drive an **ILI9341** or **ST7789** LCD via SPI + DMA.  
- Implement minimal graphics library: draw pixel, line, rectangle, text  
- Render live sensor values at 20 fps  

### 3. Protocol Sniffer Exercise
Use logic analyser (or second MCU as sniffer) to capture a full I2C transaction with BMP280.  
- Document every byte against the datasheet register map  

---

## Submission Checklist

- [ ] Amogh — push work to `week05/amogh/`
- [ ] Vaishnavi — push work to `week05/vaishnavi/`
- [ ] Sujay — push work to `week05/sujay/`
- [ ] Logic analyser capture screenshot saved
- [ ] Register map annotation included for sniffer exercise
