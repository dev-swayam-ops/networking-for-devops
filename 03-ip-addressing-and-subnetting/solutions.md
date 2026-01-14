# Solutions: IP Addressing and Subnetting

Complete solutions to all exercises with detailed explanations.

## Easy Exercises

### Exercise 1 Solution: Identify IP Address Components
**Answer:**
1. Octet 1: **192**
2. Octet 2: **168**
3. Octet 3: **1**
4. Octet 4: **100**

**Explanation:**
- Every IP address has 4 octets separated by dots
- Each octet is 8 bits (0-255)
- Full notation: 192.168.1.100 = Network.Network.Network.Host (this is typical for /24)

**Key Concept:**
The IP address 192.168.1.100 in binary is:
- 192 = 11000000
- 168 = 10101000
- 1 = 00000001
- 100 = 01100100
- **Full binary:** 11000000.10101000.00000001.01100100

---

### Exercise 2 Solution: Understand Subnet Masks

**Answers:**

| Mask | Network Bits | Host Bits | Binary |
|------|-------------|-----------|--------|
| 255.255.255.0 | 24 | 8 | 11111111.11111111.11111111.00000000 |
| 255.255.255.128 | 25 | 7 | 11111111.11111111.11111111.10000000 |
| 255.255.255.192 | 26 | 6 | 11111111.11111111.11111111.11000000 |
| 255.255.255.240 | 28 | 4 | 11111111.11111111.11111111.11110000 |
| 255.255.0.0 | 16 | 16 | 11111111.11111111.00000000.00000000 |

**Explanation:**

**Subnet Mask Rule:**
- 255 in binary = 11111111 (8 network bits)
- 0 in binary = 00000000 (8 host bits)
- Other numbers = mix of 1s and 0s

**Breaking down 255.255.255.192:**
- First 255: 11111111 = 8 network bits
- Second 255: 11111111 = 8 network bits  
- Third 255: 11111111 = 8 network bits
- Fourth 192: 11000000 = 2 network bits + 6 host bits
- **Total: 24 + 2 = 26 network bits (so /26)**

**Why this matters:**
- More 1s = more networks, fewer hosts per network
- More 0s = fewer networks, more hosts per network
- 255.255.255.0 (/24) = 1 network, 254 hosts
- 255.255.255.192 (/26) = 4 networks, 62 hosts each

---

### Exercise 3 Solution: Convert Subnet Mask to CIDR

**Answers:**

| Subnet Mask | Binary | Count 1s | CIDR |
|-------------|--------|----------|------|
| 255.255.255.0 | 11111111.11111111.11111111.00000000 | 24 | /24 |
| 255.255.255.128 | 11111111.11111111.11111111.10000000 | 25 | /25 |
| 255.255.255.192 | 11111111.11111111.11111111.11000000 | 26 | /26 |
| 255.255.0.0 | 11111111.11111111.00000000.00000000 | 16 | /16 |
| 255.0.0.0 | 11111111.00000000.00000000.00000000 | 8 | /8 |

**Explanation:**

**CIDR Notation Rule:**
The number = count of consecutive 1-bits in the subnet mask

**Quick Reference:**
- First octet always 255 = 8 bits (so at least /8)
- Second octet: 255 = +8 more, 0 = skip it
- Third octet: 255 = +8 more, 0 = skip it
- Fourth octet: 255 = +8 more, 128 = +1, 192 = +2, 224 = +3, 240 = +4, etc.

**Example: 255.255.255.192**
- 255 = /8
- 255 = /8 (/16 total)
- 255 = /8 (/24 total)
- 192 = 11000000 = 2 ones = /2 (/26 total)
- **Final answer: /26**

---

### Exercise 4 Solution: Identify IP Address Class

**Answers:**
1. 10.0.0.1 = **Class A** (private)
2. 172.16.0.1 = **Class B** (private)
3. 192.168.1.1 = **Class C** (private)
4. 224.0.0.1 = **Class D** (multicast)
5. 127.0.0.1 = **Class A** (loopback/special)

**Classification Rules:**

