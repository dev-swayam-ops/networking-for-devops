# Cheatsheet: Load Balancing and Proxies

Quick reference for load balancing concepts, commands, and configurations.

## Load Balancing Algorithms

### Algorithm Comparison

| Algorithm | Selection Method | Distribution | Session Aware | Use Case | Complexity |
|-----------|-----------------|--------------|---------------|----------|------------|
| Round Robin | Each server in order | Equal (if same capacity) | No | Standard HTTP | ⭐ Low |
| Least Connections | Fewest active connections | Based on load | Partial | WebSocket, streaming | ⭐⭐ Medium |
| IP Hash | Hash of client IP | Equal (for stable IPs) | Yes (implicit) | Caching, sticky | ⭐ Low |
| Weighted Round Robin | Weight + order | Proportional | No | Different capacity servers | ⭐⭐ Medium |
| Random | Random selection | Equal (statistically) | No | Testing, simple systems | ⭐ Low |
| Least Response Time | Lowest response time + connections | Dynamic | Partial | Performance critical | ⭐⭐⭐ High |

## nginx Configuration

### Basic Load Balancer

| Configuration | Directive | Example |
|---------------|-----------|---------|
| Define backend servers | `upstream` | `upstream backend { server 192.168.1.1:8080; }` |
| Set algorithm | (default RR) | Round robin is default |
| Least connections | `least_conn;` | `upstream backend { least_conn; server ...; }` |
| IP Hash (sticky) | `ip_hash;` | `upstream backend { ip_hash; server ...; }` |
| Weighted | `weight=N;` | `server backend1 weight=2;` |
| Health check | `max_fails` / `fail_timeout` | `server backend1 max_fails=3 fail_timeout=30s;` |
| SSL termination | `proxy_pass https://` | `proxy_pass https://backend;` |
| Session cookie | `sticky` | `sticky cookie srv expires=1h;` |

### Nginx Config Example

```nginx
upstream backend {
    # Round robin with weights
    server 192.168.1.1:8080 weight=2;
    server 192.168.1.2:8080 weight=1;
    server 192.168.1.3:8080 weight=1;
}

server {
    listen 80;
    server_name myapp.com;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    
    location /health {
        access_log off;
        return 200 "healthy\n";
    }
}
```

## HAProxy Configuration

### Basic Directives

| Section | Directive | Purpose | Example |
|---------|-----------|---------|---------|
| global | `maxconn` | Max connections | `maxconn 4096` |
| defaults | `timeout connect` | Connection timeout | `timeout connect 5000` |
| defaults | `timeout client` | Client timeout | `timeout client 50000` |
| defaults | `timeout server` | Server timeout | `timeout server 50000` |
| frontend | `bind` | Listen address | `bind *:80` |
| frontend | `default_backend` | Default backend | `default_backend web_backend` |
| backend | `balance` | Algorithm | `balance roundrobin` |
| backend | `server` | Backend server | `server backend1 127.0.0.1:8080` |
| backend | `option httpchk` | Health check | `option httpchk GET /health` |
| backend | `cookie` | Session tracking | `cookie SERVERID insert` |

### HAProxy Algorithm Options

| Algorithm | Directive |
|-----------|-----------|
| Round Robin | `balance roundrobin` |
| Least Connections | `balance leastconn` |
| Source IP Hash | `balance source` |
| URI Hash | `balance uri` |
| URL Path | `balance url_param` |

### Health Check Options

```haproxy
# TCP Check
server backend1 127.0.0.1:8080 check

# HTTP Check
server backend1 127.0.0.1:8080 check inter 5s fall 3 rise 2

# With specific path
option httpchk GET /health HTTP/1.1\r\nHost:\ example.com
server backend1 127.0.0.1:8080 check
```

## Cloud Load Balancer Commands

### AWS Elastic Load Balancer (ELB)

| Task | AWS CLI Command |
|------|----------------|
| Create Classic LB | `aws elb create-load-balancer --load-balancer-name my-lb --listeners InstancePort=80,LoadBalancerPort=80,Protocol=HTTP` |
| Register instances | `aws elb register-instances-with-load-balancer --load-balancer-name my-lb --instances i-123456` |
| Set health check | `aws elb configure-health-check --load-balancer-name my-lb --health-check Target=HTTP:80/health,Interval=30,HealthyThreshold=2,UnhealthyThreshold=2,Timeout=5` |
| Describe LB | `aws elb describe-load-balancers --load-balancer-name my-lb` |
| Delete LB | `aws elb delete-load-balancer --load-balancer-name my-lb` |

### AWS Application Load Balancer (ALB)

| Task | AWS CLI Command |
|------|----------------|
| Create ALB | `aws elbv2 create-load-balancer --name my-alb --subnets subnet-xxx` |
| Create target group | `aws elbv2 create-target-group --name my-targets --protocol HTTP --port 80` |
| Register targets | `aws elbv2 register-targets --target-group-arn arn:xxx --targets Id=i-123456` |
| Create listener | `aws elbv2 create-listener --load-balancer-arn arn:xxx --protocol HTTP --port 80 --default-actions TargetGroupArn=arn:xxx` |
| Create rule (path-based) | `aws elbv2 create-rule --listener-arn arn:xxx --conditions Field=path-pattern,Values=/api/* --priority 1 --actions TargetGroupArn=arn:xxx` |

## Session Persistence Methods

### Comparison

| Method | Setup | Persistence | Use Case | Complexity |
|--------|-------|-----------|----------|------------|
| **Source IP Hash** | Algorithm | Client IP always same server | Caching, simple sessions | Low |
| **Cookie-Based** | Load balancer cookie | Set cookie with server ID | Web applications | Medium |
| **Session Table** | Load balancer state | LB stores session mappings | Stateful apps | High |
| **Database** | Application level | Shared database for sessions | Microservices | High |
| **Cache (Redis)** | Application level | Shared cache layer | Modern apps | High |

