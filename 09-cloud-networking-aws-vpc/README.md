# 09 - Cloud Networking: AWS VPC

## What You'll Learn

By the end of this module, you'll be able to:
- Understand AWS VPC architecture and components
- Create and configure Virtual Private Clouds (VPCs)
- Work with subnets, public and private networking
- Configure and manage Internet Gateways and NAT Gateways
- Understand route tables and routing
- Configure security groups and network ACLs
- Set up VPC peering for inter-VPC communication
- Use VPC endpoints for private connectivity to AWS services
- Monitor and troubleshoot VPC networking
- Design secure, scalable network architectures

## Prerequisites

- Completion of modules 00-08
- Understanding of TCP/IP, CIDR notation, and subnets
- AWS account with basic EC2 permissions
- AWS CLI installed and configured
- Familiarity with on-premises networking concepts
- Command-line proficiency

## Key Concepts

### AWS VPC Fundamentals

1. **VPC (Virtual Private Cloud)** - Isolated network environment in AWS
2. **CIDR Block** - IP address range for the VPC (e.g., 10.0.0.0/16)
3. **Subnets** - Subdivisions of VPC for organizing resources
4. **Availability Zones (AZs)** - Isolated data centers within regions
5. **IPv4 vs IPv6** - IP version support in VPC

### VPC Components

1. **Internet Gateway (IGW)** - Enables internet connectivity for public subnets
2. **NAT Gateway** - Provides outbound internet access for private subnets
3. **VPN Gateway** - Secure connection to on-premises networks
4. **Route Tables** - Defines how traffic is routed within VPC
5. **Network ACLs** - Subnet-level stateless firewall
6. **Security Groups** - Instance-level stateful firewall

### Subnets and Networking

1. **Public Subnet** - Has route to Internet Gateway, instances can have public IPs
2. **Private Subnet** - No direct internet access, uses NAT for outbound
3. **Default Route** - 0.0.0.0/0 sends unmatched traffic to specified target
4. **Route Priority** - Most specific CIDR block wins
5. **Availability** - Distribute across AZs for high availability

### VPC Connectivity Options

1. **Internet Gateway** - For public internet access
2. **NAT Gateway** - For private subnet outbound access
3. **VPN Connection** - Secure connection to on-premises
4. **VPC Peering** - Connect two VPCs directly
5. **VPC Endpoints** - Private connectivity to AWS services
6. **Transit Gateway** - Connect multiple VPCs and networks

### IP Addressing

1. **VPC CIDR** - Primary network block (e.g., 10.0.0.0/16)
2. **Secondary CIDR** - Additional IP ranges in same VPC
3. **Subnet CIDR** - Must be subset of VPC CIDR
4. **Elastic IP** - Static public IP address
5. **Private IP** - Instance IP within subnet

## Hands-on Lab: Create and Configure a VPC with Public and Private Subnets

### Objective
Create a functional VPC with both public and private subnets, internet access, and proper routing.

### Prerequisites for Lab
- AWS account with EC2 permissions
- AWS CLI configured with credentials
- Default region (us-east-1 recommended)

### Step 1: Create VPC

```bash
# Create VPC with CIDR block 10.0.0.0/16
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Output shows VPC ID and state
# Save VPC_ID from output
VPC_ID="vpc-xxxxxxxx"

# Add name tag to VPC
aws ec2 create-tags --resources $VPC_ID --tags Key=Name,Value=training-vpc
```

**Expected Output:**
```json
{
    "Vpc": {
        "CidrBlock": "10.0.0.0/16",
        "VpcId": "vpc-xxxxxxxx",
        "State": "available",
        "OwnerId": "123456789012"
    }
}
```

### Step 2: Create Internet Gateway

```bash
# Create Internet Gateway
aws ec2 create-internet-gateway

# Save IGW_ID from output
IGW_ID="igw-xxxxxxxx"

# Attach IGW to VPC
aws ec2 attach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID

# Add name tag
aws ec2 create-tags --resources $IGW_ID --tags Key=Name,Value=training-igw
```

**Expected Output:**
```json
{
    "InternetGateway": {
        "InternetGatewayId": "igw-xxxxxxxx",
        "Attachments": [],
        "Tags": []
    }
}
```

### Step 3: Create Public Subnet

```bash
# Create public subnet in AZ us-east-1a
aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a

# Save SUBNET_PUBLIC_ID
SUBNET_PUBLIC_ID="subnet-xxxxxxxx"

# Add name tag
aws ec2 create-tags --resources $SUBNET_PUBLIC_ID --tags Key=Name,Value=public-subnet

# Enable auto-assign public IP
aws ec2 modify-subnet-attribute \
  --subnet-id $SUBNET_PUBLIC_ID \
  --map-public-ip-on-launch
```

