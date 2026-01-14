# Solutions: OSI and TCP/IP Models

Complete solutions and detailed explanations for all exercises.

## Easy Exercises

### Exercise 1: Memorize the OSI Layers

**Solution:**

The 7 OSI Layers (top to bottom):

| # | Name | Function | Memory Aid |
|---|------|----------|-----------|
| 7 | Application | User programs/apps | **P**lease |
| 6 | Presentation | Data formatting, encryption | **D**o |
| 5 | Session | Connection management | **N**ot |
| 4 | Transport | TCP/UDP, delivery | **T**hrow |
| 3 | Network | IP routing | **S**ausage |
| 2 | Data Link | MAC, frames | **P**izza |
| 1 | Physical | Cables, signals | **A**way |

**Reverse Order (bottom to top):**
Physical, Data Link, Network, Transport, Session, Presentation, Application

**Alternative Memory Aids:**
- "Please Do Not Throw Sausage Pizza Away" (original)
- "All People Seem To Need Data Processing"
- "Away Pizza Sausage Throw Not Do Please" (reversed)

**Why This Matters:**
- Understanding layers helps with troubleshooting
- Higher layers depend on lower layers
- When one layer fails, higher layers can't function

---

### Exercise 2: Map Devices to OSI Layers

**Solution:**

| Device | Layer(s) | Purpose | Example |
|--------|----------|---------|---------|
| Hub | Layer 1 | Repeats data on all ports (physical repeater) | 8-port hub |
| Switch | Layer 2 | Learns MAC addresses, forwards frames | Cisco 2960 |
| Router | Layer 3 | Routes between networks using IP | Home router |
| Firewall | Layer 3-4 | Filters packets by IP and port | Fortinet FortiGate |
| Modem | Layer 1-2 | Converts ISP signal to Ethernet | Cable modem |
| Access Point | Layer 1-2 | Wireless to wired bridge | Cisco Meraki |
| Cable | Layer 1 | Physical medium for transmission | Cat6 Ethernet |

**Explanation:**

