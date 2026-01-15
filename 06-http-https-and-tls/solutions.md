# Solutions: HTTP, HTTPS, and TLS

Complete solutions with explanations for all exercises.

## Easy Exercise Solutions

### Exercise 1: Compare HTTP and HTTPS - Solution

**Command Execution:**
```powershell
# Test HTTP
curl -v http://example.com 2>&1 | Select-String "HTTP/1.1|200"

# Test HTTPS
curl -v https://example.com 2>&1 | Select-String "HTTP/1.1|200|SSL|TLS"
```

**Expected Output:**
```
HTTP/1.1 200 OK                          # Both should return 200
...SSL/TLS handshake visible in HTTPS
```

**Explanation:**
- HTTP uses port 80 and transmits data in plain text
- HTTPS uses port 443 and encrypts data with TLS/SSL
- HTTPS requires a certificate to establish encrypted connection
- Both return 200 status, but HTTPS shows encryption details in verbose output
- TLS handshake is visible with `-v` flag, showing cipher suite negotiation

**Key Difference Identified:**
- HTTP: No encryption, no certificate validation
- HTTPS: Full encryption, certificate validation, TLS version negotiation

---

### Exercise 2: Inspect HTTP Headers - Solution

**Command:**
```powershell
curl -i https://httpbin.org/get
```

**Expected Output:**
```
HTTP/2 200
date: Mon, 15 Jan 2024 14:30:00 GMT
content-type: application/json
content-length: 456
server: gunicorn/19.9.0
access-control-allow-origin: *
access-control-allow-credentials: true
```

**Explanation:**
- `date` - When the response was generated (GMT format)
- `content-type: application/json` - Response body format is JSON
- `content-length: 456` - Response body is 456 bytes
- `server` - Web server software and version
- `access-control-allow-origin: *` - CORS: allows requests from any origin
- `access-control-allow-credentials: true` - CORS: allows credentials in requests

**Common Headers Explained:**
| Header | Purpose |
|--------|---------|
| `Content-Type` | Format of response (HTML, JSON, etc.) |
| `Content-Length` | Size of response body in bytes |
| `Server` | Web server software |
| `Cache-Control` | How to cache the response |
| `Set-Cookie` | Cookies to store on client |
| `Authorization` | Authentication token/credentials |
| `Accept` | What formats client accepts |

**Key Learning:**
- Headers contain metadata about the request/response
- Status code (200) indicates success
- Content-Type tells how to interpret the body

---

### Exercise 3: Check Certificate Validity - Solution

**Command:**
```powershell
curl -v https://google.com 2>&1 | findstr "subject:|issuer:|date"
```

**Expected Output:**
```
* subject: CN=google.com
* issuer: C=US, O=Google Trust Services LLC, CN=GTS CA 1D4
* start date: Dec 16, 2023
* expire date: Dec 15, 2024
```

**Explanation:**
- `subject: CN=google.com` - Certificate is valid for google.com domain
- `issuer: ... GTS CA 1D4` - Google's own intermediate CA signed this
- `start date` - When certificate became valid
- `expire date` - When certificate will no longer be valid
- Current date must be between start and expire date

**Certificate Validation Checklist:**
- ✓ Subject matches requested domain (google.com == google.com)
- ✓ Certificate is within valid date range
- ✓ Issuer is a trusted Certificate Authority
- ✓ Signature is valid (curl verifies automatically)

**Key Concepts:**
- Certificates prove the server owns the domain
- Issued by trusted Certificate Authorities
- Have expiration dates (typically 1-2 years)
- Browsers warn if certificate is invalid or expired

---

### Exercise 4: Test Self-Signed Certificate (Windows) - Solution

**Commands:**
```powershell
# Create certificate
$params = @{
    CertStoreLocation = 'Cert:\LocalMachine\My'
    DnsName = 'localhost'
    FriendlyName = 'LocalTest'
    NotAfter = (Get-Date).AddYears(1)
}
New-SelfSignedCertificate @params

# Verify certificate
Get-ChildItem Cert:\LocalMachine\My | Where-Object {$_.FriendlyName -eq 'LocalTest'} | Format-List Subject, Thumbprint, NotBefore, NotAfter
```

**Expected Output:**
```
Subject      : CN=localhost
Thumbprint   : ABC123DEF456789...
NotBefore    : 1/15/2024 2:00:00 PM
NotAfter     : 1/15/2025 2:00:00 PM
```

**Explanation:**
- `CertStoreLocation` - Where to store: LocalMachine\My (personal store)
- `DnsName` - Common Name (CN) for certificate
- `FriendlyName` - Easy identifier for humans
- `NotAfter` - Set to 1 year from now
- Certificate created in Windows certificate store
- Can be used for HTTPS testing on localhost

