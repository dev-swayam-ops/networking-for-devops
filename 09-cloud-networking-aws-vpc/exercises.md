# 09 - Cloud Networking: AWS VPC - Exercises

## Exercise 1: List All VPCs in Your Account

**Difficulty:** Easy

**Objective:** Get familiar with AWS CLI and querying VPC information

**Task:**
1. List all VPCs in your default region
2. Display VPC ID, CIDR block, and state
3. Identify the default VPC created by AWS

**Commands to Research:**
- `aws ec2 describe-vpcs`
- Query parameters for filtering

**Expected Result:**
- A table showing multiple VPCs
- Default VPC should be among them
- Each VPC shows its CIDR block and state

## Exercise 2: Examine VPC Components

**Difficulty:** Easy

**Objective:** Understand the components within a VPC

**Task:**
1. Choose one VPC from Exercise 1
2. List all subnets in that VPC
3. List all route tables in that VPC
4. List all Internet Gateways attached to that VPC

**Commands to Research:**
- `aws ec2 describe-subnets`
- `aws ec2 describe-route-tables`
- `aws ec2 describe-internet-gateways`
- Filtering with --filters parameter

**Expected Result:**
- Multiple subnets displayed with their CIDR blocks
- At least one route table per VPC
- IGW information if one is attached

## Exercise 3: Analyze Subnet Networking Details

**Difficulty:** Easy

**Objective:** Understand subnet structure and availability

**Task:**
1. Choose a VPC with multiple subnets
2. For each subnet, display:
   - Subnet ID
   - CIDR block
   - Availability Zone
   - Available IP count
3. Identify which subnets have public IP assignment enabled

**Commands to Research:**
- `aws ec2 describe-subnets --filters Name=vpc-id,Values=vpc-xxxxx`
- Query with available IP calculation

**Expected Result:**
- List of subnets with their networking details
- AZ distribution shown
- Public IP settings visible

## Exercise 4: Trace Network Routes

**Difficulty:** Easy

**Objective:** Understand routing within VPC

**Task:**
1. Choose a route table from a VPC
2. Display all routes in that table
3. For each route, identify:
   - Destination CIDR block
   - Target (IGW, NAT, local, etc.)
   - Route status

**Commands to Research:**
- `aws ec2 describe-route-tables`
- Route query parameters

**Expected Result:**
- All routes clearly displayed
- Local route visible (VPC CIDR)
- External routes (if any) to IGW or NAT

## Exercise 5: Query Security Group Rules

**Difficulty:** Easy

**Objective:** Understand security group ingress rules within VPC

**Task:**
1. List all security groups in a VPC
2. Choose one security group
3. Display all ingress rules including:
   - Protocol (TCP, UDP, etc.)
   - Port range
   - Source IP/CIDR

**Commands to Research:**
- `aws ec2 describe-security-groups`
- Ingress rules query

**Expected Result:**
- Security groups listed by ID and name
- Rules shown with clear source and port information
- Default rules visible

## Exercise 6: Create a New VPC from Scratch

**Difficulty:** Medium

**Objective:** Practice creating a complete VPC with proper structure

**Task:**
1. Create a new VPC with CIDR block 10.1.0.0/16
2. Create two subnets:
   - Public subnet: 10.1.1.0/24 in us-east-1a
   - Private subnet: 10.1.2.0/24 in us-east-1b
3. Create an Internet Gateway and attach to VPC
4. Tag all resources with environment=training
5. Verify all resources are created correctly

**Commands to Research:**
- `aws ec2 create-vpc`
- `aws ec2 create-subnet`
- `aws ec2 create-internet-gateway`
- `aws ec2 attach-internet-gateway`
- `aws ec2 create-tags`

**Expected Result:**
- VPC created and available
- Both subnets created in different AZs
- IGW attached to VPC
- All resources tagged

## Exercise 7: Configure Routing for Internet Access

**Difficulty:** Medium

**Objective:** Set up proper routing for public subnet internet access

**Task:**
1. Using the VPC from Exercise 6:
2. Get the main route table
3. Add a default route (0.0.0.0/0) pointing to the IGW
4. Associate the route table with the public subnet
5. Verify the route was created
6. Enable auto-assign public IP on the public subnet

