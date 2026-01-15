# 10 - VPC Connectivity: VPN, Direct Connect, and Peering

## What You'll Learn

By the end of this module, you'll be able to:
- Understand VPC peering architecture and use cases
- Set up and manage VPC peering connections
- Configure site-to-site VPN connections
- Work with VPN connections and customer gateways
- Understand AWS Direct Connect basics
- Implement VPC endpoints for private service access
- Use AWS Transit Gateway for multi-VPC connectivity
- Monitor and troubleshoot VPC connectivity
- Design hybrid cloud network architectures
- Choose the right connectivity solution for your needs

## Prerequisites

- Completion of modules 00-09
- Understanding of VPCs, subnets, and routing
- Familiarity with public/private IP addressing
- AWS account with appropriate permissions
- AWS CLI installed and configured
- Command-line proficiency

## Key Concepts

### VPC Peering Fundamentals

1. **VPC Peering** - Direct connection between two VPCs
2. **Peering Connection** - Network link enabling private communication
3. **Transitive Peering** - Peering is not transitive by default
4. **Route Tables** - Must be updated with peering routes
5. **Security Groups** - Must allow traffic for peering

### Peering Characteristics

1. **Same Region Peering** - Lower latency, no data transfer cost
2. **Cross-Region Peering** - Same VPC concept, higher latency
3. **Requester and Accepter** - One requests, one accepts
4. **CIDR Overlap** - Cannot peer VPCs with overlapping CIDRs
5. **DNS Resolution** - Can enable private DNS names across peers

### Site-to-Site VPN Concepts

1. **VPN Gateway** - AWS-side VPN termination point
2. **Customer Gateway** - On-premises side VPN termination
3. **VPN Connection** - Link between gateways
4. **VPN Tunnel** - Redundant encrypted tunnels (dual tunnels)
5. **BGP** - Border Gateway Protocol for dynamic routing

### VPN Components

1. **Virtual Private Gateway** - Deployed in VPC
2. **Customer Gateway** - On-premises router/device
3. **VPN Tunnel** - Encrypted connection (2 for redundancy)
4. **Route Propagation** - Dynamic or static routes
5. **Encryption** - IPSec encryption of traffic

### AWS Direct Connect

1. **Dedicated Connection** - Dedicated network link to AWS
2. **Hosting Connection** - Connection through AWS partner
3. **Virtual Interfaces** - Logical connections on Link
4. **BGP** - Dynamic routing over Direct Connect
5. **Private/Public VIF** - VPC access vs AWS public services

### Direct Connect Characteristics

1. **Dedicated Bandwidth** - 1 Gbps, 10 Gbps, 100 Gbps options
2. **Consistent Network Performance** - Predictable latency
3. **Lower Latency** - Direct path vs internet
4. **Cost** - Data transfer cost reduction
5. **Redundancy** - Requires dual connections for HA

### VPC Endpoints

1. **Gateway Endpoints** - For S3 and DynamoDB
2. **Interface Endpoints** - For other AWS services
3. **Endpoint Policies** - Control access to services
4. **Private DNS** - Optional DNS names
5. **No Internet Gateway** - Private access without IGW

### Transit Gateway

1. **Central Hub** - Connects multiple VPCs and networks
2. **Attachments** - VPCs, VPN, and Direct Connect links
3. **Route Tables** - Manage traffic between attachments
4. **Scaling** - Simplified multi-VPC architecture
5. **On-Premises Integration** - Single point for hybrid connectivity

## Hands-on Lab: Set Up VPC Peering Connection

### Objective
Create two VPCs and establish a peering connection with proper routing to enable private communication between instances in each VPC.

### Prerequisites for Lab
- AWS account with EC2 permissions
- AWS CLI configured with credentials
- Two VPCs available or ability to create them
- Subnets in each VPC for testing

### Step 1: Create VPCs and Subnets

```bash
# Create first VPC (VPC A)
VPC_A=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query 'Vpc.VpcId' --output text)
echo "VPC A: $VPC_A"

# Create subnet in VPC A
SUBNET_A=$(aws ec2 create-subnet \
  --vpc-id $VPC_A \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --query 'Subnet.SubnetId' \
  --output text)

echo "Subnet A: $SUBNET_A"

# Create second VPC (VPC B)
VPC_B=$(aws ec2 create-vpc --cidr-block 10.1.0.0/16 --query 'Vpc.VpcId' --output text)
echo "VPC B: $VPC_B"

# Create subnet in VPC B
SUBNET_B=$(aws ec2 create-subnet \
  --vpc-id $VPC_B \
  --cidr-block 10.1.1.0/24 \
  --availability-zone us-east-1a \
  --query 'Subnet.SubnetId' \
  --output text)

echo "Subnet B: $SUBNET_B"

# Tag resources
aws ec2 create-tags --resources $VPC_A --tags Key=Name,Value=vpc-a
aws ec2 create-tags --resources $VPC_B --tags Key=Name,Value=vpc-b
aws ec2 create-tags --resources $SUBNET_A --tags Key=Name,Value=subnet-a
aws ec2 create-tags --resources $SUBNET_B --tags Key=Name,Value=subnet-b
```

