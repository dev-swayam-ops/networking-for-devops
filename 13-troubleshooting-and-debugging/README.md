# 13 - Troubleshooting and Debugging

## What You'll Learn

By the end of this module, you'll understand:

1. **Network Troubleshooting Methodology** - Systematic approach to diagnosing network issues using OSI model
2. **Connectivity Testing Tools** - ping, traceroute, netstat, ss, nc for diagnosing connectivity problems
3. **Packet Analysis with tcpdump** - Capture and analyze network traffic at packet level for root cause analysis
4. **Log Analysis Techniques** - Extract meaningful signals from application, system, and network logs
5. **DNS Troubleshooting** - Diagnose DNS resolution failures and service discovery issues
6. **Application Layer Debugging** - Find issues in HTTP, HTTPS, gRPC, and other application protocols
7. **Performance Diagnostics** - Identify network bottlenecks, latency, and throughput issues
8. **Container and Kubernetes Debugging** - Troubleshoot networking within Docker and Kubernetes environments
9. **AWS Network Diagnosis** - Use AWS-specific tools (VPC Flow Logs, CloudWatch, Route 53 logs) for debugging
10. **Incident Response and Documentation** - Document findings, create runbooks, and improve systems after incidents

---

## Prerequisites

Before starting this module, ensure you have:

- **Completion of Modules 06-12** - Understanding of all previous networking concepts
- **Linux Command Line Proficiency** - Comfortable with bash, grep, awk, sed, pipes
- **Docker and Kubernetes Knowledge** - Understand containers, pods, services, deployments
- **AWS Account Access** - Ability to create resources and view logs
- **Basic Packet Analysis Knowledge** - Understand TCP/IP, DNS, HTTP protocols
- **Text Editors** - Use of vim, nano, or cat for viewing files
- **SSH Access** - Ability to connect to EC2 instances or EKS nodes

---

## Key Concepts

### 1. Network Troubleshooting Methodology

**The Seven-Layer Model Approach** (OSI Model):
```
Layer 7: Application (HTTP, DNS, SMTP) → Check logs, verify service
Layer 6: Presentation (Encryption, compression) → Verify TLS, certs
Layer 5: Session (TCP connections, persistence) → Check established connections
Layer 4: Transport (TCP/UDP) → Verify ports, firewalls, policies
Layer 3: Network (IP routing, ARP) → Check routing tables, IP addresses
Layer 2: Data Link (MAC, switching) → Check ARP, MAC tables
Layer 1: Physical (Cables, signals) → Check connectivity, interference
```

**Troubleshooting Flow**:
```
1. Define the problem
   (What's broken? Who's affected? When did it start?)

2. Gather information
   (Logs, metrics, traces, network topology)

3. Narrow down scope
   (Which layer? Which component? How many systems?)

4. Form hypothesis
   (Based on evidence, what's the likely cause?)

5. Test hypothesis
   (Reproduce issue, isolate variables)

6. Implement fix
   (Test before production, document changes)

7. Validate solution
   (Verify fix works, monitor for regression)

8. Document and improve
   (Post-mortem, runbooks, prevent recurrence)
```

---

### 2. Connectivity Testing Tools

**Four Essential Tools**:

| Tool | Layer | Purpose | Example |
|------|-------|---------|---------|
| `ping` | L3/L4 | ICMP connectivity test | `ping 8.8.8.8` |
| `traceroute` | L3 | Show path to destination | `traceroute example.com` |
| `netstat`/`ss` | L4 | Show open connections | `ss -tunap` |
| `nslookup` | L7 | DNS resolution test | `nslookup example.com` |

---

### 3. Packet Capture and Analysis

**tcpdump Basics**:
```bash
# Simple capture
tcpdump -i eth0 -n port 80

# Save to file for later analysis
tcpdump -i eth0 -w capture.pcap

# Filter by protocol
tcpdump -i eth0 tcp  # TCP only
tcpdump -i eth0 udp  # UDP only
tcpdump -i eth0 icmp # ICMP only

# Filter by source/destination
tcpdump -i eth0 src 10.0.0.100
tcpdump -i eth0 dst 8.8.8.8
```

---

### 4. Log Analysis Patterns

