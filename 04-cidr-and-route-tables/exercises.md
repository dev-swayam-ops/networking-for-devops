# Exercises: CIDR and Route Tables

Complete these exercises to master IP addressing with CIDR and routing table management.

## Easy Exercises

### Exercise 1: Convert Subnet Masks to CIDR
**Objective:** Learn to convert classful notation to CIDR notation

Convert each subnet mask to CIDR:

| Subnet Mask | CIDR | Hosts |
|-------------|------|-------|
| 255.255.255.0 | ? | ? |
| 255.255.255.128 | ? | ? |
| 255.255.255.192 | ? | ? |
| 255.255.255.224 | ? | ? |
| 255.255.0.0 | ? | ? |
| 255.0.0.0 | ? | ? |

**Process:**
1. Convert subnet mask to binary
2. Count the number of 1s
3. That number is your CIDR prefix

**Example:** 255.255.255.0 = 11111111.11111111.11111111.00000000 = 24 ones = /24

**Success Criteria:**
- Correctly convert all 6 masks to CIDR
- Understand the pattern (each 255 = 8 ones)

---

### Exercise 2: Convert CIDR to Subnet Masks
**Objective:** Reverse the process - convert CIDR to traditional notation

Convert CIDR notation to subnet mask:

| CIDR | Subnet Mask | Hosts |
|------|-------------|-------|
| /24 | ? | ? |
| /16 | ? | ? |
| /8 | ? | ? |
| /25 | ? | ? |
| /30 | ? | ? |
| /32 | ? | ? |

**Process:**
1. Start with 32 bits all 0s
2. Change the first N bits (CIDR number) to 1s
3. Write as dotted decimal

**Example:** /24 = 11111111.11111111.11111111.00000000 = 255.255.255.0

**Success Criteria:**
- Correctly convert all 6 CIDR notations
- Understand /32 = single host, /0 = any host

---

### Exercise 3: Calculate Hosts in CIDR Blocks
**Objective:** Determine how many hosts fit in each CIDR block

For each CIDR block, calculate usable hosts:

| CIDR | Formula | Calculation | Usable Hosts |
|------|---------|-------------|--------------|
| /24 | 2^(32-24)-2 | 2^8-2 | ? |
| /25 | 2^(32-25)-2 | 2^7-2 | ? |
| /26 | 2^(32-26)-2 | 2^6-2 | ? |
| /27 | 2^(32-27)-2 | 2^5-2 | ? |
| /30 | 2^(32-30)-2 | 2^2-2 | ? |

**Process:**
1. Subtract CIDR from 32 (host bits)
2. Calculate 2 to that power
3. Subtract 2 (network and broadcast)

**Success Criteria:**
- All calculations correct
- Understand why subtract 2
- Know when /30 is used (router links)

---

### Exercise 4: View and Interpret Routing Tables
**Objective:** Understand your computer's routing table

Run this command:
```powershell
route print
```

**Task:**
- Find the default route (0.0.0.0/0)
- Write down the gateway IP
- Find your local network route
- Write down the CIDR notation

**Answer these:**
1. What's your default gateway IP?
2. What's your local network in CIDR? (e.g., 192.168.1.0/24)
3. How many routes in your table?

**Success Criteria:**
- Can identify default route
- Understand On-link vs gateway routes
- Know what each column means

---

### Exercise 5: Match Destination to Route
**Objective:** Determine which route is used for different destinations

Given this routing table:

```
Destination     Mask           Gateway        Metric
0.0.0.0         0.0.0.0        192.168.1.1    25
127.0.0.0       255.0.0.0      On-link        306
192.168.1.0     255.255.255.0  On-link        275
```

For each destination, which route matches?

| Destination | Route Used | Why |
|-------------|------------|-----|
| 192.168.1.50 | ? | ? |
| 8.8.8.8 | ? | ? |
| 127.0.0.1 | ? | ? |
| 10.0.0.5 | ? | ? |

**Success Criteria:**
- Understand longest prefix match
- Know default route is fallback
- Can explain each choice

---

## Medium Exercises

### Exercise 6: Design an IP Addressing Scheme Using CIDR
**Objective:** Create an efficient network design using CIDR

**Scenario:** You have 10.0.0.0/8 and need to design:
- Floor 1: 50 employees
- Floor 2: 30 employees  
- Floor 3: 20 employees
- Server network: 10 servers
- Printers: 5 printers

**Task:** Design subnets using CIDR

| Department | Employees | CIDR Block | Subnet Mask | Hosts |
|------------|-----------|-----------|-------------|-------|
| Floor 1 | 50 | 10.0.1.0/? | ? | ? |
| Floor 2 | 30 | 10.0.2.0/? | ? | ? |
| Floor 3 | 20 | 10.0.3.0/? | ? | ? |
| Servers | 10 | 10.0.4.0/? | ? | ? |
| Printers | 5 | 10.0.5.0/? | ? | ? |