| Class | First Octet | Range | Default Mask | Purpose |
|-------|------------|-------|--------------|---------|
| A | 1-126 | 1.x.x.x to 126.x.x.x | /8 | Large networks |
| B | 128-191 | 128.x.x.x to 191.x.x.x | /16 | Medium networks |
| C | 192-223 | 192.x.x.x to 223.x.x.x | /24 | Small networks |
| D | 224-239 | 224.x.x.x to 239.x.x.x | N/A | Multicast |
| E | 240-255 | 240.x.x.x to 255.x.x.x | N/A | Reserved |

**Private IP Ranges:**
- **10.0.0.0 to 10.255.255.255** (10.0.0.0/8) - Class A private
- **172.16.0.0 to 172.31.255.255** (172.16.0.0/12) - Class B private
- **192.168.0.0 to 192.168.255.255** (192.168.0.0/16) - Class C private

**Special Addresses:**
- 127.0.0.1 = Loopback (always refers to your own machine)
- 224-239 = Multicast (one-to-many communication)

---

### Exercise 5 Solution: Calculate Network Address

**Answers:**

| IP Address | Mask | Network Address |
|-----------|------|-----------------|
| 192.168.1.100 | 255.255.255.0 | 192.168.1.0 |
| 10.0.5.50 | 255.255.255.0 | 10.0.5.0 |
| 172.16.20.200 | 255.255.0.0 | 172.16.0.0 |

**Explanation:**

**Network Address Calculation Rule:**
Apply a binary AND operation: IP address AND subnet mask

**Example 1: 192.168.1.100 with /24 (255.255.255.0)**

```
IP:     192.168.1.100 = 11000000.10101000.00000001.01100100
Mask:   255.255.255.0 = 11111111.11111111.11111111.00000000
AND:    Result        = 11000000.10101000.00000001.00000000
Decimal:              = 192.168.1.0 ✓
```

The last octet becomes 0 because we AND 01100100 with 00000000 = 00000000

**Example 2: 172.16.20.200 with /16 (255.255.0.0)**

```
IP:     172.16.20.200 = 10101100.00010000.00010100.11001000
Mask:   255.255.0.0   = 11111111.11111111.00000000.00000000
AND:    Result        = 10101100.00010000.00000000.00000000
Decimal:              = 172.16.0.0 ✓
```

Last two octets become 0 because all host bits are masked out

**Key Concept:**
Network address = all host bits set to 0
This is the first usable address in the range (though not usable for devices)

---

## Medium Exercises

### Exercise 6 Solution: Calculate Broadcast Address

**Answers:**

| Network | CIDR | Broadcast Address |
|---------|------|-------------------|
| 192.168.1.0 | /24 | 192.168.1.255 |
| 192.168.1.0 | /25 | 192.168.1.127 |
| 192.168.1.128 | /25 | 192.168.1.255 |
| 10.0.0.0 | /16 | 10.0.255.255 |

**Explanation:**

**Broadcast Address Calculation Rule:**
Set all host bits to 1 (OR with the inverse of subnet mask)

**Example 1: 192.168.1.0/24**

```
Network:   192.168.1.0   = 11000000.10101000.00000001.00000000
Host bits: 8 bits (mask = 255.255.255.0)
Broadcast: Set to 1s     = 11000000.10101000.00000001.11111111
Decimal:                 = 192.168.1.255 ✓
```

**Example 2: 192.168.1.128/25 (Subnet 2 of /24)**

```
Network:   192.168.1.128 = 11000000.10101000.00000001.10000000
Host bits: 7 bits (mask = 255.255.255.128)
Broadcast: Set to 1s     = 11000000.10101000.00000001.10111111
Decimal:                 = 192.168.1.191 ✗ WRONG
```

Wait, let me recalculate:
- /25 means 25 network bits, 7 host bits
- Broadcast = Network + (2^7 - 1) = 128 + 127 = 255 ✓

**Formula:** Broadcast = Network + Hosts - 1

For 192.168.1.128/25:
- Hosts = 2^7 = 128
- Broadcast = 128 + 128 - 1 = 255 ✓

