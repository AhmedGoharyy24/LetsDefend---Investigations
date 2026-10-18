# SOC167 — LS Command Detected in Requested URL

## Introduction

This investigation examines a web attack alert triggered by a URL supposedly containing the `ls` command. The investigation aimed to determine whether the request represented an attempt to execute a command through a web application or whether the detection rule generated a false positive.

The analysis included reviewing the requested URL, identifying the source device, examining web logs, and checking the discrepancy between the alert timestamp and the request timestamp.

## Alert Overview

|Field|Details|
|---|---|
|Event ID|117|
|Event Time|February 27, 2022, 00:36:04 UTC+03:00|
|Rule|SOC167 - LS Command Detected in Requested URL|
|Alert Type|Web Attack|
|Difficulty|Easy|
|MITRE ATT&CK|T1190 — Exploit Public-Facing Application|
|Hostname|EliotPRD|
|Source IP|`172.16.17.46`|
|Destination IP|`188.114.96.15`|
|HTTP Method|GET|
|Device Action|Allowed|
|Requested URL|`https://letsdefend.io/blog/?s=skills`|

## 1. Initial Alert Triage

I first inspected the requested URL to determine why the detection rule had triggered:

`https://letsdefend.io/blog/?s=skills`

The URL contains the query parameter `s=skills`, which appears to be a normal search request for blog content. It does not contain an obvious `ls` command or recognizable command-execution payload.

The User-Agent was:

`Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:24.0) Gecko/20100101 Firefox/24.0`

This identifies itself as Firefox running on Ubuntu Linux, with `i686` indicating a 32-bit x86 environment. The User-Agent appears consistent with a browser request and did not provide an obvious indicator of malicious activity.

At this stage, the URL appeared benign. However, I continued the investigation rather than dismissing the alert based on the URL alone.

## 2. Source IP Investigation

I examined the source IP address:

`172.16.17.46`

This is an RFC 1918 private IP address associated with the internal device **EliotPRD**.

Because the source was an internal device, I reviewed the activity associated with this IP to determine whether the request was part of normal browsing or related to suspicious behavior.

The source IP alone did not indicate malicious activity. The next step was to correlate the request with the surrounding web logs.

## 3. Web Log Analysis

I reviewed the available logs for the source IP and the relevant time period.

**Findings:**

- The source generated requests to pages within the `/blog/` directory.
    
- The reviewed requests appeared consistent with normal browsing activity.
    
- No suspicious command-execution payloads or related malicious requests were identified in the logs reviewed.
    
- The URL associated with the alert appeared to be a normal blog search request.
    

### Timestamp Discrepancy

One important observation was that the alert was timestamped **February 27, 2022**, while the request associated with it appeared to have occurred on **February 26, 2022**.

This discrepancy may indicate a delay in alert generation, a difference in timestamp handling, or a correlation/configuration issue. The available evidence does not establish the exact cause.

The alert and request timestamps should therefore be checked against the original event records, time zones, and detection pipeline before concluding that the rule itself is misconfigured.

## 4. Detection Rule Assessment

The detection rule was designed to identify URLs containing the `ls` command. However, the requested URL contained `?s=skills`, which is consistent with a search parameter rather than command execution.

This suggests that the rule may be matching a short string or applying overly broad pattern matching instead of identifying command-execution context.

Potential improvements include:

1. Use more precise matching for command-execution patterns rather than triggering on a short substring.
    
2. Evaluate the surrounding URL and parameter value before generating an alert.
    
3. Test the rule against benign search terms such as `skills` to reduce false positives.
    
4. Investigate the timestamp discrepancy between the alert and the underlying request.
    
5. Validate rule changes against known malicious examples to ensure that tuning does not create detection gaps.
    

## 5. Final Assessment

**Final Verdict: False Positive**

The requested URL appeared to be a legitimate blog search request, and the reviewed logs showed normal browsing activity without identified command-execution indicators.

The alert was likely generated because the detection logic matched benign URL content rather than a genuine command-execution payload. The timestamp discrepancy also warrants further review, although its root cause was not confirmed.

No evidence from the reviewed logs justified escalating this event as a confirmed web attack.

### Recommended Actions

- Tune the detection rule to reduce matches on benign search parameters.
    
- Validate the alert and request timestamps against raw logs.
    
- Continue monitoring the internal device for unrelated suspicious activity.
    
- Retest the updated detection logic with both benign and malicious URL examples.
    

## Key SOC Takeaway

Not every alert generated by a security rule represents a genuine attack. Analysts must validate the triggering evidence, correlate the event with surrounding logs, and distinguish suspicious-looking patterns from normal application behavior.

**Accurate detection tuning reduces alert fatigue while helping analysts focus on genuine threats.**