**Requirements:**
- Each subnet must fit its devices
- Use smallest CIDR that fits (efficient)
- Leave room for 20% growth

**Formula:**
- /25 = 126 hosts (use for 50+ people)
- /26 = 62 hosts (use for 25-49 people)
- /27 = 30 hosts (use for 16-24 people)
- /28 = 14 hosts (use for 5-13 people)

**Success Criteria:**
- All subnets large enough
- Using efficient block sizes
- Can explain your choices

---

### Exercise 7: Add and Remove Static Routes
**Objective:** Manage routing table entries

**Task:** Add routes and verify them

```powershell
# Add these routes (admin access needed)
route add 172.16.0.0 MASK 255.255.0.0 192.168.1.1
route add 10.0.0.0 MASK 255.255.0.0 192.168.1.1

# View routes
route print

# Delete routes
route delete 172.16.0.0 MASK 255.255.0.0
route delete 10.0.0.0 MASK 255.255.0.0

# Verify deletion
route print
```

**Questions:**
1. Did the routes appear immediately?
2. Are they persistent after reboot?
3. What happens if you use same destination twice?

**Optional - Persistent Routes:**
```powershell
# Make route permanent
route add -p 172.16.0.0 MASK 255.255.0.0 192.168.1.1
```

**Success Criteria:**
- Successfully add routes
- Verify routes in table
- Cleanly remove routes
- Understand temporary vs persistent

---

### Exercise 8: Analyze Real Network Traffic with CIDR
**Objective:** See how CIDR applies to real network traffic

**Task:** Trace a route and analyze each hop:

```powershell
tracert google.com
```

**For each hop, determine:**

| Hop | IP | CIDR Block | Private/Public | Region |
|-----|----|----|---------|--------|
| 1 | ? | ? | ? | ? |
| 2 | ? | ? | ? | ? |
| 3 | ? | ? | ? | ? |

**How to figure it out:**
- Private ranges: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
- Everything else is public/internet

**Research:**
- Is hop 1 always private? Why?
- When does it become public?
- What's the gateway's role?

**Success Criteria:**
- Can categorize IPs as private/public
- Understand private IP ranges
- See real-world routing in action

---

### Exercise 9: Compare Classful vs CIDR Efficiency
**Objective:** Understand why CIDR is better than classful

**Scenario:** You need to allocate IPs for 200 employees

**Classful Approach (old way):**
- Class C gives 256 IPs (192.168.1.0)
- Only 200 needed, but must take all 256
- Waste: 56 IPs

**CIDR Approach (modern way):**
- /23 gives 510 IPs (192.168.0.0/23)
- 200 needed, gets 510
- Waste: 310 IPs (but more efficient routing)
- Or use /24 (256 IPs) for 200 users
- Waste: 56 IPs (same as classful, but with option to subnet further)

**Task:** Calculate efficiency:

| Employees | Classful Waste | CIDR Block | CIDR Waste | Savings |
|-----------|---|---------|---|---|
| 200 | 56 (Class C) | /23 | ? | ? |
| 100 | 156 (Class C) | /25 | ? | ? |
| 50 | 206 (Class C) | /26 | ? | ? |

**Questions:**
1. When is CIDR more efficient?
2. Can you subnet a classful block?
3. Why do routers prefer CIDR?

**Success Criteria:**
- Understand efficiency differences
- Know CIDR allows flexibility
- Appreciate modern routing

---

### Exercise 10: Create a Multi-Level Routing Design
**Objective:** Design realistic routing for a multi-site organization

**Scenario:** Company with 3 offices:
- Headquarters: 10.0.0.0/16 (400 people)
- Branch A: 10.1.0.0/16 (100 people)
- Branch B: 10.2.0.0/16 (50 people)
- Connected via VPN to HQ

**Task:** Create routing tables for each location

**Headquarters Routing Table:**

| Destination | Gateway | Purpose |
|-------------|---------|---------|
| 10.0.0.0/16 | On-link | Local network |
| 10.1.0.0/16 | ? | To Branch A |
| 10.2.0.0/16 | ? | To Branch B |
| 0.0.0.0/0 | ? | To Internet |

**Branch A Routing Table:**

| Destination | Gateway | Purpose |
|-------------|---------|---------|
| 10.1.0.0/16 | On-link | Local network |
| 10.0.0.0/16 | ? | To HQ |
| 10.2.0.0/16 | ? | To Branch B (via HQ) |
| 0.0.0.0/0 | ? | To Internet |

**Task:**
- Fill in each table
- Explain each route
- How would you create these routes?

**Success Criteria:**
- Complete routing tables
- Understand hub-and-spoke model
- Know why you need each route
- Can explain the traffic flow

---

## Challenge Questions

**Test your understanding:**

1. Why is CIDR better than classful addressing?
2. How does longest prefix match work? Give an example.
3. What would happen if you deleted the 0.0.0.0/0 route?
4. Why use /30 for router-to-router links?
5. How do you design subnets for growth?

