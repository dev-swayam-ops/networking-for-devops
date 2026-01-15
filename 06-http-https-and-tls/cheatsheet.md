# Cheatsheet: HTTP, HTTPS, and TLS

Quick reference for HTTP/HTTPS concepts, commands, and SSL/TLS operations.

## HTTP Methods and Status Codes

### Common HTTP Methods

| Method | Purpose | Example |
|--------|---------|---------|
| `GET` | Retrieve resource | `curl https://api.example.com/users` |
| `POST` | Create new resource | `curl -X POST -d '{"name":"John"}' https://api.example.com/users` |
| `PUT` | Update existing resource | `curl -X PUT -d '{"name":"Jane"}' https://api.example.com/users/1` |
| `DELETE` | Delete resource | `curl -X DELETE https://api.example.com/users/1` |
| `PATCH` | Partial update | `curl -X PATCH -d '{"name":"Jim"}' https://api.example.com/users/1` |
| `HEAD` | Get headers only | `curl -I https://api.example.com/users` |
| `OPTIONS` | Get allowed methods | `curl -X OPTIONS -v https://api.example.com/users` |

### HTTP Status Code Ranges

| Range | Category | Examples |
|-------|----------|----------|
| 1xx | Informational | 100 Continue, 101 Switching Protocols |
| 2xx | Success | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirect | 301 Moved Permanently, 302 Found, 304 Not Modified |
| 4xx | Client Error | 400 Bad Request, 401 Unauthorized, 404 Not Found, 403 Forbidden |
| 5xx | Server Error | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable |

## curl Commands for Testing

### Basic HTTPS Requests

| Command | Purpose | Example |
|---------|---------|---------|
| `curl <url>` | Simple GET request | `curl https://example.com` |
| `curl -I <url>` | Headers only (HEAD request) | `curl -I https://example.com` |
| `curl -i <url>` | Full response with headers | `curl -i https://example.com` |
| `curl -v <url>` | Verbose (see handshake details) | `curl -v https://example.com` |
| `curl -X <method> <url>` | Specific HTTP method | `curl -X POST https://example.com/api` |
| `curl -d '<data>' <url>` | Send POST data | `curl -d '{"key":"value"}' https://example.com` |
| `curl -H '<header>' <url>` | Add custom header | `curl -H 'Authorization: Bearer token' https://example.com` |

### Certificate and TLS Testing

| Command | Purpose | Example |
|---------|---------|---------|
| `curl -k <url>` | Skip cert verification (testing only!) | `curl -k https://self-signed.test` |
| `curl --cacert <file> <url>` | Use custom CA certificate | `curl --cacert root-ca.pem https://example.com` |
| `curl --cert <file> <url>` | Use client certificate (mTLS) | `curl --cert client.pem https://example.com` |
| `curl --key <file> <url>` | Specify private key file | `curl --key private.key https://example.com` |
| `curl -v https://<url> 2>&1 \| findstr TLS` | Show TLS version | `curl -v https://google.com 2>&1 \| findstr TLS` |
| `curl -v https://<url> 2>&1 \| findstr cipher` | Show cipher suite | `curl -v https://google.com 2>&1 \| findstr cipher` |
| `curl -v https://<url> 2>&1 \| findstr certificate` | Show cert details | `curl -v https://google.com 2>&1 \| findstr certificate` |

## SSL/TLS Certificate Management

### Generate Self-Signed Certificate (OpenSSL)

| Task | Command |
|------|---------|
| Generate private key (2048-bit) | `openssl genrsa -out private.key 2048` |
| Generate private key (4096-bit, more secure) | `openssl genrsa -out private.key 4096` |
| Create certificate signing request (CSR) | `openssl req -new -key private.key -out request.csr -subj "/CN=example.com"` |
| Create self-signed cert (365 days) | `openssl req -x509 -new -key private.key -out cert.pem -days 365 -subj "/CN=example.com"` |
| Create self-signed cert (10 years) | `openssl req -x509 -new -key private.key -out cert.pem -days 3650 -subj "/CN=example.com"` |
| Create with Subject Alternative Names (SANs) | `openssl req -x509 -new -key private.key -out cert.pem -days 365 -subj "/CN=example.com" -addext "subjectAltName=DNS:example.com,DNS:*.example.com"` |

### Inspect and Verify Certificates (OpenSSL)

