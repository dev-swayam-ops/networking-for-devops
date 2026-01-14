# Solutions: Setup and Networking Tools

Complete solutions and explanations for all exercises.

## Easy Exercises

### Exercise 1: Display Your Network Configuration

**Command:**
```powershell
ipconfig /all
```

**Solution:**
Your output should show all network adapters with this information:

| Item | Purpose |
|------|---------|
| IPv4 Address | Your computer's IP on the local network |
| Subnet Mask | Defines which part of IP is network vs host |
| Default Gateway | The router/gateway IP to reach other networks |
| DNS Servers | Servers that translate domain names to IPs |

**Key Points:**
- `/all` flag shows ALL adapters and full details (vs just `ipconfig` which is brief)
- Windows typically shows Ethernet or Wi-Fi adapters
- Subnet mask 255.255.255.0 = /24 network (256 total addresses)
- Multiple DNS servers provide redundancy if one fails

**Example Output Interpretation:**
```
IPv4 Address: 192.168.1.100
Subnet Mask: 255.255.255.0  ← This means you're on 192.168.1.0/24 network
Default Gateway: 192.168.1.1  ← Send packets here to leave your local network
DNS Servers: 8.8.8.8, 8.8.4.4  ← Google's DNS servers
```

---

### Exercise 2: Test Basic Connectivity

**Command:**
```powershell
ping 8.8.8.8 -n 4
```

**Solution:**

| Component | Meaning |
|-----------|---------|
| 8.8.8.8 | Google's public DNS (reliable destination) |
| -n 4 | Send 4 ping requests (then stop) |
| bytes=32 | Each ICMP packet is 32 bytes |
| time=15ms | Round-trip latency to the host |
| TTL=119 | Time-to-live (hops remaining) |
| 0% loss | All packets reached destination and returned |

**Good Results:**
- 0% packet loss (all 4 replies received)
- Latency < 50ms (excellent), 50-100ms (good), > 100ms (slow)

**Bad Results:**
- "Request timed out" = Host unreachable
- Packet loss > 0% = Network instability
- High latency (> 200ms) = Poor connection

**Why Google DNS?**
- 8.8.8.8 is Google's public DNS
- Always online and responds to ping
- Global presence = available everywhere

---

### Exercise 3: Resolve a Domain Name

**Command:**
```powershell
nslookup google.com
```

**Solution:**

```
Server: 8.8.8.8                        ← Your DNS server
Address: 8.8.8.8#53                    ← DNS uses port 53

Non-authoritative answer:               ← Cached result (not authoritative)
Name: google.com                        ← Domain you queried
Address: 142.250.80.46                  ← IPv4 address of google.com
```

**Key Points:**
- `nslookup` translates domain names → IP addresses
- Your configured DNS server answers the query
- "Non-authoritative" = result from cache, not the authoritative DNS server
- Google.com has multiple IP addresses (round-robin for load balancing)

**Why This Matters:**
- Without DNS, you'd need to remember every IP address
- DNS is fundamental to internet usability

**Try These:**
```powershell
nslookup github.com
nslookup microsoft.com
nslookup 8.8.8.8  # Reverse lookup (IP → domain)
```

---

### Exercise 4: View Active Network Connections

**Command:**
```powershell
netstat -an | findstr ESTABLISHED
```

**Solution:**

Breaking down the command:
- `netstat` = network statistics utility
- `-a` = show ALL connections
- `-n` = show numerical addresses (IPs, not hostnames)
- `| findstr ESTABLISHED` = filter to only ESTABLISHED connections

**Output Example:**
```
Proto  Local Address          Foreign Address        State
TCP    192.168.1.100:52134    142.250.80.46:443     ESTABLISHED
```

| Field | Meaning |
|-------|---------|
| TCP | Protocol (TCP or UDP) |
| 192.168.1.100 | Your local IP |
| :52134 | Your local port number (random high port) |
| 142.250.80.46 | Remote server IP |
| :443 | Remote port (443 = HTTPS) |
| ESTABLISHED | Connection is active |

**Why Port 443?**
- Port 443 = HTTPS (secure web)
- Port 80 = HTTP (unsecure web)
- Port 22 = SSH (secure shell)
- Ports > 1024 are dynamic/ephemeral

---

### Exercise 5: Check Specific Port Listening

**Command:**
```powershell
netstat -an | findstr LISTENING
```

**Solution:**

| State | Meaning |
|-------|---------|
| LISTENING | Port is open, waiting for incoming connections |
| ESTABLISHED | Active connection |
| TIME_WAIT | Connection closed, waiting before reuse |
| CLOSE_WAIT | Waiting for close from remote |

**Output Example:**
```
Proto  Local Address          Foreign Address        State
TCP    0.0.0.0:80             0.0.0.0:0             LISTENING
TCP    127.0.0.1:5432         0.0.0.0:0             LISTENING
```

