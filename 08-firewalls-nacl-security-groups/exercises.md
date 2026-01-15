# Exercises: Firewalls, NACLs, and Security Groups

Complete these exercises to master firewall concepts and configurations. Progress from Easy to Medium difficulty.

## Easy Exercises

### Exercise 1: Understand Firewall Concepts
**Objective:** Learn fundamental firewall terminology and concepts

Research and explain these firewall concepts:
```
Concepts to learn:
1. Stateful vs Stateless Firewall
2. Inbound vs Outbound Rules
3. Allow vs Deny Rules
4. Rule Priority/Order
5. Default Deny vs Default Allow
```

**Task:**
- Define each concept in your own words
- Explain pros and cons of each
- Provide real-world examples
- Create comparison table (3+ attributes per concept)
- Identify which is more secure

**Success Criteria:**
- You can explain all 5 concepts clearly
- You understand advantages and disadvantages
- You can choose appropriate approach for scenarios
- You recognize concept application in firewalls

---

### Exercise 2: Check Windows Firewall Status
**Objective:** Verify current firewall configuration

Check your Windows system's firewall:
```powershell
# Check firewall enabled/disabled
Get-NetFirewallProfile

# Check specific profile
Get-NetFirewallProfile -Name Private | Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction
```

**Task:**
- Check Domain, Private, and Public profiles
- Document each profile's status (enabled/disabled)
- Check default inbound policy
- Check default outbound policy
- Record total number of rules

**Success Criteria:**
- You can retrieve firewall status
- You understand all 3 profiles
- You know each profile's default policies
- You can count total rules on system

---

### Exercise 3: List Firewall Rules by Port
**Objective:** Find and understand existing rules

List firewall rules for common ports:
```powershell
# Rules for HTTP (port 80)
Get-NetFirewallRule -DisplayName "*HTTP*"

# Rules for HTTPS (port 443)
Get-NetFirewallRule -DisplayName "*HTTPS*"

# Rules for Remote Desktop (port 3389)
Get-NetFirewallRule -DisplayName "*Remote*"
```

**Task:**
- List rules for 3 different ports
- Identify direction (inbound/outbound)
- Note action (allow/block)
- Check if rules are enabled
- Determine which profiles apply

**Success Criteria:**
- You can filter rules by name/port
- You understand rule properties
- You identify rule direction and action
- You know which rules are active

---

### Exercise 4: Test Port Connectivity
**Objective:** Use tools to verify port accessibility

Test connectivity to various ports:
```powershell
# Test HTTP
Test-NetConnection -ComputerName google.com -Port 80 -InformationLevel Detailed

# Test HTTPS
Test-NetConnection -ComputerName google.com -Port 443 -InformationLevel Detailed

# Test SSH
Test-NetConnection -ComputerName example.com -Port 22 -InformationLevel Detailed
```

**Task:**
- Test 3 different ports/services
- Document results (success/failure)
- Explain why each succeeded or failed
- Identify which failures are firewall-related
- Test to local and remote hosts

**Success Criteria:**
- You can use Test-NetConnection command
- You interpret results correctly
- You understand success vs failure indicators
- You identify firewall blocking

---

### Exercise 5: Understand Windows Firewall Rule Creation
**Objective:** Learn rule creation syntax and parameters

Study firewall rule creation:
```powershell
# Example rule creation (don't run yet)
New-NetFirewallRule `
  -Name "MyRule" `
  -DisplayName "My Test Rule" `
  -Protocol TCP `
  -LocalPort 8080 `
  -Action Allow `
  -Direction Inbound

# Required vs optional parameters
Get-Help New-NetFirewallRule -Detailed
```

**Task:**
- Review New-NetFirewallRule parameters
- Identify required parameters
- Identify optional parameters
- Create comparison of parameter options
- Document parameter meanings

**Success Criteria:**
- You understand required parameters
- You know optional parameters
- You can explain each parameter's purpose
- You're ready to create rules

---

## Medium Exercises

### Exercise 6: Create and Test HTTP Firewall Rule
**Objective:** Create working firewall rule for HTTP traffic

Create and verify HTTP rule:
```powershell
# Create rule
New-NetFirewallRule `
  -Name "AllowHTTP" `
  -DisplayName "Allow HTTP Traffic" `
  -Protocol TCP `
  -LocalPort 80 `
  -Action Allow `
  -Direction Inbound `
  -Profile Private

# Verify rule created
Get-NetFirewallRule -DisplayName "Allow HTTP Traffic"

# Get port filter details
Get-NetFirewallRule -DisplayName "Allow HTTP Traffic" | Get-NetFirewallPortFilter
```

**Task:**
- Create HTTP rule as specified
- Verify rule appears in firewall list
- Confirm all parameters set correctly
- Start simple web server on port 80 (if available)
- Test connectivity to port 80
- Document test results

**Success Criteria:**
- Rule created successfully
- Rule properties match configuration
- Rule can be listed and inspected
- Connectivity test works (if server running)
- Rule can be deleted/removed cleanly

---

### Exercise 7: Create Rule with Source IP Restriction
**Objective:** Restrict traffic to specific source IP

Create IP-restricted rule:
```powershell
# Allow RDP only from specific IP
New-NetFirewallRule `
  -Name "RDPFromAdmin" `
  -DisplayName "Allow RDP from Admin IP" `
  -Protocol TCP `
  -LocalPort 3389 `
  -RemoteAddress 192.168.1.100 `
  -Action Allow `
  -Direction Inbound `
  -Profile Any
