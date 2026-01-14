# Exercises: OSI and TCP/IP Models

Complete these exercises to master network models and troubleshooting. Progress from Easy to Medium.

## Easy Exercises

### Exercise 1: Memorize the OSI Layers
**Objective:** Learn the 7 OSI layers in order

**Task:**
- Memorize the memory aid: "Please Do Not Throw Sausage Pizza Away"
- This stands for: Physical, Data Link, Network, Transport, Session, Presentation, Application
- Write them out 5 times
- Create your own memory aid

**Verification:**
- Can you list all 7 layers top-to-bottom?
- Can you list them bottom-to-top?
- Can you explain what each does in one sentence?

**Success Criteria:**
- Instant recall of all 7 layers in order
- Can explain function of each layer
- Created your own memory aid

---

### Exercise 2: Map Devices to OSI Layers
**Objective:** Understand which network devices operate at which layers

**Task:** Complete this table:

| Device | Layer(s) | Purpose | Example |
|--------|----------|---------|---------|
| Hub | ? | ? | ? |
| Switch | ? | ? | ? |
| Router | ? | ? | ? |
| Firewall | ? | ? | ? |
| Modem | ? | ? | ? |
| Wi-Fi Access Point | ? | ? | ? |
| Cable | ? | ? | ? |

**Research:**
- Hub: Repeats data on all ports (simple)
- Switch: Learns MAC addresses (smart hub)
- Router: Routes between networks
- Firewall: Filters packets
- Modem: Converts ISP signal
- Access Point: Wireless
- Cable: Physical medium

**Success Criteria:**
- Correctly identify layer for each device
- Understand why each operates at that layer
- Can explain what makes them different

---

### Exercise 3: Map Protocols to OSI Layers
**Objective:** Learn which protocols operate at which layers

**Task:** Place these protocols in the correct layer:

Protocols: HTTP, TCP, IP, Ethernet, DNS, FTP, ICMP, SSH, UDP, Frame Relay

| Layer 7 App | Layer 6 Pres | Layer 5 Sess | Layer 4 Trans | Layer 3 Net | Layer 2 Link | Layer 1 Phys |
|-------------|-------------|------------|--------------|-----------|------------|-----------|
| ? | ? | ? | ? | ? | ? | ? |

**Answers Guide:**
- Layer 7: HTTP, DNS, FTP, SSH (user applications)
- Layer 4: TCP, UDP (transport protocols)
- Layer 3: IP, ICMP (network/routing)
- Layer 2: Ethernet, Frame Relay (framing)
- Layer 1: (None in list - physical is cables/signals)

**Success Criteria:**
- Correctly classify all 10 protocols
- Understand why each belongs in that layer
- Know which protocols you'll work with most

---

### Exercise 4: Understand Data Encapsulation
**Objective:** Learn how headers are added at each layer

**Task:** Draw the encapsulation process:

Starting with: "GET /index.html" (HTTP request)

Add layer by layer, showing what gets added at each layer:

```
┌──────────────────────────────────────┐
│ Layer 7 Application                  │
│ [_________________]                  │
├──────────────────────────────────────┤
│ Layer 6 Presentation                 │
│ [_________________] + Application    │
├──────────────────────────────────────┤
│ Layer 5 Session                      │
│ [_________________] + ? + App        │
├──────────────────────────────────────┤
│ Layer 4 Transport                    │
│ [_________________] + ? + Sess + App │
├──────────────────────────────────────┤
│ Layer 3 Network                      │
│ [_________________] + ? + ... + App  │
├──────────────────────────────────────┤
│ Layer 2 Data Link                    │
│ [_________________] + ? + ... + App  │
├──────────────────────────────────────┤
│ Layer 1 Physical                     │
│ Electrical signals                   │
└──────────────────────────────────────┘
```

**Answer:**
- Layer 7: "GET /index.html" (no header added)
- Layer 4: Add TCP header (ports)
- Layer 3: Add IP header (addresses)
- Layer 2: Add Ethernet header (MAC addresses)
- Layer 1: Convert to electrical signals

**Success Criteria:**
- Can draw encapsulation process
- Understand what each layer adds
- Know this process happens automatically

