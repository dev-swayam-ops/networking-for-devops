# Cheatsheet: OSI and TCP/IP Models

Quick reference for network models, layers, and troubleshooting.

## OSI Model - The 7 Layers

| # | Layer | Function | Devices | Protocols | PDU |
|---|-------|----------|---------|-----------|-----|
| 7 | Application | User programs, services | PC, Server | HTTP, DNS, SMTP, SSH, FTP | Data |
| 6 | Presentation | Encryption, compression | Gateway | SSL/TLS, JPEG, MPEG | Data |
| 5 | Session | Connection management | Session layer | NetBIOS, RPC | Data |
| 4 | Transport | Reliable delivery | Firewall | TCP, UDP, SCTP | Segment |
| 3 | Network | Routing | Router | IP, ICMP, IGMP | Packet |
| 2 | Data Link | MAC addressing | Switch | Ethernet, PPP, Frame Relay | Frame |
| 1 | Physical | Electrical signals | Hub, Cable | Copper, Fiber, RF | Bits |

**Memory Aid:** "Please Do Not Throw Sausage Pizza Away"

## TCP/IP Model - 4 Layers

| Layer | OSI Equivalent | Purpose | Protocols |
|-------|-----------------|---------|-----------|
| Application | 5-7 | User applications | HTTP, DNS, FTP, SSH, SMTP |
| Transport | 4 | Delivery mechanism | TCP, UDP, SCTP |
| Internet | 3 | Routing | IP, ICMP, IGMP |
| Link | 1-2 | Physical connection | Ethernet, PPP, MAC |

## Layer-by-Layer Protocols

| Layer 7 App | Layer 6 Pres | Layer 5 Sess | Layer 4 Trans | Layer 3 Net | Layer 2 Link | Layer 1 Phys |
|-----------|-----------|-----------|-----------|----------|----------|----------|
| HTTP(S) | SSL/TLS | NetBIOS | **TCP** | **IP** | **Ethernet** | Copper |
| DNS | Compression | RPC | **UDP** | ICMP | PPP | Fiber |
| FTP | Encryption | PPTP | SCTP | IGMP | Frame Relay | RF/Wi-Fi |
| SMTP | JPEG | L2TP | DCCP | IPSec | 802.11 | Signals |
| SSH | MPEG | - | - | - | ATM | Voltage |
| SNMP | - | - | - | - | MPLS | - |

## Protocol Port Reference

| Port | Layer | Protocol | Service | Type |
|------|-------|----------|---------|------|
| 20 | 4 | TCP | FTP Data | Application |
| 21 | 4 | TCP | FTP Control | Application |
| 22 | 4 | TCP | SSH | Application |
| 25 | 4 | TCP | SMTP | Application |
| 53 | 4 | TCP/UDP | DNS | Application |
| 80 | 4 | TCP | HTTP | Application |
| 110 | 4 | TCP | POP3 | Application |
| 143 | 4 | TCP | IMAP | Application |
| 443 | 4 | TCP | HTTPS | Application |
| 3306 | 4 | TCP | MySQL | Application |
| 5432 | 4 | TCP | PostgreSQL | Application |

## Device Operating Layers

| Device | Layer | Function | Intelligence |
|--------|-------|----------|---------------|
| Hub | 1 | Repeats signals | None (dumb) |
| Switch | 2 | Forwards by MAC | Learns MAC table |
| Router | 3 | Routes by IP | Routing intelligence |
| Gateway | 3-7 | Protocol conversion | Full intelligence |
| Firewall | 3-4 | Filters packets | Stateful inspection |
| Modem | 1-2 | ISP conversion | ISP protocol aware |
| Access Point | 1-2 | Wireless bridge | Minimal |
| Load Balancer | 4-7 | Distribution | Application aware |

## Data Encapsulation

**Adding Headers (Downward):**
```
Layer 7: [Application Data]
    ↓ (add transport info)
Layer 4: [TCP Hdr] [App Data]
    ↓ (add network info)
Layer 3: [IP Hdr] [TCP] [App Data]
    ↓ (add link info)
Layer 2: [Eth Hdr] [IP] [TCP] [App Data] [Checksum]
    ↓ (convert to signals)
Layer 1: 1010110101010101... (bits)
```

