# Cheatsheet: IP Addressing and Subnetting

Quick reference for IP addressing and subnetting calculations.

## IPv4 Address Structure

**Format:** `A.B.C.D` where each letter = 8 bits (0-255)

```
192.168.1.100
├─ 192   = Octet 1 (bits 25-32)
├─ 168   = Octet 2 (bits 17-24)
├─ 1     = Octet 3 (bits 9-16)
└─ 100   = Octet 4 (bits 1-8)

Total = 32 bits
```

---

## CIDR to Subnet Mask Conversion

| CIDR | Network Bits | Decimal Mask | Binary |
|------|-------------|--------------|--------|
| /8 | 8 | 255.0.0.0 | 11111111.00000000.00000000.00000000 |
| /16 | 16 | 255.255.0.0 | 11111111.11111111.00000000.00000000 |
| /24 | 24 | 255.255.255.0 | 11111111.11111111.11111111.00000000 |
| /25 | 25 | 255.255.255.128 | 11111111.11111111.11111111.10000000 |
| /26 | 26 | 255.255.255.192 | 11111111.11111111.11111111.11000000 |
| /27 | 27 | 255.255.255.224 | 11111111.11111111.11111111.11100000 |
| /28 | 28 | 255.255.255.240 | 11111111.11111111.11111111.11110000 |
| /29 | 29 | 255.255.255.248 | 11111111.11111111.11111111.11111000 |
| /30 | 30 | 255.255.255.252 | 11111111.11111111.11111111.11111100 |
| /31 | 31 | 255.255.255.254 | 11111111.11111111.11111111.11111110 |
| /32 | 32 | 255.255.255.255 | 11111111.11111111.11111111.11111111 |

---

## Last Octet Subnet Masks (Most Common)

| Decimal | Binary | Network Bits | Host Bits | Hosts | CIDR (with /24) |
|---------|--------|-------------|-----------|-------|-----------------|
| 0 | 00000000 | 0 | 8 | 256 | /24 |
| 128 | 10000000 | 1 | 7 | 128 | /25 |
| 192 | 11000000 | 2 | 6 | 64 | /26 |
| 224 | 11100000 | 3 | 5 | 32 | /27 |
| 240 | 11110000 | 4 | 4 | 16 | /28 |
| 248 | 11111000 | 5 | 3 | 8 | /29 |
| 252 | 11111100 | 6 | 2 | 4 | /30 |
| 254 | 11111110 | 7 | 1 | 2 | /31 |
| 255 | 11111111 | 8 | 0 | 1 | /32 |

---

## Subnet Calculation Formulas

**Total Addresses in Subnet:**
$$\text{Total} = 2^{\text{(32 - CIDR)}}$$

**Usable Hosts:**
$$\text{Usable} = 2^{\text{(32 - CIDR)}} - 2$$

**Number of Subnets from larger network:**
$$\text{Subnets} = 2^{\text{(New CIDR - Original CIDR)}}$$

**Host bits increment (subnet size):**
$$\text{Increment} = 2^{\text{(32 - CIDR)}}$$

---

## Quick Host Count Reference

| CIDR | Mask | Total | Usable | Use Case |
|------|------|-------|--------|----------|
| /30 | 255.255.255.252 | 4 | 2 | Point-to-point (router links) |
| /29 | 255.255.255.248 | 8 | 6 | Small office (ISP links) |
| /28 | 255.255.255.240 | 16 | 14 | Small department |
| /27 | 255.255.255.224 | 32 | 30 | Department |
| /26 | 255.255.255.192 | 64 | 62 | Small office |
| /25 | 255.255.255.128 | 128 | 126 | Medium office |
| /24 | 255.255.255.0 | 256 | 254 | Standard LAN |
| /23 | 255.255.254.0 | 512 | 510 | Large office |
| /22 | 255.255.252.0 | 1,024 | 1,022 | Campus |
| /21 | 255.255.248.0 | 2,048 | 2,046 | Large campus |
| /20 | 255.255.240.0 | 4,096 | 4,094 | Multi-building |
| /16 | 255.255.0.0 | 65,536 | 65,534 | Organization |
| /8 | 255.0.0.0 | 16.7M | 16.7M | ISP |

---

## IP Address Classes (Traditional)

| Class | First Octet | Range | Default Mask | Purpose |
|-------|------------|-------|--------------|---------|
| A | 1-126 | 1.x.x.x - 126.x.x.x | 255.0.0.0 | Large networks |
| B | 128-191 | 128.x.x.x - 191.x.x.x | 255.255.0.0 | Medium networks |
| C | 192-223 | 192.x.x.x - 223.x.x.x | 255.255.255.0 | Small networks |
| D | 224-239 | 224.x.x.x - 239.x.x.x | N/A | Multicast |
| E | 240-255 | 240.x.x.x - 255.x.x.x | N/A | Reserved |

