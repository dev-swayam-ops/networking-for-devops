# Exercises: IP Addressing and Subnetting

Complete these exercises to master IP addressing and subnet calculations. Progress from Easy to Medium.

## Easy Exercises

### Exercise 1: Identify IP Address Components
**Objective:** Understand the structure of an IP address

**Task:** Break down this IP address:

`192.168.1.100`

Answer these questions:
1. What is Octet 1? (should be 192)
2. What is Octet 2? (should be 168)
3. What is Octet 3? (should be 1)
4. What is Octet 4? (should be 100)

**Success Criteria:**
- Can identify all 4 octets
- Understand that each octet represents 8 bits
- Know that each octet can be 0-255

---

### Exercise 2: Understand Subnet Masks
**Objective:** Learn how subnet masks work

**Task:** Given these subnet masks, identify which represent networks and which represent hosts:

```
255.255.255.0      = ? network bits, ? host bits
255.255.255.128    = ? network bits, ? host bits
255.255.255.192    = ? network bits, ? host bits
255.255.255.240    = ? network bits, ? host bits
255.255.0.0        = ? network bits, ? host bits
```

**Rule:** Count the "1" bits for network, "0" bits for host

**Answer Guide:**
- 255 = 8 ones = 8 network bits
- 128 = 1 one (10000000) = 1 network bit in that octet
- 192 = 2 ones (11000000) = 2 network bits
- 240 = 4 ones (11110000) = 4 network bits
- 0 = 0 ones = 0 network bits

**Success Criteria:**
- Can count network bits in any subnet mask
- Understand inverse relationship (more network = fewer host)
- Know 255 always means /8 network bits

---

### Exercise 3: Convert Subnet Mask to CIDR
**Objective:** Practice CIDR notation

**Task:** Convert these subnet masks to CIDR notation:

| Subnet Mask | CIDR |
|-------------|------|
| 255.255.255.0 | ? |
| 255.255.255.128 | ? |
| 255.255.255.192 | ? |
| 255.255.0.0 | ? |
| 255.0.0.0 | ? |

**Rule:** Count all 1 bits = CIDR number

**Answer Guide:**
- 255.255.255.0 = 8+8+8+0 = /24
- 255.255.255.128 = 8+8+8+1 = /25
- 255.255.255.192 = 8+8+8+2 = /26
- 255.255.0.0 = 8+8+0+0 = /16
- 255.0.0.0 = 8+0+0+0 = /8

**Success Criteria:**
- Convert any subnet mask to CIDR
- Understand /24 is standard (/16 is large, /28 is small)
- Know higher CIDR = more networks but fewer hosts

---

### Exercise 4: Identify IP Address Class
**Objective:** Recognize IP address classes

**Task:** Identify the class of these IP addresses:

1. 10.0.0.1 = Class ?
2. 172.16.0.1 = Class ?
3. 192.168.1.1 = Class ?
4. 224.0.0.1 = Class ?
5. 127.0.0.1 = Class ?

**Rule:** Look at first octet

**Answer Guide:**
- 1-126: Class A
- 128-191: Class B
- 192-223: Class C
- 224-239: Class D (multicast)
- 240-255: Class E (reserved)

**Success Criteria:**
- Instantly recognize address class from first octet
- Understand purpose of each class
- Know private ranges

---

### Exercise 5: Calculate Network Address
**Objective:** Find network address from any IP

**Task:** Find the network address for:

1. 192.168.1.100 with mask 255.255.255.0 = ?
2. 10.0.5.50 with mask 255.255.255.0 = ?
3. 172.16.20.200 with mask 255.255.0.0 = ?

**Rule:** Set all host bits to 0

**Answer Guide:**
1. 192.168.1.0 (last octet becomes 0)
2. 10.0.5.0 (last octet becomes 0)
3. 172.16.0.0 (last 2 octets become 0)

**Success Criteria:**
- Can find network address with /24 mask
- Can find network address with /16 mask
- Understand network = lowest address in range

---

## Medium Exercises

### Exercise 6: Calculate Broadcast Address
**Objective:** Find broadcast address from any IP

**Task:** Find the broadcast address for:

1. 192.168.1.0/24 = ?
2. 192.168.1.0/25 = ?
3. 192.168.1.128/25 = ?
4. 10.0.0.0/16 = ?

**Rule:** Set all host bits to 1

**Answer Guide:**
1. 192.168.1.255 (last octet = 255)
2. 192.168.1.127 (last octet = 127)
3. 192.168.1.255 (last octet = 255)
4. 10.0.255.255 (last 2 octets = 255)

**Success Criteria:**
- Can find broadcast address
- Understand broadcast = highest address in range
- Know difference between /24 and /25 boundaries

