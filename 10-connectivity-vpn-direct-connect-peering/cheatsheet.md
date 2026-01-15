# 10 - VPC Connectivity: VPN, Direct Connect, and Peering - Cheatsheet

## VPC Peering Commands

### Create and Manage Peering

| Command | Purpose | Example |
|---------|---------|---------|
| `aws ec2 create-vpc-peering-connection` | Create peering request | `aws ec2 create-vpc-peering-connection --vpc-id vpc-aaa --peer-vpc-id vpc-bbb` |
| `aws ec2 accept-vpc-peering-connection` | Accept peering (cross-account) | `aws ec2 accept-vpc-peering-connection --vpc-peering-connection-id pcx-xxxxx` |
| `aws ec2 reject-vpc-peering-connection` | Reject peering request | `aws ec2 reject-vpc-peering-connection --vpc-peering-connection-id pcx-xxxxx` |
| `aws ec2 delete-vpc-peering-connection` | Delete peering connection | `aws ec2 delete-vpc-peering-connection --vpc-peering-connection-id pcx-xxxxx` |
| `aws ec2 describe-vpc-peering-connections` | List peering connections | `aws ec2 describe-vpc-peering-connections --output table` |
| `aws ec2 create-tags` | Tag peering connection | `aws ec2 create-tags --resources pcx-xxxxx --tags Key=Name,Value=vpc-peering` |

### Query Peering Status

| Command | Purpose | Example |
|---------|---------|---------|
| Filter by status | Get only active peerings | `aws ec2 describe-vpc-peering-connections --filters Name=status-code,Values=active` |
| Filter by VPC | Get peerings for VPC | `aws ec2 describe-vpc-peering-connections --filters Name=requester-vpc-info.vpc-id,Values=vpc-xxxxx` |
| Query specific fields | Get IDs and status | `--query 'VpcPeeringConnections[*].[VpcPeeringConnectionId,Status.Code]'` |
| Format as table | Human-readable output | `--output table` |
| Format as JSON | Machine-readable output | `--output json` |

---

## Peering Routes

### Add Peering Routes

| Command | Purpose | Example |
|---------|---------|---------|
| `aws ec2 create-route` | Add route through peering | `aws ec2 create-route --route-table-id rtb-xxxxx --destination-cidr-block 10.1.0.0/16 --vpc-peering-connection-id pcx-xxxxx` |
| `aws ec2 describe-route-tables` | List route tables | `aws ec2 describe-route-tables --filters Name=vpc-id,Values=vpc-xxxxx` |
| `aws ec2 delete-route` | Remove peering route | `aws ec2 delete-route --route-table-id rtb-xxxxx --destination-cidr-block 10.1.0.0/16` |
| `aws ec2 associate-route-table` | Associate RT with subnet | `aws ec2 associate-route-table --subnet-id subnet-xxxxx --route-table-id rtb-xxxxx` |

### Query Routes

| Query | Purpose | Example |
|---------|---------|---------|
| Show all routes | Display all routes in VPC | `aws ec2 describe-route-tables --filters Name=vpc-id,Values=vpc-xxxxx --output table` |
| Show peering routes | Find routes to peering | `--query 'RouteTables[*].Routes[?VpcPeeringConnectionId!=null]'` |
| Show active routes | Filter by state | `--query 'RouteTables[*].Routes[?State==\`active\`]'` |

---

## VPC Endpoints

### Gateway Endpoints (S3, DynamoDB)

| Command | Purpose | Example |
|---------|---------|---------|
| `aws ec2 create-vpc-endpoint` | Create endpoint | `aws ec2 create-vpc-endpoint --vpc-id vpc-xxxxx --service-name com.amazonaws.region.s3 --route-table-ids rtb-xxxxx` |
| `aws ec2 describe-vpc-endpoints` | List endpoints | `aws ec2 describe-vpc-endpoints --filters Name=vpc-id,Values=vpc-xxxxx` |
| `aws ec2 modify-vpc-endpoint` | Update endpoint policy | `aws ec2 modify-vpc-endpoint --vpc-endpoint-id vpce-xxxxx --policy-document file://policy.json` |
| `aws ec2 delete-vpc-endpoint` | Delete endpoint | `aws ec2 delete-vpc-endpoint --vpc-endpoint-id vpce-xxxxx` |

### Interface Endpoints (EC2, SNS, Lambda, etc.)

| Command | Purpose | Example |
|---------|---------|---------|
| Create interface endpoint | EC2 service access | `aws ec2 create-vpc-endpoint --vpc-id vpc-xxxxx --service-name com.amazonaws.region.ec2 --vpc-endpoint-type Interface --subnet-ids subnet-xxxxx` |
| Attach security group | Control interface access | `aws ec2 modify-vpc-endpoint --vpc-endpoint-id vpce-xxxxx --add-security-group-ids sg-xxxxx` |
| Enable private DNS | Use private domain names | `aws ec2 modify-vpc-endpoint --vpc-endpoint-id vpce-xxxxx --private-dns-enabled` |

