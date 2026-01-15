# 12 - Network Security and Zero Trust - Solutions

## Exercise 1: Understand Zero Trust Principles (Easy) - Solution

**Correct Answer**: B - "Verify every access request, assume breach, implement least privilege"

**Explanation**:
Zero trust architecture is built on three core concepts:
1. **Never Trust, Always Verify** - Every access request requires authentication and authorization, regardless of source
2. **Assume Breach** - Design systems as if the network is already compromised
3. **Least Privilege** - Grant only minimum necessary permissions

Why other answers are wrong:
- A) Traditional perimeter security (outdated)
- C) Encryption helps but isn't zero trust
- D) Passwords alone are insufficient; need multi-factor authentication

**Real-World Application**:
```bash
# Zero trust in practice:
# 1. Verify user identity (MFA)
aws sts assume-role --role-arn arn:aws:iam::123456789:role/AppDeploy \
  --role-session-name app-session

# 2. Check authorization (IAM policy)
# 3. Grant temporary credentials (15 min default)
# 4. Audit all actions in CloudTrail
```

---

## Exercise 2: Identify Microsegmentation Strategy (Easy) - Solution

**Correct Answer**: B - "Create separate subnets for each tier with least-privilege security groups"

**Explanation**:
Microsegmentation divides the network into smaller zones with strict access controls:

**Proper Design**:
```
[Internet] → [IGW] → [Public Subnet - ALB]
                         ↓ (port 80/443 only)
                     [Private Subnet - Web]
                         ↓ (port 8080 only)
                     [Private Subnet - App]
                         ↓ (port 5432 only)
                     [Private Subnet - DB]
```

**Security Groups**:
```bash
# Public tier (ALB)
aws ec2 authorize-security-group-ingress --group-id $PUBLIC_SG \
  --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id $PUBLIC_SG \
  --protocol tcp --port 443 --cidr 0.0.0.0/0

# Web tier (App servers)
aws ec2 authorize-security-group-ingress --group-id $WEB_SG \
  --protocol tcp --port 8080 --source-group $PUBLIC_SG

# Database tier
aws ec2 authorize-security-group-ingress --group-id $DB_SG \
  --protocol tcp --port 5432 --source-group $WEB_SG
```

**Benefits**:
- Limits lateral movement if one tier is compromised
- Easy to audit and modify security policies
- Simplifies compliance requirements

---

## Exercise 3: Network Policy Basics (Easy) - Solution

**Correct Answer**: B - "Default deny blocks everything unless explicitly allowed (more secure)"

**Explanation**:

**"Allow All" (Insecure - Default)**:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: no-policies
# No policies = all traffic allowed
# This is the default Kubernetes behavior
```

**"Default Deny" (Secure - Zero Trust)**:
```yaml
---
# Deny all ingress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
---
# Deny all egress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
---
# Then explicitly allow only needed traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-specific
spec:
  podSelector:
    matchLabels: {app: api}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector: {matchLabels: {app: web}}
    ports:
    - protocol: TCP
      port: 8080
```

**Verify Default Deny**:
```bash
# Check if pods can communicate
kubectl exec -it pod1 -- wget -O- http://pod2-service
# Should timeout (blocked by default deny)
```

---

## Exercise 4: Security Group Rule Analysis (Easy) - Solution

**Correct Answer**: B - "This exposes the database to the entire internet"

**Explanation**:

**Insecure Rule**:
```bash
# ❌ WRONG: Database publicly accessible
aws ec2 authorize-security-group-ingress --group-id $DB_SG \
  --protocol tcp --port 3306 --cidr 0.0.0.0/0
```

This means:
- Any IP address on the internet can attempt to connect to the database
- Attackers can run credential brute-force attacks
- Confidential data is at extreme risk

**Correct Approach**:
```bash
# ✓ RIGHT: Only app servers can access
aws ec2 authorize-security-group-ingress --group-id $DB_SG \
  --protocol tcp --port 3306 --source-group $APP_SG

