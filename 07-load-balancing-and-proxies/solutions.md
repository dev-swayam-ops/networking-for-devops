# Solutions: Load Balancing and Proxies

Complete solutions with explanations for all exercises.

## Easy Exercise Solutions

### Exercise 1: Understand Load Balancing Algorithms - Solution

**Algorithm Definitions and Use Cases:**

| Algorithm | Definition | Use Case | Pros | Cons |
|-----------|-----------|----------|------|------|
| **Round Robin** | Each server gets requests in order, cycling through all backends | Standard HTTP web traffic | Simple, fair distribution | Doesn't consider server load |
| **Least Connections** | Route to server with fewest active connections | Long-lived connections (WebSockets, VoIP) | Adapts to actual load | Higher CPU overhead |
| **IP Hash** | Hash client IP to determine server (same IP always same server) | Session persistence, stateful apps | No sticky cookie needed | Uneven distribution if IPs skewed |
| **Weighted Round Robin** | Each server gets requests based on assigned weight | Different capacity servers | Adapts to server power | Manual configuration needed |
| **Random** | Select random server for each request | Load testing, simple systems | Simple implementation | Uneven distribution likely |
| **Resource-Based** | Route based on server CPU, memory, connections | Complex applications | Optimal resource usage | Complex monitoring needed |

**Example Selection Scenarios:**
```
API Gateway with balanced servers → Round Robin
WebSocket connections → Least Connections
Microservices with small/large instances → Weighted Round Robin
Stateless API → IP Hash (for cache locality)
```

**Key Learning:** Algorithm selection affects distribution quality and complexity.

---

### Exercise 2: Test Multiple Backend Servers Locally - Solution

**Linux Setup:**
```bash
# Terminal 1
python3 -m http.server 8001 &

# Terminal 2
python3 -m http.server 8002 &

# Terminal 3
python3 -m http.server 8003 &
```

**Verification:**
```bash
# Test each server
curl -s http://localhost:8001 | head -1
curl -s http://localhost:8002 | head -1
curl -s http://localhost:8003 | head -1

# Verify all listening
netstat -an | findstr ":800[1-3]"
```

**Expected Output:**
```
<!DOCTYPE HTML PUBLIC "3.2//EN">
<!DOCTYPE HTML PUBLIC "3.2//EN">
<!DOCTYPE HTML PUBLIC "3.2//EN">

tcp    0      0 127.0.0.1:8001         0.0.0.0:*               LISTEN
tcp    0      0 127.0.0.1:8002         0.0.0.0:*               LISTEN
tcp    0      0 127.0.0.1:8003         0.0.0.0:*               LISTEN
```

**Explanation:**
- Python's http.server provides simple HTTP server
- Each instance listens on different port independently
- Default response is directory listing (HTML)
- netstat shows all three listening on correct ports

---

### Exercise 3: Identify Layer 4 vs Layer 7 Load Balancing - Solution

**Comparison Table:**

| Attribute | Layer 4 | Layer 7 |
|-----------|---------|---------|
| **OSI Layer** | Transport (TCP/UDP) | Application (HTTP/HTTPS) |
| **Data Inspection** | Header only (port, IP) | Full payload inspection |
| **Speed** | Very fast (microseconds) | Slower (can inspect 100s of bytes) |
| **CPU Usage** | Low | High |
| **Routing Decision** | IP:Port only | URL, cookie, hostname, headers |
| **Protocol Support** | Any (TCP, UDP, custom) | HTTP/HTTPS primarily |
| **Complexity** | Simple | Complex |
| **Use Cases** | Gaming, DNS, IoT, VoIP | Web APIs, microservices, static sites |
| **Session Awareness** | Limited | Full (can read cookies) |
| **Connection Pooling** | Possible | Yes |

**Scenario Decisions:**

