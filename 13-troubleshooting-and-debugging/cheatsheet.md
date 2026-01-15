# 13 - Troubleshooting and Debugging - Cheatsheet

## Network Troubleshooting Commands

### Connectivity Testing
| Command | Purpose | Example |
|---------|---------|---------|
| `ping <target>` | Test ICMP reachability | `ping 10.100.50.100` |
| `ping -c <count> <target>` | Limit ping count | `ping -c 5 google.com` |
| `ping -W <timeout>` | Set timeout in ms | `ping -W 2000 target.com` |
| `ping -s <size> <target>` | Test with specific packet size | `ping -s 1500 target.com` |
| `nc -zv <host> <port>` | Check if port is open | `nc -zv api.example.com 443` |
| `telnet <host> <port>` | Connect to port | `telnet api.example.com 443` |
| `curl -v http://<url>` | Test HTTP with verbose | `curl -v http://example.com/health` |
| `curl --connect-timeout <sec> <url>` | Set connection timeout | `curl --connect-timeout 5 http://example.com` |
| `curl -I http://<url>` | Get HTTP headers only | `curl -I http://example.com` |
| `traceroute <target>` | Show network path | `traceroute google.com` |
| `traceroute -m <hops> <target>` | Max number of hops | `traceroute -m 30 target.com` |

### DNS Troubleshooting
| Command | Purpose | Example |
|---------|---------|---------|
| `nslookup <hostname>` | Simple DNS lookup | `nslookup example.com` |
| `nslookup <hostname> <dns-server>` | Query specific DNS | `nslookup example.com 8.8.8.8` |
| `dig <hostname>` | Detailed DNS query | `dig example.com` |
| `dig <hostname> +short` | Short answer format | `dig example.com +short` |
| `dig <hostname> MX` | Query MX records | `dig example.com MX` |
| `dig <hostname> NS` | Query nameservers | `dig example.com NS` |
| `dig -x <ip>` | Reverse DNS lookup | `dig -x 8.8.8.8` |
| `host <hostname>` | Simple host lookup | `host google.com` |
| `whois <domain>` | Domain WHOIS information | `whois example.com` |

### Network Configuration
| Command | Purpose | Example |
|---------|---------|---------|
| `ip addr show` | Show IP addresses | `ip addr show` |
| `ip route show` | Show routing table | `ip route show` |
| `ip link show` | Show network interfaces | `ip link show` |
| `ifconfig` | Network interface config | `ifconfig eth0` |
| `route -n` | Display routing table | `route -n` |
| `hostname -I` | Show local IP | `hostname -I` |
| `hostname` | Show hostname | `hostname` |

### Network Statistics
| Command | Purpose | Example |
|---------|---------|---------|
| `netstat -an` | Show all connections | `netstat -an` |
| `netstat -tlnp` | TCP listening ports | `netstat -tlnp` |
| `netstat -ulnp` | UDP listening ports | `netstat -ulnp` |
| `netstat -s` | Network statistics | `netstat -s` |
| `ss -an` | Sockets (faster than netstat) | `ss -an` |
| `ss -tlnp` | TCP listening with process | `ss -tlnp` |
| `ss -s` | Socket statistics | `ss -s` |
| `netstat -i` | Interface statistics | `netstat -i` |

---

## Packet Capture and Analysis

### tcpdump Capture
| Command | Purpose | Example |
|---------|---------|---------|
| `tcpdump -i <interface>` | Capture on interface | `tcpdump -i eth0` |
| `tcpdump -i any` | Capture on all interfaces | `tcpdump -i any` |
| `tcpdump -c <count>` | Limit packet count | `tcpdump -c 100 -i eth0` |
| `tcpdump -w <file>` | Write to file | `tcpdump -w capture.pcap -i eth0` |
| `tcpdump -r <file>` | Read from file | `tcpdump -r capture.pcap` |
| `tcpdump -n` | Don't resolve names | `tcpdump -n -i eth0` |
| `tcpdump -nn` | Don't resolve names/ports | `tcpdump -nn -i eth0` |
| `tcpdump -A` | Print ASCII data | `tcpdump -A -i eth0` |
| `tcpdump -X` | Print hex and ASCII | `tcpdump -X -i eth0` |

