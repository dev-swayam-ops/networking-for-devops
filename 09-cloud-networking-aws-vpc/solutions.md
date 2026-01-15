# 09 - Cloud Networking: AWS VPC - Solutions

## Exercise 1: List All VPCs in Your Account

### Solution

**Command:**
```bash
aws ec2 describe-vpcs \
  --query 'Vpcs[*].[VpcId,CidrBlock,State,IsDefault]' \
  --output table
```

**Alternative - JSON format:**
```bash
aws ec2 describe-vpcs \
  --query 'Vpcs[*].[VpcId,CidrBlock,State]' \
  --output json
```

**Expected Output:**
```
----------------------------------------------------------
|                     DescribeVpcs                       |
|--------|-------------|----------|------------------|
| VpcId  | CidrBlock   | State    | IsDefault        |
|--------|-------------|----------|------------------|
| vpc-1a2b3c4d  | 172.31.0.0/16 | available | true |
| vpc-5e6f7g8h  | 10.0.0.0/16   | available | false|
| vpc-9i0j1k2l  | 10.1.0.0/16   | available | false|
----------------------------------------------------------
```

### Explanation
- `describe-vpcs` lists all VPCs in the current region
- `IsDefault` = true indicates AWS's default VPC created with the account
- `State` = available means the VPC is ready to use
- Most accounts have the default VPC (172.31.0.0/16) plus custom VPCs

---

## Exercise 2: Examine VPC Components

### Solution

**List Subnets:**
```bash
VPC_ID="vpc-1a2b3c4d"  # Replace with your VPC ID from Exercise 1

aws ec2 describe-subnets \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query 'Subnets[*].[SubnetId,CidrBlock,AvailabilityZone]' \
  --output table
```

**List Route Tables:**
```bash
aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query 'RouteTables[*].[RouteTableId,Associations[*].Main,Associations[*].SubnetId]' \
  --output table
```

**List Internet Gateways:**
```bash
aws ec2 describe-internet-gateways \
  --filters Name=attachment.vpc-id,Values=$VPC_ID \
  --query 'InternetGateways[*].[InternetGatewayId,Attachments[*].State]' \
  --output table
```

**Expected Output - Subnets:**
```
-------------------------------------------------------------------
|                     DescribeSubnets                             |
|------------|------------|---------------------------|
| SubnetId   | CidrBlock  | AvailabilityZone          |
|------------|------------|---------------------------|
| subnet-001 | 10.0.1.0/24| us-east-1a                |
| subnet-002 | 10.0.2.0/24| us-east-1b                |
| subnet-003 | 10.0.3.0/24| us-east-1c                |
-------------------------------------------------------------------
```

**Expected Output - Route Tables:**
```
------------------------------------------------------------------
|                  DescribeRouteTables                            |
|------------|--------|---------------------------|
| RouteTableId | Main | SubnetId                 |
|------------|--------|---------------------------|
| rtb-main   | true  | (main route table)       |
| rtb-pub1   | false | subnet-001, subnet-002  |
| rtb-priv   | false | subnet-003              |
------------------------------------------------------------------
```

**Expected Output - Internet Gateways:**
```
---------------------------------------
|     DescribeInternetGateways        |
|----------|----------|
| IGW ID   | State    |
|----------|----------|
| igw-001  | attached |
---------------------------------------
```

### Explanation
- Each VPC has at least one route table (the main one)
- VPCs may have multiple route tables for different subnets
- IGWs are either attached (to 1 VPC) or detached
- Most VPCs don't have an IGW by default

---

## Exercise 3: Analyze Subnet Networking Details

### Solution

**Command:**
```bash
VPC_ID="vpc-1a2b3c4d"

aws ec2 describe-subnets \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query 'Subnets[*].[SubnetId,CidrBlock,AvailabilityZone,AvailableIpAddressCount,MapPublicIpOnLaunch]' \
  --output table
```

**Detailed command with calculations:**
```bash
# For each subnet, show available IPs
aws ec2 describe-subnets \
  --filters Name=vpc-id,Values=$VPC_ID \
  --output json | jq -r '.Subnets[] | 
  "\(.SubnetId) | \(.CidrBlock) | \(.AvailabilityZone) | Available: \(.AvailableIpAddressCount) | Public: \(.MapPublicIpOnLaunch)"'
```

