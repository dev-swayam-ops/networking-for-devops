# Cheatsheet: Setup and Networking Tools

Quick reference for essential networking commands and their purposes.

## Network Configuration Commands

| Command | Purpose | Example | Output |
|---------|---------|---------|--------|
| `ipconfig` | Show brief IP config | `ipconfig` | Local IP, gateway, DNS |
| `ipconfig /all` | Show detailed IP config | `ipconfig /all` | All adapters, MAC address, DHCP info |
| `ipconfig /renew` | Renew DHCP lease | `ipconfig /renew` | New IP from DHCP server |
| `ipconfig /release` | Release DHCP lease | `ipconfig /release` | Surrenders current IP |
| `ipconfig /flushdns` | Clear DNS cache | `ipconfig /flushdns` | Purges cached DNS entries |
| `ipconfig /displaydns` | Show DNS cache | `ipconfig /displaydns` | All cached domain records |

## Connectivity Testing Commands

| Command | Purpose | Example | Output |
|---------|---------|---------|--------|
| `ping <host>` | Test connectivity | `ping 8.8.8.8` | Response times, packet loss |
| `ping <host> -n <count>` | Ping N times | `ping 8.8.8.8 -n 4` | 4 ping requests |
| `ping -t <host>` | Continuous ping | `ping -t 8.8.8.8` | Endless until Ctrl+C |
| `ping -a <host>` | Ping with hostname | `ping -a 8.8.8.8` | Shows reverse DNS lookup |
| `ping <host> -w <timeout>` | Set timeout (ms) | `ping 8.8.8.8 -w 5000` | Wait 5 seconds per reply |
| `tracert <host>` | Trace route | `tracert google.com` | Hops to destination |
| `pathping <host>` | Trace with latency | `pathping google.com` | Path + hop latency stats |

## DNS Resolution Commands

| Command | Purpose | Example | Output |
|---------|---------|---------|--------|
| `nslookup <domain>` | Lookup domain | `nslookup google.com` | IP address of domain |
| `nslookup <domain> <server>` | Query specific DNS | `nslookup google.com 8.8.8.8` | Result from Google DNS |
| `nslookup <ip>` | Reverse lookup | `nslookup 8.8.8.8` | Domain name of IP |
| `nslookup -type=A <domain>` | Query A record | `nslookup -type=A google.com` | IPv4 address |
| `nslookup -type=MX <domain>` | Query MX record | `nslookup -type=MX google.com` | Mail server |
| `nslookup -type=NS <domain>` | Query NS record | `nslookup -type=NS google.com` | Nameservers |

## Network Statistics Commands

| Command | Purpose | Example | Output |
|---------|---------|---------|--------|
| `netstat` | Show network stats | `netstat` | Active connections |
| `netstat -an` | All connections, numeric | `netstat -an` | IP:port instead of names |
| `netstat -ab` | All with process name | `netstat -ab` | Which program owns connection |
| `netstat -o` | Include process ID | `netstat -o` | PID for each connection |
| `netstat -an -p tcp` | TCP only | `netstat -an -p tcp` | Only TCP connections |
| `netstat -an -p udp` | UDP only | `netstat -an -p udp` | Only UDP connections |
| `netstat -an \| findstr ESTABLISHED` | Established only | `netstat -an \| findstr ESTABLISHED` | Active connections |
| `netstat -an \| findstr LISTENING` | Listening only | `netstat -an \| findstr LISTENING` | Open server ports |
| `netstat -s` | Statistics summary | `netstat -s` | Packets sent/received/errors |

## ARP Commands

| Command | Purpose | Example | Output |
|---------|---------|---------|--------|
| `arp -a` | Show all ARP entries | `arp -a` | IP to MAC mappings |
| `arp -a -N <ip>` | ARP for specific interface | `arp -a -N 192.168.1.1` | ARP for that interface |
| `arp -s <ip> <mac>` | Add static ARP | `arp -s 192.168.1.50 AA-BB-CC-DD-EE-FF` | Adds permanent entry |
| `arp -d <ip>` | Delete ARP entry | `arp -d 192.168.1.50` | Removes entry |
| `arp -d *` | Delete all ARP | `arp -d *` | Clears entire ARP table |

## PowerShell Network Commands

| Command | Purpose | Example | Output |
|---------|---------|---------|--------|
| `Get-NetAdapter` | List network adapters | `Get-NetAdapter` | All network interfaces |
| `Get-NetIPConfiguration` | IP configuration | `Get-NetIPConfiguration` | IP, gateway, DNS per adapter |
| `Get-NetIPAddress` | All IP addresses | `Get-NetIPAddress` | IPv4 and IPv6 addresses |
| `Get-NetRoute` | View routing table | `Get-NetRoute` | All routes |
| `Get-NetTCPConnection` | TCP connections | `Get-NetTCPConnection` | All TCP connections |
| `Get-NetTCPConnection -State Established` | Active TCP | `Get-NetTCPConnection -State Established` | Only active connections |
| `Get-NetTCPConnection \| Group-Object -Property State` | Group by state | `Get-NetTCPConnection \| Group-Object State` | Count by connection state |
| `Enable-NetAdapter -Name "Ethernet"` | Enable adapter | `Enable-NetAdapter -Name "Ethernet"` | Activates network adapter |
| `Disable-NetAdapter -Name "Ethernet"` | Disable adapter | `Disable-NetAdapter -Name "Ethernet"` | Deactivates adapter |

