# Exercises: Setup and Networking Tools

Complete these exercises to practice essential networking diagnostics. Start with Easy exercises and progress to Medium.

## Easy Exercises

### Exercise 1: Display Your Network Configuration
**Objective:** Learn to view all network settings on your machine

Run the command to display detailed network configuration:
```powershell
ipconfig /all
```

**Task:**
- Execute the command and locate:
  - Your IPv4 address
  - Your subnet mask
  - Your default gateway
  - Your DNS servers
- Write down these values

**Success Criteria:**
- You can identify all four pieces of information
- Output clearly shows "Ethernet adapter" or "Wi-Fi adapter"

---

### Exercise 2: Test Basic Connectivity
**Objective:** Verify your network is working properly

Ping a reliable public DNS server:
```powershell
ping 8.8.8.8 -n 4
```

**Task:**
- Execute the command
- Note the response times (milliseconds)
- Check if any packets were lost

**Success Criteria:**
- 4 replies received (0% packet loss)
- Response time is under 100ms
- No "Request timed out" messages

---

### Exercise 3: Resolve a Domain Name
**Objective:** Understand how DNS translates domain names to IP addresses

Query a domain name:
```powershell
nslookup google.com
```

**Task:**
- Note the IP address returned for google.com
- Record the DNS server used to resolve it
- Try with another domain: `nslookup github.com`

**Success Criteria:**
- Both domains resolve to IP addresses
- No "Can't find" error messages
- DNS server is listed in your output

---

### Exercise 4: View Active Network Connections
**Objective:** See what network connections are currently active

Display all active connections:
```powershell
netstat -an | findstr ESTABLISHED
```

**Task:**
- Run the command
- Count how many established connections you have
- Identify one connection and note the protocol (TCP/UDP)

**Success Criteria:**
- Output shows at least one ESTABLISHED connection
- You can identify local and foreign addresses
- Protocol column shows TCP or UDP

---

### Exercise 5: Check Specific Port Listening
**Objective:** Identify which ports your machine is listening on

View listening ports on your system:
```powershell
netstat -an | findstr LISTENING
```

**Task:**
- Run the command
- Identify at least one port your machine is listening on
- Note if it's localhost (127.0.0.1) or all interfaces (0.0.0.0)

**Success Criteria:**
- Output shows at least one LISTENING port
- You understand the difference between listening and established
- Can identify the port number

---

## Medium Exercises

### Exercise 6: Trace Route to a Remote Server
**Objective:** Understand how your packets travel across the internet

Trace the path to a well-known website:
```powershell
tracert microsoft.com
```

**Task:**
- Note how many hops it takes to reach the destination
- Record the response times at each hop
- Compare with a different destination: `tracert cloudflare.com`

**Success Criteria:**
- tracert completes successfully (may take 30+ seconds)
- Shows at least 5-15 hops
- Can identify faster vs slower hops
- Both destinations show different paths

---

### Exercise 7: Flush and Re-cache DNS
**Objective:** Understand DNS caching and how to clear it

First, view current DNS cache:
```powershell
ipconfig /displaydns | more
```

Then flush it:
```powershell
ipconfig /flushdns
```

Verify it's cleared:
```powershell
ipconfig /displaydns | more
```

**Task:**
- Before flushing, count cached DNS entries (use arrows to scroll)
- After flushing, verify cache is empty
- Resolve a new domain: `nslookup github.com`

**Success Criteria:**
- First command shows cached entries
- After flush, returns empty or much smaller cache
- New domain resolves successfully after clearing

---

### Exercise 8: Check ARP Table
**Objective:** Learn how MAC addresses are resolved and cached

View all ARP entries:
```powershell
arp -a
```

Then, add a static ARP entry (requires admin):
```powershell
arp -s 192.168.1.50 AA-BB-CC-DD-EE-FF
```

View it again:
```powershell
arp -a
```

**Task:**
- Note the MAC addresses on your network
- Add a static entry with the command above
- Verify the entry appears in the ARP table

**Success Criteria:**
- Initial command shows at least 3 ARP entries
- Static entry successfully added
- Static entry persists in subsequent `arp -a` command

---

### Exercise 9: Monitor Network Statistics in Real-time
**Objective:** Understand continuous network monitoring

Monitor connections continuously:
```powershell
netstat -an -p tcp
```

For advanced monitoring (PowerShell):
```powershell
Get-NetTCPConnection | Group-Object -Property State | Select-Object Name, Count
```

**Task:**
- Run the first command and note protocols
- Run the second command to see connection state summary
- Identify how many connections are in different states (Established, TIME_WAIT, etc.)

**Success Criteria:**
- Can run both commands without errors
- Understands difference between TCP and UDP statistics
- Can identify connection states

---

### Exercise 10: Disable and Re-enable Network Adapter
**Objective:** Understand how to troubleshoot network issues by managing adapters

List all adapters:
```powershell
ipconfig /all
```

Get detailed network adapter info:
```powershell
Get-NetAdapter
```

**Task:**
- Find your primary network adapter name
- Note its current status (Up/Down)
- Note its IP configuration
- Document the interface name for future use

**Advanced (optional):**
```powershell
# Disable adapter (requires admin)
Disable-NetAdapter -Name "Ethernet" -Confirm:$false

# Check status
ipconfig /all

# Re-enable adapter
Enable-NetAdapter -Name "Ethernet" -Confirm:$false

# Verify it's back
ipconfig /all
```

**Success Criteria:**
- Can identify all network adapters
- Understands which is active
- Optional: Successfully disable/re-enable without permanent issues
- Can restore full connectivity after disabling

---

## Challenge Questions

**Use the tools from this module to answer:**

1. What is the latency (round-trip time) to your default gateway?
2. How many hops does it take to reach 1.1.1.1?
3. What is the MAC address of your default gateway?
4. How many open ports are listening on your machine?
5. Which DNS server is resolving your domain names?

**Share your answers to verify understanding!**
