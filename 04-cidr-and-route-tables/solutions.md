# Solutions: CIDR and Route Tables

Complete answer key for all exercises with explanations and expected outputs.

## Easy Exercise Solutions

### Solution 1: Convert Subnet Masks to CIDR

**Complete Answer:**

| Subnet Mask | CIDR | Hosts | Explanation |
|-------------|------|-------|-------------|
| 255.255.255.0 | /24 | 254 | All octets have 8 bits except last one all 0s = 24 bits |
| 255.255.255.128 | /25 | 126 | 255.255.255 = 24 bits, 128 = 10000000 = 1 more bit = /25 |
| 255.255.255.192 | /26 | 62 | 255.255.255 = 24 bits, 192 = 11000000 = 2 more bits = /26 |
| 255.255.255.224 | /27 | 30 | 255.255.255 = 24 bits, 224 = 11100000 = 3 more bits = /27 |
| 255.255.0.0 | /16 | 65534 | Two octets = 16 bits |
| 255.0.0.0 | /8 | 16777214 | One octet = 8 bits |

**Key Insight:**
Each octet has 8 bits, so 255 (all 1s) = 8 bits. Count the 1s in the final octet:
- 128 = 10000000 = 1 bit → +1
- 192 = 11000000 = 2 bits → +2
- 224 = 11100000 = 3 bits → +3
- 240 = 11110000 = 4 bits → +4
- 248 = 11111000 = 5 bits → +5

---

### Solution 2: Convert CIDR to Subnet Masks

**Complete Answer:**

| CIDR | Subnet Mask | Hosts | Method |
|------|-------------|-------|--------|
| /24 | 255.255.255.0 | 254 | 32-24=8 host bits = .0 (all 0s) |
| /16 | 255.255.0.0 | 65534 | 32-16=16 host bits = 0.0 |
| /8 | 255.0.0.0 | 16777214 | 32-8=24 host bits = 0.0.0 |
| /25 | 255.255.255.128 | 126 | 32-25=7 host bits = .10000000 = .128 |
| /30 | 255.255.255.252 | 2 | 32-30=2 host bits = .11111100 = .252 |
| /32 | 255.255.255.255 | 1 | 32-32=0 host bits = all 1s (single host) |

**Key Insight:**
- /24 is most common (home/small networks)
- /30 is used for router links (only 2 usable IPs: one for each router)
- /32 is a single host route (rarely used for subnets, common for route entries)

**Host Calculation:**
Total hosts = 2^(32-CIDR)
Usable hosts = 2^(32-CIDR) - 2  (subtract network and broadcast addresses)

---

### Solution 3: Calculate Hosts in CIDR Blocks

**Complete Answer:**

| CIDR | Formula | Calculation | Total | Usable Hosts | Use Case |
|------|---------|-------------|-------|--------------|----------|
| /24 | 2^8-2 | 256-2 | 256 | 254 | Small office, home |
| /25 | 2^7-2 | 128-2 | 128 | 126 | Department networks |
| /26 | 2^6-2 | 64-2 | 64 | 62 | Team networks |
| /27 | 2^5-2 | 32-2 | 32 | 30 | Small teams/printers |
| /30 | 2^2-2 | 4-2 | 4 | 2 | Router links only |

**Host Formula:**
```
Usable Hosts = 2^(32 - CIDR Prefix) - 2

Why subtract 2?
- First address = Network address (cannot be assigned)
- Last address = Broadcast address (cannot be assigned)
- Everything in between = Available for hosts
```

**Quick Reference:**
- /24 = 254 hosts (most common)
- /25 = 126 hosts (half of /24)
- /26 = 62 hosts (quarter of /24)
- /27 = 30 hosts
- /28 = 14 hosts
- /29 = 6 hosts
- /30 = 2 hosts (router links)
- /31 = 0 hosts (point-to-point, no broadcast)
- /32 = single host

---

### Solution 4: View and Interpret Routing Tables

