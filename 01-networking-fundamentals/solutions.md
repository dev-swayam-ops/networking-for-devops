# Solutions: Networking Fundamentals

Complete solutions and detailed explanations for all exercises.

## Easy Exercises

### Exercise 1: Identify Your Network Components

**Command:**
```powershell
ipconfig /all
```

**Solution:**

**Sample Output Interpretation:**
```
Ethernet adapter Ethernet:
   IPv4 Address: 192.168.1.100
   Subnet Mask: 255.255.255.0
   Default Gateway: 192.168.1.1
   DNS Servers: 8.8.8.8, 8.8.4.4
   DHCP Enabled: Yes
```

**Answers:**

| Component | Example | Meaning |
|-----------|---------|---------|
| IP Address | 192.168.1.100 | Your computer's address on the local network |
| Subnet Mask | 255.255.255.0 | Defines which part of IP is network (192.168.1) vs host (100) |
| Default Gateway | 192.168.1.1 | The router - where packets go to leave the local network |
| DNS Servers | 8.8.8.8, 8.8.4.4 | Servers that translate domain names to IPs |
| DHCP | Yes | Automatic IP assignment by router |

**Key Points:**
- DHCP = Dynamic Host Configuration Protocol (automatic IP)
- Static = Manually assigned IP (doesn't change)
- Subnet mask 255.255.255.0 = /24 notation (covers 256 addresses)
- Gateway is usually .1 in private networks (192.168.1.1, 10.0.0.1)

**Why This Matters:**
- Understanding your network config is foundation for troubleshooting
- IP address is how devices find each other
- Gateway is how you communicate outside your network
- DNS is how you use domain names instead of memorizing IPs

---

### Exercise 2: Understand Network Types

**Solution:**

| Network Type | Range | Purpose | Example |
|-------------|-------|---------|---------|
| LAN (Local Area Network) | Local (same building) | Connect devices close together | Your home, office |
| WAN (Wide Area Network) | Wide (cities/countries) | Connect distant networks | The internet |
| VLAN (Virtual LAN) | Logical grouping | Segment network logically | Accounting dept on different VLAN |
| MAN (Metropolitan Area Network) | City | Connect across a city | City-wide university network |

**Network Diagram:**
```
┌──────────────────────────────────────────────────────────┐
│  Your Computer (192.168.1.100)                           │
│  • On Local Network (LAN)                                │
└────────────────┬─────────────────────────────────────────┘
                 │
                 │ Ethernet/Wi-Fi
                 │
         ┌───────┴────────┐
         │  Home Router   │
         │  192.168.1.1   │
         └───────┬────────┘
                 │
              Internet (WAN)
                 │
         ┌───────┴────────┐
         │  ISP Router    │
         │  203.0.113.1   │
         └───────┬────────┘
                 │
         ┌───────┴────────┐
         │  Web Server    │
         │  142.250.80.46 │
         └────────────────┘
```

**Key Distinctions:**

- **LAN**: All devices share same subnet (192.168.1.0/24)
  - Fast (local cables)
  - Limited distance
  - Don't need routers between devices
  
- **WAN**: Devices on different networks
  - Slower (internet involved)
  - Unlimited distance
  - Routers connect different networks

- **VLAN**: Logical LAN over physical WAN
  - Same physical switch but separate networks logically
  - Used for security and management

---

### Exercise 3: Test DNS Resolution

**Commands:**
```powershell
nslookup google.com
nslookup github.com
nslookup amazon.com
nslookup thisdomaindoesntexist123.com
```

**Solution:**

**Sample Output - Successful Resolution:**
```
Server: 8.8.8.8
Address: 8.8.8.8#53

Non-authoritative answer:
Name: google.com
Address: 142.250.80.46
```

**Sample Output - Failed Resolution:**
```
Server: 8.8.8.8
Address: 8.8.8.8#53

*** Can't find thisdomaindoesntexist123.com: Non-existent domain
```

**Answers to Questions:**

1. **Did different domains resolve to different IPs?**
   - Yes! Each domain has its own IP address
   - Example: google.com → 142.250.80.46
   - Example: github.com → 140.82.113.3
   
2. **What happens when you query a non-existent domain?**
   - Error: "Can't find ... Non-existent domain"
   - DNS server can't find it in its records
   - Usually means the domain was never registered

3. **Why is DNS important?**
   - Without DNS, you'd need to remember IPs (142.250.80.46)
   - With DNS, you just remember "google.com"
   - Users prefer domain names to IP addresses
   - Allows websites to change IPs without breaking user access

**DNS Basics:**
- DNS = Domain Name System
- Translates human-readable names → machine-readable IP addresses
- Uses UDP port 53
- Results are cached locally for speed

---

### Exercise 4: Trace Network Paths

**Commands:**
```powershell
tracert google.com
tracert cloudflare.com
```

**Solution:**

**Sample Output:**
```
Tracing route to google.com [142.250.80.46]
over a maximum of 30 hops:

  1     2 ms     2 ms     2 ms  192.168.1.1 (router)
  2    12 ms    12 ms    12 ms  10.0.0.1 (ISP)
  3    25 ms    24 ms    24 ms  203.0.113.5 (ISP backbone)
  4    35 ms    34 ms    36 ms  203.0.113.10 (ISP backbone)
  5    45 ms    44 ms    46 ms  198.51.100.5 (Internet core)
  6    55 ms    54 ms    56 ms  198.51.100.10 (Internet core)
  7    65 ms    64 ms    66 ms  142.250.80.46 (Google datacenter)

Trace complete.
```

**Answers to Questions:**

1. **How many hops to google.com?** 7 hops
2. **How many hops to cloudflare.com?** Varies (often 6-10)
3. **Which is closer (fewer hops)?** Cloudflare is often closer (fewer hops)
4. **What is the latency at hop 3?** ~24-25 ms

**Understanding Tracert Output:**

| Column | Meaning |
|--------|---------|
| 1-3 | Three ping packets sent to each hop |
| 2ms | Time for packet to reach hop 1 and return |
| 192.168.1.1 | IP address of the router at that hop |

**What Each Hop Represents:**
- Hop 1: Your home router (always fast, 1-5ms)
- Hop 2-4: Your ISP's network (12-36ms)
- Hop 5-6: Internet backbone (45-66ms)
- Hop 7: Destination server

**Why This Matters:**
- Identifies slow parts of the network path
- Shows routing inefficiencies
- Helps with troubleshooting (if slow hop 3, ask ISP)
- Latency increases with each hop (usually)

---

### Exercise 5: Monitor Your Network Connections

**Commands:**
```powershell
netstat -an | findstr ESTABLISHED
```

**Sample Output:**
```
Proto  Local Address          Foreign Address        State
TCP    192.168.1.100:52134    142.250.80.46:443     ESTABLISHED
TCP    192.168.1.100:52135    142.251.32.142:443    ESTABLISHED
TCP    192.168.1.100:52136    172.217.169.46:443    ESTABLISHED
TCP    127.0.0.1:50000        127.0.0.1:50001       ESTABLISHED
```

**Answers to Questions:**

1. **How many established connections?** In this example: 4
   - Real number varies based on open applications

2. **What are the different remote ports?**
   - Port 443 (HTTPS) - secure web browsing
   - Port 80 (HTTP) - unsecure web
   - Port 25 (SMTP) - email sending
   - Port 3306 (MySQL) - databases

3. **What application uses port 443?**
   - Port 443 is HTTPS (secure web)
   - Used by web browsers (Chrome, Firefox, Edge)
   - Provides encryption for web traffic

**Understanding the Output:**

| Component | Example | Meaning |
|-----------|---------|---------|
| Local Address | 192.168.1.100 | Your computer's IP |
| Local Port | 52134 | Random port your app uses |
| Foreign Address | 142.250.80.46 | Remote server IP |
| Foreign Port | 443 | Remote server's port |
| State | ESTABLISHED | Connection is active |

**Ephemeral Ports:**
- Ports 49152-65535 are ephemeral (temporary)
- OS assigns random port from this range for outgoing connections
- Allows your computer to have many simultaneous connections

---

## Medium Exercises

### Exercise 6: Understand TCP vs UDP

**Solution:**

| Aspect | TCP | UDP |
|--------|-----|-----|
| Reliability | Guaranteed delivery | Best effort only |
| Speed | Slower (error checking) | Faster (no overhead) |
| Error Checking | Yes, checksums | Limited |
| Example Usage | Web, email, file transfer | Video streaming, gaming, DNS |
| Port Examples | 80, 443, 22, 3306 | 53, 5353, 5004 |

**TCP Characteristics:**
- **Reliable**: Lost packets are retransmitted
- **Ordered**: Packets arrive in correct order
- **Flow control**: Sender waits for receiver ready
- **Connection-based**: Must establish connection first (3-way handshake)
- **Slower**: Extra overhead for reliability

**UDP Characteristics:**
- **Fast**: No error checking overhead
- **Connectionless**: No setup needed
- **Unreliable**: Doesn't retry lost packets
- **Good for**: Real-time apps that tolerate loss
- **Stateless**: No connection state maintained

**Applications That Use TCP:**
1. HTTP/HTTPS (web browsing) - port 80, 443
2. SMTP (email sending) - port 25
3. SSH (secure shell) - port 22
4. FTP (file transfer) - port 21
5. MySQL (database) - port 3306

**Applications That Use UDP:**
1. DNS (domain resolution) - port 53
2. VOIP (voice calls) - port 5004
3. Video streaming - various ports
4. Online gaming - various ports
5. Multicast/broadcast - various ports

**When to Use Each:**

Use **TCP** when:
- Data accuracy is critical
- Order matters
- You can tolerate some latency
- Example: Downloading files, emails, web pages

Use **UDP** when:
- Speed is critical
- Loss of some data is acceptable
- Real-time communication needed
- Example: Video calls, live gaming, live video streaming

**Why HTTP Uses TCP, DNS Uses UDP:**
- **HTTP (TCP)**: Every web page request matters; can't lose data
- **DNS (UDP)**: Fast name resolution; if one query lost, can retry; usually cached

---

### Exercise 7: Analyze Packet Structure

**Solution:**

**Step 1: Find a Connection**
```powershell
netstat -an | findstr ESTABLISHED
```

Example connection: `192.168.1.100:52134 → 142.250.80.46:443`

**Step 2: Packet Structure for This Connection**

```
┌─────────────────────────────────────────────┐
│ Ethernet Frame Header (Layer 2)             │
├─────────────────────────────────────────────┤
│ Destination MAC: 11-22-33-44-55-66          │ → Router MAC
│ Source MAC: AA-BB-CC-DD-EE-FF               │ → Your device MAC
│ Type: IPv4 (0x0800)                         │
├─────────────────────────────────────────────┤
│ IP Header (Layer 3)                         │
├─────────────────────────────────────────────┤
│ Version: 4                                   │
│ Header Length: 20 bytes                      │
│ Total Length: 40 + data bytes               │
│ TTL: 64                                      │ → Hops allowed
│ Protocol: TCP (6)                            │
│ Source IP: 192.168.1.100                    │
│ Destination IP: 142.250.80.46               │
├─────────────────────────────────────────────┤
│ TCP Header (Layer 4)                        │
├─────────────────────────────────────────────┤
│ Source Port: 52134                          │ → Your app's port
│ Destination Port: 443                       │ → HTTPS server
│ Sequence Number: 1000000001                 │ → Packet ordering
│ Acknowledgment: 2000000001                  │ → "Got your packet"
│ Flags: ACK, PSH                             │
│ Window Size: 65535                          │ → How much more can send
├─────────────────────────────────────────────┤
│ Application Data (Layer 5-7)                │
├─────────────────────────────────────────────┤
│ TLS/SSL Encrypted Handshake or Data         │
│ (Content may vary, browser sent GET request)│
│ GET /index.html HTTP/1.1                    │
│ Host: example.com                           │
│ ...                                         │
└─────────────────────────────────────────────┘
```

**How Headers are Added/Removed:**

1. **Application Layer**: Browser creates HTTP request "GET /"
2. **TLS Layer**: Encrypts the HTTP data
3. **TCP Layer**: Adds TCP header (port 52134→443)
4. **IP Layer**: Adds IP header (192.168.1.100 → 142.250.80.46)
5. **Ethernet Layer**: Adds Ethernet header (MAC addresses)
6. **Physical Layer**: Converts to electrical signals

**Return Path (Response):**
1. **Physical**: Electrical signals received
2. **Ethernet**: Extracts frame, checks MAC address
3. **IP**: Extracts packet, checks destination IP
4. **TCP**: Extracts segment, matches port 443→52134
5. **TLS**: Decrypts the data
6. **Application**: Passes HTTP response to browser

**Encapsulation Process:**
```
[HTTP Data: 1000 bytes]
→ [TCP Header + Data: 1020 bytes]
→ [IP Header + TCP Header + Data: 1040 bytes]
→ [Ethernet Header + IP + TCP + Data + Checksum: 1054 bytes]
→ Physical transmission (electrical signals)
```

---

### Exercise 8: Calculate Subnet Information

**Solution:**

| IP Address | Subnet Mask | Network Address | Broadcast | First Host | Last Host | # Hosts |
|-----------|-------------|-----------------|-----------|-----------|----------|---------|
| 192.168.1.100 | 255.255.255.0 | 192.168.1.0 | 192.168.1.255 | 192.168.1.1 | 192.168.1.254 | 254 |
| 10.0.0.50 | 255.255.255.0 | 10.0.0.0 | 10.0.0.255 | 10.0.0.1 | 10.0.0.254 | 254 |
| 172.16.5.130 | 255.255.255.128 | 172.16.5.128 | 172.16.5.255 | 172.16.5.129 | 172.16.5.254 | 126 |

**How to Calculate:**

**For 192.168.1.100 with 255.255.255.0:**

1. **Network Address** (all host bits = 0):
   - Subnet mask: 255.255.255.0 = 11111111.11111111.11111111.00000000
   - IP: 192.168.1.100 = 11000000.10101000.00000001.01100100
   - Network: 192.168.1.0

2. **Broadcast Address** (all host bits = 1):
   - Same network: 192.168.1.
   - All host bits 1: 255
   - Result: 192.168.1.255

3. **Host Range**:
   - First: Network + 1 = 192.168.1.1
   - Last: Broadcast - 1 = 192.168.1.254
   - Range: 192.168.1.1 - 192.168.1.254

4. **Number of Hosts**:
   - /24 = 256 total addresses
   - Subtract: 1 network + 1 broadcast = 254 usable hosts

**For 172.16.5.130 with 255.255.255.128 (/25):**

1. **Subnet Mask: 255.255.255.128**
   - Binary: 11111111.11111111.11111111.10000000
   - This divides the last octet in half

2. **Determine Subnet**:
   - 130 in binary: 10000010
   - Mask last octet: 10000000 = 128
   - Network: 172.16.5.128

3. **Broadcast**:
   - All host bits 1: 172.16.5.11111111 = 172.16.5.255

4. **Valid Range**: 172.16.5.129 - 172.16.5.254
   - Total: 128 addresses
   - Usable: 126 (remove network and broadcast)

**Subnet Mask Quick Reference:**

| Mask | CIDR | Hosts | Use Case |
|------|------|-------|----------|
| 255.255.255.0 | /24 | 254 | Standard home/small office |
| 255.255.255.128 | /25 | 126 | Subnet a /24 in half |
| 255.255.255.192 | /26 | 62 | Divide /24 into quarters |
| 255.255.255.224 | /27 | 30 | Small department network |
| 255.255.255.240 | /28 | 14 | Very small network |
| 255.255.0.0 | /16 | 65,534 | Large enterprise network |

---

### Exercise 9: Map Your Local Network

**Commands:**
```powershell
ipconfig /all
ping 192.168.1.1
arp -a
```

**Solution:**

**Step 1: Get Your Network**
```
IP: 192.168.1.100
Subnet: 255.255.255.0
Network: 192.168.1.0/24
Range: 192.168.1.1 - 192.168.1.254
```

**Step 2: Ping Devices**
```
ping 192.168.1.1   → Router (usually responds)
ping 192.168.1.254 → Broadcast (may respond)
ping 192.168.1.10  → Device (if present)
```

**Step 3: Check ARP**
```powershell
arp -a
```

**Sample ARP Output:**
```
Interface: 192.168.1.100 --- 0x6
  Internet Address      Physical Address      Type
  192.168.1.1          AA-BB-CC-DD-EE-FF    dynamic
  192.168.1.25         11-22-33-44-55-66    dynamic
  192.168.1.50         22-33-44-55-66-77    dynamic
  192.168.1.100        (This PC)
  192.168.1.255        FF-FF-FF-FF-FF-FF    static
```

**Answers:**

1. **How many devices responded to ping?**
   - Usually 3-10 devices
   - Depends on how many devices are on the network
   - Some devices ignore ping (security)

2. **What is the MAC address of your router?**
   - First entry in ARP table (usually 192.168.1.1)
   - Format: AA-BB-CC-DD-EE-FF

3. **Are there any unknown MAC addresses?**
   - First 3 bytes = manufacturer (OUI)
   - You can look up vendor codes online
   - Example: 00-1A-2B = Cisco

**MAC Address Lookup:**
- Online tools: macvendors.com, oui-lookup.com
- Command line: `getmac /v` (shows local MAC)

---

### Exercise 10: Design a Simple Network

**Solution:**

**Network Requirements:**
- 50 employees
- 10 servers
- 5 printers
- Wi-Fi network
- Internet connection

**Proposed Solution:**

**1. IP Addressing Scheme**
```
Use private range: 10.0.0.0/16

Subnets:
├── 10.0.1.0/24   → Employees (Wired) - 254 hosts
├── 10.0.2.0/24   → Employees (Wi-Fi) - 254 hosts
├── 10.0.3.0/24   → Servers - 254 hosts
├── 10.0.4.0/25   → Printers - 126 hosts
└── 10.0.5.0/25   → Management - 126 hosts
```

**2. Network Components**

| Component | Purpose | Model |
|-----------|---------|-------|
| Firewall | Border security | Fortinet FortiGate |
| Router | Connect to ISP | Cisco ASR 1000 |
| Core Switch | Connect all subnets | Cisco Catalyst 6500 |
| Access Switch | Employee computers | Cisco 2960X (1-2) |
| Access Points | Wi-Fi | Cisco Meraki (3-4) |
| Server Switch | Servers | Cisco Catalyst 3850 |
| DNS Server | Name resolution | Linux/Windows |
| DHCP Server | IP assignment | Windows/Linux |

**3. Network Diagram**

```
                    ┌─────────────┐
                    │   Internet  │
                    │  203.x.x.x  │
                    └──────┬──────┘
                           │
                      ┌────┴────┐
                      │ Firewall│
                      └────┬────┘
                           │
                    ┌──────┴──────┐
                    │   Router    │
                    │ 10.0.0.1    │
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    │ Core Switch │
                    └──────┬──────┘
         ┌──────────────────┼──────────────────┐
         │                  │                  │
    ┌────┴───┐        ┌────┴────┐      ┌─────┴──┐
    │Access  │        │ Servers │      │Access  │
    │Switch  │        │ Switch  │      │Points  │
    │10.0.1.0│        │10.0.3.0 │      │(Wi-Fi) │
    └────┬───┘        └────┬────┘      └─────┬──┘
         │                 │                  │
    ┌────┴──────┐    ┌─────┴─────┐   ┌──────┴──────┐
    │Employees  │    │Servers    │   │Employees    │
    │(Wired)    │    │10 servers │   │(Wi-Fi)      │
    │40 PCs     │    │Printers   │   │10 laptops   │
    │10.0.1.10+ │    │10.0.4.10+ │   │10.0.2.10+   │
    └───────────┘    └───────────┘   └─────────────┘
```

**4. Detailed Configuration**

**VLAN Configuration:**
```
VLAN 10: Employees (Wired)      - 10.0.1.0/24
VLAN 20: Employees (Wi-Fi)      - 10.0.2.0/24
VLAN 30: Servers                - 10.0.3.0/24
VLAN 40: Printers               - 10.0.4.0/25
VLAN 50: Management/IT          - 10.0.5.0/25
VLAN 99: Management Network     - 10.0.6.0/24
```

**IP Assignment:**
```
Gateway:
└─ All subnets: 10.0.X.1

DHCP Pools:
├─ VLAN 10: 10.0.1.50-200 (150 IPs for future)
├─ VLAN 20: 10.0.2.50-200 (150 IPs for future)
├─ VLAN 30: Static IPs (servers need static)
├─ VLAN 40: 10.0.4.10-100 (for printers)
└─ VLAN 50: Static IPs (IT equipment)
```

**Security Configuration:**
```
Firewall Rules:
├─ Block VLAN 10 ↔ VLAN 30 (employees can't reach servers directly)
├─ Allow VLAN 10 → VLAN 30 port 443 only
├─ Block VLAN 40 (printers) from reaching external
├─ Block all internal → external on port 23 (telnet)
└─ NAT: Internal 10.x.x.x → External to ISP IP
```

**5. Scaling Considerations**
- Use /16 (10.0.0.0/16) to allow growth
- Currently using 10.0.1-6 (6 subnets)
- Can expand to 10.0.1-254 (250+ subnets!)

**6. Redundancy Options**
- Dual firewalls (active/passive)
- Dual ISP connections
- Server clustering for HA
- Multiple access points for Wi-Fi coverage

---

## Summary Table

| Concept | Example | Key Point |
|---------|---------|-----------|
| Network | 192.168.1.0/24 | Group of connected devices |
| Subnet | 10.0.3.0/25 | Network divided into smaller pieces |
| Host Address | 192.168.1.100 | Individual device on network |
| Gateway | 192.168.1.1 | Router to reach other networks |
| Broadcast | 192.168.1.255 | Address to reach all on network |
| TCP | Port 443 HTTPS | Reliable, used by web browsers |
| UDP | Port 53 DNS | Fast, used by domain lookup |
| DNS | google.com→142.250.80.46 | Translates names to IPs |
| DHCP | Automatic IP assignment | Easier than manual configuration |
| NAT | 192.168.1.100→203.x.x.x | Hide internal IPs on external network |

