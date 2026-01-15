# 11 - Kubernetes and EKS Networking

## What You'll Learn

By the end of this module, you'll be able to:
- Understand Kubernetes networking model and Pod networking
- Deploy and manage Kubernetes clusters on AWS EKS
- Configure Services (ClusterIP, NodePort, LoadBalancer)
- Implement Ingress for HTTP/HTTPS routing
- Work with Network Policies for traffic control
- Understand CNI (Container Network Interface) plugins
- Configure service discovery and DNS in Kubernetes
- Implement network security in Kubernetes clusters
- Monitor Kubernetes network traffic and performance
- Design secure, scalable Kubernetes network architectures

## Prerequisites

- Completion of modules 00-10
- Understanding of VPCs, subnets, and security groups
- Basic container concepts (Docker)
- AWS account with EKS permissions
- kubectl installed and configured
- AWS CLI v2 installed
- Command-line proficiency

## Key Concepts

### Kubernetes Networking Fundamentals

1. **Pod** - Smallest deployable unit in Kubernetes
2. **Service** - Abstraction to expose Pods to network
3. **Ingress** - HTTP/HTTPS routing to Services
4. **Network Policy** - Firewall rules within cluster
5. **CNI Plugin** - Container Network Interface implementation

### Pod Networking

1. **Pod IP Address** - Each Pod gets unique IP
2. **Cluster Network** - Internal network for Pod-to-Pod communication
3. **Container Namespace** - Isolated network stack per container
4. **Overlay Network** - Virtual network on top of infrastructure
5. **Pod-to-Pod Communication** - Direct communication possible

### Kubernetes Services

1. **ClusterIP** - Internal service (default)
2. **NodePort** - Expose on host port
3. **LoadBalancer** - Cloud provider load balancer
4. **ExternalName** - DNS CNAME record
5. **Service Discovery** - DNS names for service access

### EKS Architecture

1. **Control Plane** - AWS-managed Kubernetes API
2. **Worker Nodes** - EC2 instances running Pods
3. **VPC Integration** - Cluster runs in AWS VPC
4. **IAM Integration** - Pod identity and RBAC
5. **Security Groups** - Network isolation and rules

### Ingress Resources

1. **Ingress Controller** - Manages Ingress resources
2. **Path-based Routing** - Route by URL path
3. **Host-based Routing** - Route by domain
4. **TLS Termination** - HTTPS support
5. **Load Balancing** - Distribute traffic across Pods

### Network Policies

1. **Ingress Rules** - Inbound traffic control
2. **Egress Rules** - Outbound traffic control
3. **Label Selectors** - Target Pods by labels
4. **Namespace Isolation** - Multi-tenant separation
5. **Default Deny** - Zero-trust networking model

### CNI Plugins

1. **AWS VPC CNI** - Native VPC networking for EKS
2. **Calico** - Network policies and BGP routing
3. **Weave** - Simple overlay networking
4. **Flannel** - Lightweight overlay network
5. **Cilium** - eBPF-based advanced networking

### Kubernetes DNS

1. **Service DNS** - `service-name.namespace.svc.cluster.local`
2. **Pod DNS** - `pod-ip.namespace.pod.cluster.local`
3. **CoreDNS** - DNS service in cluster
4. **External DNS** - Manage AWS Route53 entries
5. **DNS Resolution** - Service discovery mechanism

## Hands-on Lab: Deploy EKS Cluster and Configure Networking

### Objective
Create an EKS cluster, configure networking, deploy sample application with multiple Services and Ingress controller.

### Prerequisites for Lab
- AWS account with EKS permissions
- AWS CLI configured with credentials
- kubectl installed
- eksctl installed (EKS cluster management tool)
- Basic kubectl knowledge

### Step 1: Install Required Tools

```bash
# Install eksctl (EKS cluster management)
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

eksctl version

# Verify kubectl is installed
kubectl version --client

# Verify AWS CLI is configured
aws sts get-caller-identity
```

**Expected Output:**
```
eksctl version: 0.172.0
Client Version: v1.29.0
Account ID: 123456789012
```

### Step 2: Create EKS Cluster

```bash
# Set variables
CLUSTER_NAME="training-cluster"
REGION="us-east-1"
NODES=2

# Create EKS cluster with managed node group
eksctl create cluster \
  --name $CLUSTER_NAME \
  --region $REGION \
  --nodes $NODES \
  --node-type t3.medium \
  --managed

# This takes 10-15 minutes

echo "Cluster creation started..."
```

**Expected Output:**
```
2024-01-15 10:30:00 [ℹ]  eksctl version 0.172.0
2024-01-15 10:30:01 [ℹ]  using region us-east-1
2024-01-15 10:30:02 [ℹ]  subnets for us-east-1a: [subnet-xxxxx] 
2024-01-15 10:30:03 [ℹ]  subnets for us-east-1b: [subnet-yyyyy]
2024-01-15 10:35:00 [✓]  EKS cluster "training-cluster" in "us-east-1" is ready
```

### Step 3: Verify Cluster and Networking