**Self-Signed vs CA-Signed:**
| Self-Signed | CA-Signed |
|-------------|-----------|
| Created by owner | Created by Certificate Authority |
| Not trusted by browsers | Trusted by browsers |
| Good for testing | Required for production |
| Shows warnings | No warnings |
| Free | Cost varies |

---

### Exercise 5: Test HTTPS Endpoint with curl - Solution

**Commands:**
```powershell
# Test GitHub
curl -I https://github.com

# Test AWS
curl -I https://aws.amazon.com

# Test localhost (expect error)
curl -I https://localhost:443
```

**Expected Output:**
```
# GitHub
HTTP/2 200
server: GitHub.com
content-type: text/html; charset=utf-8

# AWS
HTTP/2 200
content-type: text/html; charset=UTF-8

# Localhost
curl: (7) Failed to connect to localhost port 443: Connection refused
# OR certificate error if HTTPS service running
```

**Explanation:**
- `curl -I` - HEAD request (headers only, no body)
- `HTTP/2 200` - Successful response using HTTP/2
- Different servers return different headers
- localhost:443 likely fails because no HTTPS service running
- Public websites have valid CA-signed certificates (no errors)

**Status Codes Explained:**
- `200` - Success
- `301` - Permanent redirect
- `404` - Not found
- `500` - Server error

**Key Learning:**
- curl can test HTTPS endpoints
- Public websites use trusted certificates
- Local testing needs self-signed or no certificate

---

## Medium Exercise Solutions

### Exercise 6: Compare TLS Versions - Solution

**Commands:**
```powershell
# Check current TLS version
[System.Net.ServicePointManager]::SecurityProtocol

# Output example
Tls, Tls11, Tls12, Tls13

# Set to TLS 1.2
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12

# Verify
[System.Net.ServicePointManager]::SecurityProtocol
# Output: Tls12

# Test with howsmyssl.com
$response = Invoke-WebRequest "https://www.howsmyssl.com/a/check" -UseBasicParsing
$content = $response.Content | ConvertFrom-Json
$content.tls_version
# Output: TLS 1.2
```

**Expected Output:**
```
TLS Version Test Results:
Default: Tls, Tls11, Tls12, Tls13
After setting to Tls12: Tls12
Server sees: TLS 1.2
```

**Explanation:**
- `SecurityProtocol` property controls TLS version used by .NET
- Default allows multiple versions (backward compatibility)
- TLS 1.3 is newest (faster, more secure)
- TLS 1.0, 1.1 are deprecated (weak, vulnerable)
- Server and client must support same TLS version

**TLS Version History:**
| Version | Year | Status | Notes |
|---------|------|--------|-------|
| SSL 3.0 | 1996 | Deprecated | Broken security |
| TLS 1.0 | 1999 | Deprecated | Vulnerabilities |
| TLS 1.1 | 2006 | Deprecated | Weak cipher support |
| TLS 1.2 | 2008 | Supported | Industry standard |
| TLS 1.3 | 2018 | Recommended | Faster, more secure |

**Key Learning:**
- Always use TLS 1.2 or higher in production
- TLS 1.3 is preferred for new implementations
- Verify client and server TLS version compatibility

---

### Exercise 7: Inspect Certificate Chain - Solution

**Windows PowerShell Solution:**
```powershell
$cert = [System.Net.ServicePointManager]::FindServicePoint("https://google.com").Certificate
$chain = New-Object System.Security.Cryptography.X509Certificates.X509Chain
$chain.Build($cert)

Write-Host "Certificate Chain for google.com:"
Write-Host "================================="
$chain.ChainElements | ForEach-Object -Begin {$i=0} -Process {
    Write-Host "[$i] Subject: $($_.Certificate.Subject)"
    Write-Host "    Issuer:  $($_.Certificate.Issuer)"
    Write-Host "    Thumbprint: $($_.Certificate.Thumbprint)"
    $i++
}
```

**Expected Output:**
```
Certificate Chain for google.com:
=================================
[0] Subject: CN=google.com
    Issuer:  C=US, O=Google Trust Services LLC, CN=GTS CA 1D4
    Thumbprint: ABC123...

[1] Subject: C=US, O=Google Trust Services LLC, CN=GTS CA 1D4
    Issuer:  C=US, O=Google Trust Services LLC, CN=GTS Root R4
    Thumbprint: DEF456...

[2] Subject: C=US, O=Google Trust Services LLC, CN=GTS Root R4
    Issuer:  C=US, O=GlobalSign nv-sa, CN=GlobalSign Root CA
    Thumbprint: GHI789...
```

**Explanation:**
- **[0] Server/Leaf Certificate** - google.com's certificate
- **[1] Intermediate Certificate** - Issued by Google's CA
- **[2] Root Certificate** - Ultimately issued by GlobalSign (trusted by OS)

