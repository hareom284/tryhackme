# LAN Topologies - Quick Notes

## 1. Bus Topology
- All devices → single backbone cable
- **Pros:** Simple, cheap, less cable
- **Cons:** Entire network fails if backbone breaks, hard to troubleshoot
- **Status:** Legacy only

## 2. Star Topology ⭐
- All devices → central switch/hub
- **Pros:** Easy to manage, one device failure doesn't affect others, easy troubleshooting
- **Cons:** Central device = single point of failure
- **Status:** **Most common in modern LANs**

## 3. Ring Topology
- Devices in circular chain, token-passing
- **Pros:** Equal access, no collisions, handles heavy traffic
- **Cons:** One device failure breaks network, hard to add/remove devices
- **Status:** Legacy (Token Ring, FDDI)

---

# Network Devices

## Switch
- **What:** Layer 2 device that connects devices within a LAN
- **Function:** Forwards data using MAC addresses
- **How it works:** Learns MAC addresses, builds MAC table, sends frames only to destination port
- **Pros:** Reduces collisions, faster than hubs, separate collision domains
- **vs Hub:** Hub broadcasts to all ports, Switch sends to specific port only

## Router
- **What:** Layer 3 device that connects different networks
- **Function:** Routes traffic between networks using IP addresses
- **How it works:** Uses routing tables to determine best path for packets
- **Key features:** NAT, firewall, DHCP server, connects LAN to Internet
- **vs Switch:** Switch = same network (MAC), Router = between networks (IP)

---

# Key Protocols

## Subnetting
- **Purpose:** Divide large network into smaller sub-networks
- **Benefits:** Better organization, security, reduced broadcast traffic, efficient IP usage
- **How:** Use subnet mask to split network/host portions
- **Example:** 192.168.1.0/24 → /26 = 4 subnets, 62 hosts each
- **Key terms:** CIDR notation (/24), subnet mask (255.255.255.0), network ID, broadcast address

## ARP (Address Resolution Protocol)
- **Purpose:** Maps IP addresses → MAC addresses
- **Process:**
  1. Device needs MAC for an IP
  2. Sends ARP broadcast: "Who has 192.168.1.10?"
  3. Target replies: "I do, my MAC is XX:XX:XX:XX:XX:XX"
  4. Requester caches the IP-MAC mapping
- **Cache:** ARP table (temporary storage)
- **Security:** Vulnerable to ARP spoofing/poisoning attacks

## DHCP (Dynamic Host Configuration Protocol)
- **Purpose:** Auto-assigns IP addresses to devices
- **DORA Process:**
  1. **D**iscover - Client asks "Any DHCP servers here?"
  2. **O**ffer - Server offers available IP
  3. **R**equest - Client says "I'll take it"
  4. **A**cknowledge - Server confirms assignment
- **Provides:** IP address, subnet mask, default gateway, DNS servers
- **Lease time:** How long IP is valid before renewal needed
- **Benefits:** No manual config, prevents IP conflicts, centralized management

