# Embedded Systems Glossary

Alphabetical reference for all key terms used across the 12-week roadmap.

---

## A

**ACK (Acknowledgement)**
A signal sent by a receiver to confirm it received data. In I2C, the slave pulls SDA low during the 9th clock pulse to ACK each byte. In TCP, an ACK packet confirms receipt of a sequence number range.

**ADC (Analog-to-Digital Converter)**
Hardware that samples an analog voltage and converts it to a digital value. Resolution (bits) determines precision: 12-bit ADC on a 3.3V range gives 3.3V / 4096 = ~0.8 mV per step.

**AHB (Advanced High-performance Bus)**
The fast bus in STM32 SoCs (up to 168 MHz on F4). GPIO, DMA, and Flash are on AHB. See also: APB.

**APB (Advanced Peripheral Bus)**
Slower bus in STM32 (APB1: 42 MHz, APB2: 84 MHz). UART, SPI, I2C, and most timers are on APB. Peripheral clock must be enabled via RCC before use.

**AAPCS (ARM Architecture Procedure Call Standard)**
Calling convention for ARM: R0–R3 pass arguments and return values, R4–R11 are callee-saved, LR holds the return address.

**ARR (Auto-Reload Register)**
In STM32 timers, ARR sets the period. The counter resets to 0 (or ARR, counting down) when it reaches ARR. PWM frequency = timer_clock / ((PSC+1) × (ARR+1)).

---

## B

**Bare-metal**
Programming an MCU directly — no OS, no HAL abstraction. You write directly to hardware registers. Maximum control and performance, minimum overhead.

**Baud rate**
Symbol transitions per second on a serial line. For UART with no additional encoding, baud rate = bit rate. Common values: 9600, 115200, 921600.

**Big-endian**
Byte order where the most significant byte is stored at the lowest memory address. Network byte order is big-endian. Use `htonl()`/`ntohl()` to convert.

**Bitfield**
A struct member with a specified bit width: `uint32_t enable : 1`. Lets the compiler manage bit packing. Layout is implementation-defined — do not use for wire formats.

**Bootloader**
Code that runs before the main application. Responsibilities: hardware initialisation, optional firmware update, integrity check, then jump to application. Lives at the reset vector (0x08000000 on STM32).

**BSS Segment**
Memory region holding uninitialised (or zero-initialised) global and static variables. Zeroed by startup code before `main()`. Lives in SRAM on MCUs.

**BRR (Baud Rate Register)**
STM32 UART register. BRR = f_CK / baud_rate. Example: 16 MHz / 115200 = 139 (0x8B).

---

## C

**CAN Bus (Controller Area Network)**
A robust differential serial protocol for automotive and industrial use. Multi-master, message-based (not addressed). Arbitration by message ID — lower ID wins. Error detection built in.

**CCR (Capture/Compare Register)**
In STM32 timers, CCR sets the PWM duty cycle. Duty cycle = CCR / (ARR + 1) × 100%.

**CFSR (Configurable Fault Status Register)**
ARM Cortex-M register that records the cause of a HardFault, MemManage, BusFault, or UsageFault. Essential for debugging crash dumps. Read at `0xE000ED28`.

**CMSIS (Cortex Microcontroller Software Interface Standard)**
ARM's vendor-neutral hardware abstraction layer. Provides standardised names for Cortex-M registers and peripherals (e.g., `NVIC->ISER[0]`, `SysTick->CTRL`).

**CoAP (Constrained Application Protocol)**
A RESTful protocol designed for IoT devices — essentially HTTP over UDP. Supports GET, POST, PUT, DELETE, and an Observe mode for subscriptions. Secured with DTLS.

**Context Switch**
The RTOS saving the current task's CPU state (registers, PC, SP) and restoring a different task's state. On Cortex-M, triggered by the PendSV exception. Takes ~12 cycles on M4.

