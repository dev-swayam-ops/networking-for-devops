# 10 - VPC Connectivity: VPN, Direct Connect, and Peering - Solutions

## Exercise 1: Query VPC Peering Connections

### Solution

**Command:**
```bash
# List all VPC peering connections
aws ec2 describe-vpc-peering-connections \
  --query 'VpcPeeringConnections[*].[VpcPeeringConnectionId,Status.Code,RequesterVpcInfo.VpcId,AccepterVpcInfo.VpcId]' \
  --output table
```

**Alternative - detailed format:**
```bash
aws ec2 describe-vpc-peering-connections \
  --output json | jq -r '.VpcPeeringConnections[] | 
  "ID: \(.VpcPeeringConnectionId) | Status: \(.Status.Code) | Requester: \(.RequesterVpcInfo.VpcId) | Accepter: \(.AccepterVpcInfo.VpcId)"'
```

**Filter for active connections only:**
```bash
aws ec2 describe-vpc-peering-connections \
  --filters Name=status-code,Values=active \
  --output table
```

**Expected Output:**
```
-------------------------------------------------------------------
|           DescribeVpcPeeringConnections                          |
|------------|--------|------------|----------|
| ID         | Status | Requester  | Accepter  |
|------------|--------|------------|----------|
| pcx-001    | active | vpc-0a1b2c | vpc-1b2c3d|
| pcx-002    | active | vpc-2c3d4e | vpc-3d4e5f|
| pcx-003    | pending| vpc-4e5f6g | vpc-5f6g7h|
-------------------------------------------------------------------
```

### Explanation
- `describe-vpc-peering-connections` lists all peering connections
- Status codes: active, pending-acceptance, failed, expired, provisioning, deleted
- Requester = VPC that initiated peering request
- Accepter = VPC that accepted the request
- Cross-account peering shows pending until accepter approves

---

## Exercise 2: Examine Peering Connection Routes

### Solution

**List Route Tables:**
```bash
VPC_ID="vpc-0a1b2c3d"  # Replace with your VPC ID

# Get all route tables
aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query 'RouteTables[*].[RouteTableId,Associations[*].SubnetId]' \
  --output table
```

**Show routes with peering connections:**
```bash
aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query 'RouteTables[*].[RouteTableId,Routes[?VpcPeeringConnectionId!=null].[DestinationCidrBlock,VpcPeeringConnectionId]]' \
  --output table
```

**Detailed view:**
```bash
aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=$VPC_ID \
  --output json | jq -r '.RouteTables[] | 
  "Route Table: \(.RouteTableId)\n" + 
  (.Routes[] | select(.VpcPeeringConnectionId != null) | 
  "  Destination: \(.DestinationCidrBlock) → Peering: \(.VpcPeeringConnectionId)") + "\n"'
```

**Expected Output:**
```
-------------------------------------------------------------------
|                    DescribeRouteTables                           |
|------------|--------|----------|----------|
| RouteTableId | SubnetId | Destination | Target |
|------------|--------|----------|----------|
| rtb-001  | subnet-a1  | 10.0.0.0/16 | local  |
| rtb-001  | subnet-a1  | 10.1.0.0/16 | pcx-001|
| rtb-002  | subnet-b1  | 10.1.0.0/16 | local  |
| rtb-002  | subnet-b1  | 10.0.0.0/16 | pcx-001|
-------------------------------------------------------------------
```

### Explanation
- Routes with VpcPeeringConnectionId show traffic destined for peer VPC
- Subnets associated with route table inherit these routes
- Local routes (same VPC CIDR) always have target "local"
- Peering routes must exist in both VPCs for bidirectional communication

---

## Exercise 3: List VPC Endpoints in a VPC

### Solution

**List all endpoints:**
```bash
VPC_ID="vpc-0a1b2c3d"

aws ec2 describe-vpc-endpoints \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query 'VpcEndpoints[*].[VpcEndpointId,VpcEndpointType,ServiceName,State]' \
  --output table
```