### tcpdump Filters
| Filter | Purpose | Example |
|--------|---------|---------|
| `host <ip>` | Traffic to/from host | `tcpdump host 10.100.50.100` |
| `src <ip>` | Traffic from source | `tcpdump src 10.100.50.100` |
| `dst <ip>` | Traffic to destination | `tcpdump dst 10.100.50.100` |
| `port <port>` | Traffic on port | `tcpdump port 443` |
| `tcp port <port>` | TCP port only | `tcpdump tcp port 80` |
| `udp port <port>` | UDP port only | `tcpdump udp port 53` |
| `tcp` | TCP protocol only | `tcpdump tcp` |
| `udp` | UDP protocol only | `tcpdump udp` |
| `icmp` | ICMP (ping) only | `tcpdump icmp` |
| `proto <num>` | Protocol number | `tcpdump proto 17` |

### tcpdump Complex Filters
| Filter | Purpose | Example |
|--------|---------|---------|
| `tcp[tcpflags] & tcp-syn` | SYN packets | `tcpdump 'tcp[tcpflags] & tcp-syn'` |
| `tcp[tcpflags] == tcp-ack` | ACK packets | `tcpdump 'tcp[tcpflags] == tcp-ack'` |
| `'tcp[0:2] > 1024'` | High source ports | `tcpdump 'tcp[0:2] > 1024'` |
| `dst net 10.0.0.0/8` | Destination network | `tcpdump dst net 10.0.0.0/8` |
| `and`, `or`, `not` | Combine filters | `tcpdump port 80 or port 443` |

---

## Kubernetes Diagnostics

### Pod and Node Information
| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl get pods` | List pods | `kubectl get pods` |
| `kubectl get pods -A` | Pods all namespaces | `kubectl get pods -A` |
| `kubectl get pods -o wide` | Pods with node info | `kubectl get pods -o wide` |
| `kubectl get pods --show-labels` | Show pod labels | `kubectl get pods --show-labels` |
| `kubectl describe pod <pod>` | Pod details | `kubectl describe pod my-pod` |
| `kubectl get nodes` | List nodes | `kubectl get nodes` |
| `kubectl describe node <node>` | Node details | `kubectl describe node node-1` |
| `kubectl top pods` | Pod resource usage | `kubectl top pods` |
| `kubectl top nodes` | Node resource usage | `kubectl top nodes` |

### Logs
| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl logs <pod>` | Pod logs | `kubectl logs my-pod` |
| `kubectl logs <pod> -c <container>` | Specific container | `kubectl logs my-pod -c app` |
| `kubectl logs -f <pod>` | Follow logs (tail -f) | `kubectl logs -f my-pod` |
| `kubectl logs --tail=50 <pod>` | Last N lines | `kubectl logs --tail=50 my-pod` |
| `kubectl logs --since=1h <pod>` | Last N time | `kubectl logs --since=1h my-pod` |
| `kubectl logs --previous <pod>` | Previous container logs | `kubectl logs --previous my-pod` |
| `kubectl logs -l <label> <pod>` | Logs for labeled pods | `kubectl logs -l app=api` |

### Pod Execution
| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl exec <pod> -- <cmd>` | Run command in pod | `kubectl exec my-pod -- ls /` |
| `kubectl exec -it <pod> -- /bin/bash` | Interactive shell | `kubectl exec -it my-pod -- /bin/bash` |
| `kubectl exec <pod> -- nslookup <svc>` | Test DNS from pod | `kubectl exec my-pod -- nslookup my-service` |
| `kubectl exec <pod> -- curl http://<svc>` | Test service from pod | `kubectl exec my-pod -- curl http://my-service` |
| `kubectl cp <pod>:<path> <local>` | Copy from pod | `kubectl cp my-pod:/var/log/app.log ./` |

