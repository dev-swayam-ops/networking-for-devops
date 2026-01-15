# 13 - Troubleshooting and Debugging - Solutions

## Exercise 1: Understand Troubleshooting Methodology (Easy) - Solution

**Correct Answer**: B - "Define the problem scope"

**Explanation**:
Good troubleshooting always starts with understanding the problem:

**What to find out**:
- **When?** "Did it just start? Is it intermittent?"
- **Who?** "All users or specific region/account?"
- **What?** "What error exactly? 503? 404? Timeout?"
- **Scope?** "Just payment or other services too?"

**Example questions to ask**:
```bash
# Gather information
- "When did users first report this?"
- "Which payment methods fail? Credit card, PayPal, both?"
- "Do other services work?"
- "What's the exact error message?"
- "Have there been recent changes?"
```

**Why NOT the other answers**:
- A) Restarting immediately might mask the problem
- C) Blaming teams prevents solving issues
- D) Protocol changes need testing, not immediate action

**Real-World Process**:
```
1. DEFINE: Gather all symptoms and scope
2. GATHER: Check logs, metrics, status dashboards
3. ISOLATE: Narrow down to specific component
4. HYPOTHESIS: Form theory of root cause
5. TEST: Reproduce or confirm the issue
6. FIX: Implement solution
7. VALIDATE: Verify fix resolves issue
8. DOCUMENT: Create runbook for future
```

---

## Exercise 2: Identify the OSI Layer (Easy) - Solution

**Correct Answer**: B - "Layer 7 (Application) - DNS service failure"

**Explanation**:
Understanding what each result tells you:

**Layer 3 (Network) is WORKING**:
- Ping succeeds = Routing works = Layer 3 OK
- ICMP packets reach destination = Network path is good

**Layer 7 (Application) has ISSUE**:
- DNS resolution fails = DNS service problem
- DNS works at Layer 7 (Application layer)

**OSI Model Layering**:
```
Layer 7: Application (DNS, HTTP, TLS)
Layer 6: Presentation (Encryption)
Layer 5: Session (Connection state)
Layer 4: Transport (TCP/UDP ports)
Layer 3: Network (IP routing, ping works here)
Layer 2: Data Link (MAC addresses)
Layer 1: Physical (Cables)

If Layer 3 works (ping succeeds) but Layer 7 fails (DNS fails),
the issue is somewhere between Layer 4-7, most likely Layer 7 DNS.
```

**Diagnostic Flow**:
```bash
# Testing reveals:
ping pod-ip              # SUCCESS → Layer 3 routing OK
nslookup pod-name       # FAILURE → Layer 7 DNS broken

# Conclusion: Check DNS service
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

---

## Exercise 3: Service Endpoint Troubleshooting (Easy) - Solution

**Correct Answer**: B - "Pods matching the service selector don't exist or aren't running"

**Explanation**:
Kubernetes Services use label selectors to find backend pods:

**How Service Endpoints Work**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api          # ← Service looks for pods with label app=api
  ports:
  - port: 80
    targetPort: 8080
```

**Endpoint Creation**:
1. Service created with selector `app: api`
2. Kubernetes controller searches for pods
3. Any pod with `app: api` label becomes an endpoint
4. If no pods match → No endpoints

**Diagnosing Empty Endpoints**:
```bash
# Check service selector
kubectl describe svc api-service | grep Selector
# Output: Selector: app=api

# Check pods and their labels
kubectl get pods --show-labels
# If no pods have app=api label → No endpoints!

# Fix: Label the pods
kubectl label pod my-pod app=api
# Now endpoints will appear
kubectl get endpoints api-service
```

**Common Mistakes**:
```bash
# ❌ WRONG: Pod labeled incorrectly
kubectl label pod my-pod application=api  # Wrong label key

# ✓ CORRECT: Exact label match
kubectl label pod my-pod app=api  # Must match selector exactly
```

---

## Exercise 4: DNS Troubleshooting Basics (Easy) - Solution

**Correct Answer**: B - "Check CoreDNS pod status and logs"

**Explanation**:
DNS issues require checking the DNS server first:

**DNS Resolution Flow**:
```
Pod wants to connect to "my-service"
           ↓
Pod sends DNS query to 10.100.0.10 (CoreDNS)
           ↓
CoreDNS resolves and returns IP 10.100.50.100
           ↓
Pod connects to 10.100.50.100
```

**If DNS fails, check the DNS server**:
```bash
# 1. Is CoreDNS running?
kubectl get pods -n kube-system -l k8s-app=kube-dns
# Should show 2+ pods in Running state

# 2. What are the errors?
kubectl logs -n kube-system -l k8s-app=kube-dns | tail -50
# Look for: "connection refused", "error", "panic"

# 3. Can CoreDNS handle the load?
kubectl top pods -n kube-system
# If CPU/memory high, might need more replicas

# 4. Is CoreDNS accessible from pod?
kubectl exec <pod> -- nc -zv 10.100.0.10 53
# Should show: Connection successful
```

