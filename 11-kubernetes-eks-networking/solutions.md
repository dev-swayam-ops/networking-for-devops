# 11 - Kubernetes and EKS Networking - Solutions

## Exercise 1: Query EKS Cluster Information

### Solution

**List all EKS clusters:**
```bash
# Using eksctl
eksctl get clusters --region us-east-1

# Using AWS CLI
aws eks list-clusters --region us-east-1
```

**Get cluster details:**
```bash
CLUSTER_NAME="training-cluster"
REGION="us-east-1"

# Get cluster information
aws eks describe-cluster \
  --name $CLUSTER_NAME \
  --region $REGION \
  --query 'cluster.[name,status,endpoint,resourcesVpcConfig.vpcId]'

# Get more detailed info
eksctl get cluster $CLUSTER_NAME --region $REGION
```

**List all nodes:**
```bash
# Using kubectl
kubectl get nodes -o wide

# Using eksctl
eksctl get nodegroup --cluster=$CLUSTER_NAME --region $REGION

# Get node details with IPs
kubectl get nodes -o jsonpath='{.items[*].status.addresses}' | jq
```

**Expected Output:**
```
CLUSTER: training-cluster
STATUS: ACTIVE
ENDPOINT: https://abc123.eks.us-east-1.amazonaws.com

NODES:
NAME                                      STATUS   ROLES    AGE     VERSION
ip-192-168-1-100.ec2.internal            Ready    <none>   2h      v1.29.0
ip-192-168-2-50.ec2.internal             Ready    <none>   2h      v1.29.0
```

### Explanation
- EKS provides AWS-managed Kubernetes control plane
- Nodes run in customer's VPC (visible in EC2 console)
- Each node has EC2 security group for network control
- Control plane endpoint is public HTTPS API

---

## Exercise 2: Examine Pod Networking

### Solution

**Deploy test pod:**
```bash
# Create simple nginx pod
kubectl run test-pod --image=nginx:latest

# Get pod details
kubectl get pods -o wide

# Get pod IP address
POD_IP=$(kubectl get pod test-pod -o jsonpath='{.status.podIP}')
echo "Pod IP: $POD_IP"
```

**Check pod details:**
```bash
# Full pod information
kubectl describe pod test-pod

# Get node name
NODE=$(kubectl get pod test-pod -o jsonpath='{.spec.nodeName}')
echo "Node: $NODE"

# Get all pods with IPs
kubectl get pods -A -o wide | grep test-pod
```

**Expected Output:**
```
NAME: test-pod
NAMESPACE: default
IP: 10.0.1.42
NODE: ip-192-168-1-100.ec2.internal

All pods:
NAMESPACE   NAME       READY STATUS  IP         NODE
default     test-pod   1/1   Running 10.0.1.42  ip-192-168-1-100.ec2.internal
```

### Explanation
- Each pod gets unique IP from VPC CIDR range
- IPs assigned by CNI plugin (AWS VPC CNI in EKS)
- Pods can communicate directly using IPs
- IP allocation tied to ENIs on EC2 node
- Pod IPs are ephemeral (change on restart)

---

## Exercise 3: Query Services and Service Types

### Solution

**List all services:**
```bash
# List services in all namespaces
kubectl get svc -A

# List services in specific namespace
kubectl get svc -n default

# Show more details
kubectl get svc -o wide
```

**Get service details:**
```bash
SERVICE_NAME="kubernetes"
NAMESPACE="default"

# Describe service
kubectl describe svc $SERVICE_NAME -n $NAMESPACE

# Get service YAML
kubectl get svc $SERVICE_NAME -n $NAMESPACE -o yaml

# Check endpoints
kubectl get endpoints $SERVICE_NAME -n $NAMESPACE
```

**Show service types:**
```bash
# Query by type
kubectl get svc -o jsonpath='{.items[*].spec.type}'

# Show all with type
kubectl get svc -A -o custom-columns=NAME:.metadata.name,TYPE:.spec.type,CLUSTER-IP:.spec.clusterIP,EXTERNAL-IP:.status.loadBalancer.ingress[0].hostname
```

