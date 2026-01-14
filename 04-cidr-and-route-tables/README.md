# 04 - CIDR and Route Tables

## What You'll Learn

By the end of this module, you'll be able to:
- Understand CIDR notation and how it simplifies IP addressing
- Convert between CIDR notation and subnet masks
- Design efficient IP addressing schemes using CIDR
- Understand routing tables and how routers make decisions
- Interpret and modify routing tables
- Configure static routes
- Understand routing algorithms and protocols

## Prerequisites

- Completion of Module 03 (IP Addressing and Subnetting)
- Understanding of binary notation and IP addressing
- Familiarity with network commands (ping, tracert, route print)

## Key Concepts

### CIDR (Classless Inter-Domain Routing)

CIDR is a method for allocating IP addresses and routing based on variable-length subnet masks (VLSM) instead of the old classful system.

**Classful vs Classless:**

| Concept | Classful | Classless (CIDR) |
|---------|----------|------------------|
| Notation | 192.168.1.0 + 255.255.255.0 | 192.168.1.0/24 |
| Flexibility | Fixed block sizes (Class A, B, C) | Any block size |
| Efficiency | Wasteful (Class B has 65,536 IPs) | Efficient (only what you need) |
| Adoption | 1980s-1990s (outdated) | 1993-present (standard) |
| Routing | More routing table entries | Summarization (fewer entries) |

### CIDR Notation Explained

**Format:** `IP_Address/Prefix_Length`

- **192.168.1.0/24** = Network with 24 bits for network, 8 bits for hosts
- **/24** is equivalent to 255.255.255.0 (standard subnet mask)
- **/25** = 128 hosts, **/26** = 64 hosts, **/30** = 4 hosts

**Common CIDR Blocks:**

| CIDR | Subnet Mask | Hosts | Use Case |
|------|-------------|-------|----------|
| /8 | 255.0.0.0 | 16,777,214 | Very large enterprise |
| /16 | 255.255.0.0 | 65,534 | Large enterprise |
| /24 | 255.255.255.0 | 254 | Standard office network |
| /25 | 255.255.255.128 | 126 | Small department |
| /26 | 255.255.255.192 | 62 | Workgroup |
| /27 | 255.255.255.224 | 30 | Small segment |
| /30 | 255.255.255.252 | 2 | Point-to-point links (routers) |
| /31 | 255.255.255.254 | 0 | Loopback/special |
| /32 | 255.255.255.255 | 1 | Single host |

### Route Tables

A routing table is a set of rules (called routes) that determine where network traffic is directed. Every router uses a routing table.

**Route Table Structure:**

| Destination | Subnet Mask | Gateway | Interface | Metric |
|-------------|-------------|---------|-----------|--------|
| 192.168.1.0 | 255.255.255.0 | On-link | Ethernet | 10 |
| 0.0.0.0 | 0.0.0.0 | 192.168.1.1 | Ethernet | 0 |

**Or in CIDR notation:**

| Destination | Gateway | Interface | Metric |
|-------------|---------|-----------|--------|
| 192.168.1.0/24 | On-link | Ethernet | 10 |
| 0.0.0.0/0 | 192.168.1.1 | Ethernet | 0 |

### Routing Decision Process

When a packet arrives at a router:

1. **Look up destination IP** in routing table
2. **Find matching route** (longest prefix match wins)
3. **Forward packet** to gateway or directly to destination
4. **If no match** → send to default route (0.0.0.0/0)
5. **If no default** → drop packet (unreachable)

### Default Route

- **0.0.0.0/0** = "Any IP address I don't specifically know about"
- Usually points to Internet gateway/ISP router
- Last resort when no other route matches
- Every router needs one (except backbone routers)

## Hands-on Lab: Understanding Routing Tables and CIDR

### Objective
Understand your computer's routing table and how packets are routed.

### Step 1: View Your Current Routing Table

```powershell
route print
```

**Expected Output:**
```
Route Table
===========

IPv4 Route Table
===============

Active Routes:
Network Destination    Netmask       Gateway    Interface  Metric
          0.0.0.0    0.0.0.0  192.168.1.1 192.168.1.100  25
      127.0.0.0  255.0.0.0      On-link    127.0.0.1     306
    127.0.0.1  255.255.255.255 On-link    127.0.0.1     306
    192.168.1.0  255.255.255.0 On-link  192.168.1.100   275
  192.168.1.100  255.255.255.255 On-link 192.168.1.100   275
  192.168.1.255  255.255.255.255 On-link 192.168.1.100   275
  224.0.0.0    240.0.0.0       On-link 192.168.1.100   275
  255.255.255.255 255.255.255.255 On-link 192.168.1.100   275
```

**Analysis:**