**Expected Output:**
```
---------------------------------------------------------------------
|              DescribeSubnets with Public IP Info                  |
|---------|---------|-----------|--------|-------------|
| Subnet  | CIDR    | AZ        | Avail  | PublicIP   |
|---------|---------|-----------|--------|-------------|
| subnet-001| 10.0.1.0/24| us-east-1a| 246 | true       |
| subnet-002| 10.0.2.0/24| us-east-1b| 251 | false      |
| subnet-003| 10.0.3.0/24| us-east-1c| 248 | false      |
---------------------------------------------------------------------
```

### Explanation
- /24 subnet has 256 IPs total
- AWS reserves 5 IPs per subnet: network, gateway, DNS, broadcast, AWS reserved
- Available count = Total - Reserved (typically 251 for /24)
- MapPublicIpOnLaunch = true means instances get public IPs automatically
- Different AZs provide availability redundancy

---

## Exercise 4: Trace Network Routes

### Solution

**Command:**
```bash
VPC_ID="vpc-1a2b3c4d"

aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query 'RouteTables[*].Routes[*].[DestinationCidrBlock,GatewayId,NatGatewayId,InstanceId,State]' \
  --output table
```

**Better formatted command:**
```bash
aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=$VPC_ID \
  --output json | jq -r '.RouteTables[] | 
  "Route Table: \(.RouteTableId)\n" + 
  (.Routes[] | "\(.DestinationCidrBlock // .DestinationIpv6CidrBlock) → \(.GatewayId // .NatGatewayId // .InstanceId // "local") [\(.State)]") + "\n"'
```

**Expected Output:**
```
Route Table: rtb-12345678
10.0.0.0/16 → local [active]
0.0.0.0/0 → igw-87654321 [active]

Route Table: rtb-87654321
10.0.0.0/16 → local [active]
0.0.0.0/0 → nat-abcdef123 [active]

Route Table: rtb-56789012
10.0.0.0/16 → local [active]
```

### Explanation
- `local` route is automatic - traffic within VPC CIDR
- Default route (0.0.0.0/0) sends internet traffic to IGW or NAT
- NAT routes direct private subnet traffic through NAT Gateway
- State = active means route is functional
- Each route table controls traffic for associated subnets

---

## Exercise 5: Query Security Group Rules

### Solution

**List all security groups in VPC:**
```bash
VPC_ID="vpc-1a2b3c4d"

aws ec2 describe-security-groups \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query 'SecurityGroups[*].[GroupId,GroupName]' \
  --output table
```

**Display rules for a specific security group:**
```bash
GROUP_ID="sg-12345678"

aws ec2 describe-security-groups \
  --group-ids $GROUP_ID \
  --query 'SecurityGroups[0].IpPermissions[*].[IpProtocol,FromPort,ToPort,IpRanges[0].CidrIp,UserIdGroupPairs[0].GroupId]' \
  --output table
```

**Detailed command with better formatting:**
```bash
aws ec2 describe-security-groups \
  --group-ids $GROUP_ID \
  --output json | jq '.SecurityGroups[0] | 
  "\(.GroupId) (\(.GroupName))\nInbound Rules:\n" + 
  (.IpPermissions[] | 
  "  Port: \(.FromPort // "N/A")-\(.ToPort // "N/A") | Protocol: \(.IpProtocol) | Source: \(.IpRanges[0].CidrIp // .UserIdGroupPairs[0].GroupId)")'
```

**Expected Output:**
```
------------------------------------------------------------------------------------
|            DescribeSecurityGroups - Ingress Rules                               |
|------------|--------|----------|-----------|---------|
| GroupId    | Port   | Protocol | Source    | Description |
|------------|--------|----------|-----------|---------|
| sg-12345678| 80     | tcp      | 0.0.0.0/0 | HTTP traffic |
| sg-12345678| 443    | tcp      | 0.0.0.0/0 | HTTPS traffic |
| sg-12345678| 22     | tcp      | 10.0.0.0/8| SSH from internal |
------------------------------------------------------------------------------------
```

### Explanation
- GroupId is the unique security group identifier
- IpProtocol = tcp/udp/-1 (all)
- FromPort/ToPort define port range (-1 = all ports)
- CidrIp is source IP range (0.0.0.0/0 = anywhere)
- UserIdGroupPairs allow traffic from another security group
- Egress rules are default allow-all unless modified

---

## Exercise 6: Create a New VPC from Scratch

### Solution

