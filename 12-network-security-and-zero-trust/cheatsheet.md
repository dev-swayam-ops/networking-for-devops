# 12 - Network Security and Zero Trust - Cheatsheet

## AWS VPC Security Group Commands

### Create and Manage Security Groups

| Command | Purpose | Example |
|---------|---------|---------|
| `aws ec2 create-security-group` | Create security group | `aws ec2 create-security-group --group-name web-sg --description "Web tier" --vpc-id vpc-12345` |
| `aws ec2 describe-security-groups` | List security groups | `aws ec2 describe-security-groups --region us-east-1` |
| `aws ec2 describe-security-groups --group-ids` | Get specific SG details | `aws ec2 describe-security-groups --group-ids sg-12345` |
| `aws ec2 delete-security-group` | Delete security group | `aws ec2 delete-security-group --group-id sg-12345` |

### Configure Security Group Rules

| Command | Purpose | Example |
|---------|---------|---------|
| `authorize-security-group-ingress` | Add inbound rule | `aws ec2 authorize-security-group-ingress --group-id sg-12345 --protocol tcp --port 80 --cidr 0.0.0.0/0` |
| `authorize-security-group-egress` | Add outbound rule | `aws ec2 authorize-security-group-egress --group-id sg-12345 --protocol tcp --port 443 --cidr 0.0.0.0/0` |
| `revoke-security-group-ingress` | Remove inbound rule | `aws ec2 revoke-security-group-ingress --group-id sg-12345 --protocol tcp --port 22 --cidr 0.0.0.0/0` |
| `revoke-security-group-egress` | Remove outbound rule | `aws ec2 revoke-security-group-egress --group-id sg-12345 --protocol -1 --cidr 0.0.0.0/0` |

### Source Types for Rules

| Type | Syntax | Use Case |
|------|--------|----------|
| CIDR Block | `--cidr 10.0.0.0/16` | Allow from subnet or network |
| Security Group | `--source-group sg-12345` | Allow from another SG |
| Prefix List | `--prefix-list s3-us-east-1` | AWS services (S3, DynamoDB) |
| All Traffic | `--protocol -1` | Allow/deny all (rarely used) |

---

## Network Policy Commands (Kubernetes)

### Query Network Policies

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl get networkpolicy` | List policies | `kubectl get networkpolicy -n default` |
| `kubectl get networkpolicy -A` | List all policies | `kubectl get networkpolicy --all-namespaces` |
| `kubectl describe networkpolicy` | Policy details | `kubectl describe networkpolicy deny-all -n default` |
| `kubectl apply -f policy.yaml` | Create/update policy | `kubectl apply -f network-policies.yaml` |
| `kubectl delete networkpolicy` | Delete policy | `kubectl delete networkpolicy deny-all` |

### Test Network Policies

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl run debug` | Create debug pod | `kubectl run -it --rm debug --image=busybox` |
| `kubectl exec -it` | Run command in pod | `kubectl exec -it pod1 -- wget http://service:80` |
| `wget` from pod | Test HTTP connectivity | Inside pod: `wget -O- http://service-name` |
| `curl` from pod | Test HTTP/HTTPS | Inside pod: `curl https://external-api.com` |
| `nc -zv` | Test port open | Inside pod: `nc -zv service-name 8080` |
| `nslookup` | DNS test | Inside pod: `nslookup service-name` |

---

## VPC Flow Logs Commands

### Enable and Manage Flow Logs

| Command | Purpose | Example |
|---------|---------|---------|
| `aws ec2 create-flow-logs` | Enable flow logs | `aws ec2 create-flow-logs --resource-type VPC --resource-ids vpc-12345 --traffic-type ALL --log-destination-type cloud-watch-logs --log-group-name /aws/vpc/flowlogs` |
| `aws ec2 describe-flow-logs` | List flow log configs | `aws ec2 describe-flow-logs --region us-east-1` |
| `aws ec2 delete-flow-logs` | Disable flow logs | `aws ec2 delete-flow-logs --flow-log-ids fl-12345` |

### Query Flow Logs

| Command | Purpose | Example |
|---------|---------|---------|
| `aws logs filter-log-events` | Search logs | `aws logs filter-log-events --log-group-name /aws/vpc/flowlogs --filter-pattern 'REJECT'` |
| `aws logs tail` | Watch logs in real-time | `aws logs tail /aws/vpc/flowlogs --follow` |
| `aws logs describe-log-streams` | List log streams | `aws logs describe-log-streams --log-group-name /aws/vpc/flowlogs` |

### Flow Log Fields