**Expected Output:**
```
VPC A: vpc-0a1b2c3d4e5f6g7h8
Subnet A: subnet-0a1b2c3d4e5f6g7h8
VPC B: vpc-1b2c3d4e5f6g7h8i9
Subnet B: subnet-1b2c3d4e5f6g7h8i9
```

### Step 2: Create Peering Connection

```bash
# Request peering from VPC A to VPC B
PEERING_ID=$(aws ec2 create-vpc-peering-connection \
  --vpc-id $VPC_A \
  --peer-vpc-id $VPC_B \
  --query 'VpcPeeringConnection.VpcPeeringConnectionId' \
  --output text)

echo "Peering Connection ID: $PEERING_ID"

# Accept peering connection (in same account, so auto-accept alternative)
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id $PEERING_ID

# Tag peering connection
aws ec2 create-tags \
  --resources $PEERING_ID \
  --tags Key=Name,Value=vpc-a-to-vpc-b
```

**Expected Output:**
```
Peering Connection ID: pcx-0a1b2c3d4e5f6g7h8
```

### Step 3: Update Route Tables

```bash
# Get or create route table for VPC A
RT_A=$(aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=$VPC_A \
  --query 'RouteTables[0].RouteTableId' \
  --output text)

echo "Route Table A: $RT_A"

# Add route in VPC A to VPC B through peering connection
aws ec2 create-route \
  --route-table-id $RT_A \
  --destination-cidr-block 10.1.0.0/16 \
  --vpc-peering-connection-id $PEERING_ID

# Get or create route table for VPC B
RT_B=$(aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=$VPC_B \
  --query 'RouteTables[0].RouteTableId' \
  --output text)

echo "Route Table B: $RT_B"

# Add route in VPC B to VPC A through peering connection
aws ec2 create-route \
  --route-table-id $RT_B \
  --destination-cidr-block 10.0.0.0/16 \
  --vpc-peering-connection-id $PEERING_ID
```

**Expected Output:**
```
Route Table A: rtb-0a1b2c3d4e5f6g7h8
Route Table B: rtb-1b2c3d4e5f6g7h8i9
```

### Step 4: Verify Peering Connection

```bash
# Check peering connection status
aws ec2 describe-vpc-peering-connections \
  --vpc-peering-connection-ids $PEERING_ID \
  --query 'VpcPeeringConnections[0].[VpcPeeringConnectionId,Status.Code,RequesterVpcInfo.VpcId,AccepterVpcInfo.VpcId]' \
  --output table

# Verify routes are in place
echo "Routes in VPC A:"
aws ec2 describe-route-tables \
  --route-table-ids $RT_A \
  --query 'RouteTables[0].Routes[*].[DestinationCidrBlock,VpcPeeringConnectionId]' \
  --output table

echo "Routes in VPC B:"
aws ec2 describe-route-tables \
  --route-table-ids $RT_B \
  --query 'RouteTables[0].Routes[*].[DestinationCidrBlock,VpcPeeringConnectionId]' \
  --output table
```

**Expected Output:**
```
Peering Connection Status:
--------------------------------------------------
| Connection ID        | Status | From    | To   |
|------|---------|----------|
| pcx-0a1b2c3d... | active | vpc-0a1b... | vpc-1b2c... |

Routes in VPC A:
------------------------------------------
| Destination | Target |
|------|---------|
| 10.0.0.0/16 | local  |
| 10.1.0.0/16 | pcx-0a1b2c... |

Routes in VPC B:
------------------------------------------
| Destination | Target |
|------|---------|
| 10.1.0.0/16 | local  |
| 10.0.0.0/16 | pcx-0a1b2c... |
```

### Step 5: Create Security Groups for Testing

