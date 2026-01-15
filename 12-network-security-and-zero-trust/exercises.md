# 12 - Network Security and Zero Trust - Exercises

## Exercise 1: Understand Zero Trust Principles (Easy)
**Objective**: Identify which statements align with zero trust principles.

**Instructions**:
Which of the following best describes "zero trust" architecture?

A) Trust all traffic within the firewall, deny external access
B) Verify every access request, assume breach, implement least privilege
C) Encrypt only sensitive data, allow everything else
D) Use passwords for all authentication

**Correct Answer**: B

---

## Exercise 2: Identify Microsegmentation Strategy (Easy)
**Objective**: Understand network segmentation approaches.

**Instructions**:
Your organization has web servers, application servers, and databases. You want to implement microsegmentation. What is the best approach?

A) Put all servers in the same subnet with one security group
B) Create separate subnets for each tier (web, app, db) with least-privilege security groups
C) Use a single security group that allows all internal traffic
D) Disable firewalls between tiers for easier debugging

**Correct Answer**: B

---

## Exercise 3: Network Policy Basics (Easy)
**Objective**: Recognize proper network policy structure.

**Instructions**:
What is the primary difference between "allow all" and "default deny" network policies?

A) They are identical
B) "Default deny" blocks everything unless explicitly allowed (more secure)
C) "Allow all" is more secure
D) They only work at the database layer

**Correct Answer**: B

---

## Exercise 4: Security Group Rule Analysis (Easy)
**Objective**: Evaluate security group configurations.

**Instructions**:
You see a security group rule: "Allow TCP 3306 from 0.0.0.0/0"
This rule is for a database. Is this appropriate?

A) Yes, it's needed for internet access
B) No, it exposes the database to the entire internet
C) Yes, if you use strong passwords
D) It depends on the database type

**Correct Answer**: B - This is a critical vulnerability. Databases should only allow traffic from authorized application servers.

---

## Exercise 5: Traffic Monitoring Importance (Easy)
**Objective**: Understand the value of visibility.

**Instructions**:
Why is enabling VPC Flow Logs important for security?

A) It prevents attacks from happening
B) It provides visibility into network traffic for forensics and anomaly detection
C) It automatically blocks malicious traffic
D) It encrypts all traffic

**Correct Answer**: B - Flow logs provide the visibility needed to detect unusual patterns and investigate security incidents.

---

## Exercise 6: Design a Segmented Network (Medium)
**Objective**: Design a secure three-tier application.

**Instructions**:
You need to design a secure AWS VPC with web servers, application servers, and PostgreSQL databases. The web tier receives traffic from the internet, the app tier processes requests, and the database tier stores data.

Define:
1. How many subnets you would create and why
2. What each security group should allow (specify ports and sources)
3. Whether you'd use NACLs and why

**Expected Answer Structure**:
- 3 subnets: Public (DMZ) for ALB, Private (Web) for App, Private (DB) for Database
- Public SG: Allow 80/443 from 0.0.0.0/0
- Web SG: Allow 8080 from Public SG only
- Database SG: Allow 5432 from Web SG only
- NACLs: Less critical than SGs for stateful apps, but good for defense-in-depth

---

## Exercise 7: Write a Kubernetes Network Policy (Medium)
**Objective**: Create network policies for microservices.

**Instructions**:
You have three microservices:
- `frontend` (web UI) - needs to accept external traffic on port 3000
- `api` (backend API) - needs to accept traffic from frontend on port 8080
- `database` (PostgreSQL) - needs to accept traffic from api on port 5432

Write network policies to implement least-privilege access. Include:
1. A default deny policy
2. Policies allowing each service to communicate only as needed
3. Consider DNS requirements (CoreDNS in kube-system)

**Expected YAML Structure**:
```yaml
# Default deny ingress
# Default deny egress
# Allow frontend to accept external traffic
# Allow frontend to call api
# Allow api to call database
# Allow DNS for all pods
```

---

## Exercise 8: Analyze a Security Incident (Medium)
**Objective**: Use logs to diagnose a security issue.

**Instructions**:
A developer reports that their application can connect to services in the same namespace but cannot access an external API endpoint outside the cluster. You have these network policies applied:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

The default deny is blocking all egress traffic. List the steps to:
1. Diagnose the issue
2. Determine what additional policy is needed
3. Write the policy to allow external API calls

---

## Exercise 9: Evaluate WAF Rules (Medium)
**Objective**: Design WAF rule strategy.

**Instructions**:
You're protecting an e-commerce website with AWS WAF. Your WAF is currently blocking 20% of legitimate user requests along with malicious ones.

What could be causing false positives and how would you fix it?

**Consider**:
- Rate limiting rules
- SQL injection detection patterns
- XSS filters
- IP reputation lists
- How to use WAF logs to identify false positives

**Expected Answer**: Analyze WAF logs to see what legitimate patterns match rules, then add exceptions or use managed rule groups instead of overly strict custom rules.

---

## Exercise 10: Design DDoS Protection Strategy (Medium)
**Objective**: Plan multi-layer DDoS defense.

**Instructions**:
Your organization runs a web application in AWS that experienced a volumetric DDoS attack. Design a defense strategy that includes:

1. **AWS Level**: What AWS services should you use? (CloudFront, Shield, WAF, Rate Limiting)
2. **Application Level**: What can the application do?
3. **Monitoring**: How would you detect ongoing attacks?
4. **Response**: What's your incident response plan?

**Expected Answer Outline**:
- CloudFront + Shield Standard (free)
- Shield Advanced for enhanced protection
- WAF with rate limiting rules
- Anycast network distribution
- Auto-scaling to absorb legitimate traffic spikes
- CloudWatch alarms for unusual traffic patterns
- Incident response runbook with IP blocking capabilities
