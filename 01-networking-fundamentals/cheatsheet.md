# Cheatsheet: Networking Fundamentals

Quick reference for essential networking concepts and terminology.

## Network Types Quick Reference

| Type | Range | Speed | Latency | Use Case |
|------|-------|-------|---------|----------|
| LAN | Local (<1km) | Fast | <10ms | Home, office |
| WAN | Global (unlimited) | Slow | 50-200ms | Internet, branches |
| MAN | City (1-100km) | Medium | 10-50ms | University campus |
| VLAN | Logical | Fast | <10ms | Network segmentation |

## OSI & TCP/IP Layers

| OSI Layer | TCP/IP Model | Role | Example |
|-----------|--------------|------|---------|
| 7 (Application) | Application | User programs | HTTP, DNS, SMTP |
| 6 (Presentation) | Application | Data formatting | Encryption, compression |
| 5 (Session) | Application | Connection management | Session control |
| 4 (Transport) | Transport | Reliable delivery | TCP, UDP |
| 3 (Network) | Internet | Routing | IP, ICMP |
| 2 (Data Link) | Link | MAC addressing | Ethernet, PPP |
| 1 (Physical) | Link | Cables & signals | Copper, fiber optics |

## IP Address Basics

| Concept | Example | Purpose |
|---------|---------|---------|
| IP Address | 192.168.1.100 | Identifies device on network |
| Subnet Mask | 255.255.255.0 | Identifies network portion of IP |
| CIDR Notation | /24 | Shorter way to write subnet mask |
| Default Gateway | 192.168.1.1 | Router to reach other networks |
| Broadcast | 192.168.1.255 | Address for all devices on network |
| Loopback | 127.0.0.1 | Address for local machine |

## Subnet Mask Quick Reference

| Mask | CIDR | Hosts | Usage |
|------|------|-------|-------|
| 255.255.255.0 | /24 | 254 | Standard small network |
| 255.255.255.128 | /25 | 126 | Half a /24 |
| 255.255.255.192 | /26 | 62 | Quarter of a /24 |
| 255.255.255.224 | /27 | 30 | Eighth of a /24 |
| 255.255.255.240 | /28 | 14 | Sixteenth of a /24 |
| 255.255.0.0 | /16 | 65,534 | Large network |
| 255.0.0.0 | /8 | 16,777,214 | Enterprise network |

## Private IP Address Ranges

| Range | CIDR | Usage |
|-------|------|-------|
| 10.0.0.0 - 10.255.255.255 | 10.0.0.0/8 | Large enterprises |
| 172.16.0.0 - 172.31.255.255 | 172.16.0.0/12 | Medium networks |
| 192.168.0.0 - 192.168.255.255 | 192.168.0.0/16 | Home networks |

## Common Network Protocols

| Protocol | Port | Type | Purpose | Reliability |
|----------|------|------|---------|-------------|
| HTTP | 80 | TCP | Web (unsecure) | High |
| HTTPS | 443 | TCP | Web (secure) | High |
| FTP | 21 | TCP | File transfer | High |
| SSH | 22 | TCP | Secure shell | High |
| SMTP | 25 | TCP | Email sending | High |
| DNS | 53 | UDP | Domain resolution | Medium |
| DHCP | 67/68 | UDP | IP assignment | Medium |
| POP3 | 110 | TCP | Email retrieval | High |
| IMAP | 143 | TCP | Email retrieval | High |
| NTP | 123 | UDP | Time sync | Low |
| SNMP | 161 | UDP | Network management | Low |

## TCP vs UDP Comparison

| Aspect | TCP | UDP |
|--------|-----|-----|
| **Connection** | Connection-based | Connectionless |
| **Reliability** | Guaranteed delivery | Best effort |
| **Ordering** | Maintains order | No ordering |
| **Speed** | Slower | Faster |
| **Error Checking** | Yes | Limited |
| **Overhead** | High | Low |
| **Use Cases** | Email, web, databases | Gaming, streaming, VoIP |

## Network Devices

| Device | Layer | Function | Example |
|--------|-------|----------|---------|
| **Hub** | 1 | Repeater (outdated) | Dumb broadcast device |
| **Switch** | 2 | MAC learning | Ethernet switch |
| **Router** | 3 | IP routing | Home router |
| **Gateway** | 4-7 | Protocol translation | VPN gateway |
| **Firewall** | 3-4 | Packet filtering | Hardware firewall |
| **Access Point** | 2 | Wireless LAN | Wi-Fi router |
| **Proxy** | 7 | Request intermediary | Squid proxy |
| **Load Balancer** | 4-7 | Traffic distribution | F5 load balancer |

## Network Calculations

### Calculate Network Address
1. AND IP address with subnet mask
2. Example: 192.168.1.100 AND 255.255.255.0 = 192.168.1.0

### Calculate Broadcast Address
1. Set all host bits to 1
2. Example: 192.168.1.0 with /24 = 192.168.1.255

### Calculate Number of Hosts
1. Formula: 2^(32-CIDR) - 2
2. Example: /24 = 2^(32-24) - 2 = 256 - 2 = 254 hosts

### Determine if IP in Same Network
1. AND both IPs with subnet mask
2. If results are same, they're on same network

## Common Configuration Scenarios

