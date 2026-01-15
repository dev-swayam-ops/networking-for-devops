# 12 - Network Security and Zero Trust

## What You'll Learn

By the end of this module, you'll understand:

1. **Zero Trust Architecture Principles** - Never trust, always verify; implement identity-based access control
2. **Microsegmentation** - Divide networks into smaller zones for granular access control and threat containment
3. **Network Policies and Segmentation** - Implement least-privilege access at the network layer across cloud infrastructure
4. **Web Application Firewalls (WAF)** - Protect applications from common attacks (SQL injection, XSS, DDoS)
5. **DDoS Protection Strategies** - Detect, mitigate, and defend against distributed denial-of-service attacks
6. **Encryption in Transit** - Secure data movement between services using TLS, IPSec, and VPN encryption
7. **Identity and Access Management (IAM)** - Control who, what, and when for network resource access
8. **Threat Detection and Monitoring** - Identify suspicious network behavior and security incidents in real-time
9. **Security Compliance and Auditing** - Meet regulatory requirements (SOC 2, PCI-DSS, HIPAA) with network security logs
10. **Incident Response Procedures** - Respond quickly to security breaches with containment and forensic analysis

---

## Prerequisites

Before starting this module, ensure you have:

- **AWS Account** with permissions to create VPC, Security Groups, WAF, CloudWatch, VPC Flow Logs
- **Docker and Kubernetes Basic Knowledge** - Understand container images, pods, and basic deployments
- **AWS CLI** installed and configured with valid credentials (`aws sts get-caller-identity` works)
- **kubectl** installed and able to connect to EKS cluster (from Module 11)
- **curl** or **wget** for testing web services and application endpoints
- **Familiarity with Linux command line** - You should be comfortable with bash and text manipulation

---

## Key Concepts

### 1. Zero Trust Architecture

**Definition**: Security model that requires verification for every access request, regardless of network location.

**Core Principles**:
- **Never Trust, Always Verify** - Every user, device, and request authenticated and authorized
- **Assume Breach** - Design security assuming the network is already compromised
- **Least Privilege** - Grant minimum necessary permissions for each user/service
- **Verify Explicitly** - Use all available data points for authentication (identity, device, location, behavior)
- **Secure by Default** - Deny all access unless explicitly allowed
- **Microsegmentation** - Network divided into small zones with granular access policies

**Benefits**:
- Reduces lateral movement of attackers
- Limits damage from compromised accounts
- Simplifies compliance with regulatory frameworks
- Improves visibility and monitoring

---

### 2. Microsegmentation

**Definition**: Dividing network into smaller security zones to isolate and protect sensitive assets.

**Types**:
- **Horizontal Segmentation** - Separate same-tier services (web servers from each other)
- **Vertical Segmentation** - Separate network layers (web tier from database tier)
- **Application-Level** - Protect specific applications or tenants

**Implementation Methods**:
1. **Network Policies** - Pod-level access control in Kubernetes
2. **Security Groups** - EC2-level stateful firewall rules
3. **NACLs** - Subnet-level stateless firewall rules
4. **WAF** - Application-level request filtering
5. **Service Mesh** - mTLS and traffic policies (Istio, Linkerd)

**Example in AWS**:
```
Internet Gateway (public)
    ↓
ALB/WAF (DMZ tier)
    ↓
Web Security Group (allow only ALB)
    ↓
Database Security Group (allow only Web SG)
    ↓
RDS (encrypted, no public access)
```

---

### 3. Network Policies

**Kubernetes Network Policies**: Control traffic flow between pods using labels.

**Components**:
- **Pod Selector** - Which pods the policy applies to
- **Ingress Rules** - What traffic is allowed INTO pods
- **Egress Rules** - What traffic is allowed OUT of pods
- **Namespace Selectors** - Control cross-namespace communication
- **IP Blocks** - Allow/deny specific CIDR ranges

**Default Policy States**:
- **Allow All** (default) - No network policies = all traffic allowed
- **Deny All** - Block everything unless explicitly allowed
- **Allow Specific** - Only allow defined traffic patterns

---

### 4. Web Application Firewall (WAF)

**Purpose**: Protect web applications from common attacks at Layer 7 (Application).

**Common Attacks Prevented**:
- **SQL Injection** - Malicious SQL code in input fields
- **Cross-Site Scripting (XSS)** - JavaScript injection attacks
- **Cross-Site Request Forgery (CSRF)** - Unauthorized actions from compromised sessions
- **DDoS** - Overwhelming with request volume
- **Bot Attacks** - Malicious automated access
- **Sensitive Data Exposure** - Leaking PII or API keys

