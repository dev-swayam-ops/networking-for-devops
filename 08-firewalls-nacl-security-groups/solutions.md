# Solutions: Firewalls, NACLs, and Security Groups

Complete solutions with explanations for all exercises.

## Easy Exercise Solutions

### Exercise 1: Understand Firewall Concepts - Solution

**Firewall Concepts Explained:**

| Concept | Definition | Pros | Cons | More Secure |
|---------|-----------|------|------|------------|
| **Stateful** | Tracks connection state, allows related packets | Efficient, flexible | More complex | ✓ Yes |
| **Stateless** | Checks each packet independently | Simple, fast | Can miss related traffic | No |
| **Inbound** | Rules for incoming traffic | Protects from outside attacks | Only one direction | Both needed |
| **Outbound** | Rules for outgoing traffic | Controls data exfiltration | Often overlooked | Both needed |
| **Allow** | Permits matching traffic | Enables services | Can be too permissive | With defaults |
| **Deny** | Blocks matching traffic | Restrictive | Can block legitimate traffic | ✓ Default |
| **Priority** | First matching rule wins | Stops processing | Order matters critically | Yes |
| **Default Deny** | If no rule matches, block | Secure by default | Requires explicit allows | ✓ Yes |
| **Default Allow** | If no rule matches, allow | Simple, permissive | Dangerous in production | No |

**Real-World Examples:**

**Stateful Firewall:**
```
1. Browser makes HTTP request to example.com:80
2. Firewall creates connection state entry
3. Response packets from example.com:80 are recognized as related
4. Response allowed even without explicit inbound rule
```

**Stateless Firewall (NACL):**
```
1. HTTP request: LocalPort > 1024 → RemotePort 80 (ALLOW)
2. Response: RemotePort 80 → LocalPort > 1024 (DENY unless explicit allow)
3. Must create separate rule for response traffic
```

**Rule Order Example:**
```
Rule 1: Allow TCP port 22 from 192.168.1.0/24
Rule 2: Deny TCP port 22 from 192.168.1.0/24

Result: SSH allowed (Rule 1 matches first)
```

vs

```
Rule 1: Deny TCP port 22 from 192.168.1.0/24
Rule 2: Allow TCP port 22 from 192.168.1.0/24

Result: SSH denied (Rule 1 matches first)
```

**Key Insights:**
- Stateful firewalls are easier to manage (Windows Firewall)
- Stateless firewalls require more rules (AWS NACLs)
- Both inbound AND outbound rules provide defense in depth
- Default Deny is more secure but requires explicit allows
- Rule order is critical; first match wins

---

### Exercise 2: Check Windows Firewall Status - Solution

**Commands:**
```powershell
# Get all profiles
Get-NetFirewallProfile

# Get specific details for each profile
Get-NetFirewallProfile -Name Domain | Format-List *
Get-NetFirewallProfile -Name Private | Format-List *
Get-NetFirewallProfile -Name Public | Format-List *

# Quick summary
Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction
```

**Expected Output:**
```
Name    Enabled DefaultInboundAction DefaultOutboundAction
----    ------- -------------------- ---------------------
Domain  True    Block                 Allow
Private True    Block                 Allow
Public  True    Block                 Allow
```

**Explanation:**

| Profile | Usage | Description |
|---------|-------|-------------|
| **Domain** | Corporate networks | Joined to domain, trusted network |
| **Private** | Home/trusted networks | Less strict than public |
| **Public** | Coffee shops, airports | Most restrictive, untrusted |

**Default Policy Meaning:**
```
DefaultInboundAction: Block
→ Incoming connections blocked by default
→ Must explicitly allow inbound traffic

DefaultOutboundAction: Allow
→ Outgoing connections allowed by default
→ Most applications can connect out
```

**Count Rules:**
```powershell
(Get-NetFirewallRule | Measure-Object).Count
```

**Typical Output:** 200-300 rules (varies by Windows version and installed software)

**Key Findings:**
- ✓ Firewall enabled on all profiles (typical)
- ✓ Inbound blocked by default (secure)
- ✓ Outbound allowed by default (usable)
- Windows provides good default security

---

### Exercise 3: List Firewall Rules by Port - Solution