**Expected Output (Windows):**
```
C:\>route print
===========================================================================
Interface List
  15...00 15 5d f2 03 c8 ......Ethernet
===========================================================================

IPv4 Route Table
===========================================================================
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0      192.168.1.1    192.168.1.50    25
        127.0.0.0      255.0.0.0         On-link       127.0.0.1      306
      127.0.0.1  255.255.255.255         On-link       127.0.0.1      306
    127.255.255.255  255.255.255.255     On-link       127.0.0.1      306
      192.168.1.0  255.255.255.0         On-link     192.168.1.50      275
     192.168.1.50  255.255.255.255       On-link     192.168.1.50      275
    192.168.1.255  255.255.255.255       On-link     192.168.1.50      275
        224.0.0.0        240.0.0.0         On-link     192.168.1.50      275
  255.255.255.255  255.255.255.255       On-link     192.168.1.50      275
===========================================================================
```

**Answers (Example):**

1. **Default gateway IP:** 192.168.1.1
2. **Local network in CIDR:** 192.168.1.0/24
3. **Number of routes:** 9 routes

**Understanding Each Row:**

| Route | Type | Meaning |
|-------|------|---------|
| 0.0.0.0/0 | Default | Route all unknown traffic to gateway 192.168.1.1 |
| 127.0.0.0/8 | Loopback | Local machine (localhost), used for testing |
| 192.168.1.0/24 | Local | Your subnet, reached directly (On-link) |
| 224.0.0.0/4 | Multicast | Broadcast traffic (not typically used) |
| 255.255.255.255/32 | Broadcast | Local network broadcast |

**Column Meanings:**

- **Network Destination:** Where traffic is going
- **Netmask:** Subnet mask (how many bits identify network)
- **Gateway:** Where to send it (IP address or "On-link" for direct)
- **Interface:** Your network card address
- **Metric:** Priority/cost (lower = preferred)

---

### Solution 5: Match Destination to Route

**Complete Answer:**

| Destination | Route Used | CIDR | Why |
|-------------|------------|------|-----|
| 192.168.1.50 | 192.168.1.0/24 (On-link) | /24 | Exact match in local subnet |
| 8.8.8.8 | 0.0.0.0/0 (gateway) | /0 | Not in any specific route, use default |
| 127.0.0.1 | 127.0.0.0/8 (loopback) | /8 | Localhost, stay on computer |
| 10.0.0.5 | 0.0.0.0/0 (gateway) | /0 | Not in any specific route, use default |

**Longest Prefix Match Rule:**
The router looks for the most specific route (longest CIDR prefix) that matches.

**Step-by-step for 8.8.8.8:**
```
1. Check 192.168.1.0/24 → 8.8.8.8 doesn't match (not in 192.168.1.x)
2. Check 127.0.0.0/8 → 8.8.8.8 doesn't match (not 127.x.x.x)
3. Check 0.0.0.0/0 → 8.8.8.8 matches (everything matches /0)
4. Use default route with gateway 192.168.1.1
```

**Step-by-step for 192.168.1.50:**
```
1. Check 192.168.1.0/24 → 192.168.1.50 matches! (/24 matches first 24 bits)
2. Send directly to that subnet (On-link, no gateway needed)
```

---

## Medium Exercise Solutions

### Solution 6: Design IP Addressing Scheme Using CIDR

**Answer:**

| Department | Employees | CIDR Block | Subnet Mask | Total Hosts | Usable Hosts |
|------------|-----------|-----------|-------------|-------------|--------------|
| Floor 1 | 50 | 10.0.1.0/25 | 255.255.255.128 | 128 | 126 ✓ |
| Floor 2 | 30 | 10.0.2.0/26 | 255.255.255.192 | 64 | 62 ✓ |
| Floor 3 | 20 | 10.0.3.0/26 | 255.255.255.192 | 64 | 62 ✓ |
| Servers | 10 | 10.0.4.0/28 | 255.255.255.240 | 16 | 14 ✓ |
| Printers | 5 | 10.0.5.0/28 | 255.255.255.240 | 16 | 14 ✓ |

**Calculation Method:**

Floor 1 (50 employees + 20% growth = 60 devices needed):
- /24 = 254 hosts (wasteful - too many)
- /25 = 126 hosts (perfect - twice what needed)
- Choice: /25 ✓

Floor 2 (30 employees + 20% growth = 36 devices needed):
- /26 = 62 hosts (perfect)
- Choice: /26 ✓

Floor 3 (20 employees + 20% growth = 24 devices needed):
- /27 = 30 hosts (just fits)
- /26 = 62 hosts (better for growth)
- Choice: /26 (allows growth) ✓