```bash
# Create security group in VPC A
SG_A=$(aws ec2 create-security-group \
  --group-name peering-test-a \
  --description "Security group for VPC A peering test" \
  --vpc-id $VPC_A \
  --query 'GroupId' \
  --output text)

echo "Security Group A: $SG_A"

# Allow ICMP (ping) and SSH from VPC B
aws ec2 authorize-security-group-ingress \
  --group-id $SG_A \
  --protocol icmp \
  --from-port -1 \
  --to-port -1 \
  --cidr 10.1.0.0/16

aws ec2 authorize-security-group-ingress \
  --group-id $SG_A \
  --protocol tcp \
  --port 22 \
  --cidr 10.1.0.0/16

# Create security group in VPC B
SG_B=$(aws ec2 create-security-group \
  --group-name peering-test-b \
  --description "Security group for VPC B peering test" \
  --vpc-id $VPC_B \
  --query 'GroupId' \
  --output text)

echo "Security Group B: $SG_B"

# Allow ICMP and SSH from VPC A
aws ec2 authorize-security-group-ingress \
  --group-id $SG_B \
  --protocol icmp \
  --from-port -1 \
  --to-port -1 \
  --cidr 10.0.0.0/16

aws ec2 authorize-security-group-ingress \
  --group-id $SG_B \
  --protocol tcp \
  --port 22 \
  --cidr 10.0.0.0/16
```

**Expected Output:**
```
Security Group A: sg-0a1b2c3d4e5f6g7h8
Security Group B: sg-1b2c3d4e5f6g7h8i9
```

### Step 6: Create Test Instances

```bash
# Note: This requires a valid AMI ID for your region
# Using Amazon Linux 2 AMI (replace with your region's AMI)
AMI_ID="ami-0c55b159cbfafe1f0"  # Update for your region

# Create instance in VPC A
INSTANCE_A=$(aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t2.micro \
  --subnet-id $SUBNET_A \
  --security-group-ids $SG_A \
  --query 'Instances[0].InstanceId' \
  --output text)

echo "Instance A: $INSTANCE_A"

# Create instance in VPC B
INSTANCE_B=$(aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t2.micro \
  --subnet-id $SUBNET_B \
  --security-group-ids $SG_B \
  --query 'Instances[0].InstanceId' \
  --output text)

echo "Instance B: $INSTANCE_B"

# Wait for instances to be running
aws ec2 wait instance-running --instance-ids $INSTANCE_A $INSTANCE_B

# Get private IPs
IP_A=$(aws ec2 describe-instances --instance-ids $INSTANCE_A \
  --query 'Reservations[0].Instances[0].PrivateIpAddress' --output text)
IP_B=$(aws ec2 describe-instances --instance-ids $INSTANCE_B \
  --query 'Reservations[0].Instances[0].PrivateIpAddress' --output text)

echo "Instance A Private IP: $IP_A"
echo "Instance B Private IP: $IP_B"
```

**Expected Output:**
```
Instance A: i-0a1b2c3d4e5f6g7h8
Instance B: i-1b2c3d4e5f6g7h8i9
Instance A Private IP: 10.0.1.10
Instance B Private IP: 10.1.1.10
```

### Step 7: Test Peering Connection

```bash
# From Instance A, ping Instance B (via Systems Manager Session Manager or EC2 Instance Connect)
# aws ssm start-session --target $INSTANCE_A

# Inside the instance:
# ping $IP_B

# Alternative: Use EC2 Reachability Analyzer (newer feature)
aws ec2 create-network-insights-path \
  --source $INSTANCE_A \
  --destination $INSTANCE_B \
  --protocol tcp

# Run analysis
aws ec2 run-network-insights-analysis \
  --network-insights-path-id insights-path-xxxxx
```

**Expected Output:**
```
Reachability Status: REACHABLE
Connection Details:
- Path: i-0a1b2c3d... → pcx-0a1b2c3d... → i-1b2c3d4e...
- Hops: Routing → Peering → Routing
- Network ACLs: Allow
- Security Groups: Allow
```

### Step 8: Verify Connectivity Details

```bash
# Summarize peering setup
echo "=== VPC A Details ==="
aws ec2 describe-vpcs --vpc-ids $VPC_A --output table

echo -e "\n=== VPC B Details ==="
aws ec2 describe-vpcs --vpc-ids $VPC_B --output table

echo -e "\n=== Peering Connection Status ==="
aws ec2 describe-vpc-peering-connections \
  --vpc-peering-connection-ids $PEERING_ID \
  --output table

echo -e "\n=== Routes in Route Table A ==="
aws ec2 describe-route-tables --route-table-ids $RT_A --output table

echo -e "\n=== Routes in Route Table B ==="
aws ec2 describe-route-tables --route-table-ids $RT_B --output table
```

**Expected Output:**
```
=== VPC A Details ===
VPC ID: vpc-0a1b2c3d4e5f6g7h8
CIDR: 10.0.0.0/16
State: available

=== Peering Connection Status ===
Connection ID: pcx-0a1b2c3d4e5f6g7h8
Status: active
State: active
```