**Three-Level Logging Strategy**:
1. **Application Logs** - What did the code do?
2. **System Logs** - What did the OS do?
3. **Network Logs** - What did the network do?

**Common Log Locations**:
```
Application: /var/log/app-name.log
System: /var/log/syslog, /var/log/messages
Network (VPC): CloudWatch Logs group
Kubernetes: kubectl logs <pod>
Container: docker logs <container>
```

---

### 5. Root Cause Analysis (RCA)

**The "5 Whys" Technique**:
```
Issue: Pod cannot connect to database
1. Why? DNS resolution failing
2. Why? CoreDNS pod not responding
3. Why? Network policy blocking DNS traffic
4. Why? Missing egress rule to kube-system namespace
5. Why? No validation process before applying policies

Root Cause: Insufficient testing before deploying network policies
Solution: Add dry-run tests, gradual rollout, rollback plan
```

---

### 6. Common Network Issues

**Connectivity Issues**:
- DNS not resolving
- Routing not working
- Firewalls blocking traffic
- Network policies too restrictive

**Performance Issues**:
- High latency (check throughput, lost packets)
- Low throughput (check bandwidth, congestion)
- Packet loss (check link quality, congestion)
- Jitter (check variable latency)

**Configuration Issues**:
- Wrong IP addresses assigned
- Security groups too permissive
- Network policies missing
- Routes not propagated

---

## Hands-On Lab: Complete Network Troubleshooting Scenario

This lab simulates real-world network problems and walks through diagnosing them systematically.

### Scenario Setup

Create a broken network environment with multiple issues to diagnose.

```bash
# Set variables
REGION="us-east-1"
NAMESPACE="troubleshooting-lab"

# Create namespace
kubectl create namespace $NAMESPACE

# Create test pods with intentional issues
cat > broken-network.yaml << 'EOF'
---
# Pod 1: Working pod
apiVersion: v1
kind: Pod
metadata:
  name: healthy-app
  namespace: troubleshooting-lab
spec:
  containers:
  - name: app
    image: nginx:latest
    ports:
    - containerPort: 80

---
# Pod 2: Pod that cannot resolve DNS
apiVersion: v1
kind: Pod
metadata:
  name: dns-issue
  namespace: troubleshooting-lab
spec:
  dnsPolicy: None
  dnsConfig:
    nameservers:
    - 1.1.1.2  # Invalid nameserver
  containers:
  - name: app
    image: busybox:latest
    command: ['sleep', '3600']

---
# Pod 3: Pod blocked by network policy
apiVersion: v1
kind: Pod
metadata:
  name: blocked-pod
  labels:
    app: untrusted
  namespace: troubleshooting-lab
spec:
  containers:
  - name: app
    image: busybox:latest
    command: ['sleep', '3600']

---
# Service with no endpoints
apiVersion: v1
kind: Service
metadata:
  name: broken-service
  namespace: troubleshooting-lab
spec:
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    app: nonexistent

---
# Network Policy blocking traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: block-untrusted
  namespace: troubleshooting-lab
spec:
  podSelector:
    matchLabels:
      app: untrusted
  policyTypes:
  - Ingress
  ingress: []  # No ingress allowed

---
# Service with endpoints
apiVersion: v1
kind: Service
metadata:
  name: healthy-service
  namespace: troubleshooting-lab
spec:
  type: ClusterIP
  selector:
    pod: healthy
  ports:
  - port: 80
    targetPort: 80

---
# Deployment for healthy service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: healthy-deploy
  namespace: troubleshooting-lab
spec:
  replicas: 2
  selector:
    matchLabels:
      pod: healthy
  template:
    metadata:
      labels:
        pod: healthy
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
EOF

kubectl apply -f broken-network.yaml
sleep 10

echo "Test environment created with intentional issues"
```

**Expected Output:**
```
pod/healthy-app created
pod/dns-issue created
pod/blocked-pod created
service/broken-service created
networkpolicy.networking.k8s.io/block-untrusted created
service/healthy-service created
deployment.apps/healthy-deploy created

Test environment created with intentional issues
```

---

## Step 1: Assess the Environment

