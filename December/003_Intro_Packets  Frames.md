# Packets & Frames - TryHackMe Room

**Date:** December 2024
**Room:** https://tryhackme.com/room/packetsframes

---

## Table of Contents
1. [OSI Model Review](#task-1-osi-model-review)
2. [What Are Packets and Frames?](#task-2-what-are-packets-and-frames)
3. [TCP/IP - The Three-Way Handshake](#task-3-tcpip---the-three-way-handshake)
4. [Practical - Handshake](#task-4-practical---handshake)
5. [UDP/IP](#task-5-udpip)
6. [Ports 101](#task-6-ports-101-practical)
7. [Extending Your Network](#task-7-extending-your-network)

---

## Task 1: OSI Model Review

The **OSI Model** is a 7-layer framework that describes how data travels across a network.

```
Layer 7: APPLICATION    ← What you interact with (Browser, Email)
Layer 6: PRESENTATION   ← Data formatting (encryption, compression)
Layer 5: SESSION        ← Maintains connections
Layer 4: TRANSPORT      ← TCP/UDP (reliability & ports)
Layer 3: NETWORK        ← IP addressing & routing
Layer 2: DATA LINK      ← MAC addresses & frames
Layer 1: PHYSICAL       ← Cables, signals, bits
```

**Memory Trick:** "All People Seem To Need Data Processing"

---

## Task 2: What Are Packets and Frames?

### Encapsulation - The Wrapping Process

Think of sending a gift:
1. **Data** - The actual content (gift)
2. **Segment** - Add port numbers (wrap in box)
3. **Packet** - Add IP addresses (shipping label)
4. **Frame** - Add MAC addresses (delivery truck)
5. **Bits** - Transmit as electrical signals

### Frames (Layer 2 - Data Link)

A **frame** is the container for **local network delivery**.

#### Frame Structure:
```
┌─────────────────────────────────────────────────┐
│ HEADER                                          │
├──────────────┬──────────────┬───────────────────┤
│ Destination  │ Source MAC   │ Payload (Packet)  │
│ MAC Address  │ Address      │                   │
├──────────────┴──────────────┴───────────────────┤
│ TRAILER (Error checking - CRC)                  │
└─────────────────────────────────────────────────┘
```

#### Key Characteristics:
- **MAC Address**: 48-bit physical address (e.g., `AA:BB:CC:DD:EE:FF`)
- **Changes at every hop** (router, switch)
- **Only cares about next destination**, not final destination
- **Error detection**: Uses CRC (Cyclic Redundancy Check)

#### Example - Frame Changes:
```
Your computer → Router:
Frame: [Router MAC: A1:B2:C3:D4:E5:F6] [Your MAC: 11:22:33:44:55:66] [Packet inside]

Router → Next Router:
Frame: [Next Router MAC: FF:EE:DD:CC:BB:AA] [Router MAC: A1:B2:C3:D4:E5:F6] [Same packet inside]
```

### Packets (Layer 3 - Network)

A **packet** is the container for **end-to-end delivery**.

#### Packet Structure (IP Header):
```
┌──────────────────────────────────────┐
│ Version (IPv4/IPv6)                  │
│ Header Length                        │
│ Total Length                         │
│ Time to Live (TTL)                   │ ← Prevents infinite loops
│ Protocol (TCP=6, UDP=17, ICMP=1)     │
│ Source IP Address                    │
│ Destination IP Address               │
│ Data (your actual content)           │
└──────────────────────────────────────┘
```

#### Key Characteristics:
- **IP addresses stay the same** throughout the journey
- **TTL (Time to Live)**: Decreases at each hop (64 → 63 → 62...), prevents loops
- **Protocol field**: Indicates if it's TCP, UDP, ICMP, etc.

#### Real Example - Visiting Google:
```
Packet headers remain constant throughout journey:
Source IP: 192.168.1.10 (your private IP)
Dest IP: 142.250.185.46 (Google's IP)
TTL: 64 → 63 → 62 → 61... (decreases each hop)
Protocol: TCP (6)
```

### Analogy: Mail Delivery System

**Packets = The Box with Address Labels**
- Contains your data
- Has source IP (return address) and destination IP (delivery address)
- Stays the same throughout journey

**Frames = The Delivery Truck**
- Carries the packet from point to point
- Uses MAC addresses (physical addresses)
- Changes at EVERY network hop

```
You (192.168.1.5) → Router → Internet → Google (142.250.185.46)

Frame 1: Your PC → Router (MAC: AA:BB → CC:DD)
Frame 2: Router → Next Router (MAC: CC:DD → EE:FF)
Frame 3: Next Router → Google (MAC: EE:FF → 11:22)

The PACKET (IP addresses) never changes!
The FRAME changes at each step!
```

---

## Task 3: TCP/IP - The Three-Way Handshake

### TCP Characteristics

- **Connection-oriented**: Establishes connection before sending data
- **Reliable**: Guarantees delivery and correct order
- **Error checking**: Detects and resends lost/corrupted packets
- **Flow control**: Prevents overwhelming receiver
- **Slower**: Due to overhead (handshake, acknowledgments, etc.)

### TCP Header Structure

```
┌─────────────────────────────────┐
│ Source Port      (16 bits)      │
│ Destination Port (16 bits)      │
│ Sequence Number  (32 bits)      │ ← Tracks packet order
│ Acknowledgment # (32 bits)      │ ← Confirms received packets
│ FLAGS: SYN, ACK, FIN, RST, PSH  │ ← Control flags
│ Window Size (flow control)      │
│ Checksum (error detection)      │
│ Data                            │
└─────────────────────────────────┘
```

### TCP Flags:
- **SYN**: Synchronize - initiates connection
- **ACK**: Acknowledge - confirms receipt
- **FIN**: Finish - closes connection
- **RST**: Reset - aborts connection
- **PSH**: Push - send data immediately

### The Three-Way Handshake (Detailed)

Think of it like a **phone call**:

#### Step 1: SYN (Client initiates)
```
Client → Server
Flags: SYN
Sequence Number: 1000 (random starting number)
Message: "Hello! Can you hear me? My starting sequence is 1000"
```

#### Step 2: SYN-ACK (Server responds)
```
Server → Client
Flags: SYN + ACK
Sequence Number: 5000 (server's random start)
Acknowledgment: 1001 (client's seq + 1)
Message: "Yes, I hear you! My starting sequence is 5000,
          and I'm ready for your packet 1001"
```

#### Step 3: ACK (Client confirms)
```
Client → Server
Flags: ACK
Sequence Number: 1001
Acknowledgment: 5001 (server's seq + 1)
Message: "Perfect! I'm ready for your packet 5001. Let's communicate!"
```

**Connection Established! Data transfer begins.**

### Why the Three-Way Handshake?

1. **Both sides agree on starting sequence numbers** (prevents confusion)
2. **Confirms both can send AND receive**
3. **Establishes connection parameters** (window size, MSS, options)
4. **Prevents old duplicate connections** from causing problems

### Real Wireshark Capture Example:
```
No.  Time     Source        Dest          Protocol  Info
1    0.000000 192.168.1.10  142.250.1.46  TCP       54321 → 443 [SYN] Seq=0 Len=0
2    0.043215 142.250.1.46  192.168.1.10  TCP       443 → 54321 [SYN,ACK] Seq=0 Ack=1 Len=0
3    0.043567 192.168.1.10  142.250.1.46  TCP       54321 → 443 [ACK] Seq=1 Ack=1 Len=0
4    0.044000 192.168.1.10  142.250.1.46  HTTP      GET / HTTP/1.1
```

### Closing a TCP Connection (Four-Way Handshake)

```
1. Client → Server: FIN (I'm done sending data)
2. Server → Client: ACK (I acknowledge your FIN)
3. Server → Client: FIN (I'm done too)
4. Client → Server: ACK (Goodbye!)

Connection fully closed.
```

### TCP vs UDP Comparison

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented | Connectionless |
| Reliability | Guaranteed delivery | No guarantee |
| Ordering | Packets arrive in order | May arrive out of order |
| Speed | Slower (overhead) | Faster (minimal overhead) |
| Header Size | 20+ bytes | 8 bytes |
| Use Cases | Web, email, file transfer | Streaming, gaming, DNS |

---

## Task 4: Practical - Handshake

### Using Wireshark to Capture TCP Handshake

#### Filter for TCP Handshakes:
```
tcp.flags.syn==1
```

This shows all packets with SYN flag set (connection attempts).

#### What to Look For:

**1. SYN Packet:**
```
Flags: 0x002 (SYN)
Sequence number: 0 (relative)
Source Port: Random high port (e.g., 54321)
Destination Port: Service port (80, 443, 22, etc.)
```

**2. SYN-ACK Packet:**
```
Flags: 0x012 (SYN, ACK)
Sequence number: 0 (relative)
Acknowledgment number: 1
```

**3. ACK Packet:**
```
Flags: 0x010 (ACK)
Sequence number: 1
Acknowledgment number: 1
```

### Common Questions:

- **What's the source port?** → Random high number (49152-65535)
- **What's the destination port?** → Service port (80, 443, 22, etc.)
- **What's the initial sequence number?** → Look at SYN packet (actual or relative)
- **How long did handshake take?** → Time between packet 1 and packet 3

### Example Analysis:
```
Packet 1: 192.168.1.10:54782 → 142.250.185.46:443 [SYN]
  - Client initiating HTTPS connection
  - Using random source port 54782
  - Targeting HTTPS port 443

Packet 2: 142.250.185.46:443 → 192.168.1.10:54782 [SYN-ACK]
  - Server accepting connection
  - Swapped source/dest

Packet 3: 192.168.1.10:54782 → 142.250.185.46:443 [ACK]
  - Client confirming
  - Connection ready for data
```

---

## Task 5: UDP/IP

### UDP Characteristics

- **Connectionless**: No handshake needed
- **Unreliable**: No guarantee of delivery
- **Fast**: Minimal overhead
- **No ordering**: Packets may arrive out of order
- **No error recovery**: Lost packets stay lost
- **No flow control**: Sender doesn't wait for receiver

### UDP Header (Simple Structure)

```
┌──────────────────────────┐
│ Source Port   (16 bits)  │
│ Dest Port     (16 bits)  │
│ Length        (16 bits)  │
│ Checksum      (16 bits)  │
│ Data                     │
└──────────────────────────┘

Only 8 bytes! (vs TCP's 20+ bytes)
```

### UDP Communication Flow

**No Handshake - Just Send!**
```
Client → Server (immediate)
├─ Packet 1: Data sent
├─ Packet 2: Data sent
├─ Packet 3: Lost ✗ (too bad!)
├─ Packet 4: Data sent
└─ Packet 5: Data sent (may arrive before packet 4)

No SYN, no ACK, no confirmation
```

### When to Use UDP

#### ✅ Good For:
- **DNS Queries**: Fast lookups needed
- **Live Video Streaming**: Skip lost frames, keep playing
- **Online Gaming**: Real-time > perfection
- **VoIP Calls**: Live voice communication
- **DHCP**: Getting IP addresses
- **TFTP**: Simple file transfers
- **Broadcasting/Multicasting**: One-to-many communication

#### ❌ Bad For:
- File downloads (would lose data)
- Email (need reliability)
- Web browsing (need complete pages)
- Database transactions (need accuracy)

### Real Example: DNS Lookup via UDP

```
You type: www.google.com

1. Your PC (192.168.1.10:54321) → DNS Server (8.8.8.8:53) via UDP
   Query: "What's the IP for google.com?"

2. DNS Server (8.8.8.8:53) → Your PC (192.168.1.10:54321) via UDP
   Response: "It's 142.250.185.46"

Total time: ~10-50ms
No handshake overhead!

If response lost? Application layer retries (not UDP's job)
```

### Real Example: Live Streaming

```
Streaming server sends video frames:
Frame 1001 arrives ✓
Frame 1002 LOST ✗
Frame 1003 arrives ✓
Frame 1004 arrives ✓

UDP doesn't resend frame 1002
Video player skips it and continues
Better to have minor glitch than buffering delay
```

### UDP vs TCP - The Megaphone Analogy

**TCP = Phone Call**
- You dial, wait for answer
- Both confirm they can hear each other
- If message unclear, repeat it
- Orderly conversation

**UDP = Shouting with Megaphone**
- Just yell into crowd
- Don't know who heard
- Don't care if they missed parts
- No confirmation needed

---

## Task 6: Ports 101 (Practical)

### What Are Ports?

Ports are **16-bit numbers** (0-65,535) that identify specific services or applications on a device.

**Analogy:**
- **IP Address** = Building address (123 Main Street)
- **Port Number** = Apartment/office number (Apt 443)
- Multiple services can run on one IP using different ports

### Port Categories

#### Well-Known Ports (0-1023)
System ports, require admin/root privileges.

| Port | Protocol | Service | Description |
|------|----------|---------|-------------|
| 20 | TCP | FTP Data | File transfer (data channel) |
| 21 | TCP | FTP Control | File transfer (commands) |
| 22 | TCP | SSH | Secure remote login |
| 23 | TCP | Telnet | Insecure remote login (avoid!) |
| 25 | TCP | SMTP | Send email |
| 53 | UDP/TCP | DNS | Domain name lookups |
| 67 | UDP | DHCP Server | IP address assignment (server) |
| 68 | UDP | DHCP Client | IP address assignment (client) |
| 80 | TCP | HTTP | Websites |
| 110 | TCP | POP3 | Receive email |
| 143 | TCP | IMAP | Email (keeps on server) |
| 443 | TCP | HTTPS | Secure websites (SSL/TLS) |
| 445 | TCP | SMB | Windows file sharing |

#### Registered Ports (1024-49151)
Used by specific applications.

| Port | Protocol | Service |
|------|----------|---------|
| 3306 | TCP | MySQL Database |
| 3389 | TCP | RDP (Remote Desktop) |
| 5432 | TCP | PostgreSQL |
| 8080 | TCP | HTTP Alternative |
| 25565 | TCP | Minecraft Server |

#### Dynamic/Private Ports (49152-65535)
Temporary ports used by client applications.

```
When you browse, your computer uses random ports:
Your PC: 192.168.1.10:54782 → Google: 142.250.185.46:443
        (random client port)         (HTTPS server port)
```

### How Ports Work in Practice

#### Scenario: Opening Multiple Browser Tabs

```
Tab 1 (Google):  192.168.1.10:54782 ↔ 142.250.185.46:443
Tab 2 (YouTube): 192.168.1.10:54783 ↔ 172.217.14.206:443
Tab 3 (GitHub):  192.168.1.10:54784 ↔ 140.82.121.4:443

Same source IP, different source ports!
All going to destination port 443 (HTTPS)
```

#### Scenario: Running a Web Server

```
You run: python -m http.server 8080

Server listening on: 0.0.0.0:8080
Others access via: http://192.168.1.10:8080

Connection flow:
Client (192.168.1.50:54321) → Your Server (192.168.1.10:8080)
```

#### Scenario: SSH to Remote Server

```
Command: ssh user@192.168.1.100

Behind the scenes:
Your PC (192.168.1.10:54999) → Server (192.168.1.100:22)
                               (SSH default port)
```

### Port Security Considerations

#### Open Ports = Potential Attack Surface

**Check your open ports:**
```bash
# Windows
netstat -an                  # Show all connections
netstat -ano                 # Show with process ID
netstat -an | find "LISTEN"  # Show listening ports

# Linux/Mac
netstat -tulpn              # Show listening ports
ss -tulpn                   # Modern alternative
lsof -i :80                 # What's using port 80?
sudo nmap localhost         # Scan your own ports
```

#### Close Unnecessary Ports

Only run services you need:
```
Bad:  Ports 21, 23, 80, 443, 445, 3389 all open
Good: Only port 22 (SSH) and 443 (HTTPS) open
```

### Port Forwarding Example

**Scenario: Home server behind router**

```
Public Internet
     ↓
Router (Public IP: 203.0.113.5)
     ↓
Internal Network (192.168.1.0/24)
     └─ Server (192.168.1.100:22)

Port forwarding rule:
External: 203.0.113.5:2222 → Internal: 192.168.1.100:22

You SSH with: ssh user@203.0.113.5 -p 2222
Router forwards to: 192.168.1.100:22
```

---

## Task 7: Extending Your Network

### Network Devices Deep Dive

#### 1. Router
- **OSI Layer**: 3 (Network)
- **Function**: Connect different networks, route packets
- **Uses**: IP addresses for routing decisions
- **Example**: Home router connects LAN (192.168.1.0/24) to Internet

```
Internet (WAN: 203.0.113.5)
        ↓
    [Router]
        ↓
Home Network (LAN: 192.168.1.0/24)
```

**Routing Table Example:**
```
Destination      Gateway        Interface
0.0.0.0/0        203.0.113.1    WAN (Internet)
192.168.1.0/24   0.0.0.0        LAN (Local)
```

#### 2. Switch
- **OSI Layer**: 2 (Data Link)
- **Function**: Connect devices within same network
- **Uses**: MAC addresses
- **Intelligence**: Learns which MAC is on which port

```
        [Switch]
    ┌─────┼─────┐
   PC1   PC2   PC3
```

**MAC Address Table:**
```
Port 1: AA:BB:CC:DD:EE:11 (PC1)
Port 2: AA:BB:CC:DD:EE:22 (PC2)
Port 3: AA:BB:CC:DD:EE:33 (PC3)
```

**How it works:**
1. PC1 sends frame to PC2
2. Switch reads destination MAC (AA:BB:CC:DD:EE:22)
3. Looks up MAC table, finds Port 2
4. Forwards frame ONLY to Port 2 (not flooding)

#### 3. Hub (Obsolete - Don't Use!)
- **OSI Layer**: 1 (Physical)
- **Function**: Repeats signal to ALL ports (dumb device)
- **Problem**: Creates collisions, security issues, slow
- **Modern alternative**: Use switches instead

### Subnetting Basics

#### Why Subnet?

1. **Organization**: Separate departments/functions
2. **Security**: Isolate sensitive systems
3. **Performance**: Reduce broadcast traffic
4. **Management**: Easier troubleshooting

#### Example: Company Network

**Before (One big network):**
```
192.168.0.0/16 → 65,534 devices in chaos!
All devices in same broadcast domain
```

**After (Subnetting):**
```
Sales Department:   192.168.10.0/24  (254 usable hosts)
Engineering:        192.168.20.0/24  (254 usable hosts)
HR:                 192.168.30.0/24  (254 usable hosts)
Servers:            192.168.40.0/24  (254 usable hosts)
Guest WiFi:         192.168.99.0/24  (254 usable hosts)
```

#### Subnet Mask Explained

```
IP Address:    192.168.10.50
Subnet Mask:   255.255.255.0  (or /24 in CIDR notation)

Binary representation:
IP:      11000000.10101000.00001010.00110010
Mask:    11111111.11111111.11111111.00000000
         └────────Network──────────┘└─Host──┘

Network portion: 192.168.10.0
Host portion:    .50
Broadcast:       192.168.10.255
```

#### Common Subnet Masks

| CIDR | Subnet Mask     | Usable Hosts | Use Case |
|------|-----------------|--------------|----------|
| /24  | 255.255.255.0   | 254          | Typical small network |
| /25  | 255.255.255.128 | 126          | Divide /24 in half |
| /26  | 255.255.255.192 | 62           | Small department |
| /27  | 255.255.255.224 | 30           | Very small group |
| /30  | 255.255.255.252 | 2            | Point-to-point links |

### VLANs (Virtual LANs)

**Separate networks logically on same physical switch.**

```
      [Managed Switch with VLANs]
    ┌───────┼────────┐
  VLAN 10 VLAN 20 VLAN 30
  (Sales) (Eng)   (HR)
```

#### VLAN Benefits:

1. **Security**: VLANs can't talk without router (inter-VLAN routing)
2. **Flexibility**: Move devices between VLANs via configuration
3. **Broadcast Control**: Each VLAN = separate broadcast domain
4. **Cost**: No need for separate physical switches

#### VLAN Example:

```
Switch Configuration:
Port 1-8:   VLAN 10 (Sales)
Port 9-16:  VLAN 20 (Engineering)
Port 17-24: VLAN 30 (HR)
Port 25:    Trunk (carries all VLANs to router)
```

**VLAN Tagging (802.1Q):**
```
Frame with VLAN tag:
[Dest MAC][Source MAC][VLAN TAG: 10][Data][CRC]
                       └─ Identifies which VLAN
```

### NAT (Network Address Translation)

#### The IPv4 Address Problem

- **Private IPs**: Not routable on internet (192.168.x.x, 10.x.x.x)
- **Public IPs**: Limited, expensive
- **Solution**: NAT - Many private IPs share one public IP

#### How NAT Works

```
Inside Network          Router (NAT)           Internet
192.168.1.10:54321  →  203.0.113.5:12345  →  142.250.1.46:443
192.168.1.11:54322  →  203.0.113.5:12346  →  8.8.8.8:53
192.168.1.12:54323  →  203.0.113.5:12347  →  1.1.1.1:443

All internal devices share ONE public IP: 203.0.113.5
```

#### NAT Translation Table

```
Internal IP:Port     External IP:Port      Destination
192.168.1.10:54321 ↔ 203.0.113.5:12345 ↔ 142.250.1.46:443
192.168.1.11:54322 ↔ 203.0.113.5:12346 ↔ 8.8.8.8:53
192.168.1.12:54323 ↔ 203.0.113.5:12347 ↔ 1.1.1.1:443
```

#### Types of NAT

**1. Static NAT (One-to-One)**
```
Internal 192.168.1.100 always maps to Public 203.0.113.10
Used for servers that need consistent external IP
```

**2. Dynamic NAT (Pool)**
```
Internal IPs map to pool of public IPs
First come, first served
```

**3. PAT - Port Address Translation (Most Common)**
```
Many internal IPs share one public IP
Different port numbers distinguish connections
This is what home routers use!
```

#### NAT Advantages:
- Conserves public IP addresses
- Adds security layer (hides internal network)
- Allows network changes without affecting external

#### NAT Disadvantages:
- Breaks end-to-end connectivity
- Complicates peer-to-peer applications
- Requires more router processing

---

## Key Takeaways & Summary

### Packets vs Frames

| Aspect | Frame (Layer 2) | Packet (Layer 3) |
|--------|----------------|------------------|
| Addressing | MAC addresses | IP addresses |
| Scope | Local network hop | End-to-end |
| Changes | At every hop | Stays constant |
| Purpose | Local delivery | Network routing |

### TCP vs UDP

| Feature | TCP | UDP |
|---------|-----|-----|
| Setup | 3-way handshake | No handshake |
| Reliability | Guaranteed | Best-effort |
| Speed | Slower | Faster |
| Use | Web, email, files | Streaming, gaming, DNS |

### Port Ranges

- **0-1023**: Well-known (HTTP, HTTPS, SSH, FTP)
- **1024-49151**: Registered (applications)
- **49152-65535**: Dynamic (temporary client ports)

### Network Devices

- **Hub**: Layer 1 - Dumb repeater (obsolete)
- **Switch**: Layer 2 - MAC-based forwarding
- **Router**: Layer 3 - IP-based routing between networks

---

## Practical Commands Reference

### Check Network Connections
```bash
# Windows
netstat -an                  # All connections
netstat -ano                 # With process IDs
ipconfig /all                # Network config

# Linux/Mac
netstat -tulpn              # Listening ports
ss -tulpn                   # Modern alternative
ifconfig                    # Network interfaces
ip addr show                # IP addresses
```

### Packet Capture
```bash
# Wireshark filters
tcp.flags.syn==1            # TCP handshakes
tcp.port==443               # HTTPS traffic
udp.port==53                # DNS queries
ip.addr==192.168.1.10       # Specific IP
```

### Port Scanning
```bash
nmap localhost              # Scan local ports
nmap 192.168.1.1           # Scan router
nmap -p 80,443 target.com  # Specific ports
```

---
