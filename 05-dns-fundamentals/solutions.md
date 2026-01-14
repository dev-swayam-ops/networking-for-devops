# Solutions: DNS Fundamentals

Complete answer key for all exercises with explanations and expected outputs.

## Easy Exercise Solutions

### Solution 1: Understand DNS Record Types

**Correct Matches:**

| Record Type | Purpose | Explanation |
|-------------|---------|-------------|
| A | IPv4 address | Maps domain name to IPv4 (e.g., 142.250.185.46) |
| AAAA | IPv6 address | Maps domain to IPv6 (4x A's = modern protocol) |
| MX | Email server address | Routes mail (lower priority number = preferred) |
| CNAME | Alias for another domain | Points to another domain name |
| TXT | Text verification | SPF, DKIM, domain verification |
| NS | Who manages this domain | Nameserver that holds the records |

**Answers:**

1. **Which record routes emails?** MX (Mail Exchange)

2. **Which record is for domain aliases?** CNAME (Canonical Name)
   - Example: www.google.com CNAME → google.com

3. **Which has 4 A's and is newer?** AAAA (IPv6 address record)
   - AAAA has 4 A's
   - IPv6 is newer (replaces IPv4)
   - Example: 2607:f8b0:4004:809::200e

---

### Solution 2: Query DNS Using nslookup

**Expected Output Example (your results will vary):**

**Query 1: A Record**
```powershell
PS C:\> nslookup microsoft.com

Server:  8.8.8.8
Address: 8.8.8.8

Non-authoritative answer:
Name:    microsoft.com
Address: 13.107.42.14
Address: 13.107.43.14
Address: 23.96.52.53
...
```

**Query 2: MX Record**
```powershell
PS C:\> nslookup -type=MX microsoft.com

Server:  8.8.8.8
Address: 8.8.8.8

Non-authoritative answer:
microsoft.com   MX preference = 10, mail exchanger = outlook-com.olc.protection.outlook.com
microsoft.com   MX preference = 20, mail exchanger = outlook-com.olc.protection.outlook.com
...
```

**Query 3: NS Record**
```powershell
PS C:\> nslookup -type=NS microsoft.com

Server:  8.8.8.8
Address: 8.8.8.8

Non-authoritative answer:
microsoft.com   nameserver = ns1.msft.net
microsoft.com   nameserver = ns2.msft.net
microsoft.com   nameserver = ns3.msft.net
microsoft.com   nameserver = ns4.msft.net
...
```

**Answer Table (example):**

| Query | Result | TTL |
|-------|--------|-----|
| microsoft.com (A) | 13.107.42.14 | 300 |
| microsoft.com MX | outlook-com.olc.protection.outlook.com (pref 10) | 3600 |
| microsoft.com NS | ns1.msft.net | 172800 |

**Answers:**

1. **How many A records?** Usually 3-5 (load balancing)

2. **Mail server?** outlook-com.olc.protection.outlook.com (priority 10)

3. **Who manages domain?** ns1.msft.net, ns2.msft.net, ns3.msft.net, ns4.msft.net (Microsoft nameservers)

---

### Solution 3: Understand TTL Values

**Complete Table:**

| TTL Value | Duration | Use Case | When to Use |
|-----------|----------|----------|------------|
| 300 | 5 minutes | Testing, frequent changes | Before DNS migration |
| 3600 | 1 hour | Server updates | Normal changes |
| 86400 | 1 day | Stable records | Primary website IP |
| 604800 | 1 week | Very stable | Secondary DNS, rarely changes |

**Scenario Analysis: google.com A 142.250.185.46 TTL=300**

**Question 1: What does TTL 300 mean?**
Answer: Cache this answer for 300 seconds (5 minutes) before asking again

**Question 2: If you query at 10:00 AM, when expires?**
Answer: At 10:05 AM (300 seconds = 5 minutes later)

**Question 3: Is 300 seconds good for stable records?**
Answer: No, 300 is too low. Use 86400 or higher.
- Pro: Quick updates if changed
- Con: More queries, more load on DNS servers
- Google.com is stable, should use high TTL (like 3600)

**Question 4: TTL for frequently changing records?**
Answer: 300-1800 (5-30 minutes)
- Reasons: Can update quickly if needed
- Examples: Testing environments, dynamic IPs

---

### Solution 4: Perform Reverse DNS Lookup

**Expected Results:**

| IP Address | Domain Name | Service |
|------------|-------------|---------|
| 8.8.8.8 | dns.google | Google's public DNS |
| 1.1.1.1 | one.one.one.one | CloudFlare's public DNS |
| 142.250.185.46 | fra16s12-in-f14.1e100.net | Google infrastructure (may vary) |

**Example Output:**

```powershell
PS C:\> nslookup 8.8.8.8

Server:  8.8.8.8
Address: 8.8.8.8

Name:    dns.google
Address: 8.8.8.8
```

**Answers:**

1. **What are these IPs?**
   - 8.8.8.8 = Google DNS
   - 1.1.1.1 = CloudFlare DNS
   - 142.250.185.46 = Google web server

2. **Why might some IPs not have reverse DNS?**
   - Reverse DNS is optional (needs PTR record)
   - Not all server operators set it up
   - Some companies keep it private for security

3. **Is reverse DNS always available?**
   - No, it's optional
   - Depends on IP owner's DNS configuration
   - Mostly ISPs and large companies set it up

---

### Solution 5: Check Your DNS Settings

**Example Output (Windows):**

```powershell
PS C:\> ipconfig /all | findstr "DNS"

DNS Servers . . . . . . . . . . : 8.8.8.8
                                  8.8.4.4
Connection-specific DNS Suffix . :
```

**Example Table:**

| Setting | Your Value | Meaning |
|---------|-----------|---------|
| Primary DNS Server | 8.8.8.8 | Google DNS primary |
| Secondary DNS Server | 8.8.4.4 | Google DNS backup |
| DNS Suffix | (none) | No local domain suffix |
| Interface Name | Ethernet | Network adapter name |

**Answers:**

1. **What DNS servers are you using?**
   - Answer will depend on your network
   - Could be ISP DNS, public DNS (Google, CloudFlare), or corporate DNS

2. **Are they ISP-provided or public?**
   - ISP: Usually in 10.0.x.x range or ISP's IP range
   - Public: 8.8.8.8 (Google), 1.1.1.1 (CloudFlare), etc.

3. **Can you identify the provider?**
   - 8.8.8.8 = Google
   - 1.1.1.1 = CloudFlare
   - 9.9.9.9 = Quad9
   - ISP's own DNS = company name

---

## Medium Exercise Solutions

### Solution 6: Query Specific Nameserver

**Step 1: Find NS records**

```powershell
PS C:\> nslookup -type=NS example.com

Server:  8.8.8.8
Address: 8.8.8.8

Non-authoritative answer:
example.com     nameserver = a.iana-servers.net
example.com     nameserver = b.iana-servers.net
```

**Step 2: Query that nameserver**

```powershell
PS C:\> nslookup example.com a.iana-servers.net

Server:  a.iana-servers.net
Address: 199.43.135.53

Name:    example.com
Address: 93.184.216.34
```

**Complete Table (examples):**

| Domain | NS Server | A Record from NS |
|--------|-----------|------------------|
| example.com | a.iana-servers.net | 93.184.216.34 |
| microsoft.com | ns1.msft.net | 13.107.42.14 |

**Answers:**

1. **Why might results differ?**
   - Regular nslookup uses ISP's resolver (might be cached, might be old)
   - Direct NS query gets fresh answer from authority
   - Caching differences between resolver and NS

2. **Why is authoritative NS more trustworthy?**
   - It owns the records (authoritative)
   - ISP resolver might have stale cache
   - Authoritative = source of truth

3. **When would you query authoritative NS?**
   - Troubleshooting DNS issues
   - Verifying DNS propagation
   - Checking if records were actually updated
   - Bypassing ISP DNS cache

---

### Solution 7: Analyze DNS Resolution Chain

**Complete Resolution Chain:**

| Step | Server Asks | Server Answers |
|------|-------------|----------------|
| 1 | Your browser | ISP DNS resolver: "I'll find www.google.com" |
| 2 | ISP resolver | Root NS: "For .com domains, ask 192.5.6.30" |
| 3 | ISP resolver | .COM TLD: "For google.com, ask ns1.google.com (216.239.32.10)" |
| 4 | ISP resolver | Google's NS: "www.google.com = 142.250.185.46" |
| 5 | ISP resolver back to browser | Browser: Here's the IP! |

**Visual Representation:**

```
Client (You)
    ↓ "What's www.google.com?"
ISP Resolver (8.8.8.8)
    ↓ "Who handles .com?"
Root Nameserver (.) 
    ↓ "Ask 192.5.6.30"
TLD Server (.com)
    ↓ "Ask ns1.google.com"
Authoritative NS (google.com)
    ↓ "It's 142.250.185.46"
ISP Resolver
    ↓ "Answer: 142.250.185.46"
Client gets IP
```

**Answers:**

1. **How many servers involved?**
   - 5 (Client, Resolver, Root, TLD, Authoritative)
   - Or 4 if not counting Root (root is implicit)

2. **Why this chain instead of one server?**
   - Scalability: No single server handles all domains
   - Redundancy: Multiple servers for each level
   - Distribution: Load spread across many servers
   - Delegation: Each level manages its part

3. **What if one server was down?**
   - Root: 13 roots worldwide, likely one up
   - TLD: Multiple servers per TLD, one usually available
   - Authoritative: Domain owner usually has 2-4 NS servers
   - Resolver: Your ISP has multiple resolvers
   - Result: DNS is highly fault-tolerant

---

### Solution 8: Troubleshoot DNS Issues

**Scenario A:**
```
Problem: nslookup example.com fails, but ping 142.250.185.46 works
Network appears connected
```

**Diagnosis:**
- **Problem:** DNS is broken (resolution fails)
- **NOT:** Network is broken (ping works)
- **Specific issue:** DNS server down, port 53 blocked, or DNS misconfigured

**How to verify:**
```powershell
nslookup 8.8.8.8                    # Can resolve IPs?
nslookup google.com 8.8.8.8         # Works with Google DNS?
ipconfig /all | findstr "DNS"       # Check your DNS servers
ping 8.8.8.8                        # Can reach DNS server?
```

**How to fix:**
1. Change to public DNS: `Set-DNSClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses ("8.8.8.8","8.8.4.4")`
2. Check firewall allows port 53 (UDP)
3. Restart network adapter
4. Check DHCP is enabled for DNS

---

**Scenario B:**
```
Problem: IP works (142.250.185.50) but domain fails (example.com)
nslookup shows: 142.250.185.46 (different IP!)
Browser shows timeout
```

**Diagnosis:**
- **Problem:** DNS returns wrong IP
- **Root cause:** Website moved to new server, DNS not updated
- **The IPs don't match:** 142.250.185.50 (actual) vs 142.250.185.46 (DNS)

**How to verify:**
```powershell
# Check what IP DNS returns
nslookup example.com

# Check what IP browser sees
Resolve-DnsName example.com

# Query authoritative NS
nslookup example.com [ns-from-above]

# Check if propagated globally
nslookup example.com 8.8.8.8
nslookup example.com 1.1.1.1
```

**How to fix:**
1. Update DNS record to correct IP (142.250.185.50)
2. Lower TTL before change (so caches update faster)
3. Wait for propagation (DNS caches must expire)
4. Clear local cache: `ipconfig /flushdns`
5. Test from different DNS servers

---

**Scenario C:**
```
Problem: DNS changed 2 hours ago, still shows old IP
Other networks show new IP correctly
```

**Diagnosis:**
- **Problem:** DNS cache hasn't expired yet
- **Root cause:** TTL not expired on your system
- **Why others work:** They have lower TTL or newer cache

**How to verify:**
```powershell
# Check your cache
Get-DnsClientCache | findstr "example.com"

# Check the DNS server directly
nslookup example.com [authoritative-ns]

# Check other resolvers
nslookup example.com 8.8.8.8
nslookup example.com 1.1.1.1
```

**How to fix (besides waiting):**
1. **Immediate:** Clear DNS cache
   ```powershell
   ipconfig /flushdns
   ```

2. **Browser:** Clear browser cache (has its own cache)
   ```
   Chrome: Ctrl+Shift+Delete → Cookies/Images
   Edge: Ctrl+Shift+Delete → Cache/Cookies
   ```

3. **Prevent next time:**
   - Lower TTL 24 hours BEFORE change
   - Set to 300 (5 minutes) during change
   - Set back to 3600+ after verification

---

### Solution 9: Compare DNS Servers

**Expected Results (will vary by domain and time):**

| DNS Server | Result for example.com | TTL | Notes |
|------------|------------------------|-----|-------|
| 8.8.8.8 | 93.184.216.34 | 86400 | Google DNS |
| 1.1.1.1 | 93.184.216.34 | 86400 | CloudFlare DNS |
| 208.67.222.222 | 93.184.216.34 | 300 | OpenDNS |
| Your ISP | 93.184.216.34 | 3600 | Your ISP |

**Answers:**

1. **Are results identical?**
   - IP should be same (93.184.216.34)
   - TTL might differ (depends on cache age)
   - Usually very similar, occasionally slightly different

2. **Why might TTL differ?**
   - Different servers = different cache times
   - If cached longer ago, original TTL reduced
   - Example: TTL=3600, cached 1 hour ago = TTL now is ~1800
   - Freshly queried = shows original TTL

3. **When should you use different DNS?**
   - ISP blocked? Try 8.8.8.8 or 1.1.1.1
   - Performance testing? Compare multiple
   - Security concern? Use 9.9.9.9 (malware filtering)
   - Privacy concern? Use 1.1.1.1 (no logging)
   - Corporate? Use corporate DNS for intranet access

4. **Benefits of public DNS (8.8.8.8, 1.1.1.1)?**
   - ✓ Faster than ISP (especially if ISP is slow)
   - ✓ No censoring/blocking (unlike some ISPs)
   - ✓ Privacy (some providers don't log)
   - ✓ Redundancy (use as backup)
   - ✗ Might not resolve intranet names (needs corporate DNS)

---

### Solution 10: Design DNS Configuration

**Recommended Design:**

| Setting | Choice | Reason |
|---------|--------|--------|
| Primary DNS | 8.8.8.8 (Google) | Reliable, fast, public |
| Secondary DNS | 1.1.1.1 (CloudFlare) | Different provider, still reliable |
| Tertiary DNS | Corporate DNS (if applicable) | For internal services |
| TTL for web | 3600 (1 hour) | Stable but can update within hour |
| TTL for email | 86400 (1 day) | Very stable, rarely changes |

**Detailed Reasoning:**

**Question 1: Which DNS servers?**

Best approach: **Hybrid**
- Primary: Public DNS (8.8.8.8) - reliable, fast
- Secondary: Different public DNS (1.1.1.1) - redundancy
- Optional: Corporate DNS for internal services

Why not ISP DNS?
- ✗ Often slower
- ✗ Sometimes blocks sites
- ✗ Subject to ISP outages
- ✓ Only benefit: resolves ISP-specific services

---

**Question 2: Single or multiple?**

**Answer: Multiple for redundancy**

Primary DNS: 8.8.8.8
Secondary DNS: 1.1.1.1

Why this setup?
- ✓ If Google DNS down, CloudFlare works
- ✓ Load balancing (queries spread)
- ✓ No single point of failure
- ✓ 500 employees need reliability

---

**Question 3: Local or cloud?**

**Answer: Local DNS server for internal services**

Should you run your own?
- Yes, if: Large internal network, many internal services, compliance requirements
- No, if: Small company, only external services, limited IT resources

Benefits:
- ✓ Control over internal service discovery
- ✓ Can resolve internal hostnames (example.internal)
- ✓ Caching at organization level
- ✓ Split-brain DNS (different external/internal answers)

Drawbacks:
- ✗ Maintenance burden
- ✗ Potential single point of failure
- ✗ Requires IT expertise
- ✗ Cost of hardware/licensing

Hybrid approach:
- Keep public DNS for internet
- Add local DNS for internal services
- Local DNS forwards to 8.8.8.8 for internet

---

**Question 4: TTL settings?**

```
Primary website: TTL = 3600 (1 hour)
  Why: Stable, but allows updates within hour
  
Email MX: TTL = 86400 (1 day)
  Why: Very rarely changes, can be high TTL
  
Internal services: TTL = 300-600 (5-10 min)
  Why: May change during maintenance/updates
  
Testing DNS: TTL = 60-300 (1-5 min)
  Why: Rapid iteration needed
```

---

## Challenge Question Answers

**1. Why is DNS important?**

Answer: DNS is the "phonebook of the internet." Without it:
- Every website would need you to remember its IP (impossible for humans)
- Server migrations would break all links (users still remember domain)
- Load balancing couldn't work (one domain → multiple IPs)
- Email routing wouldn't work (MX records)
- The internet would be unusable for non-technical people

---

**2. What would happen if all root nameservers went down?**

Answer: **Nothing would happen (immediately)**
- Recursive resolvers cache root server addresses
- Can resolve cached lookups for 24-48 hours
- New domains/TLDs couldn't be queried
- Eventually: Lookups fail after caches expire

This is why there are 13 root nameservers worldwide with redundancy.

---

**3. Why does CNAME exist if we have A records?**

Answer: CNAME is for **aliasing**

- A record: Points to IP (93.184.216.34)
- CNAME: Points to another domain (www.example.com → example.com)

Use CNAME when:
- ✓ Multiple names point to same server (www, api, mail subdomains)
- ✓ Want to avoid duplicating IPs
- ✓ Make management easier (change IP once, affects all CNAMEs)

Can't use CNAME at zone apex:
- ✓ Use A record: example.com
- ✗ Can't use CNAME: example.com CNAME → example.com

---

**4. Why doesn't DNS change take effect immediately?**

Answer: **Multiple caching layers:**

1. **Browser cache** (2 minutes) - Clear browser cache
2. **OS cache** (Windows DNS cache) - `ipconfig /flushdns`
3. **ISP resolver cache** (24+ hours) - Beyond your control
4. **Old nameserver caches** (per TTL) - Wait for TTL to expire
5. **CDN caches** (if using CDN) - May cache longer

Solution: Lower TTL 24 hours BEFORE making changes, then wait for propagation.

---

**5. How would you check if DNS has fully propagated worldwide?**

Answer: **Query from multiple locations/resolvers:**

```powershell
# Check from different public DNS servers
nslookup example.com 8.8.8.8       (Google)
nslookup example.com 1.1.1.1       (CloudFlare)
nslookup example.com 9.9.9.9       (Quad9)

# Query authoritative NS
nslookup -type=NS example.com
nslookup example.com [ns-server]

# Online tools to check globally
- whats-dns.com
- dnschecker.org
- propagationchecker.com (email specific)
```

If all match, propagation is complete.

---

**6. Difference between `nslookup example.com` vs `nslookup example.com ns1.example.com`?**

Answer:

| Command | Queries | Result |
|---------|---------|--------|
| `nslookup example.com` | Your ISP's resolver (recursive) | May be cached, might be old |
| `nslookup example.com ns1.example.com` | Authoritative nameserver (iterative) | Fresh answer, source of truth |

When to use each:
- Regular nslookup: Daily browsing, quick checks
- Direct NS query: Troubleshooting, verifying updates, bypassing cache

---

**7. Why is MX priority inverse (lower = better)?**

Answer: **Historical/convention - could've been different**

```
MX 10 → mail1.example.com  (TRY FIRST)
MX 20 → mail2.example.com  (try if 10 fails)
MX 30 → mail3.example.com  (try if 10,20 fail)
```

Lower number = higher priority (opposite of most metrics)

Could it be different?
- Yes, RFC 1035 just uses the number to order
- Tradition stuck with lower=better
- Mail servers follow this convention worldwide

---

**8. If domain has multiple A records, how does browser choose?**

Answer: **Typically round-robin or random**

```
example.com A 1.2.3.4
example.com A 1.2.3.5
example.com A 1.2.3.6
```

DNS returns all three; client chooses:
- **Round-robin**: First try 1.2.3.4, next time 1.2.3.5, etc.
- **Random**: Pick any randomly
- **OS-dependent**: Windows, Linux, Mac may differ
- **Connection order**: Usually first in list tried first

Why multiple A records?
- Load balancing (distribute traffic)
- Redundancy (if one server down, others work)
- Geographic distribution (serve from closest)

---

**9. What security risks exist with DNS?**

Answer: **Several major threats:**

1. **DNS Spoofing**
   - Attacker intercepts query, sends fake IP
   - Redirects to malicious site
   - Fix: DNSSEC (cryptographic signatures)

2. **DNS Hijacking**
   - Attacker changes nameserver records
   - All traffic goes to attacker
   - Fix: Secure nameserver access

3. **DNS Amplification (DDoS)**
   - Attacker uses DNS servers to flood target
   - Fix: Rate limiting on nameservers

4. **Cache Poisoning**
   - Attacker injects fake records into cache
   - Affects all users of that resolver
   - Fix: DNSSEC, resolver validation

5. **Information Leakage**
   - ISP/resolver sees all domains you visit
   - Fix: DNS over HTTPS (DoH), DNS over TLS (DoT), encrypted DNS

Prevention:
- Use DNSSEC-validating resolvers
- Use encrypted DNS (DoH, DoT)
- Monitor for suspicious records
- Use trusted DNS providers

---

**10. How would you block ads using DNS?**

Answer: **DNS-level blocking:**

Create a DNS server that:
1. List known ad domains (ads.doubleclick.net, google-analytics.com, etc.)
2. Return 127.0.0.1 (localhost) or 0.0.0.0 (null) for ad domains
3. Return normal IP for everything else

Example:
```
ads.doubleclick.net A 127.0.0.1    (blocks, no load)
google.com A 142.250.185.46         (normal)
```

Tools that do this:
- **Pi-hole** - Open-source DNS sinkhole
- **AdGuard Home** - Similar to Pi-hole
- **NextDNS** - Cloud-based, remote blocking
- **Cloudflare Malware** - Built into CloudFlare DNS

Advantages:
- ✓ Blocks ads for entire network
- ✓ Blocks ad tracking at source
- ✓ Works on all devices
- ✓ Very effective

Disadvantages:
- ✗ Some websites don't work if ad domain blocked too aggressively
- ✗ Requires local DNS server
- ✗ Domains constantly change

---

**End of Solutions**