Servers (10 servers + growth):
- /29 = 6 hosts (too small)
- /28 = 14 hosts (good for servers)
- Choice: /28 ✓

Printers (5 printers + growth):
- /29 = 6 hosts (just fits)
- /28 = 14 hosts (allows growth)
- Choice: /28 ✓

**Why This Design Works:**
- Efficient: No major waste
- Scalable: Room for 20% growth
- Manageable: Clear separation per floor
- Professional: Using proper subnet boundaries

---

### Solution 7: Add and Remove Static Routes

**Expected Output:**

```powershell
PS C:\> route add 172.16.0.0 MASK 255.255.0.0 192.168.1.1
The operation completed successfully.

PS C:\> route add 10.0.0.0 MASK 255.255.0.0 192.168.1.1
The operation completed successfully.

PS C:\> route print | findstr "172.16 10.0"
      172.16.0.0      255.255.0.0      192.168.1.1      192.168.1.50       25
       10.0.0.0      255.255.0.0      192.168.1.1      192.168.1.50       25

PS C:\> route delete 172.16.0.0 MASK 255.255.0.0
The operation completed successfully.

PS C:\> route delete 10.0.0.0 MASK 255.255.0.0
The operation completed successfully.

PS C:\> route print | findstr "172.16 10.0"
(no output = routes deleted)
```

**Answer:**

1. **Did routes appear immediately?** YES - Commands execute instantly, routes are in table right away
2. **Are they persistent after reboot?** NO - These are temporary routes, lost on restart
3. **What happens with duplicate destination?** Error - "The object already exists" or overwrites metric

**To Make Routes Permanent:**
```powershell
route add -p 172.16.0.0 MASK 255.255.0.0 192.168.1.1
```
The `-p` flag makes the route persistent (survives reboot)

**Checking if Route Exists (before deleting):**
```powershell
route print | findstr "172.16"
```

**Real-World Use:**
- Temporary routes: Testing, troubleshooting
- Permanent routes: Production routing, backup paths, VPN networks

---

### Solution 8: Analyze Real Network Traffic with CIDR

**Example Output (your results will vary):**

```
C:\>tracert google.com

Tracing route to google.com [142.250.185.46]
over a maximum of 30 hops:

  1    2 ms    2 ms    2 ms  192.168.1.1
  2   15 ms   15 ms   15 ms  10.0.0.1
  3   25 ms   26 ms   25 ms  203.0.113.1
  4   35 ms   35 ms   35 ms  198.51.100.1
  5   45 ms   46 ms   45 ms  192.0.2.5
  6   52 ms   52 ms   52 ms  142.250.185.1
  7  * * *  Request timed out.
  8   65 ms   65 ms   65 ms  142.250.185.46 [google.com]

Trace complete.
```

**Analysis Table (Example):**

| Hop | IP | CIDR Block | Private/Public | Region |
|-----|----|----|---------|--------|
| 1 | 192.168.1.1 | 192.168.1.0/24 | Private (Home) | Your home router |
| 2 | 10.0.0.1 | 10.0.0.0/8 | Private (ISP) | ISP gateway |
| 3 | 203.0.113.1 | 203.0.113.0/24 | Public | ISP backbone |
| 4 | 198.51.100.1 | 198.51.100.0/24 | Public | Internet backbone |
| 5 | 192.0.2.5 | 192.0.2.0/24 | Private (AS) | AS path |
| 6 | 142.250.185.1 | 142.250.185.0/24 | Public | Google network |
| 8 | 142.250.185.46 | Same as hop 6 | Public | Final destination |

**Private IP Ranges (RFC 1918):**
- 10.0.0.0/8 → 10.0.0.0 to 10.255.255.255
- 172.16.0.0/12 → 172.16.0.0 to 172.31.255.255
- 192.168.0.0/16 → 192.168.0.0 to 192.168.255.255

**Key Observations:**

1. **Hop 1 is always private** - It's your home/office router (gateway to internet)
2. **Hops 2-4 become public** - Leave your network, go through internet
3. **Hop with * * *** - This hop didn't respond (firewall blocked it)
4. **Final hops reach Google** - Google's network, recognized by IP

---

### Solution 9: Compare Classful vs CIDR Efficiency

**Complete Answer:**