**Commands:**
```powershell
# HTTP rules
Get-NetFirewallRule -DisplayName "*HTTP*" | Select-Object Name, DisplayName, Direction, Action, Enabled

# HTTPS rules
Get-NetFirewallRule -DisplayName "*HTTPS*" | Select-Object Name, DisplayName, Direction, Action, Enabled

# RDP rules
Get-NetFirewallRule -DisplayName "*Remote*" | Select-Object Name, DisplayName, Direction, Action, Enabled

# More detailed
Get-NetFirewallRule -DisplayName "*HTTP*" | ForEach-Object {
    $rule = $_
    $filter = $rule | Get-NetFirewallPortFilter
    Write-Host "Rule: $($rule.DisplayName)"
    Write-Host "  Port: $($filter.LocalPort)"
    Write-Host "  Direction: $($rule.Direction)"
    Write-Host "  Action: $($rule.Action)"
    Write-Host "  Enabled: $($rule.Enabled)"
}
```

**Expected Output:**
```
Name                              DisplayName                  Direction Action Enabled
----                              -----------                  --------- ------ -------
WinRM-HTTP-In-TCP-PUBLIC          Windows Remote Mgmt (HTTP... Inbound   Allow  True
IIS-WebServerRole-HTTP-In-Inbound IIS [HTTP Traffic]           Inbound   Allow  True
WINRM-HTTP-In-TCP                 Windows Remote Management... Inbound   Allow  True
```

**Analysis:**

| Rule | Port | Direction | Action | Enabled | Purpose |
|------|------|-----------|--------|---------|---------|
| WinRM-HTTP | 80 | Inbound | Allow | True | Remote management |
| IIS HTTP | 80 | Inbound | Allow | True | Web traffic |
| HTTPS-In | 443 | Inbound | Allow | True | Secure web |
| RDP | 3389 | Inbound | Allow | True | Remote desktop |

**Key Learning:**
- Multiple rules can use same port (different protocols/profiles)
- Rules show which services are enabled
- Direction shows if inbound or outbound
- Enabled status shows if rule is active

---

### Exercise 4: Test Port Connectivity - Solution

**Commands:**
```powershell
# Test HTTP (port 80)
Test-NetConnection -ComputerName google.com -Port 80 -InformationLevel Detailed

# Test HTTPS (port 443)
Test-NetConnection -ComputerName google.com -Port 443 -InformationLevel Detailed

# Test SSH (port 22) to online server
Test-NetConnection -ComputerName github.com -Port 22 -InformationLevel Detailed

# Test local port
Test-NetConnection -ComputerName localhost -Port 3389 -InformationLevel Detailed
```

**Expected Output (Success):**
```
ComputerName     : google.com
RemotePort       : 443
TcpTestSucceeded : True
PingSucceeded    : True
```

**Expected Output (Failure):**
```
ComputerName     : localhost
RemotePort       : 3389
TcpTestSucceeded : False
PingSucceeded    : True
WARNING: TCP connect to (127.0.0.1:3389) failed
```

**Interpretation:**

| Result | Meaning |
|--------|---------|
| TcpTestSucceeded: True | Port is open, service responding |
| TcpTestSucceeded: False | Port is closed or firewall blocking |
| PingSucceeded: True | Host is reachable |
| PingSucceeded: False | Host unreachable (network/firewall issue) |

**Common Results:**
```
HTTP (80):      Usually True (web traffic allowed)
HTTPS (443):    Usually True (web traffic allowed)
SSH (22):       Often False (not typically exposed)
RDP (3389):     Usually False (Windows-only, local network)
```

---

### Exercise 5: Understand Windows Firewall Rule Creation - Solution

**Key Parameters:**

| Parameter | Required | Type | Description | Example |
|-----------|----------|------|-------------|---------|
| `-Name` | Yes | String | Internal rule name (no spaces) | `AllowHTTP` |
| `-DisplayName` | No | String | User-friendly name | `Allow HTTP Traffic` |
| `-Protocol` | Yes | String | TCP, UDP, ICMPv4, ICMPv6, etc. | `TCP` |
| `-LocalPort` | No | Integer | Local port number | `80` |
| `-RemotePort` | No | Integer | Remote destination port | `443` |
| `-RemoteAddress` | No | String | Remote IP/CIDR/any | `192.168.1.0/24` |
| `-Action` | Yes | String | Allow or Block | `Allow` |
| `-Direction` | Yes | String | Inbound or Outbound | `Inbound` |
| `-Profile` | No | String | Domain, Private, Public, Any | `Private` |
| `-Enabled` | No | Boolean | True or False | `$true` |

