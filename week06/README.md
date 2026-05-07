# Week 6 — Networking Fundamentals: OSI Model, TCP/IP, Packets, SSH

**Month 2 · Systems & Networking** | Start: June 11, 2026

---

## Topics

🟡 OSI model: all 7 layers, real protocols at each layer  
🟡 Ethernet frame: preamble, MAC addresses, EtherType, payload, FCS, MTU (1500 bytes)  
🟡 IP addressing: IPv4, subnets, CIDR, ARP, ICMP (ping/traceroute)  
🟡 Packets deep dive: IP header fields (TTL, protocol, checksum, flags), fragmentation  
🟡 TCP: 3-way handshake, sequence numbers, ACK, sliding window, retransmission, FIN teardown  
🟡 UDP: connectionless, use cases (DNS, streaming, embedded telemetry), broadcast, multicast  
🟡 SSH: key exchange, symmetric encryption, HMAC, key-based auth, tunnels (local/remote/dynamic), scp, sftp, hardening  
🟡 Wireshark: capturing, display filters, follow TCP stream, decode protocols  
🩷 DNS: resolution process, A/AAAA/CNAME records, dig/nslookup  
🩷 DHCP: DORA process, lease time  
⬜ IPv6: 128-bit addressing, link-local, neighbor discovery  

---

## Projects

### 1. Wireshark Deep-Dive
Capture a full TCP connection lifecycle with `curl`.  
- Identify every packet type, note RTT, window scaling, retransmissions  
- Export as `.pcap` and annotate  

### 2. SSH Hardening on Raspberry Pi
Generate ed25519 keys, configure `sshd_config` (disable PasswordAuthentication, PermitRootLogin no).  
- Set up local port forward, run `iperf3` through tunnel, measure bandwidth overhead  

### 3. Packet Crafting with Python Scapy
Craft and send raw Ethernet frames, custom ICMP pings, ARP requests. Capture and parse responses.  
- Build a simple port scanner using raw TCP SYN packets  

---

## Tools to Install

```bash
sudo apt install wireshark iperf3 nmap
pip install scapy
```

---

## Submission Checklist

- [ ] Amogh — push work to `week06/amogh/`
- [ ] Vaishnavi — push work to `week06/vaishnavi/`
- [ ] Sujay — push work to `week06/sujay/`
- [ ] `.pcap` file or annotated screenshot from Wireshark exercise
- [ ] `sshd_config` diff saved showing hardening changes