### Query Endpoints

| Command | Purpose | Example |
|---------|---------|---------|
| List by type | Gateway vs Interface | `--filters Name=vpc-endpoint-type,Values=Gateway` |
| List by service | Find specific service | `--filters Name=service-name,Values=*s3*` |
| Query state | Check endpoint status | `--query 'VpcEndpoints[*].[VpcEndpointId,State,ServiceName]'` |

---

## VPN Connections

### Create VPN Components

| Command | Purpose | Example |
|---------|---------|---------|
| `aws ec2 create-vpn-gateway` | Create VPN gateway | `aws ec2 create-vpn-gateway --type ipsec.1` |
| `aws ec2 create-customer-gateway` | Create on-prem gateway | `aws ec2 create-customer-gateway --type ipsec.1 --public-ip 203.0.113.1 --bgp-asn 65000` |
| `aws ec2 create-vpn-connection` | Create VPN connection | `aws ec2 create-vpn-connection --type ipsec.1 --customer-gateway-id cgw-xxxxx --vpn-gateway-id vgw-xxxxx` |
| `aws ec2 attach-vpn-gateway` | Attach gateway to VPC | `aws ec2 attach-vpn-gateway --vpn-gateway-id vgw-xxxxx --vpc-id vpc-xxxxx` |

### Enable Route Propagation

| Command | Purpose | Example |
|---------|---------|---------|
| `aws ec2 enable-vgw-route-propagation` | Dynamic routing | `aws ec2 enable-vgw-route-propagation --route-table-id rtb-xxxxx --gateway-id vgw-xxxxx` |
| `aws ec2 disable-vgw-route-propagation` | Stop route updates | `aws ec2 disable-vgw-route-propagation --route-table-id rtb-xxxxx --gateway-id vgw-xxxxx` |

### Query VPN Configuration

| Command | Purpose | Example |
|---------|---------|---------|
| `aws ec2 describe-vpn-gateways` | List VPN gateways | `aws ec2 describe-vpn-gateways --output table` |
| `aws ec2 describe-vpn-connections` | List VPN connections | `aws ec2 describe-vpn-connections --output table` |
| `aws ec2 describe-customer-gateways` | List customer gateways | `aws ec2 describe-customer-gateways --output table` |
| Get connection details | VPN tunnel info | `aws ec2 describe-vpn-connections --vpn-connection-ids vpn-xxxxx` |

---

## Security Configuration

### Security Groups for Peering

| Command | Purpose | Example |
|---------|---------|---------|
| Allow ICMP | Enable ping across VPCs | `aws ec2 authorize-security-group-ingress --group-id sg-xxxxx --protocol icmp --from-port -1 --to-port -1 --cidr 10.1.0.0/16` |
| Allow HTTPS | Port 443 access | `aws ec2 authorize-security-group-ingress --group-id sg-xxxxx --protocol tcp --port 443 --cidr 10.1.0.0/16` |
| Allow SSH | Port 22 access | `aws ec2 authorize-security-group-ingress --group-id sg-xxxxx --protocol tcp --port 22 --cidr 10.1.0.0/16` |
| Allow all traffic | Unrestricted access | `aws ec2 authorize-security-group-ingress --group-id sg-xxxxx --protocol all --cidr 10.1.0.0/16` |

### Network ACL Rules for Peering

| Command | Purpose | Example |
|---------|---------|---------|
| `aws ec2 create-network-acl-entry` | Add NACL rule | `aws ec2 create-network-acl-entry --network-acl-id acl-xxxxx --rule-number 100 --protocol tcp --port-range From=443,To=443 --cidr-block 10.1.0.0/16 --ingress` |
| Allow bidirectional | Egress rule | `aws ec2 create-network-acl-entry --network-acl-id acl-xxxxx --rule-number 100 --protocol tcp --port-range From=443,To=443 --cidr-block 10.1.0.0/16 --egress` |
| `aws ec2 describe-network-acls` | List NACLs | `aws ec2 describe-network-acls --filters Name=vpc-id,Values=vpc-xxxxx` |
| `aws ec2 delete-network-acl-entry` | Remove rule | `aws ec2 delete-network-acl-entry --network-acl-id acl-xxxxx --rule-number 100 --ingress` |

---

## Connectivity Type Comparison

### Selection Matrix

| Metric | VPC Peering | VPN | Direct Connect |
|--------|---|---|---|
| **Setup Time** | Minutes | Minutes | Days/Weeks |
| **Data Transfer Cost** | Free (same region) | Free or minimal | $0.30/hr + $0.30/GB |
| **Bandwidth** | Up to 100 Gbps | Limited by internet | Up to 100 Gbps |
| **Latency** | Low | Variable (internet) | Consistent |
| **Encryption** | No (optional TLS) | IPSec encrypted | Dedicated link |
| **Best For** | Multi-VPC apps | Hybrid on-prem | Mission-critical |
| **Complexity** | Low | Medium | High |