**Detailed endpoint information:**
```bash
aws ec2 describe-vpc-endpoints \
  --filters Name=vpc-id,Values=$VPC_ID \
  --output json | jq -r '.VpcEndpoints[] | 
  "Endpoint: \(.VpcEndpointId)\n" +
  "  Type: \(.VpcEndpointType)\n" +
  "  Service: \(.ServiceName)\n" +
  "  State: \(.State)\n" +
  "  Subnets: \(.SubnetIds | join(", "))\n"'
```

**Filter Gateway vs Interface endpoints:**
```bash
# Gateway endpoints (S3, DynamoDB)
aws ec2 describe-vpc-endpoints \
  --filters Name=vpc-id,Values=$VPC_ID Name=vpc-endpoint-type,Values=Gateway

# Interface endpoints (other services)
aws ec2 describe-vpc-endpoints \
  --filters Name=vpc-id,Values=$VPC_ID Name=vpc-endpoint-type,Values=Interface
```

**Expected Output:**
```
-----------------------------------------------------------
|              DescribeVpcEndpoints                       |
|---------|---------|------------|---------|
| ID      | Type    | Service    | State   |
|---------|---------|------------|---------|
| vpce-001| Gateway | s3         | available |
| vpce-002| Gateway | dynamodb   | available |
| vpce-003| Interface | ec2     | available |
| vpce-004| Interface | sns     | available |
-----------------------------------------------------------
```

### Explanation
- Gateway endpoints: S3 and DynamoDB (cheaper, via route table)
- Interface endpoints: All other AWS services (VPC PrivateLink)
- State = available means endpoint is ready for use
- No charges for data transfer through VPC endpoints vs. data transfer through NAT/IGW

---

## Exercise 4: Analyze VPN Configuration

### Solution

**List VPN Gateways:**
```bash
aws ec2 describe-vpn-gateways \
  --query 'VpnGateways[*].[VpnGatewayId,State,VpcAttachments[0].VpcId]' \
  --output table
```

**List VPN Connections:**
```bash
aws ec2 describe-vpn-connections \
  --query 'VpnConnections[*].[VpnConnectionId,State,Type,VpnGatewayId,CustomerGatewayId]' \
  --output table
```

**Detailed VPN configuration:**
```bash
VPN_ID="vpn-12345678"

aws ec2 describe-vpn-connections \
  --vpn-connection-ids $VPN_ID \
  --output json | jq '.VpnConnections[0] | {
    ConnectionId: .VpnConnectionId,
    State: .State,
    Type: .Type,
    VpnGateway: .VpnGatewayId,
    CustomerGateway: .CustomerGatewayId,
    VpnTunnels: [.VpnTunnelOptions[] | {
      Address: .OutsideIpAddress,
      Phase1EncryptionAlgorithm: .Phase1EncryptionAlgorithms[0].Value
    }]
  }'
```

**List Customer Gateways:**
```bash
aws ec2 describe-customer-gateways \
  --query 'CustomerGateways[*].[CustomerGatewayId,State,Type,PublicIp]' \
  --output table
```

**Expected Output:**
```
=== VPN Gateways ===
-----------------------------------
| Gateway ID | State | VPC    |
|------|--------|----------|
| vgw-001 | available | vpc-0a1b |

=== VPN Connections ===
-------------------------------------------
| ID     | State | Type | Gateway | CustGW |
|--------|-------|------|---------|--------|
| vpn-001| available | ipsec.1 | vgw-001 | cgw-001|

=== Customer Gateways ===
--------------------------------------------
| ID    | State | Type | IP           |
|-------|-------|------|--------------|
| cgw-001| available | ipsec.1 | 203.0.113.1 |
```

### Explanation
- VPN Gateway: AWS-side termination point attached to VPC
- Customer Gateway: On-premises side (router/firewall)
- VPN Connection: Link between the two gateways
- Type = ipsec.1 (standard IPSec protocol)
- State = available means VPN is operational
- Dual tunnels for redundancy (two physical tunnels)

---

## Exercise 5: Compare Peering Connection Costs

### Solution

**Cost Comparison (as of 2024):**

| Connectivity Method | Data Transfer Cost | Use Case |
|---|---|---|
| Same-region VPC Peering | FREE | Prod-Dev, Multi-tier apps in same region |
| Cross-region Peering | $0.02/GB | Disaster recovery, Multi-region replication |
| VPN (over internet) | FREE (internet costs apply) | Secure on-premises link |
| Direct Connect | $0.30/hour + $0.30/GB | High-volume hybrid, Guaranteed bandwidth |
| NAT Gateway | $0.045/hour + $0.045/GB | Private instance outbound (expensive) |