**Expected Output:**
```
NAMESPACE     NAME         TYPE        CLUSTER-IP      EXTERNAL-IP
default       kubernetes   ClusterIP   10.100.0.1      <none>
kube-system   kube-dns     ClusterIP   10.100.0.10     <none>
monitoring    prometheus   LoadBalancer 10.100.50.100  a1b2c3d4e5f6.elb.aws.amazonaws.com

ENDPOINTS (for kubernetes service):
10.100.0.1:443
```

### Explanation
- ClusterIP: Internal service, only accessible within cluster
- NodePort: Exposed on each node's port (30000-32767 range)
- LoadBalancer: Creates AWS load balancer (NLB/ALB)
- ExternalName: DNS alias for external services
- Endpoints show which pods service is routing to

---

## Exercise 4: Examine Ingress Controllers

### Solution

**Check ingress controller installation:**
```bash
# List ingress classes
kubectl get ingressclass

# Check ingress controllers running
kubectl get deployment -A | grep ingress

# Get ingress controller pods
kubectl get pods -n ingress-nginx
```

**List ingress resources:**
```bash
# All ingresses
kubectl get ingress -A

# Ingresses in specific namespace
kubectl get ingress -n default

# Detailed info
kubectl describe ingress -n default
```

**Check ingress routing:**
```bash
INGRESS_NAME="web-app-ingress"
NAMESPACE="default"

# Get ingress rules
kubectl get ingress $INGRESS_NAME -n $NAMESPACE -o yaml | grep -A 10 rules

# Get ingress address
kubectl get ingress $INGRESS_NAME -n $NAMESPACE -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

**Check controller logs:**
```bash
# View ingress controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx --tail=50

# Watch real-time logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx -f
```

**Expected Output:**
```
INGRESSCLASS: nginx (default)
INGRESS: web-app-ingress
BACKEND SERVICE: web-app-internal:80
ADDRESS: a1b2c3d4e5f6-9876543210.us-east-1.elb.aws.amazon.com

RULES:
- Host: app.example.local
  Paths:
    - path: /
      service: web-app-internal:80
```

### Explanation
- Ingress controller watches Ingress resources and configures load balancer
- Each Ingress creates/updates ALB or NLB on AWS
- Rules define routing based on hostname and path
- Controller automatically manages SSL certificates
- Address shows where load balancer is accessible

---

## Exercise 5: Analyze Network Policies

### Solution

**List network policies:**
```bash
# All policies
kubectl get networkpolicy -A

# Policies in namespace
kubectl get networkpolicy -n demo-app

# Show labels/selectors
kubectl get networkpolicy -A -o wide
```

**Get policy details:**
```bash
POLICY_NAME="default-deny-all"
NAMESPACE="demo-app"

# Describe policy
kubectl describe networkpolicy $POLICY_NAME -n $NAMESPACE

# Get policy YAML
kubectl get networkpolicy $POLICY_NAME -n $NAMESPACE -o yaml

# Show rules
kubectl get networkpolicy $POLICY_NAME -n $NAMESPACE -o jsonpath='{.spec.ingress}'
```

**Check if policies are enforced:**
```bash
# Check CNI plugin (must support network policies)
kubectl get daemonset -n kube-system -l k8s-app=aws-node

# Check if Calico installed (adds policy support)
kubectl get daemonset -n kube-system | grep calico

# View policy on specific pod
POD_NAME="web-app-xxxxx"
kubectl get pod $POD_NAME -o yaml | grep networkPolicy
```

**Expected Output:**
```
NAMESPACE   NAME            POD-SELECTOR        AGE
demo-app    default-deny    <none>              2h
demo-app    allow-frontend  app=frontend        2h

POLICY: default-deny-all
TYPE: Ingress
POD SELECTOR: (all pods in namespace)
RULES: (none - denies all ingress)

POLICY: allow-frontend-to-backend
INGRESS RULES:
  From pods labeled: role=frontend
  To ports: 8080/tcp
```

### Explanation
- Network policies implement zero-trust networking
- Default deny blocks all traffic, then allow specific flows
- Policies use label selectors to target pods
- CIDR-based rules for external traffic
- Enforcement requires CNI plugin support (Calico, Cilium)

---

## Exercise 6: Deploy Multi-Service Application

### Solution

**Create namespace:**
```bash
kubectl create namespace demo-app
```

**Deploy backend:**
```bash
kubectl create deployment backend \
  --image=httpbin:latest \
  -n demo-app

