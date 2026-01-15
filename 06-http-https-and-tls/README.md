# 06 - HTTP, HTTPS, and TLS

## What You'll Learn

By the end of this module, you'll be able to:
- Understand the difference between HTTP and HTTPS
- Explain how TLS/SSL secures web communications
- Work with SSL/TLS certificates
- Configure HTTPS on a web server
- Use curl to test HTTP/HTTPS endpoints
- Troubleshoot SSL/TLS certificate issues
- Inspect HTTP headers and responses
- Understand certificate chains and validation

## Prerequisites

- Completion of modules 00-05
- Basic understanding of TCP/IP and DNS
- Windows 10/11 with administrative access (or Linux/Mac equivalent)
- Command-line familiarity
- OpenSSL installed (or Windows equivalent tools)
- Ability to install software

## Key Concepts

### HTTP (HyperText Transfer Protocol)

1. **Stateless Protocol** - Each request is independent
2. **Port 80** - Default HTTP port
3. **Plain Text** - Data not encrypted
4. **Request/Response Model** - Client requests, server responds
5. **Methods** - GET, POST, PUT, DELETE, HEAD, OPTIONS
6. **Status Codes** - 1xx, 2xx, 3xx, 4xx, 5xx

### HTTPS (HyperText Transfer Protocol Secure)

1. **Encrypted Communication** - Uses TLS/SSL
2. **Port 443** - Default HTTPS port
3. **Authentication** - Verifies server identity via certificates
4. **Integrity** - Data cannot be modified in transit
5. **Confidentiality** - Data is encrypted end-to-end

### TLS/SSL (Transport Layer Security / Secure Sockets Layer)

1. **Handshake Process** - Client and server negotiate encryption
2. **Cipher Suites** - Combination of algorithms for encryption
3. **Certificates** - Digital documents proving identity
4. **Public/Private Keys** - Asymmetric encryption
5. **Session Keys** - Symmetric encryption for data transfer
6. **Certificate Chain** - Root → Intermediate → Server certificate

### SSL/TLS Certificates

1. **Self-Signed** - Signed by the owner, not trusted
2. **CA-Signed** - Signed by Certificate Authority, trusted
3. **Wildcard Certificates** - Valid for all subdomains (*.example.com)
4. **SAN Certificates** - Multiple domain names in one cert
5. **Certificate Validation** - Domain name, expiration date, signature

## Hands-on Lab: Setting Up HTTPS with Self-Signed Certificate

### Objective
Create a self-signed SSL certificate, configure a basic HTTPS server, and test the connection.

### Step 1: Generate a Self-Signed Certificate

```powershell
# Create a new self-signed certificate (Windows using PowerShell)
New-SelfSignedCertificate -CertStoreLocation Cert:\LocalMachine\My -DnsName "localhost" -FriendlyName "Test Server" -NotAfter (Get-Date).AddYears(1)
```

**Expected Output:**
```
PSChildName            : ABC123DEF456...
PSDrive                : Cert
PSProvider             : Microsoft.PowerShell.Security\Certificate
PSPath                 : Microsoft.PowerShell.Security\Certificate::LocalMachine\My\ABC123DEF456...
Thumbprint             : ABC123DEF456789...
Subject                : CN=localhost
```

**Alternative (Linux/Mac with OpenSSL):**