**Common DNS Issues**:
```bash
# Issue 1: CoreDNS pod crashed
kubectl get pods -n kube-system -l k8s-app=kube-dns
# Solution: kubectl delete pod <pod> to restart

# Issue 2: Network policy blocking DNS
kubectl get networkpolicy -A
# Solution: Add egress rule for port 53 to kube-system

# Issue 3: CoreDNS unable to reach upstream DNS
kubectl logs -n kube-system -l k8s-app=kube-dns | grep "upstreamserver"
# Solution: Fix nameserver configuration
```

---

## Exercise 5: Network Policy Investigation (Easy) - Solution

**Correct Answer**: B - "List all network policies"

**Explanation**:
Network policies are a common cause of unexpected blocking:

**Network Policy Blocking Pattern**:
```
If Pod A → Pod B fails BUT Pod A → Pod C works,
it's usually a network policy issue specific to Pod B.
```

**Diagnosis Steps**:
```bash
# Step 1: List all policies
kubectl get networkpolicy -A
# Output shows all policies in all namespaces

# Step 2: Check policy details
kubectl describe networkpolicy <policy-name> -n <namespace>

# Step 3: Understand the policy
# It blocks unless explicitly allowed (default-deny pattern)
# Example:
#   Ingress allows only from app=web
#   Pod B is app=database
#   Pod A is app=web
#   Result: Pod A can reach Pod B, others cannot

# Step 4: Verify pod labels match
kubectl get pods --show-labels
# Ensure Pod A has labels matching ingress rule in policy for Pod B
```

**Real Example**:
```yaml
# Network Policy on Pod B
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-web
spec:
  podSelector:
    matchLabels:
      app: database  # This applies to Pod B (database)
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: web    # Only allow from app=web
```

```bash
# Diagnosis
Pod A labels: app=web → Can reach database
Pod C labels: app=client → CANNOT reach database (policy blocks it)

Solution: Either
1. Add Pod C to allowed sources
2. Remove the policy if overly restrictive
```

---

## Exercise 6: Multi-Layer Diagnosis (Medium) - Solution

**Systematic Diagnosis Plan**:

### Layer 3 (Network/Routing)
```bash
# Command: Test IP connectivity
ping api.example.com

# Expected Output:
# Success: ICMP replies from api.example.com
# Failure: "Destination unreachable" or timeout

# What it tells you:
# - Success: Network path exists
# - Failure: Routing broken, firewall blocking, or DNS failure (if using hostname)
```

### Layer 4 (Transport/Port)
```bash
# Command: Test port access
nc -zv api.example.com 443
# or
telnet api.example.com 443

# Expected Output:
# Success: Connection successful
# Failure: Connection refused or Connection timeout

# What it tells you:
# - Success: Port is open and listening
# - Failure: Firewall blocking or service not listening
```

### Layer 7 (Application/TLS/HTTP)
```bash
# Command: Test TLS and HTTP
curl -v https://api.example.com/health

# Expected Output:
# Success: HTTP/200 with response body
# Failure: 
#   - "SSL/TLS handshake failed" → Certificate issue
#   - "Connection refused" → Service not running
#   - "HTTP 503" → Service error (check service logs)

# What it tells you:
# - Success: Full end-to-end connectivity
# - Failure: Application layer issue
```

### Complete Diagnosis Script:
```bash
#!/bin/bash
TARGET="api.example.com"
PORT="443"

echo "=== Layer 3: IP Connectivity ==="
ping -c 1 $TARGET
if [ $? -eq 0 ]; then
  echo "✓ Network path OK"
else
  echo "✗ Network path broken"
  exit 1
fi

echo ""
echo "=== Layer 4: Port Access ==="
nc -zv $TARGET $PORT
if [ $? -eq 0 ]; then
  echo "✓ Port open"
else
  echo "✗ Port closed or firewall blocking"
  exit 1
fi

echo ""
echo "=== Layer 7: TLS/HTTP ==="
curl -v https://$TARGET/health
if [ $? -eq 0 ]; then
  echo "✓ Full connectivity working"
else
  echo "✗ Application layer error"
  exit 1
fi
```

---

## Exercise 7: Log Analysis and Root Cause (Medium) - Solution

**Root Cause Analysis**:

**The Problem**:
```
Service has no endpoints despite Pod B running
```

**Why It's Happening**:
```
Service selector: app=service,version=v2
Pod B labels:    app=service,tier=backend

Pod B is MISSING the version=v2 label!
Kubernetes can't match the pod to the service.
```