# Or restrict to specific IPs (if on-premises)
aws ec2 authorize-security-group-ingress --group-id $DB_SG \
  --protocol tcp --port 3306 --cidr 203.0.113.0/24
```

**Verify Configuration**:
```bash
# Check security group rules
aws ec2 describe-security-groups --group-ids $DB_SG \
  --query 'SecurityGroups[0].IpPermissions[*].[IpProtocol,FromPort,IpRanges[0].CidrIp,UserIdGroupPairs[0].GroupId]' \
  --output table
```

---

## Exercise 5: Traffic Monitoring Importance (Easy) - Solution

**Correct Answer**: B - "Provides visibility into network traffic for forensics and anomaly detection"

**Explanation**:

**Why VPC Flow Logs Are Critical**:

1. **Security Incident Investigation**:
```bash
# Query rejected connections
aws logs filter-log-events \
  --log-group-name /aws/vpc/flowlogs \
  --filter-pattern 'REJECT' \
  --query 'events[*].message'
```

2. **Detect Unusual Patterns**:
```bash
# Find data exfiltration (large outbound transfers)
aws logs filter-log-events \
  --log-group-name /aws/vpc/flowlogs \
  --filter-pattern '[... , bytes > 1000000000, ...]' \
  --query 'events[*].message'
```

3. **Compliance Auditing**:
- Demonstrate security controls to auditors
- Prove you know who accessed what, when
- Support forensic analysis after breaches

**Enable VPC Flow Logs**:
```bash
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids $VPC_ID \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs
```

**Sample Flow Log Entry**:
```
2 123456789012 eni-1a2b3c4d 10.0.0.100 10.0.0.200 55000 5432 6 3 1024 1000000000 1640000000 1640000001 ACCEPT OK
```

Breakdown:
- Version: 2
- Account ID: 123456789012
- Interface ID: eni-1a2b3c4d
- Source IP: 10.0.0.100
- Destination IP: 10.0.0.200
- Source Port: 55000
- Destination Port: 5432 (PostgreSQL)
- Protocol: 6 (TCP)
- Packets: 3
- Bytes: 1024
- Action: ACCEPT

---

## Exercise 6: Design a Segmented Network (Medium) - Solution

**Answer**:

### 1. Subnet Design

```
VPC: 10.0.0.0/16

Public Subnets (DMZ):
  - 10.0.1.0/24 (us-east-1a) - ALB only
  - 10.0.2.0/24 (us-east-1b) - ALB only

Private Subnets (Web/App):
  - 10.0.10.0/24 (us-east-1a) - Application servers
  - 10.0.11.0/24 (us-east-1b) - Application servers

Private Subnets (Database):
  - 10.0.20.0/24 (us-east-1a) - Database only
  - 10.0.21.0/24 (us-east-1b) - Database (replica)
```

**Why**: 
- Multiple AZs for high availability
- Public subnets contain only load balancer (limited exposure)
- Application servers in private subnets (no direct internet access)
- Database in private subnets (no internet access whatsoever)

### 2. Security Groups

```bash
# ========== PUBLIC SECURITY GROUP (ALB) ==========
ALB_SG=$(aws ec2 create-security-group \
  --group-name alb-sg --description "ALB" --vpc-id $VPC_ID \
  --query 'GroupId' --output text)

# Allow HTTPS from internet
aws ec2 authorize-security-group-ingress --group-id $ALB_SG \
  --protocol tcp --port 443 --cidr 0.0.0.0/0

# Allow HTTP from internet (for redirect to HTTPS)
aws ec2 authorize-security-group-ingress --group-id $ALB_SG \
  --protocol tcp --port 80 --cidr 0.0.0.0/0

# ========== WEB SECURITY GROUP (App Servers) ==========
WEB_SG=$(aws ec2 create-security-group \
  --group-name web-sg --description "App servers" --vpc-id $VPC_ID \
  --query 'GroupId' --output text)

