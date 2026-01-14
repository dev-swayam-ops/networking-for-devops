# 02 - OSI and TCP/IP Models

## What You'll Learn

By the end of this module, you'll be able to:
- Understand the 7-layer OSI model in detail
- Explain the TCP/IP model and how it differs from OSI
- Map protocols to each layer
- Troubleshoot network issues by layer
- Understand data encapsulation/decapsulation
- Design network solutions using layered approach

## Prerequisites

- Completion of Module 01 (Networking Fundamentals)
- Understanding of basic networking terms
- Familiarity with network protocols

## Key Concepts

### The OSI Model (Open Systems Interconnection)

The OSI model is a conceptual framework describing how network communication works. It divides the communication process into 7 layers:

```
┌─────────────────────────────────────────────────────┐
│ Layer 7 | Application   | HTTP, DNS, SMTP, FTP     │ User Apps
├─────────────────────────────────────────────────────┤
│ Layer 6 | Presentation  | Encryption, Compression  │ Data Format
├─────────────────────────────────────────────────────┤
│ Layer 5 | Session       | Session management       │ Connections
├─────────────────────────────────────────────────────┤
│ Layer 4 | Transport     | TCP, UDP, SCTP           │ Delivery
├─────────────────────────────────────────────────────┤
│ Layer 3 | Network       | IP, ICMP, IGMP           │ Routing
├─────────────────────────────────────────────────────┤
│ Layer 2 | Data Link     | Ethernet, PPP, MAC       │ Frames
├─────────────────────────────────────────────────────┤
│ Layer 1 | Physical      | Cables, Signals, Plugs   │ Bits
└─────────────────────────────────────────────────────┘
```

**Memory Aid:** Please Do Not Throw Sausage Pizza Away

### Layer Descriptions

| Layer | Name | What It Does | Example Devices | Example Protocols |
|-------|------|------------|-----------------|------------------|
| **7** | Application | User applications, services | Browser, Email client | HTTP, HTTPS, SMTP, DNS, SSH |
| **6** | Presentation | Data formatting, encryption | Encryption software | SSL/TLS, JPEG, MPEG |
| **5** | Session | Manages connections | Session controllers | NetBIOS, RPC, PPTP |
| **4** | Transport | Reliable delivery or speed | Firewalls | TCP, UDP, SCTP, DCCP |
| **3** | Network | Routing between networks | Routers | IP, ICMP, IGMP, IPSec |
| **2** | Data Link | MAC addressing on local network | Switches | Ethernet, PPP, Frame Relay |
| **1** | Physical | Electrical/optical signals | Cables, Connectors | Copper wire, Fiber optic, Radio |

### The TCP/IP Model (Practical Model)

More practical than OSI, with 4 layers instead of 7:

```
┌──────────────────────────────────────────────────┐
│ Application   | HTTP, DNS, SMTP, FTP, SSH, SNMP │
├──────────────────────────────────────────────────┤
│ Transport     | TCP, UDP, SCTP                   │
├──────────────────────────────────────────────────┤
│ Internet      | IP, ICMP, IGMP, IPSec            │
├──────────────────────────────────────────────────┤
│ Link (Network)| Ethernet, PPP, Frame Relay       │
└──────────────────────────────────────────────────┘
```

**Comparison:**
- **OSI**: Theoretical model (7 layers)
- **TCP/IP**: Practical model (4 layers)
- TCP/IP layers roughly map to OSI (with compression of sessions/presentation)

### Data Encapsulation

As data moves down the layers, headers are added:

```
Application   [HTTP Request: GET /index.html]
                    ↓ (add port info)
Transport     [TCP: SRC=52134, DST=443] [HTTP Request]
                    ↓ (add routing info)
Network       [IP: SRC=192.168.1.100, DST=142.250.80.46] [TCP] [HTTP]
                    ↓ (add MAC info)
Data Link     [MAC SRC=AA-BB, DST=11-22] [IP] [TCP] [HTTP]
                    ↓
Physical      Electrical signals
```

Each level adds its own header = **Encapsulation**

On the receiving end, headers are removed layer by layer = **Decapsulation**

## Hands-on Lab: Understanding the OSI Model in Practice

### Objective
Map network activity to OSI layers and understand how data flows through the model.

### Step 1: Monitor Traffic at Different Layers

#### Layer 1 & 2 (Physical & Data Link) - MAC Addresses
```powershell
# See MAC address resolution
arp -a
```

**Expected Output:**
```
Interface: 192.168.1.100 --- 0x6
  Internet Address      Physical Address      Type
  192.168.1.1          AA-BB-CC-DD-EE-FF    dynamic
```

**Analysis:**
- This shows Layer 2 (Data Link)
- MAC addresses resolve on local network only
- AA-BB-CC-DD-EE-FF is Layer 2 address

### Step 2: Monitor Layer 3 (Network) - IP Addresses

```powershell
# See IP routing
route print
```

**Expected Output:**
```
Route Table
===========
Destination     Gateway      Netmask         Interface
0.0.0.0         192.168.1.1  0.0.0.0         192.168.1.100
192.168.1.0     On-link      255.255.255.0   192.168.1.100
127.0.0.0       On-link      255.0.0.0       127.0.0.1
```

**Analysis:**
- This shows Layer 3 (Network)
- Destination IP 192.168.1.0 = your local network (on-link)
- Other destinations = send to gateway 192.168.1.1

### Step 3: Monitor Layer 4 (Transport) - Ports

```powershell
# See all ports and connections
netstat -ano | findstr ESTABLISHED
```

