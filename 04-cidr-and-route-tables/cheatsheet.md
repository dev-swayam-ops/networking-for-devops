# Cheatsheet: CIDR and Route Tables

Quick reference guide for CIDR notation, subnet masks, and routing commands.

---

## CIDR and Subnet Mask Quick Reference

### CIDR to Subnet Mask Conversion

| CIDR | Subnet Mask | Host Bits | Hosts | Use Case |
|------|-------------|-----------|-------|----------|
| /8 | 255.0.0.0 | 24 | 16,777,214 | Large organization |
| /9 | 255.128.0.0 | 23 | 8,388,606 | - |
| /10 | 255.192.0.0 | 22 | 4,194,302 | - |
| /11 | 255.224.0.0 | 21 | 2,097,150 | ISP allocation |
| /12 | 255.240.0.0 | 20 | 1,048,574 | - |
| /13 | 255.248.0.0 | 19 | 524,286 | - |
| /14 | 255.252.0.0 | 18 | 262,142 | - |
| /15 | 255.254.0.0 | 17 | 131,070 | - |
| /16 | 255.255.0.0 | 16 | 65,534 | Medium organization |
| /17 | 255.255.128.0 | 15 | 32,766 | - |
| /18 | 255.255.192.0 | 14 | 16,382 | - |
| /19 | 255.255.224.0 | 13 | 8,190 | - |
| /20 | 255.255.240.0 | 12 | 4,094 | Small to medium |
| /21 | 255.255.248.0 | 11 | 2,046 | Small office |
| /22 | 255.255.252.0 | 10 | 1,022 | Office (100+ people) |
| /23 | 255.255.254.0 | 9 | 510 | Office (250+ people) |
| /24 | 255.255.255.0 | 8 | 254 | Most common, home/small office |
| /25 | 255.255.255.128 | 7 | 126 | Department networks |
| /26 | 255.255.255.192 | 6 | 62 | Team networks |
| /27 | 255.255.255.224 | 5 | 30 | Small groups |
| /28 | 255.255.255.240 | 4 | 14 | Very small office |
| /29 | 255.255.255.248 | 3 | 6 | Small networks |
| /30 | 255.255.255.252 | 2 | 2 | Router links only |
| /31 | 255.255.255.254 | 1 | 0 | Point-to-point (RFC 3021) |
| /32 | 255.255.255.255 | 0 | 1 | Single host |

### Quick Math for Any CIDR

**Formula:**
```
Total Hosts = 2^(32 - CIDR)
Usable Hosts = 2^(32 - CIDR) - 2
Host Bits = 32 - CIDR
```

**Examples:**
```
/24:  2^(32-24) = 2^8 = 256 total, 254 usable
/25:  2^(32-25) = 2^7 = 128 total, 126 usable
/26:  2^(32-26) = 2^6 = 64 total, 62 usable
/30:  2^(32-30) = 2^2 = 4 total, 2 usable
```

---

## Subnet Mask to CIDR Conversion

### Octet Values and Bit Counts

| Octet Value | Binary | Bits | Remaining CIDR |
|-------------|--------|------|-----------------|
| 0 | 00000000 | 0 | +0 |
| 128 | 10000000 | 1 | +1 |
| 192 | 11000000 | 2 | +2 |
| 224 | 11100000 | 3 | +3 |
| 240 | 11110000 | 4 | +4 |
| 248 | 11111000 | 5 | +5 |
| 252 | 11111100 | 6 | +6 |
| 254 | 11111110 | 7 | +7 |
| 255 | 11111111 | 8 | +8 |

### Conversion Method

```
Subnet Mask: 255.255.255.192
             255 + 255 + 255 + 192
             8   + 8   + 8   + 2  = 26
Result: /26
```

---

## Private IP Ranges (RFC 1918)

**These are internal-only addresses:**