1. **Web API with URL-based routing** → L7 (need to read HTTP path)
2. **Gaming server with 1000s of players** → L4 (speed critical)
3. **IoT devices sending binary data** → L4 (not HTTP)
4. **Microservice architecture** → L7 (route by service path like /api/users)

**Example L4 Config:**
```
Load Balancer
├── Port 80 (TCP)
├── Route to 192.168.1.10:8080
├── Route to 192.168.1.11:8080
└── Route to 192.168.1.12:8080
(Doesn't know or care about HTTP)
```

**Example L7 Config:**
```
Load Balancer (HTTP-aware)
├── /api/users → Backend 1
├── /api/products → Backend 2
├── /static → Cache Server
└── / → All others to Backend 3
(Reads and interprets HTTP requests)
```

---

### Exercise 4: Monitor Backend Health via Netstat - Solution

**Commands:**
```powershell
# Monitor single port
netstat -an | findstr :8001

# Monitor all backend ports
netstat -an | findstr ":800[1-3]"

# Show connections with process names
netstat -ab | findstr ":800[1-3]"

# Count connections to each port
(netstat -an | findstr ":8001").Count
(netstat -an | findstr ":8002").Count
(netstat -an | findstr ":8003").Count
```

**Expected Output Before Requests:**
```
tcp    0      0 127.0.0.1:8001         0.0.0.0:*               LISTEN
tcp    0      0 127.0.0.1:8002         0.0.0.0:*               LISTEN
tcp    0      0 127.0.0.1:8003         0.0.0.0:*               LISTEN
```

**Expected Output During Requests:**
```
tcp    0      0 127.0.0.1:8001         0.0.0.0:*               LISTEN
tcp    0      0 127.0.0.1:54321        127.0.0.1:8001         ESTABLISHED
tcp    0      0 127.0.0.1:8002         0.0.0.0:*               LISTEN
tcp    0      0 127.0.0.1:54322        127.0.0.1:8002         ESTABLISHED
```

**Connection States Explained:**
| State | Meaning |
|-------|---------|
| LISTEN | Waiting for incoming connections |
| ESTABLISHED | Active connection |
| SYN_SENT | Initiating connection |
| CLOSE_WAIT | Remote closed, waiting to close |
| TIME_WAIT | Waiting before final close |

**Monitoring Pattern:**
```
1. Request arrives → Connection becomes ESTABLISHED
2. Request processed → Data sent
3. Connection closes → Moves to CLOSE_WAIT then closes
4. Eventually → Back to LISTEN only
```

---

### Exercise 5: Test Simple Reverse Proxy - Solution

**Python Reverse Proxy Script:**
```python
#!/usr/bin/env python3
import http.server
import socketserver
import urllib.request
import sys

class ReverseProxyHandler(http.server.SimpleHTTPRequestHandler):
    BACKEND_SERVERS = [
        'http://localhost:8001',
        'http://localhost:8002',
        'http://localhost:8003'
    ]
    
    def do_GET(self):
        # Route to first backend (simple routing)
        backend = self.BACKEND_SERVERS[0]
        try:
            response = urllib.request.urlopen(backend + self.path)
            self.send_response(200)
            self.send_header('Content-type', 'text/plain')
            self.end_headers()
            self.wfile.write(response.read())
        except Exception as e:
            self.send_response(503)
            self.send_header('Content-type', 'text/plain')
            self.end_headers()
            self.wfile.write(b'Backend unavailable')

if __name__ == '__main__':
    PORT = 80
    with socketserver.TCPServer(("", PORT), ReverseProxyHandler) as httpd:
        print(f"Reverse proxy listening on port {PORT}")
        httpd.serve_forever()
```

**Nginx Config:**
```nginx
upstream backends {
    server 127.0.0.1:8001;
    server 127.0.0.1:8002;
    server 127.0.0.1:8003;
}

server {
    listen 80;
    server_name localhost;
    
    location / {
        proxy_pass http://backends;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**Testing:**
```bash
# Start backends
python3 -m http.server 8001 &
python3 -m http.server 8002 &
python3 -m http.server 8003 &

