# Extending Your Network - TryHackMe Room

**Date:** December 2024
**Topics:** Port Forwarding, Firewalls, VPNs, LAN Networking Devices

---

## Table of Contents
1. [Introduction to Port Forwarding](#introduction-to-port-forwarding)
2. [Firewalls 101](#firewalls-101)
3. [Practical - Firewall](#practical---firewall)
4. [VPN Basics](#vpn-basics)
5. [LAN Networking Devices](#lan-networking-devices)

---

## Introduction to Port Forwarding

### What is Port Forwarding?

Port forwarding (also called **port mapping**) is a technique that allows external devices on the internet to access services on your private network by redirecting communication requests from one address and port to another.

### The Problem: NAT Blocking

Due to **Network Address Translation (NAT)**, devices on the internet cannot directly connect to your internal devices.

```
Internet (Public)          Router (NAT)         Home Network (Private)
                              
Anyone outside     ê    Can't reach    ê    192.168.1.100:22 (SSH server)
```

**Why this happens:**
- Private IP addresses (192.168.x.x, 10.x.x.x) are **not routable** on the internet
- Your router uses one public IP for all internal devices
- Incoming connections don't know which internal device to reach

### The Solution - Port Forwarding

Port forwarding creates a **mapping** that tells the router where to send incoming traffic.

```
Internet                   Router                 Internal Network
                       (203.0.113.5)
                           
External request    í  Port Forward Rule  í   192.168.1.100:22
to port 2222              2222 í 22
```

### Port Forwarding Rule Anatomy

A port forwarding rule consists of:

1. **External (Public) Port**: Port on your router's public IP
2. **Internal IP Address**: Private IP of the device hosting the service
3. **Internal (Private) Port**: Port on the internal device
4. **Protocol**: TCP, UDP, or Both

**Example Rule:**
```
External Port: 2222
Internal IP: 192.168.1.100
Internal Port: 22
Protocol: TCP

Complete Rule: 203.0.113.5:2222 í 192.168.1.100:22
```

**How to use it:**
```bash
# From anywhere on the internet:
ssh user@203.0.113.5 -p 2222

# Router automatically forwards to:
# 192.168.1.100:22
```

---

### Common Port Forwarding Use Cases

#### 1. **Home Web Server**
```
External: yourpublicip.com:8080
Internal: 192.168.1.50:80

Rule: 8080 (TCP) í 192.168.1.50:80
```

**Access from internet:**
```
http://203.0.113.5:8080
```

#### 2. **Gaming Server (Minecraft)**
```
External: yourpublicip:25565
Internal: 192.168.1.25:25565

Rule: 25565 (TCP) í 192.168.1.25:25565
```

**Players connect with:**
```
203.0.113.5:25565
```

#### 3. **Security Camera System**
```
External: yourpublicip:8081
Internal: 192.168.1.200:80

Rule: 8081 (TCP) í 192.168.1.200:80
```

#### 4. **Remote Desktop (RDP)**
```
External: yourpublicip:3390
Internal: 192.168.1.10:3389

Rule: 3390 (TCP) í 192.168.1.10:3389
```

**Why use 3390 instead of 3389?**
- Non-standard ports are harder for attackers to find
- Port scanners often target default ports
- Adds security through obscurity (not a replacement for strong authentication!)

#### 5. **FTP Server**
```
Control Channel: 21 (TCP) í 192.168.1.30:21
Data Channel: 20 (TCP) í 192.168.1.30:20
Passive Ports: 50000-50100 (TCP) í 192.168.1.30:50000-50100
```

---

### How to Configure Port Forwarding

#### Router Configuration (General Steps):

1. **Find your router's IP** (usually `192.168.1.1` or `192.168.0.1`)
   ```bash
   # Windows
   ipconfig | find "Default Gateway"

   # Linux/Mac
   ip route | grep default
   netstat -rn | grep default
   ```

2. **Access router admin panel**
   - Open browser: `http://192.168.1.1`
   - Login with admin credentials

3. **Locate Port Forwarding section**
   - Look for: "Port Forwarding", "Virtual Servers", "NAT Forwarding"

4. **Create forwarding rule**
   ```
   Service Name: SSH Server
   External Port: 2222
   Internal IP: 192.168.1.100
   Internal Port: 22
   Protocol: TCP
   ```

5. **Save and apply**

#### Setting Static IP (Important!)

† **Always assign static IP to devices with port forwarding!**

If your device's IP changes (via DHCP), port forwarding will break.

**Method 1: DHCP Reservation (Recommended)**
```
Router Settings í DHCP í Add Reservation
MAC Address: AA:BB:CC:DD:EE:FF
Reserved IP: 192.168.1.100
```

**Method 2: Static IP on Device**
```bash
# Linux - /etc/network/interfaces
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4
```

---

### Testing Port Forwarding

#### From Inside Network:
```bash
# Check if service is running locally
netstat -an | grep 22
ss -tulpn | grep 22

# Test local connection
ssh user@192.168.1.100
```

#### From Outside Network:
```bash
# Use online port checker
# canyouseeme.org

# Or use nmap from external network
nmap -p 2222 your-public-ip

# Test actual connection
ssh user@your-public-ip -p 2222
```

#### Troubleshooting:
```
 Service running on internal device?
 Internal firewall allowing port?
 Router firewall allowing port?
 Port forwarding rule correct?
 Using correct public IP?
 ISP blocking the port?
```

---

### Security Considerations

#### † Dangers of Port Forwarding:

1. **Direct Internet Exposure**
   - Your device is directly accessible from the internet
   - Vulnerable to brute-force attacks
   - Can be scanned by attackers

2. **Attack Surface Increase**
   - More open ports = more potential vulnerabilities
   - Exploits in services can compromise your network

3. **DDoS Vulnerability**
   - Exposed services can be targeted for denial-of-service attacks

4. **No Traffic Inspection**
   - Unlike VPNs, port forwarding doesn't inspect or filter traffic
   - Malicious traffic goes straight to your device

#### =· Port Forwarding Security Best Practices:

**1. Use Non-Standard Ports**
```
L Bad:  External 22 í Internal 22 (SSH)
 Good: External 2222 í Internal 22 (SSH)
```

**2. Implement Strong Authentication**
```bash
# SSH: Use key-based authentication, disable password auth
ssh-keygen -t ed25519
# Add to ~/.ssh/authorized_keys

# Disable password auth in /etc/ssh/sshd_config
PasswordAuthentication no
PubkeyAuthentication yes
```

**3. Use Firewall Rules**
```bash
# Only allow specific source IPs
ufw allow from 203.0.113.50 to any port 22
```

**4. Enable Fail2Ban (Rate Limiting)**
```bash
# Install fail2ban
sudo apt install fail2ban

# Protects against brute-force attacks
# Automatically blocks IPs after failed login attempts
```

**5. Keep Services Updated**
```bash
# Regular updates
sudo apt update && sudo apt upgrade
```

**6. Use VPN Instead (When Possible)**
```
Port Forwarding:  Internet í Service (Direct exposure)
VPN:             Internet í VPN í Service (Encrypted tunnel)
```

**7. Monitor Access Logs**
```bash
# SSH logs
tail -f /var/log/auth.log

# Web server logs
tail -f /var/log/nginx/access.log
```

**8. Implement Intrusion Detection**
```bash
# Install Suricata or Snort
sudo apt install suricata
```

---

### Port Forwarding vs. VPN

| Feature | Port Forwarding | VPN |
|---------|----------------|-----|
| **Security** | † Low | = High (encrypted) |
| **Setup Complexity** | Easy | Moderate |
| **Traffic Encryption** | L No |  Yes |
| **Access Control** | Port-based only | User authentication |
| **Use Case** | Single service access | Full network access |
| **Recommended For** | Quick public access | Secure remote access |

**When to use Port Forwarding:**
- Hosting public web server
- Gaming server for friends
- Quick file sharing
- Testing/development

**When to use VPN instead:**
- Corporate remote access
- Accessing multiple services
- Security is critical
- Need encrypted tunnel

---

## Firewalls 101

### What is a Firewall?

A **firewall** is a network security system that monitors and controls incoming and outgoing network traffic based on predetermined security rules.

**Analogy:** Think of a firewall as a security guard at a building:
- Checks everyone entering and leaving
- Has a list of who's allowed in
- Blocks suspicious individuals
- Logs all activity

```
Internet  í  [FIREWALL]  í  Your Network
              ì
         Allow/Deny
         Based on Rules
```

### Why Firewalls are Essential

**Without Firewall:**
```
Internet  í  Your Computer (All ports open)
             ë
         Vulnerable to:
         - Unauthorized access
         - Port scans
         - Exploits
         - Malware
```

**With Firewall:**
```
Internet  í  [FIREWALL - Blocks unwanted]  í  Your Computer
                  ì
             Only allowed traffic passes
```

---

### How Firewalls Work

Firewalls examine network packets and make decisions based on:

1. **Source IP Address**: Where is traffic coming from?
2. **Destination IP Address**: Where is it going?
3. **Source Port**: Which application sent it?
4. **Destination Port**: Which service is it targeting?
5. **Protocol**: TCP, UDP, ICMP?
6. **Packet Content**: What's inside? (Advanced firewalls)

**Decision:** Allow, Deny, or Drop

```
Packet arrives:
Source: 203.0.113.50:54321
Dest: 192.168.1.10:22
Protocol: TCP

Firewall checks rules:
Rule 1: Allow 192.168.1.0/24 í 192.168.1.10:22  (source doesn't match)
Rule 2: Allow 203.0.113.0/24 í any:22  (MATCH - ALLOW)

Result: Packet allowed through
```

---

### Types of Firewalls

#### 1. **Packet Filtering Firewall (Stateless)**

**OSI Layer:** 3-4 (Network & Transport)

Inspects each packet **independently** based on:
- IP addresses
- Ports
- Protocol

**Example Rules:**
```
ALLOW: 192.168.1.0/24 í any, port 443, TCP
DENY: any í 192.168.1.100, port 23, TCP
ALLOW: any í any, port 53, UDP
```

**Advantages:**
- Fast (minimal processing)
- Simple to configure
- Low overhead

**Disadvantages:**
- No context awareness
- Can't detect sophisticated attacks
- Doesn't track connection state

**Example: Stateless Rule Problem**
```
Outbound: 192.168.1.10:54321 í 8.8.8.8:53 (DNS query)  Allowed
Inbound:  8.8.8.8:53 í 192.168.1.10:54321 (DNS response)  Blocked!

Problem: Firewall doesn't know response is related to request
```

---

#### 2. **Stateful Firewall (Dynamic Packet Filtering)**

**OSI Layer:** 3-4 (Network & Transport)

Tracks the **state of connections** and understands context.

**Connection State Table:**
```
Source IP:Port    Dest IP:Port      State       Protocol
192.168.1.10:54321 í 8.8.8.8:53     ESTABLISHED   UDP
192.168.1.10:54322 í 142.250.1.46:443 ESTABLISHED TCP
192.168.1.11:54323 í 1.1.1.1:53     NEW           UDP
```

**How it works:**
1. Outbound request creates entry in state table
2. Related inbound traffic automatically allowed
3. Unsolicited inbound traffic blocked

**Example:**
```
1. You: 192.168.1.10:54321 í 8.8.8.8:53 (DNS query)
   Firewall: Creates state entry, allows packet

2. Response: 8.8.8.8:53 í 192.168.1.10:54321
   Firewall: Checks state table, finds matching entry, allows

3. Random: 203.0.113.99:12345 í 192.168.1.10:22
   Firewall: No state entry, DENY
```

**Advantages:**
- Understands connection context
- Better security than stateless
- Prevents many spoofing attacks

**Disadvantages:**
- More CPU/memory intensive
- State table can be exhausted (DDoS)

---

#### 3. **Application Layer Firewall (Layer 7)**

**OSI Layer:** 7 (Application)

Inspects the **actual content** of packets, not just headers.

**What it can do:**
- Block specific URLs
- Filter by file type
- Detect malware in HTTP traffic
- Enforce protocol compliance
- Block SQL injection attempts

**Example Rules:**
```
BLOCK: HTTP requests containing "malware.com"
BLOCK: Files with .exe extension via HTTP
ALLOW: Only HTTPS on port 443 (no HTTP fallback)
BLOCK: SQL queries with "DROP TABLE" in HTTP POST
```

**Deep Packet Inspection (DPI):**
```
Regular Firewall sees:
Source: 192.168.1.10:54321
Dest: 203.0.113.50:80
Protocol: HTTP
Decision: ALLOW (port 80 allowed)

Layer 7 Firewall also sees:
HTTP Method: GET
URL: http://malware.com/virus.exe
User-Agent: wget/1.20.3
Decision: BLOCK (malicious domain)
```

**Advantages:**
- Detects application-layer attacks
- Can enforce business policies
- Blocks sophisticated threats

**Disadvantages:**
- High CPU usage
- Can slow down traffic
- More complex to configure

---

#### 4. **Next-Generation Firewall (NGFW)**

Combines **all firewall types** plus advanced features:

**Features:**
-  Stateful inspection
-  Application awareness
-  Intrusion Prevention System (IPS)
-  Deep Packet Inspection (DPI)
-  Threat intelligence integration
-  SSL/TLS inspection
-  User identity awareness
-  Advanced malware protection

**Example Scenario:**
```
User tries to download file:

1. Stateful check: Connection valid? 
2. Application check: Is it really HTTP? 
3. IPS check: Contains exploit code?  BLOCK
4. Threat intel: Domain on blacklist?  BLOCK
5. Malware scan: File signature matches virus?  BLOCK
6. User policy: Is user allowed to download .exe?  BLOCK
```

**Popular NGFW Solutions:**
- Palo Alto Networks
- Cisco Firepower
- Fortinet FortiGate
- Check Point
- Sophos XG

---

### Firewall Rules - Core Concepts

#### Rule Structure

Every firewall rule contains:

```
[Action] [Protocol] [Source] [Destination] [Port] [Options]

Example:
ALLOW TCP 192.168.1.0/24 í ANY:443
DENY  TCP ANY í 192.168.1.100:23
ALLOW UDP ANY í ANY:53
```

#### Rule Evaluation Order

=® **Critical:** Firewalls evaluate rules **from top to bottom**, **first match wins!**

**Example 1 - Order Matters:**
```
Rule 1: DENY ALL
Rule 2: ALLOW 192.168.1.10 í ANY:80

Result: Everything blocked (Rule 1 matches first)
```

**Example 2 - Correct Order:**
```
Rule 1: ALLOW 192.168.1.10 í ANY:80
Rule 2: DENY ALL

Result: 192.168.1.10 can access port 80, everything else blocked
```

**Best Practice:**
```
1. Specific ALLOW rules (most restrictive first)
2. Specific DENY rules
3. General ALLOW rules
4. Default DENY ALL (implicit or explicit)
```

---

#### Default Policies

**Two approaches:**

**1. Default Allow (Blacklist Approach)**
```
Allow everything by default
Block specific threats

L NOT RECOMMENDED
- Hard to maintain
- Easy to miss threats
- Insecure
```

**2. Default Deny (Whitelist Approach)**
```
Block everything by default
Only allow what's explicitly needed

 RECOMMENDED - "Principle of Least Privilege"
- More secure
- Easier to audit
- Best practice
```

**Example Default Deny Setup:**
```
Rule 1: ALLOW SSH from 192.168.1.0/24 to any port 22
Rule 2: ALLOW HTTPS from any to any port 443
Rule 3: ALLOW HTTP from any to any port 80
Rule 4: ALLOW DNS from any to any port 53
Rule 5: DENY ALL (default policy)
```

---

### Common Firewall Actions

| Action | Behavior | When to Use |
|--------|----------|-------------|
| **ALLOW/ACCEPT** | Let packet through | Legitimate traffic |
| **DENY/REJECT** | Block and notify sender | Let sender know blocked |
| **DROP** | Silently discard | Hide existence (stealth) |
| **LOG** | Record event | Monitoring/debugging |
| **RATE LIMIT** | Throttle connections | Prevent DDoS |

**DENY vs DROP Example:**

```bash
# Attacker scans port 22

DENY action:
Attacker: SYN í Port 22
Firewall: RST (connection refused)
Attacker knows: Service exists but blocked

DROP action:
Attacker: SYN í Port 22
Firewall: (silence)
Attacker knows: Nothing (timeout)
```

**When to use each:**
- **DENY**: Internal users (helpful error messages)
- **DROP**: External/untrusted (hide infrastructure)

---

## Practical - Firewall

### Linux Firewall - UFW (Uncomplicated Firewall)

**UFW** is a user-friendly frontend for `iptables`.

#### Installation
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ufw

# Check status
sudo ufw status verbose
```

---

#### Basic UFW Commands

**1. Enable/Disable Firewall**
```bash
# Enable firewall
sudo ufw enable

# Disable firewall
sudo ufw disable

# Check status
sudo ufw status
sudo ufw status verbose
sudo ufw status numbered  # Show rule numbers
```

**2. Default Policies**
```bash
# Set default policies (do this BEFORE enabling!)
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw default deny routed

# Check defaults
sudo ufw status verbose
```

**3. Allow Specific Ports**
```bash
# Allow SSH (port 22)
sudo ufw allow 22/tcp
sudo ufw allow ssh  # Same as above (uses /etc/services)

# Allow HTTPS (port 443)
sudo ufw allow 443/tcp
sudo ufw allow https

# Allow HTTP (port 80)
sudo ufw allow 80/tcp

# Allow both TCP and UDP
sudo ufw allow 53  # DNS
```

**4. Allow from Specific IP**
```bash
# Allow from specific IP
sudo ufw allow from 192.168.1.50

# Allow from IP to specific port
sudo ufw allow from 192.168.1.50 to any port 22

# Allow from subnet
sudo ufw allow from 192.168.1.0/24 to any port 3306
```

**5. Deny Rules**
```bash
# Deny specific port
sudo ufw deny 23/tcp  # Block Telnet

# Deny from specific IP
sudo ufw deny from 203.0.113.50

# Deny from IP to specific port
sudo ufw deny from 203.0.113.50 to any port 80
```

**6. Delete Rules**
```bash
# Method 1: By rule number
sudo ufw status numbered
sudo ufw delete 3  # Delete rule #3

# Method 2: By rule specification
sudo ufw delete allow 80/tcp

# Method 3: Interactive
sudo ufw delete allow from 192.168.1.50
```

**7. Advanced Rules**
```bash
# Allow port range
sudo ufw allow 6000:6007/tcp

# Allow specific interface
sudo ufw allow in on eth0 to any port 80

# Rate limiting (prevent brute-force)
sudo ufw limit ssh  # Max 6 connections per 30 seconds
sudo ufw limit 22/tcp

# Allow with comment
sudo ufw allow 8080/tcp comment 'Web server'
```

**8. Application Profiles**
```bash
# List available profiles
sudo ufw app list

# Allow application
sudo ufw allow 'OpenSSH'
sudo ufw allow 'Nginx Full'  # HTTP + HTTPS
sudo ufw allow 'Apache'

# Show app info
sudo ufw app info 'Nginx Full'
```

---

#### Practical UFW Examples

**Example 1: Basic Web Server**
```bash
# Default deny
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (management)
sudo ufw allow 22/tcp

# Allow HTTP and HTTPS (web traffic)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable firewall
sudo ufw enable

# Verify
sudo ufw status verbose
```

**Example 2: Database Server (Restricted Access)**
```bash
# Default deny
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH from management network only
sudo ufw allow from 192.168.100.0/24 to any port 22

# Allow MySQL from app servers only
sudo ufw allow from 192.168.1.10 to any port 3306  # App Server 1
sudo ufw allow from 192.168.1.11 to any port 3306  # App Server 2

# Enable firewall
sudo ufw enable
```

**Example 3: Rate Limiting (Anti-Brute Force)**
```bash
# Limit SSH connections
sudo ufw limit ssh

# What this does:
# - Allows max 6 connections per 30 seconds from same IP
# - Blocks IP temporarily if exceeded
# - Protects against brute-force attacks
```

**Example 4: Logging**
```bash
# Enable logging
sudo ufw logging on
sudo ufw logging medium  # Low, medium, high, full

# View logs
sudo tail -f /var/log/ufw.log

# Disable logging
sudo ufw logging off
```

---

### Linux Firewall - iptables (Advanced)

**iptables** is the powerful, low-level firewall tool in Linux.

#### iptables Basics

**Chains:**
- **INPUT**: Incoming packets destined for local system
- **OUTPUT**: Outgoing packets from local system
- **FORWARD**: Packets routed through system

**Tables:**
- **filter**: Default table for packet filtering
- **nat**: Network Address Translation
- **mangle**: Packet alteration

---

#### Basic iptables Commands

**1. View Rules**
```bash
# List all rules
sudo iptables -L -v -n

# -L: List rules
# -v: Verbose (show packet/byte counts)
# -n: Numeric (don't resolve hostnames)

# List specific chain
sudo iptables -L INPUT -v -n

# List with line numbers
sudo iptables -L INPUT --line-numbers
```

**2. Flush Rules (Clear All)**
```bash
# Clear all rules
sudo iptables -F

# Clear specific chain
sudo iptables -F INPUT

# Be careful! This removes all protection
```

**3. Default Policies**
```bash
# Set default policies
sudo iptables -P INPUT DROP    # Drop incoming by default
sudo iptables -P OUTPUT ACCEPT  # Allow outgoing by default
sudo iptables -P FORWARD DROP   # Drop forwarded by default
```

**4. Allow Rules**
```bash
# Allow loopback (localhost)
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP and HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow from specific IP
sudo iptables -A INPUT -s 192.168.1.50 -j ACCEPT

# Allow from subnet to specific port
sudo iptables -A INPUT -s 192.168.1.0/24 -p tcp --dport 3306 -j ACCEPT
```

**5. Block Rules**
```bash
# Block specific IP
sudo iptables -A INPUT -s 203.0.113.50 -j DROP

# Block specific port
sudo iptables -A INPUT -p tcp --dport 23 -j DROP

# Block IP range
sudo iptables -A INPUT -m iprange --src-range 203.0.113.1-203.0.113.100 -j DROP
```

**6. Delete Rules**
```bash
# Delete by line number
sudo iptables -D INPUT 3

# Delete by specification
sudo iptables -D INPUT -p tcp --dport 80 -j ACCEPT
```

---

#### Advanced iptables Examples

**Example 1: Complete Web Server Setup**
```bash
#!/bin/bash

# Flush existing rules
iptables -F
iptables -X

# Default policies
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH (with rate limiting)
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow ping (ICMP)
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# Log dropped packets
iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables-dropped: " --log-level 7

# Drop everything else
iptables -A INPUT -j DROP
```

**Example 2: Rate Limiting / DDoS Protection**
```bash
# Limit SSH connections (4 per minute per IP)
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP

# Limit HTTP connections (20 per minute per IP)
iptables -A INPUT -p tcp --dport 80 -m state --state NEW -m recent --set --name http
iptables -A INPUT -p tcp --dport 80 -m state --state NEW -m recent --update --seconds 60 --hitcount 20 --name http -j DROP

# Protect against SYN floods
iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP
```

**Example 3: Port Knocking (Advanced Security)**
```bash
# Secret knock sequence: 7000, 8000, 9000 to open SSH

# Stage 1: Hit port 7000
iptables -A INPUT -p tcp --dport 7000 -m recent --name knock1 --set -j DROP

# Stage 2: Hit port 8000 (within 10 seconds)
iptables -A INPUT -p tcp --dport 8000 -m recent --name knock1 --rcheck --seconds 10 -m recent --name knock2 --set -j DROP

# Stage 3: Hit port 9000 (within 10 seconds)
iptables -A INPUT -p tcp --dport 9000 -m recent --name knock2 --rcheck --seconds 10 -m recent --name knock3 --set -j DROP

# Allow SSH if knock sequence completed (within 10 seconds)
iptables -A INPUT -p tcp --dport 22 -m recent --name knock3 --rcheck --seconds 10 -j ACCEPT

# Drop all other SSH
iptables -A INPUT -p tcp --dport 22 -j DROP
```

---

#### Saving and Restoring iptables Rules

**Ubuntu/Debian:**
```bash
# Save rules
sudo iptables-save > /etc/iptables/rules.v4

# Restore rules
sudo iptables-restore < /etc/iptables/rules.v4

# Install persistence package
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

**RHEL/CentOS:**
```bash
# Save rules
sudo service iptables save

# Rules saved to: /etc/sysconfig/iptables
```

---

### Windows Firewall

#### PowerShell Commands

**1. View Firewall Status**
```powershell
# Check firewall status
Get-NetFirewallProfile

# Detailed status
Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction
```

**2. Enable/Disable Firewall**
```powershell
# Enable all profiles
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

# Disable firewall (not recommended!)
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

**3. Create Allow Rules**
```powershell
# Allow specific port (TCP)
New-NetFirewallRule -DisplayName "Allow Port 8080" -Direction Inbound -LocalPort 8080 -Protocol TCP -Action Allow

# Allow specific port (UDP)
New-NetFirewallRule -DisplayName "Allow DNS" -Direction Inbound -LocalPort 53 -Protocol UDP -Action Allow

# Allow program
New-NetFirewallRule -DisplayName "Allow Python" -Direction Inbound -Program "C:\Python39\python.exe" -Action Allow

# Allow from specific IP
New-NetFirewallRule -DisplayName "Allow from Admin" -Direction Inbound -RemoteAddress 192.168.1.50 -Action Allow

# Allow from subnet
New-NetFirewallRule -DisplayName "Allow from LAN" -Direction Inbound -RemoteAddress 192.168.1.0/24 -Action Allow
```

**4. Create Block Rules**
```powershell
# Block specific port
New-NetFirewallRule -DisplayName "Block Telnet" -Direction Inbound -LocalPort 23 -Protocol TCP -Action Block

# Block specific IP
New-NetFirewallRule -DisplayName "Block Malicious IP" -Direction Inbound -RemoteAddress 203.0.113.50 -Action Block

# Block program
New-NetFirewallRule -DisplayName "Block App" -Direction Outbound -Program "C:\Program Files\SomeApp\app.exe" -Action Block
```

**5. View and Delete Rules**
```powershell
# View all rules
Get-NetFirewallRule

# View specific rule
Get-NetFirewallRule -DisplayName "Allow Port 8080"

# Delete rule
Remove-NetFirewallRule -DisplayName "Allow Port 8080"

# Delete by group
Remove-NetFirewallRule -DisplayGroup "My Custom Rules"
```

---

#### GUI Management (Windows Defender Firewall)

**Access:**
1. Windows Security í Firewall & network protection
2. Or: `wf.msc` (Advanced Firewall console)

**Creating Rules:**
1. Windows Defender Firewall í Advanced Settings
2. Inbound Rules í New Rule
3. Select rule type:
   - **Port**: Control access by port number
   - **Program**: Control specific application
   - **Predefined**: Windows services
   - **Custom**: Advanced configuration

**Example - Allow Port 8080:**
```
1. New Inbound Rule
2. Rule Type: Port
3. Protocol: TCP, Port: 8080
4. Action: Allow the connection
5. Profile: All (Domain, Private, Public)
6. Name: "Web Server Port 8080"
```

---

### Firewall Testing and Verification

**1. Check Open Ports Locally**
```bash
# Linux
sudo ss -tulpn
sudo netstat -tulpn
sudo lsof -i

# Windows
netstat -an
netstat -ano  # With process IDs
```

**2. Test Firewall Rules**
```bash
# Test from another machine
telnet target-ip 80
nc -zv target-ip 80

# Scan with nmap
nmap -p 22,80,443 target-ip
```

**3. Monitor Firewall Logs**
```bash
# Linux (UFW)
sudo tail -f /var/log/ufw.log

# Linux (iptables)
sudo tail -f /var/log/kern.log | grep iptables

# Windows (PowerShell)
Get-WinEvent -LogName "Microsoft-Windows-Windows Firewall With Advanced Security/Firewall"
```

---

## VPN Basics

### What is a VPN?

**Virtual Private Network (VPN)** creates a secure, encrypted "tunnel" over the internet, allowing you to:
- Access private networks remotely
- Encrypt your internet traffic
- Hide your IP address and location
- Bypass geographic restrictions

**Analogy:** A VPN is like a secure underground tunnel connecting two buildings, while the internet is the public street above.

---

### The Problem VPNs Solve

#### Scenario 1: Public WiFi (Unencrypted)
```
Your Laptop  í  [CafÈ WiFi - Unencrypted]  í  Bank Website
     ë                   ë
Passwords     Attacker can intercept
transmitted   and see everything!
in plain text
```

#### Scenario 2: Privacy Concerns
```
Your Computer  í  [ISP - Can see everything]  í  Websites
                         ë
                  - Tracks browsing
                  - Logs all activity
                  - Can sell data
```

#### Scenario 3: Remote Work
```
Home Worker  í  [Public Internet - Unsafe]  í  Corporate Network
    ë                                               ë
Can't access                              Sensitive resources
internal resources
```

---

### How VPNs Solve These Problems

#### With VPN - Encrypted Tunnel
```
Your Computer  í  [Encrypted VPN Tunnel]  í  VPN Server  í  Internet
     ë                      ë                    ë
  All data         ISP can't see         VPN's IP shown,
 encrypted        what you're doing      not yours
```

**What changes:**
1. **Traffic encrypted** - ISP/attackers see gibberish
2. **IP hidden** - Websites see VPN server's IP, not yours
3. **Location spoofed** - Appear to be in VPN server's location
4. **Secure access** - Safely access corporate resources

---

### VPN Technical Process

**Step-by-Step Connection:**

**1. Initiation**
```
Your Device: "I want to connect to VPN server"
VPN Server: "Okay, let's authenticate"
```

**2. Authentication**
```
Your Device: Sends credentials (username/password or certificate)
VPN Server: Verifies credentials
Result: Authenticated 
```

**3. Tunnel Establishment**
```
Your Device êí VPN Server: Negotiate encryption
- Encryption algorithm (AES-256)
- Key exchange (Diffie-Hellman)
- Authentication method (SHA-256)

Result: Secure tunnel created =
```

**4. Encapsulation**
```
Original Packet:
[Your IP: 192.168.1.10] í [Google: 142.250.185.46] [Data: "search query"]

VPN Encapsulation (adds layers):
[Your IP] í [VPN Server IP] [ENCRYPTED: original packet]
                               ë
                        AES-256 encrypted
```

**5. VPN Server Decapsulation**
```
VPN Server receives encrypted packet
Decrypts it
Sends original packet to destination:
[VPN Server IP: 203.0.113.5] í [Google: 142.250.185.46] [Data: "search query"]

Google sees VPN server's IP, not your real IP!
```

**6. Response Path (Reverse)**
```
Google í VPN Server í Encrypt í Your Device í Decrypt
```

---

### Types of VPNs

#### 1. **Remote Access VPN** (Most Common)

Individual users connect to a private network from remote location.

```
Home Worker  í  [VPN Tunnel]  í  Corporate VPN Gateway  í  Corporate LAN
   (Home)       [Internet]            (Office)              (Resources)
```

**Use Cases:**
- Work from home
- Access office files remotely
- Secure public WiFi usage
- Bypass geo-restrictions

**Topology:**
```
Employee 1 (Home)    
Employee 2 (CafÈ)     í [VPN Server] í Corporate Network
Employee 3 (Airport)                   - File servers
                                        - Databases
                                        - Internal apps
```

---

#### 2. **Site-to-Site VPN** (Corporate Networks)

Connects entire networks together (office to office).

```
Office A Network  êí  [VPN Tunnel]  êí  Office B Network
  192.168.1.0/24       [Internet]         10.0.0.0/24
```

**Use Cases:**
- Connect branch offices
- Link datacenters
- Cloud-to-on-premise integration
- Disaster recovery sites

**Topology:**
```
New York Office          VPN Tunnel          London Office
192.168.1.0/24      ê              í      10.0.0.0/24
                                               
  50 users                                  30 users
  File server                            Database server
```

**Advantages:**
- Transparent to users (always connected)
- No client software needed
- Entire networks can communicate
- Cost-effective vs dedicated lines

---

#### 3. **Client-to-Site VPN**

Similar to remote access, but emphasizes single device accessing network.

```
Your Laptop í VPN Client í Corporate Network
```

---

#### 4. **SSL/TLS VPN (Clientless VPN)**

Access via web browser, no client software needed.

```
Your Browser í HTTPS í SSL VPN Portal í Corporate Apps
                      (e.g., portal.company.com)
```

**Popular Solutions:**
- Cisco AnyConnect
- Palo Alto GlobalProtect
- Fortinet FortiClient SSL-VPN

---

### VPN Protocols

| Protocol | Speed | Security | Encryption | Port | Best For |
|----------|-------|----------|------------|------|----------|
| **OpenVPN** | PPP | PPPPP | Up to AES-256 | 1194 UDP/TCP | General purpose, highly configurable |
| **WireGuard** | PPPPP | PPPPP | ChaCha20 | 51820 UDP | Modern, fast, simple |
| **IPSec** | PPP | PPPPP | AES-256 | 500/4500 UDP | Site-to-site VPNs |
| **IKEv2/IPSec** | PPPP | PPPPP | AES-256 | 500/4500 UDP | Mobile devices (auto-reconnect) |
| **L2TP/IPSec** | PPP | PPP | AES-256 | 1701/500 UDP | Legacy support |
| **PPTP** | PPPPP | P (Weak) | MPPE-128 | 1723 TCP | L Avoid (outdated, insecure) |
| **SSTP** | PPP | PPPP | AES-256 | 443 TCP | Windows, bypass firewalls |

---

#### OpenVPN - Detailed

**Characteristics:**
- Open-source and highly trusted
- Uses SSL/TLS for encryption
- Can run on any port (TCP/UDP)
- Bypass most firewalls
- Cross-platform (Windows, Linux, Mac, mobile)

**Configuration Example:**
```conf
# Client config (client.ovpn)
client
dev tun
proto udp
remote vpn.example.com 1194
ca ca.crt
cert client.crt
key client.key
cipher AES-256-CBC
auth SHA256
comp-lzo
verb 3
```

**Usage:**
```bash
# Linux/Mac
sudo openvpn --config client.ovpn

# Windows
# Use OpenVPN GUI, import .ovpn file
```

---

#### WireGuard - Modern & Fast

**Characteristics:**
- **Extremely fast** (4x faster than OpenVPN)
- **Simple codebase** (~4,000 lines vs OpenVPN's 100,000+)
- Modern cryptography (ChaCha20, Curve25519)
- Built into Linux kernel (5.6+)
- Always-on, low battery usage

**Configuration:**

**Server (`/etc/wireguard/wg0.conf`):**
```ini
[Interface]
PrivateKey = <server_private_key>
Address = 10.0.0.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <client_public_key>
AllowedIPs = 10.0.0.2/32
```

**Client (`/etc/wireguard/wg0.conf`):**
```ini
[Interface]
PrivateKey = <client_private_key>
Address = 10.0.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = <server_public_key>
Endpoint = vpn.example.com:51820
AllowedIPs = 0.0.0.0/0  # Route all traffic through VPN
PersistentKeepalive = 25
```

**Usage:**
```bash
# Generate keys
wg genkey | tee privatekey | wg pubkey > publickey

# Start VPN
sudo wg-quick up wg0

# Check status
sudo wg show

# Stop VPN
sudo wg-quick down wg0
```

---

#### IPSec - Enterprise Standard

**Characteristics:**
- Industry standard for site-to-site VPNs
- Two modes: **Transport** (host-to-host) and **Tunnel** (network-to-network)
- Strong encryption (AES-256)
- Complex configuration

**Components:**
1. **IKE (Internet Key Exchange)**: Negotiates encryption
2. **ESP (Encapsulating Security Payload)**: Encrypts data
3. **AH (Authentication Header)**: Authenticates packets

**Use Case - Site-to-Site:**
```
Office A Router          IPSec Tunnel          Office B Router
 (10.0.1.0/24)      ê              í       (10.0.2.0/24)
```

---

### VPN Setup - Practical Examples

#### Example 1: OpenVPN Server (Ubuntu)

**Install and Configure:**
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install OpenVPN and Easy-RSA
sudo apt install openvpn easy-rsa -y

# Set up PKI
make-cadir ~/openvpn-ca
cd ~/openvpn-ca

# Edit vars file
nano vars
# Set: export KEY_COUNTRY="US"
#      export KEY_PROVINCE="CA"
#      export KEY_CITY="SanFrancisco"
#      export KEY_ORG="MyCompany"

# Source vars and build CA
source vars
./clean-all
./build-ca

# Build server certificate
./build-key-server server

# Generate Diffie-Hellman parameters
./build-dh

# Generate HMAC signature
openvpn --genkey --secret keys/ta.key

# Copy keys to OpenVPN directory
sudo cp keys/{server.crt,server.key,ca.crt,dh2048.pem,ta.key} /etc/openvpn/

# Create server config
sudo nano /etc/openvpn/server.conf
```

**Server Configuration:**
```conf
port 1194
proto udp
dev tun

ca ca.crt
cert server.crt
key server.key
dh dh2048.pem

server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt

push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 1.1.1.1"
push "dhcp-option DNS 1.0.0.1"

keepalive 10 120
tls-auth ta.key 0
cipher AES-256-CBC
auth SHA256
comp-lzo
user nobody
group nogroup
persist-key
persist-tun

status openvpn-status.log
verb 3
```

**Enable IP Forwarding:**
```bash
# Edit sysctl
sudo nano /etc/sysctl.conf
# Uncomment: net.ipv4.ip_forward=1

# Apply changes
sudo sysctl -p
```

**Start OpenVPN:**
```bash
sudo systemctl start openvpn@server
sudo systemctl enable openvpn@server
sudo systemctl status openvpn@server
```

---

#### Example 2: WireGuard Server (Simple Setup)

```bash
# Install WireGuard
sudo apt install wireguard -y

# Generate server keys
wg genkey | sudo tee /etc/wireguard/server_private.key
sudo cat /etc/wireguard/server_private.key | wg pubkey | sudo tee /etc/wireguard/server_public.key

# Generate client keys
wg genkey | sudo tee /etc/wireguard/client_private.key
sudo cat /etc/wireguard/client_private.key | wg pubkey | sudo tee /etc/wireguard/client_public.key

# Create server config
sudo nano /etc/wireguard/wg0.conf
```

**Server Config:**
```ini
[Interface]
PrivateKey = <paste server_private.key here>
Address = 10.0.0.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <paste client_public.key here>
AllowedIPs = 10.0.0.2/32
```

**Client Config:**
```ini
[Interface]
PrivateKey = <paste client_private.key here>
Address = 10.0.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = <paste server_public.key here>
Endpoint = your-server-ip:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

**Start WireGuard:**
```bash
# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# Start VPN
sudo wg-quick up wg0

# Enable on boot
sudo systemctl enable wg-quick@wg0
```

---

### VPN Security Considerations

####  VPN Benefits:

1. **Encryption**: All traffic encrypted (AES-256)
2. **Privacy**: Hides IP and location
3. **Authentication**: Strong user/device verification
4. **Integrity**: Prevents tampering with data
5. **Bypass Censorship**: Access blocked content

#### † VPN Limitations:

1. **Trust Required**: VPN provider can see your traffic
2. **Not Anonymous**: VPN knows your real IP
3. **Logging Policies**: Some VPNs log activity
4. **Speed Reduction**: Encryption adds overhead (10-30% slower)
5. **Single Point of Failure**: If VPN compromised, all traffic exposed

#### =· VPN Best Practices:

**1. Choose Strong Protocols**
```
 WireGuard, OpenVPN, IKEv2
L PPTP (broken), L2TP/IPSec (NSA backdoor concerns)
```

**2. Enable Kill Switch**
```
If VPN disconnects, block all internet traffic
Prevents accidental exposure of real IP
```

**3. Use DNS Leak Protection**
```bash
# Check for DNS leaks
dig @resolver1.opendns.com myip.opendns.com +short

# Should show VPN IP, not your real IP
```

**4. Multi-Factor Authentication**
```
Username + Password + OTP (Google Authenticator)
Certificate-based authentication
```

**5. Split Tunneling (Selective Routing)**
```
Route work traffic through VPN
Route Netflix through regular internet (avoid geo-blocking issues)
```

**6. Regular Audits**
```bash
# Check VPN is active
ip addr show wg0

# Check routing table
ip route

# Verify no IP leaks
curl ifconfig.me  # Should show VPN IP
```

---

### VPN vs Proxy vs Tor

| Feature | VPN | Proxy | Tor |
|---------|-----|-------|-----|
| **Encryption** |  Full | L Usually none |  Multi-layer |
| **Speed** | PPPP | PPPPP | P (Very slow) |
| **Privacy** | PPPP | PP | PPPPP |
| **Anonymity** | PP | P | PPPPP |
| **Use Case** | General privacy | Bypass geo-blocks | Maximum anonymity |
| **Setup** | Easy | Very easy | Easy |
| **Cost** | Paid/Free | Usually free | Free |

---

## LAN Networking Devices

### Complete Device Hierarchy

```
                                                     
  OSI Layer 7 (Application)                          
  - Gateway, Proxy Server, Load Balancer             
                                                     $
  OSI Layer 4-6 (Transport/Session/Presentation)     
  - Firewall (NGFW), Load Balancer                   
                                                     $
  OSI Layer 3 (Network)                              
  - Router, Layer 3 Switch, Multilayer Switch        
                                                     $
  OSI Layer 2 (Data Link)                            
  - Switch, Bridge, Wireless Access Point (WAP)      
                                                     $
  OSI Layer 1 (Physical)                             
  - Hub, Repeater, Modem, Cables                     
                                                     
```

---

### Physical Layer (Layer 1) Devices

#### 1. **Hub** (Obsolete - Don't Use!)

**Function:** Dumb repeater that broadcasts to all ports

```
      [HUB]
       <   
   PC1 PC2 PC3

PC1 sends to PC2:
Hub broadcasts to ALL ports (including PC3)
```

**Problems:**
- L Creates **collisions** (all devices share bandwidth)
- L **Security risk** (everyone sees everyone's traffic)
- L **Slow** (half-duplex only)
- L **No intelligence** (can't filter traffic)

**Example Collision:**
```
PC1 sends data í Hub í Broadcasts to all
PC3 sends data í Hub í COLLISION! Both transmissions corrupted
```

**Modern Alternative:** **Use switches instead!**

---

#### 2. **Repeater**

**Function:** Amplifies/regenerates signals to extend cable distance

```
Device A     50m     [Repeater]     50m     Device B
                    (regenerates signal)
```

**Use Case:**
- Extend Ethernet beyond 100m limit
- Boost WiFi signal
- Extend fiber optic runs

**Modern Examples:**
- WiFi range extenders
- Ethernet media converters
- Fiber optic repeaters

---

#### 3. **Modem**

**Function:** Converts between digital (computer) and analog (ISP) signals

**Types:**

**Cable Modem:**
```
ISP (Coax Cable)  í  [Cable Modem]  í  Ethernet  í  Router
```

**DSL Modem:**
```
ISP (Phone Line)  í  [DSL Modem]  í  Ethernet  í  Router
```

**Fiber ONT (Optical Network Terminal):**
```
ISP (Fiber Optic)  í  [ONT]  í  Ethernet  í  Router
```

**Modern Setup:**
```
ISP  í  Modem  í  Router  í  Switch  í  Devices
```

---

### Data Link Layer (Layer 2) Devices

#### 4. **Switch**

**Function:** Intelligently forwards frames based on MAC addresses

**How it works:**

**1. Learning Phase:**
```
        [Switch]
         <     
   PC1   PC2   PC3

PC1 (MAC: AA:AA) sends frame
Switch learns: "MAC AA:AA is on Port 1"

MAC Address Table:
Port 1: AA:AA:AA:AA:AA:11
Port 2: BB:BB:BB:BB:BB:22
Port 3: CC:CC:CC:CC:CC:33
```

**2. Forwarding:**
```
PC1 wants to send to PC2:

Frame: [Dest MAC: BB:BB] [Source MAC: AA:AA] [Data]

Switch checks MAC table:
  BB:BB is on Port 2
  Forward ONLY to Port 2 (not flooding)
```

**Advantages over Hub:**
-  **No collisions** (full-duplex)
-  **Better security** (traffic not visible to all)
-  **Higher performance** (dedicated bandwidth per port)
-  **Intelligent** (learns MAC addresses)

**Switch Types:**

| Type | Description | Use Case |
|------|-------------|----------|
| **Unmanaged** | Plug-and-play, no configuration | Home networks |
| **Managed** | Configurable (VLANs, QoS, monitoring) | Enterprise |
| **Smart/Web-Managed** | Basic management features | SMB |
| **Layer 3 Switch** | Can route between VLANs | Enterprise core |
| **PoE Switch** | Provides power over Ethernet | IP cameras, VoIP phones, WAPs |

---

#### 5. **Bridge**

**Function:** Connects two network segments, filters traffic between them

```
Segment A  êí  [Bridge]  êí  Segment B
```

**Use Case:**
- Extend network distance
- Reduce collision domains
- Connect different media types (Ethernet î WiFi)

**Modern Usage:**
- Mostly replaced by switches
- Still used in wireless bridging

---

#### 6. **Wireless Access Point (WAP)**

**Function:** Provides WiFi connectivity, bridges wireless to wired network

```
Wired Network  í  [WAP]  í  WiFi Devices
    (Switch)              (Laptops, phones)
```

**Configuration:**

**Standalone Mode:**
```
Internet í Router í Switch í WAP í WiFi Clients
```

**Controller-Based (Enterprise):**
```
Internet í Router í WiFi Controller
                       ì
                      <       
              WAP1   WAP2   WAP3
```

**Common WAP Features:**
- Multiple SSIDs (guest network, employee network)
- VLAN tagging
- Band steering (2.4GHz vs 5GHz)
- Captive portal
- WPA3 encryption

---

### Network Layer (Layer 3) Devices

#### 7. **Router**

**Function:** Routes packets between different networks using IP addresses

```
Internet (WAN)          Router          Home Network (LAN)
203.0.113.5                             192.168.1.0/24
     ë                    ì                    ì
Public IP          [Routing Table]      Private IPs
```

**Routing Table Example:**
```
Destination      Gateway        Interface    Metric
0.0.0.0/0        203.0.113.1    WAN (eth0)   100    (Default route to Internet)
192.168.1.0/24   0.0.0.0        LAN (eth1)   0      (Local network)
10.0.0.0/8       192.168.1.254  LAN (eth1)   10     (VPN route)
```

**Routing Decision Process:**
```
Packet arrives: Dest IP = 8.8.8.8

Router checks routing table:
  8.8.8.8 matches 0.0.0.0/0 (default route)
  Forward to gateway 203.0.113.1 via WAN interface
```

**Router Functions:**
1. **Routing**: Forward packets between networks
2. **NAT**: Translate private î public IPs
3. **Firewall**: Filter traffic
4. **DHCP**: Assign IP addresses
5. **DNS**: Resolve domain names (some routers)

**Home Router Typical Features:**
```
                            
  Consumer Router            
                            $
  - 4-port switch            
  - WiFi access point        
  - Router (NAT, routing)    
  - Firewall (basic)         
  - DHCP server              
                            
```

---

#### 8. **Layer 3 Switch (Multilayer Switch)**

**Function:** Switch + Router in one device

**Capabilities:**
- Layer 2: MAC-based switching (like regular switch)
- Layer 3: IP-based routing (like router)
- **VLAN routing** (inter-VLAN routing)

**Use Case - Inter-VLAN Routing:**
```
        [Layer 3 Switch]
             <         
  VLAN 10   VLAN 20   VLAN 30
  (Sales)   (Eng)     (HR)

Sales PC (VLAN 10) í Engineering Server (VLAN 20)
Switch routes between VLANs internally (fast!)
```

**vs Regular Router:**
```
Speed: Layer 3 switch uses ASICs (hardware routing) = much faster
       Router uses CPU (software routing) = slower

Use: Layer 3 switch = internal routing (between VLANs)
     Router = external routing (to internet)
```

---

### Application Layer Devices

#### 9. **Proxy Server**

**Function:** Intermediary between clients and internet

```
Client  í  Proxy Server  í  Internet
             ì
         - Caching
         - Filtering
         - Logging
```

**Types:**

**Forward Proxy (Client-side):**
```
Company employees í Proxy í Internet
                    ì
              - Block social media
              - Cache websites
              - Log activity
```

**Reverse Proxy (Server-side):**
```
Internet users í Reverse Proxy í Web Servers
                      ì
                - Load balancing
                - SSL termination
                - Caching
```

**Popular Proxies:**
- **Squid**: HTTP/HTTPS proxy
- **Nginx**: Reverse proxy, load balancer
- **HAProxy**: Load balancer

---

#### 10. **Load Balancer**

**Function:** Distributes traffic across multiple servers

```
Internet  í  [Load Balancer]  í  Web Server 1
                   ì           í  Web Server 2
                Distributes    í  Web Server 3
```

**Algorithms:**

**1. Round Robin:**
```
Request 1 í Server 1
Request 2 í Server 2
Request 3 í Server 3
Request 4 í Server 1 (repeat)
```

**2. Least Connections:**
```
Server 1: 10 connections
Server 2: 5 connections  ê Send here (least busy)
Server 3: 8 connections
```

**3. IP Hash:**
```
Same client IP always goes to same server
Maintains session persistence
```

**Health Checks:**
```
Load Balancer pings servers every 5 seconds
Server 2 fails to respond í Mark as down
Route traffic only to Server 1 and 3
```

**Popular Load Balancers:**
- **HAProxy** (open-source)
- **Nginx** (open-source)
- **F5 BIG-IP** (commercial)
- **AWS ELB** (cloud)

---

#### 11. **Gateway**

**Function:** Translates between different protocols

**Examples:**

**VoIP Gateway:**
```
Traditional Phone (PSTN) í Gateway í VoIP Network (SIP)
```

**Email Gateway:**
```
Internal Email í Gateway í Internet Email
                   ì
              - Spam filtering
              - Virus scanning
              - Encryption
```

**IoT Gateway:**
```
Zigbee/Z-Wave Devices í Gateway í WiFi/Ethernet Network
```

---

### Network Topology Examples

#### Small Office Network:
```
Internet
   ì
[Modem]
   ì
[Router + Firewall]
   ì
[PoE Switch]
   ì
  <  ,      ,     
WAP PC IP-Phone Camera
```

#### Enterprise Network:
```
Internet
   ì
[Firewall]
   ì
[Core Layer 3 Switch]
   ì
          <          
[Distribution Layer Switches]
   ì            ì
[Access Switches]
   ì
End Devices (PCs, Phones, Printers)
```

#### Data Center:
```
Internet
   ì
[Load Balancer]
   ì
         <         
[Web Tier Servers]
   ì
[App Tier Load Balancer]
   ì
[Application Servers]
   ì
[Database Load Balancer]
   ì
[Database Cluster]
```

---

## Summary - Key Comparisons

### Port Forwarding vs VPN

| Aspect | Port Forwarding | VPN |
|--------|----------------|-----|
| **Security** | Low (direct exposure) | High (encrypted tunnel) |
| **Use Case** | Single service | Full network access |
| **Encryption** | L No |  Yes |
| **Setup** | Easy (router config) | Moderate (server + client) |
| **Recommended For** | Public services (web server) | Remote secure access |

### Firewall Default Policies

| Policy | Security | Approach | Recommendation |
|--------|----------|----------|----------------|
| **Default Allow** | † Low | Blacklist (block bad) | L Not recommended |
| **Default Deny** | =· High | Whitelist (allow good) |  Recommended |

### VPN Protocols

| Protocol | Speed | Security | Complexity | Best For |
|----------|-------|----------|------------|----------|
| **WireGuard** | PPPPP | PPPPP | Low | Modern VPNs |
| **OpenVPN** | PPP | PPPPP | Medium | General purpose |
| **IPSec** | PPP | PPPPP | High | Site-to-site |
| **PPTP** | PPPPP | P | Low | L Avoid (insecure) |

### Network Devices by Layer

| Device | Layer | Intelligence | Use |
|--------|-------|--------------|-----|
| **Hub** | 1 | None (dumb repeater) | L Obsolete |
| **Switch** | 2 | MAC address learning |  Connect devices |
| **Router** | 3 | IP routing, NAT |  Connect networks |
| **Firewall** | 3-7 | Traffic filtering |  Security |
| **Load Balancer** | 4-7 | Traffic distribution |  Scale services |

---

## Practical Command Reference

### Firewall Commands

```bash
# UFW (Linux)
sudo ufw enable
sudo ufw allow 22/tcp
sudo ufw status verbose

# iptables (Linux)
sudo iptables -L -v -n
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Windows Firewall
Get-NetFirewallProfile
New-NetFirewallRule -DisplayName "Allow Port 8080" -Direction Inbound -LocalPort 8080 -Protocol TCP -Action Allow
```

### VPN Commands

```bash
# WireGuard
wg-quick up wg0
sudo wg show

# OpenVPN
sudo openvpn --config client.ovpn
```

### Network Testing

```bash
# Port scanning
nmap -p 22,80,443 target-ip

# Check open ports
sudo ss -tulpn
netstat -an

# Test connection
telnet target-ip 80
nc -zv target-ip 22
```

---

**End of Document**