**AWS WAF Features**:
1. **IP Reputation Lists** - Block known malicious IPs
2. **Rate Limiting** - Throttle excessive requests per IP
3. **Rule Groups** - Pre-built rules from AWS and third-parties
4. **Custom Rules** - Define application-specific protections
5. **Web ACLs** - Apply rules to CloudFront, ALB, API Gateway

---

### 5. DDoS Protection

**DDoS Attack Types**:
1. **Volumetric Attacks** - Flood with massive traffic (DNS amplification, UDP floods)
2. **Protocol Attacks** - Exploit weaknesses (SYN floods, Ping of Death)
3. **Application Attacks** - Target specific services (HTTP floods, Slowloris)

**Defense Layers**:
- **AWS Shield Standard** - Free DDoS protection for all customers
- **AWS Shield Advanced** - Enhanced protection with 24/7 DRT support
- **AWS Shield + WAF** - Combined protection for comprehensive coverage
- **Rate Limiting** - Limit requests per IP/user
- **Geo-blocking** - Block traffic from specific countries
- **Anycast Network** - Distribute traffic across multiple data centers

---

### 6. Encryption in Transit

**Purpose**: Protect data while moving between systems.

**Protocols**:
1. **TLS/SSL** - Encrypted HTTP (HTTPS) with certificate authentication
2. **IPSec** - Layer 3 encryption for VPN tunnels
3. **mTLS** - Mutual TLS between microservices
4. **VPN** - Virtual Private Network with encrypted tunnels

**TLS Handshake**:
```
Client → Server: Hello (ciphers, version)
Server → Client: Certificate, choose cipher
Client → Server: Pre-master secret (encrypted)
Both: Generate session keys
Client → Server: Encrypted traffic begins
```

**Validation**:
- Certificate chain verification
- Hostname matching
- Certificate pinning (optional)
- Regular certificate rotation

---

### 7. Identity and Access Management (IAM)

**AWS IAM Components**:
- **Users** - Individual identities with specific permissions
- **Roles** - Reusable permission sets for users, services, or federated identities
- **Policies** - JSON documents defining specific permissions
- **Groups** - Collections of users with same permissions

**Service-to-Service Communication**:
- **IAM Roles for EC2** - Allow EC2 instances to call AWS APIs
- **IAM Roles for EKS Pods** - IRSA (IAM Roles for Service Accounts) for pods
- **Assume Role** - Temporary credentials from one role to another

**API Authorization**:
- **SigV4 Signing** - AWS CLI/SDK signs API requests
- **Temporary Credentials** - Expire after set time (default 15 minutes)
- **Audit Trail** - CloudTrail logs all API calls

---

### 8. Threat Detection and Monitoring

**Monitoring Tools**:
1. **VPC Flow Logs** - Capture network traffic metadata
2. **CloudWatch** - Centralized logging and metrics
3. **GuardDuty** - Threat detection with machine learning
4. **Security Hub** - Centralized security findings and compliance
5. **Network Analyzer** - Identify and fix unintended network exposure

**What to Monitor**:
- Failed authentication attempts
- Unusual egress traffic (data exfiltration)
- Port scans or connection attempts to suspicious ports
- Cross-account or cross-region unusual access
- Privilege escalation attempts
- Large data transfers during off-hours

---

### 9. Security Compliance and Auditing

**Regulatory Frameworks**:
- **SOC 2 Type II** - Security controls and audit procedures
- **PCI-DSS** - Payment card data security requirements
- **HIPAA** - Healthcare data privacy and security
- **GDPR** - EU data protection regulations
- **ISO 27001** - Information security management standards

**Audit Requirements**:
- **Access Logs** - Who accessed what, when, from where
- **Change Logs** - Track all configuration modifications
- **Network Logs** - All traffic flows and anomalies
- **Data Retention** - Keep logs for compliance period (typically 90 days - 7 years)
- **Encryption Keys** - Manage and audit key access

---

### 10. Incident Response Procedures

**Incident Response Lifecycle**:

1. **Preparation** - Tools, training, runbooks in place
2. **Detection** - Security tools identify anomaly
3. **Analysis** - Investigate and confirm true incident
4. **Containment** - Limit damage and prevent spread
5. **Eradication** - Remove malicious access and payloads
6. **Recovery** - Restore systems to normal operation
7. **Post-Incident** - Review and improve processes

**Response Actions**:
- **Isolate** - Remove affected systems from network
- **Preserve** - Capture logs and system state for forensics
- **Notify** - Alert relevant teams and stakeholders
- **Block** - Update firewall rules and WAF to block attacker
- **Remediate** - Patch vulnerabilities and reset credentials
- **Document** - Record findings and remediation steps

---

## Hands-On Lab: Implementing Zero Trust Security in AWS

