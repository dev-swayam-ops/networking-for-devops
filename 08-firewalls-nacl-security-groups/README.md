# 08 - Firewalls, NACLs, and Security Groups

## What You'll Learn

By the end of this module, you'll be able to:
- Understand firewall concepts and types (stateful vs stateless)
- Configure Windows Firewall rules for inbound/outbound traffic
- Manage Linux firewall with iptables and ufw
- Work with Network Access Control Lists (NACLs)
- Configure AWS Security Groups
- Implement layered security with multiple firewall levels
- Test firewall rules with network commands
- Troubleshoot connectivity issues caused by firewalls
- Create and manage inbound and outbound rules
- Understand rule priority and evaluation order

## Prerequisites

- Completion of modules 00-07
- Understanding of TCP/IP, ports, and protocols
- Windows 10/11 with administrative access (or Linux/Mac equivalent)
- Basic network connectivity knowledge
- Ability to create VMs or cloud instances (AWS optional)
- Command-line proficiency

## Key Concepts

### Firewall Overview

1. **Firewall** - Software/hardware that filters network traffic
2. **Stateful Firewall** - Remembers connection state, allows related packets
3. **Stateless Firewall** - Checks each packet independently, no state tracking
4. **Inbound Rules** - Control traffic entering the system
5. **Outbound Rules** - Control traffic leaving the system
6. **Default Policy** - Allow or Deny when no rule matches

### Firewall Rules

1. **Rule Components** - Direction, Protocol, Port, Source/Destination IP, Action
2. **Rule Priority** - First matching rule wins (order matters)
3. **Allow Rules** - Permits traffic matching criteria
4. **Deny Rules** - Blocks traffic matching criteria
5. **Default Deny** - More secure: explicitly allow what's needed

### Firewall Types

1. **Windows Firewall** - Built-in Windows protection
2. **Linux iptables** - Kernel-level firewall
3. **Linux ufw** - User-friendly firewall wrapper
4. **AWS Security Groups** - Cloud-level firewall
5. **Network ACLs (NACLs)** - Subnet-level stateless firewall

### Rule Evaluation

1. **Inbound Rules** - Packets from outside to inside
2. **Outbound Rules** - Packets from inside to outside
3. **First Match Wins** - Rules evaluated top-to-bottom
4. **Implicit Deny** - If no rule matches, default action applies
5. **State Tracking** - Stateful firewalls track connection state

### Security Layers

1. **OS Firewall** - Protects individual machine (Windows Firewall)
2. **Network Firewall** - Protects network segment (iptables)
3. **Cloud Security Group** - Protects cloud resources (AWS)
4. **NACL** - Protects subnet level (AWS subnets)
5. **Defense in Depth** - Multiple layers for security

## Hands-on Lab: Configuring Windows Firewall Rules

### Objective
Create inbound and outbound firewall rules on Windows to control network traffic.

### Prerequisites for Lab
- Windows 10/11 with administrative access
- PowerShell with admin privileges
- Understanding of ports (80, 443, 3306, etc.)

### Step 1: Check Current Firewall Status

```powershell
# Check if Windows Defender Firewall is enabled
Get-NetFirewallProfile | Select-Object Name, Enabled

# Get summary of firewall state
netsh advfirewall show allprofiles
```

**Expected Output:**
```
Name    Enabled
----    -------
Domain  True
Private True
Public  True

Domain Profile Settings:
------Operation-----
State                                  ON
Firewall Policy                        BlockInbound,AllowOutbound
LocalFirewall Policy                   NotConfigured
LogDroppedPackets                      False
LogSuccessfulConnections               False
```

### Step 2: List Existing Firewall Rules

```powershell
# List all inbound rules
Get-NetFirewallRule -Direction Inbound | Select-Object Name, Enabled, Action | Head -20

# List rules for HTTP
Get-NetFirewallRule -DisplayName "*HTTP*" | Select-Object Name, Enabled, Action, Direction

# Count rules
(Get-NetFirewallRule | Measure-Object).Count
```

**Expected Output:**
```
Name                                Enabled Action Direction
----                                ------- ------ ---------
Windows Remote Management (HTTP-In) True    Allow  Inbound
BITS Peering for Windows Update... True    Allow  Inbound
...

Total rules: 287
```

### Step 3: Create Inbound Rule for HTTP

```powershell
# Create rule to allow inbound HTTP traffic
New-NetFirewallRule `
  -Name "Allow HTTP" `
  -DisplayName "Allow HTTP (Port 80)" `
  -Protocol TCP `
  -LocalPort 80 `
  -Action Allow `
  -Direction Inbound `
  -Profile Any

