# Exercises: HTTP, HTTPS, and TLS

Complete these exercises to master HTTP, HTTPS, and TLS/SSL concepts. Progress from Easy to Medium difficulty.

## Easy Exercises

### Exercise 1: Compare HTTP and HTTPS
**Objective:** Understand the fundamental differences between HTTP and HTTPS

Request the same website over both HTTP and HTTPS:
```powershell
# Test HTTP (if available)
curl -v http://example.com 2>&1 | Select-String "HTTP/1.1|200"

# Test HTTPS
curl -v https://example.com 2>&1 | Select-String "HTTP/1.1|200|SSL|TLS"
```

**Task:**
- Execute both commands
- Compare the output
- Note differences in connection details
- Identify which uses encryption

**Success Criteria:**
- Both requests complete successfully
- You observe TLS/SSL in HTTPS output
- HTTP shows plain text headers
- You understand why HTTPS is more secure

---

### Exercise 2: Inspect HTTP Headers
**Objective:** Learn what information HTTP headers contain

Query a public API and inspect headers:
```powershell
curl -i https://httpbin.org/get
```

**Task:**
- Run the command
- Identify and write down:
  - Content-Type header
  - Server header
  - Content-Length header
  - Any Set-Cookie headers
- Understand what each header means

**Success Criteria:**
- You find at least 5 different headers
- You understand the purpose of each header
- Response includes a status code (200, 404, etc.)

---

### Exercise 3: Check Certificate Validity
**Objective:** Verify SSL certificates and understand certificate components

Inspect a well-known website's certificate:
```powershell
curl -v https://google.com 2>&1 | findstr "subject:|issuer:|date"
```

**Task:**
- Run the command
- Find and note:
  - Certificate subject (domain name)
  - Certificate issuer (who signed it)
  - Not Before date
  - Not After (expiration) date
- Verify the certificate is currently valid

**Success Criteria:**
- Certificate subject matches google.com
- Current date is between "Not Before" and "Not After"
- Issuer is a trusted Certificate Authority
- No certificate errors appear

---

### Exercise 4: Test Self-Signed Certificate (Windows)
**Objective:** Create and inspect a self-signed certificate

Generate a self-signed certificate:
```powershell
$params = @{
    CertStoreLocation = 'Cert:\LocalMachine\My'
    DnsName = 'localhost'
    FriendlyName = 'LocalTest'
    NotAfter = (Get-Date).AddYears(1)
}
New-SelfSignedCertificate @params
```

**Task:**
- Execute the command
- Note the Thumbprint returned
- Verify it appears in certificate store:
```powershell
Get-ChildItem Cert:\LocalMachine\My | Where-Object {$_.FriendlyName -eq 'LocalTest'}
```
- Record the certificate details

**Success Criteria:**
- Certificate created successfully
- Certificate appears in LocalMachine\My store
- FriendlyName is "LocalTest"
- Thumbprint is visible

---

### Exercise 5: Test HTTPS Endpoint with curl
**Objective:** Use curl to test HTTPS connections and diagnose issues

Test various HTTPS endpoints:
```powershell
# GitHub
curl -I https://github.com

# AWS
curl -I https://aws.amazon.com

# Your local test
curl -I https://localhost:443
```

**Task:**
- Execute the commands for at least 2 sites
- Note the HTTP status code returned
- Record response headers
- Identify any certificate errors

**Success Criteria:**
- You receive HTTP responses (200, 301, etc.)
- Status codes are correctly interpreted
- No certificate warnings for public websites
- localhost may show certificate error (expected)

---

## Medium Exercises

### Exercise 6: Compare TLS Versions
**Objective:** Understand TLS versions and their differences

Test TLS version negotiation:
```powershell
# Test with different TLS versions
$ErrorActionPreference = 'SilentlyContinue'

# Current default
[System.Net.ServicePointManager]::SecurityProtocol

# Test specific TLS version
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12
$response = Invoke-WebRequest https://www.howsmyssl.com/a/check -Method GET
$response.RawContent | Select-String "tls"

# Reset to default
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12, [System.Net.SecurityProtocolType]::Tls13
```

**Task:**
- Check your current default TLS version
- Change to TLS 1.2 and test
- Change to TLS 1.3 if available
- Compare results from howsmyssl.com
- Document findings

**Success Criteria:**
- You understand what SecurityProtocol property does
- You can set TLS versions programmatically
- You verify which TLS version is being used
- Results show security assessment from howsmyssl

---

### Exercise 7: Inspect Certificate Chain
**Objective:** Understand certificate hierarchy and validation