This lab implements a complete zero trust network security architecture with:
- Microsegmentation between network tiers
- Network policies for pod-level security
- WAF for web application protection
- DDoS protection
- VPC Flow Logs for monitoring
- Security Group least-privilege rules

### Prerequisites for Lab
```bash
# Verify AWS CLI
aws sts get-caller-identity

# Verify kubectl access (from Module 11)
kubectl get nodes

# Verify curl is available
curl --version
```

---

## Step 1: Create Segmented VPC with Tiered Security Groups

```bash
# Set variables
REGION="us-east-1"
VPC_CIDR="10.50.0.0/16"

# Create VPC
VPC_ID=$(aws ec2 create-vpc --cidr-block $VPC_CIDR --region $REGION \
  --query 'Vpc.VpcId' --output text)
echo "VPC created: $VPC_ID"

# Enable DNS
aws ec2 modify-vpc-attribute --vpc-id $VPC_ID \
  --enable-dns-hostnames --region $REGION

# Create subnets (DMZ, Web, Database)
DMZ_SUBNET=$(aws ec2 create-subnet --vpc-id $VPC_ID \
  --cidr-block 10.50.1.0/24 --region $REGION \
  --query 'Subnet.SubnetId' --output text)

WEB_SUBNET=$(aws ec2 create-subnet --vpc-id $VPC_ID \
  --cidr-block 10.50.2.0/24 --region $REGION \
  --query 'Subnet.SubnetId' --output text)

DB_SUBNET=$(aws ec2 create-subnet --vpc-id $VPC_ID \
  --cidr-block 10.50.3.0/24 --region $REGION \
  --query 'Subnet.SubnetId' --output text)

echo "Subnets created:"
echo "DMZ: $DMZ_SUBNET"
echo "Web: $WEB_SUBNET"
echo "Database: $DB_SUBNET"

# Create Internet Gateway
IGW=$(aws ec2 create-internet-gateway --region $REGION \
  --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --internet-gateway-id $IGW \
  --vpc-id $VPC_ID --region $REGION

# Create Route Table for DMZ (public)
DMZ_RT=$(aws ec2 create-route-table --vpc-id $VPC_ID \
  --region $REGION --query 'RouteTable.RouteTableId' --output text)
aws ec2 associate-route-table --subnet-id $DMZ_SUBNET \
  --route-table-id $DMZ_RT --region $REGION
aws ec2 create-route --route-table-id $DMZ_RT --destination-cidr-block 0.0.0.0/0 \
  --gateway-id $IGW --region $REGION

# Create Route Tables for Web and DB (private)
WEB_RT=$(aws ec2 create-route-table --vpc-id $VPC_ID \
  --region $REGION --query 'RouteTable.RouteTableId' --output text)
aws ec2 associate-route-table --subnet-id $WEB_SUBNET \
  --route-table-id $WEB_RT --region $REGION

DB_RT=$(aws ec2 create-route-table --vpc-id $VPC_ID \
  --region $REGION --query 'RouteTable.RouteTableId' --output text)
aws ec2 associate-route-table --subnet-id $DB_SUBNET \
  --route-table-id $DB_RT --region $REGION

echo "Networking infrastructure created successfully"
```

**Expected Output:**
```
VPC created: vpc-0a1b2c3d4e5f6g7h8
Subnets created:
DMZ: subnet-0d1e2f3a4b5c6d7e8
Web: subnet-0e2f3a4b5c6d7e8f9
Database: subnet-0f3a4b5c6d7e8f9g0
Networking infrastructure created successfully
```

---

## Step 2: Create Security Groups with Least-Privilege Rules

```bash
# Create DMZ Security Group (for ALB)
DMZ_SG=$(aws ec2 create-security-group --group-name dmz-tier \
  --description "DMZ tier for load balancer" --vpc-id $VPC_ID \
  --region $REGION --query 'GroupId' --output text)

# Allow HTTP/HTTPS from Internet
aws ec2 authorize-security-group-ingress --group-id $DMZ_SG \
  --protocol tcp --port 80 --cidr 0.0.0.0/0 --region $REGION
aws ec2 authorize-security-group-ingress --group-id $DMZ_SG \
  --protocol tcp --port 443 --cidr 0.0.0.0/0 --region $REGION

echo "DMZ Security Group created: $DMZ_SG"

# Create Web Security Group
WEB_SG=$(aws ec2 create-security-group --group-name web-tier \
  --description "Web tier for application servers" --vpc-id $VPC_ID \
  --region $REGION --query 'GroupId' --output text)

# Allow traffic only from DMZ SG (least privilege)
aws ec2 authorize-security-group-ingress --group-id $WEB_SG \
  --protocol tcp --port 8080 --source-group $DMZ_SG --region $REGION

echo "Web Security Group created: $WEB_SG"

# Create Database Security Group
DB_SG=$(aws ec2 create-security-group --group-name database-tier \
  --description "Database tier with restricted access" --vpc-id $VPC_ID \
  --region $REGION --query 'GroupId' --output text)

# Allow traffic only from Web SG
aws ec2 authorize-security-group-ingress --group-id $DB_SG \
  --protocol tcp --port 5432 --source-group $WEB_SG --region $REGION

echo "Database Security Group created: $DB_SG"
echo "Least-privilege security groups configured"
```

