# 09 - Cloud Networking: AWS VPC - Cheatsheet

## AWS VPC Core Concepts

| Concept | Definition | Example |
|---------|-----------|---------|
| **VPC** | Isolated network environment in AWS | 10.0.0.0/16 |
| **Subnet** | Division of VPC for organizing resources | 10.0.1.0/24 (public) |
| **CIDR Block** | IP address range notation | 10.0.0.0/16 = 65,536 IPs |
| **IGW** | Internet Gateway - connects VPC to internet | Route 0.0.0.0/0 → igw-xxxxx |
| **NAT Gateway** | Allows private subnets to access internet | Placed in public subnet |
| **Availability Zone** | Isolated data center within region | us-east-1a, us-east-1b |
| **Route Table** | Defines traffic routing rules | Local + Internet routes |
| **NACL** | Subnet-level stateless firewall | Port 80,443 allow; 22 deny |
| **Security Group** | Instance-level stateful firewall | Inbound: port 80 from 0.0.0.0/0 |
| **Elastic IP** | Static public IP address | Associated with NAT or instance |

---

## VPC Creation Commands

### Create VPC
```bash
# Basic VPC creation
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# VPC with tag
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --query 'Vpc.VpcId' \
  --output text
```

### Add Tags to VPC
```bash
# Tag VPC with name and environment
aws ec2 create-tags \
  --resources vpc-xxxxx \
  --tags Key=Name,Value=my-vpc Key=environment,Value=production
```

### List VPCs
```bash
# List all VPCs
aws ec2 describe-vpcs --output table

# List specific VPC with details
aws ec2 describe-vpcs \
  --vpc-ids vpc-xxxxx \
  --query 'Vpcs[0].[VpcId,CidrBlock,State]' \
  --output table
```

### Delete VPC
```bash
# Delete VPC (must be empty)
aws ec2 delete-vpc --vpc-id vpc-xxxxx
```

---

## Subnet Commands

### Create Subnet
```bash
# Create public subnet
aws ec2 create-subnet \
  --vpc-id vpc-xxxxx \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a

# Create private subnet
aws ec2 create-subnet \
  --vpc-id vpc-xxxxx \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1b
```

### List Subnets
```bash
# List all subnets in VPC
aws ec2 describe-subnets \
  --filters Name=vpc-id,Values=vpc-xxxxx \
  --output table

# List with specific columns
aws ec2 describe-subnets \
  --filters Name=vpc-id,Values=vpc-xxxxx \
  --query 'Subnets[*].[SubnetId,CidrBlock,AvailabilityZone]' \
  --output table
```

### Enable Public IP Assignment
```bash
# Auto-assign public IP to instances in subnet
aws ec2 modify-subnet-attribute \
  --subnet-id subnet-xxxxx \
  --map-public-ip-on-launch

# Verify setting
aws ec2 describe-subnet-attribute \
  --subnet-id subnet-xxxxx \
  --attribute mapPublicIpOnLaunch
```

### Delete Subnet
```bash
aws ec2 delete-subnet --subnet-id subnet-xxxxx
```

---

## Internet Gateway Commands

### Create IGW
```bash
# Create Internet Gateway
aws ec2 create-internet-gateway \
  --query 'InternetGateway.InternetGatewayId' \
  --output text
```

### Attach IGW to VPC
```bash
# Attach to VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-xxxxx \
  --vpc-id vpc-xxxxx

# Verify attachment
aws ec2 describe-internet-gateways \
  --internet-gateway-ids igw-xxxxx
```

### List IGWs
```bash
# List all IGWs
aws ec2 describe-internet-gateways --output table

# List IGWs attached to specific VPC
aws ec2 describe-internet-gateways \
  --filters Name=attachment.vpc-id,Values=vpc-xxxxx
```

### Detach and Delete IGW
```bash
# Detach IGW from VPC
aws ec2 detach-internet-gateway \
  --internet-gateway-id igw-xxxxx \
  --vpc-id vpc-xxxxx

# Delete IGW
aws ec2 delete-internet-gateway --internet-gateway-id igw-xxxxx
```

---

## Route Table Commands

### Create Route Table
```bash
# Create route table
aws ec2 create-route-table \
  --vpc-id vpc-xxxxx \
  --query 'RouteTable.RouteTableId' \
  --output text
```

### Add Routes
```bash
# Add route to Internet Gateway
aws ec2 create-route \
  --route-table-id rtb-xxxxx \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-xxxxx

# Add route to NAT Gateway
aws ec2 create-route \
  --route-table-id rtb-xxxxx \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id nat-xxxxx

# Add route to EC2 instance
aws ec2 create-route \
  --route-table-id rtb-xxxxx \
  --destination-cidr-block 192.168.0.0/16 \
  --instance-id i-xxxxx
```

### List Route Tables
```bash
# List all route tables in VPC
aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=vpc-xxxxx

# List route table with specific format
aws ec2 describe-route-tables \
  --route-table-ids rtb-xxxxx \
  --query 'RouteTables[0].Routes[*].[DestinationCidrBlock,GatewayId,NatGatewayId]' \
  --output table
```