| Field | Meaning | Example |
|-------|---------|---------|
| srcaddr | Source IP | 10.0.1.100 |
| dstaddr | Destination IP | 10.0.2.200 |
| srcport | Source port | 55000 |
| dstport | Destination port | 5432 |
| protocol | Protocol number (6=TCP, 17=UDP) | 6 |
| action | ACCEPT or REJECT | ACCEPT |
| bytes | Bytes transferred | 1024 |

---

## AWS WAF Commands

### Create and Manage WAF

| Command | Purpose | Example |
|---------|---------|---------|
| `aws wafv2 create-web-acl` | Create WAF | `aws wafv2 create-web-acl --scope REGIONAL --name my-waf` |
| `aws wafv2 list-web-acls` | List WAFs | `aws wafv2 list-web-acls --scope REGIONAL` |
| `aws wafv2 describe-web-acl` | Get WAF details | `aws wafv2 describe-web-acl --scope REGIONAL --name my-waf` |
| `aws wafv2 update-web-acl` | Update WAF rules | `aws wafv2 update-web-acl --scope REGIONAL --name my-waf --rules file://rules.json` |
| `aws wafv2 delete-web-acl` | Delete WAF | `aws wafv2 delete-web-acl --scope REGIONAL --name my-waf` |

### Associate and Disassociate WAF

| Command | Purpose | Example |
|---------|---------|---------|
| `aws wafv2 associate-web-acl` | Attach to resource | `aws wafv2 associate-web-acl --web-acl-arn arn:... --resource-arn arn:aws:elasticloadbalancing:...` |
| `aws wafv2 disassociate-web-acl` | Detach from resource | `aws wafv2 disassociate-web-acl --resource-arn arn:...` |

### WAF Logging

| Command | Purpose | Example |
|---------|---------|---------|
| `aws wafv2 create-logging-configuration` | Enable WAF logs | `aws wafv2 create-logging-configuration --logging-configuration ResourceArn=arn:...,LogGroupName=/aws/wafv2` |
| `aws wafv2 get-logging-configuration` | Get logging config | `aws wafv2 get-logging-configuration --resource-arn arn:...` |

---

## AWS Shield Commands

### Enable DDoS Protection

| Command | Purpose | Example |
|---------|---------|---------|
| `aws shield create-subscription` | Enable Shield Advanced | `aws shield create-subscription` |
| `aws shield describe-subscription` | Check Shield status | `aws shield describe-subscription` |
| `aws shield cancel-subscription` | Disable Shield Advanced | `aws shield cancel-subscription` |

### DDoS Response

| Command | Purpose | Example |
|---------|---------|---------|
| `aws shield describe-attack` | Get attack details | `aws shield describe-attack --resource-arns arn:aws:elasticloadbalancing:...` |
| `aws shield list-attacks` | List recent attacks | `aws shield list-attacks --start-time 2024-01-01T00:00:00Z` |

---

## IAM Security Commands

### IAM Users and Roles

| Command | Purpose | Example |
|---------|---------|---------|
| `aws iam create-user` | Create user | `aws iam create-user --user-name deployer` |
| `aws iam create-role` | Create role | `aws iam create-role --role-name AppRole --assume-role-policy-document file://policy.json` |
| `aws iam list-users` | List users | `aws iam list-users` |
| `aws iam list-roles` | List roles | `aws iam list-roles` |

### IAM Policies

| Command | Purpose | Example |
|---------|---------|---------|
| `aws iam put-user-policy` | Attach user policy | `aws iam put-user-policy --user-name deployer --policy-name S3Access --policy-document file://policy.json` |
| `aws iam put-role-policy` | Attach role policy | `aws iam put-role-policy --role-name AppRole --policy-name EC2Access --policy-document file://policy.json` |
| `aws iam attach-user-policy` | Attach managed policy | `aws iam attach-user-policy --user-name deployer --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess` |

### Credentials and Assume Role

| Command | Purpose | Example |
|---------|---------|---------|
| `aws sts get-caller-identity` | Current credentials | `aws sts get-caller-identity` |
| `aws sts assume-role` | Use temporary credentials | `aws sts assume-role --role-arn arn:aws:iam::123456:role/AppRole --role-session-name app-session` |
| `aws sts get-session-token` | MFA session token | `aws sts get-session-token --serial-number arn:aws:iam::...:mfa/user --token-code 123456` |

---

## CloudTrail Audit Logging

### Enable and Query Audit Logs

| Command | Purpose | Example |
|---------|---------|---------|
| `aws cloudtrail create-trail` | Create trail | `aws cloudtrail create-trail --name my-trail --s3-bucket-name my-bucket` |
| `aws cloudtrail start-logging` | Enable logging | `aws cloudtrail start-logging --trail-name my-trail` |
| `aws cloudtrail stop-logging` | Disable logging | `aws cloudtrail stop-logging --trail-name my-trail` |
| `aws cloudtrail lookup-events` | Query events | `aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=CreateSecurityGroup` |

