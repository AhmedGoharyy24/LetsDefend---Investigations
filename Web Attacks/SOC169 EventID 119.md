# SOC Investigation: Successful IDOR Attack

## Introduction

During a LetsDefend investigation, I analyzed an alert for a possible **Insecure Direct Object Reference (IDOR)** attack against a web application.

The investigation showed that an external source repeatedly accessed the same endpoint while changing the `user_id` value. The requests were allowed and returned successful HTTP responses, with different response sizes for each user ID.

Based on the available evidence, the attack was classified as a **successful IDOR attack** and escalated to Tier 2.

---

## Alert Overview

|Field|Details|
|---|---|
|**Event ID**|119|
|**Event Time**|2022-02-28 22:48:05 UTC+03:00|
|**Rule**|SOC169 - Possible IDOR Attack Detected|
|**Alert Type**|Web Attack|
|**Severity**|Medium|
|**Hostname**|WebServer1005|
|**Source IP**|`134.209.118.137`|
|**Destination IP**|`172.16.17.15`|
|**HTTP Method**|POST|
|**Device Action**|Allowed|
|**MITRE ATT&CK**|T1190 — Exploit Public-Facing Application|

---

## 1. Initial Triage

The alert was triggered because of **consecutive requests to the same web application endpoint**:

```text
/get_user_info/
```

The request originated from:

```text
134.209.118.137
```

The User-Agent was:

```text
Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; .NET CLR 1.1.4322)
```

This identifies itself as an extremely old Internet Explorer version running on Windows XP.

A User-Agent can be spoofed, so I treated it as supporting context rather than proof of the attacker's actual operating system.

---

## 2. Source IP Investigation

I checked the source IP against threat-intelligence sources.

```text
134.209.118.137
```

- **VirusTotal:** No detections at the time of investigation.
    
- **AbuseIPDB:** The IP was present in the database.
    

The IP reputation increased the suspicion around the activity, but the actual HTTP behavior was more important for determining whether an attack occurred.

---

## 3. Log Investigation

I pivoted on the source IP and reviewed the requests generated against the web server.

The attacker made **five requests** to the same endpoint while changing the `user_id` value:

```text
/get_user_info/ → user_id=1
/get_user_info/ → user_id=2
/get_user_info/ → user_id=3
/get_user_info/ → user_id=4
/get_user_info/ → user_id=5
```

All of the requests were:

- Allowed by the security device
    
- Returned with HTTP `200 OK`
    
- Targeted at the same user-information endpoint
    

The important observation was that the **response size changed for each user ID**.

This suggests that the server was returning different data depending on the supplied identifier.

---

## 4. Why This Indicates IDOR

An IDOR vulnerability occurs when an application uses a user-controlled identifier to access an object without properly verifying whether the requesting user is authorized to access it.

For example, an application might normally request:

```text
user_id=1
```

An attacker may then change the value:

```text
user_id=2
```

If the application returns information belonging to user 2 without performing an authorization check, the attacker can potentially access other users' data.

In this investigation, the repeated modification of the `user_id` value, combined with successful responses and different response sizes, indicated that the attacker was attempting to access information belonging to multiple users.

---

## 5. Was the Attack Successful?

The key evidence was:

- Five different `user_id` values were requested.
    
- Every request was permitted.
    
- Every request returned `HTTP 200 OK`.
    
- Each request produced a different response size.
    

This behavior indicates that the application processed the modified identifiers and returned different content.

Based on the available telemetry, the attack was therefore classified as:

**Successful IDOR Attack**

The LetsDefend investigation also confirmed the attack as successful.

---

## 6. Attack Direction

The traffic direction was:

```text
Internet → Company Network
```

The source was an external IP address targeting the internal web server.

The activity was also confirmed as **not a planned security test**, increasing the likelihood that the behavior represented an actual attack rather than authorized testing.

---

## 7. Response

Because the attacker may have successfully accessed information belonging to other users, the web server should be **contained and escalated to Tier 2** for further investigation.

Recommended follow-up actions include:

- Determine exactly what information was returned for each user ID.
    
- Identify whether sensitive user information was exposed.
    
- Review authentication and authorization controls.
    
- Search for additional requests from the same source IP.
    
- Identify other affected user IDs.
    
- Review application logs for further unauthorized access.
    
- Investigate whether data was exfiltrated.
    
- Remediate the underlying authorization vulnerability.
    

---

# Final Assessment

**Verdict: True Positive — Successful IDOR Attack**

The attacker modified the `user_id` parameter across multiple requests to access different user records. The requests were allowed and returned successful responses with varying response sizes, indicating that the application was returning different information for the supplied identifiers.

The activity originated from the Internet, was not a planned test, and was assessed as malicious.

**Recommended action: Contain the affected web server and escalate to Tier 2.**

---

## Key SOC Takeaway

This investigation demonstrates why analyzing a single HTTP request is often not enough.

The individual requests may appear normal, but correlating them revealed the attack pattern:

```text
Same Endpoint
      ↓
Change user_id
      ↓
Multiple Users
      ↓
HTTP 200 Responses
      ↓
Different Response Sizes
      ↓
Potential Unauthorized Data Access
      ↓
Successful IDOR
```