**Step 1: Create VPC**
```bash
# Create VPC with 10.1.0.0/16 CIDR
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block 10.1.0.0/16 \
  --query 'Vpc.VpcId' \
  --output text)

echo "VPC ID: $VPC_ID"

# Tag the VPC
aws ec2 create-tags \
  --resources $VPC_ID \
  --tags Key=Name,Value=training-vpc Key=environment,Value=training
```

**Output:**
```
VPC ID: vpc-0a1b2c3d4e5f6g7h8
```

**Step 2: Create Public Subnet**
```bash
PUBLIC_SUBNET=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.1.1.0/24 \
  --availability-zone us-east-1a \
  --query 'Subnet.SubnetId' \
  --output text)

echo "Public Subnet: $PUBLIC_SUBNET"

# Tag public subnet
aws ec2 create-tags \
  --resources $PUBLIC_SUBNET \
  --tags Key=Name,Value=public-subnet Key=Type,Value=Public Key=environment,Value=training
```

**Output:**
```
Public Subnet: subnet-0a1b2c3d4e5f6g7h8
```

**Step 3: Create Private Subnet**
```bash
PRIVATE_SUBNET=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.1.2.0/24 \
  --availability-zone us-east-1b \
  --query 'Subnet.SubnetId' \
  --output text)

echo "Private Subnet: $PRIVATE_SUBNET"

# Tag private subnet
aws ec2 create-tags \
  --resources $PRIVATE_SUBNET \
  --tags Key=Name,Value=private-subnet Key=Type,Value=Private Key=environment,Value=training
```

**Output:**
```
Private Subnet: subnet-1b2c3d4e5f6g7h8i9
```

**Step 4: Create and Attach Internet Gateway**
```bash
IGW_ID=$(aws ec2 create-internet-gateway \
  --query 'InternetGateway.InternetGatewayId' \
  --output text)

echo "IGW ID: $IGW_ID"

# Attach IGW to VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id $IGW_ID \
  --vpc-id $VPC_ID

# Tag IGW
aws ec2 create-tags \
  --resources $IGW_ID \
  --tags Key=Name,Value=training-igw Key=environment,Value=training
```

**Output:**
```
IGW ID: igw-0a1b2c3d4e5f6g7h8
```

**Step 5: Verify All Resources**
```bash
echo "=== VPC Details ==="
aws ec2 describe-vpcs --vpc-ids $VPC_ID --output table

echo -e "\n=== Subnets ==="
aws ec2 describe-subnets \
  --filters Name=vpc-id,Values=$VPC_ID \
  --output table

echo -e "\n=== Internet Gateway ==="
aws ec2 describe-internet-gateways --internet-gateway-ids $IGW_ID --output table
```

**Expected Output:**
```
=== VPC Details ===
------------------------------------------
| VPC ID           | CIDR Block    | State    |
|-----------------|---------------|----------|
| vpc-0a1b2c3d... | 10.1.0.0/16   | available |
------------------------------------------

=== Subnets ===
--------------------------------------------------
| SubnetId | CIDR Block | AZ         | Available|
|----------|-----------|------------|----------|
| subnet-0a...| 10.1.1.0/24 | us-east-1a | 251    |
| subnet-1b...| 10.1.2.0/24 | us-east-1b | 251    |
--------------------------------------------------

=== Internet Gateway ===
------------------------------------
| IGW ID        | State    | Attached|
|---------------|----------|---------|
| igw-0a1b2c... | attached | true    |
------------------------------------
```

---

## Exercise 7: Configure Routing for Internet Access

### Solution

**Step 1: Get Main Route Table**
```bash
ROUTE_TABLE=$(aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query 'RouteTables[0].RouteTableId' \
  --output text)

echo "Route Table ID: $ROUTE_TABLE"
```

**Output:**
```
Route Table ID: rtb-0a1b2c3d4e5f6g7h8
```

**Step 2: Create Default Route to IGW**
```bash
aws ec2 create-route \
  --route-table-id $ROUTE_TABLE \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id $IGW_ID

# Verify the route was created
aws ec2 describe-route-tables \
  --route-table-ids $ROUTE_TABLE \
  --output table
```

**Expected Output:**
```
---------------------------------------------------------------------
|                     DescribeRouteTables                           |
|---------|---------|---------|
| Destination | Target | State  |
|---------|---------|---------|
| 10.1.0.0/16 | local  | active |
| 0.0.0.0/0   | igw-0a1b... | active |
---------------------------------------------------------------------
```

