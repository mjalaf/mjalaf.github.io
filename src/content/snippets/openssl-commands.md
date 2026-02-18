---
title: "OpenSSL Commands"
description: "Essential OpenSSL commands for certificate management, key generation, CSR creation, format conversion, and TLS troubleshooting."
category: "Networking"
---

## Certificate Inspection

```bash
# View full certificate details
openssl x509 -in cert.pem -noout -text

# Check expiration dates
openssl x509 -in cert.pem -noout -dates

# Print subject and issuer
openssl x509 -in cert.pem -noout -subject -issuer

# SHA256 fingerprint
openssl x509 -in cert.pem -noout -fingerprint -sha256

# View SANs (Subject Alternative Names)
openssl x509 -in cert.pem -noout -text | grep -A2 "Subject Alternative Name"

# View serial number
openssl x509 -in cert.pem -noout -serial
```

## Key Generation

```bash
# RSA key (4096-bit)
openssl genrsa -out private.key 4096

# RSA key with passphrase
openssl genrsa -aes256 -out private.key 4096

# ECDSA key (P-256 curve)
openssl ecparam -genkey -name prime256v1 -noout -out ec-private.key

# ED25519 key (modern, fast)
openssl genpkey -algorithm ED25519 -out ed25519.key

# Remove passphrase from a key
openssl rsa -in encrypted.key -out decrypted.key
```

## CSR (Certificate Signing Request)

```bash
# Generate key + CSR in one step
openssl req -new -newkey rsa:2048 -nodes \
  -keyout server.key -out server.csr \
  -subj "/C=US/ST=CA/L=SF/O=MyOrg/CN=example.com"

# Generate CSR from existing key
openssl req -new -key server.key -out server.csr

# Generate CSR with SANs (config file)
cat > san.cnf <<'EOF'
[req]
default_bits       = 2048
prompt             = no
default_md         = sha256
req_extensions     = v3_req
distinguished_name = dn

[dn]
CN = example.com

[v3_req]
subjectAltName = @alt_names

[alt_names]
DNS.1 = example.com
DNS.2 = *.example.com
DNS.3 = api.example.com
IP.1  = 10.0.0.1
EOF

openssl req -new -key server.key -out server.csr -config san.cnf

# Inspect a CSR
openssl req -in server.csr -noout -text

# Verify a CSR
openssl req -in server.csr -verify -noout
```

## Self-Signed Certificates

```bash
# Quick self-signed cert (1 year)
openssl req -x509 -newkey rsa:4096 -nodes \
  -keyout key.pem -out cert.pem -sha256 -days 365 \
  -subj "/CN=localhost"

# Self-signed with SANs
openssl req -x509 -newkey rsa:4096 -nodes \
  -keyout key.pem -out cert.pem -sha256 -days 365 \
  -config san.cnf -extensions v3_req

# Self-signed from existing key
openssl req -x509 -key server.key -out cert.pem -sha256 -days 365 \
  -subj "/CN=example.com"
```

## Certificate Signing (CA Operations)

```bash
# Sign a CSR with your CA
openssl x509 -req \
  -in server.csr \
  -CA ca.cert.pem \
  -CAkey ca.key.pem \
  -CAcreateserial \
  -out server.cert.pem \
  -days 825 -sha256

# Sign with SAN extensions
openssl x509 -req \
  -in server.csr \
  -CA ca.cert.pem \
  -CAkey ca.key.pem \
  -CAcreateserial \
  -out server.cert.pem \
  -days 825 -sha256 \
  -extfile san.cnf -extensions v3_req

# Build CA chain bundle
cat intermediate.cert.pem root.cert.pem > ca-chain.cert.pem
```

## Format Conversion

```bash
# PEM to DER
openssl x509 -in cert.pem -outform DER -out cert.der

# DER to PEM
openssl x509 -in cert.der -inform DER -outform PEM -out cert.pem

# PEM to PKCS#12 / PFX
openssl pkcs12 -export \
  -inkey server.key \
  -in server.cert.pem \
  -certfile ca-chain.cert.pem \
  -out server.p12

# PKCS#12 to PEM (extract cert + key)
openssl pkcs12 -in server.p12 -out combined.pem -nodes

# Extract cert only from PKCS#12
openssl pkcs12 -in server.p12 -out cert.pem -nokeys

# Extract key only from PKCS#12
openssl pkcs12 -in server.p12 -out key.pem -nocerts -nodes

# PEM to PKCS#7
openssl crl2pkcs7 -nocrl -certfile cert.pem -out cert.p7b

# Inspect PKCS#12
openssl pkcs12 -in server.p12 -info -noout
```

## Verification & Validation

```bash
# Verify cert against CA chain
openssl verify -CAfile ca-chain.cert.pem server.cert.pem

# Verify key matches certificate (modulus check)
openssl rsa  -in server.key -noout -modulus | openssl md5
openssl x509 -in server.cert.pem -noout -modulus | openssl md5
# Both MD5 hashes must match

# Verify CSR matches key
openssl req -in server.csr -noout -modulus | openssl md5
openssl rsa -in server.key -noout -modulus | openssl md5
```

## TLS Connection Testing

```bash
# Test TLS handshake
openssl s_client -connect example.com:443 -servername example.com

# Show full certificate chain
openssl s_client -connect example.com:443 -servername example.com -showcerts

# Test specific TLS version
openssl s_client -connect example.com:443 -tls1_3
openssl s_client -connect example.com:443 -tls1_2

# Test with custom CA bundle
openssl s_client -connect example.com:443 \
  -servername example.com \
  -CAfile ca-chain.cert.pem

# mTLS test (client certificate)
openssl s_client -connect example.com:443 \
  -servername example.com \
  -cert client.cert.pem \
  -key client.key.pem \
  -CAfile ca-chain.cert.pem

# Check supported ciphers
openssl s_client -connect example.com:443 -cipher 'ECDHE+AESGCM'

# Connection with timing (useful for debugging)
curl -w "\n  DNS: %{time_namelookup}\n  TLS: %{time_appconnect}\n  Total: %{time_total}\n" \
  -o /dev/null -s https://example.com

# Check certificate expiry of a remote host
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null \
  | openssl x509 -noout -dates
```

## Hashing & Encoding

```bash
# SHA256 hash of a file
openssl dgst -sha256 file.txt

# Base64 encode
openssl base64 -in file.bin -out file.b64

# Base64 decode
openssl base64 -d -in file.b64 -out file.bin

# Generate random bytes (hex)
openssl rand -hex 32

# Generate random password (base64)
openssl rand -base64 24
```

## CRL (Certificate Revocation List)

```bash
# Generate a CRL
openssl ca -gencrl -out crl.pem -config openssl.cnf

# Inspect a CRL
openssl crl -in crl.pem -noout -text

# Verify cert with CRL check
openssl verify -crl_check -CAfile ca-chain.cert.pem -CRLfile crl.pem cert.pem
```
