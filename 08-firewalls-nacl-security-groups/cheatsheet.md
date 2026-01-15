# Cheatsheet: Firewalls, NACLs, and Security Groups

Quick reference for firewall concepts, commands, and configurations.

## Windows Firewall Commands

### Check Firewall Status

| Command | Purpose | Output |
|---------|---------|--------|
| `Get-NetFirewallProfile` | Show all 3 profiles status | Name, Enabled, DefaultInboundAction |
| `Get-NetFirewallProfile -Name Private` | Check Private profile | Details for Private network |
| `netsh advfirewall show allprofiles` | Show all profiles (legacy) | Text summary of all profiles |
| `netsh advfirewall show currentprofile` | Check current active profile | Current profile details |

### List Firewall Rules

| Command | Purpose | Output |
|---------|---------|--------|
| `Get-NetFirewallRule` | List all rules | All firewall rules |
| `Get-NetFirewallRule -Enabled` | List enabled rules only | Only active rules |
| `Get-NetFirewallRule -Disabled` | List disabled rules | Inactive rules |
| `Get-NetFirewallRule -Direction Inbound` | Inbound rules only | Rules for incoming traffic |
| `Get-NetFirewallRule -Direction Outbound` | Outbound rules only | Rules for outgoing traffic |
| `Get-NetFirewallRule -Action Allow` | Allow rules only | Permissive rules |
| `Get-NetFirewallRule -Action Block` | Block rules only | Restrictive rules |
| `Get-NetFirewallRule -DisplayName "*HTTP*"` | Search by name | Rules matching pattern |
| `Get-NetFirewallRule \| Where-Object {$_.Direction -eq "Inbound"} \| Measure-Object` | Count inbound rules | Total count of inbound rules |

### Create Firewall Rules

| Command | Purpose |
|---------|---------|
| `New-NetFirewallRule -Name "RuleName" -Protocol TCP -LocalPort 80 -Action Allow -Direction Inbound` | Create HTTP allow rule |
| `New-NetFirewallRule -Name "RuleName" -Protocol UDP -RemotePort 53 -Action Allow -Direction Outbound` | Create DNS outbound rule |
| `New-NetFirewallRule -Name "RuleName" -Protocol TCP -LocalPort 3306 -RemoteAddress 192.168.1.0/24 -Action Allow -Direction Inbound` | Create restricted rule (MySQL from subnet) |
| `New-NetFirewallRule -Name "RuleName" -Protocol TCP -RemotePort 25 -Action Block -Direction Outbound` | Create block rule (SMTP outbound) |

### Manage Firewall Rules

| Command | Purpose | Effect |
|---------|---------|--------|
| `Disable-NetFirewallRule -DisplayName "RuleName"` | Disable rule | Rule exists but inactive |
| `Enable-NetFirewallRule -DisplayName "RuleName"` | Enable rule | Rule becomes active again |
| `Remove-NetFirewallRule -DisplayName "RuleName"` | Delete rule | Rule deleted permanently |
| `Set-NetFirewallRule -DisplayName "RuleName" -Enabled $false` | Set rule status | Alternative to Disable |
| `Rename-NetFirewallRule -Name "OldName" -NewName "NewName"` | Rename rule | Change rule identifier |

### Inspect Rule Details

| Command | Purpose | Output |
|---------|---------|--------|
| `Get-NetFirewallRule -DisplayName "RuleName" \| Get-NetFirewallPortFilter` | Get port details | Protocol, LocalPort, RemotePort |
| `Get-NetFirewallRule -DisplayName "RuleName" \| Get-NetFirewallAddressFilter` | Get IP details | LocalAddress, RemoteAddress |
| `Get-NetFirewallRule -DisplayName "RuleName" \| Get-NetFirewallApplicationFilter` | Get app details | Program path for rule |
| `Get-NetFirewallRule -DisplayName "RuleName" \| Format-List *` | Show all properties | Complete rule information |

### Test Connectivity

| Command | Purpose | Output |
|---------|---------|--------|
| `Test-NetConnection -ComputerName example.com -Port 80` | Test HTTP connectivity | TcpTestSucceeded: True/False |
| `Test-NetConnection -ComputerName example.com -Port 443 -InformationLevel Detailed` | Detailed HTTPS test | Full connection details |
| `Test-NetConnection -ComputerName localhost -Port 3389` | Test local port | Can access this system on port |
| `telnet example.com 25` | Test SMTP (interactive) | Connect or timeout |
| `ping example.com` | ICMP test | Host reachable/unreachable |