**Required Parameters (must specify):**
- `-Name` - Internal identifier
- `-Protocol` - What protocol to filter
- `-Action` - Allow or Block
- `-Direction` - Inbound or Outbound

**Optional Parameters (usually set):**
- `-DisplayName` - Shows in GUI
- `-LocalPort` or `-RemotePort` - Which ports
- `-Profile` - Which network type
- `-RemoteAddress` - Restrict to specific IPs

**Protocol Values:**
```
TCP     - Transmission Control Protocol (web, SSH, etc.)
UDP     - User Datagram Protocol (DNS, streaming)
ICMPv4  - Internet Control Message Protocol v4 (ping)
ICMPv6  - Internet Control Message Protocol v6 (IPv6 ping)
```

**Direction Values:**
```
Inbound  - Traffic coming into the system
Outbound - Traffic going out from the system
```

**Profile Values:**
```
Domain   - Computer connected to domain
Private  - Private/home network
Public   - Public network (untrusted)
Any      - All profiles
```

---

## Medium Exercise Solutions

### Exercise 6: Create and Test HTTP Firewall Rule - Solution

**Create HTTP Rule:**
```powershell
# Create the rule
New-NetFirewallRule `
  -Name "AllowHTTP" `
  -DisplayName "Allow HTTP Traffic" `
  -Protocol TCP `
  -LocalPort 80 `
  -Action Allow `
  -Direction Inbound `
  -Profile Private

# Output indicates success
Name                    : AllowHTTP
DisplayName            : Allow HTTP Traffic
Enabled                : True
Action                 : Allow
Direction              : Inbound
```

**Verify Rule Created:**
```powershell
# List the rule
Get-NetFirewallRule -DisplayName "Allow HTTP Traffic" | Select-Object Name, DisplayName, Action, Direction, Enabled

# Get port filter details
$rule = Get-NetFirewallRule -DisplayName "Allow HTTP Traffic"
$rule | Get-NetFirewallPortFilter | Select-Object Protocol, LocalPort
```

**Expected Verification Output:**
```
Name       DisplayName            Action Direction Enabled
----       -----------            ------ --------- -------
AllowHTTP  Allow HTTP Traffic     Allow  Inbound   True

Protocol LocalPort
-------- ---------
TCP      80
```

**Test Connectivity (if web server running on port 80):**
```powershell
# Test local connection
Test-NetConnection -ComputerName localhost -Port 80 -InformationLevel Detailed

# Test remote connection
curl http://localhost/
```

**Expected Test Output:**
```
ComputerName     : localhost
RemotePort       : 80
TcpTestSucceeded : True
```

**Cleanup:**
```powershell
# Remove rule
Remove-NetFirewallRule -DisplayName "Allow HTTP Traffic"

# Verify removed
Get-NetFirewallRule -DisplayName "Allow HTTP Traffic" -ErrorAction SilentlyContinue
# No output = rule removed
```

**Key Learning:**
- `-Name` is internal identifier (no spaces)
- `-DisplayName` is what users see
- `-Profile Private` applies to private networks
- Rule takes effect immediately
- Can be tested with Test-NetConnection before testing actual service

---

### Exercise 7: Create Rule with Source IP Restriction - Solution

**Create IP-Restricted Rule:**
```powershell
# Allow RDP only from specific admin IP
New-NetFirewallRule `
  -Name "RDPFromAdmin" `
  -DisplayName "Allow RDP from Admin IP" `
  -Protocol TCP `
  -LocalPort 3389 `
  -RemoteAddress 192.168.1.100 `
  -Action Allow `
  -Direction Inbound `
  -Profile Any

# Verify rule
Get-NetFirewallRule -DisplayName "Allow RDP from Admin IP" | Select-Object *
```

**Verify Source IP Filter:**
```powershell
# Get address filter
Get-NetFirewallRule -DisplayName "Allow RDP from Admin IP" | Get-NetFirewallAddressFilter

# Expected output
RemoteAddress          LocalAddress
-------------          ----------
192.168.1.100          Any
```

**Advanced IP Examples:**
```powershell
# Allow from subnet
-RemoteAddress 192.168.1.0/24

# Allow from multiple IPs
-RemoteAddress 192.168.1.100, 192.168.1.101

# Allow from any (same as not specifying)
-RemoteAddress Any
```

**How It Works:**
```
Incoming connection from 192.168.1.100:ANY → LocalPort 3389
↓
Matches rule? RemoteAddress=192.168.1.100, LocalPort=3389
↓
YES → Allow connection

