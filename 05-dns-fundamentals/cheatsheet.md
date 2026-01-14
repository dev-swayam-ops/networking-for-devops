# Cheatsheet: DNS Fundamentals

Quick reference guide for DNS tools, commands, and concepts.

---

## DNS Record Types Quick Reference

| Record | Type | Purpose | Example |
|--------|------|---------|---------|
| A | Address | IPv4 address | google.com → 142.250.185.46 |
| AAAA | Address | IPv6 address | google.com → 2607:f8b0:4004:809::200e |
| CNAME | Canonical Name | Domain alias | www.example.com → example.com |
| MX | Mail Exchange | Email server | mail.example.com (priority 10) |
| NS | Nameserver | Domain authority | ns1.google.com |
| SOA | Start of Authority | Zone file info | Serial, refresh times |
| TXT | Text | Verification/SPF | v=spf1 include:google.com |
| SRV | Service | Service location | _service._proto.domain |
| PTR | Pointer | Reverse DNS | 46.185.250.142.in-addr.arpa |
| CAA | Certification Authority | SSL/TLS cert | issuewild: letsencrypt.org |

---

## DNS Query Tools and Commands

### Windows Commands

**Query A Record (IPv4):**
```powershell
nslookup google.com
Resolve-DnsName google.com
```

**Query Specific Record Type:**
```powershell
nslookup -type=A google.com           # A record (IPv4)
nslookup -type=AAAA google.com        # AAAA record (IPv6)
nslookup -type=MX google.com          # MX record (email)
nslookup -type=NS google.com          # NS record (nameserver)
nslookup -type=CNAME www.google.com   # CNAME record (alias)
nslookup -type=TXT google.com         # TXT record
nslookup -type=SOA google.com         # SOA record
```

**Reverse DNS (IP to Domain):**
```powershell
nslookup 142.250.185.46
Resolve-DnsName -Name 142.250.185.46 -Type PTR
```

**Query Specific Nameserver:**
```powershell
nslookup google.com ns1.google.com    # Query ns1.google.com
nslookup google.com 8.8.8.8           # Query Google DNS
nslookup google.com 1.1.1.1           # Query CloudFlare DNS
```

**DNS Caching:**
```powershell
ipconfig /all | findstr "DNS"         # Show DNS settings
Get-DnsClientCache                    # Show DNS cache
Clear-DnsClientCache                  # Clear cache (or ipconfig /flushdns)
ipconfig /flushdns                    # Alternative: flush cache
```

**Advanced PowerShell:**
```powershell
Resolve-DnsName google.com -Server 8.8.8.8
Get-DnsClientNrptPolicy               # Show NRPT policies
Set-DNSClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses ("8.8.8.8","8.8.4.4")
```

---

### Linux Commands

**Query A Record:**
```bash
nslookup google.com
dig google.com
host google.com
```

**Query Specific Record Type:**
```bash
dig google.com A                # A record
dig google.com AAAA             # AAAA record
dig google.com MX               # MX record
dig google.com NS               # NS record
dig google.com CNAME            # CNAME record
dig google.com TXT              # TXT record
dig google.com SOA              # SOA record
dig google.com +short           # Short format
```

**Reverse DNS:**
```bash
dig -x 142.250.185.46
nslookup 142.250.185.46
host 142.250.185.46
```

**Query Specific Nameserver:**
```bash
dig @ns1.google.com google.com A
nslookup google.com ns1.google.com
```

**DNS Cache and Configuration:**
```bash
cat /etc/resolv.conf                 # Show DNS servers
systemctl status systemd-resolved     # Check DNS service
sudo systemctl restart systemd-resolved  # Restart DNS
sudo resolvectl flush-caches         # Clear DNS cache (systemd)
sudo systemctl restart dnsmasq       # If using dnsmasq
```

**Trace DNS Lookups:**
```bash
dig +trace google.com                # Show full resolution path
dig +recurse google.com              # Force recursive query
dig +norecurse google.com            # No recursion (iterative)
```

---

## DNS Resolution Process

**5-Step Resolution (Simplified):**

```
1. Browser  → "What's google.com?"
2. ISP Resolver (8.8.8.8) → "I'll find it"
3. Root Nameserver → "Ask TLD server for .com"
4. TLD Server → "Ask Google's nameserver"
5. Google's Nameserver → "It's 142.250.185.46"
```

**Steps:**

