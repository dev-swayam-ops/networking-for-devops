# Exercises: Load Balancing and Proxies

Complete these exercises to master load balancing concepts and proxy configurations. Progress from Easy to Medium difficulty.

## Easy Exercises

### Exercise 1: Understand Load Balancing Algorithms
**Objective:** Learn how different load balancing algorithms work

Research and compare these algorithms:
```
Algorithms to explore:
1. Round Robin
2. Least Connections
3. IP Hash
4. Weighted Round Robin
5. Random
```

**Task:**
- Define each algorithm in your own words
- Identify use case for each algorithm
- Explain pros and cons
- Document when to use which algorithm

**Success Criteria:**
- You can explain all 5 algorithms clearly
- You understand when each is most appropriate
- You recognize trade-offs between algorithms
- You can identify algorithm from behavior description

---

### Exercise 2: Test Multiple Backend Servers Locally
**Objective:** Set up and test simple backend services

Create three simple HTTP servers on different ports:
```bash
# Linux/Mac
python3 -m http.server 8001 &
python3 -m http.server 8002 &
python3 -m http.server 8003 &
```

**PowerShell Alternative:**
```powershell
# Create three listeners on ports 8001, 8002, 8003
# (See README.md Hands-on Lab for example)
```

**Task:**
- Start 3 backend servers on ports 8001, 8002, 8003
- Verify each server responds to requests
- Test connectivity to each backend
- Document responses from each server

**Success Criteria:**
- All three servers are running
- Each responds with unique identifier
- You can access all three from curl
- Servers run independently

---

### Exercise 3: Identify Layer 4 vs Layer 7 Load Balancing
**Objective:** Understand differences between L4 and L7 load balancing

Compare these two load balancing approaches:
```
Layer 4 (Transport):
- Works with TCP/UDP
- No content inspection
- Fast routing
- Good for: Gaming, VoIP, non-HTTP

Layer 7 (Application):
- Works with HTTP/HTTPS
- Can read content
- Can route by URL or cookies
- Good for: Web APIs, microservices
```

**Task:**
- Research L4 and L7 load balancing
- Create comparison table with 6+ attributes
- List pros and cons of each
- Identify which layer you'd use for:
  - Web API with URL-based routing
  - Gaming server with thousands of players
  - IoT devices sending binary data
  - Microservice architecture

**Success Criteria:**
- You understand both layers clearly
- Comparison table includes key differences
- You can decide L4 vs L7 for scenarios
- You know performance implications

---

### Exercise 4: Monitor Backend Health via Netstat
**Objective:** Use netstat to check server connection states

Monitor connections to backend servers:
```powershell
# Monitor port 8001
netstat -an | findstr :8001

# Monitor all backend ports
netstat -an | findstr ":800[1-3]"

# Show established connections
netstat -an | findstr "ESTABLISHED" | findstr ":800[1-3]"
```

**Task:**
- Make requests to all three backend servers
- Use netstat to monitor connections
- Identify connection states
- Track active connection count per backend
- Document connection lifecycle

**Success Criteria:**
- You can monitor connections with netstat
- You recognize ESTABLISHED, LISTENING, CLOSE_WAIT states
- You understand connection state transitions
- You can count active connections per server

---

### Exercise 5: Test Simple Reverse Proxy
**Objective:** Experience basic reverse proxy functionality

Configure a simple reverse proxy:
```bash
# Linux: Using simple Python proxy or nginx
# Create basic proxy config that routes to backend servers
```

**Task:**
- Set up basic reverse proxy on port 80
- Configure it to route traffic to backends
- Make requests through proxy
- Verify requests reach backends
- Document proxy behavior

**Success Criteria:**
- Proxy is running on port 80
- Requests go through proxy to backends
- Backends receive requests correctly
- You can trace request path through proxy

---

## Medium Exercises

### Exercise 6: Implement Round Robin Load Balancing
**Objective:** Create working round robin distribution

Implement a round-robin load balancer:
```powershell
# Create a load balancer that:
# 1. Accepts requests on port 80
# 2. Routes to backends in order: 8001 -> 8002 -> 8003 -> 8001
# 3. Tracks which backend receives each request
```

**Detailed Requirements:**
```
- Maintain counter for current backend
- Increment counter after each request
- Use modulo arithmetic for cycling
- Verify even distribution across 9 requests
```

**Task:**
- Create round-robin load balancer script
- Make 9 requests through load balancer
- Verify requests distribute evenly: 3 to each backend
- Test with requests in rapid succession
- Document distribution pattern