### Associate Route Table
```bash
# Associate route table with subnet
aws ec2 associate-route-table \
  --subnet-id subnet-xxxxx \
  --route-table-id rtb-xxxxx

# Verify association
aws ec2 describe-route-tables \
  --route-table-ids rtb-xxxxx \
  --query 'RouteTables[0].Associations'
```

### Delete Routes and Route Tables
```bash
# Delete route from route table
aws ec2 delete-route \
  --route-table-id rtb-xxxxx \
  --destination-cidr-block 0.0.0.0/0

# Delete route table
aws ec2 delete-route-table --route-table-id rtb-xxxxx
```

---

## NAT Gateway Commands

### Allocate Elastic IP
```bash
# Allocate Elastic IP for NAT
aws ec2 allocate-address \
  --domain vpc \
  --query 'AllocationId' \
  --output text

# Get public IP
aws ec2 allocate-address \
  --domain vpc \
  --query 'PublicIp' \
  --output text
```

### Create NAT Gateway
```bash
# Create NAT Gateway (in public subnet)
aws ec2 create-nat-gateway \
  --subnet-id subnet-xxxxx \
  --allocation-id eipalloc-xxxxx \
  --query 'NatGateway.NatGatewayId' \
  --output text

# Wait for NAT to be available
aws ec2 wait nat-gateway-available --nat-gateway-ids nat-xxxxx
```

### List NAT Gateways
```bash
# List all NAT Gateways
aws ec2 describe-nat-gateways --output table

# List NAT Gateway with details
aws ec2 describe-nat-gateways \
  --nat-gateway-ids nat-xxxxx \
  --query 'NatGateways[0].[State,SubnetId,NatGatewayAddresses[0].PublicIp]' \
  --output table
```

### Delete NAT Gateway
```bash
# Delete NAT Gateway
aws ec2 delete-nat-gateway --nat-gateway-id nat-xxxxx

# Wait for deletion
aws ec2 wait nat-gateway-deleted --nat-gateway-ids nat-xxxxx

# Release Elastic IP
aws ec2 release-address --allocation-id eipalloc-xxxxx
```

---

## Security Group Commands

### Create Security Group
```bash
# Create security group
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Security group for web servers" \
  --vpc-id vpc-xxxxx
```

### Add Inbound Rules
```bash
# Allow HTTP (port 80)
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow HTTPS (port 443)
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow SSH from specific IP
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/32
```

### Add Egress Rules
```bash
# Allow all outbound traffic (default)
aws ec2 authorize-security-group-egress \
  --group-id sg-xxxxx \
  --protocol all \
  --cidr 0.0.0.0/0
```

### List Security Groups
```bash
# List security groups in VPC
aws ec2 describe-security-groups \
  --filters Name=vpc-id,Values=vpc-xxxxx \
  --output table

# List rules for specific group
aws ec2 describe-security-groups \
  --group-ids sg-xxxxx \
  --query 'SecurityGroups[0].IpPermissions[*].[IpProtocol,FromPort,ToPort,IpRanges[0].CidrIp]' \
  --output table
```

### Delete Security Group
```bash
# Delete security group (must not be in use)
aws ec2 delete-security-group --group-id sg-xxxxx

# Revoke inbound rule first
aws ec2 revoke-security-group-ingress \
  --group-id sg-xxxxx \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

---

## Network ACL Commands

### Create Network ACL
```bash
# Create Network ACL
aws ec2 create-network-acl \
  --vpc-id vpc-xxxxx \
  --query 'NetworkAcl.NetworkAclId' \
  --output text
```

### Add NACL Rules
```bash
# Allow HTTP inbound
aws ec2 create-network-acl-entry \
  --network-acl-id acl-xxxxx \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --ingress

# Allow HTTPS inbound
aws ec2 create-network-acl-entry \
  --network-acl-id acl-xxxxx \
  --rule-number 110 \
  --protocol tcp \
  --port-range From=443,To=443 \
  --cidr-block 0.0.0.0/0 \
  --ingress

# Deny SSH inbound
aws ec2 create-network-acl-entry \
  --network-acl-id acl-xxxxx \
  --rule-number 120 \
  --protocol tcp \
  --port-range From=22,To=22 \
  --cidr-block 10.0.0.0/8 \
  --ingress \
  --egress false
```

### List NACLs
```bash
# List NACLs in VPC
aws ec2 describe-network-acls \
  --filters Name=vpc-id,Values=vpc-xxxxx

# List rules for NACL
aws ec2 describe-network-acls \
  --network-acl-ids acl-xxxxx \
  --query 'NetworkAcls[0].Entries[*].[RuleNumber,Protocol,PortRange,CidrBlock,RuleAction]' \
  --output table
