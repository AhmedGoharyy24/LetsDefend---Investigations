# SOC Investigation: Phishing Email Containing Malicious Attachment

## Introduction

During a LetsDefend investigation, I analyzed an alert triggered by **SOC140 — Phishing Mail Detected - Suspicious Task Scheduler**.

The email used a **COVID-19 themed lure** to convince the recipient to open an attached file. The investigation focused on identifying whether the attachment was malicious and determining whether the email was successfully delivered to the user.

---

## Alert Overview

|Field|Details|
|---|---|
|**Event ID**|82|
|**Event Time**|2021-03-21 12:26:57 UTC+03:00|
|**Rule**|SOC140 - Phishing Mail Detected - Suspicious Task Scheduler|
|**Alert Type**|Exchange|
|**Severity**|Medium|
|**SMTP Address**|`189.162.189.159`|
|**Sender**|`aaronluo@cmail.carleton.ca`|
|**Recipient**|`mark@letsdefend.io`|
|**Subject**|`COVID19 Vaccine`|
|**Device Action**|Blocked|
|**Final Result**|True Positive|

---

## 1. Initial Email Investigation

I started by examining the email body and looking for suspicious URLs or attachments.

The message contained:

> "Hey, did you read breaking news about Covid-19. Open it now!"

The email also contained an attachment.

The use of a **COVID-19 breaking-news theme** combined with an urgent request to open an attachment was suspicious and warranted further investigation.

---

## 2. Sender IP Investigation

The SMTP/source IP was:

```
189.162.189.159
```

I checked the IP using multiple threat-intelligence sources.

- **VirusTotal:** No vendor detections at the time of investigation.
    
- **AbuseIPDB:** The IP was present in the database.
    

Although the IP reputation provided additional context, the email content and attachment required deeper investigation.

---

## 3. Attachment Investigation

The attachment hash was:

```
72c812cf21909a48eb9cceb9e04b865d
```

I investigated the hash using VirusTotal and Hybrid Analysis.

### VirusTotal

The file was reported as malicious by:

**31 security vendors**

### ANY.RUN

The file appeared to be a PDF.

During analysis, the PDF appeared blurred and contained a hyperlink. Following the hyperlink resulted in a:

```
404 Page Not Found
```

Although the linked page was no longer accessible, the malicious reputation of the attachment indicated that the file should not be considered benign.

---

## 4. Delivery Investigation

The next step was determining whether the recipient actually received or interacted with the email.

The security device recorded:

```
Device Action: Blocked
```

I also checked the available logs and EDR telemetry for evidence of:

- The attachment being downloaded
    
- The download URL being accessed
    
- The URL contained inside the PDF being accessed
    

No corresponding requests were found.

The attachment download URL was:

```
hxxps://download.cyberlearn.academy/download/download?url=https://files-ld.s3.us-east-2.amazonaws.com/72c812cf21909a48eb9cceb9e04b865d.zip
```

The absence of download activity, combined with the **Blocked** device action, indicates that the email was stopped before delivery to the user.

---

## 5. Investigation Summary

The investigation produced the following evidence:

- The email used a COVID-19 themed social-engineering lure.
    
- A suspicious attachment was included.
    
- The attachment hash was detected as malicious by 31 VirusTotal vendors.
    
- The attachment contained a hyperlink.
    
- The linked page returned `404`, but this does not make the attachment safe.
    
- The sender IP was present in AbuseIPDB.
    
- No evidence showed that the recipient downloaded or accessed the attachment.
    
- The security device blocked the email before delivery.
    

---

## Final Verdict

**True Positive — Phishing Email**

The email was malicious and contained a malicious attachment intended to convince the recipient to open it.

However, there was **no evidence of successful delivery or user interaction**, as the security control blocked the email and no related download or URL-access activity was observed.

### Key SOC Takeaway

A phishing investigation should not stop at analyzing the sender or attachment.

The analyst should also determine:

```
Email Received?
      ↓
Attachment / URL Malicious?
      ↓
User Downloaded It?
      ↓
User Opened / Accessed It?
      ↓
Any Post-Exploitation Activity?
```

In this case, the attack was detected and blocked before reaching the recipient.