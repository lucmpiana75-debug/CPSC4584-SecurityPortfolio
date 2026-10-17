# Week 7: Network Traffic Anomaly During Night Shift
**Course:** CPSC 4584 | Special Topics in Information Security
**Date:** October 12, 2026
**Analyst:** Nfumu Mpiana
**Case ID:** NET-2026-1012-001 (related: FOR-2026-1005-001)
---

## Incident Summary

Webb's laptop made an unusual outbound transfer to 185.220.101.47, alongside repeated small connections that may indicate beaconing. The transfer occurred near the modification and deletion of `insurance_export_final.xlsx` in the Week 6 disk evidence.

---

## Network Flow Analysis

| Flow | Source | Destination | Classification | Finding |
|------|--------|-------------|----------------|---------|
| 1 | Webb laptop | 185.220.101.47 | Suspected data exfiltration | A 2,457,814-byte HTTPS transfer occurred between 11:48 PM and 11:50 PM. The encrypted flow does not identify the data transferred. |
| 2 | Webb laptop | 185.220.101.47 | Possible C2 beaconing | Repeated 312-byte connections occurred about every four minutes from 11:15 PM on October 3 until 6:42 AM on October 4. This supports investigation but does not prove C2. |
| 3 | EHR prod server | AWS | Expected/baseline traffic | The traffic matches the scheduled nightly backup to the EHR vendor's AWS-hosted service. |
| 4 | Radiology WS | 8.8.8.8 | DNS anomaly | The workstation made 23 DNS queries to a public resolver in a short period. Compare this activity with the approved DNS configuration and baseline. |
| 5 | Billing WS | 203.0.113.99 | Possible C2 beaconing | Nine similarly sized HTTP connections occurred about every four minutes to an unapproved destination. This pattern may indicate beaconing but does not prove C2. |

---

## Disk and Network Evidence Correlation

**Disk (Week 6):** `insurance_export_final.xlsx` was modified at 11:47 PM and deleted at 11:52 PM on October 3.

**Network (Week 7):** Webb's laptop transferred 2,457,814 bytes to 185.220.101.47 between 11:48 PM and 11:50 PM. Repeated small connections began at 11:15 PM.

**Correlation:** The transfer occurred between the spreadsheet's modification and deletion, supporting a possible modify-transfer-delete sequence.

**Limitation:** Encrypted flow data shows timing, endpoints, and byte counts, but cannot confirm that the spreadsheet was transferred. Additional endpoint, packet, or application evidence is needed.

---

## Network Artifact Inspection Commands Used

| Command | Purpose |
|---------|---------|
| `curl -I https://example.com` | Returned HTTP/2 200 and headers showing HTML content, Cloudflare, cache status, and support for GET and HEAD. |
| `curl -v --max-time 5` | Showed the connection IP, TLS 1.3 handshake, negotiated cipher, and HTTP/2 negotiation. |
| `curl -s -o /tmp/response.html + xxd` | The first bytes showed `<!doctype html>` and HTML document content. |
| `curl -s -o /dev/null -w` | Reported HTTP 200, 577 bytes, about 0.116 seconds, and remote IP `104.20.23.154`. These metrics relate to status, byte count, duration, and endpoint information in network flow analysis. |

---

## Escalation Summary

The logs show a large outbound transfer, repeated connections, unusual DNS activity, and repeated HTTP connections from the billing workstation. The transfer timing overlaps with the spreadsheet's modification and deletion, but does not prove the file was sent. Preserve evidence and escalate for endpoint, DNS, and packet review. Attribution and containment require further evidence and authorized review.

---
*CPSC 4584 | Governors State University | Fall 2026*
