# 10 - VPC Connectivity: VPN, Direct Connect, and Peering - Exercises

## Exercise 1: Query VPC Peering Connections

**Difficulty:** Easy

**Objective:** Get familiar with peering connection details and status

**Task:**
1. List all VPC peering connections in your account
2. Display connection ID, status, and the VPCs involved
3. Identify which peering connections are active
4. Find any pending or failed connections

**Commands to Research:**
- `aws ec2 describe-vpc-peering-connections`
- Filtering by status
- Query parameters for connection details

**Expected Result:**
- Table showing all peering connections
- Status clearly visible (active, pending-acceptance, failed, etc.)
- Source and destination VPCs identified

## Exercise 2: Examine Peering Connection Routes

**Difficulty:** Easy

**Objective:** Understand how routes support peering connections

**Task:**
1. Choose a VPC with an active peering connection
2. List all route tables in that VPC
3. For each route table, identify:
   - Routes pointing to peering connections
   - Associated subnets
   - Traffic destinations through peering

**Commands to Research:**
- `aws ec2 describe-route-tables`
- Filtering by VPC and peering connection
- Query routes specifically

**Expected Result:**
- Route tables displayed with peering routes highlighted
- Each peering route shows destination CIDR and connection ID
- Associations between route tables and subnets clear

## Exercise 3: List VPC Endpoints in a VPC

**Difficulty:** Easy

**Objective:** Identify private service access points

**Task:**
1. Choose a VPC (or use default)
2. List all VPC endpoints in that VPC
3. Display endpoint details:
   - Endpoint type (Gateway vs Interface)
   - Service name (S3, DynamoDB, etc.)
   - VPC associations
   - Status

**Commands to Research:**
- `aws ec2 describe-vpc-endpoints`
- Filtering by VPC ID
- Query endpoint service names

**Expected Result:**
- Endpoints listed with type and service
- Gateway endpoints (S3, DynamoDB) identified
- Interface endpoints with their services shown

## Exercise 4: Analyze VPN Configuration

**Difficulty:** Easy

**Objective:** Understand VPN gateway setup

**Task:**
1. If your account has a VPN gateway, list all VPN gateways
2. For each VPN gateway, identify:
   - Gateway ID
   - Associated VPC
   - VPN connections attached
   - Gateway status

**Commands to Research:**
- `aws ec2 describe-vpn-gateways`
- `aws ec2 describe-vpn-connections`
- Filtering and association details

**Expected Result:**
- VPN gateways displayed with VPC association
- All connected VPN connections listed
- Status and configuration visible

## Exercise 5: Compare Peering Connection Costs

**Difficulty:** Easy

**Objective:** Understand cost implications of connectivity options

**Task:**
Without running commands (research-based):
1. Compare data transfer costs:
   - Same-region VPC peering
   - Cross-region VPC peering
   - Peering vs NAT Gateway egress
   - Peering vs VPN
2. Identify when peering is more cost-effective
3. List scenarios where VPN makes more sense

**Expected Result:**
- Cost comparison table
- Peering benefits identified
- Hybrid connectivity cost analysis

## Exercise 6: Create VPC Peering Connection

**Difficulty:** Medium

**Objective:** Hands-on peering connection setup

**Task:**
1. Using two existing VPCs (or create them):
   - VPC A: 10.0.0.0/16
   - VPC B: 10.1.0.0/16
2. Create a VPC peering connection from A to B
3. Accept the peering connection
4. Tag the connection with environment and name
5. Verify the connection is active

**Commands to Research:**
- `aws ec2 create-vpc-peering-connection`
- `aws ec2 accept-vpc-peering-connection`
- `aws ec2 create-tags` for peering resources

**Expected Result:**
- Peering connection created with unique ID
- Status shows as active
- Resources properly tagged
- Can list the new peering connection

## Exercise 7: Configure Routes for Peering

**Difficulty:** Medium

**Objective:** Enable traffic flow through peering