# Allow only from ALB on application port
aws ec2 authorize-security-group-ingress --group-id $WEB_SG \
  --protocol tcp --port 8080 --source-group $ALB_SG

# Allow health checks from ALB
aws ec2 authorize-security-group-ingress --group-id $WEB_SG \
  --protocol tcp --port 8080 --source-group $ALB_SG

# ========== DATABASE SECURITY GROUP ==========
DB_SG=$(aws ec2 create-security-group \
  --group-name db-sg --description "Database tier" --vpc-id $VPC_ID \
  --query 'GroupId' --output text)

# Allow only from Web tier on PostgreSQL port
aws ec2 authorize-security-group-ingress --group-id $DB_SG \
  --protocol tcp --port 5432 --source-group $WEB_SG

# Allow replication between database nodes
aws ec2 authorize-security-group-ingress --group-id $DB_SG \
  --protocol tcp --port 5432 --source-group $DB_SG
```

### 3. NACL Usage

```bash
# NACLs are less critical than Security Groups for stateful apps
# But useful for defense-in-depth

# NACLs are stateless (must define both directions):
aws ec2 create-network-acl --vpc-id $VPC_ID
aws ec2 create-network-acl-entry --network-acl-id $NACL \
  --rule-number 100 --protocol tcp --port-range FromPort=5432,ToPort=5432 \
  --egress=false --cidr-block 10.0.10.0/24

# However, for most purposes, Security Groups are sufficient
# Use NACLs only if you need:
# - Explicit denial rules (SGs are implicit deny by default)
# - Protection against EC2 instance compromise (app running on instance)
# - Compliance requirements for explicit NACL rules
```

**Validation**:
```bash
# Verify web pod cannot reach database directly
kubectl run debug-pod --image=busybox
kubectl exec -it debug-pod -- timeout 2 bash -c "nc -zv db.default.svc.cluster.local 5432"
# Should timeout (Database SG only allows from Web SG)
```

---

## Exercise 7: Write Kubernetes Network Policies (Medium) - Solution

```yaml
---
# Namespace setup
apiVersion: v1
kind: Namespace
metadata:
  name: microservices
---
# Default deny all ingress traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: microservices
spec:
  podSelector: {}
  policyTypes:
  - Ingress

---
# Default deny all egress traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: microservices
spec:
  podSelector: {}
  policyTypes:
  - Egress

---
# Allow frontend to accept external traffic on port 3000
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-external-to-frontend
  namespace: microservices
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector: {}  # Allow from any namespace
    ports:
    - protocol: TCP
      port: 3000

---
# Allow frontend to call API
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-api
  namespace: microservices
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: api
    ports:
    - protocol: TCP
      port: 8080
  # Allow DNS for service discovery
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53

---
# Allow API to accept from frontend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-from-frontend
  namespace: microservices
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080

---
# Allow API to call database
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-database
  namespace: microservices
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  # Allow DNS
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53

---
# Allow database to accept from API
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-database-from-api
  namespace: microservices
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api
    ports:
    - protocol: TCP
      port: 5432
```

**Apply and Test**:
```bash
kubectl apply -f network-policies.yaml