**Step 3: Associate Route Table with Public Subnet**
```bash
aws ec2 associate-route-table \
  --subnet-id $PUBLIC_SUBNET \
  --route-table-id $ROUTE_TABLE

# Verify association
aws ec2 describe-route-tables \
  --route-table-ids $ROUTE_TABLE \
  --query 'RouteTables[0].Associations[*].[SubnetId,Main]' \
  --output table
```

**Expected Output:**
```
-------------------------------------
|    Route Table Associations        |
|---------|------------|
| SubnetId | Main      |
|---------|------------|
| subnet-0a...| false |
-------------------------------------
```

**Step 4: Enable Auto-Assign Public IP**
```bash
aws ec2 modify-subnet-attribute \
  --subnet-id $PUBLIC_SUBNET \
  --map-public-ip-on-launch

# Verify
aws ec2 describe-subnet-attribute \
  --subnet-id $PUBLIC_SUBNET \
  --attribute mapPublicIpOnLaunch \
  --output table
```

**Expected Output:**
```
------------------------------------------
| MapPublicIpOnLaunch                    |
| Value: true                            |
------------------------------------------
```

---

## Exercise 8: Set Up NAT Gateway for Private Subnet

### Solution

**Step 1: Allocate Elastic IP**
```bash
ALLOCATION_ID=$(aws ec2 allocate-address \
  --domain vpc \
  --query 'AllocationId' \
  --output text)

PUBLIC_IP=$(aws ec2 allocate-address \
  --domain vpc \
  --query 'PublicIp' \
  --output text)

echo "Allocation ID: $ALLOCATION_ID"
echo "Public IP: $PUBLIC_IP"
```

**Output:**
```
Allocation ID: eipalloc-0a1b2c3d4e5f6g7h8
Public IP: 203.0.113.42
```

**Step 2: Create NAT Gateway**
```bash
NAT_ID=$(aws ec2 create-nat-gateway \
  --subnet-id $PUBLIC_SUBNET \
  --allocation-id $ALLOCATION_ID \
  --query 'NatGateway.NatGatewayId' \
  --output text)

echo "NAT Gateway ID: $NAT_ID"

# Wait for NAT Gateway to be available
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_ID
echo "NAT Gateway is available"

# Tag NAT Gateway
aws ec2 create-tags \
  --resources $NAT_ID \
  --tags Key=Name,Value=training-nat Key=environment,Value=training
```

**Output:**
```
NAT Gateway ID: nat-0a1b2c3d4e5f6g7h8
NAT Gateway is available
```

**Step 3: Create Route Table for Private Subnet**
```bash
PRIVATE_RT=$(aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --query 'RouteTable.RouteTableId' \
  --output text)

echo "Private Route Table: $PRIVATE_RT"

# Tag it
aws ec2 create-tags \
  --resources $PRIVATE_RT \
  --tags Key=Name,Value=private-rt Key=environment,Value=training
```

**Output:**
```
Private Route Table: rtb-1b2c3d4e5f6g7h8i9
```

**Step 4: Add Route to NAT Gateway**
```bash
aws ec2 create-route \
  --route-table-id $PRIVATE_RT \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id $NAT_ID

# Verify
aws ec2 describe-route-tables \
  --route-table-ids $PRIVATE_RT \
  --output table
```

**Expected Output:**
```
---------------------------------------------------------------------
|                     DescribeRouteTables                           |
|---------|---------|---------|
| Destination | Target | State  |
|---------|---------|---------|
| 10.1.0.0/16 | local  | active |
| 0.0.0.0/0   | nat-0a1b... | available |
---------------------------------------------------------------------
```

**Step 5: Associate with Private Subnet**
```bash
aws ec2 associate-route-table \
  --subnet-id $PRIVATE_SUBNET \
  --route-table-id $PRIVATE_RT

# Verify
aws ec2 describe-route-tables \
  --route-table-ids $PRIVATE_RT \
  --query 'RouteTables[0].Associations[*].[SubnetId,Main]' \
  --output table
```

**Expected Output:**
```
-------------------------------------
|    Route Table Associations        |
|---------|------------|
| SubnetId | Main      |
|---------|------------|
| subnet-1b...| false |
-------------------------------------
```

---

## Exercise 9: Create Network ACL Rules

### Solution

**Step 1: Create Custom NACL**
```bash
NACL_ID=$(aws ec2 create-network-acl \
  --vpc-id $VPC_ID \
  --query 'NetworkAcl.NetworkAclId' \
  --output text)

echo "Network ACL ID: $NACL_ID"

# Tag it
aws ec2 create-tags \
  --resources $NACL_ID \
  --tags Key=Name,Value=training-nacl Key=environment,Value=training
```

