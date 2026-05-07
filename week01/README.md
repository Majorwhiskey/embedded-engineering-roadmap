# Week 1 — C Programming & Computer Architecture

**Month 1 · Foundations** | Start: May 7, 2026

---

## Topics

🟡 C fundamentals: pointers, arrays, structs, unions, enums, typedefs  
🟡 Memory model: stack, heap, BSS, text segment, memory-mapped I/O  
🟡 Bit manipulation: masks, shifts, bitfields, set/clear/toggle  
🟡 Computer architecture: CPU pipeline, buses, memory hierarchy, von Neumann vs Harvard  
🟡 Digital design: logic gates, Boolean algebra, flip-flops, combinational vs sequential  
🟡 Git basics: init, clone, add, commit, branch, merge, push, pull, PR workflow  
⬜ ARM assembly basics: registers, load/store, branching  

---

## Projects

### 1. C Memory Playground
Implement a **linked list**, **stack**, and **queue** in plain C using `malloc`/`free`.  
- Run Valgrind — confirm zero leaks  
- Use GDB to inspect memory addresses and pointer values  

### 2. Bit Manipulation Library
Header-only C library with: `set_bit`, `clear_bit`, `toggle_bit`, `count_set_bits`, `reverse_bits`, `rotate_left`, `rotate_right`.  
- Unit test every function with `assert()`  

### 3. Logic Gate Simulator
Simulate AND, OR, XOR, NAND, NOR, XNOR, and a D flip-flop using only C bit operations.  
- Print full truth tables to terminal  

---

## Tools to Install

```bash
sudo apt install build-essential valgrind gdb git
```
VS Code extensions: **C/C++**, **GitLens**

---

## Submission Checklist

- [ ] Amogh — push work to `week01/amogh/`
- [ ] Vaishnavi — push work to `week01/vaishnavi/`
- [ ] Sujay — push work to `week01/sujay/`
- [ ] All three projects compiling with zero warnings (`-Wall -Wextra`)
- [ ] Valgrind clean on memory playground