## AWS Security Groups Commands

### Create and Manage Security Groups

| Command | Purpose |
|---------|---------|
| `aws ec2 create-security-group --group-name web-sg --description "Web server" --vpc-id vpc-xxx` | Create security group |
| `aws ec2 describe-security-groups --group-ids sg-xxx` | Show security group details |
| `aws ec2 delete-security-group --group-id sg-xxx` | Delete security group |
| `aws ec2 authorize-security-group-ingress --group-id sg-xxx --protocol tcp --port 80 --cidr 0.0.0.0/0` | Allow HTTP from anywhere |
| `aws ec2 authorize-security-group-ingress --group-id sg-xxx --protocol tcp --port 443 --cidr 0.0.0.0/0` | Allow HTTPS from anywhere |
| `aws ec2 authorize-security-group-ingress --group-id sg-xxx --protocol tcp --port 22 --cidr 203.0.113.0/24` | Allow SSH from subnet |
| `aws ec2 revoke-security-group-ingress --group-id sg-xxx --protocol tcp --port 80 --cidr 0.0.0.0/0` | Remove inbound rule |

### Modify Rules

| Command | Purpose |
|---------|---------|
| `aws ec2 authorize-security-group-egress --group-id sg-xxx --protocol tcp --port 443 --cidr 0.0.0.0/0` | Allow outbound HTTPS |
| `aws ec2 revoke-security-group-egress --group-id sg-xxx --protocol -1 --cidr 0.0.0.0/0` | Remove all outbound allow |
| `aws ec2 describe-security-group-rules --filters Name=group-id,Values=sg-xxx` | List all rules in group |

## AWS NACL Commands

### Manage Network ACLs

| Command | Purpose |
|---------|---------|
| `aws ec2 create-network-acl --vpc-id vpc-xxx` | Create NACL |
| `aws ec2 describe-network-acls --network-acl-ids acl-xxx` | Show NACL details |
| `aws ec2 create-network-acl-entry --network-acl-id acl-xxx --rule-number 100 --protocol tcp --port-range FromPort=80,ToPort=80 --cidr-block 0.0.0.0/0 --ingress` | Allow HTTP inbound |
| `aws ec2 delete-network-acl-entry --network-acl-id acl-xxx --rule-number 100 --ingress` | Delete NACL rule |
| `aws ec2 replace-network-acl-association --association-id acla-xxx --network-acl-id acl-xxx` | Associate NACL to subnet |

## Linux Firewall: ufw (Easy)

### ufw Commands

| Command | Purpose |
|---------|---------|
| `sudo ufw enable` | Enable firewall |
| `sudo ufw disable` | Disable firewall |
| `sudo ufw status` | Show firewall status |
| `sudo ufw status verbose` | Detailed status |
| `sudo ufw allow 80/tcp` | Allow HTTP |
| `sudo ufw allow 443/tcp` | Allow HTTPS |
| `sudo ufw allow 22/tcp` | Allow SSH |
| `sudo ufw deny 25/tcp` | Block SMTP |
| `sudo ufw allow from 192.168.1.0/24 to any port 3306` | Allow MySQL from subnet |
| `sudo ufw delete allow 80/tcp` | Remove HTTP rule |
| `sudo ufw reset` | Remove all rules |

## Linux Firewall: iptables (Advanced)

### iptables Commands

| Command | Purpose | Example |
|---------|---------|---------|
| List rules | Show current rules | `sudo iptables -L -n` |
| List by chain | Show INPUT/OUTPUT/FORWARD | `sudo iptables -L INPUT -n` |
| Append rule | Add rule to end | `sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT` |
| Insert rule | Insert at position | `sudo iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT` |
| Delete rule | Remove specific rule | `sudo iptables -D INPUT 1` |
| Flush chain | Delete all rules in chain | `sudo iptables -F INPUT` |
| Set policy | Default action | `sudo iptables -P INPUT DROP` |
| Allow HTTP | Port 80 | `sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT` |
| Allow SSH | Port 22 restricted | `sudo iptables -A INPUT -p tcp -s 203.0.113.0/24 --dport 22 -j ACCEPT` |
| Save rules | Persist after reboot | `sudo iptables-save > /etc/iptables.rules` |

## Firewall Rules Quick Reference

### Common Ports and Protocols

