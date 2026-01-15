# 07 - Load Balancing and Proxies

## What You'll Learn

By the end of this module, you'll be able to:
- Understand load balancing concepts and algorithms
- Differentiate between Layer 4 (L4) and Layer 7 (L7) load balancing
- Configure and manage reverse proxies
- Implement forward proxies for outbound traffic
- Use HAProxy for load balancing
- Monitor load balancer health checks
- Distribute traffic across multiple backend servers
- Understand session persistence and sticky sessions
- Troubleshoot load balancing issues
- Implement SSL/TLS termination on load balancers

## Prerequisites

- Completion of modules 00-06
- Understanding of TCP/IP, DNS, and HTTP/HTTPS
- Windows 10/11 with administrative access (or Linux/Mac equivalent)
- Basic knowledge of web servers
- Command-line proficiency
- Multiple servers or local VM environment (optional but helpful)

## Key Concepts

### Load Balancing Overview

1. **Load Balancing** - Distributing requests across multiple servers
2. **Single Point of Failure** - Prevents outages if one server fails
3. **Scalability** - Add more servers to handle more traffic
4. **High Availability** - Ensures service continues if server goes down
5. **Performance** - Improves response times with parallel processing

### Load Balancing Algorithms

1. **Round Robin** - Distribute equally in circular order
2. **Least Connections** - Route to server with fewest active connections
3. **IP Hash** - Route based on client IP address (sticky)
4. **Weighted Round Robin** - Distribute based on server capacity
5. **Random** - Select server randomly
6. **Resource-Based** - Route based on server resources (CPU, memory)

### Layer 4 vs Layer 7 Load Balancing

1. **Layer 4 (L4)** - Works at transport layer (TCP/UDP)
   - Faster, less CPU overhead
   - Basic routing only
   - Good for non-HTTP protocols

2. **Layer 7 (L7)** - Works at application layer (HTTP/HTTPS)
   - Can inspect content
   - URL-based routing
   - Cookie-based routing
   - More CPU intensive

### Proxies

1. **Forward Proxy** - Client → Proxy → Server (protects client)
2. **Reverse Proxy** - Client → Proxy → Multiple Servers (protects backend)
3. **Proxy Functions** - Caching, filtering, compression, SSL termination
4. **API Gateway** - Advanced proxy for APIs (authentication, rate limiting)

### Health Checks

1. **TCP Health Check** - Verify port is open
2. **HTTP Health Check** - Verify HTTP response (status 200)
3. **Ping Health Check** - Simple ICMP echo
4. **Custom Health Check** - Application-specific endpoint
5. **Health Check Interval** - How often to check (typically 5-10 seconds)

### Session Persistence (Sticky Sessions)

1. **No Persistence** - Each request may go to different server
2. **Source IP** - Same client always goes to same server
3. **Cookie-Based** - Load balancer sets cookie to track session
4. **Session Table** - Load balancer maintains session state
5. **Backend-Based** - Application handles session distribution

## Hands-on Lab: Setting Up a Local Load Balancer

### Objective
Configure a load balancer to distribute traffic across multiple backend servers using HAProxy simulation with netsh URL rewriting on Windows, or nginx on Linux.

### Prerequisites for Lab
- Windows: PowerShell admin access
- Linux: nginx or HAProxy installed
- 2-3 local services or web servers running on different ports

### Step 1: Verify Backend Servers

**Windows PowerShell:**
```powershell
# Start simple HTTP servers on different ports (PowerShell)
# Terminal 1
$listener = New-Object System.Net.HttpListener
$listener.Prefixes.Add("http://localhost:8001/")
$listener.Start()

# In loop, accept and respond
while ($true) {
    $context = $listener.GetContext()
    $response = $context.Response
    [byte[]]$buffer = [System.Text.Encoding]::UTF8.GetBytes("Server 1 responding")
    $response.ContentLength64 = $buffer.Length
    $response.OutputStream.Write($buffer, 0, $buffer.Length)
    $response.Close()
}
```

**Expected Output:**
```
Server 1 listening on port 8001
```

**Repeat for Server 2 and Server 3 on ports 8002 and 8003**

**Linux/Mac (Python simple HTTP server):**
```bash
# Terminal 1
python3 -m http.server 8001 --directory . &

# Terminal 2
python3 -m http.server 8002 --directory . &

# Terminal 3
python3 -m http.server 8003 --directory . &
```

**Expected Output:**
```
Serving HTTP on 0.0.0.0 port 8001 (http://0.0.0.0:8001/)
```

### Step 2: Verify Backend Connectivity

```powershell
# Test each backend server
curl http://localhost:8001
curl http://localhost:8002
curl http://localhost:8003
```

**Expected Output:**
```
Server 1 responding
Server 2 responding
Server 3 responding
```

### Step 3: Configure Load Balancer (Linux/Mac with Nginx)

**Create nginx config file:**
```bash
# Create /etc/nginx/sites-available/loadbalancer
cat > /tmp/loadbalancer.conf << 'EOF'
upstream backend {
    server 127.0.0.1:8001;
    server 127.0.0.1:8002;
    server 127.0.0.1:8003;
}

server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
EOF
```

### Step 4: Start Load Balancer

**Linux:**
```bash
# Start nginx with custom config
nginx -c /tmp/loadbalancer.conf
```

