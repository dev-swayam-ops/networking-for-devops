# Exercises: Networking Fundamentals

Complete these exercises to master networking concepts. Progress from Easy to Medium difficulty.

## Easy Exercises

### Exercise 1: Identify Your Network Components
**Objective:** Learn the basic building blocks of your home network

Run this command to see your network configuration:
```powershell
ipconfig /all
```

**Task:**
- Identify your local IP address
- Find your subnet mask
- Note your default gateway (usually the router)
- Write down your DNS servers

**Questions to Answer:**
1. What is your IP address?
2. What is your default gateway IP?
3. How many DNS servers are configured?
4. Is your IP assigned via DHCP or static?

**Success Criteria:**
- Can identify all four pieces of information
- Understand what each component does
- Know how to interpret the output

---

### Exercise 2: Understand Network Types
**Objective:** Differentiate between different network types

Research and document:

| Network Type | Range | Purpose | Example |
|-------------|-------|---------|---------|
| LAN | Local | ? | Your home |
| WAN | ? | ? | The internet |
| VLAN | ? | ? | ? |

**Task:**
- Fill in the missing information for each network type
- Draw a simple diagram showing how your device connects

**Success Criteria:**
- Understand the difference between LANs and WANs
- Can explain each network type's purpose
- Can draw a basic network diagram

---

### Exercise 3: Test DNS Resolution
**Objective:** Understand how domain names are converted to IP addresses

Resolve multiple domains to see different IP addresses:

```powershell
nslookup google.com
nslookup github.com
nslookup amazon.com
```

**Task:**
- Document the IP address for each domain
- Note the DNS server used for resolution
- Try a domain that doesn't exist: `nslookup thisdomaindoesntexist123.com`

**Questions:**
1. Did different domains resolve to different IPs?
2. What happens when you query a non-existent domain?
3. Why is DNS important?

**Success Criteria:**
- Can resolve domains to IPs
- Understand that domains map to IP addresses
- Know what error message means domain doesn't exist

---

### Exercise 4: Trace Network Paths
**Objective:** See how your packets travel to reach remote servers

Trace routes to different destinations:

```powershell
tracert google.com
tracert cloudflare.com
```

**Task:**
- Count the number of hops to each destination
- Compare which one has more hops
- Note the response times at each hop

**Questions:**
1. How many hops to google.com?
2. How many hops to cloudflare.com?
3. Which is closer (fewer hops)?
4. What is the latency at hop 3?

**Success Criteria:**
- Understand that packets hop through multiple routers
- Can interpret tracert output
- Know that more hops = potentially more latency

---

### Exercise 5: Monitor Your Network Connections
**Objective:** See what applications are using your network

View all active network connections:

```powershell
netstat -an | findstr ESTABLISHED
```

**Task:**
- Run the command
- Identify at least one established connection
- Note the local port and remote port
- Identify the protocol (TCP/UDP)

**Questions:**
1. How many established connections do you have?
2. What are the different remote ports you see?
3. What application uses port 443?

**Success Criteria:**
- Understand that applications use ports
- Can identify local and remote ports
- Know that connections have established state

---

## Medium Exercises

### Exercise 6: Understand TCP vs UDP
**Objective:** Learn the difference between reliable and fast protocols

Research these protocols and complete the table:

| Aspect | TCP | UDP |
|--------|-----|-----|
| Reliability | ? | ? |
| Speed | ? | ? |
| Error Checking | ? | ? |
| Example Usage | ? | ? |
| Port Examples | 80, 443 | ? |

**Task:**
- Research TCP and UDP characteristics
- Fill in the comparison table
- List 3 applications that use TCP
- List 3 applications that use UDP

**Questions:**
1. When would you prefer TCP over UDP?
2. When would you prefer UDP over TCP?
3. Why is HTTP (port 80) TCP and not UDP?
4. Why is DNS (port 53) primarily UDP?

**Success Criteria:**
- Understand TCP is reliable but slower
- Understand UDP is fast but unreliable
- Can explain why certain apps use each protocol

---

### Exercise 7: Analyze Packet Structure
**Objective:** Understand how data is organized in network packets

Create a detailed packet breakdown:

**Task:**
1. Open a command prompt and run: `netstat -an | findstr ESTABLISHED`
2. Pick one connection and identify:
   - Source IP and port
   - Destination IP and port
   - Protocol (TCP)