**Why Certificate Chains?**
1. **Security** - Root CA kept offline (maximum protection)
2. **Scalability** - Multiple intermediates can issue certificates
3. **Flexibility** - Rotate certificates without trusting root

**Chain Validation:**
- Each certificate signs the next one in chain
- Root CA is trusted by operating system
- Browsers verify entire chain before accepting

---

### Exercise 8: Monitor HTTPS Traffic - Solution

**Commands:**
```powershell
# Check HTTPS connections before request
Write-Host "Before request:"
netstat -an | findstr ":443" | Measure-Object | Select-Object Count

# Make HTTPS request
Invoke-WebRequest https://google.com -UseBasicParsing | Out-Null

# Check after request
Write-Host "After request:"
netstat -an | findstr ":443" | Measure-Object | Select-Object Count

# Monitor established connections
Write-Host "Established HTTPS connections:"
netstat -an | findstr "443.*ESTABLISHED"
```

**Expected Output:**
```
Before request:
Count : 0

After request:
Count : 1

Established HTTPS connections:
TCP    192.168.1.100:54321    142.250.185.46:443    ESTABLISHED
```

**Explanation:**
- Before request: No established connections to port 443
- During request: Connection appears in ESTABLISHED state
- After request completes: Connection moves to CLOSE_WAIT or closes
- Multiple requests may see multiple connections (connection pooling)

**Connection States:**
| State | Meaning |
|-------|---------|
| ESTABLISHED | Active connection |
| SYN_SENT | Initiating connection |
| SYN_RECEIVED | Receiving connection |
| CLOSE_WAIT | Waiting to close |
| TIME_WAIT | Waiting before final close |
| LISTENING | Waiting for connections |

**Key Observation:**
- Port 443 indicates HTTPS
- Remote IP is destination server
- Local port is ephemeral (random high port)

---

### Exercise 9: Create and Validate Self-Signed Certificate Chain - Solution

**Commands (Linux/Mac/WSL):**
```bash
# 1. Create root CA private key (4096-bit RSA)
openssl genrsa -out root-key.pem 2048

# 2. Create root CA certificate (valid 10 years)
openssl req -new -x509 -days 3650 -key root-key.pem -out root-cert.pem \
  -subj "/CN=LocalRootCA"

# 3. Create server private key
openssl genrsa -out server-key.pem 2048

# 4. Create certificate signing request (CSR)
openssl req -new -key server-key.pem -out server.csr \
  -subj "/CN=localhost"

# 5. Sign server certificate with root CA
openssl x509 -req -days 365 -in server.csr \
  -CA root-cert.pem -CAkey root-key.pem -CAcreateserial \
  -out server-cert.pem

# 6. Verify the certificate chain
openssl verify -CAfile root-cert.pem server-cert.pem

# 7. Inspect root certificate
openssl x509 -in root-cert.pem -text -noout | grep -E "Subject:|Issuer:|Not Before|Not After"

# 8. Inspect server certificate
openssl x509 -in server-cert.pem -text -noout | grep -E "Subject:|Issuer:|Not Before|Not After"
```

**Expected Output:**
```
# After verify command:
server-cert.pem: OK

# Root cert details:
Subject: CN=LocalRootCA
Issuer: CN=LocalRootCA
Not Before: Jan 15 14:30:00 2024 GMT
Not After: Jan 12 14:30:00 2034 GMT

# Server cert details:
Subject: CN=localhost
Issuer: CN=LocalRootCA
Not Before: Jan 15 14:30:01 2024 GMT
Not After: Jan 15 14:30:01 2025 GMT
```

**Explanation:**
- **Root CA Private Key** - Secret key to sign other certs
- **Root CA Certificate** - Self-signed (Subject = Issuer)
- **Server CSR** - Request to be signed by root CA
- **Server Certificate** - Signed by root CA (Issuer = Root CN)
- **Verification** - Chain is valid (OK message)

**File Generated:**
- `root-key.pem` - Root CA private key (keep secure!)
- `root-cert.pem` - Root CA certificate (distribute to clients)
- `server-key.pem` - Server private key (keep secure!)
- `server-cert.pem` - Server certificate (install on server)
- `server.csr` - Can be deleted (no longer needed)
- `root-cert.srl` - Serial number tracking

---

### Exercise 10: Diagnose Common HTTPS Issues - Solution

**Test 1: Self-Signed Certificate Error**
```powershell
curl -v https://localhost:8443 2>&1 | findstr "certificate"
```
**Output:**
```
* certificate problem: self signed certificate
* SSL: CERTIFICATE_VERIFY_FAILED
```
**Issue:** Certificate not signed by trusted CA
**Solution:** 
- Use `-k` flag for testing: `curl -k https://localhost:8443`
- Install self-signed cert as trusted (production: use CA-signed)