**Windows (using netsh URL rewriting):**
```powershell
# Add URL rewrite rule (requires IIS)
# Or use a simple PowerShell proxy script

# Create a simple load balancer in PowerShell
$listener = New-Object System.Net.HttpListener
$listener.Prefixes.Add("http://localhost:80/")
$listener.Start()
Write-Host "Load Balancer listening on port 80"

$servers = @("http://localhost:8001", "http://localhost:8002", "http://localhost:8003")
$counter = 0

while ($true) {
    $context = $listener.GetContext()
    
    # Round robin selection
    $server = $servers[$counter % 3]
    $counter++
    
    try {
        $response = Invoke-WebRequest "$server/" -UseBasicParsing
        $context.Response.StatusCode = 200
        [byte[]]$buffer = [System.Text.Encoding]::UTF8.GetBytes($response.Content)
        $context.Response.ContentLength64 = $buffer.Length
        $context.Response.OutputStream.Write($buffer, 0, $buffer.Length)
    } catch {
        $context.Response.StatusCode = 503
    }
    $context.Response.Close()
}
```

### Step 5: Test Load Balancer

```powershell
# Make multiple requests to load balancer
for ($i = 1; $i -le 9; $i++) {
    Write-Host "Request $i:"
    curl http://localhost:80/
    Start-Sleep -Seconds 1
}
```

**Expected Output:**
```
Request 1:
Server 1 responding

Request 2:
Server 2 responding

Request 3:
Server 3 responding

Request 4:
Server 1 responding
...
```

**Explanation:** Requests alternate between servers in round-robin fashion.

### Step 6: Simulate Server Failure

```powershell
# Kill one backend server (e.g., Server 2)
# Then continue making requests - load balancer should skip it

for ($i = 1; $i -le 6; $i++) {
    Write-Host "Request $i (after Server 2 killed):"
    curl http://localhost:80/
    Start-Sleep -Seconds 1
}
```

**Expected Output:**
```
Request 1 (after Server 2 killed):
Server 1 responding

Request 2 (after Server 2 killed):
Server 3 responding

Request 3 (after Server 2 killed):
Server 1 responding
...
```

**Explanation:** Load balancer skips dead server, only uses 1 and 3.

## Validation

After completing this lab, validate your understanding by:

1. **Round Robin Distribution** - Verify requests alternate between servers
2. **Failure Handling** - Confirm load balancer skips dead servers
3. **Health Checks** - Monitor health check responses
4. **Backend Responses** - Identify which backend served each request
5. **Request Counting** - Each server should handle roughly equal traffic

## Cleanup

```powershell
# Stop load balancer and backend servers
Get-Process | Where-Object {$_.Name -like "*python*" -or $_.Name -like "*http*"} | Stop-Process -Force

# Stop nginx (if running)
# nginx -s stop

# Remove temporary files
Remove-Item -Path /tmp/loadbalancer.conf -ErrorAction SilentlyContinue
```

## Common Mistakes

1. **All Traffic to One Server** - Misconfigured algorithm or sticky sessions
2. **No Failover** - Not checking if backend is healthy
3. **Session Loss** - Not using sticky sessions when needed
4. **No Health Checks** - Routing to dead servers
5. **Unequal Load Distribution** - Servers not evenly weighted
6. **SSL/TLS Overhead** - Not terminating SSL at load balancer
7. **No Monitoring** - Can't detect bottlenecks or failures
8. **Load Balancer as SPOF** - Making load balancer the single point of failure

## Troubleshooting

### "All Traffic Going to First Server" Error
**Cause:** Load balancing algorithm misconfigured or algorithm not applied
**Solution:**
- Verify algorithm is set to round-robin or least connections
- Check upstream servers configuration
- Ensure all backends are healthy (run health checks)
- Restart load balancer after config change

### "Backend Server Returns 503" Error
**Cause:** Backend server is down or not responding
**Solution:**
- Check if backend server process is running: `netstat -an | findstr :8001`
- Verify firewall allows connection to backend port
- Test direct connection: `curl http://localhost:8001`
- Check backend server logs
- Load balancer health checks should detect and remove from pool

### "Sessions Lost After Request" Error
**Cause:** Requests routed to different server, no session persistence
**Solution:**
- Enable sticky sessions (cookie-based or IP-based)
- Configure session persistence in load balancer config
- Share session state between backends (cache, database)
- For nginx: Use `ip_hash` or `sticky` directive

### "Load Balancer Not Responding" Error
**Cause:** Load balancer process stopped or port binding failed
**Solution:**
- Check if load balancer is running: `netstat -an | findstr :80`
- Verify port 80 is not in use by another process
- Check firewall allows port 80
- Verify load balancer configuration syntax
- Review load balancer logs

### "High CPU on Load Balancer" Error
**Cause:** Layer 7 load balancing with heavy inspection
**Solution:**
- Use L4 load balancing if L7 features not needed
- Reduce connection timeout values
- Disable unnecessary header inspection
- Use connection pooling
- Scale load balancer horizontally (multiple load balancers)

### "Uneven Load Distribution" Error
**Cause:** Server capacity differences or misconfigured weights
**Solution:**
- Check all backends are responding to health checks
- Use weighted round robin based on server capacity
- Configure weights: `server 127.0.0.1:8001 weight=2`
- Monitor actual request distribution
- Check for sticky sessions causing imbalance

## Next Steps

1. **Module 08** - Learn about Firewalls and Security Groups (load balancer rules)
2. **Module 09** - Study Cloud Networking and AWS VPC (AWS ELB, NLB, ALB)
3. **Module 10** - Explore Connectivity options (load balancing across data centers)
4. **Practice:** Set up HAProxy or nginx with health checks and SSL termination
5. **Real-World:** Configure load balancing for your application stack