**Output:**
```
Network ACL ID: acl-0a1b2c3d4e5f6g7h8
```

**Step 2: Create Inbound Rules**

**Allow HTTP (Port 80):**
```bash
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --ingress

echo "HTTP rule created"
```

**Allow HTTPS (Port 443):**
```bash
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --rule-number 110 \
  --protocol tcp \
  --port-range From=443,To=443 \
  --cidr-block 0.0.0.0/0 \
  --ingress

echo "HTTPS rule created"
```

**Deny SSH from Internal (Port 22 from 10.0.0.0/8):**
```bash
aws ec2 create-network-acl-entry \
  --network-acl-id $NACL_ID \
  --rule-number 120 \
  --protocol tcp \
  --port-range From=22,To=22 \
  --cidr-block 10.0.0.0/8 \
  --ingress \
  --egress false

echo "SSH deny rule created"
```

**Step 3: Associate with Public Subnet**
```bash
aws ec2 associate-network-acl \
  --network-acl-id $NACL_ID \
  --subnet-id $PUBLIC_SUBNET

echo "NACL associated with public subnet"
```

**Step 4: Verify Rules**
```bash
aws ec2 describe-network-acls \
  --network-acl-ids $NACL_ID \
  --query 'NetworkAcls[0].Entries[*].[RuleNumber,Protocol,PortRange,CidrBlock,RuleAction]' \
  --output table
```

**Expected Output:**
```
---------------------------------------------------------------
|            Network ACL Rules                               |
|------|----------|---------|-----------|---------|
| Rule | Protocol | Port    | CIDR      | Action |
|------|----------|---------|-----------|---------|
| 100  | tcp      | 80-80   | 0.0.0.0/0 | allow  |
| 110  | tcp      | 443-443 | 0.0.0.0/0 | allow  |
| 120  | tcp      | 22-22   | 10.0.0.0/8| deny   |
| 32767| -1       | -       | 0.0.0.0/0 | deny   |
---------------------------------------------------------------
```

---

## Exercise 10: Troubleshoot VPC Connectivity Issues

### Solutions and Diagnostics

#### Scenario A: Private Subnet Can't Reach Internet

**Symptoms:**
- Instances in private subnet cannot reach internet
- NAT Gateway is in available state
- Public subnet works fine

**Troubleshooting Commands:**
```bash
# 1. Check NAT Gateway status
aws ec2 describe-nat-gateways \
  --nat-gateway-ids nat-xxxxx \
  --query 'NatGateways[0].[State,SubnetId,NatGatewayAddresses[0].PublicIp]' \
  --output table

# 2. Verify private route table has route to NAT
aws ec2 describe-route-tables \
  --route-table-ids rtb-xxxxx \
  --query 'RouteTables[0].Routes' \
  --output json

# 3. Check if subnet is associated with correct route table
aws ec2 describe-route-tables \
  --filters Name=association.subnet-id,Values=subnet-xxxxx \
  --query 'RouteTables[0].[RouteTableId,Routes]' \
  --output json

# 4. Verify Security Group allows outbound traffic
aws ec2 describe-security-groups \
  --group-ids sg-xxxxx \
  --query 'SecurityGroups[0].IpPermissionsEgress' \
  --output json
```

**Possible Causes and Solutions:**
```
1. Route table not associated with private subnet
   → Solution: Associate correct route table with subnet
   aws ec2 associate-route-table --subnet-id subnet-xxxxx --route-table-id rtb-xxxxx

2. Route to NAT Gateway missing or pointing wrong target
   → Solution: Verify route and recreate if needed
   aws ec2 describe-route-tables --filters Name=vpc-id,Values=vpc-xxxxx

3. NAT Gateway state is not "available"
   → Solution: Wait for NAT Gateway to complete provisioning
   aws ec2 wait nat-gateway-available --nat-gateway-ids nat-xxxxx

4. Security Group blocking outbound traffic
   → Solution: Verify egress rules allow all traffic
   Default: Allow all egress to 0.0.0.0/0
```

---

#### Scenario B: Public Subnet Instances Not Accessible

**Symptoms:**
- Cannot SSH/RDP to instances in public subnet
- Security group allows port 22/3389 from 0.0.0.0/0
- Instances have public IPs

