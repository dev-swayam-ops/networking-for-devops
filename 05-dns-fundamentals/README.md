# 05-dns-fundamentals

## What You'll Learn

By the end of this module, you will:

1. **Understand DNS basics** - What DNS is and why it matters
2. **Master DNS resolution** - How domain names become IP addresses
3. **Work with DNS records** - A records, AAAA, CNAME, MX, NS, SOA
4. **Use DNS tools** - nslookup, dig, host, ipconfig for DNS queries
5. **Troubleshoot DNS** - Find and fix DNS problems
6. **Configure DNS clients** - Set DNS servers on Windows and Linux
7. **Understand DNS security** - DNSSEC basics and DNS spoofing prevention

---

## Prerequisites

Before starting this module, you should be comfortable with:

- **Module 01: Networking Fundamentals** - Understanding network basics and protocols
- **TCP/UDP basics** - DNS uses port 53 (covered in Module 01)
- **Basic domain names** - How websites are named (google.com, example.org)
- **Command-line tools** - Running commands in PowerShell or terminal

If you need a refresher, revisit Module 01.

---

## Key Concepts

### What is DNS?

**DNS (Domain Name System)** translates human-friendly domain names to IP addresses.

```
You type: google.com
DNS resolves to: 142.250.185.46 (and others)
Your browser connects to: 142.250.185.46
```

**Why DNS matters:**
- ✓ Easy to remember (google.com vs 142.250.185.46)
- ✓ Servers can change IPs without breaking websites
- ✓ Load balancing (one domain → multiple IPs)
- ✓ Email routing (MX records)

### DNS Resolution Process (5-Step)

The journey of a DNS query:

```
1. USER QUERY
   You type: www.google.com
   Browser asks: "What's the IP for www.google.com?"

2. RECURSIVE RESOLVER
   Your ISP's DNS (usually 8.8.8.8 or similar)
   "I'll find that for you"

3. ROOT NAMESERVER
   Resolver asks: "Who handles .com domains?"
   Root responds: "Ask the .com nameserver"

4. AUTHORITATIVE NAMESERVER
   Resolver asks .com nameserver: "Who handles google.com?"
   Response: "Google's nameserver at 216.239.32.10"

5. RESPONSE BACK
   Google's nameserver: "www.google.com = 142.250.185.46"
   Back through resolver → back to your browser
   Browser connects to 142.250.185.46
```

**Illustration:**

```
┌─────────────┐
│   Browser   │  (1) "What's www.google.com?"
└─────┬───────┘
      │
      ├──→ (2) Your ISP's Resolver (8.8.8.8)
      │
      ├──→ (3) Root Nameserver (13 worldwide)
      │        "For .com, ask: 192.5.6.30"
      │
      ├──→ (4) .COM TLD Server (192.5.6.30)
      │        "For google.com, ask: 216.239.32.10"
      │
      ├──→ (5) Google's Nameserver (216.239.32.10)
      │        "www.google.com = 142.250.185.46"
      │
      └←─ Back with answer: 142.250.185.46
```

### DNS Record Types

**Most common records:**

| Record | Type | Purpose | Example |
|--------|------|---------|---------|
| A | Address | IPv4 → IP | google.com → 142.250.185.46 |
| AAAA | Address | IPv6 → IP | google.com → 2607:f8b0:4004:809::200e |
| CNAME | Canonical Name | Alias → domain | www → google.com |
| MX | Mail Exchange | Email server | mail.google.com |
| NS | Nameserver | Who manages this domain | ns1.google.com |
| SOA | Start of Authority | Zone info | Serial, refresh, retry |
| TXT | Text | Verification, SPF, DKIM | v=spf1 include:... |
| SRV | Service | Service location | _service._proto.name |

**Example A Record:**
```
Domain: google.com
Record: A
Value: 142.250.185.46
TTL: 300 (seconds)
Meaning: "google.com's IPv4 is 142.250.185.46"
```

### DNS Caching and TTL

**TTL (Time To Live)** = How long to remember the answer

