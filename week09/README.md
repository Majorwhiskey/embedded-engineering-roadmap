# Week 9 — Embedded Linux: Kernel, Drivers, Device Tree & IPC

**Month 3 · Advanced & Capstone** | Start: July 2, 2026

---

## Topics

🟡 Embedded Linux stack: U-Boot → kernel → device tree → rootfs → init system  
🟡 U-Boot: boot commands, environment variables, TFTP network boot, boot scripts  
🟡 Device tree: syntax (nodes, properties, phandles), pinmux, overlays, compiling `.dts` → `.dtb`  
🟡 Linux kernel modules: `module_init`/`module_exit`, out-of-tree Makefile, `insmod`/`rmmod`/`modprobe`  
🟡 Character device driver: `cdev`, `file_operations` (open/read/write/ioctl/release), `register_chrdev_region`, udev rules  
🩷 sysfs and procfs: expose driver state to userspace, read/write attributes  
🩷 Linux IPC: pipes, FIFOs, UNIX domain sockets, shared memory (`mmap` + `shm_open`), signals, message queues  
🩷 Pthreads: `pthread_create`, `pthread_join`, `pthread_mutex_t`, `pthread_cond_t`, thread pools  
🩷 Buildroot: board config, package selection, overlay filesystem, bootable image  
⬜ Yocto Project: layers, recipes, bitbake, `IMAGE_INSTALL`  
⬜ Qt framework: QML basics for embedded touchscreen GUI  

---

## Projects

### 1. Custom Linux Char Driver
Kernel module controlling GPIO via `/dev/mydevice`. Userspace app writes `"1"`/`"0"` to toggle LED.
- Handle concurrent access
- Test with `cat` and `echo` from shell

### 2. Multi-Process Sensor Server
- Process A reads `/dev/i2c-1` (BMP280), sends data to Process B via UNIX socket
- Process B serves latest reading over TCP socket to any client
- Use `select()` for multiplexed I/O

### 3. Buildroot Minimal Image
Build a bootable Linux image for Raspberry Pi with Buildroot.
- Include: dropbear SSH server, your sensor app as an `init.d` service, custom `/etc/motd`
- Boot from SD card and SSH in

---

## Submission Checklist

- [ ] Amogh — push work to `week09/amogh/`
- [ ] Vaishnavi — push work to `week09/vaishnavi/`
- [ ] Sujay — push work to `week09/sujay/`
- [ ] Kernel module compiles against target kernel version
- [ ] Buildroot `.config` committed so image is reproducible