## Quick Diagnostic Sequence

**5-minute network diagnosis:**

```powershell
# 1. Check if you have internet
ping 8.8.8.8 -n 1

# 2. Check if DNS works
nslookup google.com

# 3. Check your local IP
ipconfig /all

# 4. Check your gateway
ping <gateway-ip> -n 1

# 5. Check if specific service is reachable
ping <service-name> -n 1
```

## Port Reference

| Port | Service | Protocol | Usage |
|------|---------|----------|-------|
| 20 | FTP Data | TCP | File transfer data |
| 21 | FTP Control | TCP | File transfer control |
| 22 | SSH | TCP | Secure shell |
| 25 | SMTP | TCP | Email sending |
| 53 | DNS | UDP/TCP | Domain name resolution |
| 80 | HTTP | TCP | Web (unsecure) |
| 110 | POP3 | TCP | Email retrieval |
| 143 | IMAP | TCP | Email retrieval |
| 443 | HTTPS | TCP | Web (secure) |
| 3306 | MySQL | TCP | Database |
| 5432 | PostgreSQL | TCP | Database |
| 5900 | VNC | TCP | Remote desktop |
| 6379 | Redis | TCP | In-memory database |
| 8080 | HTTP Alt | TCP | Alternative web |
| 27017 | MongoDB | TCP | NoSQL database |

## Common Symbols in Output

| Symbol | Meaning | Example |
|--------|---------|---------|
| `*` | No response | `* * * request timed out` |
| TTL | Time To Live | `TTL=64` hops remaining |
| ms | Milliseconds | `14ms` latency |
| Mbps | Megabits/second | `1 Gbps` link speed |
| 0% loss | Perfect | No packet loss |
| `ESTABLISHED` | Active connection | Connection is live |
| `LISTENING` | Waiting for connection | Server port open |
| `TIME_WAIT` | Connection closed | Waiting for timeout |

## Filtering Output with PowerShell

| Technique | Purpose | Example |
|-----------|---------|---------|
| `\| findstr KEYWORD` | Find lines with KEYWORD | `netstat -an \| findstr ESTABLISHED` |
| `\| Select-Object -First N` | Show first N lines | `Get-NetAdapter \| Select-Object -First 3` |
| `\| Where-Object CONDITION` | Filter by condition | `Get-NetTCPConnection \| Where-Object State -eq Established` |
| `\| Group-Object -Property COL` | Group by column | `Get-NetTCPConnection \| Group-Object State` |
| `\| Sort-Object -Property COL` | Sort by column | `netstat -an \| Sort-Object` |
| `\| Measure-Object` | Count lines | `Get-NetAdapter \| Measure-Object` |

## Troubleshooting Quick Fixes

| Problem | Quick Fix |
|---------|-----------|
| No internet | `ipconfig /renew` then `ping 8.8.8.8` |
| Can't reach by hostname | `ipconfig /flushdns` then `nslookup <domain>` |
| Stuck IP address | `ipconfig /release` then `ipconfig /renew` |
| High latency | `ping -t <host>` to monitor, check line quality |
| DNS not responding | `nslookup google.com 8.8.8.8` (use Google DNS) |
| Port already in use | `netstat -ano \| findstr :PORT` find process, then stop it |
| Can't reach local device | `arp -a` to check ARP, `ping <local-ip>` to verify |

## TCP Connection States Explained

| State | Explanation | When It Occurs |
|-------|-------------|----------------|
| `LISTEN` | Port waiting for connections | Server started |
| `ESTABLISHED` | Active two-way communication | Client connected to server |
| `SYN_SENT` | Client sent SYN, waiting ACK | During connection establishment |
| `SYN_RECEIVED` | Server received SYN from client | Server receiving connection |
| `FIN_WAIT_1` | Local initiating close | After sending FIN |
| `FIN_WAIT_2` | Waiting for remote FIN | Remote acknowledged close |
| `TIME_WAIT` | Waiting before port reuse | Both sides closed |
| `CLOSE_WAIT` | Waiting for app to close socket | Remote closed first |
| `LAST_ACK` | Waiting for close ACK | Sending final FIN |
| `CLOSED` | No connection | Socket closed |

## Common Regex Filters

| Pattern | Matches | Example |
|---------|---------|---------|
| `ESTABLISHED` | Active connections | `netstat -an \| findstr ESTABLISHED` |
| `LISTENING` | Listening ports | `netstat -an \| findstr LISTENING` |
| `^TCP` | TCP only (line start) | `netstat -an \| findstr "^TCP"` |
| `^UDP` | UDP only | `netstat -an \| findstr "^UDP"` |
| `:443` | Port 443 (https) | `netstat -an \| findstr ":443"` |
| `:80` | Port 80 (http) | `netstat -an \| findstr ":80"` |

## Key Concepts Quick Reference

- **IP Address**: Identifies computer on network (e.g., 192.168.1.100)
- **MAC Address**: Hardware address on local network (e.g., AA-BB-CC-DD-EE-FF)
- **Gateway**: Router to reach other networks
- **DNS**: Converts domain names (google.com) to IPs (142.250.80.46)
- **TTL**: Hops remaining before packet dies
- **Latency**: Round-trip time in milliseconds
- **Packet Loss**: Percentage of packets that didn't reach destination
- **Port**: Logical endpoint for network service (0-65535)
- **Socket**: IP + Port combination (192.168.1.100:8080)
- **ARP**: Converts IP to MAC addresses on local network