**Example 3: 10.0.0.0/16**

```
Network:   10.0.0.0     = 00001010.00000000.00000000.00000000
Host bits: 16 bits
Broadcast: Set to 1s    = 00001010.00000000.11111111.11111111
Decimal:                = 10.0.255.255 ✓
```

**Why Broadcast?**
Used to send message to all devices on network
- Not assigned to any device
- Used for DHCP, ARP, and other protocols

---

### Exercise 7 Solution: Determine Valid Host Range

**Answers:**

| Network | First Host | Last Host | Total Hosts |
|---------|-----------|-----------|-------------|
| 192.168.1.0/24 | 192.168.1.1 | 192.168.1.254 | 254 |
| 192.168.1.0/25 | 192.168.1.1 | 192.168.1.126 | 126 |
| 192.168.1.128/25 | 192.168.1.129 | 192.168.1.254 | 126 |
| 10.0.0.0/16 | 10.0.0.1 | 10.0.255.254 | 65534 |

**Explanation:**

**Host Range Formula:**
- First Host = Network Address + 1
- Last Host = Broadcast Address - 1

**Example 1: 192.168.1.0/24**

```
Network:    192.168.1.0
First Host: 192.168.1.1 (network + 1)
Last Host:  192.168.1.254 (255 - 1)
Broadcast:  192.168.1.255
```

Total hosts = Last - First + 1 = 254 - 1 + 1 = 254 ✓

**Example 2: 192.168.1.0/25 (First half of /24)**

```
Network:    192.168.1.0
Broadcast:  192.168.1.127 (half of 255)
First Host: 192.168.1.1
Last Host:  192.168.1.126
```

Total hosts = 126 ✓

**Example 3: 192.168.1.128/25 (Second half of /24)**

```
Network:    192.168.1.128
Broadcast:  192.168.1.255
First Host: 192.168.1.129
Last Host:  192.168.1.254
```

Total hosts = 254 - 129 + 1 = 126 ✓