```
TTL = 300 means: "Cache this for 300 seconds"
TTL = 3600 means: "Cache this for 1 hour"
TTL = 86400 means: "Cache this for 1 day"

Caching levels:
1. Browser cache (2 minutes typical)
2. OS cache (Windows: registry, Linux: systemd-resolved)
3. ISP resolver cache (24+ hours)
4. Nameserver cache (per TTL value)
```

**When to use different TTLs:**

| TTL | Scenario | Example |
|-----|----------|---------|
| 300 | Frequently changing | Testing, deployments |
| 3600 | Normal changes | Server updates |
| 86400 | Stable records | Primary website IP |
| 604800 | Very stable | Secondary DNS |

### DNS Nameservers

**What they are:**
- Servers that answer DNS questions
- Authoritative (owns the records) or recursive (asks others)

**Example:**
```
Domain: google.com

Nameservers:
  ns1.google.com (216.239.32.10)
  ns2.google.com (216.239.34.10)
  ns3.google.com (216.239.36.10)
  ns4.google.com (216.239.38.10)

All 4 have same records (redundancy)
If one is down, others still work
```

### DNS Query Types

**Recursive query:**
"Give me the answer (or find it for me)"
- Resolver must fully resolve
- Used by clients asking ISP DNS

**Iterative query:**
"Tell me what you know (or where to ask)"
- Server gives best known answer
- Used between nameservers

---

## Hands-on Lab: DNS Queries and Records

**Objective:** Query DNS records using built-in tools

**Time:** 15-20 minutes

### Step 1: Check Your DNS Servers

**Windows:**
```powershell
ipconfig /all | findstr "DNS"
```

**Expected Output:**
```
DNS Servers . . . . . . . . . . : 8.8.8.8
                                  8.8.4.4
```

**Linux/Mac:**
```bash
cat /etc/resolv.conf
```

**What it shows:** Which DNS servers your system uses

---

### Step 2: Query A Record (IPv4 Address)

**Task:** Find the IPv4 address for google.com

**Windows (nslookup):**
```powershell
nslookup google.com
```

**Expected Output:**
```
Server:  [Your DNS Server]
Address: 8.8.8.8

Non-authoritative answer:
Name:    google.com
Address: 142.250.185.46
```

**Linux (dig):**
```bash
dig google.com A
```

**Expected Output:**
```
; <<>> DiG 9.16.1-Ubuntu
; <<>> google.com A

; ANSWER SECTION:
google.com.             300     IN      A       142.250.185.46
```

**What it shows:** The IPv4 address for google.com (usually returns multiple IPs)

---

### Step 3: Query AAAA Record (IPv6 Address)

**Task:** Find the IPv6 address for google.com

**Windows:**
```powershell
nslookup -type=AAAA google.com
```

**Expected Output:**
```
Name:    google.com
Address: 2607:f8b0:4004:809::200e
```

**Linux:**
```bash
dig google.com AAAA
```

**What it shows:** The IPv6 address (modern internet protocol)

---

### Step 4: Query MX Record (Mail Server)

**Task:** Find email servers for a domain

**Windows:**
```powershell
nslookup -type=MX google.com
```

**Expected Output:**
```
google.com  MX preference = 10, mail exchanger = smtp.google.com
google.com  MX preference = 20, mail exchanger = aspmx.l.google.com
```

**Linux:**
```bash
dig google.com MX
```

**What it shows:** Email servers (priority 10 = preferred, 20 = fallback)

---

### Step 5: Query NS Record (Nameservers)

**Task:** Find who manages the domain

**Windows:**
```powershell
nslookup -type=NS google.com
```

**Expected Output:**
```
google.com  nameserver = ns1.google.com
google.com  nameserver = ns2.google.com
google.com  nameserver = ns3.google.com
google.com  nameserver = ns4.google.com
```

**Linux:**
```bash
dig google.com NS
```

**What it shows:** The authoritative nameservers for the domain

---

### Step 6: Query Specific Nameserver

**Task:** Ask a specific nameserver for records

**Windows:**
```powershell
nslookup google.com ns1.google.com
```

**Expected Output:**
```
Server:  ns1.google.com
Address: 216.239.32.10

Name:    google.com
Address: 142.250.185.46
```

