# Week 6: Deleted Files on a Terminated Employee's Laptop
**Course:** CPSC 4584 | Special Topics in Information Security  
**Date:** October 5, 2026  
**Analyst:** Nfumu Mpiana  
**Case ID:** FOR-2026-1005-001

---

## Incident Summary

A terminated employee's laptop was received for forensic examination after an approximately 43-hour gap in documented custody. During forensic examination, five deleted files were recovered, including work-related spreadsheets, SQL, and document artifacts, along with a partial artifact and a personal image.

---

## Chain of Custody Status

**Gap Period:** Oct 3, 2:47 PM to Oct 5, 9:30 AM (approximately 43 hours)  
**Gap Significance:** The gap creates uncertainty about who had access to the device and whether it was accessed or changed during that period. It does not automatically make the evidence unusable, but it is an important limitation that must be documented and considered when evaluating the evidence.  
**Documented By:** Analyst on receipt of device

---

## Recovered File Evidence Classification

| Filename | Type | Evidentiary Value | Finding |
|----------|------|-------------------|---------|
| insurance_export_final.xlsx | Spreadsheet | High | This file may contain sensitive insurance-related information and could help determine what data was stored or handled on the laptop. Its contents and metadata should be examined further. |
| mhs_billing_schema_db.sql | SQL script | High | This file appears related to a billing database and may show database structures, tables, or other work-related information. It could help establish the employee's access or work activity. |
| vendor_contact_list_external.docx | Document | Moderate | This document may contain business contact information and could help identify external parties or business activity. Its contents, metadata, and creation or modification history should be reviewed. |
| __tmp_8f2a.dat | Partial artifact | Limited | The partial artifact may contain useful recovered data, but its incomplete nature makes interpretation more difficult. Additional forensic analysis is needed to determine its original file type and meaning. |
| personal_vacation_2025.jpg | Image | Low | The image appears personal and may have limited relevance to the incident. It should still be preserved and documented, but its evidentiary value depends on whether it can be connected to the investigation. |

---

## Metadata Inspection Commands Used

| Command | Purpose |
|---------|---------|
| `stat .bashrc` | Inspected file metadata, including access, modification, and metadata-change timestamps. A birth timestamp was also noted separately if the system provided one. |
| `ls -lai` | Displayed directory contents with inode numbers, file permissions, ownership, and other metadata useful for correlating filesystem records. |
| `xxd .bashrc \| head -3` | Examined the first bytes of the file to identify its file-signature pattern and determine what type of content it may contain. |
| `find . -newer .bashrc -type f` | Performed a time-based search for regular files newer than `.bashrc`, helping identify files that may fall within a related activity period. |

---

## Escalation Summary

The investigation confirmed the recovery of five deleted files from the terminated employee's laptop. Several recovered files appear potentially relevant to Maplewood work activity, including an insurance spreadsheet, billing SQL script, and vendor contact document. A forensic image and SHA-256 verification should be used to preserve and verify evidence integrity.

The approximately 43-hour custody gap is the primary evidence limitation because access to the laptop during that period is not fully documented. The recovered files and their timestamps can help establish a timeline, but timestamps alone cannot identify who accessed or modified a file or explain why an action occurred.

Further review should determine whether the recovered files contain protected, confidential, or otherwise sensitive information and whether their activity relates to the employee's termination or the underlying incident. Any decisions involving employee conduct, legal action, disclosure, or access to sensitive records should be referred to authorized legal, HR, security management, or other appropriate personnel.

---
*CPSC 4584 | Governors State University | Fall 2026*
