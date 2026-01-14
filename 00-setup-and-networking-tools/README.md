# 00 - Setup and Networking Tools

## What You'll Learn

By the end of this module, you'll be able to:
- Install and configure essential networking tools
- Use command-line utilities to diagnose network problems
- Understand how to test connectivity between devices
- Capture and analyze network packets
- Troubleshoot basic network issues on Windows

## Prerequisites

- Windows 10/11 with administrative access (or equivalent Linux/Mac)
- Basic command-line knowledge (PowerShell or Terminal)
- Network connectivity

## Key Concepts

### Essential Networking Tools

1. **ipconfig** - Display network configuration
2. **ping** - Test connectivity to a host
3. **nslookup** - Query DNS records
4. **tracert** - Trace the path to a destination
5. **netstat** - Display network statistics
6. **arp** - Address Resolution Protocol utility
7. **pathping** - Combines ping and tracert
8. **netsh** - Network shell for advanced configurations

## Hands-on Lab: Setting Up Your First Network Diagnosis

### Objective
Set up your networking toolkit and perform basic diagnostics on your system.

### Step 1: Check Your Network Configuration

```powershell
ipconfig /all
```

**Expected Output:**
```
Windows IP Configuration

Ethernet adapter Ethernet:
   Connection-specific DNS Suffix: .
   Link-local IPv6 Address: fe80::1234:5678:9abc:def0%5
   IPv4 Address: 192.168.1.100
   Subnet Mask: 255.255.255.0
   Default Gateway: 192.168.1.1
   DNS Servers: 8.8.8.8, 8.8.4.4
```

### Step 2: Test Connectivity to Google DNS

```powershell
ping 8.8.8.8 -n 4
```

**Expected Output:**
```
Pinging 8.8.8.8 with 32 bytes of data:
Reply from 8.8.8.8: bytes=32 time=15ms TTL=119
Reply from 8.8.8.8: bytes=32 time=14ms TTL=119
Reply from 8.8.8.8: bytes=32 time=16ms TTL=119
Reply from 8.8.8.8: bytes=32 time=14ms TTL=119

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 14ms, Maximum = 16ms, Average = 14ms
```

### Step 3: Resolve a Domain Name

```powershell
nslookup google.com
```

**Expected Output:**
```
Server: 8.8.8.8
Address: 8.8.8.8#53

Non-authoritative answer:
Name: google.com
Address: 142.250.80.46
```

### Step 4: Trace the Route to a Host

```powershell
tracert google.com
```

**Expected Output:**
```
Tracing route to google.com [142.250.80.46]
over a maximum of 30 hops:

  1    12 ms     8 ms     9 ms  192.168.1.1
  2    14 ms    15 ms    14 ms  10.0.0.1
  3    25 ms    24 ms    26 ms  203.0.113.5
...
```

### Step 5: View Network Statistics

```powershell
netstat -an
```

**Expected Output:**
```
Active Connections

Proto  Local Address          Foreign Address        State
TCP    127.0.0.1:49670       127.0.0.1:49671       ESTABLISHED
TCP    192.168.1.100:52134   142.250.80.46:443     ESTABLISHED
...
```

## Validation Checklist

- [ ] ipconfig displays your IP address and subnet mask
- [ ] ping successfully reaches 8.8.8.8 with 0% packet loss
- [ ] nslookup resolves google.com to an IP address
- [ ] tracert shows at least 3 hops to a destination
- [ ] netstat displays active connections
- [ ] All tools respond without "command not found" errors

## Cleanup

No cleanup needed for this module. All commands are read-only diagnostic tools.

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| "ipconfig is not recognized" | Use full path or ensure PATH is set correctly |
| ping fails to all hosts | Check your network adapter is enabled (ipconfig /all) |
| nslookup returns "Can't find server" | Verify DNS server in ipconfig settings |
| tracert takes too long | Some ISPs rate-limit ICMP; this is normal |
| netstat shows too much output | Use `netstat -an \| findstr ESTABLISHED` to filter |

## Troubleshooting

### Problem: "No internet connectivity"
**Solution:**
1. Check adapter status: `ipconfig /all`
2. Check if DHCP is enabled or IP is configured manually
3. Ping the default gateway: `ping <gateway-ip>`
4. Restart network adapter: `ipconfig /release` then `ipconfig /renew`

### Problem: "DNS not resolving"
**Solution:**
1. Verify DNS servers: `ipconfig /all`
2. Flush DNS cache: `ipconfig /flushdns`
3. Try alternative DNS: `nslookup google.com 8.8.8.8`

### Problem: "High packet loss in ping"
**Solution:**
1. Check if host is online: `ping <host>`
2. Check network stability: `ping -t <host>` (press Ctrl+C to stop)
3. Check for firewall blocking: Disable Windows Firewall temporarily (Windows Security)

## Next Steps

- **Module 01:** Learn networking fundamentals and ISO/OSI model
- **Module 02:** Deep dive into TCP/IP protocols
- Practice these commands daily until they become muscle memory
- Set up a home lab environment with multiple devices to test connectivity

## Resources

- [Microsoft ipconfig documentation](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/ipconfig)
- [ping Command Reference](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/ping)
- [Networking Utilities Guide](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/networking-commands)