# Scale to 3 replicas
kubectl scale deployment backend \
  --replicas=3 \
  -n demo-app
```

**Deploy frontend:**
```bash
kubectl create deployment frontend \
  --image=nginx:latest \
  -n demo-app

# Scale to 2 replicas
kubectl scale deployment frontend \
  --replicas=2 \
  -n demo-app
```

**Create services:**
```bash
# ClusterIP for backend (internal)
kubectl expose deployment backend \
  --type=ClusterIP \
  --port=80 \
  --target-port=80 \
  -n demo-app

# NodePort for frontend (node access)
kubectl expose deployment frontend \
  --type=NodePort \
  --port=80 \
  --target-port=80 \
  -n demo-app

# LoadBalancer for external access
kubectl expose deployment frontend \
  --type=LoadBalancer \
  --name=frontend-lb \
  --port=80 \
  --target-port=80 \
  -n demo-app
```

**Verify services:**
```bash
# List services
kubectl get svc -n demo-app

# Check endpoints
kubectl get endpoints -n demo-app

# Get external IPs
kubectl get svc -n demo-app -o wide
```

**Test connectivity:**
```bash
# Test from within cluster
kubectl run -it --rm debug --image=busybox --restart=Never -n demo-app -- wget -O- http://backend

# Get LoadBalancer URL
LB_URL=$(kubectl get svc frontend-lb -n demo-app -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
curl http://$LB_URL
```

**Expected Output:**
```
SERVICES:
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP                          PORT(S)
backend       ClusterIP   10.100.50.1    <none>                               80/TCP
frontend      NodePort    10.100.50.2    <nodes>                              80:30123/TCP
frontend-lb   LoadBalancer 10.100.50.3   a1b2c3d-1234567890.elb.aws.com      80:31234/TCP

ENDPOINTS:
backend       10.0.1.10:80,10.0.1.11:80,10.0.2.10:80
frontend      10.0.1.20:80,10.0.2.20:80
```

---

## Exercise 7: Create and Configure Ingress

### Solution

**Install ingress controller (if not present):**
```bash
# Add Helm repo
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Install
helm install nginx-ingress ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace
```

**Create multiple services:**
```bash
# API backend
kubectl create deployment api \
  --image=httpbin:latest \
  -n demo-app

kubectl expose deployment api \
  --type=ClusterIP \
  --port=8000 \
  --target-port=80 \
  -n demo-app

# Web frontend
kubectl create deployment web \
  --image=nginx:latest \
  -n demo-app

kubectl expose deployment web \
  --type=ClusterIP \
  --port=80 \
  --target-port=80 \
  -n demo-app
```

**Create Ingress resource:**
```bash
cat > ingress.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
  namespace: demo-app
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  # Host-based routing
  - host: api.example.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 8000
  - host: web.example.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 80
  # Path-based routing
  - host: app.example.local
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 8000
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 80
EOF

kubectl apply -f ingress.yaml
```

**Verify ingress:**
```bash
# Check ingress status
kubectl get ingress -n demo-app

# Describe ingress
kubectl describe ingress demo-ingress -n demo-app

# Get ingress controller address
kubectl get svc -n ingress-nginx
```

**Test routing:**
```bash
INGRESS_IP=$(kubectl get svc -n ingress-nginx nginx-ingress-ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

# Test host-based routing
curl -H "Host: api.example.local" http://$INGRESS_IP
curl -H "Host: web.example.local" http://$INGRESS_IP

# Test path-based routing
curl -H "Host: app.example.local" http://$INGRESS_IP/api
curl -H "Host: app.example.local" http://$INGRESS_IP/
```

**Expected Output:**
```
INGRESS:
NAME           CLASS    HOSTS                                                      ADDRESS
demo-ingress   nginx    api.example.local,web.example.local,app.example.local    a1b2c3d4e5f6.elb.aws.com

ROUTING:
api.example.local → api:8000 (HTTP 200)
web.example.local → web:80 (HTTP 200)
app.example.local/api → api:8000 (HTTP 200)
app.example.local/ → web:80 (HTTP 200)
```

---

## Exercise 8: Implement Network Policies

### Solution

**Create default-deny policy:**
```bash
cat > default-deny.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: demo-app
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
EOF

kubectl apply -f default-deny.yaml
```

**Create allow policies:**
```bash
# Allow frontend to ingress from external
cat > allow-frontend-ingress.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-ingress
  namespace: demo-app
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: ingress-controller
    ports:
    - protocol: TCP
      port: 80
EOF

# Allow frontend to backend
cat > allow-frontend-to-backend.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: demo-app
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8000
EOF

# Allow DNS egress
cat > allow-dns-egress.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
  namespace: demo-app
spec:
  podSelector: {}
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
EOF

kubectl apply -f allow-frontend-ingress.yaml
kubectl apply -f allow-frontend-to-backend.yaml
kubectl apply -f allow-dns-egress.yaml
```

**Verify policies:**
```bash
# List policies
kubectl get networkpolicy -n demo-app

# Check policy details
kubectl describe networkpolicy default-deny-all -n demo-app
```

**Test policies:**
```bash
# This should fail (blocked by default-deny)
kubectl run -it --rm test --image=busybox --restart=Never -n demo-app -- wget -O- http://frontend

# After allowing, this should work
kubectl run -it --rm test --image=busybox --restart=Never --labels="app=frontend" -n demo-app -- wget -O- http://backend
```

**Expected Output:**
```
POLICIES:
NAME                         POD-SELECTOR   AGE
default-deny-all            <none>         2m
allow-frontend-ingress      app=frontend   2m
allow-frontend-to-backend   app=backend    2m
allow-dns-egress            <none>         2m

POLICY EFFECTS:
- All pods: Deny all ingress and egress
- frontend pods: Allow ingress on port 80, allow egress to backend:8000
- backend pods: Allow ingress from frontend:80
- All pods: Allow DNS queries to kube-system
```

---

## Exercise 9: Configure Service Discovery and DNS

### Solution

**Test service DNS resolution:**
```bash
# Deploy test pod
kubectl run dns-test --image=busybox:latest -n demo-app

# Exec into pod and test DNS
kubectl exec -it dns-test -n demo-app -- nslookup web-app-internal.demo-app.svc.cluster.local

# Short name (within same namespace)
kubectl exec -it dns-test -n demo-app -- nslookup web-app-internal

# Get CoreDNS info
kubectl get pods -n kube-system | grep coredns
```

**Create ExternalName service:**
```bash
cat > external-service.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: external-db
  namespace: demo-app
spec:
  type: ExternalName
  externalName: db.example.com
  ports:
  - port: 5432
    targetPort: 5432
EOF

kubectl apply -f external-service.yaml

# Test DNS
kubectl exec -it dns-test -n demo-app -- nslookup external-db
```

**Check DNS configuration:**
```bash
# Get CoreDNS config
kubectl get configmap -n kube-system coredns -o yaml | grep -A 20 .:53

# Check resolv.conf in pod
kubectl exec -it dns-test -n demo-app -- cat /etc/resolv.conf

# Monitor DNS queries
kubectl logs -n kube-system -l k8s-app=kube-dns -f
```

**Expected Output:**
```
nslookup web-app-internal.demo-app.svc.cluster.local:
Server:         10.100.0.10
Address:        10.100.0.10#53
Name:   web-app-internal.demo-app.svc.cluster.local
Address: 10.100.50.100

nslookup external-db.demo-app.svc.cluster.local:
Server:         10.100.0.10
Address:        10.100.0.10#53
Name:   external-db.demo-app.svc.cluster.local
Address: 1.2.3.4  (resolves to db.example.com)

resolv.conf:
nameserver 10.100.0.10
search demo-app.svc.cluster.local svc.cluster.local cluster.local
```

---

## Exercise 10: Troubleshoot Kubernetes Networking Issues

### Solutions and Diagnostics

#### Scenario A: Pods Cannot Reach External Services

**Symptoms:**
- Pods cannot access external APIs/databases
- Pods can communicate with each other
- External services are reachable from host

**Troubleshooting Steps:**
```bash
# 1. Check egress network policy
kubectl get networkpolicy -A | grep egress

# 2. Test DNS from pod
kubectl exec -it <pod-name> -- nslookup 8.8.8.8

# 3. Check security group on nodes
aws ec2 describe-security-groups --group-ids sg-xxxxx

# 4. Check node egress rules
kubectl describe node <node-name> | grep -i security

# 5. Verify NAT Gateway/IGW route
kubectl exec -it <pod-name> -- curl -I https://www.example.com
```

**Root Causes & Solutions:**
```
1. Egress network policy blocking
   → Fix: Add egress rule to allow external traffic
   
2. Security group blocking outbound
   → Fix: Add egress rule to security group
   
3. No default route to IGW/NAT
   → Fix: Verify route table has 0.0.0.0/0 → NAT/IGW
   
4. DNS not working
   → Fix: Test from pod: nslookup 8.8.8.8
```

---

#### Scenario B: Service Shows Endpoints but No Pods Respond

**Symptoms:**
- kubectl get endpoints shows pod IPs
- Service exists and has ClusterIP
- Connecting to service returns no response

**Troubleshooting Steps:**
```bash
# 1. Verify endpoint IPs are valid
kubectl get endpoints <service> -o yaml

# 2. Check if pods are actually running
kubectl get pods -o wide | grep <endpoints>

# 3. Connect directly to pod IP
POD_IP=$(kubectl get pod <pod> -o jsonpath='{.status.podIP}')
kubectl run -it debug --image=busybox --restart=Never -- wget -O- http://$POD_IP

# 4. Check pod logs
kubectl logs <pod-name>

# 5. Verify service selector
kubectl get svc <service> -o yaml | grep -A 3 selector

# 6. Check pod labels
kubectl get pod <pod> --show-labels
```

**Root Causes & Solutions:**
```
1. Pod crashed or not ready
   → Fix: kubectl logs, restart pod
   
2. Service selector doesn't match pod labels
   → Fix: Update pod labels or service selector
   
3. Port mismatch (service port vs container port)
   → Fix: Verify targetPort matches container port
   
4. Pod security group blocking traffic
   → Fix: Add ingress rule to security group
```

---

#### Scenario C: Ingress Shows Address but Returns 503

**Symptoms:**
- Ingress resource created and has external IP
- curl to ingress returns HTTP 503 (Service Unavailable)
- Backend services exist

**Troubleshooting Steps:**
```bash
# 1. Check ingress status
kubectl describe ingress <ingress-name>

# 2. Verify backend services exist
kubectl get svc -n <namespace> <backend-service>

# 3. Test backend service directly
kubectl get svc <backend> -o jsonpath='{.spec.clusterIP}' | xargs -I {} curl http://{}

# 4. Check ingress controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx

# 5. Verify service endpoints
kubectl get endpoints <backend-service> -n <namespace>

# 6. Check for network policies
kubectl get networkpolicy -n ingress-nginx
```

**Root Causes & Solutions:**
```
1. Backend service doesn't exist
   → Fix: Create service: kubectl expose deployment <name>
   
2. Service has no endpoints (pods not running)
   → Fix: Check pod status: kubectl get pods
   
3. Ingress rule pointing to wrong service name
   → Fix: Update ingress YAML with correct service
   
4. Service port mismatch
   → Fix: Verify ingress port matches service port
   
5. Network policy blocking ingress controller
   → Fix: Add allow rule for ingress controller
```

---

## Quick Kubectl Commands Reference

| Command | Purpose |
|---------|---------|
| `kubectl get nodes -o wide` | List nodes with IPs |
| `kubectl get pods -A -o wide` | List all pods with IPs |
| `kubectl get svc -A` | List all services |
| `kubectl describe pod <name>` | Pod details and events |
| `kubectl logs <pod>` | Pod application logs |
| `kubectl exec -it <pod> -- bash` | SSH into pod |
| `kubectl port-forward <pod> 8080:80` | Forward local port to pod |
| `kubectl get networkpolicy -A` | List network policies |
| `kubectl get ingress -A` | List ingress resources |
| `kubectl get endpoints <svc>` | List service endpoints (pods) |