| Range | CIDR | Start | End | Total IPs | Use |
|-------|------|-------|-----|-----------|-----|
| Class A | 10.0.0.0/8 | 10.0.0.0 | 10.255.255.255 | 16,777,216 | Large networks |
| Class B | 172.16.0.0/12 | 172.16.0.0 | 172.31.255.255 | 1,048,576 | Medium networks |
| Class C | 192.168.0.0/16 | 192.168.0.0 | 192.168.255.255 | 65,536 | Small networks, home |
| Loopback | 127.0.0.0/8 | 127.0.0.1 | 127.255.255.255 | - | Local machine testing |
| Link-Local | 169.254.0.0/16 | 169.254.0.0 | 169.254.255.255 | 65,536 | DHCP fallback |

**Quick Rule:**
- 10.x.x.x → Private (any Class A)
- 172.16-31.x.x → Private (any Class B in range)
- 192.168.x.x → Private (any Class C in range)
- Everything else → Public (routable on internet)

---

## Route Table Anatomy

### Example Route Entry

```
Network Destination    Netmask           Gateway      Interface   Metric
192.168.1.0            255.255.255.0     On-link      192.168.1.50  275
```

### Column Meanings

| Column | Meaning | Example | Notes |
|--------|---------|---------|-------|
| Destination | Target network | 192.168.1.0 | What traffic are you routing? |
| Netmask | Subnet mask | 255.255.255.0 | How many bits define network? |
| Gateway | Where to send it | 192.168.1.1 | IP address of next router |
| Interface | Your network card | 192.168.1.50 | Your IP on that network |
| Metric | Cost/priority | 25 | Lower = preferred |

### Common Gateway Values

| Gateway | Type | Meaning |
|---------|------|---------|
| On-link | Direct | Traffic goes directly to subnet (no router needed) |
| 192.168.1.1 | Router | Send traffic to this router's IP |
| 0.0.0.0 | Default | This is the default route (everything else) |

---

## Windows Route Commands

### View Routes

**Command:** `route print`
```powershell
route print              # Show all routes
route print | find "172.16"    # Find specific route
ipconfig /all            # Show network config including gateway
```

**PowerShell (modern, recommended):**
```powershell
Get-NetRoute | Format-Table DestinationPrefix, NextHop, InterfaceMetric
Get-NetRoute -DestinationPrefix 192.168.1.0/24
```

### Add Routes

**Temporary (lost on reboot):**
```powershell
route add 10.0.0.0 MASK 255.255.0.0 192.168.1.1
route add 172.16.0.0 MASK 255.255.0.0 192.168.1.1 metric 20
```

**Permanent (survives reboot):**
```powershell
route add -p 10.0.0.0 MASK 255.255.0.0 192.168.1.1
```

**With metric:**
```powershell
route add 192.168.50.0 MASK 255.255.255.0 192.168.1.1 metric 5
```

### Delete Routes

```powershell
route delete 10.0.0.0 MASK 255.255.0.0
route delete 172.16.0.0 MASK 255.255.0.0

# Or PowerShell:
Remove-NetRoute -DestinationPrefix 10.0.0.0/16 -Confirm:$false
```

### Modify Routes

Windows doesn't have direct modify; use delete + add:
```powershell
route delete 10.0.0.0 MASK 255.255.0.0
route add 10.0.0.0 MASK 255.255.0.0 192.168.1.2 metric 10
```

---

## Routing Decision Process

**Step-by-step routing:**

```
1. Packet arrives at router
   Destination: 8.8.8.8

2. Check route table for matches
   ✓ 0.0.0.0/0 matches (default)
   ✗ 192.168.1.0/24 doesn't match
   ✗ 10.0.0.0/8 doesn't match

3. Apply longest prefix match
   (Most specific route wins)
   Selected: 0.0.0.0/0

4. Get gateway from route
   Gateway: 192.168.1.1 (ISP router)

5. Send packet to gateway
   Frame it for 192.168.1.1

6. If no match → DROP
   (Unreachable - no route)
```

---

## Route Matching Examples

### Example 1: Local Network Traffic