**Linux:**
```bash
dig @ns1.google.com google.com A
```

**What it shows:** Direct query to authoritative nameserver (more authoritative)

---

### Step 7: Reverse DNS Lookup

**Task:** Find the domain for an IP address

**Windows:**
```powershell
nslookup 142.250.185.46
```

**Expected Output:**
```
Name:    fra16s12-in-f14.1e100.net
Address: 142.250.185.46
```

**Linux:**
```bash
dig -x 142.250.185.46
```

**What it shows:** Reverse lookup (IP → domain name)

---

### Step 8: Check DNS Propagation

**Task:** Verify a domain has correct records globally

**Windows:**
```powershell
nslookup example.com 1.1.1.1
nslookup example.com 8.8.8.8
```

**Compare results from different resolvers**

**What it shows:** If records are the same across DNS servers (propagation)

---

### Step 9: Clear DNS Cache

**Windows:**
```powershell
ipconfig /flushdns
```

**Expected Output:**
```
Successfully flushed the DNS Resolver Cache.
```

**Linux:**
```bash
sudo systemctl restart systemd-resolved
```

**What it shows:** Forcing fresh DNS queries (useful after DNS changes)

---

### Step 10: Verify DNS Tools

**Task:** Understand different DNS query tools

**Windows (all these work):**
```powershell
nslookup google.com
Resolve-DnsName google.com
Get-DnsClientCache
```

**Linux (common tools):**
```bash
dig google.com
host google.com
nslookup google.com
```

**What it shows:** Different tools give same results, choose what you prefer

---

## Validation Checklist

After completing the lab, verify you can:

- ✓ Run `nslookup` and understand the output
- ✓ Query different record types (A, AAAA, MX, NS)
- ✓ Explain what each record type does
- ✓ Query a specific nameserver
- ✓ Perform reverse DNS lookup
- ✓ Understand DNS caching and TTL
- ✓ Check your system's current DNS servers
- ✓ Explain the 5-step DNS resolution process
- ✓ Know when DNS queries fail and why
- ✓ Identify DNS server issues vs. other network problems

---

## Cleanup

**Restore DNS if you changed it:**

```powershell
# Restore to DHCP-provided DNS
Set-DNSClientServerAddress -InterfaceAlias "Ethernet" -ResetServerAddresses

# Or use GUI Settings → Network → Reset
```

**Note:** Most labs in this module don't modify your system, so cleanup isn't necessary.

---

## Common Mistakes

### Mistake 1: Confusing A vs AAAA Records

| Record | Protocol | Example | Remember |
|--------|----------|---------|----------|
| A | IPv4 | 142.250.185.46 | A = basic Address |
| AAAA | IPv6 | 2607:f8b0:4004:809::200e | AAAA = 4x longer format |

**Fix:** A record = IPv4, AAAA = IPv6. Think: AAAA has 4 A's, IPv6 is newer/longer.

---

### Mistake 2: Not Understanding TTL

```
Problem: Changed DNS record, but old one still shows

Reason: TTL cache (waiting for expiration)
Example: TTL=86400, changed 1 hour ago
→ Need to wait 86399 more seconds

Fix: Check TTL value before making changes
Lower TTL (300-3600) before big changes
Wait for TTL to expire before assuming failure
```

---

### Mistake 3: Querying Wrong Nameserver

```
Problem: nslookup shows different result than web shows

Reason: Querying recursive resolver, not authoritative
Recursive: Might be cached, might be old
Authoritative: Direct from domain owner

Solution: Query NS record first, then ask that nameserver
```

---

### Mistake 4: Forgetting DNS Caching

```
Problem: DNS change not taking effect

Causes:
1. Browser cache (clear cache in browser)
2. OS cache (ipconfig /flushdns on Windows)
3. ISP resolver cache (wait up to 24 hours)
4. TTL not expired (check TTL, might need to wait)

Solution: Clear caches, wait for TTL, test with different resolver
```

---

### Mistake 5: Confusing MX Priority

```
Problem: Email not going to preferred server

Example:
MX 10 → mail1.example.com (preferred)
MX 20 → mail2.example.com (backup)

Mistake: Higher number = higher priority
Reality: LOWER number = HIGHER priority

Email servers try 10 first, then 20
```