---

## Private IP Address Ranges

**RFC 1918 - Reserved for Private Use:**

| Range | CIDR | Mask | Size | Class |
|-------|------|------|------|-------|
| 10.0.0.0 - 10.255.255.255 | 10.0.0.0/8 | 255.0.0.0 | 16.7M hosts | A |
| 172.16.0.0 - 172.31.255.255 | 172.16.0.0/12 | 255.240.0.0 | 1M hosts | B |
| 192.168.0.0 - 192.168.255.255 | 192.168.0.0/16 | 255.255.0.0 | 65k hosts | C |

**Special Addresses:**

| Address/Range | Purpose |
|---------------|---------|
| 127.0.0.1 | Loopback (localhost) |
| 0.0.0.0 | Default route |
| 255.255.255.255 | Broadcast all networks |
| x.x.x.0 | Network address |
| x.x.x.255 | Broadcast address |
| 169.254.x.x | Link-local (APIPA) |
| 224.0.0.0/4 | Multicast |

---

## Binary to Decimal Reference

**Powers of 2 in an Octet:**

```
Bit Position: 7    6    5    4    3    2    1    0
Decimal:      128  64   32   16   8    4    2    1

Binary: 11000000 = 128 + 64 = 192
Binary: 10101000 = 128 + 32 + 8 = 168
Binary: 11111111 = 128+64+32+16+8+4+2+1 = 255
Binary: 10000000 = 128
Binary: 01000000 = 64
```

**Quick Conversion Table:**

| Decimal | Binary |
|---------|--------|
| 0 | 00000000 |
| 1 | 00000001 |
| 2 | 00000010 |
| 4 | 00000100 |
| 8 | 00001000 |
| 16 | 00010000 |
| 32 | 00100000 |
| 64 | 01000000 |
| 128 | 10000000 |
| 192 | 11000000 |
| 224 | 11100000 |
| 240 | 11110000 |
| 248 | 11111000 |
| 252 | 11111100 |
| 254 | 11111110 |
| 255 | 11111111 |

---

## Network Calculation Cheat Sheet

**Given an IP and mask, find:**

### Network Address
```
IP: 192.168.1.100
Mask: 255.255.255.0

Method: Set all host bits to 0
Result: 192.168.1.0
```

### Broadcast Address
```
IP: 192.168.1.100
Mask: 255.255.255.0 (/24)

Method: Set all host bits to 1
Result: 192.168.1.255

Or: Network + (2^host_bits - 1) = 0 + 255 = .255
```

### First Host
```
Network: 192.168.1.0
First usable: 192.168.1.1
```

### Last Host
```
Broadcast: 192.168.1.255
Last usable: 192.168.1.254
```

### Host Count
```
/24 = 32 - 24 = 8 bits
Hosts = 2^8 - 2 = 254
```

---

## Subnetting Quick Reference

### Divide a /24 into smaller subnets

| Need | CIDR | Subnets | Size | Increment |
|------|------|---------|------|-----------|
| 2 subnets | /25 | 2 | 128 | 128 |
| 4 subnets | /26 | 4 | 64 | 64 |
| 8 subnets | /27 | 8 | 32 | 32 |
| 16 subnets | /28 | 16 | 16 | 16 |
| 32 subnets | /29 | 32 | 8 | 8 |

### Divide a /16 into /24s
```
10.0.0.0/16 divided into /24s:
10.0.0.0/24, 10.0.1.0/24, 10.0.2.0/24 ... 10.0.255.0/24
(256 total /24 networks)
```

### Divide a /16 into /25s
```
10.0.0.0/16 divided into /25s:
10.0.0.0/25, 10.0.0.128/25, 10.0.1.0/25, 10.0.1.128/25 ...
(512 total /25 networks)
```

---

## IPv4 vs IPv6 Quick Comparison

| Aspect | IPv4 | IPv6 |
|--------|------|------|
| Bits | 32 bits | 128 bits |
| Address Space | 4.3 billion | 340 undecillion |
| Format | Decimal (0.0.0.0) | Hexadecimal (::1) |
| Example | 192.168.1.1 | 2001:0db8::1 |
| Notation | dotted decimal | colon-separated hex |
| Loopback | 127.0.0.1 | ::1 |
| Broadcast | yes | no (multicast instead) |
| Compression | no | yes (::) |

---

## Windows Commands for IP Configuration

| Command | Purpose |
|---------|---------|
| `ipconfig` | Show IP configuration |
| `ipconfig /all` | Show detailed config |
| `ipconfig /release` | Release DHCP lease |
| `ipconfig /renew` | Renew DHCP lease |
| `netstat -an` | Show network connections |
| `route print` | Display routing table |
| `arp -a` | Show ARP cache |
| `nslookup google.com` | Query DNS |
| `ping 192.168.1.1` | Test connectivity |
| `tracert google.com` | Trace route to host |