**Removing Headers (Upward - Decapsulation):**
```
Layer 1: ← Electrical signals
Layer 2: ← Extract frame, verify MAC address
Layer 3: ← Extract packet, check IP address
Layer 4: ← Extract segment, match port
Layer 5-7: ← Pass to application
```

## PDU Names by Layer

| Layer | PDU Name | Contains |
|-------|----------|----------|
| 7 | Data/Message | User information |
| 6 | Data | Formatted/encrypted |
| 5 | Data | Session info + data |
| 4 | Segment/Datagram | TCP/UDP header + data |
| 3 | Packet | IP header + segment |
| 2 | Frame | Eth header + packet + checksum |
| 1 | Bits | 1s and 0s |

## Common Troubleshooting Commands

| Layer | Command | Checks | Failure = |
|-------|---------|--------|----------|
| 1 | `Get-NetAdapter` | Adapter status | No physical connection |
| 2 | `arp -a` | MAC resolution | Can't reach local device |
| 2 | `ipconfig /all` | MAC address shown | Bad NIC |
| 3 | `ipconfig /all` | IP, gateway, DNS | No network access |
| 3 | `ping <gateway>` | Gateway reachable | Routing problem |
| 3 | `route print` | Routing table | Bad routes |
| 4 | `netstat -an` | Active ports | Firewall blocking |
| 4 | `telnet <ip> <port>` | Port accessible | Service not listening |
| 5-7 | App logs | Application state | Service error |

## OSI vs TCP/IP Model

| Aspect | OSI | TCP/IP |
|--------|-----|--------|
| Purpose | Learning/Reference | Implementation |
| Layers | 7 | 4 |
| Type | Theoretical | Practical |
| Created | 1984 | 1969 |
| Real-world match | Moderate | Excellent |
| Session layer | Yes (Layer 5) | Combined |
| Presentation | Yes (Layer 6) | Combined |
| Use case | Education | Networking |

## Layer Troubleshooting Flow

```
Start
  ↓
Is adapter showing? → NO → Layer 1 Problem
  ↓ YES
Is device in ARP? → NO → Layer 2 Problem
  ↓ YES
Can ping gateway? → NO → Layer 3 Problem
  ↓ YES
Port listening? → NO → Layer 4 Problem
  ↓ YES
App connecting? → NO → Layer 5-7 Problem
  ↓ YES
Problem solved!
```

## Layer 1: Physical

**What it does:** Transmits bits over cables/wireless

| Item | Details |
|------|---------|
| Medium | Copper (Cat5/6/7), Fiber, Radio |
| Signal | Electrical voltage, light, RF |
| Speed | Mbps, Gbps |
| Devices | Hub, cable, connector |
| Error | No cable, bad cable, broken port |

**Commands:**
- `Get-NetAdapter` - Check if up
- Visual inspection - Check cable

---

## Layer 2: Data Link

**What it does:** Adds MAC addresses for local delivery

| Item | Details |
|------|---------|
| Addressing | MAC (6 bytes): AA-BB-CC-DD-EE-FF |
| Devices | Switch, NIC |
| Frames | Ethernet frames with MAC headers |
| Protocol | Ethernet, PPP, Frame Relay |
| Error | ARP failure, VLAN mismatch |

**Commands:**
- `arp -a` - Show MAC table
- `ipconfig /all` - Show local MAC
- `arp -s IP MAC` - Add static ARP

---

## Layer 3: Network

**What it does:** Routes packets between networks

| Item | Details |
|------|---------|
| Addressing | IP (IPv4 or IPv6) |
| Devices | Router, firewall |
| Packets | IP packets with source/dest IP |
| Protocol | IP, ICMP, IGMP, IPSec |
| Error | No IP, wrong gateway, no route |

**Commands:**
- `ipconfig /all` - Show IP config
- `ping <ip>` - Test connectivity
- `route print` - Show routing table
- `tracert <host>` - Trace path

---

## Layer 4: Transport

**What it does:** Ensures reliable delivery (TCP) or fast delivery (UDP)

| Item | Details |
|------|---------|
| Protocols | TCP (reliable), UDP (fast) |
| Addressing | Ports (16-bit: 0-65535) |
| Devices | Firewall |
| Segments | TCP/UDP with source/dest ports |
| Error | Port blocked, service not running |

**Commands:**
- `netstat -an` - Show connections
- `netstat -ano` - Include process ID
- `telnet <ip> <port>` - Test port

---