# Verify rule was created
Get-NetFirewallRule -DisplayName "Allow HTTP*" | Select-Object Name, Enabled, Action
```

**Expected Output:**
```
Name       Enabled Action
----       ------- ------
Allow HTTP True    Allow
```

### Step 4: Create Inbound Rule for HTTPS

```powershell
# Create rule for HTTPS
New-NetFirewallRule `
  -Name "Allow HTTPS" `
  -DisplayName "Allow HTTPS (Port 443)" `
  -Protocol TCP `
  -LocalPort 443 `
  -Action Allow `
  -Direction Inbound `
  -Profile Any

# Verify
Get-NetFirewallRule -DisplayName "Allow HTTPS*"
```

**Expected Output:**
```
Name        Enabled Action Direction
----        ------- ------ ---------
Allow HTTPS True    Allow  Inbound
```

### Step 5: Create Inbound Rule with Source IP Restriction

```powershell
# Allow MySQL from specific subnet
New-NetFirewallRule `
  -Name "Allow MySQL from Corp" `
  -DisplayName "Allow MySQL (3306) from 192.168.1.0/24" `
  -Protocol TCP `
  -LocalPort 3306 `
  -RemoteAddress 192.168.1.0/24 `
  -Action Allow `
  -Direction Inbound `
  -Profile Any

# Verify
Get-NetFirewallRule -DisplayName "*MySQL*" | Select-Object Name, Enabled, Action
```

**Expected Output:**
```
Name                  Enabled Action
----                  ------- ------
Allow MySQL from Corp True    Allow
```

### Step 6: Create Outbound Rule to Block Specific Traffic

```powershell
# Block outbound traffic to port 25 (SMTP) to prevent spam
New-NetFirewallRule `
  -Name "Block SMTP Outbound" `
  -DisplayName "Block SMTP (Port 25) Outbound" `
  -Protocol TCP `
  -RemotePort 25 `
  -Action Block `
  -Direction Outbound `
  -Profile Any

# Verify
Get-NetFirewallRule -DisplayName "*SMTP*" | Select-Object Name, Enabled, Action, Direction
```

**Expected Output:**
```
Name                  Enabled Action Direction
----                  ------- ------ ---------
Block SMTP Outbound   True    Block  Outbound
```

### Step 7: Test Firewall Rules

```powershell
# Test HTTP connectivity (should work after allowing port 80)
Test-NetConnection -ComputerName google.com -Port 80 -InformationLevel Detailed

# Test HTTPS connectivity (should work after allowing port 443)
Test-NetConnection -ComputerName google.com -Port 443 -InformationLevel Detailed

# Test blocked port (SSH from outside should fail)
Test-NetConnection -ComputerName localhost -Port 3389 -InformationLevel Detailed
```

**Expected Output (Allowed):**
```
ComputerName     : google.com
RemotePort       : 80
TcpTestSucceeded : True

ComputerName     : google.com
RemotePort       : 443
TcpTestSucceeded : True
```

**Expected Output (Blocked):**
```
ComputerName     : localhost
RemotePort       : 3389
TcpTestSucceeded : False
WARNING: TCP connect to (127.0.0.1:3389) failed
```

### Step 8: Disable and Remove a Rule

```powershell
# Disable a rule (keep it but don't use it)
Disable-NetFirewallRule -DisplayName "Allow HTTP (Port 80)"

# Verify it's disabled
Get-NetFirewallRule -DisplayName "Allow HTTP*" | Select-Object Name, Enabled

# Remove a rule completely
Remove-NetFirewallRule -DisplayName "Allow HTTP (Port 80)"

# Verify it's gone
Get-NetFirewallRule -DisplayName "Allow HTTP*" -ErrorAction SilentlyContinue
```

**Expected Output:**
```
Name       Enabled
----       -------
Allow HTTP False

# After removal, no output (rule doesn't exist)
```

## Validation

After completing this lab, validate your understanding by:

1. **Rule Creation** - Create HTTP, HTTPS, and custom port rules
2. **Rule Verification** - List and inspect created rules
3. **Connectivity Testing** - Test allowed vs blocked ports
4. **Source Restriction** - Create rules with specific source IPs
5. **Outbound Control** - Block specific outbound traffic
6. **Rule Management** - Enable, disable, and remove rules

## Cleanup

```powershell
# Remove all created rules
Remove-NetFirewallRule -DisplayName "Allow HTTP (Port 80)" -ErrorAction SilentlyContinue
Remove-NetFirewallRule -DisplayName "Allow HTTPS (Port 443)" -ErrorAction SilentlyContinue
Remove-NetFirewallRule -DisplayName "Allow MySQL (3306) from 192.168.1.0/24" -ErrorAction SilentlyContinue
Remove-NetFirewallRule -DisplayName "Block SMTP (Port 25) Outbound" -ErrorAction SilentlyContinue