```bash
# Update kubeconfig
eksctl get cluster --name $CLUSTER_NAME --region $REGION

# Test kubectl access
kubectl cluster-info

# Check nodes
kubectl get nodes -o wide

# Check node network configuration
kubectl get nodes -o json | jq '.items[0].status.addresses'

# Check CoreDNS (cluster DNS)
kubectl get pods --namespace kube-system | grep coredns
```

**Expected Output:**
```
NAME                                          STATUS   ROLES    AGE   VERSION
ip-192-168-1-100.ec2.internal                Ready    <none>   2m    v1.29.0
ip-192-168-2-50.ec2.internal                 Ready    <none>   2m    v1.29.0

CoreDNS Running:
coredns-558bd4d5db-xxxxx                     Running    0            2m
coredns-558bd4d5db-yyyyy                     Running    0            2m
```

### Step 4: Deploy Sample Application

```bash
# Create namespace for application
kubectl create namespace demo-app

# Create simple web server deployment
cat > deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: demo-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 100m
            memory: 128Mi
          requests:
            cpu: 50m
            memory: 64Mi
EOF

# Apply deployment
kubectl apply -f deployment.yaml

# Wait for deployment to be ready
kubectl rollout status deployment/web-app -n demo-app

# Check pods
kubectl get pods -n demo-app -o wide
```

**Expected Output:**
```
NAME                       READY   STATUS    RESTARTS   AGE   IP           NODE
web-app-6b7d4f8f-xxxxx    1/1     Running   0          30s   10.0.1.10   ip-192-168-1-100.ec2.internal
web-app-6b7d4f8f-yyyyy    1/1     Running   0          30s   10.0.1.11   ip-192-168-1-100.ec2.internal
web-app-6b7d4f8f-zzzzz    1/1     Running   0          25s   10.0.2.10   ip-192-168-2-50.ec2.internal
```

### Step 5: Create Services

```bash
# Create ClusterIP service (internal)
cat > service-clusterip.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: web-app-internal
  namespace: demo-app
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: web-app
EOF

# Create NodePort service (external on node port)
cat > service-nodeport.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: web-app-nodeport
  namespace: demo-app
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
  selector:
    app: web-app
EOF

# Create LoadBalancer service (AWS NLB/ALB)
cat > service-loadbalancer.yaml << 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: web-app-lb
  namespace: demo-app
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: web-app
EOF

# Apply services
kubectl apply -f service-clusterip.yaml
kubectl apply -f service-nodeport.yaml
kubectl apply -f service-loadbalancer.yaml

# Check services
kubectl get svc -n demo-app
```

**Expected Output:**
```
NAME                  TYPE           CLUSTER-IP       EXTERNAL-IP                                          PORT(S)
web-app-internal      ClusterIP      10.100.50.100    <none>                                              80/TCP
web-app-nodeport      NodePort       10.100.100.100   <nodes>                                             80:30080/TCP
web-app-lb            LoadBalancer   10.100.150.100   a1b2c3d4e5f6-1234567890.us-east-1.elb.aws...      80:31234/TCP
```

### Step 6: Install Ingress Controller

```bash
# Add Helm repository for ingress-nginx
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Install ingress-nginx controller
helm install nginx-ingress ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=LoadBalancer

# Wait for LoadBalancer to get external IP
kubectl get svc -n ingress-nginx -w

# Get the ingress controller external IP
INGRESS_IP=$(kubectl get svc -n ingress-nginx nginx-ingress-ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo "Ingress Controller IP: $INGRESS_IP"
```

**Expected Output:**
```
NAME                                          TYPE           CLUSTER-IP       EXTERNAL-IP
nginx-ingress-ingress-nginx-controller        LoadBalancer   10.100.200.100   a1b2c3d4e5f6-9876543210.us-east-1.elb.aws...
nginx-ingress-ingress-nginx-default-backend  ClusterIP      10.100.210.100   <none>
```

### Step 7: Create Ingress Resource

```bash
# Create ingress for application
cat > ingress.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-app-ingress
  namespace: demo-app
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: app.example.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-app-internal
            port:
              number: 80
  - host: app.example.local
    http:
      paths:
      - path: /health
        pathType: Exact
        backend:
          service:
            name: web-app-internal
            port:
              number: 80
EOF

# Apply ingress
kubectl apply -f ingress.yaml

# Check ingress status
kubectl get ingress -n demo-app

# Get ingress details
kubectl describe ingress web-app-ingress -n demo-app
```

**Expected Output:**
```
NAME               CLASS    HOSTS               ADDRESS                                                 PORTS
web-app-ingress    nginx    app.example.local   a1b2c3d4e5f6-9876543210.us-east-1.elb.aws.amazon.com  80
```

### Step 8: Test Application Connectivity

```bash
# Test ClusterIP service (from within cluster)
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://web-app-internal.demo-app.svc.cluster.local

# Test NodePort service (from outside cluster)
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="ExternalIP")].address}')
curl http://$NODE_IP:30080

# Test LoadBalancer service
LB_URL=$(kubectl get svc web-app-lb -n demo-app -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
curl http://$LB_URL

# Test Ingress controller
INGRESS_URL=$(kubectl get svc -n ingress-nginx nginx-ingress-ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
curl -H "Host: app.example.local" http://$INGRESS_URL
```