**Interpretation:**
- `0.0.0.0:80` = Listening on port 80 on ALL interfaces (web server)
- `127.0.0.1:5432` = Listening on port 5432 only on localhost (PostgreSQL)

**Common Listening Ports:**
| Port | Service |
|------|---------|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |

---

## Medium Exercises

### Exercise 6: Trace Route to a Remote Server

**Command:**
```powershell
tracert microsoft.com
```

**Solution:**

tracert sends packets with increasing TTL (Time-to-Live) values:
- First packet has TTL=1 (dies at hop 1)
- Second packet has TTL=2 (dies at hop 2)
- Continues until reaching destination or max hops

**Output Example:**
```
Tracing route to microsoft.com [13.77.161.179]
over a maximum of 30 hops:

  1     2 ms     2 ms     2 ms  192.168.1.1
  2    12 ms    12 ms    12 ms  10.0.0.1
  3    25 ms    24 ms    26 ms  203.0.113.5
  4    45 ms    44 ms    46 ms  203.0.113.10
  5    55 ms    54 ms    56 ms  198.51.100.5
  6    65 ms    64 ms    66 ms  198.51.100.10
  7    75 ms    74 ms    76 ms  13.77.161.179 [microsoft.com]

Trace complete.
```

**Reading the Results:**

| Hop | Router | Response Times | Analysis |
|-----|--------|-----------------|----------|
| 1 | 192.168.1.1 | 2ms | Your home router (fast) |
| 2-5 | ISP routers | 12-55ms | Getting to ISP backbone |
| 6-7 | Internet core | 65-76ms | Microsoft's data center |

**What to Look For:**
- Latency increases with each hop (usually)
- Sudden jumps indicate network congestion
- `* * *` = router not responding to tracert (some filter ICMP)
- Compare different destinations to understand topology

**Why It Matters:**
- Identifies where packets slow down
- Shows routing path for troubleshooting
- Helps identify ISP or peering issues

---

### Exercise 7: Flush and Re-cache DNS

**Commands:**
```powershell
# View cache
ipconfig /displaydns | more

# Flush cache
ipconfig /flushdns

# Verify cleared
ipconfig /displaydns
```

**Solution:**

**DNS Caching Concept:**
- First DNS query goes to DNS server (slow)
- Result is cached locally (fast)
- Subsequent queries use cache (no network needed)

**Cache Benefits:**
- Faster lookups (0ms vs 50-100ms)
- Reduces DNS server load
- Works offline if cache has entries

**When to Clear Cache:**
- DNS record changed (site moved)
- Stuck on old IP address
- Troubleshooting DNS issues
- Malware redirected cache

**Process:**

1. **Before flush:**
```
Name: google.com
Data: 142.250.80.46
Time To Live: 3599
[many more entries...]
```

2. **After flush:**
```
Flushing the DNS Resolver Cache... All clear!
```

3. **After new query:**
```
ipconfig /displaydns | findstr google.com
```
Shows the newly cached entry with fresh TTL

**Why This Works:**
- DNS cache lives in RAM
- `ipconfig /flushdns` clears it
- Next lookup queries server for fresh data

---

### Exercise 8: Check ARP Table

**Commands:**
```powershell
# View ARP table
arp -a

# Add static entry (requires admin)
arp -s 192.168.1.50 AA-BB-CC-DD-EE-FF

# View again
arp -a

# Delete entry
arp -d 192.168.1.50
```

**Solution:**

**ARP (Address Resolution Protocol):**
- Converts IP addresses → MAC addresses
- Essential for local network communication

**Output Example:**
```
Interface: 192.168.1.100 --- 0x6
  Internet Address      Physical Address      Type
  192.168.1.1          AA-BB-CC-DD-EE-FF    dynamic
  192.168.1.50         AA-BB-CC-DD-EE-FF    static
  224.0.0.1            FF-FF-FF-FF-FF-FF    static
```

| Column | Meaning |
|--------|---------|
| Internet Address | IP address on your network |
| Physical Address | MAC address (hardware address) |
| dynamic | Learned via ARP requests |
| static | Manually added by admin |

**Why ARP Matters:**
- Without ARP, can't find MAC address from IP
- Without MAC, packets can't reach local device
- ARP spoofing is a security risk

**MAC Address Format:**
- 48 bits (6 bytes): AA-BB-CC-DD-EE-FF
- First 3 bytes = vendor (OUI)
- Last 3 bytes = device specific

**Commands:**
| Command | Purpose |
|---------|---------|
| `arp -a` | Display all ARP entries |
| `arp -s IP MAC` | Add static entry |
| `arp -d IP` | Delete entry |
| `arp -a -N IP` | Show ARP for specific interface |