---

## Troubleshooting

### Problem: DNS Query Returns No Answer

**Symptoms:**
```
nslookup google.com
Server:  8.8.8.8
*** Can't find google.com: No answer
```

**Solutions:**

| Cause | Check | Fix |
|-------|-------|-----|
| DNS server down | `ping 8.8.8.8` | Use different DNS (8.8.4.4, 1.1.1.1) |
| Network down | `ping 8.8.8.8` | Check network connection |
| Domain doesn't exist | Check spelling | google.com (not googl.com) |
| Firewall blocking | Check firewall | Allow port 53 (UDP) outbound |
| Wrong nameserver | Query NS record | Try different nameserver |

---

### Problem: DNS Resolution Very Slow

**Symptoms:**
```
Takes 10+ seconds for DNS query to respond
```

**Solutions:**

| Cause | Check | Fix |
|-------|-------|-----|
| DNS server slow | Try 8.8.8.8 vs 1.1.1.1 | Switch to faster DNS |
| Network latency | `ping 8.8.8.8` | Check network path |
| DNS overloaded | Try off-peak | Wait or switch DNS |
| Complex lookup | Run `tracert` | May be legitimate |

---

### Problem: Inconsistent DNS Results

**Symptoms:**
```
nslookup google.com 8.8.8.8     → 142.250.185.46
nslookup google.com 1.1.1.1     → 142.250.185.30
```

**Solutions:**

| Cause | Solution |
|-------|----------|
| Round-robin (normal) | Google uses many servers - this is normal |
| Cache out of sync | Wait for TTL to expire |
| Different resolvers | Query authoritative NS for consistency |
| Geo-routing | DNS based on location - also normal |

**Fix:** Query the authoritative nameserver for most accurate answer:
```powershell
nslookup google.com ns1.google.com
```

---

### Problem: Website Works by IP, Not by Domain

**Symptoms:**
```
ping 142.250.185.46 → Success
ping google.com → Fails
nslookup google.com → Fails
```

**Causes & Solutions:**

| Cause | Solution |
|-------|----------|
| DNS not working | Port 53 blocked? DNS server down? |
| Firewall blocks port 53 | Allow port 53 (UDP) outbound |
| DNS server not responding | Change DNS server |
| Network down (DNS layer) | Restart network interface |

**Debug steps:**
```powershell
# Test DNS connectivity
nslookup 8.8.8.8

# Try different DNS servers
nslookup google.com 1.1.1.1
nslookup google.com 8.8.8.8

# Check your DNS settings
ipconfig /all | findstr "DNS"
```

---

## Next Steps

**Ready to move deeper?**

1. **Advanced DNS Concepts**
   - DNSSEC (DNS security)
   - DNS load balancing
   - GeoDNS routing

2. **DNS Server Administration**
   - Setting up Windows DNS or bind9
   - Creating zones and records
   - Managing DNS infrastructure

3. **Network Integration**
   - DNS + VPNs
   - DNS + DHCP integration
   - DNS in Kubernetes

4. **Real-world Projects**
   - Multi-site DNS setup
   - DNS failover configuration
   - DNS monitoring and alerting

---

## Resources

**Learning Resources:**
- [RFC 1035](https://tools.ietf.org/html/rfc1035) - DNS Specification (technical)
- [Google DNS Guide](https://developers.google.com/speed/public-dns) - Free public DNS
- [CloudFlare DNS](https://1.1.1.1/) - Free DNS with privacy focus

**Tools Reference:**
- **nslookup** - Built-in DNS query tool (Windows/Linux)
- **dig** - Advanced DNS query tool (Linux/Mac)
- **host** - Simple DNS lookup tool
- **Resolve-DnsName** - PowerShell DNS queries

**Free Public DNS Servers:**
- Google: 8.8.8.8 / 8.8.4.4
- CloudFlare: 1.1.1.1 / 1.0.0.1
- Quad9: 9.9.9.9 / 149.112.112.112

---

**End of Module 05: DNS Fundamentals**

Ready to master DNS? Complete the exercises below!
