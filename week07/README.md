# Week 7 — Socket Programming, MQTT & Wi-Fi on Microcontrollers

**Month 2 · Systems & Networking** | Start: June 18, 2026

---

## Topics

🟡 BSD socket API: `socket()`, `bind()`, `listen()`, `accept()`, `connect()`, `send()`, `recv()`, `close()`, `setsockopt()`  
🟡 TCP server patterns: single-client loop, multi-client with `select()`, multi-client with threads  
🟡 TCP client: connect with timeout, reconnect logic, heartbeat/keepalive  
🟡 Non-blocking I/O: `fcntl O_NONBLOCK`, `select()`, `poll()`, `epoll()` basics  
🟡 UDP sockets: `sendto()`, `recvfrom()`, broadcast with `SO_BROADCAST`  
🟡 MQTT: broker/client model, topics and wildcards (`#` and `+`), QoS 0/1/2, retained messages, LWT, clean session  
🩷 Wi-Fi on ESP32: station mode, AP mode, WPA2, DHCP client, ESP-IDF event loop  
🩷 lwIP: lightweight TCP/IP stack on MCUs, RTOS integration  
🩷 HTTP client from MCU: GET/POST, parsing JSON responses  
⬜ WebSockets: upgrade handshake, frames, real-time browser ↔ MCU  

---

## Projects

### 1. TCP Echo Server in C (Linux)
Multi-client echo server using `select()`. Handle up to 10 concurrent clients.  
- Log connect/disconnect events  
- Measure round-trip latency under load with `netcat`  

### 2. ESP32 MQTT Weather Station
ESP32 reads BME280, connects to Wi-Fi, publishes JSON to local Mosquitto broker every 5 s.  
- Python subscriber logs to CSV  
- Node-RED dashboard visualizes live data  

### 3. UDP Sensor Broadcast
Embedded device broadcasts sensor reading as UDP packet every second.  
- Python script on LAN receives broadcasts and plots live chart with `matplotlib`  

---

## Tools to Install

```bash
sudo apt install mosquitto mosquitto-clients
npm install -g node-red
# MQTT Explorer: https://mqtt-explorer.com
```

---

## Submission Checklist

- [ ] Amogh — push work to `week07/amogh/`
- [ ] Vaishnavi — push work to `week07/vaishnavi/`
- [ ] Sujay — push work to `week07/sujay/`
- [ ] Echo server handles 10 concurrent clients without crash
- [ ] Node-RED flow JSON exported and saved
