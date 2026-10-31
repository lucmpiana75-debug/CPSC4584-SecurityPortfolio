# Week 9: SQL Injection Attempt on Appointment Scheduling System

**Course:** CPSC 4584 | Special Topics in Information Security  
**Date:** October 26, 2026  
**Analyst:** Nfumu Mpiana  
**Case ID:** SQLI-2026-1024-001

---

## Incident Summary

Between 2:14 AM and 3:01 AM on October 24, 2026, six search values were submitted to the Maplewood appointment lookup field from one source IP address. The values progressed from a normal name to SQL injection probes and a targeted attempt to select patient information; the access logs do not prove that the database returned any records.

## SQL Injection Log Analysis

| Entry | Time | Decoded Input | Type | Severity |
|-------|------|---------------|------|----------|
| 1 | 02:14:07 | Johnson | Benign Input | Info |
| 2 | 02:14:52 | ' | Quote Probe | Medium |
| 3 | 02:18:31 | Smith' -- | Comment Syntax Test | High |
| 4 | 02:23:14 | ' OR '1'='1 | Tautology Pattern | High |
| 5 | 02:48:39 | ' UNION SELECT NULL,NULL,NULL -- | UNION Injection Attempt | Critical |
| 6 | 03:01:22 | ' UNION SELECT patient_name,date_of_birth,ssn FROM patients -- | Targeted PHI Extraction Attempt | Critical |

## Attack Progression Summary

**Entry 1:** `Johnson` looks like a normal last-name search and gives a baseline for comparison.

**Entry 2:** A single quote may test whether the search value is inserted into a SQL string and whether the application responds with a syntax error or other change.

**Entry 3:** `Smith' --` may try to close a quoted value and use the SQL comment marker to make the database ignore the rest of a query.

**Entry 4:** `' OR '1'='1` contains a condition that is always true. If interpreted as SQL, it could change a search condition and return more records than intended.

**Entry 5:** `UNION SELECT` with three `NULL` values may test whether a second query result with three columns can be combined with the original result.

**Entry 6:** The names `patient_name`, `date_of_birth`, and `ssn` suggest possible knowledge of the patient table's fields and an attempt to retrieve sensitive information. The names could also be guesses or come from another source. The access log does not confirm that the database accepted the query or returned any patient records.

## Remediation Summary

**Primary application control:** Replace direct SQL string construction with PHP PDO prepared statements and bound parameters. This keeps user input separate from SQL instructions, so a search value is treated as data rather than executable SQL.

**Input handling:** Validate that search values fit the expected format and length. Removing quote characters alone is not a reliable defense because other forms of input may still cause problems, and filtering can interfere with legitimate names. Validation should support parameterization, not replace it.

**Database permissions:** Apply least privilege. The application account should have only the permissions and table access needed for its job. In particular, access to patient information should be restricted unless the appointment lookup genuinely requires it.

**Detection and monitoring:** Add database and application logging with alerts for unusual query patterns, and consider a Web Application Firewall as an additional layer. A WAF can help detect or block suspicious requests, but it does not replace fixing the unsafe query.

## Escalation Summary

The brief confirms that the PHP search field passes user input directly into SQL without parameterization, and that the application database account can read both the appointments and patients tables. Six inputs were logged, progressing from a normal name to quote, comment, tautology, and UNION patterns, ending with an attempt that names patient fields. These facts establish attempted activity and a vulnerable query construction, but they do not establish that patient data was returned or disclosed. The DBA team should preserve and review database query or audit logs and correlate them with application errors, response records, and other server-side evidence to determine what executed and what results were returned. The application team should fix the query and review permissions. Privacy and legal teams can assess disclosure implications once the evidence clarifies whether data was accessed or disclosed.

## Query Parameter Inspection Log

These exercises used the approved `httpbin.org` training endpoint. They did not test the Maplewood application or run a database query.

### Entry 1 — Observe a legitimate search parameter

**Command:**

```bash
curl -v "https://httpbin.org/get?last_name=Johnson" 2>&1 | grep -E "GET|last_name"
```

**Terminal output:**

```text
> GET /get?last_name=Johnson HTTP/2
    "last_name": "Johnson"
  "method": "GET",
  "url": "https://httpbin.org/get?last_name=Johnson"
```

**Analyst observation:** The `last_name` parameter appears in the URL after the `?`, so it is part of the query string. A request body is separate from the URL. A proxy or server access log may record the request URL and its parameters, similar to the submitted values in the Maplewood logs.

### Entry 2 — Observe what the server receives

**Command:**

```bash
curl -s "https://httpbin.org/get?last_name=Johnson" | python3 -m json.tool | grep last_name
```

**Terminal output:**

```text
       "last_name": "Johnson"
    "url": "https://httpbin.org/get?last_name=Johnson"
```

**Analyst observation:** The endpoint received `Johnson` as the value of `last_name`. Entry 1 showed the value in the request URL, while Entry 2 shows it in the server's JSON response. A parameterized database query would treat the value as data, not executable SQL syntax.

### Entry 3 — Observe a tautology string as received by the server

**Command:**

```bash
curl -s "https://httpbin.org/get?last_name=%27+OR+%271%27%3D%271" | python3 -m json.tool | grep last_name
```

**Terminal output:**

```text
        "last_name": "' OR '1'='1"
    "url": "https://httpbin.org/get?last_name=%27+OR+%271%27%3D%271"
```

**Analyst observation:** The endpoint decoded the URL-encoded value as `' OR '1'='1`. This string could affect a query if an application joins untrusted input directly into SQL. With a parameterized query, it remains a data value. This exercise sent the string only to the training endpoint; no database query was run.

### Entry 4 — Generate URL-encoded parameter values locally

**Command:**

```bash
python3 - <<'PY'
import urllib.parse
strings = ["Johnson", "'", "Smith' --", "' OR '1'='1"]
for s in strings:
    print("last_name=" + urllib.parse.quote(s))
PY
```

**Terminal output:**

```text
last_name=Johnson
last_name=%27
last_name=Smith%27%20--
last_name=%27%20OR%20%271%27%3D%271
```

**Analyst observation:** URL encoding represents the single quote as `%27` and spaces as `%20`. Encoded values can look different from their readable form, so analysts need to recognize and decode them when reviewing web logs. Encoding changes how a value is represented in a URL; it does not make unsafe SQL construction safe.

---

*CPSC 4584 | Governors State University | Fall 2026*
