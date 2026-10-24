# Week 8: Patient Portal Login Page Behaving Strangely

**Course:** CPSC 4584 \| Special Topics in Information Security\
**Date:** October 19, 2026\
**Analyst:** Nfumu Mpiana\
**Case ID:** WEB-2026-1019-001

------------------------------------------------------------------------

## Incident Summary

Patients reported unusual behavior on the Maplewood Health System
patient portal login page. The SOC investigation identified four
potential HTTP security weaknesses: credentials included in a GET
request URL, session cookies missing important security attributes,
missing response security headers, and a redirect that accepts an
external destination. These findings require validation against the
actual portal and escalation to the authorized application security
team.

------------------------------------------------------------------------

## HTTP Vulnerability Findings

  -----------------------------------------------------------------------------
  Finding    Location     Observed Value     Vulnerability          Severity
  ---------- ------------ ------------------ ---------------------- -----------
  1          Login        The incident brief Credential exposure    High
             request      reports that the   through URL            
             method       login request      parameters. URLs may   
                          includes the       be stored in browser   
                          username and       history, server logs,  
                          password in a GET  proxy logs, and other  
                          URL. The separate  monitoring systems.    
                          `example.com`                             
                          header test did                           
                          not inspect the                           
                          patient portal                            
                          login request.                            

  2          Set-Cookie   The incident brief Insecure               High
             header       reports that the   session-cookie         
                          session cookie     configuration. Without 
                          lacks the          `HttpOnly`, scripts    
                          `HttpOnly` and     may be able to access  
                          `Secure`           the cookie; without    
                          attributes. The    `Secure`, the browser  
                          `example.com` test is not instructed to   
                          did not return a   send it only over      
                          `Set-Cookie`       HTTPS.                 
                          header.                                   

  3          Response     The incident brief Missing HTTP security  Medium
             headers      reports missing    headers, reducing      
                          HSTS, Content      defense-in-depth       
                          Security Policy    against downgrade,     
                          (CSP), and         content-injection, and 
                          X-Frame-Options    framing/clickjacking   
                          protections. The   risks.                 
                          filter against                            
                          `example.com`                             
                          returned no                               
                          matching                                  
                          security-header                           
                          lines, which only                         
                          describes that                            
                          test domain.                              

  4          Redirect     The incident brief Potential open         High
             handling     reports that an    redirect. An attacker  
                          external redirect  may use a trusted      
                          destination is     portal URL to send     
                          accepted without   users to a malicious   
                          adequate           external destination.  
                          validation. The                           
                          `example.com`                             
                          request returned                          
                          `HTTP/2 200` and                          
                          no `Location` line                        
                          in the filtered                           
                          output; it did not                        
                          test the patient                          
                          portal redirect                           
                          behavior.                                 
  -----------------------------------------------------------------------------

------------------------------------------------------------------------

## Analysis Summary

**Finding 1:** The incident brief indicates that credentials are placed
in a GET URL. URL parameters can be retained in browser history and
logs, increasing the chance of credential exposure. The application team
should use POST for credential submission over HTTPS, avoid placing
secrets in URLs, and review relevant logs for possible exposure.

**Finding 2:** The incident brief states that the session cookie is
missing `HttpOnly` and `Secure`. Without `HttpOnly`, client-side scripts
may be able to read the cookie, and without `Secure`, the browser is not
restricted to sending it only over HTTPS. The application team should
set both attributes and review appropriate `SameSite` behavior and
session lifecycle controls.

**Finding 3:** The incident brief identifies missing HSTS, CSP, and
X-Frame-Options headers. These protections help enforce HTTPS and reduce
certain browser-based attacks, including some content-injection and
framing risks. The application team should select and test appropriate
policies, including a suitable CSP and framing protection, before
deploying them to production.

**Finding 4:** The incident brief indicates that the portal accepts an
external redirect destination without sufficient validation. This could
let an attacker craft a portal link that redirects users to a malicious
site and makes the destination appear more trustworthy. The application
team should use an allowlist of approved destinations or server-side
destination identifiers, reject untrusted external URLs, and test
redirect handling.

------------------------------------------------------------------------

## HTTP Header Inspection Commands Used

The following commands were run against `https://example.com`, a
demonstration domain. They are useful for practicing HTTP inspection but
do **not** independently confirm the patient portal findings.

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Command                                                                                                          Purpose and Observed Result
  ---------------------------------------------------------------------------------------------------------------- ------------------------------------------------------
  `curl -I https://example.com 2>/dev/null`                                                                        Retrieved response headers without printing the
                                                                                                                   response body. The observed response was `HTTP/2 200`;
                                                                                                                   it included `content-type: text/html; charset=utf-8`,
                                                                                                                   `server: cloudflare`, and `allow: GET, HEAD`, among
                                                                                                                   other headers.

  `curl -I https://example.com 2>/dev/null \| grep -iE "strict\|content-security\|x-frame\|x-content\|referrer"`   Filtered the response headers for terms associated
                                                                                                                   with common browser security headers. The command
                                                                                                                   returned no matching lines. This means no lines
                                                                                                                   matched the filter in that response; it does not prove
                                                                                                                   the patient portal is missing those headers.

  `curl -v --max-time 5 https://example.com 2>&1 \| grep -E "^[<>]" \| head -20`                                   Inspected request and response lines. The observed
                                                                                                                   request used `GET / HTTP/2` with `Host: example.com`;
                                                                                                                   the response was `HTTP/2 200` and included response
                                                                                                                   headers such as `content-type` and
                                                                                                                   `server: cloudflare`.

  `curl -IL --max-time 5 https://example.com 2>/dev/null \| grep -E "HTTP\|Location\|Set-Cookie"`                  Followed redirects and filtered for HTTP status,
                                                                                                                   redirect destination, and cookie-setting headers. The
                                                                                                                   observed output was `HTTP/2 200`; no `Location` or
                                                                                                                   `Set-Cookie` lines appeared in the filtered output.
                                                                                                                   This result applies only to the tested request to
                                                                                                                   `example.com`.
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

## Escalation Summary

The incident brief describes four findings that warrant escalation:
credential exposure through a GET URL, session cookies missing
`HttpOnly` and `Secure`, missing browser security headers, and an
insufficiently validated external redirect. The first, second, and
fourth findings are assessed as High severity, while the missing-header
finding is assessed as Medium; these are initial analyst assessments and
should be confirmed by authorized security personnel. Recommended next
steps are to validate each issue against the actual patient portal in an
approved test environment, preserve relevant request/response evidence
and logs, assess whether credentials or sessions may have been exposed,
and prioritize remediation based on verified exploitability and impact.
The application owner and security team should decide implementation
details, coordinate any production changes, and retest after fixes are
applied. The `example.com` command results are practice observations
only and must not be presented as proof of the Maplewood portal's
behavior.

------------------------------------------------------------------------

*CPSC 4584 \| Governors State University \| Fall 2026*
