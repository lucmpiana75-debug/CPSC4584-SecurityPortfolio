# Week 3: Unauthorized USB Drive in Radiology
**Course:** CPSC 4584 | Special Topics in Information Security  
**Date:** September 14, 2026  
**Analyst:** Nfumu Mpiana  
**Incident ID:** INC-2026-0914-001  

---

## Incident Summary

An unmarked USB drive was found plugged into workstation MHS-RAD-WS-03 in the restricted Radiology imaging suite. The drive was removed, secured, and transferred to the SOC without being opened or connected to another computer.

---

## Chain of Custody

Chain of custody shows who handled the USB, when, and what they did. The recorded transfers show that the drive was protected, not accessed, and preserved for forensic examination.

---

## Key Encoding Finding

**String Found:** Y3VybCAtcyAtbyAvZGV2L251bGw=  
**Encoding Type:** Base64  
**Decoded Content:** curl -s -o /dev/null  
**Significance:** The decoded command can make a web request while hiding the output. However, this fragment alone does not prove malicious activity or show whether the command was actually executed.

---

## Terminal Commands Used

| Command | Purpose |
|---------|---------|
| `echo "unauthorized access" | base64` | Encoded plain text into a Base64 string and showed how data can be represented in another format. |
| `echo "Y3VybCAtcyAtbyAvZGV2L251bGw=" | base64 -d` | Decoded the Base64 string and revealed the curl command fragment. |
| `xxd .bashrc | head -6` | Displayed the file's hexadecimal bytes and text representation for inspection. |
| `strings .bashrc | grep -i "path\|export\|alias"` | Filtered readable strings to focus on paths, exports, and aliases. |

---

## Escalation Recommendation

I would escalate this incident to Tier 2 for deeper investigation. The strongest evidence is the unauthorized USB drive found on a restricted clinical workstation. The drive's contents are still unknown, and it is not known how it got there, whether it was used, or whether any systems or data were affected.

---

*CPSC 4584 | Governors State University | Fall 2026*