**Expected Output:**
```json
{
    "Subnet": {
        "VpcId": "vpc-xxxxxxxx",
        "CidrBlock": "10.0.1.0/24",
        "State": "available",
        "SubnetId": "subnet-xxxxxxxx",
        "AvailabilityZone": "us-east-1a"
    }
}
```

### Step 4: Create Private Subnet

```bash
# Create private subnet in AZ us-east-1b
aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1b

# Save SUBNET_PRIVATE_ID
SUBNET_PRIVATE_ID="subnet-xxxxxxxx"

# Add name tag
aws ec2 create-tags --resources $SUBNET_PRIVATE_ID --tags Key=Name,Value=private-subnet
```

**Expected Output:**
```json
{
    "Subnet": {
        "VpcId": "vpc-xxxxxxxx",
        "CidrBlock": "10.0.2.0/24",
        "State": "available",
        "SubnetId": "subnet-xxxxxxxx",
        "AvailabilityZone": "us-east-1b"
    }
}
```

### Step 5: Configure Route Tables

```bash
# Get the main route table (created with VPC)
ROUTE_TABLE_ID=$(aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query 'RouteTables[0].RouteTableId' \
  --output text)

# Add route to Internet Gateway
aws ec2 create-route \
  --route-table-id $ROUTE_TABLE_ID \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id $IGW_ID

# Associate route table with public subnet
aws ec2 associate-route-table \
  --subnet-id $SUBNET_PUBLIC_ID \
  --route-table-id $ROUTE_TABLE_ID

# Tag route table
aws ec2 create-tags --resources $ROUTE_TABLE_ID --tags Key=Name,Value=public-rt
```

**Expected Output (Create Route):**
```json
{
    "Return": true
}
```

### Step 6: Create and Configure NAT Gateway

```bash
# Allocate Elastic IP for NAT Gateway
EIP=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)

# Create NAT Gateway in public subnet
NAT_GATEWAY_ID=$(aws ec2 create-nat-gateway \
  --subnet-id $SUBNET_PUBLIC_ID \
  --allocation-id $EIP \
  --query 'NatGateway.NatGatewayId' \
  --output text)

# Wait for NAT Gateway to be available
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_GATEWAY_ID

# Tag NAT Gateway
aws ec2 create-tags --resources $NAT_GATEWAY_ID --tags Key=Name,Value=training-nat
```

**Expected Output:**
```json
{
    "NatGateway": {
        "NatGatewayId": "nat-xxxxxxxx",
        "SubnetId": "subnet-xxxxxxxx",
        "State": "pending",
        "NatGatewayAddresses": [
            {
                "PublicIp": "203.0.113.1",
                "AllocationId": "eipalloc-xxxxxxxx"
            }
        ]
    }
}
```

### Step 7: Create Route Table for Private Subnet

```bash
# Create private route table
PRIVATE_RT=$(aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --query 'RouteTable.RouteTableId' \
  --output text)

# Add route to NAT Gateway for internet access
aws ec2 create-route \
  --route-table-id $PRIVATE_RT \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id $NAT_GATEWAY_ID

# Associate with private subnet
aws ec2 associate-route-table \
  --subnet-id $SUBNET_PRIVATE_ID \
  --route-table-id $PRIVATE_RT

# Tag route table
aws ec2 create-tags --resources $PRIVATE_RT --tags Key=Name,Value=private-rt
```

**Expected Output:**
```json
{
    "Return": true
}
```

### Step 8: Verify VPC Configuration

```bash
# List all route tables
aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query 'RouteTables[*].[RouteTableId,Associations[*].SubnetId]' \
  --output table

# List all subnets
aws ec2 describe-subnets \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query 'Subnets[*].[SubnetId,CidrBlock,AvailabilityZone]' \
  --output table

# Verify Internet Gateway attachment
aws ec2 describe-internet-gateways \
  --filters Name=attachment.vpc-id,Values=$VPC_ID \
  --query 'InternetGateways[*].[InternetGatewayId,Attachments[*].State]' \
  --output table
```

**Expected Output:**
```
-------------------------------------------
|      DescribeRouteTables               |
-------------------------------------------
|  Route Table ID  |  Associated Subnets |
|-----------------|---------------------|
|  rtb-xxxxxxxx   |  subnet-xxxxxxxx    |
|  rtb-yyyyyyyy   |  subnet-yyyyyyyy    |
-------------------------------------------
```

