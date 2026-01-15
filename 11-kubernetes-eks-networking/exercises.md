# 11 - Kubernetes and EKS Networking - Exercises

## Exercise 1: Query EKS Cluster Information

**Difficulty:** Easy

**Objective:** Understand cluster structure and network configuration

**Task:**
1. List all EKS clusters in your region
2. Get cluster details (endpoint, VPC, subnets)
3. List all nodes in the cluster
4. Get node IP addresses and roles
5. Identify control plane endpoint URL

**Commands to Research:**
- `eksctl get clusters`
- `eksctl get nodegroup`
- `kubectl get nodes -o wide`
- `aws eks describe-cluster`

**Expected Result:**
- EKS clusters listed with names and regions
- Node information showing IPs and statuses
- Cluster endpoint accessible from kubectl
- Nodes in ready state

## Exercise 2: Examine Pod Networking

**Difficulty:** Easy

**Objective:** Understand Pod IP allocation and cluster networking

**Task:**
1. Deploy a simple nginx pod
2. Get pod IP address
3. Check which node the pod is running on
4. List all pods with their IPs
5. Identify IP CIDR blocks used for pods

**Commands to Research:**
- `kubectl run`
- `kubectl get pods -o wide`
- `kubectl get pod <name> -o yaml | grep podIP`
- `kubectl describe node`

**Expected Result:**
- Pod receives IP address from cluster network
- Pod IP visible when querying pod details
- Pods running on different nodes
- Node capacity shows maximum pod count

## Exercise 3: Query Services and Service Types

**Difficulty:** Easy

**Objective:** Understand service routing and service discovery

**Task:**
1. List all services in default namespace
2. List services in kube-system namespace
3. For each service, identify:
   - Service type (ClusterIP, NodePort, LoadBalancer)
   - Cluster IP address
   - Ports mapped to pods
4. Check service endpoints

**Commands to Research:**
- `kubectl get svc -A`
- `kubectl get endpoints`
- `kubectl describe svc <service-name>`
- Service YAML structure

**Expected Result:**
- Services listed with types and cluster IPs
- Each service has endpoints matching pod IPs
- Default services (kubernetes, dns) visible
- Service discovery through cluster DNS

## Exercise 4: Examine Ingress Controllers

**Difficulty:** Easy

**Objective:** Understand Ingress resources and controllers

**Task:**
1. Check if ingress controller is installed
2. List all ingress resources
3. Get ingress details (backend services, hosts, paths)
4. Check ingress controller logs
5. Identify load balancer created by ingress

**Commands to Research:**
- `kubectl get ingressclass`
- `kubectl get ingress -A`
- `kubectl describe ingress <name>`
- `kubectl logs -n ingress-nginx`

**Expected Result:**
- Ingress controller detected (nginx, AWS ALB, etc.)
- Ingress resources showing routing rules
- Service backends correctly referenced
- External load balancer address assigned

## Exercise 5: Analyze Network Policies

**Difficulty:** Easy

**Objective:** Understand network isolation and traffic control

**Task:**
1. List all network policies in cluster
2. Identify policies targeting specific namespaces
3. Check policy ingress and egress rules
4. Understand label selectors in policies
5. Check if default-deny policy exists

**Commands to Research:**
- `kubectl get networkpolicy -A`
- `kubectl describe networkpolicy <name>`
- Network policy YAML structure
- Label selectors in policies

**Expected Result:**
- Network policies listed by namespace
- Policies showing ingress/egress rules
- Label selectors matching pods
- Policy enforcement status visible

## Exercise 6: Deploy Multi-Service Application

**Difficulty:** Medium

**Objective:** Create application with multiple service types

**Task:**
1. Create namespace for application
2. Deploy backend and frontend deployments
3. Create ClusterIP service for backend
4. Create NodePort service for frontend
5. Create LoadBalancer service for external access
6. Verify service endpoints
7. Test connectivity between services

**Commands to Research:**
- `kubectl create namespace`
- `kubectl create deployment`
- `kubectl expose deployment`
- Service manifest creation
- `kubectl port-forward`

**Expected Result:**
- Multiple services created and running
- Services correctly expose pods
- Endpoints show running pod IPs
- Connectivity verified between services