**Summary:**
- Network address reserved (can't assign to device)
- Broadcast address reserved (can't assign to device)
- Everything in between = usable for devices

---

### Exercise 8 Solution: Calculate Subnet Divisions

**Basic Answer: Divide 192.168.0.0/24 into 4 subnets**

```
Original: 192.168.0.0/24 = 256 addresses, 254 usable
Need 4 subnets: 2^2 = 4, so add 2 bits = /26

Subnet 1: 192.168.0.0/26 = 192.168.0.0 to 192.168.0.63
  Network: 192.168.0.0, First Host: 192.168.0.1, Last: 192.168.0.62, Broadcast: 192.168.0.63

Subnet 2: 192.168.0.64/26 = 192.168.0.64 to 192.168.0.127
  Network: 192.168.0.64, First Host: 192.168.0.65, Last: 192.168.0.126, Broadcast: 192.168.0.127

Subnet 3: 192.168.0.128/26 = 192.168.0.128 to 192.168.0.191
  Network: 192.168.0.128, First Host: 192.168.0.129, Last: 192.168.0.190, Broadcast: 192.168.0.191

Subnet 4: 192.168.0.192/26 = 192.168.0.192 to 192.168.0.255
  Network: 192.168.0.192, First Host: 192.168.0.193, Last: 192.168.0.254, Broadcast: 192.168.0.255
```

**Calculation Steps:**
1. Original = /24 (8 host bits)
2. Need 4 subnets = 2^2 = need 2 new bits
3. New CIDR = 24 + 2 = /26
4. Subnet size = 2^(32-26) = 2^6 = 64 addresses each
5. Increment = 64 (0, 64, 128, 192)

---

**Challenge Answer: Divide 10.0.0.0/16 into 8 subnets**

```
Original: 10.0.0.0/16 = 65,536 addresses
Need 8 subnets: 2^3 = 8, so add 3 bits = /19

Each subnet: 2^(32-19) = 2^13 = 8,192 addresses
Increment: 256 * 32 = 8,192 (or 0x2000)

Subnet 1: 10.0.0.0/19     (10.0.0.0 - 10.0.31.255)
Subnet 2: 10.0.32.0/19    (10.0.32.0 - 10.0.63.255)
Subnet 3: 10.0.64.0/19    (10.0.64.0 - 10.0.95.255)
Subnet 4: 10.0.96.0/19    (10.0.96.0 - 10.0.127.255)
Subnet 5: 10.0.128.0/19   (10.0.128.0 - 10.0.159.255)
Subnet 6: 10.0.160.0/19   (10.0.160.0 - 10.0.191.255)
Subnet 7: 10.0.192.0/19   (10.0.192.0 - 10.0.223.255)
Subnet 8: 10.0.224.0/19   (10.0.224.0 - 10.0.255.255)
```

**Calculation:**
- 32 * 256 = 8192 (addresses per subnet)
- Each third octet jumps by 32 (0, 32, 64, 96, 128, 160, 192, 224)

---

### Exercise 9 Solution: Plan an IP Addressing Scheme

**Scenario Requirements:**
- 50 employees (growing to 100)
- 5 servers
- 10 printers
- 5 management devices
- **Total:** ~120 devices (planning for growth)

**Available Space:** 192.168.0.0/22 = 4 networks of /24 = 4,096 addresses

**Addressing Plan:**

```
192.168.0.0/22 (1024 addresses total)
├── 192.168.0.0/24 - Employees Network
│   └── Capacity: 254 hosts ✓ (covers 50, grows to 100+)
│   └── Network: 192.168.0.0
│   └── Gateway: 192.168.0.1
│   └── Usable: 192.168.0.2 - 192.168.0.254
│   └── Broadcast: 192.168.0.255
│
├── 192.168.1.0/25 - Servers Network
│   └── Capacity: 62 hosts ✓ (only needs 5)
│   └── Network: 192.168.1.0
│   └── Gateway: 192.168.1.1
│   └── Usable: 192.168.1.2 - 192.168.1.62
│   └── Broadcast: 192.168.1.63
│
├── 192.168.1.128/26 - Printers Network
│   └── Capacity: 62 hosts ✓ (only needs 10)
│   └── Network: 192.168.1.128
│   └── Gateway: 192.168.1.129
│   └── Usable: 192.168.1.130 - 192.168.1.190
│   └── Broadcast: 192.168.1.191
│
└── 192.168.2.0/27 - Management Network
    └── Capacity: 30 hosts ✓ (only needs 5)
    └── Network: 192.168.2.0
    └── Gateway: 192.168.2.1
    └── Usable: 192.168.2.2 - 192.168.2.30
    └── Broadcast: 192.168.2.31
```

**Verification:**
```
Used addresses:     254 + 62 + 62 + 30 = 408 hosts
Total available:    1024 addresses
Utilization:        40%
Unused:             616 addresses (for future growth)
```

**Answer to "Why this plan?"**
✓ Separates concerns (employees, servers, printers, management)
✓ Each segment properly sized with growth room
✓ Follows best practice of segregating server traffic
✓ Management isolated for security
✓ Efficient use of address space

---

**Challenge: Growth to 200 Employees**

Original /24 has 254 hosts = enough for 200 employees!

But if needed even more or different topology:
```
Option 1: Use 192.168.0.0/23 for employees
  Capacity: 510 hosts (covers 200+ easily)
  Leaves 192.168.2.0/22 for other networks

Option 2: Multiple employee subnets
  192.168.0.0/25 for Sales (126 hosts)
  192.168.0.128/25 for Engineering (126 hosts)
  192.168.1.0/25 for HR + Mixed (126 hosts)
```

---

### Exercise 10 Solution: Convert Between Binary and Decimal

**Decimal to Binary Answers:**

| Decimal | Binary | Powers of 2 |
|---------|--------|------------|
| 192 | 11000000 | 128 + 64 |
| 168 | 10101000 | 128 + 32 + 8 |
| 255 | 11111111 | 128 + 64 + 32 + 16 + 8 + 4 + 2 + 1 |
| 128 | 10000000 | 128 |
| 64 | 01000000 | 64 |

**Explanation of Conversion:**

Each bit position represents a power of 2:
```
Position: 7    6    5    4    3    2    1    0
Value:    128  64   32   16   8    4    2    1

Example: 192
= 128 + 64 + 0 + 0 + 0 + 0 + 0 + 0
= 11000000 ✓

Example: 168
= 128 + 0 + 32 + 0 + 8 + 0 + 0 + 0
= 10101000 ✓
```

**Binary to Decimal Answers:**

| Binary | Decimal | Powers of 2 |
|--------|---------|------------|
| 11000000 | 192 | 128 + 64 |
| 10101000 | 168 | 128 + 32 + 8 |
| 11111111 | 255 | All bits |
| 10000000 | 128 | 128 only |
| 01000000 | 64 | 64 only |

**Common Subnet Mask Values in Binary:**

| Decimal | Binary | CIDR |
|---------|--------|------|
| 255 | 11111111 | /8 per octet |
| 192 | 11000000 | /2 in octet (/26 with three 255s) |
| 224 | 11100000 | /3 in octet (/27 with three 255s) |
| 240 | 11110000 | /4 in octet (/28 with three 255s) |
| 248 | 11111000 | /5 in octet (/29 with three 255s) |
| 252 | 11111100 | /6 in octet (/30 with three 255s) |

**Conversion Technique:**
Always start with 128:
1. Does the number include 128? Put 1, else 0
2. Remaining for 64? Put 1, else 0
3. Continue for 32, 16, 8, 4, 2, 1

Example: Convert 240
- 240 ≥ 128? Yes (240 - 128 = 112) → 1
- 112 ≥ 64? Yes (112 - 64 = 48) → 1
- 48 ≥ 32? Yes (48 - 32 = 16) → 1
- 16 ≥ 16? Yes (16 - 16 = 0) → 1
- Remaining 0 < 8? → 0,0,0,0
- **Result: 11110000** ✓

---

## Challenge Questions Answers

**1. Why use private IPs instead of public everywhere?**

**Answer:**
- Public IP addresses are limited (only 4.3 billion IPv4 addresses)
- Private IPs can be reused in different organizations
- Private networks are more secure (not directly on internet)
- Reduces IPv4 address waste
- NAT/PAT translates private to public at borders

---

**2. What's the advantage of CIDR over class-based?**

**Answer:**
- **Old way (class-based):**
  - Class C = /24 always (256 addresses)
  - Wastes addresses if you need fewer
  - No flexibility between sizes

- **CIDR (Classless):**
  - /25 = 128 addresses (if you need 100)
  - /26 = 64 addresses (if you need 50)
  - /22 = 1024 addresses (if you need 500)
  - Flexible and efficient

**Result:** Better address utilization, less waste

---

**3. How many usable hosts are in a /30?**

**Answer:**
```
/30 = 4 total addresses
Minus network address = 3 remaining
Minus broadcast address = 2 usable hosts

4 - 2 = 2 usable hosts

Example: 192.168.1.0/30
Network: 192.168.1.0 (reserved)
Host 1: 192.168.1.1
Host 2: 192.168.1.2
Broadcast: 192.168.1.3 (reserved)
```

**Use case:** Point-to-point links (router-to-router)

---

**4. Can two subnets have the same network address?**

**Answer:** NO

**Why?**
- Network address uniquely identifies a subnet
- Two different subnets MUST have different network addresses
- If they overlap, routing won't work (ambiguous)

**Example of wrong plan:**
```
❌ Subnet 1: 192.168.1.0/25 (0-127)
❌ Subnet 2: 192.168.1.0/26 (0-63)
Same network address! Not allowed.
```

**Correct plan:**
```
✓ Subnet 1: 192.168.1.0/25 (0-127)
✓ Subnet 2: 192.168.1.128/25 (128-255)
Different network addresses. Correct!
```

---

**5. What's the broadcast address of 192.168.1.0/26?**

**Answer:**
```
/26 = last 6 bits are host bits
Subnet size = 2^6 = 64 addresses
Network starts at: 192.168.1.0
Broadcast = Network + Size - 1
Broadcast = 0 + 64 - 1 = 63

Broadcast address: 192.168.1.63 ✓

Full range:
Network:    192.168.1.0 (reserved)
First host: 192.168.1.1
Last host:  192.168.1.62
Broadcast:  192.168.1.63 (reserved)
```

---

## Hands-On Challenge Lab Solution

**Network Design Example:**

```
Corporate Network Design: 10.0.0.0/16

Department Layout:
┌─────────────────────────────────────────────┐
│          10.0.0.0/16 (65,536 hosts)         │
├─────────────────────────────────────────────┤
│ Sales Department      │ 10.0.1.0/24 (254)  │
│ Engineering Department│ 10.0.2.0/24 (254)  │
│ HR Department         │ 10.0.3.0/24 (254)  │
│ Server Farm           │ 10.0.10.0/24 (254) │
│ Guest Network         │ 10.0.20.0/25 (126) │
│ Management Network    │ 10.0.20.128/26 (62)│
│ Reserved/Future       │ Rest (65,536)      │
└─────────────────────────────────────────────┘
```

**Subnet Allocation Table:**

| Department | Network | Mask | CIDR | Hosts | Gateway | Broadcast |
|-----------|---------|------|------|-------|---------|-----------|
| Sales | 10.0.1.0 | 255.255.255.0 | /24 | 254 | 10.0.1.1 | 10.0.1.255 |
| Engineering | 10.0.2.0 | 255.255.255.0 | /24 | 254 | 10.0.2.1 | 10.0.2.255 |
| HR | 10.0.3.0 | 255.255.255.0 | /24 | 254 | 10.0.3.1 | 10.0.3.255 |
| Servers | 10.0.10.0 | 255.255.255.0 | /24 | 254 | 10.0.10.1 | 10.0.10.255 |
| Guest | 10.0.20.0 | 255.255.255.128 | /25 | 126 | 10.0.20.1 | 10.0.20.127 |
| Management | 10.0.20.128 | 255.255.255.192 | /26 | 62 | 10.0.20.129 | 10.0.20.191 |

**Host Range Examples:**

**Sales (10.0.1.0/24):**
- Gateway: 10.0.1.1
- First workstation: 10.0.1.2
- Last workstation: 10.0.1.254
- Broadcast: 10.0.1.255
- Capacity: 252 usable

**Guest (10.0.20.0/25):**
- Gateway: 10.0.20.1
- First device: 10.0.20.2
- Last device: 10.0.20.126
- Broadcast: 10.0.20.127
- Capacity: 126 usable

**Validation Checklist:**

```
✓ No overlap between subnets (all have unique network addresses)
✓ Each department properly sized (room for growth)
✓ Servers isolated (separate security zone)
✓ Guest network segregated (for security/compliance)
✓ Management network small but separate (for admin access)
✓ Total used: 254+254+254+254+126+62 = 1,204 addresses
✓ Utilization: 1,204 / 65,536 = 1.8% (lots of growth room)
✓ All subnets within 10.0.0.0/16 boundary
✓ CIDR blocks are properly aligned and non-overlapping

Network Diagram:
┌──────────────────────┐
│   Internet/WAN       │
└──────────┬───────────┘
           │
       ┌───┴────┐
       │ Router │ (Gateway)
       └───┬────┘
           │
      ┌────┴──────────────────────────┐
      │        Core Switch            │
      └────┬──────────┬──────────┬────┘
           │          │          │
       ┌───┴──┐  ┌───┴──┐  ┌───┴──────┐
       │Sales │  │Eng   │  │HR   │Svr │
       │10.0.1│  │10.0.2│  │10.0.3    │
       └──────┘  └──────┘  └──────────┘
           │         │         │
       ┌───┴─────────┴─────────┴──┐
       │  Guest (10.0.20.0/25)    │
       │  Management (10.0.20.128)│
       └─────────────────────────┘
```