Incoming connection from 192.168.1.50:ANY → LocalPort 3389
↓
Matches rule? RemoteAddress=192.168.1.100, LocalPort=3389
↓
NO → Blocked by default deny
```

**Create MySQL Rule with Subnet:**
```powershell
New-NetFirewallRule `
  -Name "MySQLFromCorp" `
  -DisplayName "Allow MySQL from Corporate Network" `
  -Protocol TCP `
  -LocalPort 3306 `
  -RemoteAddress 192.168.1.0/24 `
  -Action Allow `
  -Direction Inbound

# Verify
Get-NetFirewallRule -DisplayName "Allow MySQL*" | Get-NetFirewallAddressFilter
```

**Key Learning:**
- RemoteAddress restricts who can connect
- Can use individual IPs or CIDR notation
- More secure than allowing any IP
- Essential for database server access

---

### Exercise 8: Create Outbound Rule to Block Traffic - Solution

**Create Outbound Block Rule:**
```powershell
# Block Telnet outbound (port 23)
New-NetFirewallRule `
  -Name "BlockTelnet" `
  -DisplayName "Block Telnet Outbound" `
  -Protocol TCP `
  -RemotePort 23 `
  -Action Block `
  -Direction Outbound `
  -Profile Any

# Verify
Get-NetFirewallRule -DisplayName "Block Telnet*" | Select-Object Name, Action, Direction

# Expected output
Name         Action Direction
----         ------ ---------
BlockTelnet  Block  Outbound
```

**Create Additional SMTP Block Rule:**
```powershell
# Block SMTP outbound (port 25)
New-NetFirewallRule `
  -Name "BlockSMTP" `
  -DisplayName "Block SMTP Outbound" `
  -Protocol TCP `
  -RemotePort 25 `
  -Action Block `
  -Direction Outbound `
  -Profile Any
```

**List All Outbound Rules:**
```powershell
# Show all outbound block rules
Get-NetFirewallRule -Direction Outbound -Action Block | Select-Object DisplayName, Protocol | Format-Table

# Show all outbound allow rules
Get-NetFirewallRule -Direction Outbound -Action Allow | Select-Object DisplayName, Protocol | Format-Table
```

**Why Outbound Rules Matter:**
```
Inbound Rules:  Prevent unauthorized access FROM outside
Outbound Rules: Prevent data theft TO outside

Example threats blocked by outbound rules:
- Malware sending stolen data to attacker
- Spyware exfiltrating personal information
- Compromised app connecting to C&C server
```

**Difference: Inbound vs Outbound**

| Direction | Type | Purpose | Example |
|-----------|------|---------|---------|
| Inbound | Allow port 80 | Accept web requests | Allows web server to work |
| Outbound | Allow port 80 | Send web requests | Allows downloads, Windows Update |
| Outbound | Block port 25 | Prevent email sending | Stops malware sending spam |

---

### Exercise 9: Manage Rule Enable/Disable and Removal - Solution

**Rule Lifecycle Management:**

**Step 1: Create Rule**
```powershell
New-NetFirewallRule `
  -Name "TestRule" `
  -DisplayName "Test Rule for Management" `
  -Protocol TCP `
  -LocalPort 9999 `
  -Action Allow `
  -Direction Inbound

# Verify created and enabled
Get-NetFirewallRule -DisplayName "Test Rule*" | Select-Object Name, Enabled

# Output
Name     Enabled
----     -------
TestRule True
```