# Verify frontend can reach API
POD=$(kubectl get pods -n microservices -l app=frontend -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -n microservices -- curl http://api-service:8080/status

# Verify frontend cannot reach database (should timeout)
kubectl exec -it $POD -n microservices -- timeout 2 bash -c "curl http://database:5432" || echo "✓ Blocked as expected"
```

---

## Exercise 8: Analyze a Security Incident (Medium) - Solution

**Diagnosis Steps**:

### Step 1: Verify Network Policies
```bash
# Check policies in namespace
kubectl get networkpolicy -n <namespace>
kubectl describe networkpolicy default-deny -n <namespace>
```

Output shows the policy has:
```yaml
policyTypes:
- Ingress
- Egress
```

With no egress rules, it's blocking ALL egress traffic.

### Step 2: Identify What's Needed
Application needs to make external API calls, so it needs:
1. Egress to external IP/DNS on port 443 (HTTPS)
2. DNS resolution (UDP port 53 to CoreDNS)

### Step 3: Write Required Policies

```yaml
---
# Allow DNS queries (required for all pods to resolve service names)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
  namespace: <your-namespace>
spec:
  podSelector: {}  # Applies to all pods
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53

---
# Allow external HTTPS calls
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-external-api
  namespace: <your-namespace>
spec:
  podSelector:
    matchLabels:
      app: my-app  # Label your pods with this
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector: {}  # Allow to other namespaces
    - podSelector: {}  # Allow to pods in same namespace
    ports:
    - protocol: TCP
      port: 443  # HTTPS
    - protocol: TCP
      port: 80   # HTTP if needed
  # Also need DNS
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
```

### Step 4: Verification
```bash
# Apply the new policy
kubectl apply -f allow-external-api.yaml

# Test external API call
kubectl exec -it <pod> -n <namespace> -- curl https://api.example.com

# Check logs if still failing
kubectl logs <pod> -n <namespace>
```

**Key Learning**: Always allow DNS (port 53 UDP) when implementing egress rules!

---

## Exercise 9: Evaluate WAF Rules (Medium) - Solution

**Root Causes of False Positives**:

### 1. Overly Aggressive SQL Injection Rules
```yaml
# ❌ Too Strict - Blocks any request with "OR" keyword
Rule: 'SELECT.*OR' -> BLOCK

# Result: Legitimate queries like "SELECT * FROM users WHERE active OR premium"
```

### 2. Strict XSS Filters on Dynamic Content
```yaml
# ❌ Too Strict - Blocks all <script> tags
Rule: '<script' -> BLOCK

# Result: Blocks legitimate user-generated content that includes code examples
```

### 3. IP Reputation Lists
```yaml
# ❌ False Positive - Shared IP gets blocked for one user's abuse
CloudflareIPs might be in reputation list but used by millions
```

### 4. Rate Limiting Set Too Low
```bash
# ❌ False Positive - 10 requests per minute
# Legitimate users can trigger this with normal browsing
```

**Solution Steps**:

### Step 1: Enable WAF Logging
```bash
# Configure WAF to log all requests
aws wafv2 create-logging-configuration \
  --logging-configuration ResourceArn=<WAF_ARN>,LogGroupName=/aws/wafv2

# Query logs for blocked requests
aws logs filter-log-events \
  --log-group-name /aws/wafv2 \
  --filter-pattern 'BLOCK'
```

### Step 2: Analyze False Positives
```bash
# Check what rules are matching legitimate traffic
aws logs filter-log-events \
  --log-group-name /aws/wafv2 \
  --filter-pattern '[... , "action":"BLOCK" , ...]'

# Look for patterns:
# - Same user-agent (legitimate)
# - From known IP range (company)
# - Valid request headers
```

### Step 3: Create Exceptions
```bash
# Use managed rule groups instead of overly strict custom rules
cat > waf-policy.json << 'EOF'
{
  "Rules": [
    {
      "Name": "RateLimitRule",
      "Priority": 1,
      "Statement": {
        "RateBasedStatement": {
          "Limit": 2000,  # 2000 requests per 5 minutes (reasonable)
          "AggregateKeyType": "IP"
        }
      },
      "Action": {"Block": {}},
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "RateLimitRule"
      }
    },
    {
      "Name": "AWSManagedRulesCommonRuleSet",
      "Priority": 2,
      "OverrideAction": {"None": {}},
      "Statement": {
        "ManagedRuleGroupStatement": {
          "VendorName": "AWS",
          "Name": "AWSManagedRulesCommonRuleSet"
        }
      },
      "VisibilityConfig": {
        "SampledRequestsEnabled": true,
        "CloudWatchMetricsEnabled": true,
        "MetricName": "CommonRuleSet"
      }
    },
    {
      "Name": "APIWhitelist",
      "Priority": 0,
      "Action": {"Allow": {}},
      "Statement": {
        "IpSetReferenceStatement": {
          "ARN": "arn:aws:wafv2:...:ipset/company-ips"
        }
      }
    }
  ]
}
EOF