```bash
# Check all pods
echo "=== Pod Status ==="
kubectl get pods -n $NAMESPACE

# Check all services
echo "=== Service Status ==="
kubectl get svc -n $NAMESPACE

# Check endpoints
echo "=== Service Endpoints ==="
kubectl get endpoints -n $NAMESPACE

# Check network policies
echo "=== Network Policies ==="
kubectl get networkpolicy -n $NAMESPACE
```

**Expected Output:**
```
=== Pod Status ===
NAME                              READY   STATUS    RESTARTS   AGE
healthy-app                       1/1     Running   0          30s
dns-issue                         1/1     Running   0          30s
blocked-pod                       1/1     Running   0          30s
healthy-deploy-7f5c8d9b2-abc12   1/1     Running   0          30s
healthy-deploy-7f5c8d9b2-def45   1/1     Running   0          30s

=== Service Status ===
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)
broken-service    ClusterIP   10.100.50.123    <none>        8080/TCP
healthy-service   ClusterIP   10.100.50.124    <none>        80/TCP

=== Service Endpoints ===
NAME              ENDPOINTS
broken-service    <none>
healthy-service   10.0.1.100:80, 10.0.1.101:80

=== Network Policies ===
NAME              POD-SELECTOR      AGE
block-untrusted   app=untrusted     30s
```

---

## Step 2: Diagnose DNS Issues

```bash
# Test DNS from healthy pod
HEALTHY_POD=$(kubectl get pods -n $NAMESPACE -l pod=healthy -o jsonpath='{.items[0].metadata.name}')
echo "Testing DNS from healthy pod: $HEALTHY_POD"

kubectl exec -it $HEALTHY_POD -n $NAMESPACE -- nslookup healthy-service
kubectl exec -it $HEALTHY_POD -n $NAMESPACE -- nslookup kubernetes.default

# Test DNS from broken pod
DNS_ISSUE_POD=$(kubectl get pods -n $NAMESPACE -n $NAMESPACE -o name | grep dns-issue | cut -d'/' -f2)
echo ""
echo "Testing DNS from dns-issue pod: $DNS_ISSUE_POD"
kubectl exec -it $DNS_ISSUE_POD -n $NAMESPACE -- timeout 5 nslookup healthy-service || echo "DNS FAILED (as expected)"

# Check CoreDNS
echo ""
echo "=== Checking CoreDNS Status ==="
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns --tail 20
```

**Expected Output:**
```
Testing DNS from healthy pod: healthy-deploy-7f5c8d9b2-abc12
Server:         10.100.0.10
Address:        10.100.0.10:53

Name:   healthy-service.troubleshooting-lab.svc.cluster.local
Address: 10.100.50.124

Testing DNS from dns-issue pod: dns-issue
;; connection timed out; try again
DNS FAILED (as expected)

=== Checking CoreDNS Status ===
NAME                      READY   STATUS    RESTARTS   AGE
coredns-5d78c0869d-abc12  1/1     Running   0          2h
coredns-5d78c0869d-def45  1/1     Running   0          2h
```

---

## Step 3: Diagnose Service Connectivity Issues

```bash
# Test service endpoints
echo "=== Testing Service Connectivity ==="

# Healthy service (should work)
HEALTHY_POD=$(kubectl get pods -n $NAMESPACE -l pod=healthy -o jsonpath='{.items[0].metadata.name}')

echo "Test 1: Healthy pod accessing healthy-service"
kubectl exec -it $HEALTHY_POD -n $NAMESPACE -- wget -O- http://healthy-service/
echo "✓ Success"

# Broken service (should fail - no endpoints)
echo ""
echo "Test 2: Healthy pod accessing broken-service (no endpoints)"
kubectl exec -it $HEALTHY_POD -n $NAMESPACE -- timeout 2 wget -O- http://broken-service:8080 || echo "✓ Failed as expected (no endpoints)"

# Check service details
echo ""
echo "=== Service Details ==="
kubectl describe svc broken-service -n $NAMESPACE
kubectl describe svc healthy-service -n $NAMESPACE
```

**Expected Output:**
```
=== Testing Service Connectivity ===
Test 1: Healthy pod accessing healthy-service
(Nginx welcome page returned)
✓ Success

Test 2: Healthy pod accessing broken-service (no endpoints)
✓ Failed as expected (no endpoints)

=== Service Details ===
Name:              broken-service
Namespace:         troubleshooting-lab
Labels:            <none>
Annotations:       <none>
Selector:          app=nonexistent
Type:              ClusterIP
IP:                10.100.50.123
Port:              <unset>  8080/TCP
TargetPort:        8080/TCP
Endpoints:         <none>
```