## Validation

After completing this lab, validate your understanding by:

1. **VPC Created** - VPC exists with correct CIDR block (10.0.0.0/16)
2. **Subnets Functional** - Both public and private subnets created in different AZs
3. **Internet Access** - Public subnet can reach internet via IGW
4. **NAT Configuration** - Private subnet can reach internet via NAT Gateway
5. **Routing Correct** - Route tables properly configured and associated
6. **Tags Applied** - Resources have meaningful names

## Cleanup

```bash
# Delete NAT Gateway (takes a few minutes)
aws ec2 delete-nat-gateway --nat-gateway-id $NAT_GATEWAY_ID
aws ec2 wait nat-gateway-deleted --nat-gateway-ids $NAT_GATEWAY_ID

# Release Elastic IP
aws ec2 release-address --allocation-id $EIP

# Detach and delete Internet Gateway
aws ec2 detach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID
aws ec2 delete-internet-gateway --internet-gateway-id $IGW_ID

# Delete route tables
aws ec2 delete-route-table --route-table-id $ROUTE_TABLE_ID
aws ec2 delete-route-table --route-table-id $PRIVATE_RT

# Delete subnets
aws ec2 delete-subnet --subnet-id $SUBNET_PUBLIC_ID
aws ec2 delete-subnet --subnet-id $SUBNET_PRIVATE_ID

# Delete VPC
aws ec2 delete-vpc --vpc-id $VPC_ID

echo "VPC deletion complete"
```

## Common Mistakes

1. **Wrong CIDR Blocks** - Overlapping CIDRs cause routing issues
2. **Forgetting NAT Gateway** - Private subnets without NAT can't reach internet
3. **Missing Route** - Default route (0.0.0.0/0) not configured
4. **IGW Not Attached** - Internet Gateway created but not attached to VPC
5. **Wrong Availability Zone** - Putting all resources in one AZ loses redundancy
6. **No Public IPs** - Instances in public subnet need public IPs to be accessed
7. **Unused Elastic IPs** - Charged even when not associated
8. **Insufficient Subnet Capacity** - Planning /28 subnet when /24 needed

## Troubleshooting

### "Cannot Reach Instance" Error
**Cause:** No public IP, wrong security group, or routing issue
**Solution:**
- Verify instance has public IP: `aws ec2 describe-instances --query 'Reservations[0].Instances[0].PublicIpAddress'`
- Check security group allows port: `aws ec2 describe-security-groups --group-id sg-xxxxx`
- Verify route table has internet route: `aws ec2 describe-route-tables --route-table-id rtb-xxxxx`

### "Private Subnet Can't Reach Internet" Error
**Cause:** NAT Gateway not properly configured or route missing
**Solution:**
- Check NAT Gateway status: `aws ec2 describe-nat-gateways --nat-gateway-ids nat-xxxxx`
- Verify route table points to NAT: `aws ec2 describe-route-tables --route-table-id rtb-xxxxx`
- Ensure Elastic IP is associated: `aws ec2 describe-addresses`

### "Subnet Creation Failed - Invalid CIDR" Error
**Cause:** Subnet CIDR not within VPC CIDR block
**Solution:**
- Check VPC CIDR: `aws ec2 describe-vpcs --vpc-ids vpc-xxxxx`
- Ensure subnet CIDR is subset of VPC CIDR
- Verify no overlapping subnet ranges exist

### "IGW Attachment Failed" Error
**Cause:** IGW already attached to another VPC or VPC already has IGW
**Solution:**
- Check existing IGWs: `aws ec2 describe-internet-gateways`
- Only one IGW per VPC allowed
- Detach from other VPC before reusing

### "Route Creation Failed - Invalid Target" Error
**Cause:** Target ID doesn't exist or wrong resource type
**Solution:**
- Verify resource exists: `aws ec2 describe-internet-gateways --internet-gateway-ids igw-xxxxx`
- Use correct resource ID format (igw-, nat-, vpn-, etc.)
- Ensure resource is in same VPC

## Next Steps

1. **Module 10** - Learn about VPC Connectivity (VPN, Direct Connect, Transit Gateway)
2. **Module 11** - Study Kubernetes Networking (VPC for containerized workloads)
3. **Module 12** - Explore Network Security and Zero Trust (VPC security hardening)
4. **Practice:** Create multi-tier VPC architecture with public/private/database layers
5. **Real-World:** Implement VPC for production application with load balancing