```bash
# Generate private key
openssl genrsa -out server.key 2048

# Generate certificate signing request
openssl req -new -key server.key -out server.csr -subj "/CN=localhost"

# Create self-signed certificate
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

**Expected Output:**
```
Generating RSA private key, 2048 bit long modulus (2 primes)
...
Signature ok
subject=CN=localhost
Getting Private key
```

### Step 2: Test HTTPS Connection with curl

```powershell
# Test with curl (allow insecure for self-signed cert)
curl -v -k https://localhost:443
```

**Expected Output:**
```
*   Trying 127.0.0.1:443...
* Connected to localhost (127.0.0.1) port 443 (#0)
* TLS 1.2 connection using TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
* Server certificate:
*  subject: CN=localhost
*  start date: Jan 15, 2024
*  expire date: Jan 15, 2025
*  issuer: CN=localhost
*  SSL certificate verify result: self signed certificate (18), continuing anyway...
```

### Step 3: Inspect Certificate Details

```powershell
# View certificate details (Windows)
$cert = Get-ChildItem Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*localhost*"}
$cert | Format-List Subject, Issuer, Thumbprint, NotBefore, NotAfter
```

**Expected Output:**
```
Subject      : CN=localhost
Issuer       : CN=localhost
Thumbprint   : ABC123DEF456789...
NotBefore    : 1/15/2024 10:30:45 AM
NotAfter     : 1/15/2025 10:30:45 AM
```

**Alternative (OpenSSL):**

```bash
openssl x509 -in server.crt -text -noout
```

**Expected Output:**
```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 12345...
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = localhost
        Subject: CN = localhost
        Validity
            Not Before: Jan 15 10:30:45 2024 GMT
            Not After : Jan 15 10:30:45 2025 GMT
        Public-Key: (2048 bit)
```

### Step 4: Test HTTP vs HTTPS

```powershell
# Test HTTP (unencrypted)
curl -v http://httpbin.org/get

# Test HTTPS (encrypted)
curl -v https://httpbin.org/get
```

**Expected Output Difference:**
- HTTP: No encryption visible, plain text headers
- HTTPS: TLS handshake visible, encrypted payload

### Step 5: Check Certificate Chain

```powershell
# Download and inspect Google's certificate
curl -v https://google.com 2>&1 | findstr "subject:"
```

**Expected Output:**
```
* subject: CN=google.com
```

**Alternative (OpenSSL):**

```bash
openssl s_client -connect google.com:443 -showcerts
```

**Expected Output:**
```
Certificate chain
 0 s:CN = google.com
   i:C = US, O = Google Trust Services LLC, CN = GTS CA 1D4
 1 s:C = US, O = Google Trust Services LLC, CN = GTS CA 1D4
   i:C = US, R = US, O = Google Trust Services LLC, CN = GTS Root R4
 2 s:C = US, R = US, O = Google Trust Services LLC, CN = GTS Root R4
   i:C = US, O = GlobalSign nv-sa, CN = GlobalSign Root CA
```

## Validation

After completing this lab, validate your understanding by:

1. **Certificate Inspection** - View your self-signed certificate and identify:
   - Subject name
   - Issuer name
   - Expiration date
   - Thumbprint/fingerprint

2. **TLS Connection** - Connect to https://httpbin.org/get and confirm:
   - Connection is established securely
   - Certificate is valid
   - Response contains HTTP headers

3. **Protocol Difference** - Compare:
   - HTTP headers in plain text
   - HTTPS headers after TLS encryption

4. **Certificate Error** - Intentionally connect to wrong hostname and observe:
   - Certificate verification failure
   - Subject name mismatch error

## Cleanup

```powershell
# Remove self-signed certificate (Windows)
$cert = Get-ChildItem Cert:\LocalMachine\My | Where-Object {$_.FriendlyName -eq "Test Server"}
if ($cert) {
    Remove-Item -Path "Cert:\LocalMachine\My\$($cert.Thumbprint)"
}

# Remove certificate files (Linux/Mac)
rm server.key server.csr server.crt
```

## Common Mistakes

1. **Ignoring Certificate Warnings** - Self-signed certs are untrusted; always verify
2. **Using HTTP for Sensitive Data** - Never send passwords or tokens over HTTP
3. **Expired Certificates** - Regularly monitor and renew certificates
4. **Wrong Certificate for Domain** - Cert subject must match domain name
5. **Weak Cipher Suites** - Use modern, strong algorithms (TLS 1.2+)
6. **Mixing HTTP and HTTPS** - Use HTTPS consistently
7. **Not Checking Certificate Chain** - Verify full chain of trust
8. **Self-Signed in Production** - Only use CA-signed certs in production

## Troubleshooting

### "Certificate Verify Failed" Error
**Cause:** Server certificate is self-signed or invalid
**Solution:** 
- For testing: Use `curl -k` to skip verification
- For production: Use valid CA-signed certificate
- Inspect cert with: `openssl x509 -in cert.pem -text -noout`

### "Subject Name Does Not Match" Error
**Cause:** Certificate domain doesn't match requested hostname
**Solution:**
- Verify certificate subject: `openssl x509 -in cert.pem -noout -subject`
- Use correct hostname or wildcard certificate
- Add SANs (Subject Alternative Names) for multiple domains

### "Certificate Has Expired" Error
**Cause:** Certificate expiration date has passed
**Solution:**
- Check expiration: `openssl x509 -in cert.pem -noout -dates`
- Generate new certificate
- Set calendar reminder for renewal (usually 30 days before expiry)

### "Unable to Connect on Port 443" Error
**Cause:** HTTPS server not running or firewall blocking
**Solution:**
- Check if service is running: `netstat -an | findstr :443`
- Verify firewall rules: `Get-NetFirewallRule -Name *443*`
- Ensure HTTPS binding is active on web server

### "TLS Version Not Supported" Error
**Cause:** Server or client doesn't support negotiated TLS version
**Solution:**
- Check client TLS version: `[System.Net.ServicePointManager]::SecurityProtocol`
- Update to modern TLS: `[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12`
- Server must support same or higher TLS version

## Next Steps

1. **Module 07** - Learn about Load Balancing and Proxies (HTTPS termination, certificate distribution)
2. **Module 08** - Study Firewalls and Security Groups (port 443 rules, traffic filtering)
3. **Module 12** - Deep dive into Network Security and Zero Trust (certificate pinning, mTLS)
4. **Practice:** Set up HTTPS on a local web server or cloud instance
5. **Real-World:** Explore certificate management with Let's Encrypt or AWS Certificate Manager