---

## VPC Flow Logs Analysis Examples

### Block Traffic Analysis
```bash
# Find all REJECTED connections
aws logs filter-log-events \
  --log-group-name /aws/vpc/flowlogs \
  --filter-pattern '[version, account, interface_id, srcaddr, dstaddr, srcport, dstport="5432", protocol="6", packets, bytes, start, end, action="REJECT", flow_log_status]' \
  --query 'events[*].message'

# Result: Shows rejected PostgreSQL connections
```

### Data Exfiltration Detection
```bash
# Find large data transfers (> 1GB)
aws logs filter-log-events \
  --log-group-name /aws/vpc/flowlogs \
  --filter-pattern '[... , bytes > 1073741824 , ...]' \
  --query 'events[*].message'
```

### Suspicious Port Activity
```bash
# Find connections to non-standard ports (potential tunneling)
aws logs filter-log-events \
  --log-group-name /aws/vpc/flowlogs \
  --filter-pattern '[... , dstport != "80" && dstport != "443" && dstport != "22" , ...]'
```

### Specific Time Range Analysis
```bash
# Find rejections in last hour
START_TIME=$(date -d '1 hour ago' +%s)000
END_TIME=$(date +%s)000

aws logs filter-log-events \
  --log-group-name /aws/vpc/flowlogs \
  --filter-pattern 'REJECT' \
  --start-time $START_TIME \
  --end-time $END_TIME
```

---

## Kubernetes RBAC Security

### User and Role Management

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl auth can-i` | Check permissions | `kubectl auth can-i create pods --as=user@example.com` |
| `kubectl auth reconcile` | Update RBAC | `kubectl auth reconcile -f rbac-config.yaml` |
| `kubectl get role` | List roles | `kubectl get role -A` |
| `kubectl get rolebinding` | List role bindings | `kubectl get rolebinding -A` |

### Service Accounts for Pods

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl create serviceaccount` | Create service account | `kubectl create serviceaccount app-sa` |
| `kubectl get serviceaccount` | List service accounts | `kubectl get serviceaccount -n kube-system` |
| `kubectl describe serviceaccount` | Service account details | `kubectl describe serviceaccount default` |

---

## Network Segmentation Quick Reference

### Three-Tier Security Model
```
┌─────────────────────────────────────────┐
│   Internet (0.0.0.0/0)                  │
└──────────────┬──────────────────────────┘
               │ Allow 80, 443
        ┌──────▼──────────┐
        │  DMZ (Public)   │ AWS Shield, WAF
        │  ALB/CloudFront │
        └──────┬──────────┘
               │ Allow 8080 from ALB only
        ┌──────▼──────────────┐
        │  Web Tier (Private) │ App Servers
        │  Security Group:    │ No internet
        │  :8080 from ALB     │
        └──────┬──────────────┘
               │ Allow 5432 from Web only
        ┌──────▼──────────────┐
        │  DB Tier (Private)  │ Database
        │  Security Group:    │ No internet
        │  :5432 from Web     │ No exposure
        └─────────────────────┘
```

### Default Deny + Allow Specific Pattern
```yaml
# Step 1: Deny all
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]

---
# Step 2: Explicitly allow only needed traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-api
spec:
  podSelector:
    matchLabels: {app: api}
  policyTypes: [Ingress]
  ingress:
  - from:
    - podSelector: {matchLabels: {app: frontend}}
    ports:
    - protocol: TCP
      port: 8080
```

---

## Common Security Group Patterns

### Allow from Load Balancer
```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-app \
  --protocol tcp \
  --port 8080 \
  --source-group sg-alb
```

### Allow from Another VPC (Peering)
```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-db \
  --protocol tcp \
  --port 5432 \
  --cidr 10.1.0.0/16  # Peer VPC CIDR
```

### Allow SSH from Bastion Host
```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-app \
  --protocol tcp \
  --port 22 \
  --source-group sg-bastion
```

### Allow S3 Access (Gateway Endpoint)
```bash
aws ec2 authorize-security-group-egress \
  --group-id sg-app \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0  # S3 via HTTPS

# Better: Use S3 Gateway Endpoint (no need for explicit rule)
```

---

## Quick Reference: Zero Trust Checklist

- ✓ Default Deny at network layer (Security Groups, Network Policies)
- ✓ Least privilege (only open needed ports/services)
- ✓ Microsegmentation (separate tiers, separate subnets)
- ✓ Encryption in transit (TLS for all connections)
- ✓ Authentication (Mutual TLS, mTLS for service-to-service)
- ✓ Monitoring (VPC Flow Logs, CloudWatch, WAF logging)
- ✓ Audit trails (CloudTrail for API calls, logs for all traffic)
- ✓ Incident response (Runbooks, auto-remediation)