**Decision Matrix:**

```
Use VPC Peering if:
- VPCs in same region
- Non-sensitive workloads (peering is direct, not encrypted by default)
- High data transfer volumes
- Cost-sensitive

Use VPN if:
- On-premises to AWS connection needed
- Can tolerate internet path
- Security encryption required
- Lower bandwidth requirements

Use Direct Connect if:
- High consistent bandwidth (100+ Mbps)
- Predictable latency needed
- Cost justified by volume
- Mission-critical connectivity

Use Transit Gateway if:
- 3+ VPCs need interconnection
- Simplify hub-and-spoke architecture
- Mix of on-premises and AWS services
```

### Key Financial Insights
- Peering: ~$0 same-region, $0.02/GB cross-region
- VPN: $0 if under 4GB/month, competitive at low volumes
- NAT egress: $0.045/GB (most expensive!)
- Direct Connect: Flat fee at high volumes

---

## Exercise 6: Create VPC Peering Connection

### Solution

**Step 1: Create or Identify VPCs**
```bash
# Create VPC A
VPC_A=$(aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --query 'Vpc.VpcId' \
  --output text)

echo "VPC A: $VPC_A"

# Create VPC B
VPC_B=$(aws ec2 create-vpc \
  --cidr-block 10.1.0.0/16 \
  --query 'Vpc.VpcId' \
  --output text)

echo "VPC B: $VPC_B"
```

**Step 2: Create Peering Connection**
```bash
# Request peering from A to B
PEERING_ID=$(aws ec2 create-vpc-peering-connection \
  --vpc-id $VPC_A \
  --peer-vpc-id $VPC_B \
  --query 'VpcPeeringConnection.VpcPeeringConnectionId' \
  --output text)

echo "Peering ID: $PEERING_ID"
```

**Step 3: Accept Peering Connection**
```bash
# In same account, accept automatically
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id $PEERING_ID

# Verify acceptance
aws ec2 describe-vpc-peering-connections \
  --vpc-peering-connection-ids $PEERING_ID \
  --query 'VpcPeeringConnections[0].Status.Code' \
  --output text
```

**Step 4: Tag the Connection**
```bash
aws ec2 create-tags \
  --resources $PEERING_ID \
  --tags Key=Name,Value=vpc-a-to-vpc-b \
  --tags Key=environment,Value=training \
  --tags Key=purpose,Value=peering-test

# Verify tags
aws ec2 describe-vpc-peering-connections \
  --vpc-peering-connection-ids $PEERING_ID \
  --query 'VpcPeeringConnections[0].Tags'
```

**Expected Output:**
```
Peering ID: pcx-0a1b2c3d4e5f6g7h8
Status: active
Tags: [
  {"Key": "Name", "Value": "vpc-a-to-vpc-b"},
  {"Key": "environment", "Value": "training"},
  {"Key": "purpose", "Value": "peering-test"}
]
```

---

## Exercise 7: Configure Routes for Peering

### Solution

**Step 1: Get Route Table IDs**
```bash
# Get main route table for VPC A
RT_A=$(aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=$VPC_A \
  --query 'RouteTables[0].RouteTableId' \
  --output text)

echo "VPC A Route Table: $RT_A"

# Get main route table for VPC B
RT_B=$(aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=$VPC_B \
  --query 'RouteTables[0].RouteTableId' \
  --output text)

echo "VPC B Route Table: $RT_B"
```

**Step 2: Create Route in VPC A**
```bash
# Add route from VPC A to VPC B through peering
aws ec2 create-route \
  --route-table-id $RT_A \
  --destination-cidr-block 10.1.0.0/16 \
  --vpc-peering-connection-id $PEERING_ID

echo "Route created in VPC A: 10.1.0.0/16 → $PEERING_ID"
```