**Expected Output:**
```
Proto  Local Address        Foreign Address      State      PID
TCP    192.168.1.100:52134  142.250.80.46:443    ESTABLISHED 2048
```

**Analysis:**
- This shows Layer 4 (Transport)
- TCP = reliable protocol
- Port 52134 (your app) to port 443 (HTTPS)
- Ensures data delivery

### Step 4: Monitor Layer 5-7 (Session/Presentation/Application)

```powershell
# See what applications are using network
tasklist /v | findstr "2048"
```

Or use PowerShell:
```powershell
# Find process using port 443
(Get-Process | Where-Object {$_.Handles -gt 100}).Name
```

**Analysis:**
- This shows Layer 5-7
- Firefox, Chrome, Outlook, etc. use the network
- Application layer decides what data to send

### Step 5: Create a Complete Packet Diagram

Use the information from all layers to draw what happens:

```
┌──────────────────────────────────────────────────────────┐
│ Layer 7: Application (Firefox Browser)                   │
│ [GET /index.html HTTP/1.1, Host: example.com]           │
├──────────────────────────────────────────────────────────┤
│ Layer 6: Presentation (TLS Encryption)                   │
│ [Encrypted HTTPS data]                                   │
├──────────────────────────────────────────────────────────┤
│ Layer 5: Session (TLS Session)                           │
│ [Session ID, Sequence numbers]                           │
├──────────────────────────────────────────────────────────┤
│ Layer 4: Transport (TCP)                                 │
│ [SRC Port: 52134, DST Port: 443, Seq: 1000]            │
├──────────────────────────────────────────────────────────┤
│ Layer 3: Network (IP)                                    │
│ [SRC IP: 192.168.1.100, DST IP: 142.250.80.46]         │
├──────────────────────────────────────────────────────────┤
│ Layer 2: Data Link (Ethernet)                            │
│ [SRC MAC: AA-BB-CC-DD-EE-FF, DST MAC: 11-22-33-44-55]  │
├──────────────────────────────────────────────────────────┤
│ Layer 1: Physical (Cable)                                │
│ [Electrical signals sent through Ethernet cable]         │
└──────────────────────────────────────────────────────────┘
```

### Step 6: Test Layer-Specific Issues

**Layer 1 Problem**: No cable
```powershell
# Check if interface is up
Get-NetAdapter

Status: Disconnected
```

**Layer 2 Problem**: MAC not found
```powershell
# Check ARP - device not in table
arp -a | findstr 192.168.1.5
# (No output = device not found on local network)
```

**Layer 3 Problem**: Can't reach network
```powershell
# Ping gateway
ping 192.168.1.1
# Should respond
```

**Layer 4 Problem**: Port blocked
```powershell
# Check if port listening
netstat -ano | findstr 443
# No output = port not listening
```

**Layer 7 Problem**: Application error
```powershell
# Browser shows error, but network works fine
# This is application layer issue
```

## Validation Checklist

- [ ] Can explain all 7 OSI layers
- [ ] Understand data encapsulation process
- [ ] Can map protocols to layers
- [ ] Understand difference between OSI and TCP/IP models
- [ ] Can identify which layer a problem is on
- [ ] Know which devices operate at which layers
- [ ] Can troubleshoot using layer-based approach
- [ ] Understand why models are important

## Cleanup

No cleanup needed. All commands are read-only.

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| "HTTP is Layer 4" | HTTP is Layer 7 (Application). TCP is Layer 4 (Transport) |
| "Switches work at Layer 3" | Switches work at Layer 2 (Data Link). Routers work at Layer 3 |
| "MAC addresses route packets" | MAC addresses work locally (Layer 2). IP addresses route (Layer 3) |
| "TCP guarantees delivery" | TCP has error checking but app layer must handle recovery |
| Confusing TCP/IP model with TCP protocol | TCP/IP model has 4 layers; TCP is a protocol at Layer 4 |

## Troubleshooting by Layer

### Layer 1 (Physical) Issues
- Cable unplugged
- Bad cable (twisted/bent)
- Port broken
- **Fix:** Check physical connection, replace cable

### Layer 2 (Data Link) Issues
- MAC not resolving
- Switch port down
- VLAN mismatch
- **Fix:** Check ARP (arp -a), verify switch port

### Layer 3 (Network) Issues
- Can't ping gateway
- Routing table wrong
- Wrong subnet
- **Fix:** Check routing table (route print), verify gateway

### Layer 4 (Transport) Issues
- Port blocked
- Firewall rule blocking
- Wrong port number
- **Fix:** Check netstat, verify firewall rules

### Layer 5-7 (Session/App) Issues
- Application error
- Wrong credentials
- Service not running
- **Fix:** Check application logs, restart service

## Next Steps

- **Module 03:** IP Addressing and Subnetting (deep dive)
- **Module 04:** CIDR and Route Tables
- Create a troubleshooting flowchart by layer
- Practice mapping common protocols to OSI layers

## Resources

- [OSI Model Explained](https://www.cisco.com/c/en/us/support/docs/security/index.html)
- [TCP/IP Model Visual Guide](https://tools.ietf.org/html/rfc1122)
- [Protocol Layer Mapping](https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml)

## Quick Layer Reference

**For troubleshooting, ask:**
1. Is the cable connected? (Layer 1)
2. Can you see the device on network? (Layer 2)
3. Can you ping the device? (Layer 3)
4. Can you reach the port? (Layer 4)
5. Does the application work? (Layer 5-7)

**Answer "no" to any = problem at that layer**