# Update WAF
aws wafv2 update-web-acl --name my-web-acl \
  --default-action Block={} \
  --rules file://waf-policy.json
```

### Step 4: Monitor and Adjust
```bash
# After adjusting, re-analyze logs
aws logs filter-log-events \
  --log-group-name /aws/wafv2 \
  --filter-pattern 'BLOCK' \
  --start-time $(date -d '1 hour ago' +%s)000

# Track metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/WAFV2 \
  --metric-name BlockedRequests \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-02T00:00:00Z \
  --period 3600 \
  --statistics Sum
```

**Best Practices**:
- Use AWS Managed Rules (already tuned)
- Start with COUNT mode, switch to BLOCK after validation
- Monitor logs continuously
- Create IP whitelists for known legitimate sources

---

## Exercise 10: Design DDoS Protection Strategy (Medium) - Solution

**Comprehensive DDoS Defense**:

### 1. AWS Service Selection

```bash
# Enable Shield Standard (free for all AWS customers)
# Automatically protects against common attacks

# Enable Shield Advanced (24/7 support)
aws shield create-subscription

# Create WAF
aws wafv2 create-web-acl \
  --name DDoS-Protection-WAF \
  --scope CLOUDFRONT

# Attach to CloudFront + ALB
aws wafv2 associate-web-acl \
  --web-acl-arn arn:aws:wafv2:... \
  --resource-arn arn:aws:cloudfront:...
```

### 2. CloudFront Configuration (Best Practice)

```json
{
  "DistributionConfig": {
    "Enabled": true,
    "Origins": [
      {
        "DomainName": "myapp.elb.amazonaws.com",
        "OriginShield": {
          "Enabled": true,
          "OriginShieldRegion": "us-east-1"
        }
      }
    ],
    "CacheBehaviors": [
      {
        "ViewerProtocolPolicy": "redirect-to-https",
        "CachePolicyId": "658327ea-f89d-4fab-a63d-7e88639e58f6",
        "Compress": true
      }
    ],
    "WebACLId": "arn:aws:wafv2:us-east-1:::/global/webacl/DDoS"
  }
}
```

### 3. WAF Rate Limiting Rules

```yaml
apiVersion: wafv2:2019-07-29
Spec:
  WebACL:
    Rules:
      - Name: RateLimitPerIP
        Priority: 1
        Statement:
          RateBasedStatement:
            Limit: 2000  # Requests per 5 minutes
            AggregateKeyType: IP
        Action:
          Block:
            CustomResponse:
              ResponseCode: 429  # Too Many Requests
      
      - Name: GeoBlockingRule
        Priority: 2
        Statement:
          GeoMatchStatement:
            CountryCodes: [CN, RU, KP]  # Example
        Action:
          Block: {}

      - Name: BotControlRule
        Priority: 3
        Statement:
          ManagedRuleGroupStatement:
            VendorName: AWS
            Name: AWSManagedRulesBotControlRuleSet
        OverrideAction:
          None: {}
```

### 4. Application-Level Defenses

```python
# Flask application with rate limiting
from flask import Flask, request
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

app = Flask(__name__)
limiter = Limiter(app, key_func=get_remote_address)

@app.route('/api/data')
@limiter.limit("100 per minute")  # 100 requests/minute per IP
def get_data():
    return {"status": "ok"}

