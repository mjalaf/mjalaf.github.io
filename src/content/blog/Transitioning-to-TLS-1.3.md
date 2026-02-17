---
title: 'Transitioning to TLS 1.3: Improving Performance and Security in Enterprise Environments'
description: 'Why the final approval of TLS 1.3 is a game-changer for latency and why you should start your migration today.'
pubDate: 'July 16 2019'
tags:
  - english
  - security
  - architecture
  - networking
  - azure
---
In the world of enterprise architecture, we have long accepted the "encryption tax"—that unavoidable latency penalty during the initial handshake of a secure connection. However, with the recent finalization and growing browser support for **TLS 1.3 (RFC 8446)**, that trade-off is finally becoming a thing of the past.

As we look at our high-scale distributed systems today, moving to TLS 1.3 isn't just a security patch; it's a performance requirement.

---

## The Core: A Faster, Leaner Handshake

The most significant change from TLS 1.2 is the efficiency of the handshake. In the previous standard, a full handshake required two round-trips (2-RTT) before any encrypted data could be sent. 

TLS 1.3 slashes this to a **single round-trip (1-RTT)**. By including the key exchange and cipher negotiation in the first message, we are seeing a dramatic reduction in connection setup time, especially in mobile networks where latency is high.

Furthermore, for recurring visitors, the **0-RTT (Zero Round-Trip Time Resumption)** feature allows clients to send encrypted data in the very first packet. This is revolutionary for API performance and user experience in global cloud applications.

---

## Architectural Insight: Security by Subtraction

One of the most interesting aspects of TLS 1.3 is that it is more secure because it is simpler. It has effectively removed legacy, broken, and weak cryptographic primitives that were causing us headaches in 1.2. 

Gone are the days of worrying about:
* **SHA-1**
* **MD5**
* **DES and RC4**
* **CBC-mode ciphers** (which were prone to Padding Oracle attacks)

By mandating **Perfect Forward Secrecy (PFS)** as a requirement rather than an option, we are ensuring that if a private key is compromised in the future, past sessions remain encrypted. As architects, this reduces our risk surface significantly without adding configuration complexity.

---

## Key Tech and Implementation

If you are managing workloads on **Azure**, we are starting to see native support rolling out across various services. Here is where you should be focusing your implementation efforts:

1.  **Application Gateway & Azure Front Door:** These are the ideal places to terminate TLS 1.3 to provide that low-latency experience to your global users.
2.  **OpenSSL 1.1.1:** If you are running Linux-based microservices, ensure you have upgraded to this version, as it is the first to provide stable support for 1.3.
3.  **Modern Browsers:** With Chrome 70+ and Firefox 63+ already supporting it, the majority of your end-users are likely ready to benefit from these speed improvements today.

---

## Why start the migration now?

Waiting to adopt TLS 1.3 is essentially leaving performance on the table. The "Security by Default" posture of this new standard aligns perfectly with the **Zero Trust** models we are beginning to implement in our enterprise environments. 

It is rare that we get a protocol update that makes things both significantly faster AND significantly more secure. 