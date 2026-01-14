# Exercises: DNS Fundamentals

Complete these exercises to master DNS queries, records, and troubleshooting.

## Easy Exercises

### Exercise 1: Understand DNS Record Types

**Objective:** Learn what different DNS records do

Match each record type to its purpose:

| Record Type | Purpose | Your Answer |
|-------------|---------|-------------|
| A | Email server address | |
| AAAA | IPv4 address | |
| MX | Text verification (SPF, DKIM) | |
| CNAME | Alias for another domain | |
| TXT | IPv6 address | |
| NS | Who manages this domain | |

**Questions:**
1. Which record is used to route emails?
2. Which record is used for domain aliases (like www)?
3. Which record has 4 A's and is for newer internet?

**Success Criteria:**
- All 6 record types matched correctly
- Can explain each one in a sentence

---

### Exercise 2: Query DNS Using nslookup

**Objective:** Learn to use nslookup tool

**Task:** Query these domains and record results:

**Windows:**
```powershell
nslookup microsoft.com
nslookup -type=MX microsoft.com
nslookup -type=NS microsoft.com
```

**Linux:**
```bash
nslookup microsoft.com
nslookup -type=MX microsoft.com
nslookup -type=NS microsoft.com
```

**Complete this table:**

| Query | Result | TTL |
|-------|--------|-----|
| microsoft.com (A record) | ? | ? |
| microsoft.com MX | ? | ? |
| microsoft.com NS | ? | ? |

**Questions:**
1. How many A records does microsoft.com have?
2. What's the mail server for microsoft.com?
3. Who manages microsoft.com domain?

**Success Criteria:**
- Successfully ran all three queries
- Filled in complete results
- Understand what each record means

---

### Exercise 3: Understand TTL Values

**Objective:** Learn what TTL means and why it matters

**Scenario:** DNS record: `google.com A 142.250.185.46 TTL=300`

**Questions:**
1. What does TTL 300 mean?
2. If you query at 10:00 AM, when will cache expire?
3. Is 300 seconds good for stable records?
4. What TTL would you use for frequently changing records?

**Complete this table:**

| TTL Value | Duration | Use Case |
|-----------|----------|----------|
| 300 | ? | ? |
| 3600 | ? | ? |
| 86400 | ? | ? |
| 604800 | ? | ? |

**Success Criteria:**
- Correctly convert TTL to time (seconds → minutes/hours)
- Understand when to use each TTL
- Know why low TTL is used before changes

---

### Exercise 4: Perform Reverse DNS Lookup

**Objective:** Find domain names from IP addresses

**Task:** Query these IPs and find their domains:

**Windows:**
```powershell
nslookup 8.8.8.8
nslookup 1.1.1.1
nslookup 142.250.185.46
```

**Linux:**
```bash
dig -x 8.8.8.8
dig -x 1.1.1.1
dig -x 142.250.185.46
```

**Complete this table:**

| IP Address | Domain Name | Service |
|------------|-------------|---------|
| 8.8.8.8 | ? | ? |
| 1.1.1.1 | ? | ? |
| 142.250.185.46 | ? | ? |

**Questions:**
1. What are these IPs (identify the companies)?
2. Why might some IPs not have reverse DNS?
3. Is reverse DNS always available?

**Success Criteria:**
- Successfully queried at least 2 IPs
- Found corresponding domain names
- Understand reverse DNS is optional

---

### Exercise 5: Check Your DNS Settings

**Objective:** Identify which DNS servers you're using

**Windows:**
```powershell
ipconfig /all | findstr "DNS"
```

**Linux:**
```bash
cat /etc/resolv.conf
```

**Complete this:**

| Setting | Your Value |
|---------|-----------|
| Primary DNS Server | ? |
| Secondary DNS Server | ? |
| DNS Suffix | ? |
| Interface Name | ? |

**Questions:**
1. What DNS servers is your system using?
2. Are they ISP-provided or public?
3. Can you identify the provider (Google, CloudFlare, etc.)?

**Success Criteria:**
- Successfully ran ipconfig or cat command
- Identified your primary DNS server
- Know what the values mean

---

## Medium Exercises

### Exercise 6: Query Specific Nameserver

**Objective:** Query authoritative nameserver instead of resolver

**Task:** Find the authoritative nameserver, then query it

**Step 1: Find NS records**
```powershell
nslookup -type=NS example.com
```

**Step 2: Query that nameserver directly**
```powershell
nslookup example.com [nameserver-from-step1]
```

**Complete this:**

| Domain | NS Server | A Record from NS |
|--------|-----------|------------------|
| example.com | ? | ? |
| microsoft.com | ? | ? |

**Questions:**
1. Why might results differ from regular nslookup?
2. Why is authoritative NS more trustworthy?
3. When would you query authoritative NS instead of ISP DNS?

**Success Criteria:**
- Successfully identified NS records
- Queried authoritative nameserver
- Understand difference: recursive vs authoritative

---

### Exercise 7: Analyze DNS Resolution Chain

**Objective:** Understand how DNS resolution works end-to-end

**Scenario:** You query: `nslookup www.google.com`

**Task:** Draw or describe the lookup chain:

1. Client (your computer) → ?
2. Recursive Resolver (8.8.8.8) → ?
3. Root Nameserver → ?
4. TLD Server (.com) → ?
5. Authoritative NS (Google) → ?

**Complete this table:**

| Step | Server Asks | Server Answers |
|------|-------------|----------------|
| 1 | Your computer | DNS server: ? |
| 2 | Recursive resolver | Root NS: "ask ? for .com" |
| 3 | Recursive to TLD | TLD server: "ask ? for google.com" |
| 4 | Recursive to Auth | Auth NS: "www.google.com = ?" |
| 5 | Back to you | Result: 142.250.185.46 |

**Questions:**
1. How many servers are involved in one DNS query?
2. Why do we need this chain instead of asking one server?
3. What would happen if one server was down?

**Success Criteria:**
- Understand 5-step DNS resolution
- Know each server's role
- Can explain why each step is needed

---

### Exercise 8: Troubleshoot DNS Issues

**Objective:** Diagnose common DNS problems

**Scenarios:**

**Scenario A:**
```
Problem: nslookup example.com fails, but ping 142.250.185.46 works
Network appears connected
```

**Your diagnosis:**
- What's the problem?
- How would you verify?
- How would you fix it?

**Scenario B:**
```
Problem: Website accessible by IP (142.250.185.50) but not by domain (example.com)
nslookup shows: 142.250.185.46 (different IP!)
Browser shows timeout
```

**Your diagnosis:**
- What's wrong?
- How would you verify?
- How would you fix it?

**Scenario C:**
```
Problem: DNS change made 2 hours ago, but still shows old IP
Other networks show new IP correctly
```

**Your diagnosis:**
- What's the cause?
- How would you verify?
- What's the fix (besides waiting)?

**Success Criteria:**
- Identify root cause for each scenario
- Know troubleshooting steps
- Understand DNS caching impact

---

### Exercise 9: Compare DNS Servers

**Objective:** Understand that different DNS servers may give different results

**Task:** Query the same domain from different DNS servers

**Windows:**
```powershell
nslookup example.com 8.8.8.8      (Google DNS)
nslookup example.com 1.1.1.1      (CloudFlare DNS)
nslookup example.com 208.67.222.222 (OpenDNS)
```

**Complete this table:**

| DNS Server | Result for example.com | TTL | Cache Age |
|------------|------------------------|-----|-----------|
| 8.8.8.8 | ? | ? | ? |
| 1.1.1.1 | ? | ? | ? |
| 208.67.222.222 | ? | ? | ? |
| Your ISP | ? | ? | ? |

**Questions:**
1. Are the results identical?
2. Why might TTL differ between servers?
3. When should you use different DNS servers?
4. What's the benefit of public DNS (8.8.8.8, 1.1.1.1)?

**Success Criteria:**
- Queried same domain from 3+ DNS servers
- Compared results
- Understand DNS caching differences

---

### Exercise 10: Design DNS Configuration

**Objective:** Plan DNS setup for a scenario

**Scenario:** You're setting up internet for a company:

- 500 employees across 3 offices
- Needs reliable DNS (can't have downtime)
- Some services require fast response
- Security is important

**Task: Design DNS setup:**

**Question 1: Which DNS servers?**
```
[ ] ISP-provided (example: 10.0.0.1, 10.0.0.2)
[ ] Public DNS (Google 8.8.8.8, CloudFlare 1.1.1.1)
[ ] Private DNS server (your own)
[ ] Hybrid approach: ?
```

**Question 2: Single or multiple?**
```
Primary DNS: ?
Secondary DNS: ?
Why this setup?
```

**Question 3: Local or cloud?**
```
Should you run your own DNS server?
Why or why not?
Benefits/drawbacks?
```

**Question 4: TTL settings?**
```
For primary website:  TTL = ?  (why?)
For internal services: TTL = ? (why?)
For email MX: TTL = ? (why?)
```

**Your design:**

| Setting | Choice | Reason |
|---------|--------|--------|
| Primary DNS | ? | ? |
| Secondary DNS | ? | ? |
| Tertiary DNS | ? | ? |
| TTL for web | ? | ? |
| TTL for email | ? | ? |

**Success Criteria:**
- Reasonable DNS server choices
- Understands redundancy needs
- Considers performance and reliability
- Explains each decision

---

## Challenge Questions

**Test your deep understanding:**

1. **Why is DNS important?** Explain to someone who thinks "We could just use IPs"

2. **What would happen to the internet if all root nameservers went down?** Think about it.

3. **Why does CNAME exist if we have A records?** When would you use each?

4. **If you change a DNS record, why doesn't it take effect immediately?** What are all the cache layers?

5. **How would you check if a domain's DNS has fully propagated worldwide?** What would you query?

6. **What's the difference between `nslookup example.com` and `nslookup example.com ns1.example.com`?** When use each?

7. **Why is MX priority number inverse (lower = better)?** Could it be different?

8. **If a company has multiple A records for one domain, how does the browser choose?** Is it random?

9. **What security risks exist with DNS?** (Hint: DNS poisoning, spoofing)

10. **How would you block ads using DNS?** Think about what you could do at the DNS server level.