# Start proxy
nginx -c /path/to/config.conf
# or: python3 reverse_proxy.py

# Test through proxy
curl http://localhost/
```

**Request Path Through Proxy:**
```
Client
   ↓ (HTTP request to :80)
Reverse Proxy (:80)
   ↓ (selects backend, forwards)
Backend Server (:8001/:8002/:8003)
   ↓ (responds)
Reverse Proxy
   ↓ (forwards response)
Client
```

---

## Medium Exercise Solutions

### Exercise 6: Implement Round Robin Load Balancing - Solution

**PowerShell Round Robin Load Balancer:**
```powershell
# Start load balancer on port 80
$listener = New-Object System.Net.HttpListener
$listener.Prefixes.Add("http://localhost:80/")

try {
    $listener.Start()
    Write-Host "Load Balancer running on port 80 (Round Robin)"
} catch {
    Write-Error "Cannot listen on port 80 (needs admin or port busy)"
    exit
}

$backends = @("http://localhost:8001", "http://localhost:8002", "http://localhost:8003")
$counter = 0
$requestCount = @{8001 = 0; 8002 = 0; 8003 = 0}

while ($true) {
    try {
        $context = $listener.GetContext()
        
        # Round robin selection
        $backend = $backends[$counter % 3]
        $port = $backend.Split(':')[-1]
        $counter++
        $requestCount[$port]++
        
        Write-Host "Request #$counter → $backend (Total: $($requestCount[$port]) to this server)"
        
        # Forward request
        try {
            $response = Invoke-WebRequest $backend -UseBasicParsing
            $context.Response.StatusCode = 200
            [byte[]]$buffer = [System.Text.Encoding]::UTF8.GetBytes("Response from $backend")
            $context.Response.ContentLength64 = $buffer.Length
            $context.Response.OutputStream.Write($buffer, 0, $buffer.Length)
        } catch {
            $context.Response.StatusCode = 503
            [byte[]]$buffer = [System.Text.Encoding]::UTF8.GetBytes("Service unavailable")
            $context.Response.ContentLength64 = $buffer.Length
            $context.Response.OutputStream.Write($buffer, 0, $buffer.Length)
        }
        
        $context.Response.Close()
    } catch {
        Write-Error "Error: $_"
    }
}
```

**Test Script:**
```powershell
# Make 9 requests and verify round-robin
for ($i = 1; $i -le 9; $i++) {
    Write-Host "Request $i:"
    curl -s http://localhost/
    Start-Sleep -Milliseconds 100
}
```

**Expected Output:**
```
Request 1:
Response from http://localhost:8001

Request 2:
Response from http://localhost:8002

Request 3:
Response from http://localhost:8003

Request 4:
Response from http://localhost:8001

Request 5:
Response from http://localhost:8002

Request 6:
Response from http://localhost:8003

Request 7:
Response from http://localhost:8001

Request 8:
Response from http://localhost:8002

Request 9:
Response from http://localhost:8003
```

**Distribution Analysis:**
- Server 8001: 3 requests (33%)
- Server 8002: 3 requests (33%)
- Server 8003: 3 requests (33%)
- ✓ Perfect round-robin distribution

---

### Exercise 7: Add Health Checks to Load Balancer - Solution

**Enhanced Load Balancer with Health Checks:**
```powershell
$listener = New-Object System.Net.HttpListener
$listener.Prefixes.Add("http://localhost:80/")
$listener.Start()

$backends = @{
    8001 = @{url = "http://localhost:8001"; healthy = $true; failCount = 0}
    8002 = @{url = "http://localhost:8002"; healthy = $true; failCount = 0}
    8003 = @{url = "http://localhost:8003"; healthy = $true; failCount = 0}
}

$counter = 0

