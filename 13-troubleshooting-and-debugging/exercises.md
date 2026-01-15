# 13 - Troubleshooting and Debugging - Exercises

## Exercise 1: Understand Troubleshooting Methodology (Easy)
**Objective**: Apply systematic approach to network problems.

**Instructions**:
You receive a report: "Users can't access the payment service."

Which is the correct next step in the troubleshooting process?

A) Restart all payment service pods immediately
B) Define the problem scope (when did it start, how many users affected, what error do they see?)
C) Blame the network team
D) Update the service to use a different protocol

**Correct Answer**: B - First understand the problem before attempting fixes

---

## Exercise 2: Identify the OSI Layer (Easy)
**Objective**: Recognize which layer has the issue.

**Instructions**:
A pod can ping another pod (ICMP works), but cannot resolve the pod's DNS name.

Which OSI layer has the problem?

A) Layer 3 (Network) - Routing is broken
B) Layer 7 (Application) - DNS service failure
C) Layer 1 (Physical) - Cable disconnected
D) Layer 4 (Transport) - TCP port closed

**Correct Answer**: B - DNS works at the application layer; Layer 3 (routing) is fine since ping works

---

## Exercise 3: Service Endpoint Troubleshooting (Easy)
**Objective**: Diagnose missing service endpoints.

**Instructions**:
A service named `api-service` exists but has no endpoints. What's the most likely cause?

A) The service definition is wrong
B) Pods matching the service selector don't exist or aren't running
C) The firewall is blocking endpoint creation
D) Kubernetes version is incompatible

**Correct Answer**: B - Services use selectors to find pods; if no endpoints exist, pods don't match the selector

---

## Exercise 4: DNS Troubleshooting Basics (Easy)
**Objective**: Diagnose DNS issues systematically.

**Instructions**:
You run `kubectl exec <pod> -- nslookup my-service` and get "connection timed out."

What should you check first?

A) The pod's application logs
B) CoreDNS pod status and logs in kube-system namespace
C) AWS security groups
D) The pod's network interface MTU

**Correct Answer**: B - DNS issues require checking the DNS server (CoreDNS) first

---

## Exercise 5: Network Policy Investigation (Easy)
**Objective**: Identify network policy blocking.

**Instructions**:
Pod A cannot reach Pod B in the same namespace. All other services work fine.

Which is the first diagnostic step?

A) Check if Pod B is running
B) List all network policies with `kubectl get networkpolicy -A`
C) Restart both pods
D) Check cloud provider security groups

**Correct Answer**: B - Network policies are a common cause of unexpected blocking

---

## Exercise 6: Multi-Layer Diagnosis (Medium)
**Objective**: Troubleshoot a complex connectivity issue.

**Instructions**:
You have an application receiving "Connection refused" errors when connecting to an external API (`api.example.com:443`).

Create a systematic diagnosis plan that checks:
1. Which OSI layers to test
2. What tools to use at each layer
3. What the expected outputs should be
4. How to narrow down the scope

Expected checklist:
- Layer 3: IP connectivity
- Layer 4: Port access
- Layer 7: TLS/HTTP

---

## Exercise 7: Log Analysis and Root Cause (Medium)
**Objective**: Find root cause from logs.

**Instructions**:
You see these log entries:

```
Pod A: "Waiting for Pod B to respond"
Pod B: "Listening on port 8080"
Service endpoint list: empty
Pod B labels: app=service,tier=backend
Service selector: app=service,version=v2
```

Identify the root cause and explain why:
1. Why is the service missing endpoints?
2. How would you fix this?
3. What validation would you do?

Expected answer: Pod B has `version=` label missing, doesn't match service selector `version=v2`

---

## Exercise 8: Performance Issue Diagnosis (Medium)
**Objective**: Identify network vs. application bottleneck.

**Instructions**:
Users report that requests to your service take 10 seconds sometimes, but are instant other times.

Design a diagnostic approach to determine if this is:
1. A network latency issue
2. A pod resource constraint (CPU/memory)
3. An application performance issue

What metrics and commands would you check? (Provide at least 5)

Expected tools:
- `kubectl top pods/nodes`
- `kubectl describe node <node>`
- Network latency: `ping`, `traceroute`
- Pod readiness: `kubectl describe pod <pod>`
- Connection count: `netstat`, `ss`

---

## Exercise 9: Simulate and Fix a Network Issue (Medium)
**Objective**: Create a broken scenario and diagnose.

**Instructions**:
Describe what diagnostic steps you would take for this scenario:

**Scenario**:
- Kubernetes cluster with 2 pods (web, database)
- Service created for database
- Pod trying to connect gets "Service not found" DNS error
- But service exists: `kubectl get svc` shows it

Step-by-step diagnosis:
1. Test DNS resolution from pod
2. Check CoreDNS status
3. Verify service has endpoints
4. Check pod network configuration
5. Implement fix and validate

What would each step reveal?

---

## Exercise 10: Incident Response and Documentation (Medium)
**Objective**: Create a post-incident runbook.

**Instructions**:
After diagnosing a network issue, you need to create a runbook for your team.

Create a runbook that includes:

1. **Problem Statement** - What is the issue?
2. **Symptoms** - How do users/monitoring detect it?
3. **Root Cause** - What causes this?
4. **Diagnosis Steps** - How to confirm?
5. **Fix Steps** - How to resolve?
6. **Validation** - How to verify it works?
7. **Prevention** - How to prevent recurrence?

Example format:
```
## Issue: "Pod Cannot Resolve Service DNS"

### Symptoms
- Application logs show: "connection refused" or "no such host"
- kubectl exec pod -- nslookup service-name returns error

### Root Cause Possibilities
1. CoreDNS pod not running
2. Network policy blocking DNS (port 53)
3. Pod DNS config incorrect

### Diagnosis Steps
(List each command and expected output)

### Fix
(Step-by-step fix procedure)

### Validation
(How to verify the fix works)

### Prevention
(Changes to prevent this in future)
```