**Task:**
1. Using peering from Exercise 6:
2. Get route table IDs for both VPCs
3. Create route in VPC A route table:
   - Destination: 10.1.0.0/16
   - Target: Peering connection
4. Create route in VPC B route table:
   - Destination: 10.0.0.0/16
   - Target: Peering connection
5. Verify routes are created and active

**Commands to Research:**
- `aws ec2 describe-route-tables`
- `aws ec2 create-route` with peering connection
- Route table association details

**Expected Result:**
- Both route tables show peering routes
- Routes are in active state
- Traffic can be routed between VPCs

## Exercise 8: Create VPC Endpoint for S3

**Difficulty:** Medium

**Objective:** Set up private access to AWS services

**Task:**
1. Choose a VPC
2. Create a Gateway VPC Endpoint for S3:
   - Service name: com.amazonaws.region.s3
   - Select route tables to associate
3. Create endpoint policy allowing GetObject and PutObject
4. Verify endpoint is created
5. Check endpoint association with route tables

**Commands to Research:**
- `aws ec2 create-vpc-endpoint`
- Endpoint policy JSON structure
- Service name format for S3

**Expected Result:**
- VPC endpoint created and available
- S3 service accessible without NAT/IGW
- Endpoint routes added to route tables
- Policy restricts access appropriately

## Exercise 9: Configure Security Groups for Peering

**Difficulty:** Medium

**Objective:** Ensure traffic can flow across peering

**Task:**
1. Using peering and instances from previous exercises:
2. Create security group in VPC A:
   - Allow ICMP (ping) from VPC B CIDR
   - Allow TCP 443 (HTTPS) from VPC B CIDR
3. Create security group in VPC B:
   - Allow ICMP from VPC A CIDR
   - Allow TCP 443 from VPC A CIDR
4. Apply groups to test instances
5. Verify rules permit cross-VPC communication

**Commands to Research:**
- `aws ec2 create-security-group`
- `aws ec2 authorize-security-group-ingress` with CIDR blocks
- Security group rule listing

**Expected Result:**
- Security groups created in both VPCs
- Ingress rules allow communication between CIDR blocks
- Rules are visible when querying security groups

## Exercise 10: Troubleshoot Peering Connection Issues

**Difficulty:** Medium

**Objective:** Diagnose and fix connectivity problems

**Task:**
You have peering connections that are not working. For each scenario, determine the problem and fix:

**Scenario A:**
- Peering connection is in "initializing" state for 30+ minutes
- VPC IDs are correct
- What could be wrong? How to fix?

**Scenario B:**
- Peering connection is active
- Route table has route to peering connection (active)
- Traffic between VPCs still fails
- What could be blocking communication?

**Scenario C:**
- Trying to peer VPC A (10.0.0.0/16) with VPC B (10.0.1.0/16)
- Peering creation fails immediately
- What is the root cause?

**Commands to Help:**
- `aws ec2 describe-vpc-peering-connections`
- `aws ec2 describe-route-tables`
- `aws ec2 describe-security-groups`
- `aws ec2 describe-network-acls`

**Expected Result:**
- Identify root cause for each scenario
- Explain the solution with specific commands
- Describe preventive measures

## Answer Guide Available

Solutions with complete commands and expected outputs are provided in `solutions.md`.

## Tips for Success

1. **Test with Instances** - Deploy EC2 instances to actually test peering
2. **Use Ping (ICMP)** - Simple way to verify connectivity
3. **Check All Layers** - Routes, security groups, NACLs
4. **Document State** - Save IDs and details as you work
5. **Verify After Each Step** - Ensure each component is correct
6. **Tag Resources** - Makes tracking and cleanup easier
7. **Start Simple** - Single peering before multi-VPC designs
8. **Review Error Messages** - AWS errors often explain the issue

## Learning Outcomes

After completing these exercises, you should understand:
- How to query and manage VPC peering connections
- VPC endpoint setup and use cases
- Route configuration for peering
- Security group and NACL implications
- Troubleshooting connectivity issues
- Cost implications of different connectivity options
- When to use peering vs VPN vs Direct Connect