# Health check job
$healthCheckJob = {
    param($backends)
    while ($true) {
        Start-Sleep -Seconds 5
        foreach ($port in 8001, 8002, 8003) {
            try {
                $response = Invoke-WebRequest $backends[$port].url -UseBasicParsing -TimeoutSec 2
                if ($response.StatusCode -eq 200) {
                    $backends[$port].healthy = $true
                    $backends[$port].failCount = 0
                    Write-Host "✓ Server $port is HEALTHY"
                } else {
                    $backends[$port].failCount++
                }
            } catch {
                $backends[$port].failCount++
                if ($backends[$port].failCount -ge 2) {
                    $backends[$port].healthy = $false
                    Write-Host "✗ Server $port is DOWN (fails: $($backends[$port].failCount))"
                }
            }
        }
    }
}

# Start health check background job
Start-Job -ScriptBlock $healthCheckJob -ArgumentList $backends | Out-Null

# Main request handling
while ($true) {
    try {
        $context = $listener.GetContext()
        
        # Find next healthy backend
        $attempts = 0
        $selectedBackend = $null
        while ($attempts -lt 3) {
            $backend = $backends[[array]$backends.Keys[$counter % 3]]
            $counter++
            $attempts++
            
            if ($backend.healthy) {
                $selectedBackend = $backend
                break
            }
        }
        
        if ($selectedBackend) {
            Write-Host "→ Routing to healthy backend: $($selectedBackend.url)"
            $response = Invoke-WebRequest $selectedBackend.url -UseBasicParsing
            $context.Response.StatusCode = 200
            [byte[]]$buffer = [System.Text.Encoding]::UTF8.GetBytes("Response from $($selectedBackend.url)")
            $context.Response.ContentLength64 = $buffer.Length
            $context.Response.OutputStream.Write($buffer, 0, $buffer.Length)
        } else {
            Write-Host "✗ No healthy backends available"
            $context.Response.StatusCode = 503
            [byte[]]$buffer = [System.Text.Encoding]::UTF8.GetBytes("No backends available")
            $context.Response.ContentLength64 = $buffer.Length
            $context.Response.OutputStream.Write($buffer, 0, $buffer.Length)
        }
        
        $context.Response.Close()
    } catch {
        Write-Error $_
    }
}
```

**Test Scenario:**
```powershell
# Start 3 backends
python3 -m http.server 8001 &
python3 -m http.server 8002 &
python3 -m http.server 8003 &

# Start load balancer in separate terminal
# PowerShell script above

# Make requests (health checks run every 5 seconds)
for ($i = 1; $i -le 6; $i++) {
    curl http://localhost/ ; Start-Sleep -Seconds 1
}

# Stop server 2 (kill python process on port 8002)
# Continue making requests
for ($i = 1; $i -le 6; $i++) {
    curl http://localhost/ ; Start-Sleep -Seconds 1
}

# Restart server 2
# Health check will detect and resume using it
```

**Expected Behavior:**
- 5-6 seconds: Health checks validate all backends are UP
- Request arrives: Load balancer routes to healthy servers
- Server 2 stops: After 2 health check failures (10 seconds), marked DOWN
- Requests skip Server 2: Only use servers 1 and 3
- Server 2 restarts: Health check detects recovery, resumes routing

---

### Exercise 8: Configure Sticky Sessions (Session Persistence) - Solution

**Sticky Session Load Balancer:**
```powershell
$listener = New-Object System.Net.HttpListener
$listener.Prefixes.Add("http://localhost:80/")
$listener.Start()

$backends = @("http://localhost:8001", "http://localhost:8002", "http://localhost:8003")
$sessionMap = @{}  # Maps session ID to backend

$counter = 0

