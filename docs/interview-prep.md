# Embedded Systems Interview Preparation

Questions organised by topic. Each section maps to a month of the roadmap.
Use this after completing the full 3 months — or as a checkpoint to test yourself at the end of each week.

**Difficulty key:** 🟢 Junior · 🟡 Mid · 🔴 Senior

---

## C Programming & Memory

**🟢 What is the difference between a pointer and a reference?**
C has no references — only pointers. A pointer is a variable that stores a memory address. You explicitly dereference it with `*`. References (in C++) are aliases — they cannot be null and cannot be reseated.

**🟢 What is a dangling pointer? How do you avoid it?**
A pointer that points to memory that has been freed or is out of scope. Avoid by: setting pointer to NULL after `free()`, not returning pointers to local variables, using address sanitizer (ASAN) during development.

**🟢 What is the difference between `malloc` and `calloc`?**
`malloc(n)` allocates n bytes — contents are uninitialised (garbage). `calloc(count, size)` allocates `count × size` bytes and zeroes them. `calloc` is slightly slower due to the zero-fill. On embedded, prefer static allocation over both.

**🟡 Why should you avoid dynamic memory allocation on bare-metal MCUs?**
Three reasons: (1) non-deterministic timing — the allocator scans a free list of unpredictable length; (2) heap fragmentation — many small alloc/free cycles leave unusable gaps, eventually causing `malloc` to return NULL; (3) no OS to clean up leaks. Use static pools or stack allocation instead.

**🟡 What does `volatile` do, and when must you use it?**
Tells the compiler this variable can change outside the program's control — prevents it from caching the value in a register or optimising away reads/writes. Required for: (1) memory-mapped peripheral registers, (2) variables modified in an ISR, (3) variables shared between tasks (though `volatile` alone doesn't guarantee atomicity — use mutexes too).

**🟡 Explain the difference between `const int *p`, `int * const p`, and `const int * const p`.**
- `const int *p` — pointer to const int: you can change where p points, but not `*p`
- `int * const p` — const pointer to int: you cannot change where p points, but can change `*p`
- `const int * const p` — const pointer to const int: neither the pointer nor the value can change. Peripheral registers are typically `volatile uint32_t * const`.

**🟡 What is struct padding? How do you eliminate it?**
The compiler inserts padding bytes so each field starts at its natural alignment (typically its own size). Eliminating it: reorder fields largest-first, or use `__attribute__((packed))`. Packed structs risk unaligned access faults on MCUs — use only for protocol buffers and never for normal data.

**🟡 What are the four memory segments of a C program? What lives in each?**
- **Text** — compiled machine code (read-only, in flash on MCUs)
- **Data** — initialised global and static variables (values copied from flash to RAM at startup)
- **BSS** — uninitialised global and static variables (zeroed by startup code before `main()`)
- **Stack** — local variables, function parameters, return addresses (grows down)
- **Heap** — dynamically allocated memory (grows up, managed by programmer)

**🔴 What is a memory barrier, and when do you need one on ARM?**
A memory barrier is an instruction that prevents the CPU or compiler from reordering memory operations across it. ARM provides `DMB` (data memory barrier), `DSB` (data sync barrier), and `ISB` (instruction sync barrier). Needed when: configuring a peripheral register that must be written in a specific order, after enabling a clock before accessing the peripheral, and in lock-free data structures shared between an ISR and main code.

---

## Bit Manipulation

**🟢 How do you set, clear, toggle, and read a specific bit in a register?**
```c
reg |=  (1 << n);    // set bit n
reg &= ~(1 << n);    // clear bit n
reg ^=  (1 << n);    // toggle bit n
if (reg & (1 << n))  // read bit n
```

**🟢 What is a bitmask? Give an example.**
A constant that isolates specific bits. Example: `0x0F` (`0b00001111`) extracts the lower nibble: `value & 0x0F`. Used to read/write multi-bit fields in registers without disturbing adjacent bits.

**🟡 How do you extract a multi-bit field from a register?**
```c
#define FIELD_POS  4
#define FIELD_MASK (0x7 << FIELD_POS)   // 3-bit field at bits [6:4]

uint8_t field = (reg & FIELD_MASK) >> FIELD_POS;
```
Read-modify-write to update: `reg = (reg & ~FIELD_MASK) | ((new_val << FIELD_POS) & FIELD_MASK);`