---

## Step 4: Diagnose Network Policy Issues

```bash
# Test network policy blocking
echo "=== Testing Network Policy Enforcement ==="

BLOCKED_POD=$(kubectl get pods -n $NAMESPACE -l app=untrusted -o jsonpath='{.items[0].metadata.name}')
HEALTHY_POD=$(kubectl get pods -n $NAMESPACE -l pod=healthy -o jsonpath='{.items[0].metadata.name}')

# Try to access blocked pod from healthy pod
echo "Attempt to access blocked-pod from healthy pod:"
kubectl exec -it $HEALTHY_POD -n $NAMESPACE -- timeout 2 wget -O- http://blocked-pod || echo "✓ Access denied by network policy"

# Check network policies
echo ""
echo "=== Network Policy Details ==="
kubectl describe networkpolicy block-untrusted -n $NAMESPACE

# Get pod labels (used by policy)
echo ""
echo "=== Pod Labels ==="
kubectl get pods -n $NAMESPACE --show-labels
```

**Expected Output:**
```
=== Testing Network Policy Enforcement ===
Attempt to access blocked-pod from healthy pod:
✓ Access denied by network policy

=== Network Policy Details ===
Name:         block-untrusted
Namespace:    troubleshooting-lab
Pod Selector: app=untrusted
Policies:
  Ingress:
    <none>
```

---

## Step 5: Use Packet Capture for Deep Diagnosis

```bash
# Install tcpdump in a pod if needed
HEALTHY_POD=$(kubectl get pods -n $NAMESPACE -l pod=healthy -o jsonpath='{.items[0].metadata.name}')

# Capture traffic from pod
echo "Capturing packets on port 53 (DNS)..."
kubectl exec -it $HEALTHY_POD -n $NAMESPACE -- timeout 10 sh -c "tcpdump -i any -n port 53" 2>/dev/null &
CAPTURE_PID=$!

# Generate traffic
sleep 2
kubectl exec -it $HEALTHY_POD -n $NAMESPACE -- nslookup healthy-service > /dev/null 2>&1

# Wait for capture to finish
wait $CAPTURE_PID 2>/dev/null || true

echo "✓ Packet capture complete"
```

**Expected Output:**
```
Capturing packets on port 53 (DNS)...
22:15:45.234567 IP 10.0.1.100.53456 > 10.100.0.10.53: 11111+ A? healthy-service.troubleshooting-lab.svc.cluster.local. (59)
22:15:45.234890 IP 10.100.0.10.53 > 10.0.1.100.53456: 11111 1/0/0 A 10.100.50.124 (75)

✓ Packet capture complete
```

---

## Step 6: Analyze Logs from Multiple Sources

```bash
# Application logs
echo "=== Application Logs ==="
HEALTHY_POD=$(kubectl get pods -n $NAMESPACE -l pod=healthy -o jsonpath='{.items[0].metadata.name}')
kubectl logs $HEALTHY_POD -n $NAMESPACE --tail 10

# System logs from cluster (if available)
echo ""
echo "=== Kubelet Logs ==="
kubectl get events -n $NAMESPACE --sort-by='.lastTimestamp' | tail -20

# Check resource usage (might indicate performance issues)
echo ""
echo "=== Pod Resource Usage ==="
kubectl top pods -n $NAMESPACE

# Check pod descriptions for warnings
echo ""
echo "=== Pod Events ==="
kubectl describe pod dns-issue -n $NAMESPACE | grep -A 10 "Events:"
```

**Expected Output:**
```
=== Application Logs ===
(Nginx access logs)
10.0.1.101 - - [15/Jan/2026:22:15:45 +0000] "GET / HTTP/1.1" 200 612

=== Kubelet Logs ===
<none>   Pod   Normal   Created pod

=== Pod Resource Usage ===
NAME                      CPU(cores)  MEMORY(bytes)
healthy-app               1m          2Mi
dns-issue                 1m          3Mi

=== Pod Events ===
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  5m    default-scheduler  Successfully assigned troubleshooting-lab/dns-issue to node-1
  Normal  Pulled     5m    kubelet            Container image "busybox:latest" already present
  Normal  Created    5m    kubelet            Created container app
  Normal  Started    5m    kubelet            Started container app
```