**Step 2: Disable Rule (Keep but don't use)**
```powershell
# Disable rule
Disable-NetFirewallRule -DisplayName "Test Rule*"

# Verify disabled
Get-NetFirewallRule -DisplayName "Test Rule*" | Select-Object Name, Enabled

# Output
Name     Enabled
----     -------
TestRule False
```

**Effect of Disabled Rule:**
```
- Rule exists in firewall
- Rule doesn't match any traffic
- Port 9999 acts like no rule exists
- Traffic to port 9999 blocked by default deny
```

**Step 3: Re-Enable Rule**
```powershell
# Enable rule again
Enable-NetFirewallRule -DisplayName "Test Rule*"

# Verify enabled
Get-NetFirewallRule -DisplayName "Test Rule*" | Select-Object Name, Enabled

# Output
Name     Enabled
----     -------
TestRule True
```

**Effect of Re-Enabled Rule:**
```
- Rule becomes active again
- Matches traffic to port 9999
- Port 9999 allow rule in effect
- No need to recreate rule
```

**Step 4: Remove Rule Completely**
```powershell
# Remove rule entirely
Remove-NetFirewallRule -DisplayName "Test Rule*"

# Verify removed
Get-NetFirewallRule -DisplayName "Test Rule*" -ErrorAction SilentlyContinue

# No output = rule removed
```

**Effect of Removed Rule:**
```
- Rule no longer exists
- Port 9999 acts like no rule
- Must recreate if needed again
- No way to "re-enable" deleted rule
```

**Difference: Disable vs Remove**

| Operation | State | Recovery | Use Case |
|-----------|-------|----------|----------|
| Disable | Exists but inactive | Enable rule | Temporary testing |
| Remove | Deleted completely | Recreate rule | Permanent removal |

**Best Practice:**
- Use **Disable** for troubleshooting (can quickly re-enable)
- Use **Remove** only when sure rule not needed
- Keep multiple rules disabled for quick testing

---

### Exercise 10: Review and Analyze Multiple Rules - Solution

**Export Rules:**
```powershell
# Export enabled rules to CSV
Get-NetFirewallRule -Enabled | Get-NetFirewallRulePortFilter | `
  Select-Object InstanceID, Protocol, LocalPort | `
  Export-Csv rules.csv -NoTypeInformation

# View exported file
Get-Content rules.csv | Select-Object -First 10
```

**Analyze by Action:**
```powershell
# Count allow vs block rules
Get-NetFirewallRule | Group-Object Action | Select-Object Name, Count

# Expected output
Name  Count
----  -----
Allow 280
Block 15
```

**Analyze by Direction:**
```powershell
# Count inbound vs outbound
Get-NetFirewallRule | Group-Object Direction | Select-Object Name, Count

# Expected output
Name     Count
----     -----
Inbound  150
Outbound 145
```

**Find Web Traffic Rules:**
```powershell
# Rules for HTTP/HTTPS
Get-NetFirewallRule | Get-NetFirewallPortFilter | `
  Where-Object {$_.LocalPort -in 80,443} | `
  Select-Object Protocol, LocalPort

# Check which services have rules
Get-NetFirewallRule -DisplayName "*HTTP*" | Select-Object DisplayName
```

**Find Database Rules:**
```powershell
# Rules for MySQL, MSSQL, PostgreSQL
Get-NetFirewallRule | Get-NetFirewallPortFilter | `
  Where-Object {$_.LocalPort -in 3306,1433,5432} | `
  Select-Object Protocol, LocalPort
```

**Find Block Rules:**
```powershell
# All blocking rules
Get-NetFirewallRule -Action Block | `
  Select-Object DisplayName, Direction, Action

# Outbound blocks specifically
Get-NetFirewallRule -Direction Outbound -Action Block | `
  Select-Object DisplayName
```

**Create Summary Report:**
```powershell
# Comprehensive report
$summary = @{
    'Total Rules' = (Get-NetFirewallRule | Measure-Object).Count
    'Allow Rules' = (Get-NetFirewallRule -Action Allow | Measure-Object).Count
    'Block Rules' = (Get-NetFirewallRule -Action Block | Measure-Object).Count
    'Inbound' = (Get-NetFirewallRule -Direction Inbound | Measure-Object).Count
    'Outbound' = (Get-NetFirewallRule -Direction Outbound | Measure-Object).Count
    'Enabled' = (Get-NetFirewallRule -Enabled | Measure-Object).Count
    'Disabled' = (Get-NetFirewallRule -Disabled | Measure-Object).Count
}

$summary | Format-Table

# Expected output
Name           Value
----           -----
Total Rules    295
Allow Rules    280
Block Rules    15
Inbound        150
Outbound       145
Enabled        290
Disabled       5
```

**Key Insights:**
- Most rules are Allow (typical configuration)
- Few Block rules (only for specific threats)
- Roughly equal inbound/outbound
- Most rules enabled, some disabled for testing

---

## Challenge Exercise Solution

### Create Complete Firewall Profile for Web Server - Solution

**Complete Rule Set:**

**1. Allow Inbound HTTP:**
```powershell
New-NetFirewallRule `
  -Name "WebServerHTTP" `
  -DisplayName "Web Server - Allow HTTP" `
  -Protocol TCP `
  -LocalPort 80 `
  -Action Allow `
  -Direction Inbound `
  -Profile Any
```

**2. Allow Inbound HTTPS:**
```powershell
New-NetFirewallRule `
  -Name "WebServerHTTPS" `
  -DisplayName "Web Server - Allow HTTPS" `
  -Protocol TCP `
  -LocalPort 443 `
  -Action Allow `
  -Direction Inbound `
  -Profile Any
```

**3. Allow Inbound SSH (Admin Only):**
```powershell
New-NetFirewallRule `
  -Name "WebServerSSH" `
  -DisplayName "Web Server - Allow SSH from Admin" `
  -Protocol TCP `
  -LocalPort 22 `
  -RemoteAddress 203.0.113.1 `
  -Action Allow `
  -Direction Inbound `
  -Profile Any
```

**4. Allow Outbound DNS:**
```powershell
New-NetFirewallRule `
  -Name "WebServerDNS" `
  -DisplayName "Web Server - Allow DNS Outbound" `
  -Protocol UDP `
  -RemotePort 53 `
  -Action Allow `
  -Direction Outbound `
  -Profile Any
```

**5. Allow Outbound HTTP/HTTPS:**
```powershell
New-NetFirewallRule `
  -Name "WebServerHTTPOut" `
  -DisplayName "Web Server - Allow HTTP Outbound" `
  -Protocol TCP `
  -RemotePort 80 `
  -Action Allow `
  -Direction Outbound `
  -Profile Any

New-NetFirewallRule `
  -Name "WebServerHTTPSOut" `
  -DisplayName "Web Server - Allow HTTPS Outbound" `
  -Protocol TCP `
  -RemotePort 443 `
  -Action Allow `
  -Direction Outbound `
  -Profile Any
```

**6. Block Other Inbound (Implicit, but can be explicit):**
```powershell
# Windows already blocks by default, but can add explicit rule
# (usually not needed since default deny is in place)
```

**Verify All Rules:**
```powershell
# List all created rules
Get-NetFirewallRule -DisplayName "Web Server*" | `
  Select-Object DisplayName, Direction, Action, Protocol | `
  Format-Table

# Expected output
DisplayName                             Direction Action Protocol
-----------                             --------- ------ --------
Web Server - Allow HTTP                 Inbound   Allow  TCP
Web Server - Allow HTTPS                Inbound   Allow  TCP
Web Server - Allow SSH from Admin       Inbound   Allow  TCP
Web Server - Allow DNS Outbound         Outbound  Allow  UDP
Web Server - Allow HTTP Outbound        Outbound  Allow  TCP
Web Server - Allow HTTPS Outbound       Outbound  Allow  TCP
```

**Test Allowed Traffic:**
```powershell
# Test HTTP
Test-NetConnection -ComputerName localhost -Port 80

# Test HTTPS
Test-NetConnection -ComputerName localhost -Port 443

# Test SSH from admin IP (if admin IP is localhost)
Test-NetConnection -ComputerName localhost -Port 22

# Test DNS outbound
nslookup google.com
```

**Test Blocked Traffic:**
```powershell
# Test SSH from non-admin IP (should fail)
Test-NetConnection -ComputerName localhost -Port 22 -RemoteAddress 192.168.1.50

# Test SMTP outbound (not allowed)
Test-NetConnection -ComputerName mail.example.com -Port 25

# Test RDP (not allowed)
Test-NetConnection -ComputerName localhost -Port 3389
```

**Documentation:**

| Port | Protocol | Direction | Source | Action | Purpose | Status |
|------|----------|-----------|--------|--------|---------|--------|
| 80 | TCP | Inbound | Any | Allow | Web Traffic | ✓ Working |
| 443 | TCP | Inbound | Any | Allow | Secure Web | ✓ Working |
| 22 | TCP | Inbound | 203.0.113.1 | Allow | SSH Admin | ✓ Restricted |
| 53 | UDP | Outbound | Any | Allow | DNS | ✓ Working |
| 80 | TCP | Outbound | Any | Allow | HTTP Updates | ✓ Working |
| 443 | TCP | Outbound | Any | Allow | HTTPS Updates | ✓ Working |
| Other | * | Inbound | Any | Deny | Default Deny | ✓ Blocked |
| Other | * | Outbound | Any | Deny | Default Deny | ✓ Blocked |

**Key Achievements:**
- Web server accepts HTTP/HTTPS from anyone (public)
- Admin can SSH from specific IP only
- Server can resolve DNS for updates
- Server can download updates (HTTP/HTTPS)
- All other traffic blocked by default
- Defense in depth: only needed services open