# Verify cleanup
Get-NetFirewallRule -DisplayName "Allow*" | Select-Object Name
```

## Common Mistakes

1. **Blocking All Inbound Traffic** - Locks out remote access completely
2. **Forgetting Outbound Rules** - Only controlling inbound leaves systems exposed
3. **Too Permissive Rules** - Using 0.0.0.0/0 (any IP) when restricting is better
4. **Rule Order Mistakes** - Putting deny before allow, so allow never matches
5. **Not Testing Rules** - Creating rules without verifying they work
6. **Blocking Essential Services** - Blocking DNS, NTP, or DHCP breaks connectivity
7. **Missing Ephemeral Ports** - Not allowing high-numbered ports for responses
8. **Firewall = Security** - Thinking firewall alone is sufficient security

## Troubleshooting

### "Connection Refused" Error
**Cause:** Firewall rule is blocking traffic or port is closed
**Solution:**
- Check if rule exists: `Get-NetFirewallRule -DisplayName "*HTTP*"`
- Verify rule allows traffic: `Get-NetFirewallRulePortFilter -AssociatedRule (Get-NetFirewallRule -DisplayName "Allow HTTP*")`
- Test specific port: `Test-NetConnection -ComputerName localhost -Port 80`
- Check service is running: `Get-Service | Where-Object {$_.Name -like "*HTTP*"}`

### "Cannot Connect to Remote Computer" Error
**Cause:** Firewall blocking inbound connections
**Solution:**
- List inbound rules: `Get-NetFirewallRule -Direction Inbound | Where-Object {$_.Enabled -eq $true}`
- Check if port is allowed: `Get-NetFirewallRule -DisplayName "*Your Rule*" | Get-NetFirewallPortFilter`
- Verify source IP is not restricted: Check RemoteAddress in rule
- Enable rule if disabled: `Enable-NetFirewallRule -DisplayName "Your Rule"`

### "Outbound Traffic Blocked" Error
**Cause:** Outbound firewall rules are too restrictive
**Solution:**
- Check outbound rules: `Get-NetFirewallRule -Direction Outbound -Action Block`
- Verify destination port is allowed: `Get-NetFirewallRule -Direction Outbound | Get-NetFirewallPortFilter | Where-Object {$_.RemotePort -eq 443}`
- Check if service needs specific protocol: Some apps need TCP, others UDP
- Test with specific destination: `Test-NetConnection -ComputerName example.com -Port 443`

### "Rule Not Taking Effect" Error
**Cause:** Rule is created but not being applied or evaluated
**Solution:**
- Verify firewall is enabled: `Get-NetFirewallProfile | Select-Object Name, Enabled`
- Check rule is enabled: `Get-NetFirewallRule -DisplayName "Your Rule" | Select-Object Enabled`
- Verify rule applies to correct profile (Domain, Private, Public)
- Check rule priority (order matters): `Get-NetFirewallRule | Select-Object Name, DisplayName | Format-Table -AutoSize`
- Restart service: `Restart-Service mpssvc` (Windows Defender Firewall)

### "Firewall Blocking Legitimate Traffic" Error
**Cause:** Rule is too restrictive or blocking necessary traffic
**Solution:**
- Review rule: `Get-NetFirewallRule -DisplayName "Your Rule" | Format-List *`
- Check if it's a deny rule blocking allow rule
- Verify source/destination IP is correct
- Check protocol (TCP vs UDP)
- Allow related traffic (ICMP for ping): `New-NetFirewallRule -Protocol ICMPv4 -IcmpType EchoRequest -Action Allow`

## Next Steps

1. **Module 09** - Learn about Cloud Networking and AWS VPC (Security Groups in AWS)
2. **Module 12** - Study Network Security and Zero Trust (advanced firewall concepts)
3. **Module 13** - Explore Troubleshooting and Debugging (network debugging with firewalls)
4. **Practice:** Configure firewall rules for your application stack
5. **Real-World:** Implement firewall rules in production environments with change management