---

**Test 2: Skip Certificate Verification**
```powershell
curl -k -v https://localhost:8443
```
**Output:**
```
* SSL: Certificate verification skipped (-k flag)
* Connection successful
```
**Explanation:**
- `-k` flag disables certificate verification
- **Only for testing!** Never use in production
- Dangerous: allows man-in-the-middle attacks

---

**Test 3: Certificate Error on badssl.com**
```powershell
curl -v https://self-signed.badssl.com/ 2>&1 | findstr "certificate|error"
```
**Output:**
```
* certificate problem: self signed certificate (18)
* SSL: CERTIFICATE_VERIFY_FAILED
```
**Issue:** Demonstrates actual certificate errors
**Learning:** Real-world websites with bad certificates

---

**Test 4: Check Certificate Dates**
```powershell
curl -v https://google.com 2>&1 | findstr "start date|expire date"
```
**Output:**
```
* start date: Dec 16, 2023
* expire date: Dec 15, 2024
```
**Check:** Verify current date is within range
**Issue:** Expired certs cause CERTIFICATE_VERIFY_FAILED error
**Solution:** Renew certificate before expiration

---

**Test 5: Hostname Mismatch**
```powershell
curl -v https://example.com --resolve example.com:443:127.0.0.1 2>&1
```
**Output:**
```
* subject: CN=different-domain.com
* SSL: Certificate subject name 'different-domain.com' does not match target name 'example.com'
```
**Issue:** Certificate CN doesn't match requested hostname
**Solution:**
- Use correct hostname
- Get certificate with Subject Alternative Names (SANs)
- Use wildcard certificate (*.example.com)

---

**Troubleshooting Reference Table:**

| Error | Cause | Solution |
|-------|-------|----------|
| CERTIFICATE_VERIFY_FAILED | Self-signed or untrusted | Use `-k` for testing, CA-signed for production |
| Subject name mismatch | Wrong hostname | Use correct domain or wildcard cert |
| Certificate expired | Past expiration date | Renew certificate |
| Certificate not yet valid | Future date on cert | Check system time, or wait for validity start |
| Unable to get local issuer certificate | Root CA not trusted | Add root CA to trusted store |

**Key Learning:**
- Always read certificate error messages carefully
- Understand root cause before troubleshooting
- Use `-k` flag only for development/testing
- Production systems must have valid CA-signed certificates
- Monitor certificate expiration dates

---

## Challenge Exercise Solution: Configure HTTPS on Local Web Server

**Objective:** Complete HTTPS setup with real server

**Step 1: Create Self-Signed Certificate**
```powershell
$params = @{
    CertStoreLocation = 'Cert:\LocalMachine\My'
    DnsName = 'localhost'
    FriendlyName = 'LocalWebServer'
    NotAfter = (Get-Date).AddYears(1)
}
$cert = New-SelfSignedCertificate @params
$thumbprint = $cert.Thumbprint
```

**Step 2: Configure IIS Binding (Windows)**
```powershell
# Add HTTPS binding in IIS
New-IISSiteBinding -Name "Default Web Site" `
  -BindingInformation "*:443:localhost" `
  -Protocol https `
  -CertificateThumbprint $thumbprint `
  -CertificateStoreName My

# Or manually:
# IIS Manager > Default Web Site > Add HTTPS Binding
# Port: 443, Certificate: LocalWebServer
```

**Step 3: Test HTTPS Connection**
```powershell
# Test with curl
curl -k -v https://localhost:443

# Or in browser
# https://localhost:443 (accept certificate warning)
```

**Step 4: Verify Certificate**
```powershell
$webCert = Get-Item Cert:\LocalMachine\My\$thumbprint
$webCert | Format-List Subject, Issuer, NotBefore, NotAfter
```

**Step 5: Compare HTTP vs HTTPS**
```powershell
# HTTP request
curl -I http://localhost:80

# HTTPS request
curl -kI https://localhost:443

# Check Dev Tools in browser (F12)
# Network tab shows HTTPS indicator and TLS version
```

**Expected Results:**
- HTTPS works (with certificate warning for self-signed)
- Browser shows security icon but certificate error
- Network traffic encrypted (can't read in plain text)
- Both HTTP and HTTPS respond

**Verification Checklist:**
- ✓ Certificate created and stored
- ✓ IIS binding configured
- ✓ Port 443 listening (netstat -an | findstr 443)
- ✓ HTTPS requests work (-k flag suppresses warning)
- ✓ Browser shows padlock icon
- ✓ Certificate details accessible
- ✓ TLS version visible in curl output

**This demonstrates:**
- Certificate creation and installation
- Web server HTTPS configuration
- Client connection to HTTPS server
- Certificate validation process
- Real-world HTTPS setup

