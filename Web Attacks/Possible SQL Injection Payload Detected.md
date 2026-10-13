# SOC165 — EventID 115 Possible SQL Injection Payload Detected

## Introduction

This investigation examines a web security alert triggered by a suspicious SQL injection payload submitted through a web application's search functionality. The source repeatedly tested different SQL expressions, including Boolean conditions and an `ORDER BY` statement, potentially attempting to manipulate the application's database query.

The investigation focused on source IP reputation, URL decoding, request sequencing, and HTTP response codes to assess whether the attack succeeded.

## Alert Overview

|Field|Details|
|---|---|
|Event ID|115|
|Event Time|February 25, 2022, 11:34:32 UTC+03:00|
|Rule|SOC165 - Possible SQL Injection Payload Detected|
|Alert Type|Web Attack|
|Difficulty|Medium|
|MITRE ATT&CK|T1190 — Exploit Public-Facing Application|
|Hostname|WebServer1001|
|Source IP|`167.99.169.17`|
|Destination IP|`172.16.17.18`|
|HTTP Method|GET|
|Device Action|Allowed|
|Detection Trigger|Requested URL contains `OR 1 = 1`|

## 1. Initial Triage

The alert was triggered by a request to the application's search endpoint containing a recognizable SQL injection payload.

**User-Agent:**

`Mozilla/5.0 (Windows NT 6.1; WOW64; rv:40.0) Gecko/20100101 Firefox/40.1`

The User-Agent identifies itself as an older Firefox browser on Windows 7. However, User-Agent values can be spoofed and should not be treated as reliable proof of the attacker's actual environment.

The initial request under investigation was:

`https://172.16.17.18/search/?q=%22%20OR%201%20%3D%201%20--%20-`

Because the URL was encoded, I decoded its query parameter to understand the submitted input.

## 2. Source IP Reputation Analysis

The source IP address, `167.99.169.17`, was checked using VirusTotal and AbuseIPDB.

**Findings:**

- **VirusTotal:** Reported as malicious by 2 vendors and suspicious by 3 vendors.
    
- **AbuseIPDB:** The IP address was found with existing reports.
    

These results increase the suspicion associated with the source IP. However, reputation data is supporting evidence rather than proof that a particular request succeeded or that the IP's current operator is the attacker.

## 3. SQL Injection Payload Analysis

After URL decoding, the main payload was:

`" OR 1 = 1 -- -`

The payload contains several recognizable SQL injection components:

|Component|Potential Purpose|
|---|---|
|`"`|Attempts to terminate a double-quoted string, depending on the application's query construction.|
|`OR 1 = 1`|Adds a condition that evaluates to true, potentially altering query logic.|
|`--`|Starts a SQL comment in many SQL dialects, potentially ignoring the remainder of a query.|
|`-`|An additional character often used in comment-based payloads; its interpretation depends on the database dialect and syntax.|

The payload is consistent with an attempt to manipulate the SQL statement used by the search functionality. Whether it would work depends on how the application constructs queries and which database engine it uses.

## 4. Web Server Log Analysis

I reviewed the web logs to reconstruct the sequence of requests made to the application.

### Observed Attack Flow

1. The source IP accessed the main page: `https://172.16.17.18/`
    
2. A search request was sent with the payload `' OR '1`:  
    `https://172.16.17.18/search/?q=%27%20OR%20%271`
    
3. A second Boolean-based payload was submitted:  
    `https://172.16.17.18/search/?q=%27%20OR%20%27x%27%3D%27x`
    
4. An `ORDER BY` payload was submitted:  
    `https://172.16.17.18/search/?q=1%27%20ORDER%20BY%203--%2B`
    
5. The final request contained the `OR 1 = 1` payload that triggered the alert:  
    `https://172.16.17.18/search/?q=%22%20OR%201%20%3D%201%20--%20-`
    

### HTTP Response Analysis

All observed requests containing SQL injection payloads returned:

- **HTTP status:** `500 Internal Server Error`
    

The repeated requests suggest deliberate testing of the search parameter using multiple SQL injection techniques.

However, an HTTP 500 response only establishes that the server encountered an error. It does not independently prove that the database rejected the payload or that exploitation was unsuccessful. No successful exploitation was demonstrated by the evidence available in the reviewed logs.

To establish the outcome with greater confidence, additional evidence would be needed, such as application exceptions, database logs, response bodies, and indicators of unauthorized data retrieval.

## 5. Attack Assessment

The investigation identified multiple indicators consistent with a SQL injection attempt:

- The URL parameters contained recognizable SQL injection syntax.
    
- Multiple payload variations targeted the same search endpoint.
    
- The source IP had existing reputation reports on VirusTotal and AbuseIPDB.
    
- Every observed SQL injection request returned HTTP 500.
    

**Assessment:**

- **Detection accuracy:** The alert correctly identified suspicious SQL injection patterns.
    
- **Attack intent:** Strongly consistent with deliberate SQL injection testing.
    
- **Exploitation success:** Not demonstrated by the available evidence.
    
- **Data disclosure:** No evidence of successful data retrieval was identified in the reviewed logs.
    
- **Further activity:** Continued monitoring is recommended to identify retries or related requests.
    

## 6. Final Assessment

**Final Verdict: True Positive — Suspected SQL Injection Attempt, Success Unconfirmed**

The alert is a True Positive because the request sequence contains multiple recognizable SQL injection payloads directed at the application's search endpoint.

All observed payload-bearing requests returned HTTP 500 responses. Based on the available evidence, successful exploitation was not demonstrated, but the response codes alone are insufficient to conclusively establish that the attack failed.

Tier 2 escalation was not considered necessary based on the findings available for this case. The activity should remain documented and monitored, with additional escalation if evidence of successful exploitation, sensitive data exposure, or continued attack activity emerges.

### Recommended Actions

1. Review application and database logs to identify the cause of the HTTP 500 errors and check for successful query execution.
    
2. Ensure database queries use parameterized statements or prepared statements instead of unsafe string concatenation.
    
3. Validate and sanitize user input, while recognizing that input filtering alone is not a reliable SQL injection defense.
    
4. Review web application firewall rules and tune detection logic to recognize encoded and variant SQL injection payloads.
    
5. Monitor for repeated requests from the same source and similar payloads from other sources.
    
6. Confirm that the search endpoint handles malformed input safely without exposing internal errors or sensitive information.
    

## Key SOC Takeaway

Repeated SQL injection payloads, suspicious source reputation, and consistent server errors provide strong evidence of an attack attempt—but they do not, by themselves, prove whether exploitation succeeded.

**A True Positive detection confirms that suspicious malicious activity occurred; it does not automatically confirm a successful compromise.**