**Success Criteria:**
- Requests distribute evenly (±1 per server)
- Rotation works correctly
- All backends receive traffic
- No server is overloaded
- Distribution is predictable

---

### Exercise 7: Add Health Checks to Load Balancer
**Objective:** Implement health checking for backend servers

Enhance load balancer with health checks:
```powershell
# Load balancer should:
# 1. Periodically test each backend (every 5 seconds)
# 2. Mark unhealthy servers as DOWN
# 3. Skip dead servers in round-robin
# 4. Resume using server when it comes back online
```

**Detailed Requirements:**
```
- Use TCP connect or HTTP HEAD request
- Track each backend's status
- Log health check results
- Skip dead servers automatically
- Resume when server recovers
```

**Task:**
- Add health check function to load balancer
- Start all three backends
- Stop one backend
- Make requests - verify skipped server
- Restart stopped backend
- Verify it's used again

**Success Criteria:**
- Health checks run every ~5 seconds
- Dead server is detected quickly
- Requests don't go to dead server
- Server recovery is detected
- Requests resume to recovered server

---

### Exercise 8: Configure Sticky Sessions (Session Persistence)
**Objective:** Implement cookie-based session persistence

Create a session-aware load balancer:
```powershell
# Load balancer should:
# 1. Set a cookie on first request
# 2. Remember which backend served client
# 3. Route subsequent requests from same client to same backend
```

**Detailed Requirements:**
```
- Use cookie-based session tracking
- Set cookie like: LB-Server=8001
- Route requests with same cookie to same backend
- Test with multiple clients simultaneously
```

**Task:**
- Modify load balancer to set session cookie
- Simulate multiple clients (different IP sources)
- Verify each client always goes to same backend
- Test that cookie value matches backend port
- Verify new clients get different backends

**Success Criteria:**
- Cookies are set on first request
- Same client always hits same backend
- Cookies persist across requests
- Different clients can go to different backends
- Sessions are maintained correctly

---

### Exercise 9: Implement Weighted Round Robin
**Objective:** Distribute load based on server capacity

Configure weighted round robin:
```powershell
# Set weights for backends:
# Server 1 (8001): weight=2 (gets 2 requests per cycle)
# Server 2 (8002): weight=1 (gets 1 request per cycle)
# Server 3 (8003): weight=1 (gets 1 request per cycle)
# Total cycle: 4 requests (2+1+1)
```

**Detailed Requirements:**
```
- Maintain weights for each backend
- Distribute requests proportionally
- Cycle through weighted pattern
- Verify 2:1:1 distribution over 8 requests
```

**Task:**
- Add weight configuration to load balancer
- Set weights: 2, 1, 1 for servers 1, 2, 3
- Make 8 requests through load balancer
- Verify distribution:
  - Server 1: 4 requests (50%)
  - Server 2: 2 requests (25%)
  - Server 3: 2 requests (25%)
- Document weight pattern

**Success Criteria:**
- Distribution matches configured weights
- Weighted pattern cycles correctly
- All backends receive traffic proportionally
- Configuration is easily changeable
- Different weights work as expected

---

### Exercise 10: Monitor and Analyze Load Balancer Metrics
**Objective:** Implement monitoring for load balancer

Create comprehensive load balancer monitoring:
```powershell
# Track metrics:
# 1. Requests per backend
# 2. Average response time per backend
# 3. Failed requests per backend
# 4. Server health status
# 5. Load distribution percentage
```

**Detailed Requirements:**
```
- Count requests sent to each backend
- Calculate response times
- Track errors/failures
- Log all metrics
- Display metrics summary
```

**Task:**
- Add metrics collection to load balancer
- Make 30 requests with varying response times
- Generate summary report:
  - Total requests per server
  - Percentage distribution
  - Response time stats
  - Error rate per server
  - Health status of all backends
- Identify any imbalances

**Success Criteria:**
- All metrics are collected
- Distribution is roughly equal (for round robin)
- Metrics are accurate and logged
- You can identify load imbalances
- Report shows clear performance picture

---

## Challenge Exercise (Optional)

### Configure HAProxy or Nginx with Multiple Features

**Objective:** Implement production-ready load balancer

1. Install HAProxy or nginx
2. Configure with:
   - Multiple backend servers
   - Health checks (HTTP 200 verification)
   - Sticky sessions (cookie-based)
   - Weighted round robin
   - SSL/TLS termination
   - Access logging
   - Request routing by URL path
3. Monitor metrics and logs
4. Test failover scenarios
5. Performance test with multiple concurrent clients

This combines all learned concepts and provides real-world load balancing experience.