Examine the full certificate chain of a major website:
```bash
# Using OpenSSL (Linux/Mac) or WSL
openssl s_client -connect google.com:443 -showcerts | grep -A 5 "subject="
```

**Windows PowerShell Alternative:**
```powershell
$cert = [System.Net.ServicePointManager]::FindServicePoint("https://google.com").Certificate
$chain = New-Object System.Security.Cryptography.X509Certificates.X509Chain
$chain.Build($cert)
$chain.ChainElements | ForEach-Object {$_.Certificate.Subject}
```

**Task:**
- Execute the command
- Identify the certificate chain:
  - Server certificate (leaf)
  - Intermediate certificate(s)
  - Root Certificate Authority
- Understand the chain of trust
- Document each certificate's subject and issuer

**Success Criteria:**
- You see at least 2-3 certificates in the chain
- Server cert is issued by intermediate
- Intermediate is issued by root CA
- You understand why chains exist (scalability and security)

---

### Exercise 8: Monitor HTTPS Traffic
**Objective:** Capture and analyze HTTPS traffic characteristics

Monitor traffic to an HTTPS endpoint:
```powershell
# Using NetStat to monitor ports
netstat -an | findstr "443"

# Before and after request
netstat -an | findstr "ESTABLISHED.*443"

# Count active HTTPS connections
(netstat -an | findstr "443" | measure-object).Count
```

**Task:**
- Check active HTTPS connections
- Make an HTTPS request to a website
- Check connections again
- Document the difference
- Identify connection states (SYN, ESTABLISHED, CLOSE_WAIT)

**Success Criteria:**
- You can monitor port 443 traffic
- You see established connections
- You understand the difference before/after requests
- You recognize connection states

---

### Exercise 9: Create and Validate a Self-Signed Certificate Chain
**Objective:** Build a certificate chain with root and intermediate certs (Linux/Mac or WSL)

Create a minimal certificate chain:
```bash
# Create root CA private key
openssl genrsa -out root-key.pem 2048

# Create root certificate
openssl req -new -x509 -days 3650 -key root-key.pem -out root-cert.pem \
  -subj "/CN=LocalRootCA"

# Create server private key
openssl genrsa -out server-key.pem 2048

# Create server certificate signing request
openssl req -new -key server-key.pem -out server.csr \
  -subj "/CN=localhost"

# Sign server certificate with root CA
openssl x509 -req -days 365 -in server.csr \
  -CA root-cert.pem -CAkey root-key.pem -CAcreateserial \
  -out server-cert.pem

# Verify the chain
openssl verify -CAfile root-cert.pem server-cert.pem
```

**Task:**
- Execute all commands
- Verify certificate creation: `ls -la *.pem`
- Check certificate details: `openssl x509 -in root-cert.pem -text -noout`
- Verify chain validity
- Document the hierarchy

**Success Criteria:**
- Root CA certificate created
- Server certificate signed by root CA
- Chain verification shows "OK"
- All PEM files are present
- You understand the relationship between certificates

---

### Exercise 10: Diagnose Common HTTPS Issues
**Objective:** Troubleshoot typical HTTPS and certificate problems

Simulate and troubleshoot common issues:
```powershell
# 1. Test with self-signed cert (expect error)
curl -v https://localhost:8443 2>&1 | findstr "certificate"

# 2. Skip certificate verification (use -k flag)
curl -k -v https://localhost:8443

# 3. Test expired or invalid certificate
curl -v https://self-signed.badssl.com/ 2>&1 | findstr "certificate|error"

# 4. Check certificate dates
curl -v https://google.com 2>&1 | findstr "start date|expire date"

# 5. Test hostname mismatch
curl -v https://example.com --resolve example.com:443:127.0.0.1 2>&1 | findstr "subject:|error"
```

**Task:**
- Execute each test
- Document the error message for each scenario
- Understand why each error occurs
- Identify the solution for each issue
- Create a troubleshooting reference

**Success Criteria:**
- You recognize certificate-related error messages
- You understand root causes for each error
- You know when to use `-k` flag (testing only)
- You can explain solutions to each problem
- You document findings for reference

---

## Challenge Exercise (Optional)

### Configure HTTPS on a Local Web Server

**Objective:** Set up a real HTTPS server with certificate

1. Create a self-signed certificate
2. Configure IIS (Windows) or Apache/Nginx (Linux) with the certificate
3. Access the server over HTTPS
4. Verify certificate and TLS connection
5. Compare HTTP vs HTTPS traffic in browser dev tools

This combines all learned concepts and provides practical HTTPS experience.