**Step 3: Create Route in VPC B**
```bash
# Add route from VPC B to VPC A through peering
aws ec2 create-route \
  --route-table-id $RT_B \
  --destination-cidr-block 10.0.0.0/16 \
  --vpc-peering-connection-id $PEERING_ID

echo "Route created in VPC B: 10.0.0.0/16 → $PEERING_ID"
```

**Step 4: Verify Routes**
```bash
echo "=== Routes in VPC A ==="
aws ec2 describe-route-tables \
  --route-table-ids $RT_A \
  --query 'RouteTables[0].Routes[*].[DestinationCidrBlock,VpcPeeringConnectionId,State]' \
  --output table

echo -e "\n=== Routes in VPC B ==="
aws ec2 describe-route-tables \
  --route-table-ids $RT_B \
  --query 'RouteTables[0].Routes[*].[DestinationCidrBlock,VpcPeeringConnectionId,State]' \
  --output table
```

**Expected Output:**
```
=== Routes in VPC A ===
---------------------------------------------------
| Destination | VPC Peering Connection | State  |
|------|---------|----------|
| 10.0.0.0/16 | local                  | active |
| 10.1.0.0/16 | pcx-0a1b2c3d4e5f6g7h8 | active |

=== Routes in VPC B ===
---------------------------------------------------
| Destination | VPC Peering Connection | State  |
|------|---------|----------|
| 10.1.0.0/16 | local                  | active |
| 10.0.0.0/16 | pcx-0a1b2c3d4e5f6g7h8 | active |
```

---

## Exercise 8: Create VPC Endpoint for S3

### Solution

**Step 1: Get Service Name for S3**
```bash
# Get S3 service name for your region
REGION=$(aws ec2 describe-availability-zones --query 'AvailabilityZones[0].RegionName' --output text)
S3_SERVICE="com.amazonaws.${REGION}.s3"

echo "S3 Service: $S3_SERVICE"
```

**Step 2: Get Route Table IDs**
```bash
VPC_ID="vpc-0a1b2c3d"
RT_IDS=$(aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=$VPC_ID \
  --query 'RouteTables[*].RouteTableId' \
  --output text)

echo "Route Tables: $RT_IDS"
```

**Step 3: Create Gateway VPC Endpoint**
```bash
# Create S3 endpoint
ENDPOINT_ID=$(aws ec2 create-vpc-endpoint \
  --vpc-id $VPC_ID \
  --service-name $S3_SERVICE \
  --route-table-ids $RT_IDS \
  --query 'VpcEndpoint.VpcEndpointId' \
  --output text)

echo "VPC Endpoint ID: $ENDPOINT_ID"
```

**Step 4: Create Endpoint Policy**
```bash
# Create policy file (allow all S3 actions)
cat > s3-endpoint-policy.json << 'EOF'
{
  "Statement": [
    {
      "Principal": "*",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::*",
        "arn:aws:s3:::*/*"
      ]
    }
  ]
}
EOF

# Apply policy to endpoint
aws ec2 modify-vpc-endpoint \
  --vpc-endpoint-id $ENDPOINT_ID \
  --policy-document file://s3-endpoint-policy.json
```

**Step 5: Verify Endpoint**
```bash
# Check endpoint status
aws ec2 describe-vpc-endpoints \
  --vpc-endpoint-ids $ENDPOINT_ID \
  --query 'VpcEndpoints[0].[VpcEndpointId,ServiceName,State,RouteTableIds]' \
  --output table
```

**Expected Output:**
```
-----------------------------------------------------------
| Endpoint ID | Service | State     | Route Tables |
|-------|---------|-----------|----------|
| vpce-001 | com.amazonaws.us-east-1.s3 | available | rtb-001, rtb-002 |
```

### Explanation
- Gateway endpoints added as routes in route tables
- No charge for S3 endpoint (part of S3 pricing)
- Instances can access S3 without NAT Gateway or IGW
- Endpoint policy can restrict bucket access

---

## Exercise 9: Configure Security Groups for Peering

### Solution

**Step 1: Create Security Group in VPC A**
```bash
SG_A=$(aws ec2 create-security-group \
  --group-name peering-sg-a \
  --description "Security group for peering in VPC A" \
  --vpc-id $VPC_A \
  --query 'GroupId' \
  --output text)

echo "SG A: $SG_A"
```