| Port | Protocol | Service | Rule Type |
|------|----------|---------|-----------|
| 21 | TCP | FTP | Allow inbound for FTP server |
| 22 | TCP | SSH | Allow inbound for remote admin |
| 25 | TCP | SMTP | Block outbound to prevent spam |
| 53 | TCP/UDP | DNS | Allow outbound for DNS resolution |
| 80 | TCP | HTTP | Allow inbound for web server |
| 110 | TCP | POP3 | Allow for email clients |
| 143 | TCP | IMAP | Allow for email clients |
| 443 | TCP | HTTPS | Allow inbound for secure web |
| 445 | TCP | SMB | Allow for file sharing |
| 3306 | TCP | MySQL | Allow from specific subnet |
| 3389 | TCP | RDP | Allow from admin subnet |
| 5432 | TCP | PostgreSQL | Allow from app servers |
| 6379 | TCP | Redis | Allow from app servers |

## Rule Creation Template

### Windows Firewall Rule Template

```powershell
New-NetFirewallRule `
  -Name "UniqueName" `
  -DisplayName "User Friendly Name" `
  -Description "What this rule does" `
  -Protocol TCP `
  -LocalPort 8080 `
  -RemoteAddress 192.168.1.0/24 `
  -Action Allow `
  -Direction Inbound `
  -Profile Any `
  -Enabled $true
```

### ufw Rule Template

```bash
sudo ufw allow from 192.168.1.0/24 to any port 3306 proto tcp
sudo ufw deny out to 10.0.0.0/8 port 25 proto tcp
```

### iptables Rule Template

```bash
sudo iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 3306 -j ACCEPT
sudo iptables -A OUTPUT -p tcp -d 0.0.0.0/0 --dport 25 -j DROP
```

## Rule Priority and Evaluation Order

### Rule Matching (First Match Wins)

```
Packet arrives → Windows Firewall
↓
Check each rule in order (top to bottom)
↓
First matching rule: ACTION TAKEN (ALLOW/BLOCK)
↓
No match: Apply default policy (usually DENY inbound, ALLOW outbound)
```

### Example with Multiple Rules

```
Rule 1: Allow TCP port 22 from 192.168.1.0/24 → MATCHES → ALLOW (stop checking)
Rule 2: Deny TCP port 22 from any
Rule 3: Allow TCP port 22 from 10.0.0.0/8

Result: Connection from 192.168.1.100 → ALLOWED
Result: Connection from 10.0.0.1 → BLOCKED (Rule 1 didn't match, Rule 2 did)
```

## Firewall Architecture Layers

### Defense in Depth

```
Internet
↓
[1] Network Firewall (Hardware)
↓
AWS Security Group (Cloud)
↓
NACL (Subnet level)
↓
[2] OS Firewall (Software) ← This module focuses here
↓
Application
↓
[3] Application Firewall (App level)
```

## Stateful vs Stateless Comparison

| Aspect | Stateful | Stateless |
|--------|----------|-----------|
| **Tracking** | Remembers connection state | Checks each packet independently |
| **Complexity** | Complex rule matching | Simple rules |
| **Inbound Response** | Automatic (related traffic allowed) | Must explicitly allow response |
| **Examples** | Windows Firewall, AWS Security Groups | AWS NACLs, Legacy iptables |
| **Ephemeral Ports** | Automatically handled | Must specify port ranges |
| **CPU Overhead** | Higher (state tracking) | Lower (simple matching) |
| **Use Case** | Most systems | Network perimeter |

## Troubleshooting Checklist

| Issue | Check | Command |
|-------|-------|---------|
| Port not responding | Firewall blocking? | `Get-NetFirewallRule -DisplayName "*port*"` |
| Rule not applied | Rule enabled? | `Get-NetFirewallRule -DisplayName "Rule" \| Select Enabled` |
| Connection refused | Default deny active? | `Get-NetFirewallProfile \| Select DefaultInboundAction` |
| Outbound blocked | Outbound rules too strict? | `Get-NetFirewallRule -Direction Outbound -Action Block` |
| Intermittent access | Rule conflict? | `Get-NetFirewallRule -DisplayName "*pattern*"` |
| Service not working | Port rule exists? | `Test-NetConnection localhost -Port 8080` |

## Security Best Practices

1. **Default Deny** - Block by default, allow specific services
2. **Least Privilege** - Open only ports needed
3. **Source Restriction** - Limit by IP/subnet when possible
4. **Regular Audit** - Review rules regularly
5. **Documentation** - Document why each rule exists
6. **Testing** - Test rules after creation
7. **Cleanup** - Remove unused/disabled rules
8. **Defense in Depth** - Use multiple firewall layers