---

### Exercise 7: Determine Valid Host Range
**Objective:** Find first and last usable host addresses

**Task:** Find valid host range (first host to last host):

| Network | First Host | Last Host |
|---------|-----------|-----------|
| 192.168.1.0/24 | ? | ? |
| 192.168.1.0/25 | ? | ? |
| 192.168.1.128/25 | ? | ? |
| 10.0.0.0/16 | ? | ? |

**Rule:** 
- First host = Network + 1
- Last host = Broadcast - 1

**Answer Guide:**
1. 192.168.1.1 to 192.168.1.254
2. 192.168.1.1 to 192.168.1.126
3. 192.168.1.129 to 192.168.1.254
4. 10.0.0.1 to 10.0.255.254

**Success Criteria:**
- Find valid host range from any network
- Understand why we subtract 1 and 2
- Apply to different CIDR notations

---

### Exercise 8: Calculate Subnet Divisions
**Objective:** Divide a large network into smaller subnets

**Task:** Divide 192.168.0.0/24 into 4 equal subnets

**Steps:**
1. Original network: 192.168.0.0/24 (256 addresses)
2. Need 4 subnets = need /26 (/24 + 2)
3. Each subnet gets 64 addresses

**Answer:**
```
Subnet 1: 192.168.0.0/26   (0-63)
Subnet 2: 192.168.0.64/26  (64-127)
Subnet 3: 192.168.0.128/26 (128-191)
Subnet 4: 192.168.0.192/26 (192-255)
```

**Challenge:** Divide 10.0.0.0/16 into 8 subnets

**Answer Guide:**
- /16 + 3 bits = /19 (2^3 = 8 subnets)
- Each subnet: 2^(32-19) = 2^13 = 8192 addresses
- First subnet: 10.0.0.0/19 (0 to 8191)
- Second subnet: 10.0.32.0/19 (8192 to 16383)

**Success Criteria:**
- Divide network correctly
- Calculate subnet addresses
- Understand subnet increment

---

### Exercise 9: Plan an IP Addressing Scheme
**Objective:** Design addressing for a small network

**Scenario:** Small office needs:
- 50 employees (with growth to 100)
- 5 servers
- 10 printers
- Management network (5 devices)

**Task:** Design an addressing scheme using 192.168.0.0/22 (1024 addresses)

**Solution Framework:**
```
Employees:    192.168.0.0/25   (126 hosts) ✓
Servers:      192.168.0.128/26 (62 hosts)  ✓
Printers:     192.168.0.192/27 (30 hosts)  ✓
Management:   192.168.1.0/27   (30 hosts)  ✓
```

**Task:** Verify this plan covers all needs

**Challenge:** What if employees need to grow to 200?

**Success Criteria:**
- Design practical addressing scheme
- Calculate requirements properly
- Plan for growth
- No wasted addresses

---

### Exercise 10: Convert Between Binary and Decimal
**Objective:** Understand binary representation of IPs

**Task:** Convert these to binary (one octet at a time):

1. 192 = ?
2. 168 = ?
3. 255 = ?
4. 128 = ?
5. 64 = ?

Then convert back:

6. 11000000 = ?
7. 10101000 = ?
8. 11111111 = ?
9. 10000000 = ?
10. 01000000 = ?

**Answer Guide:**
1. 192 = 11000000
2. 168 = 10101000
3. 255 = 11111111
4. 128 = 10000000
5. 64 = 01000000
6. 11000000 = 192
7. 10101000 = 168
8. 11111111 = 255
9. 10000000 = 128
10. 01000000 = 64

**Success Criteria:**
- Convert decimal ↔ binary reliably
- Understand powers of 2
- Apply to subnet mask conversion

---

## Challenge Questions

**Research and answer:**

1. Why do we use private IP addresses instead of public IPs everywhere?
2. What's the advantage of CIDR notation over class-based addressing?
3. How many usable host addresses are in a /30 network?
4. Can two subnets have the same network address? Why?
5. What's the broadcast address of 192.168.1.0/26?

---

## Hands-On Challenge Lab

**Create a complete subnetting plan:**

1. **Diagram a network** with these requirements:
   - 3 departments (Sales, Engineering, HR)
   - Central server farm
   - Guest network
   - Management network

2. **Design IP scheme** using 10.0.0.0/16:
   - Allocate subnets for each department
   - Size for growth
   - Separate management

3. **Document:**
   - Network diagram
   - Subnet allocation table
   - Host ranges for each network
   - Gateway addresses

4. **Validate:**
   - No overlap between subnets
   - Adequate addresses for each
   - Room for growth