| Step | Query Type | Server | Response |
|------|-----------|--------|----------|
| 1 | Recursive | Resolver | "I'll find it for you" |
| 2 | Iterative | Root NS | "Ask 192.5.6.30 for .com" |
| 3 | Iterative | TLD NS | "Ask 216.239.32.10 for google.com" |
| 4 | Iterative | Auth NS | "142.250.185.46" |
| 5 | Response | Resolver | "Here's the answer" |

---

## DNS Caching

### TTL (Time To Live) Reference

| TTL | Duration | Use Case |
|-----|----------|----------|
| 60 | 1 minute | Testing, very frequent changes |
| 300 | 5 minutes | Before DNS migration |
| 900 | 15 minutes | Testing with moderate duration |
| 1800 | 30 minutes | Frequent updates expected |
| 3600 | 1 hour | Standard, normal changes |
| 7200 | 2 hours | Stable services |
| 86400 | 1 day | Very stable, rarely change |
| 604800 | 1 week | Extremely stable |

### Cache Levels

```
Browser Cache (2 min typical)
    ↓
OS Cache (Windows DNS, Linux systemd-resolved)
    ↓
ISP Resolver Cache (24+ hours)
    ↓
Authoritative Nameserver Cache (per TTL)
```

**Clear Caches:**

Windows:
```powershell
ipconfig /flushdns
```

Linux:
```bash
sudo systemctl restart systemd-resolved
```

Browser: Ctrl+Shift+Delete → Clear Cookies and Cache

---

## Public DNS Servers

| Provider | Primary | Secondary | Notes |
|----------|---------|-----------|-------|
| Google | 8.8.8.8 | 8.8.4.4 | Fast, reliable |
| CloudFlare | 1.1.1.1 | 1.0.0.1 | Privacy-focused |
| Quad9 | 9.9.9.9 | 149.112.112.112 | Security-focused, malware filtering |
| OpenDNS | 208.67.222.222 | 208.67.220.220 | Family/content filtering available |
| Level3 | 209.244.0.3 | 209.244.0.4 | Older but reliable |

---

## DNS Configuration

### Change DNS on Windows

**Via PowerShell (requires admin):**
```powershell
# Set custom DNS
Set-DNSClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses ("8.8.8.8","8.8.4.4")

# Reset to DHCP-provided
Set-DNSClientServerAddress -InterfaceAlias "Ethernet" -ResetServerAddresses
```

**Via GUI:**
```
Settings → Network & Internet 
→ Advanced Network Settings
→ Change adapter options
→ Right-click adapter → Properties
→ IPv4 Properties → Preferred DNS
```

### Change DNS on Linux

**Temporary (until reboot):**
```bash
sudo nano /etc/resolv.conf
# Add: nameserver 8.8.8.8
#      nameserver 8.8.4.4
```

**Permanent (systemd-resolved):**
```bash
sudo nano /etc/systemd/resolved.conf
# Uncomment and set:
# DNS=8.8.8.8 8.8.4.4
# FallbackDNS=1.1.1.1

sudo systemctl restart systemd-resolved
```

---

## DNS Troubleshooting Commands

### Test DNS Connectivity

```powershell
# Check if DNS works
nslookup 8.8.8.8

# Try different DNS servers
nslookup google.com 8.8.8.8
nslookup google.com 1.1.1.1

# Check your current DNS
ipconfig /all | findstr "DNS"

# Test with dig (Linux)
dig @8.8.8.8 google.com

# Trace the full resolution
dig +trace google.com
```

### Common Issues and Fixes

**Issue: Slow DNS**
```
Fix: Try different DNS
nslookup google.com 1.1.1.1 (faster?)
```

**Issue: DNS doesn't resolve**
```
Fix 1: Clear cache (ipconfig /flushdns)
Fix 2: Try different DNS (8.8.8.8, 1.1.1.1)
Fix 3: Check firewall allows port 53
Fix 4: Restart network adapter
```

**Issue: Wrong IP returned**
```
Fix 1: Wait for TTL to expire
Fix 2: Query authoritative nameserver
Fix 3: Check DNS propagation globally
```

**Issue: Can't reach by domain, but IP works**
```
Diagnosis: DNS is broken
Fix: Check DNS settings, try public DNS
```

---

## DNS Record Examples

### A Record Example
```
Domain: example.com
Type: A
Value: 93.184.216.34
TTL: 3600
Meaning: "example.com's IPv4 is 93.184.216.34, cache for 1 hour"
```

### MX Record Example
```
Domain: example.com
Type: MX
Priority: 10
Value: mail.example.com
TTL: 3600
Meaning: "Email for example.com goes to mail.example.com (priority 10)"
```