```

**Task:**
- Create rule with RemoteAddress restriction
- Verify rule includes source IP filter
- List rule with all filters: `Get-NetFirewallRule -DisplayName "Allow RDP*" | Get-NetFirewallAddressFilter`
- Test rule specifics (what exactly does it allow/block)
- Create rule for different port with different source
- Document behavior

**Success Criteria:**
- Rule created with source IP restriction
- Rule properties show correct IP range
- You understand how RemoteAddress works
- You can verify filter details
- You know what "Any IP" vs "Specific IP" means

---

### Exercise 8: Create Outbound Rule to Block Traffic
**Objective:** Control outbound traffic

Create blocking rule for outbound traffic:
```powershell
# Block outbound to specific port
New-NetFirewallRule `
  -Name "BlockTelnet" `
  -DisplayName "Block Telnet Outbound" `
  -Protocol TCP `
  -RemotePort 23 `
  -Action Block `
  -Direction Outbound `
  -Profile Any

# Verify it's a block rule
Get-NetFirewallRule -DisplayName "Block Telnet*" | Select-Object Action, Direction
```

**Task:**
- Create outbound block rule for port 23 (Telnet)
- Verify it's set to Block (not Allow)
- Verify Direction is Outbound
- Create additional outbound rule to block port 25 (SMTP)
- List all outbound rules
- Document difference between inbound and outbound

**Success Criteria:**
- Rule created as outbound (not inbound)
- Action is Block (not Allow)
- You understand outbound rule purpose
- You can create multiple block rules
- You can list and identify block rules

---

### Exercise 9: Manage Rule Enable/Disable and Removal
**Objective:** Control rule lifecycle (create, disable, enable, remove)

Manage rules through lifecycle:
```powershell
# Create test rule
New-NetFirewallRule `
  -Name "TestRule" `
  -DisplayName "Test Rule for Management" `
  -Protocol TCP `
  -LocalPort 9999 `
  -Action Allow `
  -Direction Inbound

# Disable rule (keep it but don't use)
Disable-NetFirewallRule -DisplayName "Test Rule*"

# Verify disabled
Get-NetFirewallRule -DisplayName "Test Rule*" | Select-Object Enabled

# Enable rule again
Enable-NetFirewallRule -DisplayName "Test Rule*"

# Remove rule completely
Remove-NetFirewallRule -DisplayName "Test Rule*"
```

**Task:**
- Create a test rule
- Disable it without removing
- Verify it's disabled
- Re-enable it
- Verify it's enabled
- Delete the rule
- Verify it's gone
- Create and remove multiple rules

**Success Criteria:**
- You can create rules
- You can disable rules without deleting
- You can enable disabled rules
- You can permanently remove rules
- You understand difference between disable/remove
- Rules can be recreated if needed

---

### Exercise 10: Review and Analyze Multiple Rules
**Objective:** Understand complex rule configurations

Analyze real firewall rules:
```powershell
# Export all enabled rules to CSV
Get-NetFirewallRule -Enabled | Get-NetFirewallRulePortFilter | Select-Object InstanceID, Protocol, LocalPort | Export-Csv rules.csv

# Group rules by action
Get-NetFirewallRule | Group-Object Action | Select-Object Name, Count

# Find all block rules
Get-NetFirewallRule -Action Block | Select-Object DisplayName, Direction, Protocol

# Rules for multiple ports
Get-NetFirewallRule | Get-NetFirewallPortFilter | Where-Object {$_.LocalPort -in 80,443,3306}
```

**Task:**
- Export all firewall rules
- Analyze rule count by action (Allow vs Block)
- Analyze rule count by direction (Inbound vs Outbound)
- Find rules for web traffic (ports 80, 443)
- Find rules for database traffic (port 3306)
- Identify most restrictive rules
- Create summary report

**Success Criteria:**
- You can export and analyze rules
- You understand distribution of rules
- You can filter rules by multiple criteria
- You can identify specific rule purposes
- You can create rule report
- You understand rule complexity in real systems

---

## Challenge Exercise (Optional)

### Create Complete Firewall Profile for Web Server

**Objective:** Design and implement firewall rules for production web server

1. Create rules for:
   - Inbound HTTP (port 80)
   - Inbound HTTPS (port 443)
   - Inbound SSH for admin (port 22) - restricted to admin IP
   - Outbound DNS (port 53)
   - Outbound HTTP/HTTPS for updates (ports 80, 443)
   - Block all other inbound traffic
   - Block all other outbound traffic

2. Verify each rule:
   - Test allowed traffic (should work)
   - Test blocked traffic (should fail)
   - Verify source IP restrictions work

3. Document:
   - All rules created
   - Testing results
   - Any issues encountered
   - Improvement recommendations

This combines all learned concepts and provides real-world firewall design experience.
