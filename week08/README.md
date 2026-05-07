# Week 8 — RTOS: FreeRTOS / Zephyr, Tasks, Sync & Scheduling

**Month 2 · Systems & Networking** | Start: June 25, 2026

---

## Topics

🟡 RTOS fundamentals: why RTOS vs bare-metal, preemptive vs cooperative, tick rate, context switch  
🟡 Tasks: `xTaskCreate`, task states (running/ready/blocked/suspended), stack sizing, priorities  
🟡 Queues: `xQueueCreate`, `xQueueSend`, `xQueueReceive`, blocking with timeout, queue sets  
🟡 Mutexes: `xSemaphoreCreateMutex`, priority inheritance, recursive mutexes, deadlock avoidance  
🟡 Semaphores: binary (signalling), counting (resource pools)  
🟡 Event groups: `xEventGroupCreate`, `xEventGroupSetBits`, `xEventGroupWaitBits`  
🩷 Task notifications: `ulTaskNotifyTake`, `xTaskNotifyGive` — lightweight alternative to semaphores  
🩷 Software timers: `xTimerCreate`, one-shot vs auto-reload, timer daemon task  
🩷 Heap management: heap_1–heap_5, `xPortGetFreeHeapSize`, `uxTaskGetStackHighWaterMark`  
🩷 RTOS + networking: FreeRTOS + lwIP, network task, socket calls in tasks  
⬜ Zephyr RTOS: west build system, device tree, Kconfig  

---

## Projects

### 1. RTOS Sensor Pipeline
Three FreeRTOS tasks:
- **Task A** — reads sensor via I2C every 100 ms → pushes to queue
- **Task B** — processes data (calibration, threshold detection)
- **Task C** — publishes to MQTT via Wi-Fi

Protect shared config struct with mutex. Log stack high-water marks every 10 s.

### 2. Priority Inversion Demonstration
Deliberately create a priority inversion: low-priority task holds mutex, high-priority task blocks, medium-priority task starves the system.
- Observe via UART logs
- Fix with priority inheritance mutex and confirm resolution

### 3. RTOS + MQTT Telemetry Node
Combine Week 7 MQTT with FreeRTOS. Separate tasks for Wi-Fi management, sensor reading, MQTT publishing.
- Implement reconnect logic: buffer readings in queue while Wi-Fi is down, flush on reconnect

---

## Submission Checklist

- [ ] Amogh — push work to `week08/amogh/`
- [ ] Vaishnavi — push work to `week08/vaishnavi/`
- [ ] Sujay — push work to `week08/sujay/`
- [ ] Stack high-water mark logs saved/screenshotted
- [ ] Priority inversion: UART log showing the bug AND the fix