---

## Linux/Mac Commands for IP Configuration

| Command | Purpose |
|---------|---------|
| `ip addr show` | Show IP addresses |
| `ifconfig` | Show network config (older) |
| `ip route show` | Display routing table |
| `arp -a` | Show ARP cache |
| `dig google.com` | DNS query |
| `nslookup google.com` | Query DNS |
| `ping 192.168.1.1` | Test connectivity |
| `traceroute google.com` | Trace route to host |

---

## Subnetting Decision Tree

```
Need to subnet a network?
│
├─ How many subnets needed?
│  │
│  ├─ 2? → /25 (add 1 bit)
│  ├─ 4? → /26 (add 2 bits)
│  ├─ 8? → /27 (add 3 bits)
│  ├─ 16? → /28 (add 4 bits)
│  └─ More? → Calculate: need N bits where 2^N ≥ subnets needed
│
└─ What size per subnet?
   │
   ├─ Point-to-point? → /30 (2 hosts)
   ├─ Small office? → /28-/26 (14-62 hosts)
   ├─ Department? → /25-/24 (126-254 hosts)
   ├─ Large campus? → /22-/20 (1K-4K hosts)
   └─ Multi-building? → /16-/8 (65K+ hosts)
```

---

## Common Mistakes to Avoid

| Mistake | Correct |
|---------|---------|
| Using network address for device | Assign from first host (x.x.x.1) |
| Using broadcast address | Don't assign broadcast to device |
| Overlapping subnets | Plan non-overlapping ranges |
| Wrong CIDR math | Count network bits correctly |
| Forgetting -2 (net & bcast) | Usable = 2^bits - 2 |
| Wrong subnet increment | = 256 / number_of_subnets |
| Mixing binary/decimal | Convert consistently |
| Not planning growth | Leave 30-40% unused |

---

## Step-by-Step Subnetting Example

**Problem:** Subnet 192.168.0.0/24 into 4 equal parts

**Step 1:** How many bits for 4 subnets?
$$2^x = 4 \rightarrow x = 2$$
Add 2 bits to CIDR: 24 + 2 = /26

**Step 2:** Subnet size
$$2^{32-26} = 2^6 = 64 \text{ addresses}$$

**Step 3:** Subnet increment
$$256 / 4 = 64 \text{ (or use subnet size)}$$

**Step 4:** List subnets
- Subnet 1: 192.168.0.0/26 (0-63)
- Subnet 2: 192.168.0.64/26 (64-127)
- Subnet 3: 192.168.0.128/26 (128-191)
- Subnet 4: 192.168.0.192/26 (192-255)

**Step 5:** Identify usable ranges (for each subnet)
```
Subnet 1:
  Network: 192.168.0.0
  First host: 192.168.0.1
  Last host: 192.168.0.62
  Broadcast: 192.168.0.63

Subnet 2:
  Network: 192.168.0.64
  First host: 192.168.0.65
  Last host: 192.168.0.126
  Broadcast: 192.168.0.127

(etc. for Subnets 3 & 4)
```

---

## VLSM (Variable Length Subnet Masks) Quick Guide

**Problem:** Need networks of different sizes
- Dept A: 100 hosts
- Dept B: 50 hosts
- Dept C: 200 hosts
- Server link: 2 hosts

**Solution: Use different CIDR for each**
```
Dept A: 192.168.1.0/25 (126 hosts) ✓
Dept B: 192.168.1.128/26 (62 hosts) ✓
Dept C: 192.168.2.0/23 (510 hosts) ✓
Link: 192.168.4.0/30 (2 hosts) ✓
```

**Key:** Start with largest, work down to smallest

---

## IPv6 Basics (Quick Intro)

**Format:** 8 groups of 4 hex digits
```
Example: 2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

**Compressed:**
```
2001:db8:85a3::8a2e:370:7334
(:: represents consecutive zeros)
```

**Loopback:** `::1`

**Link-local:** `fe80::1`

**CIDR:** Same concept as IPv4
- `/64` = common subnet size
- `/48` = typical organization allocation

---

## Subnet Calculator Formula Cheat

| Want to Find | Formula | Example |
|--------------|---------|---------|
| Total addresses | 2^(32-CIDR) | /24 → 2^8 = 256 |
| Usable hosts | 2^(32-CIDR) - 2 | /24 → 254 |
| Broadcast | Network + 2^(32-CIDR) - 1 | 192.168.1.0 + 255 = .255 |
| Last host | Broadcast - 1 | .255 - 1 = .254 |
| Number of subnets | 2^(new_bits) | /24→/26 = 2^2 = 4 |
| Increment | 2^(32-CIDR) | /26 → 64 |