| Employees | Method | Allocation | Total IPs | Waste | Efficiency |
|-----------|--------|-----------|-----------|-------|-----------|
| 200 | Classful | Class C | 256 | 56 | 78% |
| 200 | CIDR | /23 | 510 | 310 | 39% |
| 200 | CIDR | /24 | 256 | 56 | 78% |
| 100 | Classful | Class C | 256 | 156 | 39% |
| 100 | CIDR | /25 | 128 | 28 | 78% |
| 50 | Classful | Class C | 256 | 206 | 20% |
| 50 | CIDR | /26 | 64 | 14 | 78% |

**Why CIDR Wins:**

1. **Flexibility:** Choose exact size needed
   - Classful: Forced to take full Class C (256 IPs)
   - CIDR: Take exactly /26 (64 IPs) for 50 people

2. **Scalability:** Easy to grow
   - Need more IPs? Just adjust CIDR prefix
   - No need to change whole network

3. **Efficient Routing:** Fewer routes needed
   - Classful: Each network = one route
   - CIDR: Can group networks (route aggregation)
   - Example: Can route 200.0.0.0/22 instead of 200.0.0.0/24, 200.0.1.0/24, 200.0.2.0/24

**Real Example - Route Aggregation:**

Classful (3 networks):
```
Route 192.168.0.0/24 → Gateway A
Route 192.168.1.0/24 → Gateway A
Route 192.168.2.0/24 → Gateway A
Route 192.168.3.0/24 → Gateway A
```

CIDR aggregation:
```
Route 192.168.0.0/22 → Gateway A
(covers all 4 above networks!)
```

**Questions Answered:**

1. When is CIDR more efficient?
   - When you need specific sizes (not full Class)
   - When you have mixed requirements (some /25, some /26)
   - Always for modern networks

2. Can you subnet a classful block?
   - Yes! Classful blocks can be broken into CIDR subnets
   - But CIDR is more flexible

3. Why do routers prefer CIDR?
   - Fewer routes to maintain
   - Route aggregation (summarization)
   - More efficient routing protocols
   - Better scalability

---

### Solution 10: Create Multi-Level Routing Design

**Complete Answer:**

**Headquarters Routing Table:**

| Destination | Gateway | Metric | Purpose |
|-------------|---------|--------|---------|
| 10.0.0.0/16 | On-link | 10 | Local network (HQ) |
| 10.1.0.0/16 | VPN-Gateway-A | 20 | To Branch A via VPN |
| 10.2.0.0/16 | VPN-Gateway-B | 20 | To Branch B via VPN |
| 0.0.0.0/0 | ISP-Gateway | 5 | To Internet |

**Branch A Routing Table:**

| Destination | Gateway | Metric | Purpose |
|-------------|---------|--------|---------|
| 10.1.0.0/16 | On-link | 10 | Local network (Branch A) |
| 10.0.0.0/16 | VPN-Gateway-HQ | 15 | To HQ via VPN |
| 10.2.0.0/16 | VPN-Gateway-HQ | 25 | To Branch B (through HQ) |
| 0.0.0.0/0 | ISP-Gateway | 5 | To Internet |

**Branch B Routing Table:**

| Destination | Gateway | Metric | Purpose |
|-------------|---------|--------|---------|
| 10.2.0.0/16 | On-link | 10 | Local network (Branch B) |
| 10.0.0.0/16 | VPN-Gateway-HQ | 15 | To HQ via VPN |
| 10.1.0.0/16 | VPN-Gateway-HQ | 25 | To Branch A (through HQ) |
| 0.0.0.0/0 | ISP-Gateway | 5 | To Internet |

**Explanation:**

**Hub-and-Spoke Model:**
- HQ is the "hub" - central point
- Branches are the "spokes"
- All inter-office traffic goes through HQ

**Traffic Examples:**

1. Branch A → Branch B:
   ```
   Branch A (10.1.0.0) 
   → VPN to HQ (10.0.0.0) 
   → VPN to Branch B (10.2.0.0)
   ```
   Uses metric 25 route (lowest cost path)

2. Branch A → Internet:
   ```
   Branch A → ISP Gateway (internet connection)
   Uses metric 5 route (direct internet)
   ```

3. HQ ↔ Branch A:
   ```
   Direct VPN connection
   Uses metric 20 route (dedicated VPN tunnel)
   ```

**How to Create These Routes:**

