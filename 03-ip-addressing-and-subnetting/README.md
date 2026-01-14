# 03 - IP Addressing and Subnetting

## What You'll Learn

By the end of this module, you'll be able to:
- Understand IPv4 address structure and classes
- Calculate network and broadcast addresses
- Determine valid host ranges from a subnet
- Convert between decimal and binary representations
- Subnet larger networks into smaller networks
- Plan IP addressing for organizations
- Understand IPv6 basics

## Prerequisites

- Completion of Module 01 (Networking Fundamentals)
- Basic understanding of binary numbers (helpful but not required)
- Familiarity with ipconfig and network configuration

## Key Concepts

### IPv4 Address Structure

An IPv4 address is 32 bits (4 octets) separated by dots:

```
192.168.1.100
│   │   │  │
│   │   │  └─ Octet 4 (host bits)
│   │   └───── Octet 3 (network + host)
│   └───────── Octet 2 (network)
└───────────── Octet 1 (network)
```

**Each octet = 8 bits = decimal 0-255**

Example: 192.168.1.100
```
192        = 11000000 (network)
168        = 10101000 (network)
1          = 00000001 (network/host)
100        = 01100100 (host)
```

### Subnet Mask

The subnet mask determines which bits are **network** and which are **host**:

```
255.255.255.0
│   │   │   │
All 1s = network bits
        All 0s = host bits
```

In binary:
```
255        = 11111111 (all network)
255        = 11111111 (all network)
255        = 11111111 (all network)
0          = 00000000 (all host)
```

**Subnet Mask 255.255.255.0 = /24 notation**

### IP Address Classes (Legacy)

| Class | First Octet | Default Mask | Private Range | Usage |
|-------|-------------|--------------|---------------|-------|
| A | 1-126 | 255.0.0.0 | 10.0.0.0/8 | Large networks |
| B | 128-191 | 255.255.0.0 | 172.16.0.0/12 | Medium networks |
| C | 192-223 | 255.255.255.0 | 192.168.0.0/16 | Small networks |
| D | 224-239 | N/A | N/A | Multicast |
| E | 240-255 | N/A | N/A | Reserved |

### Private IP Ranges

These addresses cannot be routed on the internet:

- **10.0.0.0 to 10.255.255.255** (Class A) - 16.7 million addresses
- **172.16.0.0 to 172.31.255.255** (Class B) - 1 million addresses
- **192.168.0.0 to 192.168.255.255** (Class C) - 65,536 addresses
- **127.0.0.0 to 127.255.255.255** - Loopback/localhost

### Subnet Calculations

For any IP and subnet mask:

1. **Network Address** = IP AND Subnet Mask (all host bits = 0)
2. **Broadcast Address** = Network OR inverted Mask (all host bits = 1)
3. **First Host** = Network Address + 1
4. **Last Host** = Broadcast Address - 1
5. **Number of Hosts** = 2^(host bits) - 2

### IPv6 Basics

IPv6 addresses are 128 bits (much larger than IPv4):

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334

Shortened: 2001:db8:85a3::8a2e:370:7334
```

**Benefits:**
- 340 trillion trillion trillion addresses
- Better routing
- Built-in security (IPSec)
- Autoconfiguration

## Hands-on Lab: Subnet a Network

### Objective
Divide a network into smaller subnets and identify all addresses.

### Step 1: Understand Your Current Network

```powershell
ipconfig /all
```

**Expected Output:**
```
IPv4 Address: 192.168.1.100
Subnet Mask: 255.255.255.0
```

**Analysis:**
- Network: 192.168.1.0/24
- Range: 192.168.1.1 to 192.168.1.254
- Broadcast: 192.168.1.255
- Total hosts: 254

### Step 2: Manually Calculate Subnet Information

**Given:** 192.168.1.0/24 (your current network)

Convert to binary:
```
192.168.1.0 = 11000000.10101000.00000001.00000000
Mask /24    = 11111111.11111111.11111111.00000000
```

**Calculations:**
- Network: 192.168.1.0 (last octet = 0)
- Broadcast: 192.168.1.255 (last octet = 255)
- First host: 192.168.1.1
- Last host: 192.168.1.254
- Total hosts: 254

### Step 3: Subnet Division (Divide /24 into /25s)

**Problem:** Split 192.168.1.0/24 into 2 subnets of 128 addresses each

**Solution:**
```
Original:    192.168.1.0/24   (256 addresses)
Subnets:     
Subnet 1:    192.168.1.0/25   (128 addresses: .0-.127)
Subnet 2:    192.168.1.128/25 (128 addresses: .128-.255)
```

**Detailed Breakdown:**

Subnet 1 (192.168.1.0/25):
- Network: 192.168.1.0
- Broadcast: 192.168.1.127
- First host: 192.168.1.1
- Last host: 192.168.1.126
- Hosts: 126

Subnet 2 (192.168.1.128/25):
- Network: 192.168.1.128
- Broadcast: 192.168.1.255
- First host: 192.168.1.129
- Last host: 192.168.1.254
- Hosts: 126

### Step 4: Further Subdivision

**Divide Subnet 1 (192.168.1.0/25) into /26s:**

```
192.168.1.0/26    (0-63)      - 62 hosts
192.168.1.64/26   (64-127)    - 62 hosts
```

### Step 5: Create Addressing Plan

```
Department       | Subnet         | Range              | Hosts
-----------------|----------------|-------------------|-------
Management       | 192.168.1.0/26 | .1 to .62          | 62
Employees        | 192.168.1.64/26| .65 to .126        | 62
Servers          | 192.168.1.128/25| .129 to .254      | 126
```

### Step 6: Verify with PowerShell

```powershell
# Create a custom calculation (manual verification)
$ip = "192.168.1.100"
$mask = "255.255.255.0"

