---
title: "Implementing SSL from Scratch with OpenSSL"
description: "Step-by-step: create a Root CA, Intermediate CA, issue certificates, export PFX/PKCS#12, generate CSRs, and validate authenticity with common OpenSSL queries."
pubDate: "Jun 15 2019"
tags:
  - english
  - security
  - ssl
  - pki
  - openssl
---

This is a practical, command-heavy guide to stand up a simple PKI using **OpenSSL**:

- Install the tools (OpenSSL)  
- Create a **Root CA**  
- Create an **Intermediate CA**  
- Issue server/client certificates  
- Generate artifacts (**CSR**, **CRT**, **KEY**, **PKCS#12 / PFX**)  
- Validate trust, authenticity, and certificate chains  

> Assumption: Linux shell, OpenSSL 1.1.x (typical in 2019-era distros). Commands still work today with minor path/flag differences.

---

## 1) Install / Download OpenSSL

On Debian/Ubuntu:

```bash
sudo apt update
sudo apt install -y openssl ca-certificates
openssl version
```

On RHEL/CentOS:

```bash
sudo yum install -y openssl ca-certificates
openssl version
```

---

## 2) Folder Structure (Recommended)

Create a clean workspace:

```bash
mkdir -p ~/pki/{root,intermediate,certs,private,csr,crl,newcerts}
cd ~/pki
```

For a more classic OpenSSL CA layout:

```bash
mkdir -p root/{certs,crl,newcerts,private}
mkdir -p intermediate/{certs,crl,csr,newcerts,private}
chmod 700 root/private intermediate/private

touch root/index.txt intermediate/index.txt
echo 1000 > root/serial
echo 1000 > intermediate/serial
```

---

## 3) Create the Root CA (Offline CA)

### 3.1 Generate Root CA private key

```bash
openssl genrsa -out root/private/rootCA.key.pem 4096
chmod 400 root/private/rootCA.key.pem
```

### 3.2 Create Root CA certificate (self-signed)

```bash
openssl req -x509 -new -nodes   -key root/private/rootCA.key.pem   -sha256 -days 3650   -out root/certs/rootCA.cert.pem   -subj "/C=CL/ST=Santiago/L=Santiago/O=ExampleOrg/OU=Security/CN=Example Root CA"
```

Check it:

```bash
openssl x509 -in root/certs/rootCA.cert.pem -noout -text
```

---

## 4) Create the Intermediate CA (Online CA)

The intermediate CA signs server/client certs. The root CA signs only the intermediate.

### 4.1 Generate intermediate private key

```bash
openssl genrsa -out intermediate/private/intermediate.key.pem 4096
chmod 400 intermediate/private/intermediate.key.pem
```

### 4.2 Create intermediate CSR

```bash
openssl req -new   -key intermediate/private/intermediate.key.pem   -out intermediate/csr/intermediate.csr.pem   -subj "/C=CL/ST=Santiago/L=Santiago/O=ExampleOrg/OU=Security/CN=Example Intermediate CA"
```

### 4.3 Sign the intermediate CSR with the Root CA

> This is a simplified signing process (no custom openssl.cnf). Good for labs and internal setups.

```bash
openssl x509 -req   -in intermediate/csr/intermediate.csr.pem   -CA root/certs/rootCA.cert.pem   -CAkey root/private/rootCA.key.pem   -CAcreateserial   -out intermediate/certs/intermediate.cert.pem   -days 1825 -sha256
```

Verify intermediate is signed by root:

```bash
openssl verify -CAfile root/certs/rootCA.cert.pem intermediate/certs/intermediate.cert.pem
```

---

## 5) Build the CA Chain Bundle

Create a chain file (intermediate + root). Many servers and clients expect this.

```bash
cat intermediate/certs/intermediate.cert.pem root/certs/rootCA.cert.pem > intermediate/certs/ca-chain.cert.pem
```

---

## 6) Create and Sign a Server Certificate

### 6.1 Generate server key

```bash
openssl genrsa -out private/server.key.pem 2048
chmod 400 private/server.key.pem
```

### 6.2 Create server CSR

Use CN + SAN (modern TLS expects SAN). If you don’t provide SAN, some clients will reject the cert.

Create a config file `csr-server.conf`:

```bash
cat > csr-server.conf <<'EOF'
[req]
default_bits       = 2048
prompt             = no
default_md         = sha256
req_extensions     = req_ext
distinguished_name = dn

[dn]
C  = CL
ST = Santiago
L  = Santiago
O  = ExampleOrg
OU = Platform
CN = api.example.internal

[req_ext]
subjectAltName = @alt_names

[alt_names]
DNS.1 = api.example.internal
DNS.2 = api
IP.1  = 10.0.0.10
EOF
```

Generate CSR:

```bash
openssl req -new   -key private/server.key.pem   -out csr/server.csr.pem   -config csr-server.conf
```

### 6.3 Sign server CSR using Intermediate CA

```bash
openssl x509 -req   -in csr/server.csr.pem   -CA intermediate/certs/intermediate.cert.pem   -CAkey intermediate/private/intermediate.key.pem   -CAcreateserial   -out certs/server.cert.pem   -days 825 -sha256   -extfile csr-server.conf -extensions req_ext
```

Check server cert:

```bash
openssl x509 -in certs/server.cert.pem -noout -text | sed -n '1,80p'
```

Validate chain:

```bash
openssl verify -CAfile intermediate/certs/ca-chain.cert.pem certs/server.cert.pem
```

---

## 7) Create and Sign a Client Certificate (mTLS)

### 7.1 Generate client key

```bash
openssl genrsa -out private/client.key.pem 2048
chmod 400 private/client.key.pem
```

### 7.2 Create client CSR

```bash
openssl req -new   -key private/client.key.pem   -out csr/client.csr.pem   -subj "/C=CL/ST=Santiago/L=Santiago/O=ExampleOrg/OU=Clients/CN=client01"
```

### 7.3 Sign client CSR

```bash
openssl x509 -req   -in csr/client.csr.pem   -CA intermediate/certs/intermediate.cert.pem   -CAkey intermediate/private/intermediate.key.pem   -CAcreateserial   -out certs/client.cert.pem   -days 825 -sha256
```

Validate chain:

```bash
openssl verify -CAfile intermediate/certs/ca-chain.cert.pem certs/client.cert.pem
```

---

## 8) Generate Different Certificate Formats

### 8.1 Create PKCS#12 / PFX (.p12) for server

Useful for Java keystores import, IIS, some clients, etc.

```bash
openssl pkcs12 -export   -inkey private/server.key.pem   -in certs/server.cert.pem   -certfile intermediate/certs/ca-chain.cert.pem   -out certs/server.p12
```

### 8.2 Create PKCS#12 / PFX for client

```bash
openssl pkcs12 -export   -inkey private/client.key.pem   -in certs/client.cert.pem   -certfile intermediate/certs/ca-chain.cert.pem   -out certs/client.p12
```

### 8.3 Inspect PKCS#12 content

```bash
openssl pkcs12 -in certs/server.p12 -info -noout
```

### 8.4 Generate CSR (already shown)

CSR is the artifact you send to a CA (internal/external). It contains public key + subject + optional extensions.

To inspect a CSR:

```bash
openssl req -in csr/server.csr.pem -noout -text
```

---

## 9) Common Validation Queries (Authenticity / Trust)

### 9.1 Validate a certificate chain

```bash
openssl verify -CAfile intermediate/certs/ca-chain.cert.pem certs/server.cert.pem
```

### 9.2 Confirm issuer / subject

```bash
openssl x509 -in certs/server.cert.pem -noout -issuer -subject
```

### 9.3 Print SHA256 fingerprint

```bash
openssl x509 -in certs/server.cert.pem -noout -fingerprint -sha256
```

### 9.4 Verify private key matches certificate (modulus check)

```bash
openssl rsa  -in private/server.key.pem -noout -modulus | openssl md5
openssl x509 -in certs/server.cert.pem -noout -modulus | openssl md5
```

The hashes should match.

### 9.5 Check certificate expiration

```bash
openssl x509 -in certs/server.cert.pem -noout -enddate -startdate
```

### 9.6 Validate SANs

```bash
openssl x509 -in certs/server.cert.pem -noout -text | grep -A2 "Subject Alternative Name"
```

### 9.7 TLS handshake validation (remote endpoint)

```bash
openssl s_client -connect api.example.internal:443 -servername api.example.internal -showcerts
```

To validate trust using your chain:

```bash
openssl s_client -connect api.example.internal:443 -servername api.example.internal   -CAfile intermediate/certs/ca-chain.cert.pem
```

### 9.8 mTLS handshake test

```bash
openssl s_client -connect api.example.internal:443 -servername api.example.internal   -CAfile intermediate/certs/ca-chain.cert.pem   -cert certs/client.cert.pem   -key private/client.key.pem
```

---

## 10) Operational Notes (Practical)

- Keep the **Root CA offline** (ideally on a separate machine or encrypted storage).
- Rotate intermediate CA on a schedule (and document the process).
- Use **SANs** for server certs (CN-only is not enough for modern TLS).
- For production PKI, use proper `openssl.cnf` with extensions:
  - `basicConstraints`  
  - `keyUsage`  
  - `extendedKeyUsage`  
  - `authorityKeyIdentifier` / `subjectKeyIdentifier`  
- Track issued certs and revocations (CRL / OCSP) if you need real governance.

---

## Appendix: Minimal Cleanup / Re-run

If you need to wipe and regenerate everything (lab only):

```bash
rm -rf ~/pki
```