3. Now imagine a packet traveling on that connection. Draw/describe:
   - What's in the Ethernet header
   - What's in the IP header
   - What's in the TCP header
   - What data might be in the payload

**Challenge:**
- Explain what happens to this packet at each network layer
- Describe how each header is added/removed as data travels

**Success Criteria:**
- Understand layered packet structure
- Can identify headers at different layers
- Understand encapsulation concept

---

### Exercise 8: Calculate Subnet Information
**Objective:** Understand IP addressing and subnet masks

For each IP address and subnet mask, calculate:

| IP Address | Subnet Mask | Network Address | Broadcast | First Host | Last Host | # Hosts |
|-----------|-------------|-----------------|-----------|-----------|----------|---------|
| 192.168.1.100 | 255.255.255.0 | ? | ? | ? | ? | ? |
| 10.0.0.50 | 255.255.255.0 | ? | ? | ? | ? | ? |
| 172.16.5.130 | 255.255.255.128 | ? | ? | ? | ? | ? |

**Task:**
- Fill in the missing values for each row
- For each network, list valid host addresses

**Hints:**
- Network address: all host bits = 0
- Broadcast address: all host bits = 1
- 255.255.255.0 = /24 = 254 usable hosts
- 255.255.255.128 = /25 = 126 usable hosts

**Success Criteria:**
- Can calculate network and broadcast addresses
- Understand how subnet masks define networks
- Know how many hosts fit in a subnet

---

### Exercise 9: Map Your Local Network
**Objective:** Discover devices on your local network

Scan your local network for active devices:

```powershell
# First, get your network info
ipconfig /all

# Then ping the entire network
# Example: if your IP is 192.168.1.100, your network is 192.168.1.0/24
# Ping all possible addresses (or first few)
ping 192.168.1.1
ping 192.168.1.254
```

**Task:**
- Get your network address from ipconfig
- Create a list of 5 IP addresses on your network
- Ping each one
- Document which ones respond
- Check ARP table: `arp -a`

**Questions:**
1. How many devices responded to ping?
2. What is the MAC address of your router?
3. Are there any unknown MAC addresses on your network?

**Challenge:**
- Find the vendor of each MAC address (search online)
- Create a network diagram showing all discovered devices

**Success Criteria:**
- Can identify devices on local network
- Understand MAC to IP mapping
- Can interpret ARP table

---

### Exercise 10: Design a Simple Network
**Objective:** Apply networking fundamentals to design a network

**Task:** Design a small office network with:
- 50 employees
- 10 servers
- 5 printers
- Wi-Fi network
- Internet connection

**Requirements:**
- Choose appropriate IP addressing scheme
- Identify network components needed
- Calculate subnet sizes
- Create a logical network diagram

**Document:**
1. Network topology (star, mesh, hybrid?)
2. IP addressing scheme (which private range?)
3. Subnets (for employees, servers, printers)
4. Network components (router, switch, firewall, etc.)
5. A simple diagram showing the design

**Example Network Design:**

```
                  Internet
                     |
              ┌──────┴──────┐
              │   Firewall  │
              └──────┬──────┘
                     |
              ┌──────┴──────┐
              │   Router    │
              └──────┬──────┘
                     |
         ┌───────────┼───────────┐
         |           |           |
      (Eth)      (Wi-Fi)     (Servers)
     Switch     Access Pt    Switch
        |           |           |
    Devices     Laptops      Servers
```

**Success Criteria:**
- Choose appropriate IP ranges (private addresses)
- Design subnets for different device types
- Include security considerations (firewalls)
- Create a clear visual diagram

---

## Challenge Questions

**Research and answer these advanced questions:**

1. What is Network Address Translation (NAT)? Why is it important?
2. Explain how a packet moves from your computer to a web server.
3. What is the difference between IPv4 and IPv6?
4. Why do we use private IP addresses instead of public ones everywhere?
5. How does DHCP work? Why is it important?

---

## Hands-On Challenge Lab

**Create a network troubleshooting scenario:**

1. Document your current network configuration completely
2. Find the MAC address of your default gateway
3. Trace the complete path to google.com
4. Identify all active connections on your computer
5. Calculate your subnet information
6. Create a visual network diagram of your setup

**Deliverables:**
- Network configuration document
- Network diagram
- List of active connections
- Subnet calculation worksheet
- Troubleshooting steps