**Label Matching Logic**:
```
Service REQUIRES BOTH:
  ✓ app=service (Pod B has this)
  ✗ version=v2  (Pod B is missing this)

Since ALL selector labels must match, endpoint not created.
```

**Fix**:
```bash
# Add missing label to Pod B
kubectl label pod <pod-b-name> version=v2

# Verify endpoint appears
kubectl get endpoints <service-name>
# Should now show endpoint IP
```

**Validation**:
```bash
# 1. Check service selector
kubectl describe svc <service-name> | grep Selector
# Output: Selector: app=service,version=v2

# 2. Check pod labels
kubectl get pods --show-labels
# Pod B should have: app=service,version=v2

# 3. Verify endpoints exist
kubectl get endpoints <service-name>
# Output: ENDPOINTS should show IP:port

# 4. Test connectivity
kubectl exec <pod-a> -- curl http://<service-name>/
# Should succeed
```

**Prevention**:
```yaml
# In deployment/pod template, ensure ALL selector labels are set
metadata:
  labels:
    app: service
    version: v2        # ← Don't forget version!
```

---

## Exercise 8: Performance Issue Diagnosis (Medium) - Solution

**Diagnostic Approach**:

### Issue 1: Resource Constraints
```bash
# Check if pods have CPU/memory limits
kubectl describe pod <pod-name>
# Look for: Requests, Limits

# Check current usage
kubectl top pods
# If CPU/Memory near limit → Resource bottleneck

# Check if node has resources
kubectl describe node <node>
# Look for: Memory Pressure, Disk Pressure, CPU Pressure
```

### Issue 2: Network Latency
```bash
# Measure round-trip time
kubectl exec <pod> -- ping -c 100 <backend-ip>
# High variance in RTT = Network latency issue

# Check packet loss
kubectl exec <pod> -- ping -c 1000 <backend-ip>
# Any packet loss > 1% = Network quality issue

# Trace network path
kubectl exec <pod> -- traceroute <backend-ip>
# High hop latency = Network issue
```

### Issue 3: Application Performance
```bash
# Check pod logs for slow operations
kubectl logs <pod> | grep "slow\|timeout\|error"

# Monitor active connections
kubectl exec <pod> -- netstat -an | grep ESTABLISHED | wc -l
# If connection count high = Could be connection pool issue

# Check readiness probe success
kubectl describe pod <pod> | grep "Readiness"
# If failing = Pod restarting, causing delays
```

### Decision Matrix:
```
High pod resource usage       → Resource constraint (scale up)
High network latency          → Network bottleneck (check CNI, topology)
High error rate in logs       → Application issue (check code, database)
Variable latency              → Could be resource thrashing
Consistent latency            → Could be network path (MTU, hops)
```

---

## Exercise 9: Simulate and Fix a Network Issue (Medium) - Solution

**Diagnostic Workflow**:

### Step 1: Test DNS from Pod
```bash
# Command
kubectl exec <pod> -- nslookup database-service

# Expected if working:
# Server:  10.100.0.10
# Address: 10.100.0.10:53
# Name:    database-service.default.svc.cluster.local
# Address: 10.100.50.100

# Expected if broken:
# ;; connection timed out; try again
```

**What it reveals**:
- If works: DNS is fine, problem is elsewhere
- If fails: DNS server (CoreDNS) has issue

### Step 2: Check CoreDNS Status
```bash
# Command
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns

# What it reveals**:
- Not running → CoreDNS is down
- Errors in logs → CoreDNS has internal issue
- High CPU → CoreDNS overloaded
```

### Step 3: Verify Service Has Endpoints
```bash
# Command
kubectl get endpoints database-service

# Expected if working:
# NAME                  ENDPOINTS
# database-service      10.0.1.50:5432

# Expected if broken:
# NAME                  ENDPOINTS
# database-service      <none>
```

**What it reveals**:
- No endpoints → Pod selector doesn't match any pods
- Has endpoints → Service is OK, other issue

### Step 4: Check Pod Network Configuration
```bash
# Command
kubectl describe pod <pod> | grep -A 10 "Volumes:"

# Check DNS settings
kubectl get pods <pod> -o yaml | grep -A 5 "dnsPolicy\|dnsConfig"

# What it reveals:
# dnsPolicy: Default → Uses cluster DNS
# dnsPolicy: None → Custom DNS only (might be broken config)
```

### Fix and Validate
```bash
# Fix 1: If service has no endpoints (pods don't match selector)
kubectl label pod <database-pod> app=database
kubectl get endpoints database-service  # Should show endpoint now

# Fix 2: If CoreDNS is down
kubectl delete pod -n kube-system -l k8s-app=kube-dns  # Restarts them
kubectl wait --for=condition=Ready pod -l k8s-app=kube-dns -n kube-system

# Fix 3: If pod has wrong DNS config
kubectl patch pod <pod> -p '{"spec":{"dnsPolicy":"ClusterFirst"}}'

# Validate: Test again
kubectl exec <pod> -- nslookup database-service  # Should work now
```