**Step 2: Add Ingress Rules to SG A**
```bash
# Allow ICMP (ping) from VPC B
aws ec2 authorize-security-group-ingress \
  --group-id $SG_A \
  --protocol icmp \
  --from-port -1 \
  --to-port -1 \
  --cidr 10.1.0.0/16

# Allow HTTPS (443) from VPC B
aws ec2 authorize-security-group-ingress \
  --group-id $SG_A \
  --protocol tcp \
  --port 443 \
  --cidr 10.1.0.0/16

# Allow SSH (22) from VPC B
aws ec2 authorize-security-group-ingress \
  --group-id $SG_A \
  --protocol tcp \
  --port 22 \
  --cidr 10.1.0.0/16

echo "Rules added to SG A"
```

**Step 3: Create Security Group in VPC B**
```bash
SG_B=$(aws ec2 create-security-group \
  --group-name peering-sg-b \
  --description "Security group for peering in VPC B" \
  --vpc-id $VPC_B \
  --query 'GroupId' \
  --output text)

echo "SG B: $SG_B"
```

**Step 4: Add Ingress Rules to SG B**
```bash
# Allow ICMP from VPC A
aws ec2 authorize-security-group-ingress \
  --group-id $SG_B \
  --protocol icmp \
  --from-port -1 \
  --to-port -1 \
  --cidr 10.0.0.0/16

# Allow HTTPS from VPC A
aws ec2 authorize-security-group-ingress \
  --group-id $SG_B \
  --protocol tcp \
  --port 443 \
  --cidr 10.0.0.0/16

# Allow SSH from VPC A
aws ec2 authorize-security-group-ingress \
  --group-id $SG_B \
  --protocol tcp \
  --port 22 \
  --cidr 10.0.0.0/16

echo "Rules added to SG B"
```

**Step 5: Verify Rules**
```bash
echo "=== Security Group A Rules ==="
aws ec2 describe-security-groups \
  --group-ids $SG_A \
  --query 'SecurityGroups[0].IpPermissions[*].[IpProtocol,FromPort,ToPort,IpRanges[0].CidrIp]' \
  --output table

echo -e "\n=== Security Group B Rules ==="
aws ec2 describe-security-groups \
  --group-ids $SG_B \
  --query 'SecurityGroups[0].IpPermissions[*].[IpProtocol,FromPort,ToPort,IpRanges[0].CidrIp]' \
  --output table
```

**Expected Output:**
```
=== Security Group A Rules ===
---------------------------------------------
| Protocol | Port | Source     |
|---|---------|----------|
| icmp | -1   | 10.1.0.0/16 |
| tcp  | 443  | 10.1.0.0/16 |
| tcp  | 22   | 10.1.0.0/16 |

=== Security Group B Rules ===
---------------------------------------------
| Protocol | Port | Source     |
|---|---------|----------|
| icmp | -1   | 10.0.0.0/16 |
| tcp  | 443  | 10.0.0.0/16 |
| tcp  | 22   | 10.0.0.0/16 |
```

---

## Exercise 10: Troubleshoot Peering Connection Issues

### Solutions and Diagnostics

#### Scenario A: Peering in "Initializing" State for 30+ Minutes

**Root Cause Analysis:**
```bash
# Check peering connection status
aws ec2 describe-vpc-peering-connections \
  --vpc-peering-connection-ids pcx-xxxxx \
  --query 'VpcPeeringConnections[0].[Status.Code,Status.Message]'

# Possible causes:
# 1. Invalid VPC IDs
# 2. Insufficient permissions
# 3. Account issues
# 4. AWS service problem
```

**Troubleshooting Steps:**
```bash
# 1. Verify VPC IDs are correct
aws ec2 describe-vpcs --vpc-ids vpc-xxxxx vpc-yyyyy

# 2. Check IAM permissions
aws iam get-user  # Verify you have ec2:CreateVpcPeeringConnection

# 3. Reject and recreate
aws ec2 reject-vpc-peering-connection --vpc-peering-connection-id pcx-xxxxx

# 4. Create new peering connection
aws ec2 create-vpc-peering-connection --vpc-id vpc-xxxxx --peer-vpc-id vpc-yyyyy
```