# These would be calculated manually
Write-Host "IP: $ip"
Write-Host "Mask: $mask"
Write-Host "Network: 192.168.1.0"
Write-Host "Broadcast: 192.168.1.255"
Write-Host "Usable hosts: 254"
```

## Validation Checklist

- [ ] Can identify network, broadcast, and host range from IP/mask
- [ ] Can convert subnet mask to CIDR notation (/24, /25, etc.)
- [ ] Can calculate number of hosts in a subnet
- [ ] Can subnet a network into smaller pieces
- [ ] Understand private IP address ranges
- [ ] Can identify IP address class
- [ ] Know the difference between network and broadcast addresses
- [ ] Understand basic IPv6 concept

## Cleanup

No cleanup needed. All calculations are theoretical.

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Forgetting -2 for hosts | Subnet address and broadcast don't count |
| Confusing /24 and 255.255.255.0 | They're the same thing |
| Including broadcast in usable range | Broadcast is for all devices, not individual use |
| Wrong binary conversion | Practice converting decimal ↔ binary |
| Not understanding "all 0s" for network | Network = all host bits are 0 |
| Not understanding "all 1s" for broadcast | Broadcast = all host bits are 1 |

## Troubleshooting

### Problem: "I don't understand binary"
**Solution:**
1. Start with one octet (8 bits = 0-255)
2. Practice: 128 + 64 + 32 + 16 + 8 + 4 + 2 + 1 = 255
3. Memorize powers of 2: 1, 2, 4, 8, 16, 32, 64, 128
4. Convert a few numbers until pattern clicks

### Problem: "Subnet calculations are too hard"
**Solution:**
1. Use the formula: 2^n where n = host bits
2. For /25: 2 subnets (2^1), 128 addresses each (2^7)
3. For /26: 4 subnets (2^2), 64 addresses each (2^6)
4. Practice one type until comfortable

### Problem: "I get wrong broadcast addresses"
**Solution:**
1. Network address = all host bits = 0
2. Broadcast = all host bits = 1
3. Example: .0 and .127 are boundaries for /25
4. Last address before boundary change = broadcast

## Next Steps

- **Module 04:** Learn CIDR notation and routing tables
- **Module 05:** Understand how DNS resolves addresses
- Practice subnet calculations until automatic
- Create addressing plan for sample network

## Resources

- [Subnet Calculator Online](https://www.calculator.net/ip-subnet-calculator.html)
- [Binary Converter Tool](https://www.rapidtables.com/convert/number/binary-converter.html)
- [IPv4 Addressing Architecture](https://tools.ietf.org/html/rfc791)
- [IPv6 Basics](https://www.cisco.com/c/en/us/support/docs/ip/ip-version-6/6051-ipv6-summary.html)

## Quick Reference Tables

### Subnet Masks to CIDR

| Mask | CIDR | Hosts | Use |
|------|------|-------|-----|
| 255.255.255.0 | /24 | 254 | Standard LAN |
| 255.255.255.128 | /25 | 126 | Half LAN |
| 255.255.255.192 | /26 | 62 | Quarter LAN |
| 255.255.255.224 | /27 | 30 | Small subnet |
| 255.255.255.240 | /28 | 14 | Very small |
| 255.255.0.0 | /16 | 65,534 | Large network |

### Binary to Decimal (Single Octet)

| 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 | = Decimal |
|-----|----|----|----|----|----|----|---|----------|
| 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | = 255 |
| 1 | 1 | 1 | 1 | 1 | 1 | 1 | 0 | = 254 |
| 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | = 128 |
| 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | = 64 |
| 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | = 32 |