**Expected Output:**
```
HTTP/1.1 200 OK
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to nginx!</title>
</head>
```

### Step 9: Monitor Network Traffic

```bash
# Check pod network interfaces
kubectl get pods -n demo-app -o wide

# Get pod IP addresses
kubectl get pods -n demo-app -o jsonpath='{.items[*].status.podIP}'

# Check service endpoints
kubectl get endpoints -n demo-app

# View service DNS
kubectl exec -it $(kubectl get pod -n demo-app -o jsonpath='{.items[0].metadata.name}') -n demo-app -- nslookup web-app-internal.demo-app.svc.cluster.local
```

**Expected Output:**
```
Endpoints:
web-app-internal        10.0.1.10:80,10.0.1.11:80,10.0.2.10:80

nslookup result:
Server:         10.100.0.10
Address:        10.100.0.10#53
Name:   web-app-internal.demo-app.svc.cluster.local
Address: 10.100.50.100
```

## Validation

After completing this lab, validate your understanding by:

1. **Cluster Created** - EKS cluster running with 2+ nodes
2. **Application Deployed** - 3 replicas of web-app running
3. **Services Working** - ClusterIP, NodePort, LoadBalancer accessible
4. **Ingress Configured** - Ingress controller installed and routing traffic
5. **Connectivity Verified** - All service types can reach pods

## Cleanup

```bash
# Delete ingress resources
kubectl delete ingress web-app-ingress -n demo-app

# Delete services
kubectl delete svc web-app-internal web-app-nodeport web-app-lb -n demo-app

# Delete deployment
kubectl delete deployment web-app -n demo-app

# Delete namespace
kubectl delete namespace demo-app

# Uninstall ingress-nginx
helm uninstall nginx-ingress --namespace ingress-nginx
kubectl delete namespace ingress-nginx

# Delete EKS cluster (takes 10-15 minutes)
eksctl delete cluster --name $CLUSTER_NAME --region $REGION
```

## Common Mistakes

1. **Wrong Namespace** - Services in different namespaces not reachable by name
2. **Missing CNI Plugin** - Pods pending because CNI not installed
3. **Incorrect Selectors** - Service labels don't match pod labels
4. **Network Policies Blocking** - Default deny blocks legitimate traffic
5. **Ingress Class Mismatch** - Ingress class doesn't match controller
6. **Resource Requests/Limits** - Pods evicted due to resource constraints
7. **DNS Caching Issues** - Old DNS entries causing connectivity issues
8. **Security Groups** - EKS security groups blocking traffic

## Troubleshooting

### "Pods Not Getting IP Addresses" Error
**Cause:** CNI plugin not initialized or ENI limit reached
**Solution:**
```bash
# Check CNI plugin status
kubectl get daemonset -n kube-system | grep aws-node

# Check Pod status
kubectl describe pod <pod-name> -n <namespace>

# Check node ENI availability
aws ec2 describe-network-interfaces --filters Name=vpc-id,Values=vpc-xxxxx

# Increase node size if hitting ENI limits
eksctl create nodegroup --cluster $CLUSTER_NAME --nodes 3
```

### "Service Cannot Reach Pods" Error
**Cause:** Service selector doesn't match pod labels
**Solution:**
```bash
# Verify pod labels
kubectl get pods -n demo-app --show-labels

# Check service selector
kubectl get svc web-app-internal -n demo-app -o yaml | grep -A 3 selector

# Verify endpoints
kubectl get endpoints web-app-internal -n demo-app

# Ensure selector matches pod labels
```

### "Ingress Not Routing Traffic" Error
**Cause:** Ingress class or service name mismatch
**Solution:**
```bash
# Check ingress status
kubectl describe ingress web-app-ingress -n demo-app

# Verify service exists and is accessible
kubectl get svc -n demo-app

# Check ingress controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx

# Verify ingress class matches controller
kubectl get ingressclass
```

### "DNS Resolution Failing" Error
**Cause:** CoreDNS not running or service not found
**Solution:**
```bash
# Check CoreDNS pods
kubectl get pods -n kube-system | grep coredns

# Test DNS from pod
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup web-app-internal.demo-app.svc.cluster.local

# Check CoreDNS logs
kubectl logs -n kube-system -l k8s-app=kube-dns

# Verify service DNS entry
kubectl get svc web-app-internal -n demo-app -o yaml
```

## Next Steps

1. **Module 12** - Learn Network Security and Zero Trust architectures
2. **Module 13** - Practice troubleshooting and debugging techniques
3. **Module 14** - Real-world networking labs and case studies
4. **Advanced:** Implement service mesh (Istio, Linkerd) for advanced networking
5. **Production:** Deploy multi-cluster setup with federation
6. **GitOps:** Implement ArgoCD for declarative cluster management