## Layer 5: Session

**What it does:** Manages connection states

| Item | Details |
|------|---------|
| Purpose | Keep connection alive |
| Examples | Login sessions, RPC calls |
| Timeout | Closes idle connections |
| Error | Session drops, timeout |

**Checking:**
- Monitor connection duration
- Check timeout settings
- Application-specific

---

## Layer 6: Presentation

**What it does:** Formats and encrypts data

| Item | Details |
|------|---------|
| Encryption | SSL/TLS (usually Layer 4) |
| Compression | ZIP, GZIP |
| Format | ASCII, EBCDIC, JPEG, MPEG |
| Error | Certificate error, format mismatch |

**Checking:**
- Browser certificate warnings
- Check TLS version
- Verify encryption

---

## Layer 7: Application

**What it does:** User applications and services

| Item | Details |
|------|---------|
| Examples | Web browser, email, FTP |
| Protocols | HTTP, DNS, SMTP, SSH, FTP |
| Interface | User interacts here |
| Error | Application crashes, wrong data |

**Checking:**
- Check application logs
- Service status
- Configuration files

---

## Network Stack Order

**When data goes OUT (Sender):**
```
Layer 7 Application creates data
    ↓
Layer 6 Formats/encrypts
    ↓
Layer 5 Manages session
    ↓
Layer 4 Adds TCP/UDP (ports)
    ↓
Layer 3 Adds IP (addresses)
    ↓
Layer 2 Adds Ethernet (MAC)
    ↓
Layer 1 Sends as electrical signals
```

**When data comes IN (Receiver):**
```
Layer 1 Receives electrical signals
    ↓
Layer 2 Reads Ethernet, checks MAC
    ↓
Layer 3 Reads IP, checks address
    ↓
Layer 4 Reads TCP/UDP, matches port
    ↓
Layer 5 Manages session state
    ↓
Layer 6 Decrypts/decompresses
    ↓
Layer 7 Application reads data
```

## Key Concepts

| Concept | Definition | Example |
|---------|-----------|---------|
| Encapsulation | Adding headers at each layer | TCP adds ports to HTTP data |
| Decapsulation | Removing headers when going up | Switch removes Ethernet header |
| Protocol Stack | Layers of protocols working together | TCP/IP network stack |
| PDU | Protocol Data Unit (data with headers) | Frame at Layer 2 |
| Header | Info added by layer | MAC header at Layer 2 |
| Multiplexing | Many connections on one link | Multiple TCP streams on IP |
| Demultiplexing | Separating multiplexed data | Routing packets by IP |

## Error Messages by Layer

| Layer | Error | Cause |
|-------|-------|-------|
| 1 | "No connection" | Cable unplugged |
| 2 | "Device not found" | ARP failure |
| 3 | "Host unreachable" | No route to network |
| 4 | "Connection refused" | Port blocked or closed |
| 5 | "Session timeout" | Idle timeout |
| 6 | "SSL error" | Certificate problem |
| 7 | "Page not found" | Server doesn't have page |

## Detailed Layer Functions

**Layer 1 (Physical):**
- Convert data to electrical/optical/RF signals
- Handle timing and synchronization
- No intelligence (just transmit)

**Layer 2 (Data Link):**
- Learn MAC addresses
- Forward frames locally
- Prevent loops (STP)
- Separate collision domains

**Layer 3 (Network):**
- Route between networks
- Make path decisions
- Manage IP addressing
- Separate broadcast domains

**Layer 4 (Transport):**
- TCP: Reliable delivery, ordering, flow control
- UDP: Fast, no guarantees
- Port multiplexing

**Layer 5 (Session):**
- Establish/maintain/close connections
- Checkpoint data
- Manage dialog control

**Layer 6 (Presentation):**
- Translate between formats
- Encrypt/decrypt data
- Compress/decompress

**Layer 7 (Application):**
- Provide services to users
- HTTP for web, SMTP for email, etc.
- User interacts here

---

## Quick Decision Matrix

**Is problem at...?**

| Question | YES → | NO → |
|----------|-------|------|
| No physical link? | Layer 1 | Check 2 |
| Can't find device? | Layer 2 | Check 3 |
| Can't reach gateway? | Layer 3 | Check 4 |
| Port blocked? | Layer 4 | Check 5-7 |
| App not responding? | Layer 5-7 | Not network |