| Task | Command |
|------|---------|
| View certificate details | `openssl x509 -in cert.pem -text -noout` |
| View certificate subject | `openssl x509 -in cert.pem -noout -subject` |
| View certificate issuer | `openssl x509 -in cert.pem -noout -issuer` |
| View certificate dates | `openssl x509 -in cert.pem -noout -dates` |
| View certificate fingerprint (SHA1) | `openssl x509 -in cert.pem -noout -fingerprint` |
| View certificate fingerprint (SHA256) | `openssl x509 -in cert.pem -noout -fingerprint -sha256` |
| Verify certificate signature | `openssl verify -CAfile root.pem cert.pem` |
| Check if cert matches private key | `openssl pkey -in private.key -pubout -outform PEM \| openssl dgst -sha256` |
| Extract public key from cert | `openssl x509 -in cert.pem -noout -pubkey` |
| Convert PEM to DER format | `openssl x509 -in cert.pem -outform der -out cert.der` |
| Convert DER to PEM format | `openssl x509 -in cert.der -inform der -out cert.pem` |
| View certificate chain | `openssl s_client -connect example.com:443 -showcerts` |

### Sign Certificate with Certificate Authority (OpenSSL)

| Task | Command |
|------|---------|
| Create CA key and cert | `openssl genrsa -out ca-key.pem 4096` |
| | `openssl req -x509 -new -key ca-key.pem -out ca-cert.pem -days 3650 -subj "/CN=MyCA"` |
| Create server CSR | `openssl req -new -key server.key -out server.csr -subj "/CN=server.example.com"` |
| Sign server CSR with CA | `openssl x509 -req -days 365 -in server.csr -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem` |
| Sign with SANs | `openssl x509 -req -days 365 -in server.csr -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extensions v3_san -extfile <(printf "subjectAltName=DNS:example.com,DNS:*.example.com")` |

## Windows Certificate Management (PowerShell)

### Certificate Operations

| Task | Command |
|------|---------|
| Create self-signed cert | `New-SelfSignedCertificate -CertStoreLocation Cert:\LocalMachine\My -DnsName "localhost" -FriendlyName "Test" -NotAfter (Get-Date).AddYears(1)` |
| List all certs in LocalMachine\My | `Get-ChildItem Cert:\LocalMachine\My` |
| Find cert by friendly name | `Get-ChildItem Cert:\LocalMachine\My \| Where-Object {$_.FriendlyName -eq "Test"}` |
| Find cert by subject | `Get-ChildItem Cert:\LocalMachine\My \| Where-Object {$_.Subject -like "*example*"}` |
| View cert details | `$cert = Get-ChildItem Cert:\LocalMachine\My | Where-Object {$_.FriendlyName -eq "Test"}; $cert \| Format-List Subject,Issuer,Thumbprint,NotBefore,NotAfter` |
| Remove certificate | `Get-ChildItem Cert:\LocalMachine\My \| Where-Object {$_.Thumbprint -eq "ABC123..."} \| Remove-Item` |
| Export cert to PEM | `Export-Certificate -Cert (Get-ChildItem Cert:\LocalMachine\My\ABC123...) -FilePath cert.pem -Type CERT` |
| Export cert with key (PFX) | `Export-PfxCertificate -Cert Cert:\LocalMachine\My\ABC123... -FilePath cert.pfx -Password (ConvertTo-SecureString -String "password" -AsPlainText -Force)` |

### TLS Configuration

| Task | Command |
|------|---------|
| Check default TLS version | `[System.Net.ServicePointManager]::SecurityProtocol` |
| Set TLS 1.2 only | `[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12` |
| Set TLS 1.2 and 1.3 | `[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12 -bor [System.Net.SecurityProtocolType]::Tls13` |
| Disable SSL 3.0 (deprecated) | `[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -band -bnot [System.Net.SecurityProtocolType]::Ssl3` |

## Network Monitoring for HTTPS

### Check Port 443 Activity

| Task | Command |
|------|---------|
| Show all port 443 connections | `netstat -an \| findstr :443` |
| Show established port 443 | `netstat -an \| findstr "443.*ESTABLISHED"` |
| Show listening on 443 | `netstat -an \| findstr "443.*LISTENING"` |
| Count active HTTPS connections | `(netstat -an \| findstr :443 \| measure-object).Count` |
| Show HTTPS with process name | `netstat -ab \| findstr :443` |
| Show HTTPS with PID | `netstat -ano \| findstr :443` |

## Common HTTPS Errors and Solutions

### Error Reference