### Home Network
```
Network: 192.168.1.0/24
Gateway: 192.168.1.1 (router)
DHCP: 192.168.1.100-200
Devices: 1-50
```

### Small Office
```
Network: 10.0.1.0/24
Gateway: 10.0.1.1
Employees: 10.0.1.10-50
Servers: 10.0.1.100-110
Printers: 10.0.1.200-210
```

### Enterprise
```
Primary: 10.0.0.0/8
Subnets:
├─ 10.0.1.0/24 (Floor 1)
├─ 10.0.2.0/24 (Floor 2)
├─ 10.0.3.0/25 (Servers)
├─ 10.0.3.128/25 (Storage)
└─ 10.0.4.0/24 (Printers)
```

## DNS Hierarchy

```
Root Nameserver
       ↑
.com Nameserver
       ↑
google.com Nameserver
       ↑
google.com records (A, MX, CNAME, etc.)
```

**Resolution Process:**
1. Local → Default resolver
2. Resolver → Root nameserver ("Where's .com?")
3. Root → ".com nameserver" 
4. .com nameserver → "google.com nameserver"
5. google.com nameserver → IP address

## DHCP Process (DORA)

| Step | Acronym | Description |
|------|---------|-------------|
| 1 | Discover | Client broadcasts "Who's a DHCP server?" |
| 2 | Offer | DHCP server responds "I can give you an IP" |
| 3 | Request | Client broadcasts "I accept your offer" |
| 4 | Acknowledge | DHCP confirms and assigns lease time |

## TCP 3-Way Handshake

```
Client                          Server

SYN (SEQ=100)              →
                           ← SYN-ACK (SEQ=200, ACK=101)
ACK (SEQ=101, ACK=201)     →

Connection Established!
```

## Network Troubleshooting Checklist

- [ ] Can ping default gateway? (local connectivity)
- [ ] Can ping 8.8.8.8? (internet connectivity)
- [ ] Can resolve DNS? (nslookup google.com)
- [ ] Check ipconfig /all (IP, gateway, DNS)
- [ ] Check arp -a (MAC addresses)
- [ ] Check netstat -an (active connections)
- [ ] Check route print (routing table)
- [ ] Verify firewall rules
- [ ] Check interface status (up/down)
- [ ] Verify correct subnet mask

## Port Ranges

| Range | Type | Usage |
|-------|------|-------|
| 0-1023 | Well-known | Standard services (HTTP, SSH, DNS) |
| 1024-49151 | Registered | User applications |
| 49152-65535 | Dynamic/Private | Temporary/ephemeral ports |

## Packet Flow Summary

```
Application Layer  → Data (HTTP request)
        ↓
Transport Layer    → TCP header + Data
        ↓
Network Layer      → IP header + TCP + Data
        ↓
Data Link Layer    → Ethernet header + All above
        ↓
Physical Layer     → Electrical signals over cable
```

## Network Topology Types

| Type | Advantages | Disadvantages | Usage |
|------|-----------|-----------------|-------|
| **Star** | Easy to manage | Central point failure | Most networks |
| **Mesh** | Redundant | Expensive | Critical networks |
| **Bus** | Simple | Any break stops all | Legacy networks |
| **Ring** | Balanced | Complex | Token ring (obsolete) |
| **Hybrid** | Flexible | Complex | Enterprise networks |

## Common Network Issues & Causes

| Issue | Possible Causes | Solution |
|-------|-----------------|----------|
| No internet | Router down, ISP issue | Reboot router, ping gateway |
| Slow speed | Congestion, far server | Check latency, bandwith |
| Can't reach host | Different subnet, firewall | Check routing, firewall rules |
| DNS failure | DNS server down | Use alternate DNS (8.8.8.8) |
| High packet loss | Bad link, interference | Replace cable, scan for Wi-Fi |

## Key Formulas

**Subnet Calculations:**
- Network Address = IP AND Subnet Mask
- Broadcast = Network + (2^host_bits - 1)
- Usable Hosts = 2^host_bits - 2
- Host Bits = 32 - CIDR

**Example for 192.168.1.0/24:**
- Host Bits = 32 - 24 = 8
- Usable Hosts = 2^8 - 2 = 254

## Network Terminology

| Term | Definition |
|------|-----------|
| **Bandwidth** | Max data rate (Mbps, Gbps) |
| **Latency** | Delay in transmission (ms) |
| **Throughput** | Actual data rate achieved |
| **Jitter** | Variation in latency |
| **Hop** | Each router/device in path |
| **TTL** | Time-To-Live (hops allowed) |
| **MTU** | Maximum Transmission Unit (bytes) |
| **QoS** | Quality of Service (priority) |
| **NAT** | Network Address Translation |
| **VPN** | Virtual Private Network |

## Helpful Commands Summary

```powershell
# Configuration
ipconfig /all              # View all network config
ipconfig /renew            # Get new IP via DHCP

# Testing
ping host                  # Test connectivity
tracert host              # Trace route
nslookup domain           # DNS lookup

# Monitoring
netstat -an               # All connections
netstat -an | findstr 443 # Filter port 443
arp -a                    # Show ARP table

# Management
route print               # Show routing table
Get-NetAdapter            # List adapters
Get-NetIPConfiguration    # Detailed IP config
```