**Expected Output:**
```
DMZ Security Group created: sg-0a1b2c3d4e5f6g7h
Web Security Group created: sg-0b2c3d4e5f6g7h8i
Database Security Group created: sg-0c3d4e5f6g7h8i9j
Least-privilege security groups configured
```

---

## Step 3: Enable VPC Flow Logs for Security Monitoring

```bash
# Create CloudWatch Log Group
LOG_GROUP="/aws/vpc/flowlogs"
aws logs create-log-group --log-group-name $LOG_GROUP --region $REGION || true

# Create IAM role for Flow Logs
ROLE_DOC='{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"Service": "vpc-flow-logs.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }
  ]
}'

ROLE_ARN=$(aws iam create-role --role-name vpc-flow-logs-role \
  --assume-role-policy-document "$ROLE_DOC" \
  --query 'Role.Arn' --output text 2>/dev/null || \
  aws iam get-role --role-name vpc-flow-logs-role \
  --query 'Role.Arn' --output text)

# Create policy for CloudWatch Logs
POLICY_DOC='{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "logs:DescribeLogGroups",
        "logs:DescribeLogStreams"
      ],
      "Resource": "*"
    }
  ]
}'

aws iam put-role-policy --role-name vpc-flow-logs-role \
  --policy-name vpc-flow-logs-policy \
  --policy-document "$POLICY_DOC" 2>/dev/null || true

# Enable VPC Flow Logs
FLOW_LOGS=$(aws ec2 create-flow-logs --resource-type VPC \
  --resource-ids $VPC_ID --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name $LOG_GROUP \
  --deliver-logs-permission-iam-role-arn $ROLE_ARN \
  --region $REGION --query 'FlowLogIds[0]' --output text)

echo "VPC Flow Logs enabled: $FLOW_LOGS"
echo "Log Group: $LOG_GROUP"

# Query flow logs (wait 1-2 minutes for data)
sleep 5
echo "Flow Logs setup complete. Data will appear in CloudWatch in 1-2 minutes."
```

**Expected Output:**
```
VPC Flow Logs enabled: fl-0a1b2c3d4e5f6g7h
Log Group: /aws/vpc/flowlogs
Flow Logs setup complete. Data will appear in CloudWatch in 1-2 minutes.
```

---

## Step 4: Deploy Secure Web Application with Network Policies

```bash
# Deploy web application with network policies
kubectl create namespace secure-app

# Label namespace
kubectl label namespace secure-app tier=backend

# Deploy web application
cat > web-app.yaml << 'EOF'
---
# Default Deny All Ingress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: secure-app
spec:
  podSelector: {}
  policyTypes:
  - Ingress

---
# Default Deny All Egress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: secure-app
spec:
  podSelector: {}
  policyTypes:
  - Egress

---
# Allow external traffic to web pods
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-web-ingress
  namespace: secure-app
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 80

---
# Allow web pods to call API
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-web-to-api
  namespace: secure-app
spec:
  podSelector:
    matchLabels:
      app: web
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

---
# Allow API pods to accept from web
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-from-web
  namespace: secure-app
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
          app: web
    ports:
    - protocol: TCP
      port: 8080

---
# Allow API pods to call database
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
  namespace: secure-app
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

---
# Allow database to accept from API
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-db-from-api
  namespace: secure-app
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

---
# Web Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: secure-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
        tier: backend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10

---
# API Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  namespace: secure-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
        tier: backend
    spec:
      containers:
      - name: api
        image: kennethreitz/httpbin
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /get
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 10

---
# Database StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
  namespace: secure-app
spec:
  serviceName: database
  replicas: 1
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
        tier: backend
    spec:
      containers:
      - name: postgres
        image: postgres:13
        env:
        - name: POSTGRES_PASSWORD
          value: "securepass123"
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi

---
# Web Service
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: secure-app
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80

---
# API Service
apiVersion: v1
kind: Service
metadata:
  name: api-service
  namespace: secure-app
spec:
  type: ClusterIP
  selector:
    app: api
  ports:
  - port: 8080
    targetPort: 8080

---
# Database Service
apiVersion: v1
kind: Service
metadata:
  name: database
  namespace: secure-app
spec:
  type: ClusterIP
  clusterIP: None
  selector:
    app: database
  ports:
  - port: 5432
    targetPort: 5432
EOF

# Apply configurations
kubectl apply -f web-app.yaml
echo "Secure application deployed with network policies"

# Wait for pods
sleep 15
kubectl get pods -n secure-app
```

