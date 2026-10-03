# Week 5: Weak Password Policy Exposes Billing Portal
**Course:** CPSC 4584 | Special Topics in Information Security  
**Date:** September 28, 2026  
**Analyst:** Nfumu Mpiana  
**Incident ID:** INC-2026-0928-001  

---

## Incident Summary

A credential stuffing attack successfully accessed the billing administrator account using the compromised password `Maplewood2024!`, which had appeared in a leaked credential set from an unrelated breach. The attacker accessed insurance records for 3,247 unique patients, including names, dates of birth, insurance policy numbers, and claim histories.

---

## TLS Assessment

**TLS Version:** TLS 1.3  
**Status:** Compliant  
**What TLS Protected:** Data in transit between the client and server and the integrity of the communication channel.  
**What TLS Did Not Protect:** Authentication — the attacker had valid credentials and accessed the system as an authenticated user.

---

## Authentication Controls Gap Analysis

| Control | Required | Status | Finding |
|---------|----------|--------|---------|
| MFA | Maplewood sensitive-account standard | Not Implemented | The billing administrator account only required a password, so the compromised password was enough to authenticate. |
| Failed Attempt Protection | Account-based throttling and alerting | Not Implemented | 847 failed attempts occurred over 72 hours without account-based throttling or a SOC alert. |
| Password Policy | NIST SP 800-63B-4 aligned | Needs Improvement | The portal required 12 characters and complexity rules but did not block known compromised passwords. |
| Compromised Credential Response | Detect and invalidate confirmed compromised authenticators | Not Implemented | Maplewood had no documented process to identify and invalidate the compromised password. |
| Automated Attack Controls | Throttling, bot detection, or adaptive controls as appropriate | Not Implemented | No additional controls slowed or challenged the distributed login attempts from 12 IP addresses. |

---

## OpenSSL Commands Practiced

| Command | Purpose |
|---------|---------|
| `openssl genrsa -out private_key.pem 2048` | Creates a 2048-bit RSA private key and saves it as `private_key.pem`. |
| `openssl rsa -in private_key.pem -pubout -out public_key.pem` | Derives the public key from the private key and saves it as `public_key.pem`. |
| `openssl rsa -in private_key.pem -text -noout \| head -20` | Displays the first 20 lines of the RSA private key's structural details without outputting the key in PEM format. |
| `cat public_key.pem` | Displays the contents of the public key PEM file, including its header, Base64-encoded key material, and footer. |

---

## Analyst Note

The OpenSSL commands used temporary practice keys created in the CyLab Security Academy webshell. They are not Maplewood production keys or incident evidence.