```
Routes in table:
  0.0.0.0/0 → 192.168.1.1
  192.168.1.0/24 → On-link

Destination: 192.168.1.50

Process:
  0.0.0.0/0 → matches (everything matches /0)
  192.168.1.0/24 → matches (192.168.1.50 is in 192.168.1.x)
  
Longest match: 192.168.1.0/24 (/24 is longer than /0)
Result: On-link (send directly, no router needed)
```

### Example 2: Remote Network Traffic

```
Routes in table:
  0.0.0.0/0 → 192.168.1.1
  192.168.1.0/24 → On-link
  10.0.0.0/8 → 10.0.0.1

Destination: 172.16.0.5

Process:
  0.0.0.0/0 → matches
  Others → don't match
  
Result: Use default route (0.0.0.0/0) → 192.168.1.1
```

### Example 3: Specific Route Priority

```
Routes in table:
  0.0.0.0/0 → 192.168.1.1 (metric 10)
  10.0.0.0/8 → 192.168.1.2 (metric 5)
  10.1.0.0/16 → 192.168.1.3 (metric 2)

Destination: 10.1.5.50

Matches:
  0.0.0.0/0 → /0 prefix
  10.0.0.0/8 → /8 prefix
  10.1.0.0/16 → /16 prefix

Longest match: 10.1.0.0/16 (metric 2)
Result: Send to 192.168.1.3
```

---

## Common Subnetting Scenarios

### Dividing /24 into Smaller Subnets

```
192.168.1.0/24  →  254 hosts

Divide into /25 (2 subnets):
  192.168.1.0/25      → .0 to .127
  192.168.1.128/25    → .128 to .255

Divide into /26 (4 subnets):
  192.168.1.0/26      → .0 to .63
  192.168.1.64/26     → .64 to .127
  192.168.1.128/26    → .128 to .191
  192.168.1.192/26    → .192 to .255

Divide into /27 (8 subnets):
  192.168.1.0/27      → .0 to .31
  192.168.1.32/27     → .32 to .63
  192.168.1.64/27     → .64 to .95
  192.168.1.96/27     → .96 to .127
  ... and so on (32 address increments)
```

---

## Default Gateway and Route

### What is the Default Route?

```
Destination: 0.0.0.0
Netmask: 0.0.0.0
CIDR: /0
Meaning: ANY IP address (0 matches everything)

This is the "fallback" route:
- If no specific route matches → use default
- Usually points to ISP gateway
- Example: 0.0.0.0/0 → 192.168.1.1
```

### Finding Your Default Gateway

**Windows:**
```powershell
ipconfig              # Look for "Default Gateway"
route print | find "0.0.0.0"   # Find default route
```

**Expected output:**
```
Default Gateway . . . . . . . . . : 192.168.1.1

IPv4 Route Table:
0.0.0.0          0.0.0.0          192.168.1.1      192.168.1.50    25
```

---

## Metric and Route Priority

### Metric Values (Lower = Better)

```
Route 1:  192.168.1.1  metric 5   ← Preferred (lower)
Route 2:  192.168.1.2  metric 10  ← Fallback (higher)

Priority: Route 1 is used first
```

### When to Use Metrics

```
Primary path (fast, reliable):    metric 5
Backup path (slower, less reliable): metric 20
Avoid this path:                    metric 100
```

---

## Static Route Persistence (Windows)

### Checking if Route is Persistent

```powershell
route print
# Routes with "Persist" in output are permanent
# No "Persist" = temporary (lost on reboot)
```

### Making Routes Persistent

**Add with -p flag:**
```powershell
route add -p 10.0.0.0 MASK 255.255.0.0 192.168.1.1
```

**Remove persistent route:**
```powershell
route delete 10.0.0.0
# Must use same netmask as when added
```

**Check persistent routes only:**
```powershell
ipconfig /displaydns    # DNS cache
route print             # Includes persistent
```

---

## Troubleshooting Quick Guide

### Route Not Working?

**Checklist:**