**Solutions:**
- Verify VPC IDs and account IDs are correct
- Check AWS service status for region
- For cross-account, ensure proper role/trust relationship
- Wait up to 5 minutes after rejection before retrying

---

#### Scenario B: Peering Active but Communication Fails

**Root Cause Analysis:**
```bash
# 1. Check route tables
aws ec2 describe-route-tables \
  --filters Name=association.subnet-id,Values=subnet-xxxxx \
  --query 'RouteTables[0].Routes'

# 2. Check security groups
aws ec2 describe-security-groups --group-ids sg-xxxxx

# 3. Check NACLs
aws ec2 describe-network-acls \
  --filters Name=association.subnet-id,Values=subnet-xxxxx

# 4. Verify instances are in correct subnet
aws ec2 describe-instances --instance-ids i-xxxxx
```

**Possible Causes & Solutions:**
```
1. Missing routes in route table
   → Add route to peer VPC via peering connection
   aws ec2 create-route --route-table-id rtb-xxxxx \
     --destination-cidr-block 10.1.0.0/16 --vpc-peering-connection-id pcx-xxxxx

2. Security group blocking traffic
   → Add inbound rule for peer CIDR
   aws ec2 authorize-security-group-ingress --group-id sg-xxxxx \
     --protocol tcp --port 443 --cidr 10.1.0.0/16

3. NACL rule blocking traffic
   → Add inbound/outbound rules to NACL
   aws ec2 create-network-acl-entry --network-acl-id acl-xxxxx \
     --rule-number 100 --protocol tcp --port-range From=443,To=443 \
     --cidr-block 10.1.0.0/16 --ingress

4. Instances in wrong subnet
   → Verify subnet is associated with correct route table
```

---

#### Scenario C: Cannot Peer VPC A (10.0.0.0/16) with VPC B (10.0.1.0/16)

**Root Cause:**
```bash
# CIDR overlap prevents peering!
# VPC A: 10.0.0.0/16 covers 10.0.0.0 - 10.0.255.255
# VPC B: 10.0.1.0/16 covers 10.0.1.0 - 10.0.1.255
# Overlap: 10.0.1.0/24 is in both!

aws ec2 describe-vpcs --query 'Vpcs[*].[VpcId,CidrBlock]'
```

**Error Message:**
```
InvalidParameterValue: Invalid value specified for VpcPeeringConnectionId: 'vpc-xxxxx'
Reason: CIDR blocks must not overlap
```

**Solutions:**
1. **Change VPC B CIDR** (if not in use):
```bash
# Delete VPC B and recreate with different CIDR
aws ec2 delete-vpc --vpc-id vpc-yyyyy
# Recreate with 10.1.0.0/16 (different network)
aws ec2 create-vpc --cidr-block 10.1.0.0/16
```

2. **Add Secondary CIDR** (if VPC is large):
```bash
# Add secondary CIDR to existing VPC
aws ec2 associate-vpc-cidr-block \
  --vpc-id vpc-yyyyy \
  --cidr-block 172.16.0.0/16
```

3. **Use Different Primary CIDR**:
```bash
# Best approach: Plan CIDR blocks to avoid overlap
# VPC A: 10.0.0.0/16
# VPC B: 10.1.0.0/16  (non-overlapping)
```

**Prevention:**
- Always plan CIDR blocks before creating VPCs
- Document CIDR ranges in a spreadsheet
- Use consistent subnetting scheme (e.g., VPC N uses 10.N.0.0/16)

---

## Summary of Key Commands

| Command | Purpose |
|---------|---------|
| `aws ec2 create-vpc-peering-connection` | Create peering request |
| `aws ec2 accept-vpc-peering-connection` | Accept peering (cross-account) |
| `aws ec2 describe-vpc-peering-connections` | List peering connections |
| `aws ec2 create-route` | Add peering route |
| `aws ec2 describe-vpc-endpoints` | List VPC endpoints |
| `aws ec2 create-vpc-endpoint` | Create S3/DynamoDB endpoint |
| `aws ec2 authorize-security-group-ingress` | Add security group rule |
| `aws ec2 describe-network-acls` | List network ACLs |
| `aws ec2 create-network-acl-entry` | Add NACL rule |