**CPOL/CPHA**
SPI configuration bits. CPOL: clock polarity (idle state). CPHA: clock phase (sample on first or second edge). Four combinations give SPI modes 0–3. Master and slave must match.

---

## D

**DAC (Digital-to-Analog Converter)**
Hardware that converts a digital value to an analog voltage. Used for audio output, signal generation, reference voltages.

**Data Segment**
Memory region holding initialised global and static variables. Initial values stored in flash, copied to SRAM by startup code at boot.

**Deadlock**
Two or more tasks each holding a resource the other needs, both blocked forever. Prevented by: always acquiring locks in the same order, using lock timeouts, or avoiding nested locking.

**DFU (Device Firmware Upgrade)**
A USB or UART-based protocol for updating firmware. STM32 has a built-in ROM bootloader that supports DFU over USB.

**DMA (Direct Memory Access)**
Hardware that transfers data between memory and peripherals without CPU involvement. Frees the CPU for other work. Supports circular mode (for continuous streaming) and half-transfer interrupts.

**DORA (Discover, Offer, Request, Acknowledge)**
The four-step DHCP process by which a device obtains an IP address from a DHCP server.

**DTB / DTS (Device Tree Blob / Source)**
Binary/source format describing hardware topology to the Linux kernel. `.dts` → compiled with `dtc` → `.dtb` loaded by bootloader. Replaces board-specific `#ifdef` in kernel code.

**DTLS (Datagram TLS)**
TLS adapted for UDP (used by CoAP). Handles packet reordering and loss that standard TLS assumes TCP handles.

**DWT (Data Watchpoint and Trace)**
ARM Cortex-M debug module. Contains `CYCCNT` — a 32-bit cycle counter running at CPU clock speed. Used to measure execution time with single-cycle precision.

---

## E

**ECDHE (Elliptic Curve Diffie-Hellman Ephemeral)**
Key exchange algorithm used in TLS 1.3. Generates a shared secret without transmitting it, using elliptic curve math. "Ephemeral" means a new key pair is generated per session — provides forward secrecy.

**eDRX (Extended Discontinuous Reception)**
Power-saving mode for NB-IoT/LTE-M where the device wakes periodically to check for downlink data, sleeping in between. Extends battery life significantly vs always-on.

**EXTI (External Interrupt)**
STM32 hardware that triggers an interrupt on a GPIO pin edge (rising, falling, or both). Up to 16 EXTI lines (EXTI0–15), one per pin number across all ports.

---

## F

**FCS (Frame Check Sequence)**
CRC at the end of an Ethernet frame, used to detect transmission errors.

**FIR Filter (Finite Impulse Response)**
Digital filter whose output depends only on current and past inputs (no feedback). Always stable, linear phase. Implemented as a weighted sum of the last N samples (convolution).

**FreeRTOS**
The most widely used open-source RTOS for microcontrollers. Provides tasks, queues, mutexes, semaphores, event groups, software timers, and multiple heap allocators.

**FSM (Finite State Machine)**
A design pattern where a system is always in one of a finite set of states, and transitions between states are triggered by events. Eliminates complex if-else chains.

---

## G

**GAP (Generic Access Profile)**
Bluetooth LE layer that defines device roles (Central, Peripheral, Broadcaster, Observer) and how devices discover and connect to each other.

**GATT (Generic Attribute Profile)**
Bluetooth LE layer that defines how data is structured and exchanged as Services and Characteristics after a connection is established.

**GDB (GNU Debugger)**
The standard debugger for C/C++ programs. Supports remote debugging of embedded targets via GDB server (OpenOCD, J-Link GDB server) over JTAG/SWD.

**GPIO (General Purpose Input/Output)**
MCU pins that can be configured as digital input, digital output, analog, or alternate function (connected to a peripheral). Controlled via MODER, ODR, IDR, PUPDR registers.

---

## H