---

### Exercise 5: Identify Layers in Real Network Activity
**Objective:** See OSI layers in action on your computer

Run these commands and identify which OSI layer each reveals:

```powershell
# Command 1
ipconfig /all
# Which layer? _____

# Command 2
arp -a
# Which layer? _____

# Command 3
route print
# Which layer? _____

# Command 4
netstat -an | findstr ESTABLISHED
# Which layer? _____

# Command 5
Get-Process | Where-Object {$_.Name -eq "chrome"}
# Which layer? _____
```

**Task:**
- Run each command
- Identify what layer it shows
- Explain why it shows that layer

**Answers:**
1. ipconfig = Layer 3 (IP addresses) + Layer 2 (MAC addresses)
2. arp -a = Layer 2 (MAC to IP mapping)
3. route print = Layer 3 (IP routing)
4. netstat = Layer 4 (Ports and protocols)
5. Get-Process = Layer 5-7 (Applications)

**Success Criteria:**
- Can identify layer for each command
- Understand why it shows that layer
- Can use these commands to diagnose layer issues

---

## Medium Exercises

### Exercise 6: Compare OSI vs TCP/IP Models
**Objective:** Understand the difference between theoretical and practical models

**Task:** Create a comparison table:

| Aspect | OSI Model | TCP/IP Model |
|--------|-----------|--------------|
| Number of layers | ? | ? |
| Type (theoretical/practical) | ? | ? |
| Layer 1 name | ? | ? |
| Has Session layer | ? | ? |
| Commonly used | ? | ? |
| Layer 3 name | ? | ? |

**Research:**
- OSI: 7 layers, theoretical, doesn't include session in real protocols
- TCP/IP: 4 layers, practical, combine presentation/session/application

**Advanced:**
- Map each TCP/IP layer to OSI layers:
  - TCP/IP Application = OSI Layers ?
  - TCP/IP Transport = OSI Layer ?
  - TCP/IP Internet = OSI Layer ?
  - TCP/IP Link = OSI Layers ?

**Success Criteria:**
- Understand both models serve different purposes
- Can map between the two models
- Know which to use when (OSI for learning, TCP/IP for practice)

---

### Exercise 7: Troubleshoot Layer by Layer
**Objective:** Use OSI model to solve network problems

**Scenario:** User says "I can't access any websites"

**Task:** Create a troubleshooting flowchart:

**Step 1: Physical Layer**
```powershell
# Command:
# Check: ?
# If fails: Check cable, ports, network adapter
```

**Step 2: Data Link Layer**
```powershell
# Command:
# Check: ?
# If fails: Check MAC address, ARP
```

**Step 3: Network Layer**
```powershell
# Command:
# Check: ?
# If fails: Check IP config, gateway, routing
```

**Step 4: Transport Layer**
```powershell
# Command:
# Check: ?
# If fails: Check firewall, ports
```

**Step 5: Application Layer**
```powershell
# Command:
# Check: ?
# If fails: Check browser, DNS, application settings
```

**Answer Guide:**
1. Physical: `Get-NetAdapter` (Status: Up?)
2. Data Link: `arp -a` (See any devices?)
3. Network: `ipconfig /all` (Have IP? Can ping gateway?)
4. Transport: `netstat -an` (Any connections?)
5. Application: Try different website (maybe that site is down)

**Success Criteria:**
- Can systematically troubleshoot from Layer 1 to 7
- Know which commands to use at each layer
- Understand each layer depends on lower layers

---

### Exercise 8: Analyze a Real Packet
**Objective:** Understand packet structure from real network traffic

**Task:** Create a detailed packet breakdown:

1. Establish a network connection:
   ```powershell
   # Open web browser
   # Visit: https://example.com
   ```

2. Capture the connection:
   ```powershell
   netstat -an | findstr example.com
   ```