---

## Troubleshooting Quick Reference

### Peering Issues

| Problem | Possible Causes | Solutions |
|---------|---|---|
| Connection stuck "initializing" | Bad VPC IDs, permissions | Reject and recreate, check IAM |
| Cannot communicate | Missing routes, SG rules | Add routes to both VPCs, check SG rules |
| Overlapping CIDR error | CIDR blocks overlap | Use non-overlapping ranges or secondary CIDR |
| Peering "pending" | Cross-account, not accepted | Accept in peer account |
| High latency | Cross-region peering | Expected; consider Direct Connect |

### Endpoint Issues

| Problem | Solution |
|---------|----------|
| Cannot reach S3 | Verify endpoint policy allows access, check route table |
| Interface endpoint not working | Add security group, check subnet route |
| Endpoint deleted accidentally | Recreate endpoint, reapply policy |

### VPN Issues

| Problem | Solution |
|---------|----------|
| VPN tunnel down | Check customer gateway config, verify credentials |
| Routes not propagating | Enable VGW route propagation, check BGP |
| Cannot ping across VPN | Check security groups, NACLs, on-prem firewall |

---

## AWS Service Names for Endpoints

| Service | Endpoint Name |
|---------|---|
| S3 | `com.amazonaws.region.s3` |
| DynamoDB | `com.amazonaws.region.dynamodb` |
| EC2 | `com.amazonaws.region.ec2` |
| SNS | `com.amazonaws.region.sns` |
| SQS | `com.amazonaws.region.sqs` |
| Secrets Manager | `com.amazonaws.region.secretsmanager` |
| CloudWatch Logs | `com.amazonaws.region.logs` |
| Lambda | `com.amazonaws.region.lambda` |
| RDS | `com.amazonaws.region.rds` |

---

## Common Patterns

### Complete Peering Setup

```bash
# Variables
VPC_A="vpc-aaaa"
VPC_B="vpc-bbbb"

# 1. Create peering
PCX=$(aws ec2 create-vpc-peering-connection --vpc-id $VPC_A --peer-vpc-id $VPC_B --query 'VpcPeeringConnection.VpcPeeringConnectionId' --output text)

# 2. Accept peering
aws ec2 accept-vpc-peering-connection --vpc-peering-connection-id $PCX

# 3. Add routes (get route table IDs first)
RT_A=$(aws ec2 describe-route-tables --filters Name=vpc-id,Values=$VPC_A --query 'RouteTables[0].RouteTableId' --output text)
RT_B=$(aws ec2 describe-route-tables --filters Name=vpc-id,Values=$VPC_B --query 'RouteTables[0].RouteTableId' --output text)

# 4. Create routes
aws ec2 create-route --route-table-id $RT_A --destination-cidr-block 10.1.0.0/16 --vpc-peering-connection-id $PCX
aws ec2 create-route --route-table-id $RT_B --destination-cidr-block 10.0.0.0/16 --vpc-peering-connection-id $PCX

# 5. Verify
aws ec2 describe-vpc-peering-connections --vpc-peering-connection-ids $PCX --output table
```

### Create Gateway VPC Endpoint for S3

```bash
# Get S3 service name
REGION=$(aws ec2 describe-availability-zones --query 'AvailabilityZones[0].RegionName' --output text)
S3_SERVICE="com.amazonaws.${REGION}.s3"

# Get route tables
VPC_ID="vpc-xxxxx"
RT_IDS=$(aws ec2 describe-route-tables --filters Name=vpc-id,Values=$VPC_ID --query 'RouteTables[*].RouteTableId' --output text)

# Create endpoint
ENDPOINT=$(aws ec2 create-vpc-endpoint --vpc-id $VPC_ID --service-name $S3_SERVICE --route-table-ids $RT_IDS --query 'VpcEndpoint.VpcEndpointId' --output text)

# Verify
aws ec2 describe-vpc-endpoints --vpc-endpoint-ids $ENDPOINT --output table
```

### Allow Traffic Across Peering

```bash
# Create security groups
SG_A=$(aws ec2 create-security-group --group-name sg-peering-a --description "Peering SG A" --vpc-id $VPC_A --query 'GroupId' --output text)
SG_B=$(aws ec2 create-security-group --group-name sg-peering-b --description "Peering SG B" --vpc-id $VPC_B --query 'GroupId' --output text)

# Add rules
aws ec2 authorize-security-group-ingress --group-id $SG_A --protocol tcp --port 443 --cidr 10.1.0.0/16
aws ec2 authorize-security-group-ingress --group-id $SG_B --protocol tcp --port 443 --cidr 10.0.0.0/16

# Apply to instances
# aws ec2 modify-instance-attribute --instance-id i-xxxxx --groups $SG_A
```