```powershell
# On HQ Router:
route add 10.1.0.0 MASK 255.255.0.0 VPN-Gateway-A
route add 10.2.0.0 MASK 255.255.0.0 VPN-Gateway-B

# On Branch A Router:
route add 10.0.0.0 MASK 255.255.0.0 VPN-Gateway-HQ
route add 10.2.0.0 MASK 255.255.0.0 VPN-Gateway-HQ

# On Branch B Router:
route add 10.0.0.0 MASK 255.255.0.0 VPN-Gateway-HQ
route add 10.1.0.0 MASK 255.255.0.0 VPN-Gateway-HQ
```

**Or using PowerShell (persistent):**
```powershell
route add -p 10.1.0.0 MASK 255.255.0.0 VPN-Gateway-A
```

**Advantages of This Design:**
- ✓ All branches can communicate
- ✓ HQ controls security
- ✓ Easy to add new branches
- ✓ Clear routing hierarchy
- ✓ Internet connection at HQ (saves bandwidth)

**Disadvantages:**
- ✗ Slow for Branch A ↔ Branch B (goes through HQ)
- ✗ HQ is single point of failure
- ✗ More traffic on HQ connection

**Alternative: Mesh Network:**
If Branch A ↔ Branch B need fast direct connection:
```
Branch A ↔ Branch B (direct VPN)
Branch A ↔ HQ (direct VPN)
Branch B ↔ HQ (direct VPN)
```
More routes but faster inter-branch communication.

---

## Challenge Question Answers

**1. Why is CIDR better than classful addressing?**

Answer:
- Classful: Fixed sizes (256, 65536, 16M IPs) - often wasteful
- CIDR: Flexible sizes (any /8 to /32)
- Classful routing needs many entries; CIDR allows aggregation
- Modern internet runs on CIDR, classful is obsolete

**2. How does longest prefix match work? Give an example.**

Answer:
Longest prefix match selects the most specific (longest) matching route.

Example:
```
Routes:
  0.0.0.0/0 → Default Gateway
  10.0.0.0/8 → Corporate VPN
  10.1.0.0/16 → Branch Office VPN
  10.1.5.0/24 → Department LAN

Destination: 10.1.5.50

Matching routes:
  0.0.0.0/0 ✓ matches (everything matches /0)
  10.0.0.0/8 ✓ matches (10.x.x.x matches)
  10.1.0.0/16 ✓ matches (10.1.x.x matches)
  10.1.5.0/24 ✓ matches (10.1.5.x matches)

Winner: 10.1.5.0/24 (longest = most specific)
```

**3. What would happen if you deleted 0.0.0.0/0?**

Answer:
- No internet connectivity
- Any traffic not matching a specific route is dropped
- Only works for local networks and specific routes
- Will see "Destination unreachable" for internet IPs
- Users can't reach any external network

**4. Why use /30 for router-to-router links?**

Answer:
- /30 = 4 IPs, 2 usable
- Router A uses one IP
- Router B uses one IP
- Wastes no IPs (efficient)
- Smallest practical subnet for point-to-point
- Default in many routing protocols

Example:
```
10.0.1.0/30
  10.0.1.0 = Network address
  10.0.1.1 = Router A
  10.0.1.2 = Router B
  10.0.1.3 = Broadcast
```

**5. How do you design subnets for growth?**

Answer:
- Calculate current needs + 20-30% buffer
- Choose CIDR that doubles your need
- Document the design
- Plan for future departments/services
- Leave room for consolidation

Example:
```
Need 40 IPs today:
  /25 = 126 IPs (3x buffer - good growth room)

Need 150 IPs:
  /24 = 254 IPs (1.6x buffer - moderate growth)

Need 1000+ IPs:
  /21 = 2046 IPs (good for major expansion)
```

---

## Summary

**Key Takeaways:**

1. **CIDR notation** (/XX) is better than classful for modern networks
2. **Subnetting** divides networks into smaller pieces efficiently
3. **Routing tables** determine where packets go
4. **Longest prefix match** selects most specific route
5. **Route aggregation** reduces routing complexity
6. **Static routes** give you control; dynamic routes scale
7. **/30 for routers**, /24 for small networks, larger for growth
8. **Private IPs** stay inside your network; public routable
9. **Metrics** determine preferred path (lower = better)
10. **Hub-and-spoke** works for branch offices; mesh for peers