## Exercise 7: Create and Configure Ingress

**Difficulty:** Medium

**Objective:** Set up HTTP routing with Ingress

**Task:**
1. Ensure ingress controller is installed
2. Create multiple backend services
3. Create Ingress resource with:
   - Path-based routing (/api, /web)
   - Host-based routing (api.example.com, web.example.com)
4. Verify ingress configuration
5. Test routing to different backends
6. Add TLS configuration (optional)

**Commands to Research:**
- `helm install` ingress controller
- Ingress manifest with rules
- `kubectl describe ingress`
- Ingress annotations

**Expected Result:**
- Ingress controller running
- Multiple backend services configured
- Ingress rules routing traffic correctly
- External load balancer with health checks

## Exercise 8: Implement Network Policies

**Difficulty:** Medium

**Objective:** Configure traffic control within cluster

**Task:**
1. Create namespace for application
2. Create frontend and backend pods
3. Implement default-deny policy (deny all ingress)
4. Create allow policy for frontend → backend
5. Create allow policy for external → frontend
6. Test traffic flow (should block unauthorized)
7. Verify policies in action

**Commands to Research:**
- `kubectl apply` network policy YAML
- Network policy selectors
- Ingress and egress rules
- CIDR blocks in policies

**Expected Result:**
- Network policies deployed
- Default deny blocks traffic
- Specific policies allow intended flows
- Unauthorized traffic blocked
- Policy logs show blocked connections

## Exercise 9: Configure Service Discovery and DNS

**Difficulty:** Medium

**Objective:** Understand Kubernetes DNS and service discovery

**Task:**
1. Deploy test pod in application namespace
2. Query service DNS names from pod
3. Test FQDN resolution (service.namespace.svc.cluster.local)
4. Implement ExternalName service for external database
5. Query external service DNS
6. Check CoreDNS configuration
7. Verify DNS caching behavior

**Commands to Research:**
- `kubectl exec` into pod
- `nslookup` and `dig` commands
- Service DNS naming convention
- ExternalName service type
- CoreDNS configuration

**Expected Result:**
- Service DNS names resolve correctly
- FQDN format working
- ExternalName service created and resolvable
- CoreDNS logs showing queries
- DNS caching working as expected

## Exercise 10: Troubleshoot Kubernetes Networking Issues

**Difficulty:** Medium

**Objective:** Identify and fix connectivity problems

**Task:**
You have networking issues in the cluster. For each scenario, diagnose and fix:

**Scenario A:**
- Pods cannot reach external services
- Pods can communicate with each other
- What layer is blocking traffic? How to fix?

**Scenario B:**
- Service shows endpoints but no pods respond
- Service selector configured correctly
- What could be wrong? How to verify?

**Scenario C:**
- Ingress shows address but traffic returns 503
- Backend services exist
- What could be the issue? How to debug?

**Commands to Help:**
- `kubectl describe pod/svc/ingress`
- `kubectl logs` pod and controller
- `kubectl exec` into pod for testing
- `kubectl top nodes` for resources
- Network policy review

**Expected Result:**
- Identify root cause of connectivity issue
- Understand networking layers (pods, services, ingress)
- Fix issues with specific commands
- Verify resolution with tests

## Answer Guide Available

Solutions with complete commands and expected outputs are provided in `solutions.md`.

## Tips for Success

1. **Use Labels Effectively** - Services and policies rely on labels
2. **Test Connectivity** - Use busybox pod to test from inside cluster
3. **Check All Layers** - Service, pod, network policy, node issues
4. **Monitor Logs** - Check pod logs and controller logs
5. **Use Port-Forward** - Bypass service for direct pod connection
6. **DNS Testing** - Use nslookup/dig inside pod
7. **Document State** - Keep track of IPs and service names
8. **Verify Resources** - Ensure enough CPU/memory for pods

## Learning Outcomes

After completing these exercises, you should understand:
- EKS cluster structure and node networking
- Pod IP allocation and networking model
- Service types and their use cases
- Ingress routing and load balancing
- Network policies and traffic control
- Kubernetes DNS and service discovery
- Connectivity troubleshooting techniques
- Networking best practices in Kubernetes