| Error Message | Cause | Quick Fix |
|---------------|-------|-----------|
| `SSL: CERTIFICATE_VERIFY_FAILED` | Self-signed or untrusted cert | Use `curl -k` for testing; get CA-signed for production |
| `Subject name does not match` | Domain mismatch | Use correct hostname or wildcard/SAN cert |
| `Certificate has expired` | Past expiration date | Renew certificate |
| `Certificate is not yet valid` | Future date on cert | Check system time |
| `Unable to get local issuer certificate` | Root CA not trusted | Add CA cert to trust store |
| `Connection refused on port 443` | HTTPS service not running | Start HTTPS service or check firewall |
| `tlsv1 alert unknown ca` | Server doesn't trust client cert | Verify client certificate is signed by trusted CA |
| `TLS version not supported` | Client/server TLS mismatch | Update TLS version on client or server |

## HTTP Headers Quick Reference

### Request Headers

| Header | Purpose | Example |
|--------|---------|---------|
| `Host` | Target server hostname | `Host: example.com` |
| `User-Agent` | Client application info | `User-Agent: curl/7.64.1` |
| `Accept` | Preferred response format | `Accept: application/json` |
| `Authorization` | Authentication credentials | `Authorization: Bearer token123` |
| `Content-Type` | Request body format | `Content-Type: application/json` |
| `Content-Length` | Request body size | `Content-Length: 256` |
| `Cache-Control` | Caching instructions | `Cache-Control: no-cache` |
| `Cookie` | Session/tracking cookies | `Cookie: sessionid=abc123` |

### Response Headers

| Header | Purpose | Example |
|--------|---------|---------|
| `Content-Type` | Response body format | `Content-Type: text/html; charset=UTF-8` |
| `Content-Length` | Response body size | `Content-Length: 5120` |
| `Server` | Web server software | `Server: Apache/2.4.41` |
| `Set-Cookie` | Store cookie on client | `Set-Cookie: sessionid=xyz789; Path=/` |
| `Location` | Redirect URL (3xx responses) | `Location: https://example.com/new-page` |
| `Cache-Control` | Caching instructions | `Cache-Control: max-age=3600, public` |
| `Expires` | Expiration timestamp | `Expires: Wed, 15 Jan 2025 14:30:00 GMT` |
| `Strict-Transport-Security` | Force HTTPS | `Strict-Transport-Security: max-age=31536000` |
| `X-Content-Type-Options` | MIME type sniffing prevention | `X-Content-Type-Options: nosniff` |
| `X-Frame-Options` | Clickjacking protection | `X-Frame-Options: SAMEORIGIN` |
| `X-XSS-Protection` | XSS attack prevention | `X-XSS-Protection: 1; mode=block` |

## SSL/TLS Cipher Suites (Recommended)

### Modern Secure Ciphers (TLS 1.2+)

| Cipher Suite | Protocol | Security Level | Notes |
|--------------|----------|-----------------|-------|
| `ECDHE-RSA-AES128-GCM-SHA256` | TLS 1.2 | ✓ Good | Elliptic Curve + AES |
| `ECDHE-RSA-AES256-GCM-SHA384` | TLS 1.2 | ✓ Excellent | Stronger variant |
| `ECDHE-ECDSA-AES128-GCM-SHA256` | TLS 1.2 | ✓ Good | ECDSA variant |
| `AES128-GCM-SHA256` | TLS 1.2 | ✓ Good | Static DH variant |
| `TLS_AES_128_GCM_SHA256` | TLS 1.3 | ✓ Excellent | TLS 1.3 standard |
| `TLS_AES_256_GCM_SHA384` | TLS 1.3 | ✓ Excellent | TLS 1.3 stronger |
| `TLS_CHACHA20_POLY1305_SHA256` | TLS 1.3 | ✓ Excellent | Mobile-optimized |

### Weak Ciphers (Avoid!)

| Cipher Suite | Protocol | Security Level | Reason |
|--------------|----------|-----------------|--------|
| `RC4` | TLS 1.0-1.2 | ✗ Broken | Known weaknesses |
| `DES` | TLS 1.0-1.2 | ✗ Broken | 56-bit key too small |
| `MD5` | TLS 1.0-1.2 | ✗ Broken | Hash collision attacks |
| `SSLv3` | SSL 3.0 | ✗ Broken | POODLE attack vulnerability |
| `TLS 1.0` | TLS 1.0 | ✗ Weak | Multiple vulnerabilities |
| `TLS 1.1` | TLS 1.1 | ✗ Weak | Deprecated, weak cipher support |

