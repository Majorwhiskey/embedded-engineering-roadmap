# Week 2 — Electronics, Data Structures & Algorithms

**Month 1 · Foundations** | Start: May 14, 2026

---

## Topics

🟡 Principles of electric circuits: Ohm's law, KVL, KCL, Thevenin/Norton  
🟡 Electronics fundamentals: resistors, capacitors, diodes, BJT/MOSFET, op-amp basics  
🟡 Test equipment: multimeter (V/I/R/continuity), oscilloscope basics  
🟡 Algorithms: sorting (bubble, merge, quick), binary search, Big O  
🟡 Data structures: queues, circular buffers, hash maps, trees — with embedded use cases  
🩷 Design patterns: state machines, observer pattern, command pattern  
⬜ Python basics for scripting and test automation  

---

## Projects

### 1. Voltage Divider on Breadboard
Design a 5V→3.3V divider, calculate resistor values, build it, measure with multimeter.  
- Simulate in [Falstad](https://falstad.com/circuit/) and verify against measured values  

### 2. Circular Buffer in C
Lock-free circular buffer (used in every UART/DMA driver).  
- Functions: `init`, `push`, `pop`, `is_full`, `is_empty`, `peek`  
- Stress-test with a producer-consumer simulation  

### 3. State Machine Traffic Light
Traffic light controller as an explicit FSM in C: states enum, events enum, transition table, action functions.  
- No if-else chains — pure transition table  

---

## Hardware Needed by This Week

- Breadboard
- Resistor kit
- Capacitor kit
- LEDs
- Multimeter
- Jumper wires

---

## Submission Checklist

- [ ] Amogh — push work to `week02/amogh/`
- [ ] Vaishnavi — push work to `week02/vaishnavi/`
- [ ] Sujay — push work to `week02/sujay/`
- [ ] Falstad screenshot saved to your folder
- [ ] FSM has no if-else chains