while ($true) {
    try {
        $context = $listener.GetContext()
        $request = $context.Request
        
        # Check for session cookie
        $sessionId = $null
        $selectedBackend = $null
        
        if ($request.Cookies["LB-Session"]) {
            $sessionId = $request.Cookies["LB-Session"].Value
            if ($sessionMap.ContainsKey($sessionId)) {
                $selectedBackend = $sessionMap[$sessionId]
                Write-Host "↻ Session $sessionId → $selectedBackend (existing)"
            }
        }
        
        # If no session, create new one
        if (-not $selectedBackend) {
            $selectedBackend = $backends[$counter % 3]
            $counter++
            $sessionId = [guid]::NewGuid().ToString().Substring(0, 8)
            $sessionMap[$sessionId] = $selectedBackend
            Write-Host "✓ New session $sessionId → $selectedBackend"
        }
        
        # Forward request
        try {
            $response = Invoke-WebRequest $selectedBackend -UseBasicParsing
            $context.Response.StatusCode = 200
            
            # Set session cookie
            $cookie = New-Object System.Net.Cookie
            $cookie.Name = "LB-Session"
            $cookie.Value = $sessionId
            $cookie.Path = "/"
            $context.Response.Cookies.Add($cookie)
            
            [byte[]]$buffer = [System.Text.Encoding]::UTF8.GetBytes("Session: $sessionId → $selectedBackend")
            $context.Response.ContentLength64 = $buffer.Length
            $context.Response.OutputStream.Write($buffer, 0, $buffer.Length)
        } catch {
            $context.Response.StatusCode = 503
        }
        
        $context.Response.Close()
    } catch {
        Write-Error $_
    }
}
```

**Test Script:**
```powershell
# Simulate Client 1 (makes 3 requests)
Write-Host "=== Client 1 ==="
for ($i = 1; $i -le 3; $i++) {
    Write-Host "Request $i:"
    curl -b "LB-Session=client1" http://localhost/
    Start-Sleep -Milliseconds 500
}

# Simulate Client 2 (makes 3 requests)
Write-Host "=== Client 2 ==="
for ($i = 1; $i -le 3; $i++) {
    Write-Host "Request $i:"
    curl -b "LB-Session=client2" http://localhost/
    Start-Sleep -Milliseconds 500
}
```

**Expected Output:**
```
=== Client 1 ===
Request 1:
Session: client1 → http://localhost:8001

Request 2:
Session: client1 → http://localhost:8001

Request 3:
Session: client1 → http://localhost:8001

=== Client 2 ===
Request 1:
Session: client2 → http://localhost:8002

Request 2:
Session: client2 → http://localhost:8002

Request 3:
Session: client2 → http://localhost:8002
```

**Key Points:**
- Client 1 always gets Server 8001 (sticky)
- Client 2 always gets Server 8002 (sticky)
- Session cookie maintains affinity
- Different clients can use different backends

---

### Exercise 9: Implement Weighted Round Robin - Solution

**Weighted Round Robin Load Balancer:**
```powershell
$listener = New-Object System.Net.HttpListener
$listener.Prefixes.Add("http://localhost:80/")
$listener.Start()

# Define backends with weights
$backends = @(
    @{url = "http://localhost:8001"; weight = 2}
    @{url = "http://localhost:8002"; weight = 1}
    @{url = "http://localhost:8003"; weight = 1}
)

# Build weighted list [server1, server1, server2, server3]
$weightedList = @()
foreach ($backend in $backends) {
    for ($i = 0; $i -lt $backend.weight; $i++) {
        $weightedList += $backend
    }
}

Write-Host "Weighted List: $($weightedList | % {$_.url})"
Write-Host "Total requests per cycle: $($weightedList.Count)"

$counter = 0
$stats = @{8001 = 0; 8002 = 0; 8003 = 0}

