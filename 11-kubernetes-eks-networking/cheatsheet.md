# 11 - Kubernetes and EKS Networking - Cheatsheet

## EKS Cluster Commands

### Create and Manage Clusters

| Command | Purpose | Example |
|---------|---------|---------|
| `eksctl create cluster` | Create EKS cluster | `eksctl create cluster --name my-cluster --nodes 3` |
| `eksctl get clusters` | List clusters | `eksctl get clusters --region us-east-1` |
| `eksctl describe cluster` | Get cluster details | `eksctl describe cluster --name my-cluster` |
| `eksctl delete cluster` | Delete cluster | `eksctl delete cluster --name my-cluster` |
| `eksctl create nodegroup` | Add node group | `eksctl create nodegroup --cluster my-cluster --nodes 2` |
| `eksctl get nodegroup` | List node groups | `eksctl get nodegroup --cluster my-cluster` |

### AWS CLI Cluster Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `aws eks list-clusters` | List clusters | `aws eks list-clusters --region us-east-1` |
| `aws eks describe-cluster` | Get cluster details | `aws eks describe-cluster --name my-cluster --region us-east-1` |
| `aws eks update-kubeconfig` | Update kubectl config | `aws eks update-kubeconfig --name my-cluster --region us-east-1` |

---

## Pod and Node Management

### Query Pods

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl get pods` | List pods in namespace | `kubectl get pods -n default` |
| `kubectl get pods -A` | List all pods | `kubectl get pods -A` or `kubectl get pods --all-namespaces` |
| `kubectl get pods -o wide` | Pods with IPs and nodes | `kubectl get pods -o wide` |
| `kubectl get pods --show-labels` | Pods with labels | `kubectl get pods --show-labels` |
| `kubectl describe pod` | Pod details | `kubectl describe pod <name> -n <namespace>` |
| `kubectl logs` | Pod application logs | `kubectl logs <pod-name> -n <namespace>` |
| `kubectl exec -it` | Execute command in pod | `kubectl exec -it <pod> -n <ns> -- bash` |

### Node Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl get nodes` | List cluster nodes | `kubectl get nodes` |
| `kubectl get nodes -o wide` | Nodes with IPs | `kubectl get nodes -o wide` |
| `kubectl describe node` | Node details | `kubectl describe node <node-name>` |
| `kubectl top nodes` | Node resource usage | `kubectl top nodes` |
| `kubectl drain` | Evacuate node for maintenance | `kubectl drain <node> --ignore-daemonsets` |

---

## Service Management

### Create Services

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl expose` | Create service from deployment | `kubectl expose deployment web --type=LoadBalancer --port=80` |
| `kubectl create service` | Create service directly | `kubectl create service clusterip my-svc --tcp=80:8080` |

### Query Services

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl get svc` | List services | `kubectl get svc -n default` |
| `kubectl get svc -A` | All services | `kubectl get svc -A` |
| `kubectl get svc -o wide` | Services with details | `kubectl get svc -o wide` |
| `kubectl describe svc` | Service details | `kubectl describe svc <name>` |
| `kubectl get endpoints` | Service endpoints (pods) | `kubectl get endpoints <service-name>` |

### Service Types

| Type | Purpose | Use Case |
|------|---------|----------|
| ClusterIP | Internal only | Pod-to-pod communication |
| NodePort | External on node port | Testing, low traffic |
| LoadBalancer | Cloud LB (AWS ALB/NLB) | Production external access |
| ExternalName | DNS alias | External service access |

---

## Ingress Management