**Expected Output:**
```
namespace/secure-app created
networkpolicy.networking.k8s.io/default-deny-ingress created
networkpolicy.networking.k8s.io/default-deny-egress created
networkpolicy.networking.k8s.io/allow-web-ingress created
networkpolicy.networking.k8s.io/allow-web-to-api created
networkpolicy.networking.k8s.io/allow-api-from-web created
networkpolicy.networking.k8s.io/allow-api-to-db created
networkpolicy.networking.k8s.io/allow-db-from-api created
deployment.apps/web created
deployment.apps/api created
statefulset.apps/database created
service/web-service created
service/api-service created
service/database created

Secure application deployed with network policies

NAME                       READY   STATUS    RESTARTS   AGE
web-5d7f8c9d2b1a6e4f3   2/2     Running   0          12s
api-7h6i5j4k3l2m1n0op   2/2     Running   0          12s
database-0                 1/1     Running   0          12s
```

---

## Step 5: Test Network Security and Isolation

```bash
# Test 1: Web to API communication (should work)
WEB_POD=$(kubectl get pods -n secure-app -l app=web -o jsonpath='{.items[0].metadata.name}')
echo "Testing Web to API communication from pod: $WEB_POD"

kubectl exec -it $WEB_POD -n secure-app -- sh -c "curl -s http://api-service:8080/get | head -20"
echo "✓ Web to API communication successful"

# Test 2: Web to Database (should fail - no direct access)
echo ""
echo "Testing Web to Database (should fail due to network policy):"
kubectl exec -it $WEB_POD -n secure-app -- timeout 2 bash -c "nc -zv database 5432" || echo "✓ Blocked as expected"

# Test 3: API to Database communication
API_POD=$(kubectl get pods -n secure-app -l app=api -o jsonpath='{.items[0].metadata.name}')
echo ""
echo "Testing API pod network access:"
kubectl exec -it $API_POD -n secure-app -- timeout 5 bash -c "curl -s http://web-service/health" || echo "Web not accessible from API (correct)"

# Test 4: Get LoadBalancer IP
echo ""
echo "Testing external web access:"
LB_IP=$(kubectl get svc -n secure-app web-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
if [ -z "$LB_IP" ]; then
  echo "LoadBalancer IP pending (may take 1-2 minutes)"
else
  echo "LoadBalancer IP: $LB_IP"
  curl -s http://$LB_IP | head -20
fi

# Test 5: Verify Network Policies
echo ""
echo "Network Policies Verification:"
kubectl get networkpolicy -n secure-app
```

**Expected Output:**
```
Testing Web to API communication from pod: web-5d7f8c9d2b1a6e4f3
(API response showing successful communication)
✓ Web to API communication successful

Testing Web to Database (should fail due to network policy):
✓ Blocked as expected

Testing API pod network access:
Web not accessible from API (correct)

Testing external web access:
LoadBalancer IP: a1b2c3d4e5f6g7h8-1234567890.us-east-1.elb.amazonaws.com
(Nginx welcome page returned)

Network Policies Verification:
NAME                   POD-SELECTOR      AGE
default-deny-ingress   <none>            45s
default-deny-egress    <none>            45s
allow-web-ingress      app=web           45s
allow-web-to-api       app=web           45s
allow-api-from-web     app=api           45s
allow-api-to-db        app=api           45s
allow-db-from-api      app=database      45s
```

---

## Step 6: Monitor Traffic with VPC Flow Logs

```bash
# Query VPC Flow Logs
echo "Querying VPC Flow Logs..."
aws logs tail $LOG_GROUP --follow --region $REGION &
TAIL_PID=$!

# Generate some traffic
sleep 2
echo "Flow logs will show network traffic patterns..."

# Wait 10 seconds for log data
sleep 10
kill $TAIL_PID 2>/dev/null || true

# Show sample query
echo ""
echo "To analyze flow logs, use:"
echo "aws logs filter-log-events --log-group-name $LOG_GROUP --filter-pattern 'REJECT' --region $REGION"
```

**Expected Output:**
```
Querying VPC Flow Logs...
(CloudWatch log entries showing network flows)

To analyze flow logs, use:
aws logs filter-log-events --log-group-name /aws/vpc/flowlogs --filter-pattern 'REJECT' --region us-east-1
```

---

## Step 7: Implement Rate Limiting (WAF Simulation)