while ($true) {
    try {
        $context = $listener.GetContext()
        
        # Select from weighted list
        $selectedBackend = $weightedList[$counter % $weightedList.Count]
        $counter++
        $port = $selectedBackend.url.Split(':')[-1]
        $stats[$port]++
        
        Write-Host "Request #$counter → $($selectedBackend.url) (Total: $($stats[$port]))"
        
        # Forward request
        try {
            $response = Invoke-WebRequest $selectedBackend.url -UseBasicParsing
            $context.Response.StatusCode = 200
            [byte[]]$buffer = [System.Text.Encoding]::UTF8.GetBytes("Response from $($selectedBackend.url)")
            $context.Response.ContentLength64 = $buffer.Length
            $context.Response.OutputStream.Write($buffer, 0, $buffer.Length)
        } catch {
            $context.Response.StatusCode = 503
        }
        
        $context.Response.Close()
    } catch {
        Write-Error $_
    }
}
```

**Test with 8 Requests:**
```powershell
for ($i = 1; $i -le 8; $i++) {
    Write-Host "Request $i:"
    curl -s http://localhost/
    Start-Sleep -Milliseconds 100
}
```

**Expected Output:**
```
Request 1: Response from http://localhost:8001
Request 2: Response from http://localhost:8002
Request 3: Response from http://localhost:8003
Request 4: Response from http://localhost:8001

Request 5: Response from http://localhost:8001
Request 6: Response from http://localhost:8002
Request 7: Response from http://localhost:8003
Request 8: Response from http://localhost:8001
```

**Distribution Analysis:**
- Server 8001: 4 requests (50%) ✓
- Server 8002: 2 requests (25%) ✓
- Server 8003: 2 requests (25%) ✓
- Matches configured weights: 2:1:1

---

### Exercise 10: Monitor and Analyze Load Balancer Metrics - Solution

**Load Balancer with Comprehensive Metrics:**
```powershell
$listener = New-Object System.Net.HttpListener
$listener.Prefixes.Add("http://localhost:80/")
$listener.Start()

$backends = @(
    @{port = 8001; url = "http://localhost:8001"; requests = 0; errors = 0; totalTime = 0; lastHealthy = $true}
    @{port = 8002; url = "http://localhost:8002"; requests = 0; errors = 0; totalTime = 0; lastHealthy = $true}
    @{port = 8003; url = "http://localhost:8003"; requests = 0; errors = 0; totalTime = 0; lastHealthy = $true}
)

$counter = 0
$totalRequests = 0
$startTime = Get-Date

while ($totalRequests -lt 30) {
    try {
        $context = $listener.GetContext()
        $requestTime = Measure-Command {
            
            # Select backend (round robin)
            $backend = $backends[$counter % 3]
            $counter++
            $totalRequests++
            
            # Forward request and measure time
            try {
                $response = Invoke-WebRequest $backend.url -UseBasicParsing -TimeoutSec 2
                $context.Response.StatusCode = 200
                
                $backend.requests++
                $backend.lastHealthy = $true
                
                [byte[]]$buffer = [System.Text.Encoding]::UTF8.GetBytes("OK")
                $context.Response.ContentLength64 = $buffer.Length
                $context.Response.OutputStream.Write($buffer, 0, $buffer.Length)
            } catch {
                $context.Response.StatusCode = 503
                $backend.errors++
                $backend.lastHealthy = $false
                
                [byte[]]$buffer = [System.Text.Encoding]::UTF8.GetBytes("Error")
                $context.Response.ContentLength64 = $buffer.Length
                $context.Response.OutputStream.Write($buffer, 0, $buffer.Length)
            }
            
            $context.Response.Close()
        }
        
        $backend.totalTime += $requestTime.TotalMilliseconds
        
    } catch {
        Write-Error $_
    }
}

# Generate metrics report
Write-Host "`n=== LOAD BALANCER METRICS REPORT ===" -ForegroundColor Cyan
Write-Host "Total Requests: $totalRequests"
Write-Host "Duration: $((Get-Date) - $startTime)"
Write-Host ""

