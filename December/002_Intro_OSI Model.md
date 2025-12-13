# The OSI Model - Quick Notes

## What is the OSI Model?
The **OSI (Open Systems Interconnection) Model** is a conceptual framework created by the **International Organization for Standardization (ISO)** that standardizes how different communication systems communicate using standard protocols. It splits network communication into **7 abstract layers**, each stacked upon the last, enabling diverse systems to communicate effectively.

**Purpose:** Breaks down complex network communication into manageable layers for easier understanding, troubleshooting, and development.

![OSI Model 7 Layers](https://cf-assets.www.cloudflare.com/slt3lc6tev37/6ZH2Etm3LlFHTgmkjLmkxp/59ff240fb3ebdc7794ffaa6e1d69b7c2/osi_model_7_layers.png)
*Image source: [Cloudflare - What is the OSI Model?](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/)*

---

## The 7 Layers (Top to Bottom)

### Layer 7 - Application Layer
- **What:** Closest to user, interface for network services
- **Function:** Provides network services to applications
- **Protocols:** HTTP, HTTPS, FTP, DNS, SMTP, SSH
- **Think:** Web browsers, email clients, file transfers
- **Example:** Opening a website in Chrome

### Layer 6 - Presentation Layer
- **What:** Data translator and formatter
- **Function:** Encryption/decryption, compression, data formatting
- **Protocols:** SSL/TLS, JPEG, GIF, ASCII, MPEG
- **Think:** Makes sure data is in readable format
- **Example:** Encrypting HTTPS traffic

### Layer 5 - Session Layer
- **What:** Session/connection manager
- **Function:** Establishes, maintains, terminates connections
- **Protocols:** NetBIOS, RPC, PPTP
- **Think:** Keeps conversations organized between apps
- **Example:** Maintaining login session on website

### Layer 4 - Transport Layer
- **What:** End-to-end communication control
- **Function:** Reliable delivery, error checking, flow control, segmentation
- **Protocols:** TCP (reliable, connection-oriented), UDP (fast, connectionless)
- **Port Numbers:** Identifies applications (HTTP=80, HTTPS=443)
- **Think:** Ensures complete data delivery or fast transmission
- **Example:** TCP ensures all email data arrives intact

### Layer 3 - Network Layer
- **What:** Routing and logical addressing
- **Function:** Routes packets between different networks using IP addresses
- **Protocols:** IP (IPv4/IPv6), ICMP, routers
- **Addressing:** IP addresses (192.168.1.1)
- **Think:** GPS for data - finds best path
- **Example:** Router forwarding packets to destination network

### Layer 2 - Data Link Layer
- **What:** Node-to-node data transfer within same network
- **Function:** Physical addressing (MAC), error detection, frame formatting
- **Protocols:** Ethernet, Wi-Fi (802.11), PPP, switches
- **Addressing:** MAC addresses (AA:BB:CC:DD:EE:FF)
- **Sub-layers:** LLC (Logical Link Control), MAC (Media Access Control)
- **Think:** Direct communication between physically connected devices
- **Example:** Switch forwarding frame to correct port

### Layer 1 - Physical Layer
- **What:** Raw bit transmission over physical medium
- **Function:** Electrical/optical signals, physical specs
- **Examples:** Ethernet cables (Cat5e, Cat6), fiber optic, Wi-Fi radio signals, hubs
- **Units:** Bits (1s and 0s)
- **Think:** The actual hardware and cables
- **Example:** Electrical signals traveling through Ethernet cable

---

## Mnemonics to Remember

**Bottom to Top (1�7):**
- **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way
- (Physical, Data Link, Network, Transport, Session, Presentation, Application)

**Top to Bottom (7�1):**
- **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing
- (Application, Presentation, Session, Transport, Network, Data Link, Physical)

---

## Layer Summary Table

| Layer | Name | Protocol Data Unit | Key Function | Protocols/Devices | Addressing |
|-------|------|-------------------|--------------|-------------------|------------|
| 7 | Application | Data | User interface | HTTP, DNS, FTP, SMTP | - |
| 6 | Presentation | Data | Format, encrypt | SSL/TLS, JPEG | - |
| 5 | Session | Data | Manage sessions | NetBIOS, RPC | - |
| 4 | Transport | Segment/Datagram | Reliable delivery | TCP, UDP | Port numbers |
| 3 | Network | Packet | Routing | IP, ICMP, Router | IP address |
| 2 | Data Link | Frame | Local transfer | Ethernet, Switch | MAC address |
| 1 | Physical | Bit | Physical medium | Cables, Hubs | - |

---

## Encapsulation & De-encapsulation

### How Data Travels Through the OSI Model

**Encapsulation (Sending - Layers 7→1):**
When data is sent from one device to another, it travels **down** through the seven layers of the OSI model, with each layer adding its own **header** (and sometimes **trailer/footer**) to the data.

**Process:**
1. **Application Layer (7):** User creates data (e.g., email, web request)
2. **Presentation Layer (6):** Data is formatted/encrypted
3. **Session Layer (5):** Session information added
4. **Transport Layer (4):** Data becomes **Segments** (TCP) or **Datagrams** (UDP)
   - Adds: Source/destination port numbers, sequence numbers
5. **Network Layer (3):** Segments become **Packets**
   - Adds: IP header with source/destination IP addresses
6. **Data Link Layer (2):** Packets become **Frames**
   - Adds: MAC header with source/destination MAC addresses
   - Adds: Trailer for error checking (FCS - Frame Check Sequence)
7. **Physical Layer (1):** Frames converted to **Bits** (1s and 0s)
   - Transmitted as electrical signals, light pulses, or radio waves

**Data Transformation:**
```
Data → Segments → Packets → Frames → Bits
```

**De-encapsulation (Receiving - Layers 1→7):**
At the receiving device, the process reverses. Data travels **up** through the layers:

1. **Physical Layer (1):** Receives bits, converts to frames
2. **Data Link Layer (2):** Reads MAC addresses, checks errors, strips header/trailer
3. **Network Layer (3):** Reads IP addresses, strips IP header
4. **Transport Layer (4):** Reassembles segments, strips transport header
5. **Session Layer (5):** Manages session state
6. **Presentation Layer (6):** Decrypts/decompresses data
7. **Application Layer (7):** Delivers data to application (web browser, email client)

**Key Concept:** Each layer interprets and strips its corresponding header/footer, passing the data up in a usable form for the next layer.

---

## Real-World Example: Browsing a Website

**Sending Request (7→1) - Encapsulation:**
1. **Application (L7):** Type `www.example.com` in browser → HTTP GET request created
2. **Presentation (L6):** Data encrypted with TLS/SSL for HTTPS
3. **Session (L5):** Establish and maintain session with web server
4. **Transport (L4):** Data segmented, TCP header added
   - **Added:** Source port (random), destination port (443), sequence numbers
5. **Network (L3):** IP header added to create packets
   - **Added:** Source IP (your computer), destination IP (example.com server)
6. **Data Link (L2):** Ethernet header/trailer added to create frames
   - **Added:** Source MAC (your NIC), destination MAC (router), FCS trailer
7. **Physical (L1):** Frame converted to bits, transmitted as electrical/optical signals

**What gets sent:** `[Ethernet Header][IP Header][TCP Header][TLS][HTTP Data][Ethernet Trailer]`

**Receiving Response (1→7) - De-encapsulation:**
1. **Physical (L1):** Electrical signals received, converted to bits
2. **Data Link (L2):** Check FCS for errors, read MAC address, strip Ethernet header/trailer
3. **Network (L3):** Read destination IP, strip IP header
4. **Transport (L4):** Reassemble TCP segments in order, strip TCP header, verify delivery
5. **Session (L5):** Maintain connection state, manage session
6. **Presentation (L6):** Decrypt TLS/SSL encrypted data
7. **Application (L7):** Render HTML/CSS in browser → you see the webpage!

---

## TCP vs UDP (Layer 4)

### TCP (Transmission Control Protocol)
- **Connection:** Connection-oriented (handshake required)
- **Reliability:** Guaranteed delivery, error checking, retransmission
- **Speed:** Slower due to overhead
- **Order:** Maintains packet order
- **Use cases:** Web browsing (HTTP/HTTPS), email (SMTP), file transfer (FTP)
- **Example:** Downloading a file - every byte must arrive correctly

### UDP (User Datagram Protocol)
- **Connection:** Connectionless (no handshake)
- **Reliability:** No guarantee of delivery
- **Speed:** Faster, less overhead
- **Order:** No packet ordering
- **Use cases:** Video streaming, gaming, DNS, VoIP
- **Example:** Live video stream - speed matters more than perfection

---

## Why the OSI Model Matters

### Troubleshooting
- **Physical issues?** Check cables (Layer 1)
- **Can't reach website?** DNS problem (Layer 7) or routing (Layer 3)
- **Switch not forwarding?** Layer 2 issue
- **Packets dropping?** Transport layer (Layer 4)

### Security
- **Layer 1:** Physical security (cable tapping)
- **Layer 2:** MAC spoofing, ARP poisoning
- **Layer 3:** IP spoofing, routing attacks
- **Layer 4:** Port scanning, SYN floods
- **Layer 7:** Web attacks (XSS, SQL injection)

### Communication
- Standardized framework so different vendors' devices can communicate
- Each layer serves the layer above and uses services from the layer below

---

## OSI vs TCP/IP Model

| OSI Layer | TCP/IP Layer |
|-----------|--------------|
| Application, Presentation, Session | Application |
| Transport | Transport |
| Network | Internet |
| Data Link, Physical | Network Access |

**Note:** TCP/IP is more practical and widely used in real networks, but OSI is better for learning and troubleshooting concepts.

---

## Key Takeaways

- **7 layers** from Physical (1) to Application (7)
- Each layer has **specific responsibilities**
- Data travels **down the stack** to send, **up the stack** to receive
- **Encapsulation:** Each layer adds its own header (and sometimes trailer)
- **Troubleshooting:** Know which layer to investigate
- **Security:** Each layer has different attack vectors

---

## References & Credits

- **Image Source:** [Cloudflare - What is the OSI Model?](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/)
- **Additional Resources:**
  - [Cloudflare Network Layers](https://developers.cloudflare.com/fundamentals/reference/network-layers/)
  - TryHackMe OSI Model Room

*Last Updated: December 2024*