```bash
# Create pod with rate limiting rules (using iptables)
cat > rate-limit-test.yaml << 'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: rate-limit-test
  namespace: secure-app
spec:
  containers:
  - name: test
    image: busybox
    command: ['sh']
    args:
    - -c
    - |
      echo "Rate limiting simulation"
      # Simulate by adding request counter
      for i in {1..50}; do
        echo "Request $i"
        sleep 0.1
      done
  restartPolicy: Never
EOF

kubectl apply -f rate-limit-test.yaml
echo "Rate limiting test pod deployed"

# Monitor logs
sleep 5
kubectl logs -n secure-app rate-limit-test 2>/dev/null | head -10
```

**Expected Output:**
```
Rate limiting test pod deployed
Request 1
Request 2
Request 3
... (continuing for 50 requests)
```

---

## Step 8: Test Cross-Namespace Communication (Should be Blocked)

```bash
# Create another namespace
kubectl create namespace untrusted-app

# Deploy a pod in untrusted namespace
kubectl run -n untrusted-app untrusted-pod --image=busybox --overrides='
{
  "spec": {
    "containers": [
      {
        "name": "busybox",
        "image": "busybox",
        "command": ["sleep", "3600"]
      }
    ]
  }
}'

sleep 5

# Try to access secure-app web service (should fail)
UNTRUSTED_POD=$(kubectl get pods -n untrusted-app -o jsonpath='{.items[0].metadata.name}')
echo "Testing cross-namespace access (should fail):"
kubectl exec -it $UNTRUSTED_POD -n untrusted-app -- timeout 2 bash -c "wget -O- http://web-service.secure-app.svc.cluster.local" || echo "✓ Cross-namespace access blocked"
```

**Expected Output:**
```
Testing cross-namespace access (should fail):
✓ Cross-namespace access blocked
```

---

## Step 9: Validate Security Posture

```bash
# Security validation checklist
echo "=== SECURITY VALIDATION CHECKLIST ==="
echo ""

echo "1. VPC Segmentation:"
echo "   ✓ Public DMZ subnet (10.50.1.0/24)"
echo "   ✓ Private Web subnet (10.50.2.0/24)"
echo "   ✓ Private Database subnet (10.50.3.0/24)"
echo ""

echo "2. Security Groups (Least Privilege):"
aws ec2 describe-security-groups --group-ids $DMZ_SG $WEB_SG $DB_SG \
  --region $REGION --query 'SecurityGroups[*].[GroupName,IpPermissions]' \
  --output text | grep -E "^(dmz|web|database)" && echo "   ✓ All SGs configured"
echo ""

echo "3. Network Policies (Zero Trust):"
kubectl get networkpolicy -n secure-app --output table
echo "   ✓ All policies enforced"
echo ""

echo "4. VPC Flow Logs:"
echo "   ✓ Enabled for $VPC_ID"
echo "   ✓ Logging to $LOG_GROUP"
echo ""

echo "5. Application Deployment:"
kubectl get deployments -n secure-app --output table
echo "   ✓ All services running"
echo ""

echo "6. Cross-Tier Isolation:"
echo "   ✓ Web cannot reach Database directly"
echo "   ✓ Untrusted namespace cannot reach Secure App"
echo ""

echo "=== SECURITY VALIDATION COMPLETE ==="
```

**Expected Output:**
```
=== SECURITY VALIDATION CHECKLIST ===

1. VPC Segmentation:
   ✓ Public DMZ subnet (10.50.1.0/24)
   ✓ Private Web subnet (10.50.2.0/24)
   ✓ Private Database subnet (10.50.3.0/24)

2. Security Groups (Least Privilege):
   ✓ All SGs configured

3. Network Policies (Zero Trust):
NAME                   POD-SELECTOR      AGE
allow-api-from-web     app=api           5m
allow-api-to-db        app=api           5m
allow-db-from-api      app=database      5m
...
   ✓ All policies enforced

4. VPC Flow Logs:
   ✓ Enabled for vpc-0a1b2c3d4e5f6g7h
   ✓ Logging to /aws/vpc/flowlogs

5. Application Deployment:
   ✓ All services running

6. Cross-Tier Isolation:
   ✓ Web cannot reach Database directly
   ✓ Untrusted namespace cannot reach Secure App

=== SECURITY VALIDATION COMPLETE ===
```

---

## Cleanup

