# 01 - Networking Fundamentals

## What You'll Learn

By the end of this module, you'll be able to:
- Understand the fundamentals of how networks work
- Explain the difference between LAN, WAN, and other network types
- Understand network protocols and how data flows
- Calculate subnets and IP address ranges
- Design simple network topologies
- Understand data packets and frames

## Prerequisites

- Completion of Module 00 (Setup and Networking Tools)
- Basic understanding of IP addresses
- Ability to run network commands

## Key Concepts

### What is a Network?

A network is a group of computers connected to share resources and communicate. Key characteristics:

- **LAN (Local Area Network)**: Computers in same building/office (192.168.x.x)
- **WAN (Wide Area Network)**: Networks across cities/countries (internet)
- **MAN (Metropolitan Area Network)**: Computers across a city
- **Topology**: Physical arrangement (star, mesh, bus, ring)

### Network Models

Networks operate in layers. Each layer handles specific functions:

1. **Physical Layer**: Cables, connectors (electrical signals)
2. **Data Link Layer**: MAC addresses, switches (frames)
3. **Network Layer**: IP addresses, routers (packets)
4. **Transport Layer**: TCP/UDP, ports (reliability)
5. **Application Layer**: HTTP, DNS, email (user applications)

### Key Network Components

| Component | Purpose | Example |
|-----------|---------|---------|
| **Router** | Directs packets between networks | Home router 192.168.1.1 |
| **Switch** | Connects devices on same network | Office network switch |
| **Firewall** | Controls incoming/outgoing traffic | Windows Defender |
| **DNS Server** | Translates domain names to IPs | 8.8.8.8 |
| **DHCP Server** | Assigns IP addresses automatically | Built into router |
| **Server** | Provides services | Web, email, database servers |

### Protocols

A protocol is a set of rules for communication:

| Protocol | Purpose | Port | Layer |
|----------|---------|------|-------|
| **TCP** | Reliable delivery (error-checked) | Various | Transport |
| **UDP** | Fast delivery (no error-checking) | Various | Transport |
| **HTTP** | Web pages (unsecure) | 80 | Application |
| **HTTPS** | Web pages (secure) | 443 | Application |
| **DNS** | Domain name resolution | 53 | Application |
| **SMTP** | Sending email | 25 | Application |
| **FTP** | File transfer | 21 | Application |

## Hands-on Lab: Understanding Network Flow

### Objective
Trace how data flows through your network when you visit a website.

### Step 1: Visit a Website and Capture DNS Resolution

```powershell
# Clear DNS cache
ipconfig /flushdns

# Now resolve a domain (watch it query DNS server)
nslookup example.com
```

**Expected Output:**
```
Server: 8.8.8.8
Address: 8.8.8.8#53

Non-authoritative answer:
Name: example.com
Address: 93.184.216.34
```

**What Happened:**
1. Your computer asked your DNS server "What's the IP for example.com?"
2. DNS server responded with 93.184.216.34
3. Now your browser knows where to connect

### Step 2: Trace the Network Path

```powershell
# See the path your packets take
tracert example.com
```

**Expected Output:**
```
Tracing route to example.com [93.184.216.34]
over a maximum of 30 hops:

  1     2 ms     2 ms     2 ms  192.168.1.1
  2    12 ms    11 ms    12 ms  10.0.0.1
  3    25 ms    24 ms    24 ms  203.0.113.5
  4    35 ms    34 ms    36 ms  203.0.113.10
...
 13    45 ms    44 ms    46 ms  93.184.216.34

Trace complete.
```

**What This Shows:**
- Hop 1: Your home router
- Hops 2-5: ISP's network
- Hops 6-12: Internet backbone
- Hop 13: Destination website

### Step 3: Monitor Active Connections During Browsing

Open PowerShell and run:

```powershell
# Get current connections
netstat -an | findstr ESTABLISHED

# Or in PowerShell
Get-NetTCPConnection -State Established | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort
```

Then open a web browser and visit https://example.com

Run again:

```powershell
Get-NetTCPConnection -State Established | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort
```

**Expected Output:**
```
LocalAddress    LocalPort RemoteAddress      RemotePort
-----------     --------- ---------------    ----------
192.168.1.100   52135     93.184.216.34      443
192.168.1.100   52136     142.251.32.142     443
```

**What This Shows:**
- Your IP: 192.168.1.100
- Your random port: 52135, 52136 (ephemeral ports)
- Remote server: 93.184.216.34 (example.com)
- Remote port: 443 (HTTPS)

### Step 4: Examine a Network Packet (Concept)

The data traveling through the network is organized in layers:

**Packet Structure:**

