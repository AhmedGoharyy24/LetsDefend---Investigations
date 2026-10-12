# SOC170 — Passwd Found in Requested URL: Possible LFI Attack

## Introduction

This investigation analyzes a web attack alert generated when a request to a web server contained a path traversal payload targeting the Linux `/etc/passwd` file. The request attempted to access a sensitive system file through a URL parameter, which is a common indicator of a **Local File Inclusion (LFI) or path traversal attack**.

The investigation focused on source IP reputation, web server logs, and the HTTP response to determine whether the attack succeeded.

## Alert Overview

|Field|Details|
|---|---|
|Event ID|120|
|Event Time|March 1, 2022, 10:10:41 UTC+03:00|
|Rule|SOC170 - Passwd Found in Requested URL - Possible LFI Attack|
|Alert Type|Web Attack|
|Difficulty|Easy|
|MITRE ATT&CK|T1190 — Exploit Public-Facing Application|
|Hostname|WebServer1006|
|Destination IP|`172.16.17.13`|
|Source IP|`106.55.45.162`|
|HTTP Method|GET|
|Device Action|Allowed|
|Requested URL|`https://172.16.17.13/?file=../../../../etc/passwd`|

## 1. Initial Triage

The alert was triggered because the requested URL contained `passwd`, together with multiple directory traversal sequences (`../`).

The payload attempted to navigate outside the application's expected directory and access:

`/etc/passwd`

On Linux systems, this file contains local account information, including usernames and other account-related fields. It does not normally contain users' plaintext passwords.

The User-Agent was:

`Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; .NET CLR 1.1.4322)`

This identifies itself as an old Internet Explorer browser running on Windows XP. Although unusual, a User-Agent string can be spoofed and is not sufficient by itself to identify the attacker's actual operating system or tools.

## 2. Source IP Reputation Analysis

The source IP address, `106.55.45.162`, was checked using VirusTotal and AbuseIPDB.

**Findings:**

- VirusTotal: No reports found at the time of investigation.
    
- AbuseIPDB: No reports found at the time of investigation.
    
- Reported geolocation: China.
    

No existing reputation reports were identified on either platform. However, the absence of reports does not establish that an IP address is safe. Geolocation also does not establish the identity or physical location of the attacker.

The source IP therefore remained an indicator associated with the suspicious request, but its malicious reputation was not independently confirmed through these services.

## 3. Web Server Log Analysis

The web server logs were examined to determine whether the request succeeded.

**Observed evidence:**

- Only one request was identified from the source IP for the suspicious URL.
    
- The request attempted to access `/etc/passwd` through the `file` parameter.
    
- HTTP response status: `500 Internal Server Error`.
    
- Response size: `0` bytes.
    

The HTTP 500 response indicates that the server encountered an error while processing the request. The zero-byte response provides no evidence that file contents were returned to the requester.

Based on the available logs, there was no evidence of successful file disclosure. However, the response code and size alone cannot conclusively prove that no file access occurred internally; application and server-side logs would be needed to establish that with greater confidence.

## 4. Attack Analysis

The request contains a recognizable directory traversal payload targeting a sensitive Linux file. This is consistent with an attempted LFI or path traversal attack against a public-facing application.

The evidence supports the following assessment:

- **Malicious intent:** The requested path strongly indicates an attempt to access a sensitive system file.
    
- **Detection accuracy:** The rule correctly identified a suspicious URL.
    
- **Exploit success:** Not demonstrated by the available evidence.
    
- **Data disclosure:** No file contents were returned in the recorded response.
    
- **Further activity:** Only one matching request was found in the reviewed logs.
    

The request was allowed through the device, but this does not mean the application-level attack succeeded. The server returned an error rather than a successful response containing data.

## 5. Final Assessment

**Final Verdict: True Positive — Unsuccessful LFI/Path Traversal Attempt**

The alert is a True Positive because the request contained a suspicious traversal payload targeting `/etc/passwd`. However, the available evidence does not demonstrate successful exploitation or file disclosure.

No Tier 2 escalation was considered necessary for this alert based on the investigation findings and the case's assessment. The event should still be documented, and the web server should be monitored for repeated traversal attempts or related suspicious requests.

### Recommended Actions

1. Review application error logs to determine why the server returned HTTP 500.
    
2. Validate and sanitize file-related URL parameters.
    
3. Restrict file access to explicitly permitted directories.
    
4. Monitor for repeated requests containing `../`, `/etc/passwd`, and similar traversal patterns.
    
5. Investigate further if additional requests, successful responses, or evidence of sensitive file access appears.
    

## Key SOC Takeaway

A security alert can correctly identify a malicious attempt even when exploitation is unsuccessful. In this case, the requested URL provided strong evidence of an LFI/path traversal attempt, while the HTTP 500 response and zero-byte response size provided no evidence of successful data disclosure.

**The key distinction is between detecting an attack attempt and confirming that the attack succeeded.**