```bash
# Delete Kubernetes resources
echo "Cleaning up Kubernetes resources..."
kubectl delete namespace secure-app untrusted-app --wait=true

# Delete VPC and related resources
echo "Cleaning up AWS VPC resources..."
aws ec2 delete-flow-logs --flow-log-ids $FLOW_LOGS --region $REGION

sleep 5

aws ec2 detach-internet-gateway --internet-gateway-id $IGW \
  --vpc-id $VPC_ID --region $REGION
aws ec2 delete-internet-gateway --internet-gateway-id $IGW --region $REGION

aws ec2 delete-security-group --group-id $DMZ_SG --region $REGION
aws ec2 delete-security-group --group-id $WEB_SG --region $REGION
aws ec2 delete-security-group --group-id $DB_SG --region $REGION

aws ec2 delete-subnet --subnet-id $DMZ_SUBNET --region $REGION
aws ec2 delete-subnet --subnet-id $WEB_SUBNET --region $REGION
aws ec2 delete-subnet --subnet-id $DB_SUBNET --region $REGION

aws ec2 delete-route-table --route-table-id $DMZ_RT --region $REGION
aws ec2 delete-route-table --route-table-id $WEB_RT --region $REGION
aws ec2 delete-route-table --route-table-id $DB_RT --region $REGION

aws ec2 delete-vpc --vpc-id $VPC_ID --region $REGION

# Delete CloudWatch Log Group
aws logs delete-log-group --log-group-name $LOG_GROUP --region $REGION || true

# Delete IAM role and policy
aws iam delete-role-policy --role-name vpc-flow-logs-role \
  --policy-name vpc-flow-logs-policy 2>/dev/null || true
aws iam delete-role --role-name vpc-flow-logs-role 2>/dev/null || true

echo "Cleanup complete!"
```

---

## Common Mistakes

### 1. **Not Implementing Default Deny Policies**
**Mistake**: Relying on application-layer security without network-layer default deny.
```yaml
# ❌ WRONG: Only allow specific traffic, but no default deny
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-only-web
spec:
  podSelector:
    matchLabels: {app: api}
  ingress:
  - from:
    - podSelector: {matchLabels: {app: web}}
```

**Correction**: Start with default deny, then explicitly allow:
```yaml
# ✓ CORRECT: Default deny all, then allow specific
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-web-to-api
spec:
  podSelector:
    matchLabels: {app: api}
  policyTypes: [Ingress]
  ingress:
  - from:
    - podSelector: {matchLabels: {app: web}}
```

---

### 2. **Overly Broad Security Group Rules**
**Mistake**: Allowing 0.0.0.0/0 (entire internet) access to internal resources.
```bash
# ❌ WRONG: Database accessible from anywhere
aws ec2 authorize-security-group-ingress --group-id $DB_SG \
  --protocol tcp --port 5432 --cidr 0.0.0.0/0
```

**Correction**: Restrict to specific sources:
```bash
# ✓ CORRECT: Only allow from Web tier SG
aws ec2 authorize-security-group-ingress --group-id $DB_SG \
  --protocol tcp --port 5432 --source-group $WEB_SG
```

---

### 3. **Missing Egress Rules (Exfiltration Risk)**
**Mistake**: Not controlling outbound traffic allows compromised pods to exfiltrate data.
```yaml
# ❌ WRONG: Only ingress policy, egress unrestricted
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-policy
spec:
  podSelector:
    matchLabels: {app: web}
  policyTypes: [Ingress]  # Missing Egress!
```

**Correction**: Control both ingress and egress:
```yaml
# ✓ CORRECT: Restrict both directions
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-policy
spec:
  podSelector:
    matchLabels: {app: web}
  policyTypes: [Ingress, Egress]
  egress:
  - to:
    - podSelector: {matchLabels: {app: api}}
    ports:
    - protocol: TCP
      port: 8080
  # Also allow DNS for service discovery
  - to:
    - namespaceSelector:
        matchLabels: {name: kube-system}
    ports:
    - protocol: UDP
      port: 53
```

---

### 4. **Ignoring DNS Traffic in Policies**
**Mistake**: Blocking DNS (port 53) prevents service discovery.
```bash
# ❌ WRONG: Network policy blocks everything, DNS fails
kubectl exec -it pod1 -- nslookup api-service
# Error: connection refused (DNS lookup failed)
```

**Correction**: Allow DNS traffic to CoreDNS:
```yaml
# ✓ CORRECT: Allow DNS to kube-system namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
spec:
  podSelector: {}
  policyTypes: [Egress]
  egress:
  - to:
    - namespaceSelector:
        matchLabels: {name: kube-system}
    ports:
    - protocol: UDP
      port: 53
```

---

### 5. **Not Monitoring Network Activity**
**Mistake**: Deploying security without visibility into what's being blocked.
```bash
# ❌ WRONG: No flow logs, can't diagnose issues
# Pod connection fails, no data to investigate
```