## Validation

After completing this lab, validate your understanding by:

1. **Peering Created** - Peering connection exists in active state
2. **Routes Configured** - Both VPCs have routes through peering connection
3. **Security Groups Allow** - Rules permit traffic across CIDR ranges
4. **Connectivity Works** - Instances can communicate across VPCs
5. **No Internet Gateway Required** - Traffic stays private

## Cleanup

```bash
# Delete test instances
aws ec2 terminate-instances --instance-ids $INSTANCE_A $INSTANCE_B

# Wait for termination
aws ec2 wait instance-terminated --instance-ids $INSTANCE_A $INSTANCE_B

# Delete peering connection
aws ec2 delete-vpc-peering-connection --vpc-peering-connection-id $PEERING_ID

# Delete security groups
aws ec2 delete-security-group --group-id $SG_A
aws ec2 delete-security-group --group-id $SG_B

# Delete route tables (if not main)
aws ec2 delete-route-table --route-table-id $RT_A
aws ec2 delete-route-table --route-table-id $RT_B

# Delete subnets
aws ec2 delete-subnet --subnet-id $SUBNET_A
aws ec2 delete-subnet --subnet-id $SUBNET_B

# Delete VPCs
aws ec2 delete-vpc --vpc-id $VPC_A
aws ec2 delete-vpc --vpc-id $VPC_B

echo "Cleanup complete"
```

## Common Mistakes

1. **Overlapping CIDR Blocks** - Peering fails if VPC CIDRs overlap
2. **Missing Routes** - Adding peering but not updating route tables
3. **Security Group Rules** - Forgetting to allow traffic from peer VPC
4. **Transitive Peering** - Assuming A↔B and B↔C means A↔C (not true)
5. **NACL Rules** - Network ACLs may block peering traffic
6. **Wrong Region** - Peering is region-specific by default
7. **Route Confusion** - Using wrong CIDR ranges in routes
8. **Peering Not Accepted** - Cross-account peering requires explicit acceptance

## Troubleshooting

### "Cannot Communicate Between VPCs" Error
**Cause:** Missing route, security group rule, or NACL rule
**Solution:**
```bash
# Verify route exists
aws ec2 describe-route-tables --route-table-ids rtb-xxxxx \
  --query 'RouteTables[0].Routes'

# Verify peering connection is active
aws ec2 describe-vpc-peering-connections --vpc-peering-connection-ids pcx-xxxxx

# Check security groups allow traffic
aws ec2 describe-security-groups --group-ids sg-xxxxx

# Check NACLs allow traffic
aws ec2 describe-network-acls \
  --filters Name=association.subnet-id,Values=subnet-xxxxx
```

### "VPC Peering Connection Failed" Error
**Cause:** Invalid VPC IDs, overlapping CIDR, or account issues
**Solution:**
```bash
# Verify VPC IDs exist
aws ec2 describe-vpcs --vpc-ids vpc-xxxxx vpc-yyyyy

# Check CIDR blocks don't overlap
aws ec2 describe-vpcs --query 'Vpcs[*].[VpcId,CidrBlock]'

# In cross-account scenario, verify acceptance
aws ec2 describe-vpc-peering-connections \
  --query 'VpcPeeringConnections[*].[VpcPeeringConnectionId,Status.Code]'
```

### "Peering Request Pending" Error
**Cause:** Peering not yet accepted (cross-account)
**Solution:**
```bash
# List pending peering connections
aws ec2 describe-vpc-peering-connections \
  --filters Name=status-code,Values=pending-acceptance

# Accept if you're the accepter
aws ec2 accept-vpc-peering-connection --vpc-peering-connection-id pcx-xxxxx

# Or reject if not needed
aws ec2 reject-vpc-peering-connection --vpc-peering-connection-id pcx-xxxxx
```

### "Transitive Peering Not Working" Error
**Cause:** Expecting A→B→C communication (not supported)
**Solution:**
- Each VPC must have direct peering with other VPCs it communicates with
- Use Transit Gateway for hub-and-spoke topology
- If needing multi-VPC connectivity, implement Transit Gateway instead

## Next Steps

1. **Module 11** - Study Kubernetes Networking (containerized workloads in VPCs)
2. **Module 12** - Learn Network Security and Zero Trust architectures
3. **Module 13** - Practice troubleshooting and debugging techniques
4. **Real-World Labs** - Implement site-to-site VPN with on-premises simulation
5. **Advanced:** Explore AWS Transit Gateway for multi-VPC architectures
6. **Production Setup** - Design hybrid cloud connectivity for enterprise workloads