3. Create packet diagram:
   ```
   Layer 1 Physical: ?
   ├─ Medium: Copper/Fiber?
   ├─ Signal: Electrical/Optical?
   └─ Speed: ? Mbps/Gbps?
   
   Layer 2 Data Link: ?
   ├─ Source MAC: ?
   ├─ Dest MAC: ?
   └─ Frame type: Ethernet?
   
   Layer 3 Network: ?
   ├─ Source IP: ?
   ├─ Dest IP: ?
   ├─ Protocol: IPv4/IPv6?
   └─ TTL: ?
   
   Layer 4 Transport: ?
   ├─ Source Port: ?
   ├─ Dest Port: 443?
   ├─ Protocol: TCP?
   └─ Sequence: ?
   
   Layer 5-7 Application: ?
   ├─ Protocol: TLS/SSL?
   ├─ HTTP Method: GET/POST?
   └─ Data: HTTPS request?
   ```

**Advanced:**
- Can you explain what happens at each layer?
- What information is critical at each layer?
- What happens if one piece of information is wrong?

**Success Criteria:**
- Can identify real packet headers
- Understand what information each layer carries
- Can trace a real packet through OSI model

---

### Exercise 9: Design a Network Using OSI Model
**Objective:** Apply OSI model to network design

**Scenario:** Design a secure network for a small business with:
- 20 employees
- 3 servers
- Printers
- Internet access
- Security requirements

**Task:** Plan each OSI layer:

**Layer 1: Physical**
- What cables? Copper/Fiber?
- What devices? Switches?
- Layout: ?

**Layer 2: Data Link**
- What MAC addressing scheme?
- VLANs needed?
- Switch types?

**Layer 3: Network**
- IP addressing scheme?
- Subnets?
- Routers/gateways?

**Layer 4: Transport**
- TCP or UDP preferred? Why?
- Firewall rules?
- Port policies?

**Layer 5-7: Session/Presentation/Application**
- What applications needed?
- Encryption (TLS)?
- Authentication?

**Success Criteria:**
- Can design network using systematic layer approach
- Understand how layers depend on each other
- Make security decisions at appropriate layers

---

### Exercise 10: Create an OSI Model Troubleshooting Guide
**Objective:** Build your personal troubleshooting reference

**Task:** Create a complete troubleshooting guide:

```markdown
# OSI Layer Troubleshooting Guide

## Layer 1: Physical
**Problems that indicate Layer 1:**
- ?
- ?

**Diagnostic commands:**
- ?
- ?

**Common fixes:**
- ?
- ?

---

## Layer 2: Data Link
**Problems:**
- ?

**Commands:**
- ?

**Fixes:**
- ?

---

[Repeat for Layers 3-7]
```

**Requirements:**
- At least 2 problems per layer
- At least 2 commands per layer
- At least 2 fixes per layer
- Use real commands you've learned
- Make it practical and actionable

**Example for Layer 1:**
```markdown
## Layer 1: Physical

**Problems:**
- Cable disconnected
- Port broken
- NIC not working

**Commands:**
- Get-NetAdapter (check Status)
- ipconfig /all (check if adapter listed)

**Fixes:**
- Reconnect cable
- Try different port
- Replace network cable
- Update NIC drivers
```

**Success Criteria:**
- Complete guide for all 7 layers
- Practical, actionable information
- Based on real troubleshooting
- Can be used as quick reference

---

## Challenge Questions

**Test your understanding:**

1. Why do we need both OSI and TCP/IP models?
2. How does understanding layers help with troubleshooting?
3. Why is Layer 2 (MAC) important if Layer 3 (IP) does routing?
4. Can you have a Layer 4 problem if Layer 3 is working?
5. What's the relationship between encapsulation and the OSI model?

---

## Hands-On Challenge Lab

**Create a complete network analysis:**

1. **Document your computer's network stack:**
   - Layer 1: Physical medium (Ethernet/Wi-Fi)
   - Layer 2: MAC address
   - Layer 3: IP address and routing
   - Layer 4: Active ports and protocols
   - Layer 5-7: Active applications

2. **Trace a complete connection:**
   - Visit a website
   - Document each layer's information
   - Explain what happens at each layer

3. **Simulate a network problem:**
   - Disable network adapter
   - Try to troubleshoot step-by-step
   - Use OSI model to identify problem layer
   - Fix the problem

**Deliverables:**
- Network stack documentation
- Connection trace with OSI layers
- Troubleshooting analysis