### Services and Endpoints
| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl get svc` | List services | `kubectl get svc` |
| `kubectl describe svc <svc>` | Service details | `kubectl describe svc my-service` |
| `kubectl get endpoints <svc>` | Service endpoints | `kubectl get endpoints my-service` |
| `kubectl get svc -o wide` | Services with cluster IP | `kubectl get svc -o wide` |
| `kubectl expose pod <pod> --port=<port>` | Create service for pod | `kubectl expose pod my-pod --port=8080` |

### Network Policies
| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl get networkpolicy` | List policies | `kubectl get networkpolicy` |
| `kubectl get networkpolicy -A` | All policies all NS | `kubectl get networkpolicy -A` |
| `kubectl describe networkpolicy <pol>` | Policy details | `kubectl describe networkpolicy deny-all` |
| `kubectl apply -f policy.yaml` | Create policy | `kubectl apply -f policy.yaml` |
| `kubectl delete networkpolicy <pol>` | Delete policy | `kubectl delete networkpolicy deny-all` |

### Events and Status
| Command | Purpose | Example |
|---------|---------|---------|
| `kubectl get events` | Cluster events | `kubectl get events` |
| `kubectl get events -A` | Events all namespaces | `kubectl get events -A` |
| `kubectl get events --sort-by='.lastTimestamp'` | Events sorted by time | `kubectl get events --sort-by='.lastTimestamp'` |
| `kubectl get events -n <ns>` | Events in namespace | `kubectl get events -n default` |
| `kubectl describe node <node>` | Node events | `kubectl describe node node-1` |

---

## AWS Network Diagnostics

### VPC Flow Logs
| Command | Purpose | Example |
|---------|---------|---------|
| `aws ec2 describe-flow-logs` | List Flow Logs | `aws ec2 describe-flow-logs` |
| `aws ec2 create-flow-logs --resource-ids <id> --resource-type NetworkInterface --traffic-type ALL --log-destination-type cloud-watch-logs --log-group-name /aws/vpc --deliver-logs-permission-role-name vpc-flow-logs` | Enable Flow Logs | Create VPC Flow Logs |
| `aws logs filter-log-events --log-group-name /aws/vpc` | Query Flow Logs in CloudWatch | `aws logs filter-log-events --log-group-name /aws/vpc --filter-pattern "REJECT"` |

### Security Groups and NACLs
| Command | Purpose | Example |
|---------|---------|---------|
| `aws ec2 describe-security-groups` | List security groups | `aws ec2 describe-security-groups` |
| `aws ec2 describe-security-groups --group-ids <id>` | Get specific SG | `aws ec2 describe-security-groups --group-ids sg-12345` |
| `aws ec2 authorize-security-group-ingress --group-id <id> --protocol tcp --port 443 --cidr 10.0.0.0/8` | Add inbound rule | Add HTTPS from 10.0.0.0/8 |
| `aws ec2 describe-network-acls` | List NACLs | `aws ec2 describe-network-acls` |

### EC2 and ENI
| Command | Purpose | Example |
|---------|---------|---------|
| `aws ec2 describe-instances` | List instances | `aws ec2 describe-instances` |
| `aws ec2 describe-network-interfaces` | List ENIs | `aws ec2 describe-network-interfaces` |
| `aws ec2 describe-route-tables` | List route tables | `aws ec2 describe-route-tables` |
| `aws ec2 describe-internet-gateways` | List IGWs | `aws ec2 describe-internet-gateways` |
| `aws ec2 describe-nat-gateways` | List NAT gateways | `aws ec2 describe-nat-gateways` |