---

## Exercise 10: Incident Response Runbook (Medium) - Solution

**Complete Runbook Template**:

```markdown
## Issue: Pod Cannot Resolve Service DNS

### Problem Statement
Pods in the application namespace cannot resolve Kubernetes service DNS names.
Example: `nslookup my-service` returns "connection timed out"

### Symptoms
- Application logs: "connection refused: no such host"
- `kubectl exec <pod> -- nslookup <service>` hangs/times out
- Some pods affected, others not
- Issue started at [TIME] after [CHANGE]

### Root Cause Possibilities
1. CoreDNS pod crashed or unhealthy
2. Network policy blocking DNS traffic (port 53)
3. Pod configured with custom/broken DNS
4. CoreDNS service has no endpoints
5. Cluster DNS IP unreachable

### Diagnosis Steps

**Step 1: Check CoreDNS Status**
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```
Expected: 2+ pods in Running state
If: Pods not running → Go to step 4

**Step 2: Check CoreDNS Service**
```bash
kubectl get svc -n kube-system kube-dns
kubectl get endpoints -n kube-system kube-dns
```
Expected: Service exists with endpoints
If: No endpoints → CoreDNS pods not selecting correctly

**Step 3: Test DNS from Pod**
```bash
kubectl exec -it <pod> -- nslookup kubernetes.default
```
Expected: Returns IP 10.100.0.1
If: Timeout → Network policy blocking
If: Server unreachable → DNS IP wrong

**Step 4: Check for Network Policies**
```bash
kubectl get networkpolicy -n default
kubectl get networkpolicy -n kube-system
```
Expected: No policies blocking port 53 to kube-system
If: Found → Check egress rules

**Step 5: Check Pod DNS Config**
```bash
kubectl get pods <pod> -o yaml | grep -A 5 "dnsPolicy"
```
Expected: dnsPolicy: ClusterFirst
If: dnsPolicy: None → Custom DNS misconfigured
```

### Fix Steps (Choose based on diagnosis)

**Fix 1: CoreDNS Pod Not Running**
```bash
# Restart CoreDNS
kubectl delete pod -n kube-system -l k8s-app=kube-dns
# Wait for replacement
kubectl wait --for=condition=Ready pod -l k8s-app=kube-dns -n kube-system
```

**Fix 2: Network Policy Blocking**
```bash
# Add egress rule for DNS
kubectl apply -f - << EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: default
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
```

**Fix 3: Pod with Wrong DNS Config**
```bash
kubectl delete pod <pod>
# Pod will restart with correct DNS (from spec)
# Verify deployment has correct dnsPolicy
```

### Validation Steps
```bash
# 1. Test from affected pod
kubectl exec <pod> -- nslookup kubernetes.default
# Expected: Returns cluster IP

# 2. Test service resolution
kubectl exec <pod> -- nslookup my-service
# Expected: Returns service IP

# 3. Test actual connectivity
kubectl exec <pod> -- curl http://my-service:8080/health
# Expected: HTTP 200

# 4. Check application logs
kubectl logs <pod> | grep "connection\|error"
# Expected: No DNS-related errors
```

### Prevention for Future
1. Add monitoring: Alert if CoreDNS pod restarts > 2x/hour
2. Default deny + explicit allow for DNS in network policies
3. Pre-check network policies before deploying
4. Test DNS in staging before production changes
5. Document service selectors in runbooks

### Incident Classification
- **Severity**: P1 (Application-wide failure)
- **RCA Owner**: [Name]
- **Duration**: [Start time] - [End time]
- **User Impact**: [Number of users/requests affected]

### Post-Incident Actions
- [ ] Update monitoring alerts
- [ ] Document in wiki
- [ ] Training: Review with team
- [ ] Code/Config change: [What changed]
```

---

## Quick Reference: Troubleshooting Commands

```bash
# Connectivity Testing
ping <target>               # Test ICMP
traceroute <target>        # Show network path
nc -zv <host> <port>       # Test specific port
curl -v http://<target>    # Test HTTP
nslookup <service>         # Test DNS

# Kubernetes Diagnostics
kubectl describe pod <pod>          # Pod events and config
kubectl logs <pod>                  # Application logs
kubectl get events -n <namespace>   # Cluster events
kubectl exec <pod> -- <command>     # Run command in pod
kubectl top pods/nodes              # Resource usage

# Network Policy Debugging
kubectl get networkpolicy -A        # List all policies
kubectl describe networkpolicy <name> # Policy details
kubectl get pods --show-labels      # Pod labels for matching

# Service Debugging
kubectl get endpoints <service>     # Service endpoints
kubectl describe svc <service>      # Service details
```