**🟡 What is the difference between logical and arithmetic right shift?**
Logical shift fills vacated bits with 0 — used for unsigned values. Arithmetic shift fills with the sign bit — used for signed values to preserve the sign. In C, right-shifting a signed integer is implementation-defined (though GCC uses arithmetic shift on ARM).

**🟡 How do you check if a number is a power of 2?**
```c
bool is_power_of_two(uint32_t n) { return n && !(n & (n - 1)); }
```
`n - 1` flips all bits below the single set bit. ANDing with `n` gives 0 only if there's exactly one set bit.

**🔴 What is the difference between `uint8_t`, `uint_fast8_t`, and `uint_least8_t`?**
- `uint8_t` — exactly 8 bits, guaranteed (may not exist on platforms without native 8-bit)
- `uint_fast8_t` — fastest unsigned type of at least 8 bits (may be 32 bits on ARM for speed)
- `uint_least8_t` — smallest unsigned type of at least 8 bits
For MCU registers, always use `uint8_t` / `uint32_t`. For loop counters where speed matters, `uint_fast8_t`.

---

## Microcontroller Peripherals

**🟢 What is GPIO and what modes can it be configured in?**
General Purpose Input/Output — pins that can be software-controlled. Modes: input (floating, pull-up, pull-down), output (push-pull, open-drain), alternate function (connected to a peripheral like UART/SPI), analog.

**🟢 What is PWM and how is it generated?**
Pulse Width Modulation — a digital square wave where the duty cycle (% of time HIGH) encodes an analog value. Generated by a hardware timer: the counter resets at the period, and a compare register defines when the output flips. Duty cycle = CCR / ARR × 100%.

**🟡 What is the difference between polling and interrupt-driven I/O?**
Polling: CPU continuously checks a status flag — wastes cycles, but simple and deterministic. Interrupt: peripheral signals the CPU only when data is ready — CPU can do other work in between. Interrupts are preferred for event-driven embedded systems; polling is used when timing is critical and the wait is very short.

**🟡 What is DMA, and why use it instead of CPU-driven transfers?**
Direct Memory Access — a hardware block that moves data between peripherals and memory independently of the CPU. Benefits: CPU is free to do other work during the transfer, or can sleep to save power; higher sustained throughput; less jitter. Used for: UART/SPI receive buffers, ADC continuous sampling, audio streaming.

**🟡 What are the rules for writing good ISR (Interrupt Service Routine) code?**
1. Keep it short and fast — no blocking calls, no `printf`, no `malloc`
2. No `delay()` or busy-wait loops
3. Use `volatile` for all variables shared with main code
4. Set a flag or push to a queue; do the real work in main loop or a task
5. Be careful with reentrancy — if the ISR can be interrupted by itself, protect shared state
6. Clear the interrupt pending flag at the start (some peripherals require this)

**🟡 What is the NVIC?**
Nested Vectored Interrupt Controller — ARM Cortex-M hardware that manages interrupts. Features: configurable priority levels (0–255, lower number = higher priority), nested interrupts (higher-priority ISR can interrupt a lower-priority one), tail-chaining (back-to-back ISR execution without full state save/restore), late arrival (pending higher-priority ISR pre-empts current ISR before it starts).

**🔴 What is priority inversion and how does FreeRTOS solve it?**
A high-priority task is blocked waiting for a mutex held by a low-priority task. If a medium-priority task is ready, it preempts the low-priority task — the high-priority task is effectively blocked by the medium-priority one. FreeRTOS solves this with **priority inheritance**: the low-priority task temporarily inherits the priority of the highest-priority task waiting for its mutex, so it runs and releases the mutex without being preempted.

**🔴 Explain the difference between IWDG and WWDG on STM32.**
IWDG (Independent Watchdog): clocked by LSI (independent of main clock), cannot be disabled once started, must be kicked within a configurable timeout (1 ms–26 s). WWDG (Window Watchdog): clocked from APB1 (stops if clock stops), must be kicked within a **window** — not too early and not too late. WWDG detects software that kicks the watchdog too frequently (stuck in a tight loop), while IWDG only detects software that never kicks it.

---

## Serial Protocols

**🟢 What is the difference between UART, I2C, and SPI?**

| | UART | I2C | SPI |
|--|------|-----|-----|
| Wires | 2 (TX, RX) | 2 (SDA, SCL) | 4 (MOSI, MISO, SCK, CS) |
| Topology | Point-to-point | Multi-master, multi-slave bus | Single master, multi-slave |
| Speed | Up to ~5 Mbps | 100k/400k/1M/3.4M bps | 10s of Mbps |
| Addressing | None (dedicated wires) | 7-bit or 10-bit address | CS pin per slave |
| Duplex | Full | Half | Full |
| Clock | Each side has its own | Shared (master drives) | Shared (master drives) |