```
□ 1. Route exists?
   ipconfig /all
   route print | find "destination"

□ 2. Destination correct?
   route print
   Double-check CIDR block

□ 3. Gateway reachable?
   ping [gateway-ip]

□ 4. Persistent or temporary?
   If temporary, lost on reboot - use -p flag

□ 5. Metric too high?
   route print
   Check if another route has lower metric

□ 6. Syntax correct?
   route add [destination] MASK [mask] [gateway]
   NOT: route add [destination]/[CIDR]
```

### Longest Prefix Match Not Working?

```
Problem: Wrong route being used

Solution:
□ Check for more specific routes
  route print | sort (visually look for longer matches)

□ Check metric values
  Longer prefix should have lower metric

□ Delete conflicting routes
  route delete [conflicting-route]
```

### No Default Route?

```
Problem: Destination unreachable for internet

Solution:
□ Add default route:
  route add 0.0.0.0 MASK 0.0.0.0 [gateway-ip]

□ Or verify existing:
  route print | find "0.0.0.0"
```

---

## Memory Aids

### CIDR Quick Estimation

```
/24 = 256 hosts         (most common)
/25 = 128 hosts         (1/2 of /24)
/26 = 64 hosts          (1/4 of /24)
/27 = 32 hosts          (1/8 of /24)
/28 = 16 hosts          (1/16 of /24)
/30 = 4 hosts           (router links)
/32 = 1 host            (single IP)
```

### Subnet Mask Octet Values

```
255 = all 1s  (full 8 bits for network)
  0 = all 0s  (full 8 bits for host)

128 = 10000000 (1 network bit)
192 = 11000000 (2 network bits)
224 = 11100000 (3 network bits)
240 = 11110000 (4 network bits)
248 = 11111000 (5 network bits)
252 = 11111100 (6 network bits)
254 = 11111110 (7 network bits)
```

### Private IP Ranges

```
10.0.0.0      (10 + /8)         Big networks
172.16.0.0    (1726 + /12)      Medium networks
192.168.0.0   (19216 + /16)     Home/small office
```

### Route Destination Remember

```
0.0.0.0       = Any IP (default route)
127.0.0.0     = Localhost (your computer)
192.168.x.x   = Private local network
224.0.0.0     = Multicast (special)
255.255.255.255 = Broadcast (everyone)
```

---

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| `route add 10.0.0.0/24` | Wrong syntax (CIDR doesn't work) | Use `MASK 255.255.255.0` instead |
| Adding temp route, loses on reboot | Forgot -p flag | Use `route add -p` for persistent |
| Longest prefix match ignored | Metric too high | Check route print, use lower metric |
| Local network route missing | Manual deletion | `route print` and manually re-add |
| Can't reach recently added route | Incomplete ping time | Wait 1-2 seconds, try again |

---

## Reference Formula Sheet

### Host Calculation
```
Total Hosts = 2^(32 - CIDR)
Usable = 2^(32 - CIDR) - 2

Example: /24
2^(32-24) = 2^8 = 256 total
256 - 2 = 254 usable
```

### Network Address from CIDR
```
Convert CIDR to binary mask
AND with IP address
Result = network address

Example: 192.168.1.50 with /24
192.168.1.x with /24 mask = 192.168.1.0
```

### Broadcast Address
```
Network address + (hosts - 1)

Example: 192.168.1.0 /24
192.168.1.0 + 255 = 192.168.1.255
```

---

## Additional Resources

**Windows Commands:**
- `route print` - View routing table
- `route add` - Add route
- `route delete` - Remove route
- `ipconfig /all` - Show network config
- `tracert` - Trace route to destination
- `ping` - Test connectivity
- `netstat -r` - Alternative route view

**PowerShell Commands:**
- `Get-NetRoute` - Modern route listing
- `New-NetRoute` - Modern add route
- `Remove-NetRoute` - Modern delete route
- `Get-NetIPConfiguration` - IP configuration

**Testing Commands:**
```powershell
ping 192.168.1.1           # Test gateway
tracert 8.8.8.8            # See path to internet
pathping 8.8.8.8           # Detailed path analysis
```