**HAL (Hardware Abstraction Layer)**
A software layer that hides hardware details behind a standard API. STM32 HAL is ST's vendor library. Speeds up development but adds overhead and hides what's actually happening — avoid in bare-metal learning.

**HardFault**
An ARM Cortex-M exception triggered by illegal memory access, divide by zero, unaligned access (if enabled), or execution of an invalid instruction. Read CFSR/HFSR registers to diagnose.

**HMAC (Hash-based Message Authentication Code)**
A MAC that uses a cryptographic hash (e.g., SHA-256) combined with a secret key. Provides both integrity (detects tampering) and authentication (proves the sender has the key).

**HSE / HSI**
High-Speed External / Internal oscillator on STM32. HSE: crystal, more accurate (4–26 MHz). HSI: internal RC oscillator (16 MHz on F4), less accurate but needs no external component. PLL multiplies either to get the CPU clock.

---

## I

**I2C (Inter-Integrated Circuit)**
Two-wire serial protocol (SDA + SCL). Multi-master, multi-slave. Devices identified by 7-bit address. ACK/NACK on every byte. Max speed: 400 kHz (Fast mode), 1 MHz (Fast-plus), 3.4 MHz (High-speed).

**IIR Filter (Infinite Impulse Response)**
Digital filter that uses feedback (output fed back as input). More efficient than FIR for same frequency response, but can be unstable and has non-linear phase. Implemented as biquad sections.

**IP (Internet Protocol)**
Layer 3 protocol that routes packets between networks. IPv4: 32-bit addresses. IPv6: 128-bit. Key header fields: TTL (prevents infinite loops), protocol (TCP=6, UDP=17), source/dest address.

**ISR (Interrupt Service Routine)**
The function that runs when a hardware interrupt fires. Must be fast, non-blocking, and marked `volatile`-safe. On ARM, the entry point address is stored in the vector table.

**IWDG (Independent Watchdog)**
STM32 watchdog clocked by the independent LSI oscillator. Cannot be disabled once started. Must be kicked (`IWDG->KR = 0xAAAA`) within the configured timeout or the MCU resets.

---

## J

**JTAG (Joint Test Action Group)**
A 4/5-wire debug interface (TDI, TDO, TCK, TMS, optional TRST). Allows reading/writing CPU registers, memory, and flash. Slower than SWD but supports daisy-chaining multiple devices.

---

## L

**LR (Link Register)**
ARM register R14. Stores the return address when `BL` (Branch and Link) is called. ISRs push LR to the stack automatically; it contains an EXC_RETURN magic value indicating the interrupt context.

**Little-endian**
Byte order where the least significant byte is stored at the lowest memory address. Default on ARM Cortex-M and x86.

**LoRa / LoRaWAN**
LoRa: physical layer — spread-spectrum RF modulation. LoRaWAN: MAC protocol on top. Long-range (km), very low power, low data rate. Used for battery-powered IoT sensors.

**LWT (Last Will and Testament)**
An MQTT message that the broker automatically publishes if a client disconnects unexpectedly (without a clean disconnect). Used to notify other subscribers that a device went offline.

**lwIP (Lightweight IP)**
Open-source TCP/IP stack designed for embedded systems (as little as 40 KB RAM). Includes TCP, UDP, DNS, DHCP, and HTTP. Used on ESP32 and integrated with FreeRTOS.

---

## M

**mbedTLS**
Portable, open-source TLS/DTLS library optimised for embedded systems. Configurable to reduce RAM/flash footprint. Used for TLS on MCUs in this roadmap.

**MODER (Mode Register)**
STM32 GPIO register. 2 bits per pin: 00=input, 01=output, 10=alternate function, 11=analog.

**Mosquitto**
A lightweight, open-source MQTT broker by Eclipse. Widely used for local IoT testing. Supports TLS on port 8883.

**MQTT (Message Queuing Telemetry Transport)**
Lightweight publish-subscribe protocol for IoT. Runs over TCP. Devices publish to topics; subscribers receive messages from topics they've subscribed to. Broker routes messages.