### CloudWatch
| Command | Purpose | Example |
|---------|---------|---------|
| `aws cloudwatch describe-alarms` | List alarms | `aws cloudwatch describe-alarms` |
| `aws logs tail <log-group>` | Watch logs in real-time | `aws logs tail /aws/lambda/my-function --follow` |
| `aws logs describe-log-groups` | List log groups | `aws logs describe-log-groups` |

---

## Linux System Diagnostics

### System Information
| Command | Purpose | Example |
|---------|---------|---------|
| `uname -a` | System info | `uname -a` |
| `lsb_release -a` | OS release info | `lsb_release -a` |
| `df -h` | Disk usage | `df -h` |
| `du -sh <path>` | Directory size | `du -sh /var/log` |
| `free -h` | Memory usage | `free -h` |
| `uptime` | System uptime | `uptime` |

### Process Information
| Command | Purpose | Example |
|---------|---------|---------|
| `ps aux` | All processes | `ps aux` |
| `ps aux \| grep <name>` | Find process | `ps aux | grep nginx` |
| `top -b -n 1` | Top processes once | `top -b -n 1` |
| `top -p <pid>` | Monitor specific PID | `top -p 1234` |
| `htop` | Interactive process monitor | `htop` |

### File Operations
| Command | Purpose | Example |
|---------|---------|---------|
| `ls -lah <path>` | List files with details | `ls -lah /var/log` |
| `tail -f <file>` | Follow file (like tail -f) | `tail -f /var/log/syslog` |
| `grep <pattern> <file>` | Search in file | `grep "error" app.log` |
| `grep -i <pattern> <file>` | Case-insensitive search | `grep -i "error" app.log` |
| `grep -c <pattern> <file>` | Count matches | `grep -c "error" app.log` |
| `wc -l <file>` | Line count | `wc -l app.log` |

### Permissions and Ownership
| Command | Purpose | Example |
|---------|---------|---------|
| `chmod <mode> <file>` | Change permissions | `chmod 755 script.sh` |
| `chown <user>:<group> <file>` | Change owner | `chown root:root script.sh` |
| `sudo <command>` | Run as root | `sudo tail /var/log/syslog` |

---

## Troubleshooting Workflow

### Quick Diagnosis Checklist
```bash
# 1. Is it reachable? (Connectivity)
ping <target>
curl -v <target>

# 2. Is port open? (Firewall)
nc -zv <host> <port>
netstat -tlnp | grep <port>

# 3. Is service responding? (Application)
curl -v http://<target>/health
kubectl logs <pod>

# 4. Is DNS working? (Name resolution)
nslookup <hostname>
kubectl exec <pod> -- nslookup <service>

# 5. Are policies blocking? (Network Policy)
kubectl get networkpolicy -A
kubectl describe networkpolicy <name>

# 6. Capture traffic for deep analysis (Packet analysis)
tcpdump -i any host <ip> -w capture.pcap
```

### Common Issues Quick Fix
| Issue | Command | Expected Result |
|-------|---------|-----------------|
| DNS not resolving | `kubectl logs -n kube-system -l k8s-app=kube-dns` | No errors in CoreDNS logs |
| Service no endpoints | `kubectl get endpoints <svc>` | Shows IP:port endpoints |
| Pod can't reach service | `kubectl describe networkpolicy` | Policy allows traffic |
| High latency | `traceroute <target>` | Consistent hop latencies |
| Port unreachable | `netstat -tlnp` | Service listening on port |
| SSL error | `curl -v https://<target>` | SSL handshake successful |

### Severity Levels
```
CRITICAL: Service completely unavailable
- Affects all users
- All pods unable to communicate
- Fix immediately

HIGH: Partial service degradation  
- Affects some users/regions
- Some pods failing
- Investigate quickly

MEDIUM: Warnings or slow performance
- Isolated instances failing
- Performance degraded
- Monitor and schedule fix

LOW: Non-critical issues
- Warnings in logs
- One-time events
- Plan fix in sprint
```