$totalTime = 0
foreach ($backend in $backends) {
    $percentage = ($backend.requests / $totalRequests) * 100
    $avgTime = if ($backend.requests -gt 0) {$backend.totalTime / $backend.requests} else {0}
    $errorRate = if ($backend.requests -gt 0) {($backend.errors / $backend.requests) * 100} else {0}
    $health = if ($backend.lastHealthy) {"HEALTHY"} else {"DOWN"}
    
    Write-Host "Server $($backend.port):" -ForegroundColor Yellow
    Write-Host "  Requests: $($backend.requests) ($([Math]::Round($percentage, 1))%)"
    Write-Host "  Errors: $($backend.errors) ($([Math]::Round($errorRate, 1))%)"
    Write-Host "  Avg Response: $([Math]::Round($avgTime, 2))ms"
    Write-Host "  Status: $health"
    Write-Host ""
    
    $totalTime += $backend.totalTime
}

Write-Host "Overall Avg Response Time: $([Math]::Round($totalTime / $totalRequests, 2))ms"
```

**Metrics Report Output:**
```
=== LOAD BALANCER METRICS REPORT ===
Total Requests: 30
Duration: 00:00:15

Server 8001:
  Requests: 10 (33.3%)
  Errors: 0 (0%)
  Avg Response: 25.43ms
  Status: HEALTHY

Server 8002:
  Requests: 10 (33.3%)
  Errors: 0 (0%)
  Avg Response: 24.67ms
  Status: HEALTHY

Server 8003:
  Requests: 10 (33.3%)
  Errors: 0 (0%)
  Avg Response: 25.12ms
  Status: HEALTHY

Overall Avg Response Time: 25.07ms
```

**Metrics Interpretation:**
- ✓ Perfect distribution (33% each server)
- ✓ No errors on any backend
- ✓ Response times consistent (~25ms)
- ✓ All servers healthy
- Conclusion: Load balancer working optimally

**Identifying Imbalances:**
If distribution was uneven:
```
Server 8001: 15 requests (50%)
Server 8002: 10 requests (33%)
Server 8003: 5 requests (17%)

→ Problem: Algorithm or health check issue
→ Solution: Check round-robin implementation
```

---

## Challenge Exercise Solution

### Configure HAProxy or Nginx with Multiple Features

**HAProxy Production Configuration:**
```haproxy
global
    log stdout local0
    maxconn 4096
    daemon
    
defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000

# Frontend for incoming requests
frontend web_frontend
    bind *:80
    mode http
    default_backend web_backend

# Backend servers with health checks
backend web_backend
    mode http
    balance roundrobin
    option httpchk GET /
    
    # Servers with health checks
    server backend1 localhost:8001 check inter 5s fall 2 rise 2 weight 2
    server backend2 localhost:8002 check inter 5s fall 2 rise 2 weight 1
    server backend3 localhost:8003 check inter 5s fall 2 rise 2 weight 1
    
    # Sticky sessions
    cookie SERVERID insert indirect nocache
    
    # Session sharing configuration
    stick-table type string len 32 size 100k expire 1h
    stick on cookie(SERVERID)

# Stats page
listen stats
    bind *:8080
    mode http
    stats enable
    stats uri /stats
    stats refresh 30s
```

**Key Features Demonstrated:**
1. ✓ Multiple backend servers
2. ✓ Health checks (every 5 seconds)
3. ✓ Sticky sessions (cookie-based)
4. ✓ Weighted round robin (2:1:1)
5. ✓ Access logging
6. ✓ Stats monitoring
7. ✓ Failover handling

**Testing Commands:**
```bash
# Start backends
python3 -m http.server 8001 &
python3 -m http.server 8002 &
python3 -m http.server 8003 &

# Start HAProxy
haproxy -f haproxy.cfg

# Test load balancing
for i in {1..10}; do curl http://localhost/; done

# Monitor stats
curl http://localhost:8080/stats

# Kill a backend and test failover
kill %1
for i in {1..5}; do curl http://localhost/; done
```

**Validation:**
- Load distributed evenly
- Health checks detect dead servers
- Sessions persist across requests
- No manual intervention needed during failure
- Statistics show correct distribution