**MTU (Maximum Transmission Unit)**
Maximum payload size for a protocol layer. Ethernet MTU: 1500 bytes. If an IP packet is larger, it is fragmented. Embedded devices should avoid fragmentation — keep UDP payloads under 1472 bytes.

**mTLS (Mutual TLS)**
TLS where both client and server present certificates. The server verifies the client's identity (not just the client verifying the server). Used in IoT to authenticate individual devices.

---

## N

**NACK (Not Acknowledge)**
In I2C, the slave leaves SDA high during the ACK clock pulse to indicate: it didn't recognise its address, it can't accept more data, or a communication error occurred. The master must send STOP.

**NB-IoT (Narrowband IoT)**
Cellular IoT standard for low-power, wide-area devices. Uses a narrow 200 kHz channel within LTE spectrum. Very low power, deep indoor coverage, low cost module.

**NVIC (Nested Vectored Interrupt Controller)**
ARM Cortex-M hardware that handles interrupt routing and priority. Features: configurable priority levels, nested interrupts, tail-chaining, late-arrival optimisation. Configured via `NVIC->ISER`, `NVIC->IP`.

---

## O

**OBD-II (On-Board Diagnostics)**
Standardised vehicle diagnostic protocol. PIDs (Parameter IDs) define requests for RPM, speed, temperature, etc. Runs over CAN bus or K-line.

**ODR (Output Data Register)**
STM32 GPIO register. Writing a bit HIGH drives the pin HIGH (push-pull mode). Reading ODR shows the last written value, not the actual pin state — use IDR for that.

**OpenOCD (Open On-Chip Debugger)**
Open-source tool that bridges a hardware debug probe (ST-Link, J-Link, CMSIS-DAP) to GDB. Supports JTAG and SWD. Can flash firmware and provide a GDB server.

**OTA (Over-the-Air update)**
Updating firmware wirelessly (Wi-Fi, cellular, BLE). On ESP32: dual-bank flash — new firmware written to the inactive partition, verified, then set as the boot partition.

---

## P

**PC (Program Counter)**
ARM register R15. Always holds the address of the next instruction to fetch. Manipulating PC directly causes a branch.

**PCE (Parity Control Enable)**
UART configuration bit. When set, the UART adds/checks a parity bit for error detection.

**PendSV**
ARM Cortex-M exception used by FreeRTOS to perform context switches. Set to the lowest priority so the switch only happens when no other interrupt is pending.

**PLL (Phase-Locked Loop)**
Hardware that multiplies a reference clock frequency. STM32F4: PLL can multiply HSI/HSE up to 168 MHz for the CPU clock.

**PSM (Power Saving Mode)**
NB-IoT/LTE-M feature where the device enters deep sleep for a negotiated period (minutes to hours), completely unreachable, then wakes to send/receive data.

**PUPDR (Pull-Up/Pull-Down Register)**
STM32 GPIO register. 2 bits per pin: 00=no pull, 01=pull-up, 10=pull-down, 11=reserved.

**PWM (Pulse Width Modulation)**
A digital signal with a configurable duty cycle used to encode an analog value. Used for motor control, LED brightness, servo positioning. Generated by hardware timers.

---

## Q

**QoS (Quality of Service)**
In MQTT: delivery guarantee level (0=at most once, 1=at least once, 2=exactly once). In networking: traffic prioritisation to guarantee bandwidth/latency for critical flows.

**Queue (FreeRTOS)**
Thread-safe FIFO buffer for passing data between tasks or from ISR to task. `xQueueSend` blocks if full; `xQueueReceive` blocks if empty. Has ISR-safe variants (`FromISR`).

---

## R

**RCC (Reset and Clock Control)**
STM32 peripheral that controls all clocks and resets. Must enable a peripheral's clock here before accessing any of its registers, e.g., `RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN`.