---

## Step 7: Create a Diagnostic Report

```bash
# Comprehensive diagnosis script
cat > diagnosis.sh << 'EOF'
#!/bin/bash
NAMESPACE="troubleshooting-lab"

echo "=== NETWORK DIAGNOSIS REPORT ==="
echo "Timestamp: $(date)"
echo ""

echo "1. Pod Status"
kubectl get pods -n $NAMESPACE -o wide

echo ""
echo "2. Service Status and Endpoints"
kubectl get svc,endpoints -n $NAMESPACE

echo ""
echo "3. Network Policies"
kubectl get networkpolicy -n $NAMESPACE

echo ""
echo "4. DNS Tests"
HEALTHY_POD=$(kubectl get pods -n $NAMESPACE -l pod=healthy -o jsonpath='{.items[0].metadata.name}')
echo "DNS from healthy pod:"
kubectl exec $HEALTHY_POD -n $NAMESPACE -- nslookup healthy-service 2>&1 | grep -E "Address|Name"

echo ""
echo "5. Connectivity Tests"
echo "Healthy service endpoints:"
kubectl get endpoints healthy-service -n $NAMESPACE

echo ""
echo "6. Resource Usage"
kubectl top pods -n $NAMESPACE

echo ""
echo "7. Recent Events"
kubectl get events -n $NAMESPACE --sort-by='.lastTimestamp' | head -10

echo ""
echo "=== END DIAGNOSIS REPORT ==="
EOF

chmod +x diagnosis.sh
./diagnosis.sh
```

**Expected Output:**
```
=== NETWORK DIAGNOSIS REPORT ===
Timestamp: Wed Jan 15 22:15:45 UTC 2026

1. Pod Status
NAME                      READY   STATUS    RESTARTS   AGE
healthy-app               1/1     Running   0          5m
dns-issue                 1/1     Running   0          5m
blocked-pod               1/1     Running   0          5m
healthy-deploy-7f5c8d9b2  1/1     Running   0          5m

2. Service Status and Endpoints
NAME              TYPE        CLUSTER-IP       PORT(S)
broken-service    ClusterIP   10.100.50.123    8080/TCP
healthy-service   ClusterIP   10.100.50.124    80/TCP

ENDPOINTS
<none>
10.0.1.100:80,10.0.1.101:80

3. Network Policies
NAME              POD-SELECTOR      AGE
block-untrusted   app=untrusted     5m

... (additional output)

=== END DIAGNOSIS REPORT ===
```

---

## Step 8: Implement Fixes and Validate

```bash
# Fix 1: Correct DNS issue
echo "Fixing DNS pod..."
kubectl delete pod dns-issue -n $NAMESPACE
kubectl run dns-fixed --image=busybox --command -- sleep 3600 -n $NAMESPACE

# Fix 2: Create endpoints for broken service
echo "Fixing broken service..."
kubectl label pod healthy-app -n $NAMESPACE app=working
kubectl patch svc broken-service -n $NAMESPACE -p '{"spec":{"selector":{"app":"working"}}}'

# Fix 3: Update network policy to allow traffic
echo "Fixing network policy..."
kubectl patch networkpolicy block-untrusted -n $NAMESPACE \
  --type merge -p '{"spec":{"ingress":[{"from":[{"podSelector":{}}]}]}}'

# Validate fixes
echo ""
echo "=== Validation ==="
echo "1. DNS test:"
DNS_FIXED_POD=$(kubectl get pods -n $NAMESPACE -o name | grep dns-fixed | cut -d'/' -f2)
kubectl exec -it $DNS_FIXED_POD -n $NAMESPACE -- nslookup healthy-service

echo ""
echo "2. Service endpoints:"
kubectl get endpoints broken-service -n $NAMESPACE

echo ""
echo "3. Network connectivity:"
BLOCKED_POD=$(kubectl get pods -n $NAMESPACE -l app=untrusted -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $BLOCKED_POD -n $NAMESPACE -- timeout 2 wget -O- http://healthy-app || echo "✓ Still blocked (test fix needed)"
```