# Also implement:
# - Connection pooling
# - Query caching
# - Load balancing across instances
# - Circuit breakers for overloaded services
```

### 5. Monitoring and Alerting

```bash
# CloudWatch Alarms for DDoS indicators
aws cloudwatch put-metric-alarm \
  --alarm-name HighTrafficAlert \
  --alarm-description "Alert if traffic > 100k req/min" \
  --metric-name RequestCount \
  --namespace AWS/WAF \
  --statistic Sum \
  --period 60 \
  --threshold 100000 \
  --comparison-operator GreaterThanThreshold

# Monitor rejected connections
aws cloudwatch put-metric-alarm \
  --alarm-name HighRejectionRate \
  --metric-name BlockedRequests \
  --namespace AWS/WAF \
  --statistic Sum \
  --period 60 \
  --threshold 10000 \
  --comparison-operator GreaterThanThreshold
```

### 6. Incident Response Runbook

```bash
#!/bin/bash
# DDoS Incident Response Automation

echo "DDoS Attack Detected!"

# Step 1: Activate emergency WAF rules
aws wafv2 update-web-acl \
  --name Emergency-Rules \
  --rules file://emergency-rules.json

# Step 2: Lower rate limits
# Change from 2000 to 500 requests per 5 minutes
LOWER_RATE_POLICY=$(cat <<'EOF'
{
  "Limit": 500,
  "AggregateKeyType": "IP"
}
EOF
)

# Step 3: Enable geographic blocking if needed
# Block non-customer regions

# Step 4: Scale application
aws autoscaling set-desired-capacity \
  --auto-scaling-group-name my-asg \
  --desired-capacity 20  # Scale up

# Step 5: Enable Origin Shield
aws cloudfront update-distribution \
  --id <DISTRIBUTION_ID> \
  --distribution-config file://shield-enabled-config.json

# Step 6: Notify team
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789:DDoS-Alert \
  --message "DDoS attack detected and mitigations activated"

# Step 7: Monitor metrics
watch -n 10 'aws cloudwatch get-metric-statistics \
  --namespace AWS/WAF \
  --metric-name BlockedRequests \
  --start-time $(date -u -d "1 hour ago" +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 60 \
  --statistics Sum'

echo "Incident response activated"
```

### 7. Post-Incident Analysis

```bash
# Document the attack
cat > incident-report.md << 'EOF'
# DDoS Incident Report

## Attack Timeline
- 2024-01-15 14:00 UTC: Attack started (20k req/sec)
- 14:05: WAF rules activated (reduction to 5k req/sec)
- 14:10: Auto-scaling engaged (capacity +100%)
- 14:15: Attack ended

## Metrics
- Peak traffic: 20,000 req/sec
- Blocked by WAF: 18,500 req/sec
- Application impact: <1% error rate
- User impact: None

## Lessons Learned
1. Rate limiting threshold was appropriate
2. Auto-scaling responded within 5 minutes
3. Geographic blocking would have helped

## Improvements for Next Time
1. Lower alerting thresholds
2. Pre-stage auto-scaling capacity
3. Add more geographic filtering
EOF
```

**Defense-in-Depth Summary**:
```
[Internet Attack] → [CloudFront CDN] → [Shield Standard]
                          ↓
                    [Rate Limiting]
                          ↓
                   [Shield Advanced]
                          ↓
                    [WAF Rules]
                          ↓
                  [Application SG]
                          ↓
                  [App Load Balancer]
                          ↓
                [Healthy Instances]
                (Auto-scaling)
```

---

## Quick Reference: kubectl Commands

```bash
# Get all network policies
kubectl get networkpolicy -A

# Describe a policy
kubectl describe networkpolicy <name> -n <namespace>

# Apply network policies
kubectl apply -f policy.yaml

# Test connectivity between pods
kubectl run debug --image=busybox --rm -it -- sh
# Inside pod: wget -O- http://service-name:port

# Check pod labels (used in policies)
kubectl get pods --show-labels
```