**🟢 What is baud rate?**
The number of symbol changes (transitions) per second on the line. For UART with no encoding overhead, baud rate equals bit rate. Both sides must agree on baud rate — mismatch causes framing errors.

**🟡 What is I2C clock stretching?**
A slave device can hold SCL LOW to pause the transaction while it prepares data. The master must detect this and wait. Some masters (including some STM32 silicon revisions) have bugs with clock stretching — check the errata.

**🟡 What is SPI CPOL/CPHA?**
Two bits that define the clock polarity and phase:
- CPOL=0: clock idles LOW; CPOL=1: clock idles HIGH
- CPHA=0: data sampled on first clock edge; CPHA=1: data sampled on second edge
This gives 4 modes (0–3). Master and slave must use the same mode. Check the sensor datasheet's timing diagram to determine which mode it requires.

**🟡 What happens when no device acknowledges an I2C address?**
The master sends the 7-bit address + R/W bit. If no slave pulls SDA low during the ACK clock pulse, the master sees a NACK. The master must then send a STOP condition and handle the error. Common causes: wrong address, device not powered, address configured incorrectly (ADD0 pin).

**🔴 How would you debug a failing SPI transaction?**
1. Check CPOL/CPHA matches the datasheet
2. Check CS polarity (active low vs active high) and timing (CS must go low before first clock)
3. Check clock frequency (does not exceed slave's maximum)
4. Capture with a logic analyser — decode the bytes and compare against expected register values
5. Check MISO line isn't floating when not driven (add pull-up if needed)
6. Verify power and decoupling capacitors on the slave

---

## Networking & Protocols

**🟢 What are the 7 layers of the OSI model?**
7 Application, 6 Presentation, 5 Session, 4 Transport (TCP/UDP), 3 Network (IP), 2 Data Link (Ethernet/MAC), 1 Physical (wires, RF). In embedded: you mostly care about layers 3–4 (IP, TCP/UDP) and layer 7 (MQTT, CoAP, HTTP).

**🟢 What is the difference between TCP and UDP?**
TCP: connection-oriented, reliable (retransmits lost packets), ordered, congestion-controlled, higher overhead. UDP: connectionless, unreliable, unordered, no congestion control, low overhead. Use TCP for commands and configuration; UDP for telemetry and streaming where a stale packet is worse than a missed one.

**🟡 Describe the TCP 3-way handshake.**
1. Client → Server: SYN (seq=x)
2. Server → Client: SYN-ACK (seq=y, ack=x+1)
3. Client → Server: ACK (ack=y+1)
Connection is established. SYN-ACK simultaneously acknowledges the client's SYN and sends the server's own sequence number.

**🟡 What is MQTT QoS and when do you use each level?**
- **QoS 0** (at most once): fire and forget, no acknowledgement. Use for high-frequency telemetry where losing a reading is fine.
- **QoS 1** (at least once): broker acknowledges with PUBACK; sender retransmits if no ACK. Message may arrive more than once — receiver must be idempotent. Use for sensor data where delivery matters.
- **QoS 2** (exactly once): 4-step handshake (PUBLISH → PUBREC → PUBREL → PUBCOMP). Guaranteed exactly once. Use for commands (turn actuator ON/OFF) where duplicates cause problems.

**🟡 What is a socket and what is the difference between a TCP and UDP socket?**
A socket is an OS abstraction for a network endpoint: (IP address, port, protocol). TCP socket: after `connect()` / `accept()`, provides a reliable byte stream — `send()`/`recv()` work like a pipe. UDP socket: no connection, each `sendto()`/`recvfrom()` is an independent datagram with explicit destination address.

**🟡 What does the `select()` syscall do?**
Monitors multiple file descriptors simultaneously and blocks until one or more become ready (readable, writable, or error). Returns a bitmask of ready FDs. Allows a single-threaded server to handle multiple clients without threads. Limitation: only scales to ~1024 FDs. For larger scale, use `poll()` or `epoll()`.

**🔴 What is TLS and what does the handshake establish?**
Transport Layer Security — a protocol that provides authentication, integrity, and confidentiality over TCP. The handshake (TLS 1.3): (1) client sends supported cipher suites and a random nonce, (2) server sends its certificate and chosen cipher suite, (3) both sides compute a shared secret using key exchange (ECDHE), (4) both derive session keys from the shared secret. After the handshake, all data is encrypted with a symmetric cipher (AES-GCM) and authenticated with HMAC. Certificate chains allow the client to verify the server's identity against a trusted CA.

**🔴 What is the difference between TLS and mTLS?**
In standard TLS, only the server authenticates itself (via certificate). In mutual TLS (mTLS), both sides present certificates. The server verifies the client's certificate against its CA. Used in embedded IoT for device authentication: each device has a unique client certificate, so the broker can reject unauthorised devices even if they know the broker's address.

---

## RTOS

**🟢 What is an RTOS and why use it instead of a bare-metal superloop?**
A Real-Time Operating System provides: (1) a scheduler that runs tasks with defined priorities and timing guarantees; (2) synchronisation primitives (mutexes, semaphores, queues); (3) deterministic response to events. A bare-metal superloop (`while(1){ check_a(); check_b(); }`) cannot guarantee when any function runs — adding more functions makes all of them slower. RTOS tasks run independently, and higher-priority tasks always preempt lower ones.

**🟢 What are the FreeRTOS task states?**
- **Running** — currently executing on the CPU
- **Ready** — able to run but a higher-priority task is running
- **Blocked** — waiting for an event (queue message, semaphore, timeout)
- **Suspended** — deliberately paused with `vTaskSuspend()`, not scheduled at all

**🟡 What is the difference between a mutex and a binary semaphore in FreeRTOS?**
Both block and unblock tasks. Key difference: a **mutex** has ownership — only the task that took it can give it, and it supports **priority inheritance** (prevents priority inversion). A **binary semaphore** has no ownership — any task can give it, making it suitable for signalling between tasks or from an ISR. Use mutex to protect a shared resource; use binary semaphore to signal an event.

**🟡 What is a queue in FreeRTOS and why is it preferred over a global variable?**
A queue is a thread-safe FIFO buffer. It blocks the sender if full and blocks the receiver if empty, with configurable timeouts. Preferred over globals because: (1) built-in synchronisation — no manual locking; (2) decouples producer and consumer — they don't need to run at the same rate; (3) ISR-safe variants exist (`xQueueSendFromISR`).

**🟡 What is `vTaskDelay` vs `vTaskDelayUntil`?**
`vTaskDelay(100)` blocks for 100 ticks from the moment it's called — if the task took 20 ticks to execute, the next run starts 120 ticks after the last start. `vTaskDelayUntil(&last_wake, 100)` blocks until an absolute tick count — maintains a fixed period regardless of execution time. Use `vTaskDelayUntil` for periodic sensor reading where jitter matters.

**🔴 What is stack overflow detection in FreeRTOS?**
FreeRTOS fills each task's stack with a canary pattern (`0xA5A5A5A5`) at creation. `configCHECK_FOR_STACK_OVERFLOW=1` checks if the stack pointer went below the bottom at each context switch. `=2` checks if the canary bytes at the bottom are still intact — catches slower overflow. Production code should also call `uxTaskGetStackHighWaterMark()` periodically to log the minimum free stack space seen.

---

## Embedded Linux

**🟢 What is the difference between a kernel module and a user-space program?**
A kernel module runs in kernel space (ring 0) — has direct access to hardware, can crash the entire system if buggy. A user-space program runs in ring 3 — protected memory, crashes only that process. Device drivers are kernel modules; applications that talk to hardware through `/dev/` or sysfs are user-space.

**🟡 What is a device tree and why does Linux use it?**
A device tree is a data structure (`.dts` file compiled to `.dtb`) that describes the hardware to the kernel — which peripherals exist, at what addresses, with what interrupt numbers. Decouples the kernel from board-specific knowledge. The kernel reads the DTB at boot and instantiates the right drivers. Alternative to platform-specific `#ifdef` code in the kernel.

**🟡 What is the difference between a character device and a block device?**
Character device: data is accessed as a stream of bytes (read/write sequentially) — terminal, UART, GPIO. Block device: data is accessed in fixed-size blocks (random access) — SD card, eMMC, NAND flash. Character devices appear as `/dev/tty*`, `/dev/i2c-*`; block devices as `/dev/mmcblk*`, `/dev/sda`.

**🟡 What is `mmap()` and when would you use it in embedded Linux?**
`mmap()` maps a file or device into the process's virtual address space. In embedded: (1) map `/dev/mem` to access physical addresses directly from user space (for quick prototyping); (2) map a shared memory region between processes; (3) map a large file for zero-copy access. Faster than `read()`/`write()` for large data because it avoids a kernel-to-user copy.

**🔴 What is the difference between `spinlock` and `mutex` in the Linux kernel?**
A spinlock busy-waits (spins) in a tight loop until the lock is free — it never sleeps. A mutex can sleep (puts the task in a wait queue). Use spinlock: in interrupt context (cannot sleep), for very short critical sections. Use mutex: in process context for longer critical sections. Taking a spinlock disables kernel preemption on that CPU; taking a mutex allows other tasks to run.

---

## Security & TLS

**🟢 What is the difference between symmetric and asymmetric encryption?**
Symmetric: same key for encrypt and decrypt (AES, ChaCha20) — fast, used for bulk data. Asymmetric: key pair (public encrypts, private decrypts; or private signs, public verifies) — slow, used for key exchange and authentication (RSA, ECDSA). TLS uses asymmetric crypto to establish a shared secret, then switches to symmetric for the data transfer.

**🟡 What is a buffer overflow and how do you prevent it in C?**
Writing beyond the end of a buffer — overwrites adjacent memory, potentially including the return address, enabling code execution. Prevention: (1) always use bounded functions (`strncpy` not `strcpy`, `snprintf` not `sprintf`); (2) check array index before access; (3) use stack canaries (`-fstack-protector`); (4) compile with ASAN for testing; (5) never trust input length from external sources.

**🟡 What is secure boot?**
A chain of trust from power-on to application code. Each stage verifies the digital signature of the next before executing it. On STM32: RDP (Read Protection) levels prevent JTAG readout of flash. On ESP32: flash encryption + secure boot verify a signed firmware image using a key burned into eFuse. If verification fails, the device refuses to boot.

**🔴 Explain the difference between code signing and encryption in firmware security.**
Code signing: the firmware binary is hashed (SHA-256) and the hash is signed with a private key. The bootloader verifies the signature using the corresponding public key — guarantees authenticity (who made it) and integrity (not tampered). Encryption: the firmware binary is encrypted — guarantees confidentiality (cannot be read back from flash). A secure system uses both: sign to verify authenticity, encrypt to protect IP.

---

## General / System Design

**🟢 What is the difference between big-endian and little-endian?**
The byte order of multi-byte values in memory. Little-endian: least significant byte at the lowest address (ARM, x86). Big-endian: most significant byte at the lowest address (network byte order, some old MCUs). Use `htonl()`/`ntohl()` when sending integers over a network.

**🟡 How would you design a system to handle a sensor that occasionally returns corrupt data?**
Multiple strategies, usually combined: (1) CRC or checksum on each reading — discard if mismatch; (2) validate range — if temperature reads -999°C, discard; (3) exponential moving average or median filter — outliers have minimal effect; (4) triple redundancy — take 3 readings, use the median; (5) track consecutive error count — if too many, flag sensor as failed and alert.

**🟡 What is a circular buffer and why is it used in embedded systems?**
A fixed-size FIFO where the write and read pointers wrap around. Used for: UART receive buffers (ISR writes, main reads without blocking), DMA ping-pong buffers, audio pipelines. Advantages: O(1) push and pop, no dynamic allocation, cache-friendly, can be made lock-free with atomic operations for single-producer single-consumer.

**🔴 How do you measure the execution time of a function on a Cortex-M MCU?**
Use the DWT (Data Watchpoint and Trace) cycle counter:
```c
CoreDebug->DEMCR |= CoreDebug_DEMCR_TRCENA_Msk;  // enable DWT
DWT->CYCCNT = 0;
DWT->CTRL  |= DWT_CTRL_CYCCNTENA_Msk;

uint32_t start = DWT->CYCCNT;
my_function();
uint32_t cycles = DWT->CYCCNT - start;
float time_us = (float)cycles / SystemCoreClock * 1e6f;
```
Accurate to 1 CPU cycle. No printf overhead. Works in production code.

**🔴 You have 16 KB of RAM left. An RTOS task needs a 4 KB stack and a 4 KB buffer. How do you decide whether to allocate them statically or dynamically?**
Static allocation: define as global arrays — deterministic, no fragmentation risk, no failure at runtime. Dynamic: `pvPortMalloc()` — more flexible, but risks fragmentation over time and `malloc` can return NULL after hours of uptime. In a safety-critical or long-running system, always use static allocation. FreeRTOS supports fully static tasks (`xTaskCreateStatic`) — prefer this in production embedded firmware.