---

### Exercise 9: Monitor Network Statistics

**Commands:**
```powershell
# TCP connections only
netstat -an -p tcp

# PowerShell: connection state summary
Get-NetTCPConnection | Group-Object -Property State | Select-Object Name, Count

# Details by state
Get-NetTCPConnection | Where-Object State -eq ESTABLISHED | Measure-Object
```

**Solution:**

**Connection States:**

| State | Meaning | Duration |
|-------|---------|----------|
| LISTEN | Waiting for incoming connection | Indefinite |
| ESTABLISHED | Active, two-way communication | Active session |
| TIME_WAIT | Closed, waiting for timeout | 2-4 minutes |
| CLOSE_WAIT | Remote closed, local still open | Brief |
| SYN_SENT | Initiating connection | Very brief |
| FIN_WAIT | Closing connection | Brief |

**Example PowerShell Output:**
```
Name           Count
----           -----
Established       15
Listen             3
TimeWait           2
CloseWait          1
```

**Interpretation:**
- 15 active connections
- 3 ports listening for new connections
- 2 closed connections waiting for reuse timeout
- 1 connection in half-closed state

**Advanced Monitoring:**

```powershell
# Show all states
Get-NetTCPConnection | Group-Object -Property State | Sort-Object Count -Descending

# Processes with connections
Get-NetTCPConnection -State Established | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, OwningProcess

# Find process using specific port
Get-Process -Id ((Get-NetTCPConnection -LocalPort 8080).OwningProcess)
```

---

### Exercise 10: Network Adapter Management

**Commands:**
```powershell
# List adapters
Get-NetAdapter

# Get specific adapter
Get-NetAdapter -Name "Ethernet" | Format-List

# Check IP configuration
Get-NetIPConfiguration

# Disable adapter (requires admin)
Disable-NetAdapter -Name "Ethernet" -Confirm:$false

# Enable adapter (requires admin)
Enable-NetAdapter -Name "Ethernet" -Confirm:$false
```

**Solution:**

**Output Example:**
```
Name: Ethernet
InterfaceDescription: Intel(R) Ethernet Connection (11) I219-V
IfIndex: 3
Status: Up
MacAddress: 00-11-22-33-44-55
LinkSpeed: 1 Gbps
```

| Property | Meaning |
|----------|---------|
| Name | Short name for interface |
| Status | Up (connected) or Down (disconnected) |
| MacAddress | Hardware address |
| LinkSpeed | Connection speed |
| IfIndex | Interface index number |

**Why Disable/Enable?**
- Troubleshoot connectivity issues
- Reset adapter without restart
- Release and renew DHCP lease

**Safer Alternative to Disable/Enable:**

```powershell
# Release DHCP lease (keeps adapter up)
ipconfig /release

# Renew DHCP lease
ipconfig /renew

# Set static IP
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.50 -PrefixLength 24 -DefaultGateway 192.168.1.1

# Restore DHCP
Set-NetIPInterface -InterfaceAlias "Ethernet" -DHCP Enabled
```

**When to Use Each:**

| Situation | Solution |
|-----------|----------|
| No IP address | `ipconfig /renew` |
| Wrong IP address | Release and renew |
| Can't reach gateway | Check gateway with `ping` |
| Complete connectivity loss | Disable/enable adapter |
| Need static IP | Use `New-NetIPAddress` |

---

## Challenge Question Answers

**How to find answers:**

**1. Latency to default gateway?**
```powershell
ipconfig /all                          # Get default gateway IP
ping <gateway-ip> -n 4                 # Measure latency
```

**2. Hops to 1.1.1.1?**
```powershell
tracert 1.1.1.1                        # Count the hops shown
```

**3. MAC address of default gateway?**
```powershell
ipconfig /all                          # Get gateway IP
arp -a | findstr <gateway-ip>          # Find its MAC
```

**4. Open listening ports?**
```powershell
netstat -an | findstr LISTENING        # Count lines shown
```

**5. DNS server?**
```powershell
ipconfig /all                          # Look for "DNS Servers" line
```

---

## Summary Table

| Task | Command | Purpose |
|------|---------|---------|
| View IP config | `ipconfig /all` | See all network settings |
| Test connectivity | `ping 8.8.8.8 -n 4` | Verify internet access |
| Resolve DNS | `nslookup google.com` | Look up domain names |
| View connections | `netstat -an` | See all network connections |
| Trace route | `tracert microsoft.com` | See packet path |
| Clear DNS cache | `ipconfig /flushdns` | Reset DNS cache |
| View ARP table | `arp -a` | See IP to MAC mappings |
| Monitor connections | `Get-NetTCPConnection` | PowerShell TCP details |
| Check adapters | `Get-NetAdapter` | List network interfaces |