### CNAME Example
```
Domain: www.example.com
Type: CNAME
Value: example.com
TTL: 3600
Meaning: "www.example.com is an alias for example.com"
```

### NS Record Example
```
Domain: example.com
Type: NS
Value: ns1.example.com
TTL: 172800
Meaning: "example.com's nameserver is ns1.example.com"
```

### TXT Record Example (SPF)
```
Domain: example.com
Type: TXT
Value: v=spf1 include:sendgrid.net ~all
TTL: 3600
Meaning: "SPF record for email authentication"
```

---

## Recursive vs Iterative Queries

### Recursive Query

**"Give me the answer (or find it for you)"**

- Client → Resolver: "What's google.com?" 
- Resolver must find answer or fail
- Used by: Browsers, clients asking ISP DNS

```powershell
nslookup google.com
```

### Iterative Query

**"Tell me what you know or where to ask next"**

- Resolver → Root: "Who handles .com?"
- Root: "Ask 192.5.6.30"
- Resolver → TLD: "Who handles google.com?"
- Used between: Nameservers, servers talking to servers

```bash
dig +norecurse google.com
```

---

## Memory Aids

### Record Types (Remember)

| Record | Remember as |
|--------|-------------|
| A | **A**ddress (IPv4) |
| AAAA | **A**ddress (IPv6, 4x longer) |
| MX | **M**ail e**X**change |
| CNAME | **C**anonical **NAME** (alias) |
| NS | **N**ame**S**erver |
| TXT | Te**X**T (verification) |
| SOA | **S**tart **O**f **A**uthority |
| PTR | **P**oin**T**e**R** (reverse) |

### DNS Resolution Order

Remember: **CRANT**
- **C**lient queries Resolver
- **R**esolver asks Root
- **A**sk TLD (authority)
- **N**ameserver (authoritative)
- **T**ransfer answer back

### TTL Values (Quick)

- **300** = 5 min (testing)
- **3600** = 1 hour (standard)
- **86400** = 1 day (stable)
- Remember: Higher = longer cache, lower = faster updates

---

## Online Tools Reference

**Check DNS Globally:**
- whats-dns.com
- dnschecker.org
- mxtoolbox.com

**Reverse DNS:**
- mxtoolbox.com (IP lookup)
- whatismyip.com (your IP + reverse)

**DNS Propagation:**
- dnschecker.org
- whatsmydns.net
- mxtoolbox.com/propagationchecker

**DNS Record Format:**
- nslookup output
- dig output
- official RFCs (RFC 1035)

---

## Common Commands Summary

### Windows (One-liners)

```powershell
# Query A record
nslookup google.com

# Query MX record
nslookup -type=MX google.com

# Clear DNS cache
ipconfig /flushdns

# Show DNS servers
ipconfig /all | findstr "DNS"

# Reverse DNS
nslookup 142.250.185.46

# Query specific nameserver
nslookup google.com ns1.google.com

# Query specific resolver
nslookup google.com 8.8.8.8
```

### Linux (One-liners)

```bash
# Query A record
dig google.com

# Query MX record
dig google.com MX

# Trace resolution path
dig +trace google.com

# Reverse DNS
dig -x 142.250.185.46

# Query specific nameserver
dig @ns1.google.com google.com

# Query specific resolver
dig @8.8.8.8 google.com

# Short answer only
dig google.com +short

# Check DNS servers
cat /etc/resolv.conf
```

---

## Protocol Details

### DNS Port and Protocol

| Aspect | Value |
|--------|-------|
| Port | 53 |
| Protocol | UDP (queries), TCP (zone transfers, large responses) |
| Default timeout | 2-5 seconds per query |
| Typical size | < 512 bytes (UDP), larger with EDNS |

### Query Flow

```
Client :random_port → DNS Server :53 (UDP)
      ← Response comes back to :random_port
```

---

## Common Errors and Meanings

| Error | Cause | Solution |
|-------|-------|----------|
| "Can't find X" | Domain doesn't exist or DNS failure | Check spelling, try different DNS |
| "Server failed" | Nameserver error | Try authoritative NS |
| "Timeout" | No response from DNS | Check network, try different DNS |
| "Format error" | Bad query format | Check syntax of nslookup command |
| "NXDOMAIN" | Non-existent domain | Domain truly doesn't exist |
| "NOTIMP" | Not implemented | Requested feature not supported |
| "REFUSED" | Query refused | Firewall blocking, wrong NS |

---

**End of DNS Cheatsheet**