```

### Associate NACL
```bash
# Associate NACL with subnet
aws ec2 associate-network-acl \
  --network-acl-id acl-xxxxx \
  --subnet-id subnet-xxxxx
```

### Delete NACL Rules
```bash
# Delete NACL entry
aws ec2 delete-network-acl-entry \
  --network-acl-id acl-xxxxx \
  --rule-number 100 \
  --ingress

# Delete NACL
aws ec2 delete-network-acl --network-acl-id acl-xxxxx
```

---

## VPC Peering Commands

### Create VPC Peering
```bash
# Request peering connection
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-xxxxx \
  --peer-vpc-id vpc-yyyyy \
  --query 'VpcPeeringConnection.VpcPeeringConnectionId' \
  --output text

# Accept peering (if in different account/region)
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id pcx-xxxxx
```

### List Peering Connections
```bash
# List all peering connections
aws ec2 describe-vpc-peering-connections --output table

# List active peerings
aws ec2 describe-vpc-peering-connections \
  --filters Name=status-code,Values=active \
  --output table
```

### Create Routes for Peering
```bash
# Add route through peering connection
aws ec2 create-route \
  --route-table-id rtb-xxxxx \
  --destination-cidr-block 10.1.0.0/16 \
  --vpc-peering-connection-id pcx-xxxxx
```

---

## Troubleshooting Commands

### Check Instance Network Details
```bash
# Get instance IP addresses and VPC info
aws ec2 describe-instances \
  --instance-ids i-xxxxx \
  --query 'Reservations[0].Instances[0].[PrivateIpAddress,PublicIpAddress,SubnetId,VpcId]'

# List all instances in VPC
aws ec2 describe-instances \
  --filters Name=vpc-id,Values=vpc-xxxxx \
  --query 'Reservations[*].Instances[*].[InstanceId,PrivateIpAddress,PublicIpAddress]' \
  --output table
```

### Verify Connectivity Path
```bash
# Check security group for instance
aws ec2 describe-instances \
  --instance-ids i-xxxxx \
  --query 'Reservations[0].Instances[0].SecurityGroups'

# Verify route table for subnet
aws ec2 describe-route-tables \
  --filters Name=association.subnet-id,Values=subnet-xxxxx \
  --query 'RouteTables[0].Routes' \
  --output table

# Check NACL for subnet
aws ec2 describe-network-acls \
  --filters Name=association.subnet-id,Values=subnet-xxxxx
```

### Elastic IP Commands
```bash
# List all Elastic IPs
aws ec2 describe-addresses --output table

# Check Elastic IP association
aws ec2 describe-addresses \
  --allocation-ids eipalloc-xxxxx \
  --query 'Addresses[0].[PublicIp,AssociationId,InstanceId]'

# Associate Elastic IP with instance
aws ec2 associate-address \
  --instance-id i-xxxxx \
  --allocation-id eipalloc-xxxxx

# Disassociate Elastic IP
aws ec2 disassociate-address \
  --association-id eipassoc-xxxxx
```

---

## Quick Reference: CIDR to Subnet Count

| CIDR | IPs | Usable | Typical Use |
|------|-----|--------|------------|
| /16 | 65,536 | 65,531 | VPC |
| /20 | 4,096 | 4,091 | Large subnet |
| /24 | 256 | 251 | Standard subnet |
| /25 | 128 | 123 | Small subnet |
| /28 | 16 | 11 | Minimal subnet |

---

## Common Task Patterns

### Complete VPC Setup
```bash
# 1. Create VPC
VPC=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query 'Vpc.VpcId' --output text)

# 2. Create subnet
SUBNET=$(aws ec2 create-subnet --vpc-id $VPC --cidr-block 10.0.1.0/24 --az us-east-1a --query 'Subnet.SubnetId' --output text)

# 3. Create and attach IGW
IGW=$(aws ec2 create-internet-gateway --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --igw-id $IGW --vpc-id $VPC

# 4. Create route table
RT=$(aws ec2 create-route-table --vpc-id $VPC --query 'RouteTable.RouteTableId' --output text)

# 5. Add route to IGW
aws ec2 create-route --route-table-id $RT --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW

# 6. Associate with subnet
aws ec2 associate-route-table --subnet-id $SUBNET --route-table-id $RT
```

### Private Subnet with NAT
```bash
# After VPC and public subnet setup:

# 1. Allocate Elastic IP
EIP=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)

# 2. Create NAT Gateway
NAT=$(aws ec2 create-nat-gateway --subnet-id $PUBLIC_SUBNET --allocation-id $EIP --query 'NatGateway.NatGatewayId' --output text)

# 3. Create private route table
PRIVATE_RT=$(aws ec2 create-route-table --vpc-id $VPC --query 'RouteTable.RouteTableId' --output text)

# 4. Add route to NAT
aws ec2 create-route --route-table-id $PRIVATE_RT --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT

# 5. Associate with private subnet
aws ec2 associate-route-table --subnet-id $PRIVATE_SUBNET --route-table-id $PRIVATE_RT
```