**Expected Output:**
```
Fixing DNS pod...
Fixing broken service...
Fixing network policy...

=== Validation ===
1. DNS test:
Server:         10.100.0.10
Address:        10.100.0.10:53

Name:   healthy-service.troubleshooting-lab.svc.cluster.local
Address: 10.100.50.124

2. Service endpoints:
NAME              ENDPOINTS
broken-service    10.0.0.50:80

3. Network connectivity:
(Nginx welcome page returned)
✓ Fix successful
```

---

## Validation Checklist

```bash
# Verify all systems operational
echo "=== FINAL VALIDATION ==="
echo ""

echo "✓ All pods running:"
kubectl get pods -n $NAMESPACE --no-headers | awk '{print $3}' | sort | uniq -c

echo ""
echo "✓ All services have endpoints:"
kubectl get endpoints -n $NAMESPACE

echo ""
echo "✓ DNS working:"
HEALTHY_POD=$(kubectl get pods -n $NAMESPACE -l pod=healthy -o jsonpath='{.items[0].metadata.name}')
kubectl exec $HEALTHY_POD -n $NAMESPACE -- nslookup kubernetes.default > /dev/null && echo "  DNS operational"

echo ""
echo "✓ Network policies enforced correctly:"
kubectl get networkpolicy -n $NAMESPACE

echo ""
echo "=== VALIDATION COMPLETE ==="
```

---

## Cleanup

```bash
# Remove test resources
kubectl delete namespace $NAMESPACE --wait=true
rm -f broken-network.yaml diagnosis.sh

echo "Cleanup complete"
```

---

## Common Mistakes

### 1. **Not Checking Service Endpoints First**
**Mistake**: Trying to debug pod issues when the real problem is missing service endpoints.
```bash
# ❌ WRONG: Directly checking pod logs
kubectl logs -l app=web

# ✓ CORRECT: First verify service has endpoints
kubectl get endpoints <service-name>
# If empty, pods don't match service selector
```

---

### 2. **Overlooking Network Policies**
**Mistake**: Assuming connectivity issues are DNS/routing when network policies are blocking.
```bash
# ❌ WRONG: Testing without checking policies
kubectl exec <pod> -- curl http://other-service

# ✓ CORRECT: Check policies first
kubectl get networkpolicy -A
kubectl describe networkpolicy <name>
# Verify pods can reach target pod based on selectors
```

---

### 3. **Not Checking All Three Logging Levels**
**Mistake**: Only checking application logs, missing network or system issues.
```bash
# ❌ WRONG: Single source of truth
kubectl logs <pod>

# ✓ CORRECT: Multi-level diagnosis
kubectl logs <pod>                          # Application
kubectl describe pod <pod>                   # System/events
kubectl exec <pod> -- curl http://service   # Network connectivity
```

---

### 4. **Ignoring Resource Constraints**
**Mistake**: Dismissing performance issues as "the network" when it's actually CPU/memory.
```bash
# ❌ WRONG: Assuming network saturation
# "It's slow, must be network"

# ✓ CORRECT: Check resources first
kubectl top nodes
kubectl top pods
kubectl describe node <node-name>
# Check for NotReady, memory pressure, disk pressure
```

---

### 5. **Not Using Isolation to Narrow Scope**
**Mistake**: Trying to debug entire system instead of isolating variables.
```bash
# ❌ WRONG: Testing everything at once
kubectl logs -n prod -l app=web | grep error

# ✓ CORRECT: Isolate and test systematically
# 1. Test single pod
kubectl exec <pod> -- curl http://localhost:8080

# 2. Test service within same namespace
kubectl exec <pod> -- curl http://<service>

# 3. Test cross-namespace if applicable
kubectl exec <pod> -- curl http://<service>.<namespace>.svc.cluster.local

# 4. Test external DNS
kubectl exec <pod> -- nslookup example.com
```

---

## Troubleshooting

### Issue 1: "Pod cannot reach service in same namespace"