**Commands to Research:**
- `aws ec2 create-route`
- `aws ec2 associate-route-table`
- `aws ec2 modify-subnet-attribute`

**Expected Result:**
- Route table shows 0.0.0.0/0 pointing to IGW
- Public subnet associated with route table
- Auto-assign public IP enabled

## Exercise 8: Set Up NAT Gateway for Private Subnet

**Difficulty:** Medium

**Objective:** Enable outbound internet access for private subnet

**Task:**
1. Using the VPC from Exercise 6:
2. Allocate an Elastic IP address
3. Create a NAT Gateway in the public subnet
4. Create a separate route table for private subnet
5. Add route 0.0.0.0/0 pointing to NAT Gateway
6. Associate with private subnet
7. Verify the configuration

**Commands to Research:**
- `aws ec2 allocate-address`
- `aws ec2 create-nat-gateway`
- `aws ec2 wait nat-gateway-available`
- `aws ec2 describe-nat-gateways`

**Expected Result:**
- Elastic IP allocated
- NAT Gateway in available state
- Route table for private subnet with NAT route
- Private subnet associated with route table

## Exercise 9: Create Network ACL Rules

**Difficulty:** Medium

**Objective:** Configure subnet-level network access controls

**Task:**
1. Get the default NACL for the VPC from Exercise 6
2. Create a custom NACL
3. Add inbound rule:
   - Allow HTTP (port 80) from 0.0.0.0/0
   - Allow HTTPS (port 443) from 0.0.0.0/0
   - Deny SSH (port 22) from 10.0.0.0/8 (internal range)
4. Associate with public subnet
5. List the rules to verify

**Commands to Research:**
- `aws ec2 describe-network-acls`
- `aws ec2 create-network-acl`
- `aws ec2 create-network-acl-entry`
- `aws ec2 associate-network-acl-subnet`

**Expected Result:**
- Custom NACL created
- Rules added with correct order numbers
- NACL associated with public subnet
- Rules visible when listing

## Exercise 10: Troubleshoot VPC Connectivity Issues

**Difficulty:** Medium

**Objective:** Identify and resolve common VPC connectivity problems

**Task:**
You have a VPC with these issues. For each, determine the problem and solution:

**Scenario A:**
- Private subnet instances cannot reach the internet
- NAT Gateway is in available state
- Public subnet internet access works fine
- What could be wrong?

**Scenario B:**
- Instances in public subnet cannot be accessed from outside
- Security group allows SSH (port 22) from 0.0.0.0/0
- Instances have public IPs
- What could be wrong?

**Scenario C:**
- Created subnet with CIDR 10.0.0.5/24 in VPC with CIDR 10.0.0.0/16
- Subnet creation failed
- What went wrong?

**Commands to Help Debug:**
- `aws ec2 describe-route-tables`
- `aws ec2 describe-network-acls`
- `aws ec2 describe-security-groups`
- `aws ec2 describe-subnets`

**Expected Result:**
- Identify root cause of each scenario
- Explain how to fix (specific AWS commands)
- Describe preventive measures

## Answer Guide Available

Solutions with complete commands and expected outputs are provided in `solutions.md`.

## Tips for Success

1. **Always Tag Resources** - Use environment and name tags for easy identification
2. **Plan CIDR Blocks** - Reserve adequate address space for each subnet
3. **Separate Concerns** - Keep public and private subnets in different route tables
4. **Monitor Routes** - Verify routing before deploying instances
5. **Use describe commands** - Verify state after each create/modify operation
6. **Take Notes** - Save VPC IDs and subnet IDs in variables during exercises
7. **Test Connectivity** - Create test instances to verify network configuration
8. **Review Errors** - Read AWS error messages carefully - they often describe the issue

## Learning Outcomes

After completing these exercises, you should understand:
- How to query and analyze VPC components using AWS CLI
- VPC creation and subnet design
- Internet Gateway and NAT Gateway configuration
- Route table management and troubleshooting
- Network ACL rules and security
- Common VPC configuration errors and fixes