### Sticky Session Examples

**nginx:**
```nginx
upstream backend {
    server 192.168.1.1:8080;
    server 192.168.1.2:8080;
    sticky cookie srv expires=1h path=/;
}
```

**HAProxy:**
```haproxy
backend web_backend
    cookie SERVERID insert indirect
    server backend1 127.0.0.1:8080 cookie server1
    server backend2 127.0.0.1:8081 cookie server2
```

## Health Check Configuration

### TCP Health Check (L4)

| Tool | Command |
|------|---------|
| netstat | `netstat -an \| findstr :8080` |
| telnet | `telnet 192.168.1.1 8080` |
| nc (netcat) | `nc -zv 192.168.1.1 8080` |

### HTTP Health Check (L7)

| Tool | Command |
|------|---------|
| curl | `curl -f http://server:8080/health` |
| curl + status | `curl -o /dev/null -s -w "%{http_code}" http://server:8080/health` |
| curl + timeout | `curl --max-time 5 http://server:8080/health` |
| wget | `wget --spider http://server:8080/health` |

### HAProxy Health Check

```haproxy
# TCP
server backend1 127.0.0.1:8080 check

# HTTP GET
option httpchk GET /health
server backend1 127.0.0.1:8080 check

# HTTP POST
option httpchk POST /health
server backend1 127.0.0.1:8080 check

# Timing options
# check inter 5s = check every 5 seconds
# fall 3 = mark DOWN after 3 failures
# rise 2 = mark UP after 2 successes
server backend1 127.0.0.1:8080 check inter 5s fall 3 rise 2
```

## Load Balancer Testing Commands

### Distributed Load Testing

| Task | Command |
|------|---------|
| Single request | `curl http://load-balancer/` |
| Show headers | `curl -i http://load-balancer/` |
| Verbose output | `curl -v http://load-balancer/` |
| Request N times | `for i in {1..10}; do curl http://load-balancer/; done` |
| Parallel requests | `parallel 'curl http://load-balancer/' ::: {1..10}` |
| With session cookie | `curl -b "SESSIONID=abc" http://load-balancer/` |
| Request rate | `ab -n 100 -c 10 http://load-balancer/` |
| WebSocket test | `wscat -c ws://load-balancer/ws` |

### Apache Bench Load Test

```bash
# 100 total requests, 10 concurrent
ab -n 100 -c 10 http://load-balancer/

# With persistent connections
ab -n 100 -c 10 -k http://load-balancer/

# Output metrics: Min, Mean, Median, Max response times
```

### Identify Backend Server

```bash
# Method 1: Check response headers
curl -i http://load-balancer/ | grep X-Real-IP

# Method 2: Add tracking header in backend response
curl -v http://load-balancer/ 2>&1 | grep "Server:"

# Method 3: Check body if backend includes identifier
curl http://load-balancer/ | grep -o "Server [0-9]*"
```

## Common Issues and Solutions

### Issue Reference

| Issue | Symptom | Solution |
|-------|---------|----------|
| **All traffic to one server** | One backend has 100% traffic | Check algorithm configured, verify health checks |
| **Session loss** | User logged out after request | Enable sticky sessions, share session data |
| **High latency** | Requests slow | Reduce timeout values, add more backends, check backend capacity |
| **Backend unavailable** | 503 Service Unavailable | Start backends, check firewall, verify connectivity |
| **Imbalanced load** | Uneven request distribution | Check weights, verify no sticky sessions forcing imbalance |
| **Health check false positive** | Good server marked DOWN | Increase fail threshold, improve health check endpoint |
| **Load balancer single point of failure** | LB down = all down | Use active-active or active-passive load balancer pair |
| **SSL errors on HTTPS** | Certificate mismatch | Configure SNI (Server Name Indication) |

## Load Balancer Metrics to Monitor

| Metric | What to Track | Alert Threshold |
|--------|---------------|-----------------|
| **Request rate** | Requests per second | > 90% capacity |
| **Response time** | Average, p50, p95, p99 | p95 > 1s |
| **Error rate** | 4xx and 5xx responses | > 5% |
| **Active connections** | Current active connections | > 80% limit |
| **Backend health** | Number of healthy backends | < N-1 (need redundancy) |
| **CPU usage** | Load balancer CPU | > 80% |
| **Memory usage** | RAM consumption | > 80% |
| **Network throughput** | Mbps in/out | > 90% link capacity |
| **Dropped connections** | Overflow/timeout drops | > 0 |

## SSL/TLS on Load Balancers

### SSL Termination

```nginx
# Terminate SSL at load balancer
server {
    listen 443 ssl;
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    
    location / {
        # Forward to backend over HTTP (internal only)
        proxy_pass http://backend;
        # Add header showing original protocol
        proxy_set_header X-Forwarded-Proto https;
    }
}
```

### SSL Pass-Through

```nginx
# Pass encrypted traffic through to backend
stream {
    upstream backend {
        server backend1.example.com:443;
        server backend2.example.com:443;
    }
    
    server {
        listen 443;
        proxy_pass backend;
    }
}
```

### SNI (Server Name Indication)

```nginx
# Multiple SSL certificates based on hostname
server {
    listen 443 ssl;
    server_name app1.example.com;
    ssl_certificate /path/to/app1.crt;
    ssl_certificate_key /path/to/app1.key;
}

server {
    listen 443 ssl;
    server_name app2.example.com;
    ssl_certificate /path/to/app2.crt;
    ssl_certificate_key /path/to/app2.key;
}
```

