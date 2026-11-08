# Week 10: Suspicious Executable Running on a Server
**Course:** CPSC 4584 | Special Topics in Information Security  
**Date:** November 2, 2026  
**Analyst:** Nfumun Mpiana  
**Case ID:** EXEC-2026-1030-001

---

## Incident Summary

An unauthorized `mhs_monitor` process has been running for at least 72 hours on Maplewood Health System’s production EHR application server. Its repeated HTTPS connections to `185.220.101.47` match the Week 7 destination indicator, warranting prompt incident response review.

---

## Static Analysis Findings

| # | Tool | Output Line | Finding | Severity |
|---|------|-------------|---------|----------|
| 1 | `strings` | `185.220.101.47` | The IP is embedded in the binary and matches the destination observed in server network records. The string suggests a possible destination; it does not prove the binary contacted it. | High — the match is corroborated by repeated HTTPS connections in the host records. |
| 2 | `strings` | `/tmp/.mhs_session` | A dot-prefixed temporary path is embedded in the binary. It suggests possible access to a hidden file, but does not prove this binary created or used it. | Medium — investigate and preserve the path and related file telemetry. |
| 3 | `strings` | `Mozilla/5.0 (compatible; MHSMonitor/1.0)` | A browser-like user-agent string is present and could be used in an HTTP request. Its presence does not prove a request was sent. | Medium — it is a useful value to search for in proxy and network logs. |
| 4 | `readelf -d`; `strings` | `[NEEDED] libcurl.so.4`; `curl_easy_perform` | The ELF binary depends on libcurl and contains a related request symbol, suggesting HTTP capability. These artifacts do not establish that a request occurred or reveal its contents. | Medium — correlate the capability indicators with observed network activity. |

The ELF output identifies a 64-bit, dynamically linked, stripped executable. The four findings are static artifacts and inferred capabilities, not proof of runtime behavior or intent. Their combination is more useful than any one string or library reference because the destination, file path, user-agent, and HTTP-related symbols can be compared with independent host and network telemetry.

---

## Process-Level IOCs

| IOC | Observed Value | Significance |
|-----|----------------|--------------|
| Not in inventory | No package-manager entry; no approved software inventory entry | The process is unauthorized and needs investigation. This fact alone does not establish who installed it or why. |
| Parent process | `init` (PID 1); no documented service entry | This is the observed parent, but does not by itself establish persistence, a legitimate service, or how the process was started. |
| Repeated outbound connection | `185.220.101.47` over HTTPS every 4 minutes | The regular interval is beacon-like and the destination matches the Week 7 indicator. Network records show connections, but do not alone establish what data was sent or received. |
| Dot-prefixed file | `/tmp/.mhs_session` | The hidden-looking path is associated with a reported local file write and should be checked and preserved. The static string alone does not attribute file creation to the binary. |

---

## Dynamic Analysis Assessment

Running the unknown binary on a test server with normal outbound access is not appropriate. Its behavior is unknown; it could communicate with external systems, retrieve additional content, or reveal that it is being analyzed. A connected test server could also expose other systems to unanticipated effects.

Authorized malware-analysis personnel should use an isolated, instrumented sandbox with a disposable snapshot. The environment should capture process, file, and network activity and provide controlled or simulated network services while blocking uncontrolled communication with real external systems. The sample should not be run on the production EHR server or an ordinary networked test system.

---

## Escalation Summary

Escalate this as a high-priority incident for Maplewood’s incident response team. `mhs_monitor` is unauthorized, has run for at least 72 hours on the production EHR application server, and has no documented service entry. The strongest correlation is between the binary’s embedded `185.220.101.47` string and the server’s repeated HTTPS connections to that same Week 7 destination. The libcurl reference, user-agent string, and `/tmp/.mhs_session` path add investigative leads, but static analysis does not confirm what the process did.

Before containment, preserve the binary and its cryptographic hash, relevant process and network telemetry, timestamps, server and proxy logs, and available evidence associated with `/tmp/.mhs_session`, following evidence-handling procedures. The incident response team should assess scope and decide on containment and controlled analysis. Tier 1 documentation records and correlates evidence; authorized incident response personnel make and approve production containment decisions. No production change is prescribed here.

---
*CPSC 4584 | Governors State University | Fall 2026*