```
┌─────────────────────────────────────────────────┐
│ Ethernet Header (Data Link Layer)               │
│ Source MAC: AA-BB-CC-DD-EE-FF                  │
│ Dest MAC: 11-22-33-44-55-66                    │
├─────────────────────────────────────────────────┤
│ IP Header (Network Layer)                       │
│ Source IP: 192.168.1.100                        │
│ Dest IP: 93.184.216.34                          │
├─────────────────────────────────────────────────┤
│ TCP Header (Transport Layer)                    │
│ Source Port: 52135                              │
│ Dest Port: 443                                  │
│ Sequence Number: 1001                           │
│ Acknowledgment: 2001                            │
├─────────────────────────────────────────────────┤
│ Data (Application Layer)                        │
│ GET /index.html HTTP/1.1                        │
│ Host: example.com                               │
│ ...                                             │
└─────────────────────────────────────────────────┘
```

### Step 5: Test Network Speed

Test your network latency to different servers:

```powershell
# Ping Google's DNS (usually fast)
ping 8.8.8.8 -n 10

# Ping a far away server
ping 1.1.1.1 -n 10

# Compare the latencies
```

**Expected Output:**
```
Pinging 8.8.8.8 with 32 bytes of data:
Reply from 8.8.8.8: bytes=32 time=15ms TTL=119
[3 more replies]

Statistics: Min = 14ms, Max = 18ms, Average = 15ms
```

**Interpretation:**
- Average latency 15ms = good
- Average latency 50ms = acceptable
- Average latency 100ms+ = slow or distance

## Validation Checklist

- [ ] Can explain what DNS does (resolves domains to IPs)
- [ ] Can trace a route to a remote server using `tracert`
- [ ] Can identify your local IP address and gateway
- [ ] Can see active network connections on your computer
- [ ] Understand packet structure (headers + data)
- [ ] Can explain the difference between TCP and UDP
- [ ] Know what protocols use ports 80, 443, 53, 21
- [ ] Understand LANs vs WANs and the difference

## Cleanup

No cleanup needed. All commands are read-only or monitoring.

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| "My IP is 127.0.0.1" | That's localhost. Use `ipconfig /all` to find real IP |
| "Tracert shows * * *" | Some routers block ICMP; continue - destination may respond |
| "Can't resolve google.com" | Check if DNS is configured in `ipconfig /all` |
| "High latency to nearby server" | Network congestion or poor connection quality |
| Confusing MAC and IP addresses | MAC = hardware address (local), IP = network address (routable) |

## Troubleshooting

### Problem: "Can't reach a website but ping works"
**Solution:**
1. Website may block ICMP ping: `ping` doesn't mean HTTP works
2. Check if port 443 is open: `telnet example.com 443`
3. Verify DNS resolves: `nslookup example.com`
4. Check firewall isn't blocking port 443

### Problem: "Slow network speed"
**Solution:**
1. Test to nearby server: `ping 8.8.8.8 -n 10`
2. Test to far away server: `ping 1.1.1.1 -n 10`
3. High latency everywhere = internet connection problem
4. High latency to some = routing or peering issue
5. Check for packet loss: should be 0%

### Problem: "One device on network not responding"
**Solution:**
1. Check if it's online: `ping <device-ip>`
2. Check ARP table: `arp -a | findstr <device-ip>`
3. Verify MAC address is correct
4. Check if firewall is blocking on that device
5. Try `arp -d <device-ip>` to force new ARP request

## Next Steps

- **Module 02:** Deep dive into OSI and TCP/IP models
- Create a small network diagram on paper
- Practice calculating subnets (covered in Module 03)
- Experiment with port scanning for security

## Resources

- [Cisco Networking Basics](https://www.cisco.com/c/en/us/support/docs/ip/routing-information-protocol-rip/13788-3.html)
- [How Networks Work - Khan Academy](https://www.khanacademy.org)
- [Networking Fundamentals](https://www.comptia.org/content/articles/what-is-network-fundamentals)

## Network Types Summary

```
┌─────────────────────────────────────────────────┐
│ INTERNET (Global WAN)                           │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌──────────────┐         ┌──────────────┐     │
│  │ISP Router    │         │ISP Router    │     │
│  │203.0.113.1   │         │203.0.113.2   │     │
│  └──────┬───────┘         └──────┬───────┘     │
│         │                        │             │
│  ┌──────┴────────┐        ┌──────┴────────┐   │
│  │Home Router    │        │Office Router  │   │
│  │192.168.1.1    │        │10.0.0.1       │   │
│  └──────┬────────┘        └──────┬────────┘   │
│         │                        │             │
│  ┌──────┴──────┬────────┐ ┌──────┴──────┬──┐ │
│  │Laptop       │Desktop │ │Server       │PC│ │
│  │192.168.1.10 │.20     │ │10.0.0.10    │  │ │
│  └─────────────┴────────┘ └─────────────┴──┘ │
│  (Home LAN)                 (Office LAN)     │
│                                               │
└─────────────────────────────────────────────────┘
```