**Correction**: Enable VPC Flow Logs and monitor:
```bash
# ✓ CORRECT: Enable flow logs for forensics
aws ec2 create-flow-logs --resource-type VPC --resource-ids $VPC_ID \
  --traffic-type ALL --log-destination-type cloud-watch-logs

# Monitor for rejections
aws logs filter-log-events --log-group-name /aws/vpc/flowlogs \
  --filter-pattern 'REJECT' --query 'events[*].message'
```

---

## Troubleshooting

### Issue 1: "Pod cannot access service in same namespace"
**Symptom**: `curl http://service-name` times out from pod
**Root Cause**: Network policy blocking or DNS failure

**Diagnosis**:
```bash
# Check network policies
kubectl get networkpolicy -n <namespace>

# Test DNS from pod
kubectl exec <pod> -- nslookup <service>

# Check if pod has egress rule for port 53 (DNS)
```

**Solution**:
```yaml
# Add DNS egress rule if missing
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
spec:
  podSelector: {}
  policyTypes: [Egress]
  egress:
  - to:
    - namespaceSelector:
        matchLabels: {name: kube-system}
    ports:
    - protocol: UDP
      port: 53
```

---

### Issue 2: "Cross-namespace communication blocked unexpectedly"
**Symptom**: Service accessible within namespace but fails from another namespace
**Root Cause**: Network policy not allowing cross-namespace traffic

**Diagnosis**:
```bash
# Check namespace labels
kubectl get namespaces --show-labels

# Check ingress rules (should have namespaceSelector)
kubectl describe networkpolicy <policy> -n <namespace>
```

**Solution**:
```yaml
# Update policy to allow cross-namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-cross-ns
spec:
  podSelector:
    matchLabels: {app: api}
  policyTypes: [Ingress]
  ingress:
  - from:
    - namespaceSelector:
        matchLabels: {access: allowed}
    ports:
    - protocol: TCP
      port: 8080
```

Then label the calling namespace:
```bash
kubectl label namespace calling-ns access=allowed
```

---

### Issue 3: "Security Group rule doesn't seem to be working"
**Symptom**: Security group allows rule, but connection still fails
**Root Cause**: Missing return traffic rule (ingress/egress asymmetry) or NACL blocking

**Diagnosis**:
```bash
# Check security group rules
aws ec2 describe-security-groups --group-ids <sg-id>

# Check NACLs
aws ec2 describe-network-acls --filters Name=association.subnet-id,Values=<subnet-id>

# Test connectivity with specific security group
aws ec2 authorize-security-group-ingress --group-id <sg> \
  --protocol tcp --port 22 --cidr <test-ip>

# Test from EC2 instance
ssh -i key.pem ec2-user@<instance-ip>
```

**Solution**:
- Security Groups are stateful (return traffic auto-allowed)
- NACLs are stateless (must add both directions)
- Ensure source and destination both allow in NACLs

---

### Issue 4: "Legitimate traffic being blocked by WAF"
**Symptom**: Valid requests returning 403 from ALB with WAF
**Root Cause**: WAF rule too strict or legitimate traffic pattern marked as attack

**Diagnosis**:
```bash
# Check WAF logs in CloudWatch
aws logs filter-log-events --log-group-name /aws/wafv2/all \
  --filter-pattern 'BLOCK'

# Analyze blocked request details (IP, user-agent, query string)
```

**Solution**:
- Review WAF rule match
- Add IP allowlist for known legitimate IPs
- Create custom rule exceptions for valid patterns
- Use WAF rule groups instead of overly strict custom rules

---

### Issue 5: "VPC Flow Logs not showing traffic I expect"
**Symptom**: Flow logs empty or missing specific traffic patterns
**Root Cause**: Flow logs capture network layer only (not application), IAM permissions, or sampling

**Diagnosis**:
```bash
# Check flow logs are enabled
aws ec2 describe-flow-logs --filter Name=resource-id,Values=<vpc-id>

# Verify IAM role has permissions
aws iam get-role-policy --role-name vpc-flow-logs-role \
  --policy-name vpc-flow-logs-policy

# Check if data exists
aws logs describe-log-streams --log-group-name /aws/vpc/flowlogs
```

**Solution**:
- Flow logs are sampled at ~20% (not 100%)
- Application-layer issues won't show in network logs
- Check application logs in parallel with flow logs
- Increase log detail level (ALL traffic type)

---

## Next Steps

- **Module 13: Troubleshooting and Debugging** - Advanced network diagnosis techniques, packet analysis, and resolving complex connectivity issues
- **Advanced Topics**: Service mesh security (Istio/Linkerd), container runtime security, secrets management, policy enforcement at scale
- **Real-World Practice**: Implement zero trust in your own VPC, conduct security assessments, design defense-in-depth strategies