**Diagnosis Steps**:
```bash
# Step 1: Verify service exists and has endpoints
kubectl get svc <service-name>
kubectl get endpoints <service-name>

# Step 2: Check pod labels match service selector
kubectl get pods --show-labels
kubectl describe svc <service-name>

# Step 3: Test DNS resolution
kubectl exec <pod> -- nslookup <service-name>

# Step 4: Check network policies
kubectl get networkpolicy -A
kubectl describe networkpolicy <name>

# Step 5: Test with curl/wget
kubectl exec <pod> -- curl http://<service-name>:<port>
```

**Common Solutions**:
- Service selector doesn't match pod labels
- Network policy blocking traffic (check ingress rules)
- DNS not working (check CoreDNS pod status)
- Service port mismatch (targetPort ≠ application port)

---

### Issue 2: "DNS resolution is slow"

**Diagnosis Steps**:
```bash
# Measure DNS latency
kubectl exec <pod> -- sh -c 'time nslookup <service>'

# Check CoreDNS pod status
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl top pods -n kube-system

# Check CoreDNS logs for errors
kubectl logs -n kube-system -l k8s-app=kube-dns

# Test with direct IP (to exclude DNS)
kubectl exec <pod> -- curl http://10.100.50.124:80
```

**Common Solutions**:
- CoreDNS pod overloaded (scale up replicas)
- Network misconfiguration (check iptables/ipvs)
- High cluster load (check node resources)
- DNS cache issues (clear cache, restart pod)

---

### Issue 3: "Service returns 503 errors intermittently"

**Diagnosis Steps**:
```bash
# Check pod readiness probes
kubectl describe pod <pod-name>
kubectl get pods <pod-name> -o yaml | grep -A 5 "readinessProbe"

# Check pod logs for errors
kubectl logs <pod-name>
kubectl logs <pod-name> --previous  # If crashed

# Check resource usage
kubectl top pods <pod-name>
kubectl describe node <node-name>

# Check for network issues
kubectl get networkpolicy -A
kubectl exec <pod> -- netstat -an | wc -l
```

**Common Solutions**:
- Pod unhealthy but still in service (readiness probe failing)
- Pod out of memory/CPU (increase resources)
- Network saturation (check load distribution)
- All backend pods in same AZ (spread across zones)

---

### Issue 4: "Packet loss between pods"

**Diagnosis Steps**:
```bash
# Check network interface stats
kubectl exec <pod> -- cat /proc/net/dev

# Run ping with packet loss percentage
kubectl exec <pod> -- ping -c 100 <other-pod-ip>

# Check MTU mismatch
kubectl exec <pod> -- ifconfig | grep MTU
kubectl exec <other-pod> -- ifconfig | grep MTU

# Check route to destination
kubectl exec <pod> -- traceroute <other-pod-ip>

# Check iptables/netfilter stats
kubectl exec <pod> -- cat /proc/net/netstat | grep -i drop
```

**Common Solutions**:
- MTU mismatch (typically set to 1500, check CNI plugin)
- Network congestion (check bandwidth usage)
- Faulty network interface (check node health)
- Overlay network issues (check CNI logs)

---

### Issue 5: "Cross-namespace communication fails"

**Diagnosis Steps**:
```bash
# Verify both pods exist and are running
kubectl get pods -n ns1
kubectl get pods -n ns2

# Test DNS across namespaces
kubectl exec <pod-in-ns1> -- nslookup <service>.<ns2>.svc.cluster.local

# Check network policies in both namespaces
kubectl get networkpolicy -n ns1
kubectl get networkpolicy -n ns2

# Check namespace labels (some policies reference them)
kubectl get namespaces --show-labels

# Test connectivity directly
kubectl exec <pod-in-ns1> -- curl http://<service>.<ns2>.svc.cluster.local
```

**Common Solutions**:
- Network policy blocking cross-namespace traffic (add namespaceSelector)
- Pods not found in other namespace (verify they exist)
- Service not exposed to other namespace (create headless service)
- Missing namespace labels (label namespaces, add to policy)

---

## Next Steps

- **Production Deployment** - Apply troubleshooting practices to your own infrastructure
- **Monitoring and Observability** - Set up proactive monitoring to catch issues before users notice
- **Runbook Automation** - Automate troubleshooting steps for faster incident response
- **Advanced Topics** - Service mesh observability (Istio, Linkerd), eBPF-based tracing, APM integration
- **Continuous Learning** - Stay updated with networking best practices, tool updates, community discussions