**Troubleshooting Commands:**
```bash
# 1. Verify instance has public IP
aws ec2 describe-instances \
  --instance-ids i-xxxxx \
  --query 'Reservations[0].Instances[0].[PublicIpAddress,PrivateIpAddress]' \
  --output table

# 2. Check Internet Gateway is attached and routing
aws ec2 describe-internet-gateways \
  --filters Name=attachment.vpc-id,Values=vpc-xxxxx \
  --query 'InternetGateways[*].[InternetGatewayId,Attachments[0].State]' \
  --output table

# 3. Verify public subnet route table has IGW route
aws ec2 describe-route-tables \
  --filters Name=association.subnet-id,Values=subnet-xxxxx \
  --query 'RouteTables[0].Routes[*].[DestinationCidrBlock,GatewayId]' \
  --output table

# 4. Check security group rules
aws ec2 describe-security-groups \
  --group-ids sg-xxxxx \
  --query 'SecurityGroups[0].IpPermissions[*].[IpProtocol,FromPort,ToPort,IpRanges[0].CidrIp]' \
  --output table

# 5. Check NACL rules (if custom NACL)
aws ec2 describe-network-acls \
  --filters Name=association.subnet-id,Values=subnet-xxxxx \
  --query 'NetworkAcls[0].Entries[*].[RuleNumber,Protocol,PortRange,CidrBlock,RuleAction]' \
  --output table
```

**Possible Causes and Solutions:**
```
1. Internet Gateway not attached to VPC
   → Solution: Attach IGW
   aws ec2 attach-internet-gateway --igw-id igw-xxxxx --vpc-id vpc-xxxxx

2. No default route (0.0.0.0/0) to IGW in route table
   → Solution: Create route to IGW
   aws ec2 create-route --route-table-id rtb-xxxxx --destination-cidr-block 0.0.0.0/0 --gateway-id igw-xxxxx

3. Route table not associated with public subnet
   → Solution: Associate route table
   aws ec2 associate-route-table --subnet-id subnet-xxxxx --route-table-id rtb-xxxxx

4. Security group doesn't have inbound rule for port
   → Solution: Authorize inbound rule
   aws ec2 authorize-security-group-ingress --group-id sg-xxxxx --protocol tcp --port 22 --cidr 0.0.0.0/0

5. Network ACL blocking traffic (if custom NACL)
   → Solution: Add inbound rule to NACL
   aws ec2 create-network-acl-entry --network-acl-id acl-xxxxx --rule-number 100 --protocol tcp --port-range From=22,To=22 --cidr-block 0.0.0.0/0 --ingress
```

---

#### Scenario C: Subnet Creation Failed with Invalid CIDR

**Symptoms:**
- Subnet creation fails with CIDR error
- CIDR block: 10.0.0.5/24 in VPC 10.0.0.0/16
- Error: "InvalidParameterValue"

**Troubleshooting:**
```bash
# This CIDR is INVALID for the following reasons:
# 1. 10.0.0.5 is not a valid subnet network address
#    - Network address must have all host bits as 0
#    - For /24, the last octet must be 0, 1, 2, ... 255
#    - 10.0.0.5 has host bits set

# 2. 10.0.0.0/24 would be valid
# 3. 10.0.1.0/24 would be valid
# 4. 10.0.2.0/24 would be valid

# Verify the VPC CIDR
aws ec2 describe-vpcs --vpc-ids vpc-xxxxx --output table

# To fix, use valid CIDR:
aws ec2 create-subnet \
  --vpc-id vpc-xxxxx \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a
```

**Key Learning:**
- Subnet CIDR must be a valid network address (host bits = 0)
- CIDR must be fully contained within VPC CIDR
- AWS validates both conditions

---

## Summary of AWS VPC CLI Commands

| Command | Purpose |
|---------|---------|
| `aws ec2 describe-vpcs` | List VPCs and their CIDR blocks |
| `aws ec2 create-vpc` | Create new VPC |
| `aws ec2 delete-vpc` | Delete VPC |
| `aws ec2 create-subnet` | Create subnet in VPC |
| `aws ec2 create-internet-gateway` | Create IGW |
| `aws ec2 attach-internet-gateway` | Attach IGW to VPC |
| `aws ec2 create-route` | Add route to route table |
| `aws ec2 allocate-address` | Create Elastic IP |
| `aws ec2 create-nat-gateway` | Create NAT Gateway |
| `aws ec2 create-route-table` | Create route table |
| `aws ec2 associate-route-table` | Associate route table with subnet |
| `aws ec2 create-network-acl` | Create Network ACL |
| `aws ec2 create-network-acl-entry` | Add rule to NACL |