**RDP (Read Protection)**
STM32 flash protection levels. Level 0: unprotected. Level 1: JTAG readout disabled (debugger can't dump flash). Level 2: completely locked (irreversible).

**RTOS (Real-Time Operating System)**
An OS that provides deterministic timing guarantees. Preemptive RTOS: scheduler can interrupt any task to run a higher-priority task. Used when multiple tasks must run with defined timing.

---

## S

**Semaphore**
A synchronisation primitive. Binary semaphore: signalling between tasks (or ISR → task). Counting semaphore: tracks a pool of N resources. Unlike a mutex, has no ownership.

**SPI (Serial Peripheral Interface)**
Four-wire synchronous serial protocol (MOSI, MISO, SCK, CS). Full-duplex. Master drives clock. Multiple slaves using separate CS pins. Much faster than I2C — 10s of Mbps.

**SP (Stack Pointer)**
ARM register R13. Points to the top of the current stack. The MCU has two stack pointers: MSP (Main Stack Pointer, used by interrupts) and PSP (Process Stack Pointer, used by RTOS tasks).

**STRIDE**
Threat modelling framework: Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege.

**SWD (Serial Wire Debug)**
2-wire debug interface for ARM Cortex-M (SWDIO + SWDCLK). Faster and lower pin count than JTAG. Used by ST-Link and most ARM debuggers.

**sysfs**
Linux virtual filesystem at `/sys/`. Exposes kernel data structures — including device driver state — as files. Write to `/sys/class/gpio/gpio17/value` to toggle a GPIO from a shell script.

---

## T

**TCP (Transmission Control Protocol)**
Connection-oriented, reliable, ordered byte stream protocol. Establishes connections with 3-way handshake. Handles retransmission, congestion control, and flow control.

**Text Segment**
Memory region holding compiled machine code. Read-only. Lives in flash on MCUs.

**TLS (Transport Layer Security)**
Cryptographic protocol that provides authentication, integrity, and confidentiality over TCP. TLS 1.3 (current): 1-RTT handshake, uses ECDHE for key exchange, AES-GCM for encryption.

**TTL (Time to Live)**
IP header field decremented by each router. When TTL reaches 0, the packet is discarded and an ICMP "Time Exceeded" message is sent back. Prevents infinite loops. Used by `traceroute`.

---

## U

**UART (Universal Asynchronous Receiver/Transmitter)**
Asynchronous serial protocol. No shared clock — both sides must be configured to the same baud rate. Frame: start bit, 8 data bits, optional parity, 1–2 stop bits.

**UDP (User Datagram Protocol)**
Connectionless, unreliable, unordered datagram protocol. No handshake, no retransmission, low overhead. Used for DNS, DHCP, streaming, and embedded telemetry.

**U-Boot**
Universal Boot Loader. The standard bootloader for embedded Linux. Initialises DRAM, loads kernel + DTB from SD/eMMC/network (TFTP), and boots.

---

## V

**Vector Table**
Array of function pointers at the start of flash (0x08000000 on STM32). Index 0: initial stack pointer. Index 1: reset handler. Subsequent entries: exception and interrupt handlers.

**volatile**
C keyword that prevents the compiler from caching or optimising away reads/writes to a variable. Required for memory-mapped registers, ISR-shared variables, and variables shared between tasks.

---

## W

**Watchdog Timer**
Hardware timer that resets the MCU if not kicked (refreshed) within a timeout. Detects software hangs. IWDG: independent clock, simpler. WWDG: window-based, detects both too-fast and too-slow kicks.

**WWDG (Window Watchdog)**
STM32 watchdog that must be kicked within a specific time window — not too early and not too late. Detects software that kicks the watchdog too frequently (stuck in a tight loop).

---

## Z

**Zephyr**
An open-source RTOS by the Linux Foundation. Strong safety/security focus, extensive hardware support, built-in networking stack. Uses west build system and Kconfig for configuration.