**Hub (Layer 1):**
- Operates at physical layer
- No intelligence (doesn't learn MAC)
- Every frame goes to every port (flooding)
- Creates collisions, poor performance
- Largely obsolete

**Switch (Layer 2):**
- Learns MAC addresses (MAC table)
- Forwards frames only to correct port
- More efficient than hub
- Standard for LANs

**Router (Layer 3):**
- Makes routing decisions based on IP
- Connects different networks
- Can filter traffic (basic firewall)
- Only device that connects LANs to WAN

**Firewall (Layer 3-4):**
- Filters by IP address (Layer 3)
- Filters by port (Layer 4)
- Stateful (remembers connections)
- Security device

**Modem (Layer 1-2):**
- Converts ISP's protocol to Ethernet (Layer 1-2)
- Modulates/demodulates signal
- DOCSIS for cable, PPP for DSL

**Access Point (Layer 1-2):**
- Converts wireless (802.11) to wired (Ethernet)
- Operates at physical and data link

**Why Device at Each Layer:**

- **Layer 1 devices**: Just repeat signals
- **Layer 2 devices**: Learn hardware addresses
- **Layer 3 devices**: Make intelligent routing decisions
- **Layer 4+ devices**: Can understand protocols

---

### Exercise 3: Map Protocols to OSI Layers

**Solution:**

| Layer 7 App | Layer 6 Pres | Layer 5 Sess | Layer 4 Trans | Layer 3 Net | Layer 2 Link | Layer 1 Phys |
|-------------|-------------|------------|--------------|-----------|------------|-----------|
| HTTP, DNS, FTP, SSH | (none) | (none) | TCP, UDP | IP, ICMP | Ethernet, Frame Relay | (none) |

**Detailed Protocol Reference:**

**Layer 7 (Application):**
- **HTTP** (port 80): Web pages (unencrypted)
- **HTTPS** (port 443): Web pages (encrypted)
- **DNS** (port 53): Domain name resolution
- **SMTP** (port 25): Email sending
- **POP3** (port 110): Email retrieval
- **IMAP** (port 143): Email sync
- **FTP** (port 21): File transfer
- **SSH** (port 22): Secure shell
- **TELNET** (port 23): Remote terminal
- **SNMP** (port 161): Network management

**Layer 6 (Presentation):**
- Usually combined with Layer 7 in TCP/IP model
- TLS/SSL (encryption) - technically Layer 6
- JPEG, MPEG (compression)

**Layer 5 (Session):**
- Usually combined with Layer 7 in TCP/IP model
- NetBIOS (Windows networking)
- RPC (Remote Procedure Call)

**Layer 4 (Transport):**
- **TCP** (Transmission Control Protocol)
  - Reliable, ordered, connection-oriented
  - Used by HTTP, SMTP, SSH, FTP
  
- **UDP** (User Datagram Protocol)
  - Fast, connectionless, unreliable
  - Used by DNS, video streaming, gaming
  
- **SCTP** (Stream Control Transmission Protocol)
  - Between TCP and UDP
  - Used by telecom

**Layer 3 (Network):**
- **IP** (Internet Protocol)
  - IPv4 most common
  - Routes packets between networks
  
- **ICMP** (Internet Control Message Protocol)
  - Used by ping command
  - Network diagnostics
  
- **IGMP** (Internet Group Management Protocol)
  - Multicast group management
  
- **IPSec** (IP Security)
  - Encryption at Layer 3
  - VPN tunnels

**Layer 2 (Data Link):**
- **Ethernet**: Wired LANs (most common)
- **PPP** (Point-to-Point Protocol): Serial links
- **Frame Relay**: WAN protocol (legacy)
- **802.11**: Wi-Fi protocol
- **ARP**: Maps IP to MAC addresses

**Layer 1 (Physical):**
- Not really protocols, but media:
- Copper Ethernet cables
- Fiber optic cables
- Wireless (radio frequencies)
- Connectors (RJ45, SC, etc.)

**Why This Matters:**

**Upper layers (5-7):**
- User cares about these
- Encryption happens here
- Applications interact here

**Middle layers (3-4):**
- Internet infrastructure
- Routing and delivery
- Firewall rules apply here

**Lower layers (1-2):**
- Physical connectivity
- MAC addressing
- Local network communication

---

### Exercise 4: Understand Data Encapsulation

**Solution:**

**Complete Encapsulation Process:**

```
Layer 7: Application (Firefox)
├─ Data: GET /index.html HTTP/1.1
│         Host: example.com
│         User-Agent: Firefox
├─ Size: ~100 bytes
└─ Action: Application creates request

Layer 6: Presentation (TLS)
├─ Adds: Encryption wrapper
├─ Data: [Encrypted] GET /index.html...
├─ Size: ~120 bytes
└─ Action: Encrypt the data

Layer 5: Session (TLS Session)
├─ Adds: Session ID, sequence numbers
├─ Data: [Session Header] [Encrypted Data]
├─ Size: ~128 bytes
└─ Action: Manage session state

Layer 4: Transport (TCP)
├─ Adds: TCP Header
│        - Source Port: 52134
│        - Dest Port: 443
│        - Sequence: 1000000001
│        - Acknowledgment: 2000000001
│        - Flags: SYN, ACK
├─ Data: [TCP Header] [Session] [Encrypted] [App Data]
├─ Size: ~148 bytes (TCP header = 20 bytes)
└─ Action: Add transport reliability

Layer 3: Network (IP)
├─ Adds: IP Header
│        - Version: 4
│        - Source IP: 192.168.1.100
│        - Dest IP: 142.250.80.46
│        - TTL: 64
│        - Protocol: TCP (6)
├─ Data: [IP Header] [TCP Header] [Session] [Encrypted] [App]
├─ Size: ~168 bytes (IP header = 20 bytes)
└─ Action: Add routing information

Layer 2: Data Link (Ethernet)
├─ Adds: Ethernet Frame Header & Trailer
│        - Source MAC: AA-BB-CC-DD-EE-FF (your NIC)
│        - Dest MAC: 11-22-33-44-55-66 (router)
│        - Type: IPv4 (0x0800)
│        - FCS: Checksum
├─ Data: [Eth Hdr] [IP] [TCP] [Session] [Encrypted] [App] [FCS]
├─ Size: ~1514 bytes (Ethernet header = 14 bytes, trailer = 4)
└─ Action: Prepare for local transmission

Layer 1: Physical
├─ Converts: Frame to electrical signals
├─ Encoding: Manchester or 4B/5B encoding
├─ Medium: Copper wire
├─ Signal: +5V for 1, 0V for 0 (example)
└─ Action: Transmit bits on wire
```

**Key Points:**

1. **Each layer adds its own header**: This is encapsulation
2. **Each header serves a purpose**: 
   - Layer 7: Application data
   - Layer 6: Encryption format
   - Layer 5: Session management
   - Layer 4: Port and reliability
   - Layer 3: IP routing
   - Layer 2: MAC addressing
   - Layer 1: Physical transmission

3. **Receiving end reverses**: Decapsulation
   - Layer 1: Receives electrical signals
   - Layer 2: Extracts frame, checks destination MAC
   - Layer 3: Extracts packet, checks destination IP
   - Layer 4: Extracts segment, matches port
   - Layer 5: Manages session
   - Layer 6: Decrypts data
   - Layer 7: Browser displays web page

4. **Headers are necessary**: Each layer needs its information

**Why This Matters:**

- Understanding encapsulation explains why network is layered
- Helps with troubleshooting (know what headers to look for)
- Explains why data size changes through network
- Shows why protocols at each layer are specialized

---

### Exercise 5: Identify Layers in Real Network Activity

**Solution:**

```powershell
# Command 1: ipconfig /all
Layer: 2 and 3

Output shows:
- Layer 3: IPv4 Address, Subnet Mask, Default Gateway
- Layer 2: Physical Address (MAC), Description
- Why: Network configuration at multiple layers
```

```powershell
# Command 2: arp -a
Layer: 2

Output shows:
- Layer 2: Internet Address (IP) to Physical Address (MAC) mapping
- Why: ARP resolves MAC from IP (local network communication)
```

```powershell
# Command 3: route print
Layer: 3

Output shows:
- Layer 3: Destination networks, Gateways, Interfaces
- Why: IP routing decisions are Layer 3 responsibility
```

```powershell
# Command 4: netstat -an | findstr ESTABLISHED
Layer: 4

Output shows:
- Layer 4: TCP/UDP protocols, source/dest ports
- Why: Ports are Layer 4 (transport) concept
```

```powershell
# Command 5: Get-Process | Where-Object {$_.Name -eq "chrome"}
Layer: 5-7

Output shows:
- Layer 7: Chrome is an application (HTTP, HTTPS)
- Layer 5: Chrome manages sessions
- Layer 6: Chrome handles encryption
- Why: Applications operate at top layers
```

**Summary:**

| Command | Layer | Shows What |
|---------|-------|-----------|
| ipconfig | 2, 3 | IP and MAC addresses |
| arp | 2 | MAC to IP mapping |
| route print | 3 | IP routing table |
| netstat | 4 | Ports and connections |
| tasklist/Get-Process | 5-7 | Applications |

---

## Medium Exercises

### Exercise 6: Compare OSI vs TCP/IP Models

**Solution:**

| Aspect | OSI Model | TCP/IP Model |
|--------|-----------|--------------|
| Number of layers | 7 layers | 4 layers |
| Type | Theoretical/Conceptual | Practical/Functional |
| Layer 1 name | Physical | Link/Network Interface |
| Has Session layer | Yes (Layer 5) | Combined with Application |
| Commonly used | Learning/reference | Real-world implementation |
| Layer 3 name | Network | Internet |
| Created | 1984 by ISO | 1969 by ARPANET |
| Scope | Idealized model | Based on real TCP/IP |

**Detailed Comparison:**

**OSI Model:**
- **7 separate layers**: Each has distinct purpose
- **Theoretical**: Created before widespread internet
- **Comprehensive**: Session and Presentation separate
- **Teaching tool**: Best for understanding networking
- **Doesn't match reality**: Real protocols don't fit perfectly

**TCP/IP Model:**
- **4 pragmatic layers**: Combines similar functions
- **Practical**: Based on actual internet protocols
- **Simplified**: Groups related layers
- **Implementation guide**: Real network stack
- **Better matches reality**: Protocols fit well

**Layer Mapping:**

```
OSI                          TCP/IP
─────                        ──────
Layer 7 (Application)    →   Application Layer
Layer 6 (Presentation)   →   (combined with 7)
Layer 5 (Session)        →   (combined with 7)
Layer 4 (Transport)      →   Transport Layer
Layer 3 (Network)        →   Internet Layer
Layer 2 (Data Link)      →   Link Layer
Layer 1 (Physical)       →   (part of Link Layer)
```

**Why Both Models Exist:**

**OSI is better for:**
- Learning fundamentals
- Understanding theoretical concepts
- Communication between networking people
- Academic study
- Organizing knowledge

**TCP/IP is better for:**
- Actual implementation
- Real-world troubleshooting
- Protocol design
- Internet standards
- Practical networking

**In Practice:**
- Learn OSI for concepts
- Use TCP/IP for implementation
- Both are useful, serve different purposes

---

### Exercise 7: Troubleshoot Layer by Layer

**Solution:**

**Complete Troubleshooting Flowchart:**

```
User Problem: "I can't access any websites"
│
├─ LAYER 1: PHYSICAL
│  └─ Command: Get-NetAdapter
│     └─ Check: Status = "Up"?
│        ├─ YES → Go to Layer 2
│        └─ NO → PROBLEM FOUND!
│           └─ Fixes:
│              • Check cable connection
│              • Try different port
│              • Replace cable
│              • Update network driver
│
├─ LAYER 2: DATA LINK
│  └─ Command: ipconfig /all
│     └─ Check: "Physical Address" shows MAC?
│        ├─ YES → Go to Layer 3
│        └─ NO → PROBLEM FOUND!
│           └─ Fixes:
│              • Restart adapter
│              • Update NIC driver
│              • Check switch port
│
├─ LAYER 3: NETWORK
│  └─ Command: ipconfig /all
│     └─ Check: "IPv4 Address" is not 169.x.x.x?
│        ├─ YES → Proceed
│        └─ NO → Check: Can ping default gateway?
│                 ping <gateway-ip>
│        ├─ YES → Go to Layer 4
│        └─ NO → PROBLEM FOUND!
│           └─ Fixes:
│              • Release and renew DHCP: ipconfig /renew
│              • Check router is powered on
│              • Check DHCP is enabled
│              • Restart network adapter
│
├─ LAYER 4: TRANSPORT
│  └─ Command: netstat -an | findstr ESTABLISHED
│     └─ Check: Any established connections to 443 or 80?
│        ├─ YES → Go to Layer 5-7
│        └─ NO → Check: Firewall blocking ports?
│           └─ Check: Windows Firewall status
│              • Temporarily disable Windows Firewall
│              • Retry website access
│              • PROBLEM FOUND!
│                 └─ Fixes:
│                    • Disable firewall temporarily
│                    • Check firewall rules
│                    • Add exception for browser
│
└─ LAYER 5-7: SESSION/PRESENTATION/APPLICATION
   └─ Check: Can you ping 8.8.8.8? (Layer 3)
      ├─ YES → Check: Does nslookup work?
      │        nslookup google.com
      │        ├─ YES → Browser problem!
      │        │  └─ Fixes:
      │        │     • Clear browser cache
      │        │     • Restart browser
      │        │     • Try different browser
      │        │     • Check browser proxy settings
      │        │
      │        └─ NO → DNS problem!
      │           └─ Fixes:
      │              • ipconfig /flushdns
      │              • Set DNS: 8.8.8.8
      │              • Restart system
      │
      └─ NO → Problem is Layer 3 (go back up)
```

**Specific Commands by Layer:**

| Layer | Command | What to Check |
|-------|---------|---------------|
| 1 | `Get-NetAdapter` | Status = Up |
| 2 | `ipconfig /all` | Physical Address present |
| 3 | `ipconfig /all` | IPv4 Address (not 169.x.x.x) |
| 3 | `ping <gateway-ip>` | Reaches gateway |
| 3 | `route print` | Routing table correct |
| 4 | `netstat -an` | Connections exist |
| 4 | Windows Firewall | Not blocking ports |
| 7 | `nslookup google.com` | DNS working |
| 7 | `ping google.com` | Website reachable |

**Common Outcomes:**

**If Layer 1 fails**: Physical connectivity problem
- Check cables, ports, drivers

**If Layer 2 fails**: MAC addressing problem
- Check NIC, drivers, switch port

**If Layer 3 fails**: IP configuration problem
- Check DHCP, gateway, routing

**If Layer 4 fails**: Firewall/port problem
- Check Windows Firewall, port filtering

**If Layer 5-7 fails**: Application/DNS problem
- Check browser, DNS settings, applications

---

### Exercise 8: Analyze a Real Packet

**Solution:**

**Sample Network Connection:**
```powershell
netstat -an | findstr example.com
# Output: TCP    192.168.1.100:52135   93.184.216.34:443    ESTABLISHED
```

**Complete Packet Breakdown:**

```
┌─────────────────────────────────────────────────────────┐
│ LAYER 1: PHYSICAL                                       │
├─────────────────────────────────────────────────────────┤
│ Medium: Copper twisted pair (Cat6 Ethernet cable)       │
│ Signal: Electrical voltage pulses                       │
│ Speed: 1000 Mbps (1 Gigabit)                            │
│ Encoding: 8B/10B (8 bits → 10 bits for clock sync)     │
│ Signal levels: +0.5V = 1 bit, -0.5V = 0 bit           │
│ Line rate: 1,250 Mbaud (million baud)                  │
└─────────────────────────────────────────────────────────┘
```

**Layer 2: Data Link**
```
Ethernet Frame Structure:
┌──────────────────────────────────────────────────────┐
│ Preamble (7 bytes):        10101010 10101010 ...     │
│                            (clock synchronization)   │
│ Start Frame Delimiter (1): 10101011                  │
│ Dest MAC (6 bytes):        11-22-33-44-55-66         │
│                            (router MAC address)      │
│ Source MAC (6 bytes):      AA-BB-CC-DD-EE-FF         │
│                            (your NIC MAC address)    │
│ Type (2 bytes):            0x0800 (IPv4)             │
│ Payload:                   [IP Packet - see below]   │
│ FCS Checksum (4 bytes):    0x1A2B3C4D                │
│ Frame size: 1514 bytes max (MTU = 1500)              │
└──────────────────────────────────────────────────────┘
```

**Layer 3: Network (IP)**
```
IPv4 Packet Structure:
┌──────────────────────────────────────────────────┐
│ Version (4 bits):         4                      │
│ Header Length (4 bits):   5 (20 bytes)           │
│ Type of Service (8 bits): 0x00 (normal)          │
│ Total Length (16 bits):   460 bytes              │
│ Identification (16 bits): 0x1234                 │
│ Flags (3 bits):           010 (don't fragment)   │
│ Fragment Offset (13 b):   0                      │
│ TTL (8 bits):             64 hops remaining      │
│ Protocol (8 bits):        06 (TCP)               │
│ Header Checksum (16 b):   0xABCD                 │
│ Source IP (32 bits):      192.168.1.100          │
│ Dest IP (32 bits):        93.184.216.34          │
│ Options (variable):       (usually none)         │
│ Payload:                  [TCP Segment]          │
└──────────────────────────────────────────────────┘
```

**Layer 4: Transport (TCP)**
```
TCP Segment Structure:
┌──────────────────────────────────────────────────┐
│ Source Port (16 bits):    52135                  │
│ Dest Port (16 bits):      443 (HTTPS)            │
│ Sequence Number (32 bits): 1000000001            │
│ Acknowledgment (32 b):     2000000001            │
│ Data Offset (4 bits):      5 (20 bytes header)   │
│ Flags (8 bits):            ACK, PSH              │
│   - URG: Not set                                 │
│   - ACK: Set (acknowledgment valid)              │
│   - PSH: Set (push to application)               │
│   - RST: Not set                                 │
│   - SYN: Not set (not connection start)          │
│   - FIN: Not set (not closing)                   │
│ Window Size (16 bits):     65535 bytes           │
│ Checksum (16 bits):        0x5678                │
│ Urgent Pointer (16 bits):  0 (not used)          │
│ Options (variable):        (usually none)        │
│ Payload:                   [TLS/SSL Data]        │
└──────────────────────────────────────────────────┘
```

**Layer 5-7: Session/Presentation/Application**
```
TLS/SSL Handshake (Presentation/Session):
┌──────────────────────────────────────────────────┐
│ TLS Record Header:                               │
│   Type: Handshake (0x16)                         │
│   Version: TLS 1.2 (0x0303)                      │
│   Length: 150 bytes                              │
│                                                   │
│ Handshake Data:                                  │
│   Message Type: Client Hello (0x01)              │
│   Length: 146 bytes                              │
│   Version: TLS 1.2                               │
│   Random: 32 random bytes (nonce)                │
│   Session ID: (empty for new)                    │
│   Cipher Suites: [list of supported]             │
│   Compression Methods: null                      │
│   Extensions: [server name, supported versions]  │
└──────────────────────────────────────────────────┘

HTTP Request (Application):
┌──────────────────────────────────────────────────┐
│ GET /index.html HTTP/1.1                         │
│ Host: example.com                                │
│ User-Agent: Mozilla/5.0                          │
│ Accept: text/html,application/xhtml+xml          │
│ Accept-Language: en-US,en;q=0.9                  │
│ Connection: keep-alive                           │
│ Upgrade-Insecure-Requests: 1                     │
│                                                   │
│ [Body is empty for GET request]                  │
└──────────────────────────────────────────────────┘
```

**Complete Packet Visualization:**

```
Physical (bits on wire):
10101010 10101010 10101011 [MAC data] [IP] [TCP] [TLS] [HTTP] [Checksum]

Data Link (frame):
[Ethernet Hdr] [IP Packet] [FCS]

Network (packet):
[IP Hdr] [TCP Segment] 

Transport (segment):
[TCP Hdr] [TLS Record]

Session/App (data):
[TLS Handshake] → [HTTP Request]
```

**What Happens at Each Layer:**

1. **Layer 1**: Electrical signals transmitted
2. **Layer 2**: Ethernet frame added, sent on local link
3. **Layer 3**: IP routing decision (gateway or direct?)
4. **Layer 4**: TCP port 52135→443 established
5. **Layer 5**: TLS session negotiation
6. **Layer 6**: TLS encryption applied
7. **Layer 7**: HTTP GET request sent

**Key Information at Each Layer:**

| Layer | Critical Info | Failure Result |
|-------|--------------|-----------------|
| 1 | Signal integrity | Cable error |
| 2 | Destination MAC | Can't reach local |
| 3 | Destination IP | Can't reach network |
| 4 | Destination port | Connection refused |
| 5-7 | Application protocol | Service not responding |

---

### Exercise 9: Design a Network Using OSI Model

**Solution:**

**Small Business Network Design (20 employees, 3 servers, printers)**

**Layer 1: Physical Design**

```
Physical Infrastructure:
├─ Media: Cat6A twisted pair Ethernet cables
│  • Supports up to 10 Gbps
│  • Future proof for upgrades
│
├─ Topology: Star topology
│  • All devices connect to central switch
│  • Easy to manage and troubleshoot
│  • Reliable
│
├─ Devices:
│  ├─ Main Distribution Frame (MDF)
│  │  └─ Cat6A cabling to all areas
│  ├─ Wall plates: RJ45 jacks in each office
│  ├─ Patch cables: From wall to devices
│  └─ Racks: Equipment mounting
│
└─ Layout:
   Server Room
   └─ [Core Switch] [Firewall] [Router]
            ↓
   Building Main Floor
   ├─ [Office 1: Desktop PC]
   ├─ [Office 2: Desktop PC]
   ├─ [Office 3: Laptop]
   ├─ [Printer 1]
   └─ [Printer 2]
   
   Building 2nd Floor
   ├─ [Office 4: Desktop]
   ├─ [Office 5: Desktop]
   └─ [Printer 3]
```

**Physical Requirements:**
- Run Cat6A cables from server room to each office
- Install wall plates with RJ45 jacks
- Label all cables for troubleshooting
- Cable management with conduits
- Redundant power for critical equipment

---

**Layer 2: Data Link Design**

```
Switches & VLAN Design:
┌─────────────────────────────────────────┐
│ Core Switch (Layer 3 capable)           │
│ - 24-port Gigabit Ethernet              │
│ - VLAN support                          │
│ - Managed switch (not consumer)         │
└─────────────────────────────────────────┘
        ↓
┌─────────────────────────────────────────┐
│ Access Switch (Floor 1)                 │
│ - 12-port Gigabit                       │
│ - Connects PCs, printers                │
└─────────────────────────────────────────┘

MAC Address Management:
├─ Each device has unique MAC
│  ├─ Servers: xx-xx-xx-aa-00-01 through 03
│  ├─ Printers: xx-xx-xx-bb-00-01 through 05
│  ├─ PCs: xx-xx-xx-cc-00-01 through 20
│  └─ Network gear: xx-xx-xx-dd-00-*
│
└─ ARP entries for management devices
   (static ARP for servers and printers)
```

**VLAN Configuration:**
```
VLAN 10: Employees (192.168.10.0/24)
├─ 20 PCs (DHCP)
├─ Gateway: 192.168.10.1
└─ Subnet: 192.168.10.1-254

VLAN 20: Servers (192.168.20.0/25)
├─ Server 1: 192.168.20.10
├─ Server 2: 192.168.20.11
├─ Server 3: 192.168.20.12
└─ Gateway: 192.168.20.1

VLAN 30: Printers (192.168.30.0/25)
├─ Printer 1: 192.168.30.10
├─ Printer 2: 192.168.30.11
├─ Printer 3: 192.168.30.12
└─ Gateway: 192.168.30.1

VLAN 99: Management (192.168.99.0/25)
├─ Switch management IPs
├─ Router management IP
└─ Firewall management IP
```

---

**Layer 3: Network Design**

```
IP Addressing Scheme:
Network: 192.168.0.0/16 (65,536 addresses)

Subnets:
├─ 192.168.10.0/24 - Employees (256 addresses)
├─ 192.168.20.0/25 - Servers (128 addresses)
├─ 192.168.30.0/25 - Printers (128 addresses)
└─ 192.168.99.0/25 - Management (128 addresses)

Routing Configuration:
┌──────────────────────────────────────┐
│ Firewall/Router                      │
│ WAN Interface: ISP IP (public)        │
│ LAN Interface: 192.168.0.1 (gateway) │
│ Routing:                             │
│ - Default route → ISP                │
│ - Subnets → Local               │
└──────────────────────────────────────┘

Gateway Configuration:
├─ All employees: Default gateway = 192.168.10.1
├─ All servers: Default gateway = 192.168.20.1
└─ All printers: Default gateway = 192.168.30.1
```

**DHCP Configuration:**
```
DHCP Pools:
├─ Employees (VLAN 10)
│  Pool: 192.168.10.100-200
│  Lease: 24 hours
│  Gateway: 192.168.10.1
│  DNS: 192.168.10.10 (local), 8.8.8.8 (backup)
│
└─ Printers (VLAN 30)
   Pool: 192.168.30.50-100 (optional)
   Static: Printers at .10, .11, .12
```

---

**Layer 4: Transport Design**

```
Firewall Rules:

Inbound (from Internet):
├─ Port 443 (HTTPS) → Server 192.168.20.10 (web)
├─ Port 25 (SMTP) → Server 192.168.20.11 (mail)
└─ All other → Block

Outbound (Internal):
├─ Employees (VLAN 10) → Any (internet)
├─ Servers (VLAN 20) → Each other + DNS
├─ Printers (VLAN 30) → Local network only (blocked to WAN)
└─ Management (VLAN 99) → All internal

Inter-VLAN Rules:
├─ Employees → Servers: Ports 443, 3306 (DB), 5432 (DB)
├─ Employees → Printers: Ports 9100 (LPD), 515 (LPR)
├─ Servers → Servers: All ports
├─ Servers → Internet: Ports 53 (DNS), 123 (NTP), 25 (SMTP out)
└─ Printers → Internet: Blocked
```

**Port Management:**
```
Well-known Ports in Use:
├─ SSH (22): Server management
├─ SMTP (25): Email sending
├─ DNS (53): DNS queries
├─ HTTP (80): Web (may redirect to 443)
├─ HTTPS (443): Web server
├─ MySQL (3306): Database
├─ PostgreSQL (5432): Database
├─ IMAP (143): Email
├─ LPD/LPR (515): Printing
└─ SNMP (161): Network monitoring
```

---

**Layer 5-7: Session/Presentation/Application Design**

```
Services & Applications:

Web Server (Server 1):
├─ Service: Apache/Nginx HTTPS
├─ Port: 443 (TLS 1.2+)
├─ Authentication: AD (Active Directory)
├─ Session timeout: 1 hour
└─ Data encryption: TLS, AES-256

Database Server (Server 2):
├─ Service: MySQL/PostgreSQL
├─ Port: 3306/5432 (internal only)
├─ Authentication: Database users
├─ Encryption: SSL connections
└─ Backup: Daily, stored offsite

Email Server (Server 3):
├─ Service: Postfix/Exim SMTP
├─ Port: 25 (sending), 993 (IMAP encrypted)
├─ Authentication: LDAP (Active Directory)
├─ Encryption: TLS required
└─ Spam filtering: SpamAssassin

Client Applications:
├─ Web Browser: HTTPS (TLS 1.2+)
├─ Email Client: IMAP over TLS
├─ Database Client: MySQL client (SSL)
├─ SMB/CIFS: File sharing (encrypted)
└─ RDP: Remote desktop (encrypted)

Authentication:
├─ Active Directory (Kerberos)
├─ All services integrated with AD
├─ Password policy: 12+ chars, 90-day rotation
├─ MFA: For remote access (VPN)
└─ Session logging: All access recorded
```

---

**Security Summary:**

```
Multi-layer Security:

Layer 1 (Physical):
├─ Locked server room
├─ Cable locks on equipment
└─ Backup power (UPS)

Layer 2 (Data Link):
├─ VLAN separation
├─ Port security (limit MACs per port)
└─ STP (Spanning Tree) enabled

Layer 3 (Network):
├─ Access Control Lists (ACLs)
├─ No direct employee→server routes
├─ DNS filtering (block malicious sites)
└─ Subnet isolation

Layer 4 (Transport):
├─ Firewall rule enforcement
├─ Stateful inspection
├─ Intrusion detection
└─ DDoS protection

Layer 5-7 (Application):
├─ TLS/SSL encryption everywhere
├─ Authentication (LDAP/AD)
├─ Application firewalls
├─ Logging and auditing
└─ Regular security patches
```

---

### Exercise 10: Create OSI Layer Troubleshooting Guide

**Solution:**

**Complete OSI Troubleshooting Guide**

```markdown
# OSI Layer Troubleshooting Guide

## Layer 1: Physical

**Problems that indicate Layer 1:**
- "Network disconnected" message
- No link lights on network port
- Computer won't see network at all
- "Device driver unavailable" error
- Intermittent disconnections (cable issue)

**Diagnostic Commands:**
- `Get-NetAdapter` (check Status: Up/Down)
- `Get-NetAdapter | Format-List` (detailed info)
- `ipconfig /all` (if adapter doesn't appear = Layer 1)
- Visual inspection (loose cable, bent pin)

**Common Fixes:**
- Reconnect Ethernet cable firmly
- Replace damaged cable
- Try different port on switch
- Update network adapter driver
- Disable and re-enable adapter (`Get-NetAdapter | Enable/Disable`)
- Check for physical damage (cable, port)
- Restart computer

---

## Layer 2: Data Link

**Problems that indicate Layer 2:**
- Can't reach any local devices (ARP fails)
- "MAC address not found" errors
- Some ports on switch not working
- Duplicate MAC addresses on network
- VLAN configuration issues

**Diagnostic Commands:**
- `arp -a` (check for devices on network)
- `ipconfig /all` (verify Physical Address/MAC)
- `arp -a | findstr <device-ip>` (find device MAC)
- `Get-NetAdapter | Format-List` (MAC address info)

**Common Fixes:**
- Verify switch port is enabled and not blocked
- Check for duplicate MAC addresses
- Restart network adapter (power cycle)
- Update NIC drivers
- Check VLAN configuration
- Replace defective network cable
- Test with different port on switch

---

## Layer 3: Network

**Problems that indicate Layer 3:**
- Can't reach remote networks/internet
- "No route to host" errors
- Ping works to gateway but not beyond
- Default gateway unreachable
- Routing table doesn't include destinations

**Diagnostic Commands:**
- `ipconfig /all` (check IP, gateway, subnet)
- `ping <gateway-ip>` (test local gateway)
- `ping 8.8.8.8` (test internet connectivity)
- `route print` (view routing table)
- `tracert <destination>` (show hops to destination)
- `Get-NetRoute` (PowerShell routing info)

**Common Fixes:**
- `ipconfig /renew` (get new IP via DHCP)
- Verify gateway address is correct
- Add static route if needed
- Check gateway is powered on and responding
- Disable Windows Firewall temporarily (test)
- Verify subnet mask is correct
- Check if on correct network segment

---

## Layer 4: Transport

**Problems that indicate Layer 4:**
- Application "can't connect" despite network working
- "Connection refused" errors
- Firewall blocking messages
- Specific ports unreachable
- Telnet fails to port but ping works

**Diagnostic Commands:**
- `netstat -an` (view all connections)
- `netstat -ano` (include process IDs)
- `netstat -an | findstr :443` (check specific port)
- `netstat -an | findstr LISTENING` (open ports)
- `telnet <ip> <port>` (test port connectivity)
- `Test-NetConnection -ComputerName <ip> -Port 443` (PowerShell)

**Common Fixes:**
- Disable Windows Firewall temporarily
- Add firewall exception for application
- Check if service is running (`Get-Service`)
- Verify listening port is correct
- Check application configuration
- Restart the service
- Allow port through firewall rules

---

## Layer 5: Session

**Problems that indicate Layer 5:**
- Connection drops after period of time
- Session state lost unexpectedly
- Login/logout issues
- Can't maintain persistent connections
- VPN disconnects frequently

**Diagnostic Commands:**
- `netstat -an` (check TIME_WAIT, CLOSE_WAIT states)
- `Get-NetTCPConnection | Group-Object State` (connection states)
- Check application logs
- Monitor connection duration
- Test with different application

**Common Fixes:**
- Check session timeout settings
- Increase connection timeout values
- Restart application
- Check application logging
- Verify persistent connection settings
- Update application/OS

---

## Layer 6: Presentation

**Problems that indicate Layer 6:**
- "SSL/TLS error" messages
- "Untrusted certificate" warnings
- Data appears corrupted or garbled
- File format errors
- Decryption failures

**Diagnostic Commands:**
- Browser developer tools (F12)
- Check certificate validity
- Test with different client application
- `openssl s_client -connect <server>:443` (test TLS)

**Common Fixes:**
- Update browser/application
- Clear browser cache and cookies
- Import correct SSL certificate
- Disable proxy temporarily
- Check time synchronization (important for TLS)
- Update security certificates

---

## Layer 7: Application

**Problems that indicate Layer 7:**
- Application starts but gives errors
- Wrong data displayed
- Service unavailable messages
- Authentication failures
- Application-specific error messages

**Diagnostic Commands:**
- Check application logs
- Check service status: `Get-Service <service-name>`
- Test with different client
- Check database connectivity
- Monitor system resources (CPU, RAM)
- Review application configuration

**Common Fixes:**
- Restart application/service
- Check application configuration
- Verify database connectivity
- Check credentials/permissions
- Update application
- Check system resources (disk space, RAM)
- Verify service dependencies are running

```

**Troubleshooting Decision Tree:**

```
Network problem occurs
│
├─ Can you see network adapter?
│  └─ NO → LAYER 1 PROBLEM
│         └─ Check cable, driver, port
│  └─ YES → Continue
│
├─ Can you see device in ARP table? (arp -a)
│  └─ NO → LAYER 2 PROBLEM
│         └─ Check MAC, switch port, VLAN
│  └─ YES → Continue
│
├─ Can you ping gateway? (ping <gateway-ip>)
│  └─ NO → LAYER 3 PROBLEM
│         └─ Check IP, gateway, subnet, routing
│  └─ YES → Continue
│
├─ Can you reach port? (telnet <ip> <port>)
│  └─ NO → LAYER 4 PROBLEM
│         └─ Check firewall, service running, port
│  └─ YES → Continue
│
└─ Can application connect?
   └─ NO → LAYER 5-7 PROBLEM
          └─ Check app logs, TLS, authentication, config
```

**Quick Troubleshooting Checklist:**

```
□ Layer 1: Cable connected? Lights on? Status: Up?
□ Layer 2: Device in ARP table? MAC address correct?
□ Layer 3: Have IP address? Can ping gateway? Route exists?
□ Layer 4: Port listening? Firewall allowing? Service running?
□ Layer 5-7: Application running? Configuration correct? Credentials?
```

This guide serves as quick reference for diagnosing which layer has a problem.
```

---

## Summary

The OSI model provides a systematic way to troubleshoot networks:

1. **Start at bottom (Layer 1)**: Can you reach the physical network?
2. **Move up one layer at a time**: Only move up when layer below works
3. **Use appropriate tools**: Each layer has specific commands
4. **Fix identified layer**: Apply fixes for that specific layer
5. **Re-test**: Verify fix works before moving to next layer

This layered approach makes complex network problems manageable and solvable.