### Create and Manage Ingress

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl create ingress` | Create ingress resource | `kubectl create ingress my-ing --rule=foo.com/*=svc:80` |
| `kubectl get ingress` | List ingress resources | `kubectl get ingress -A` |
| `kubectl describe ingress` | Ingress details | `kubectl describe ingress <name>` |
| `kubectl get ingressclass` | List ingress classes | `kubectl get ingressclass` |

### Install Ingress Controller

| Command | Purpose | Example |
|---------|---------|---------|
| `helm repo add` | Add Helm repo | `helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx` |
| `helm install` | Install ingress controller | `helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-nginx` |
| `helm list` | List Helm releases | `helm list -n ingress-nginx` |
| `helm uninstall` | Remove ingress controller | `helm uninstall nginx-ingress -n ingress-nginx` |

---

## Network Policy Commands

### Create and Manage Policies

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl create networkpolicy` | Create network policy | `kubectl create networkpolicy my-policy --pod-selector app=web` |
| `kubectl get networkpolicy` | List policies | `kubectl get networkpolicy -A` |
| `kubectl get networkpolicy -A` | All namespace policies | `kubectl get networkpolicy --all-namespaces` |
| `kubectl describe networkpolicy` | Policy details | `kubectl describe networkpolicy <name>` |
| `kubectl apply -f` | Apply policy YAML | `kubectl apply -f policy.yaml` |

### Network Policy Types

| Type | Purpose | Example |
|------|---------|---------|
| Ingress | Control inbound traffic | Allow pod A to pod B |
| Egress | Control outbound traffic | Allow pod to external DB |
| Default Deny | Block all traffic | Base policy for zero-trust |

---

## DNS and Service Discovery

### DNS Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl get svc -n kube-system` | Find CoreDNS service | `kubectl get svc -n kube-system` |
| `kubectl logs -n kube-system` | CoreDNS logs | `kubectl logs -l k8s-app=kube-dns -n kube-system` |

### DNS from Pod

| Command | Purpose | Example |
|---------|---------|---------|
| `nslookup <service>` | DNS lookup from pod | `nslookup web-app-internal` |
| `dig <service>` | Detailed DNS lookup | `dig web-app-internal.default.svc.cluster.local` |
| `getent hosts` | Resolve service IP | `getent hosts web-app-internal` |

### Service DNS Names

| Format | Scope | Example |
|--------|-------|---------|
| `<service>` | Same namespace | `web-app-internal` |
| `<service>.<namespace>` | Any namespace | `web-app-internal.default` |
| `<service>.<ns>.svc.cluster.local` | Full FQDN | `web-app-internal.default.svc.cluster.local` |
| `<pod-ip>.<namespace>.pod.cluster.local` | Pod DNS | `10-0-1-10.default.pod.cluster.local` |

---

## Network Connectivity Testing

### Port Forwarding

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl port-forward` | Forward local port to pod | `kubectl port-forward pod/web-app 8080:80` |
| `kubectl port-forward svc/` | Forward to service | `kubectl port-forward svc/web-app 8080:80` |

### Direct Connectivity Tests

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl run debug` | Create debug pod | `kubectl run -it --rm debug --image=busybox` |
| `wget` from pod | Test HTTP connectivity | `wget -O- http://service-name` |
| `curl` from pod | Test HTTPS connectivity | `curl https://service-name` |
| `nc -zv` | Test port connectivity | `nc -zv service-name 80` |

### Network Policy Testing

| Command | Purpose | Example |
|---------|---------|---------|
| Create labeled pod | Deploy with labels | `kubectl run pod1 --labels=app=frontend` |
| Test with labels | Test policy enforcement | `kubectl run -it test --labels=app=frontend -- wget svc` |
| View denials | Check policy logs | `kubectl logs <pod> -n demo-app` |

---

## Deployment and Scaling

### Deployment Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl create deployment` | Create deployment | `kubectl create deployment web --image=nginx` |
| `kubectl scale deployment` | Scale replicas | `kubectl scale deployment web --replicas=3` |
| `kubectl set image` | Update image | `kubectl set image deployment/web web=nginx:1.19` |
| `kubectl rollout status` | Check rollout | `kubectl rollout status deployment/web` |
| `kubectl rollout history` | Deployment history | `kubectl rollout history deployment/web` |

### Pod Debugging

| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl get events` | View cluster events | `kubectl get events -A` |
| `kubectl describe pod` | Pod troubleshooting | `kubectl describe pod <name>` |
| `kubectl top pods` | Pod resource usage | `kubectl top pods -n <namespace>` |
| `kubectl cp` | Copy files to/from pod | `kubectl cp <pod>:/path/file ./file` |

---

## Kubernetes Networking Concepts

### CNI Plugins

| Plugin | Features | Use Case |
|--------|----------|----------|
| AWS VPC CNI | Native ENI, high pods/node | EKS default, performance |
| Calico | Network policies, BGP | Multi-cloud, security |
| Weave | Simple overlay, multi-cloud | Development, hybrid |
| Flannel | Lightweight, simple | Cost-sensitive |
| Cilium | eBPF, advanced security | High security, visibility |

### Service Discovery Methods

| Method | Scope | Example |
|--------|-------|---------|
| DNS | Internal and external | `web-app-internal.default.svc.cluster.local` |
| Environment Variables | Within same pod | `MYSQL_HOST`, `MYSQL_PORT` |
| kubectl exec | Manual testing | `nslookup service` |
| Headless Service | Direct pod access | `port: null` in service |

---

## Common Networking Patterns

### Complete Service Setup

```bash
# Create deployment
kubectl create deployment web --image=nginx --replicas=3

# Expose as ClusterIP
kubectl expose deployment web --type=ClusterIP --port=80

# Expose as NodePort
kubectl expose deployment web --type=NodePort --name=web-np --port=80

# Expose as LoadBalancer
kubectl expose deployment web --type=LoadBalancer --name=web-lb --port=80

# Get access info
kubectl get svc
kubectl get endpoints web
kubectl describe svc web-lb
```

### Ingress with Multiple Services

```bash
cat > ingress.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-svc
spec:
  rules:
  # Host-based routing
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-svc
            port:
              number: 8000
  # Path-based routing
  - host: app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-svc
            port:
              number: 8000
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
EOF

kubectl apply -f ingress.yaml
```

### Default Deny Network Policy

```bash
cat > default-deny.yaml << 'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
EOF

# Then add specific allow policies for needed traffic
kubectl apply -f default-deny.yaml
```

---

## Troubleshooting Commands Summary

| Issue | Command | Purpose |
|-------|---------|---------|
| Pod not running | `kubectl describe pod <name>` | See why pod isn't starting |
| Service not accessible | `kubectl get endpoints <svc>` | Check if pods are registered |
| DNS not resolving | `kubectl exec <pod> -- nslookup svc` | Test DNS from pod |
| Network policy blocking | `kubectl get networkpolicy -A` | Check active policies |
| Ingress not routing | `kubectl describe ingress <name>` | Verify routing rules |
| Node resources full | `kubectl top nodes` | Check node usage |
| Controller logs | `kubectl logs -n ingress-nginx -f` | Real-time troubleshooting |

---

## Quick Reference: kubectl Namespaces

```bash
# Create namespace
kubectl create namespace demo-app

# Set default namespace
kubectl config set-context --current --namespace=demo-app

# Commands with -n flag
kubectl get pods -n demo-app
kubectl apply -f file.yaml -n demo-app

# List all namespaces
kubectl get namespaces
```

## Quick Reference: Labels and Selectors

```bash
# Run pod with labels
kubectl run pod1 --labels=app=frontend,tier=web

# Show labels
kubectl get pods --show-labels

# Filter by label
kubectl get pods -l app=frontend

# Label existing pod
kubectl label pod pod1 env=prod

# Remove label
kubectl label pod pod1 env-
```