| Route | Meaning |
|-------|---------|
| 0.0.0.0/0 → 192.168.1.1 | Default: everything to gateway (internet) |
| 127.0.0.0/8 → On-link | Loopback: local host only |
| 192.168.1.0/24 → On-link | Local network: directly reachable |

### Step 2: Convert Routes to CIDR Notation

Take these routes and convert to CIDR:

**Classful:**
```
Network: 192.168.1.0
Mask: 255.255.255.0
```

**CIDR:**
```
192.168.1.0/24
```

**How to Calculate:**
- Count the 1s in the subnet mask
- 255.255.255.0 = 11111111.11111111.11111111.00000000
- Count: 8 + 8 + 8 + 0 = 24
- Result: /24

### Step 3: Understand Route Matching

Test which route is used for different destinations:

```powershell
# This packet uses which route?
tracert 192.168.1.50
```

**Expected:**
- Destination is 192.168.1.50
- Matches route 192.168.1.0/24 (On-link)
- Sent directly to destination

```powershell
# This packet uses which route?
tracert 8.8.8.8
```

**Expected:**
- Destination is 8.8.8.8
- Doesn't match 192.168.1.0/24 or 127.0.0.0/8
- Matches default 0.0.0.0/0
- Sent to gateway 192.168.1.1

### Step 4: Add a Static Route

Add a new route to your routing table:

```powershell
# Add route to a network
route add 10.0.0.0 MASK 255.255.0.0 192.168.1.1
```

**Expected:**
```
Route table successfully modified
```

Verify it was added:
```powershell
route print | findstr 10.0.0.0
```

**Expected:**
```
      10.0.0.0  255.255.0.0  192.168.1.1 192.168.1.100   25
```

### Step 5: View in CIDR Notation (PowerShell)

```powershell
Get-NetRoute | Select-Object DestinationPrefix, NextHop, InterfaceMetric
```

**Expected Output:**
```
DestinationPrefix NextHop        InterfaceMetric
------------------ ---------        ---------------
0.0.0.0/0         192.168.1.1              25
127.0.0.1/32      On-link                 306
192.168.1.0/24    On-link                 275
10.0.0.0/16       192.168.1.1              25
```

**Key Points:**
- CIDR notation is clearer than classful
- /32 = single host
- /24 = standard office subnet
- /0 = default route (all traffic)

### Step 6: Delete the Test Route

```powershell
route delete 10.0.0.0 MASK 255.255.0.0
```

**Expected:**
```
Route table successfully modified
```

Verify deletion:
```powershell
route print | findstr 10.0.0.0
# (No output = deleted successfully)
```

## Validation Checklist

- [ ] Can convert subnet mask to CIDR notation
- [ ] Can convert CIDR notation to subnet mask
- [ ] Understand what /24, /16, /8, /32 mean
- [ ] Can read a routing table
- [ ] Understand default route (0.0.0.0/0)
- [ ] Can add and delete static routes
- [ ] Know why CIDR is better than classful
- [ ] Understand longest prefix match concept

## Cleanup

Delete the test route if not already deleted:

```powershell
route delete 10.0.0.0 MASK 255.255.0.0
```

## Common Mistakes

| Mistake | Solution |
|---------|----------|
| Confusing /24 with /32 | /24 = 256 hosts, /32 = 1 host |
| Adding routes to local network | Only add if not on-link (directly connected) |
| Forgetting default gateway | Every router needs 0.0.0.0/0 route |
| Routes not persistent after reboot | Use ROUTE ADD -P for persistent, or use PowerShell |
| Can't reach new route after adding | Check metric isn't higher than default |

## Troubleshooting

### Problem: "Route not working after adding"
**Solution:**
1. Verify route was added: `route print | findstr <destination>`
2. Check metric isn't too high: Lower metric = preferred
3. Check gateway is reachable: `ping <gateway-ip>`
4. Verify interface is up: `Get-NetAdapter`

### Problem: "Which route is being used for my destination?"
**Solution:**
1. Use `tracert <destination>` to see actual path
2. Use longest prefix match rule (most specific wins)
3. Example: For 192.168.1.100, /24 wins over /0

### Problem: "Default route not working"
**Solution:**
1. Verify gateway is correct: `ipconfig /all`
2. Check gateway is reachable: `ping <gateway-ip>`
3. Check for conflicting routes with higher priority

## Next Steps

- **Module 05:** DNS Fundamentals
- **Module 06:** HTTP, HTTPS, and TLS
- Practice subnetting with calculator tools
- Learn about dynamic routing (OSPF, BGP)
- Understand route summarization

## Resources

- [CIDR Calculator](https://cidr.xyz)
- [RFC 4632 - CIDR](https://tools.ietf.org/html/rfc4632)
- [Routing Table Guide](https://docs.microsoft.com/en-us/windows-server/networking/technologies/network-subsystem/network-subsystem)
