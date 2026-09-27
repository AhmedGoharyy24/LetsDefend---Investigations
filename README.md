# SOC Alert Investigations

A collection of **Security Operations Center (SOC) alert investigations** documenting the process of analyzing, triaging, and investigating security alerts.

This repository contains practical case studies covering areas such as **malware, web attacks, suspicious network activity, phishing, endpoint alerts, and threat intelligence investigations**.

The goal is to demonstrate how I approach security alerts from a **SOC Analyst perspective**, moving from initial triage to investigation, correlation, and final assessment.

---

## 🎯 Objectives

The main objectives of these investigations are to demonstrate:

- Alert triage and classification
- Identification of Indicators of Compromise (IOCs)
- Log and event analysis
- Endpoint investigation
- Network traffic investigation
- Threat intelligence analysis
- MITRE ATT&CK mapping
- Attack timeline reconstruction
- Correlation of multiple security events
- Distinguishing between attack attempts and successful exploitation
- Determining appropriate alert disposition

---

## 🔎 Investigation Methodology

Most investigations follow a general workflow:

```text
Security Alert
     │
     ▼
Initial Triage
     │
     ▼
Identify IOCs
     │
     ├── File Hash
     ├── IP Address
     ├── Domain
     ├── URL
     └── Host / User
     │
     ▼
Threat Intelligence
     │
     ▼
Log & Endpoint Investigation
     │
     ▼
Event Correlation
     │
     ▼
MITRE ATT&CK Mapping
     │
     ▼
Impact Assessment
     │
     ▼
Final Verdict
```

The exact investigation process varies depending on the type of alert and the available telemetry.

---

## 📂 Repository Structure

```text
SOC-Alert-Investigations/
│
├── README.md
│
├── Malware/
│   ├── Suspicious-XLSM-File/
│   │   └── README.md
│   └── ...
│
├── Web-Attacks/
│   ├── XSS-Attempt/
│   │   └── README.md
│   └── ...
│
├── Network/
│   └── ...
│
├── Phishing/
│   └── ...
│
├── Endpoint/
│   └── ...
│
└── Threat-Intelligence/
    └── ...
```

Each investigation contains a standalone write-up describing the alert, investigation process, evidence, and final assessment.

---

## 📋 Investigation Format

Each case study generally contains:

### 1. Alert Overview
Basic information such as:

- Event ID
- Detection rule
- Timestamp
- Host
- Source/Destination
- Alert type
- Severity

### 2. Initial Triage

Identify the suspicious behavior and determine the first investigation steps.

### 3. IOC Investigation

Investigate relevant indicators such as:

- IP addresses
- File hashes
- Domains
- URLs
- File names
- User accounts

### 4. Log & Endpoint Analysis

Correlate available telemetry to understand what happened before, during, and after the alert.

### 5. Threat Intelligence

Use external intelligence sources to enrich the investigation and identify relationships between indicators.

### 6. MITRE ATT&CK

Map the observed behavior to relevant MITRE ATT&CK techniques where applicable.

### 7. Final Assessment

Determine the most appropriate disposition based on the available evidence.

Examples:

```text
True Positive
False Positive
Benign Activity
Suspicious - Further Investigation Required
Attack Attempt - Exploitation Unconfirmed
```

---

## 🛠️ Tools & Resources

The investigations may use different tools depending on the alert.

Common resources include:

- Splunk
- VirusTotal
- ANY.RUN
- AbuseIPDB
- MITRE ATT&CK
- EDR telemetry
- SIEM logs
- Web server logs
- Firewall logs
- Network traffic

Tool usage is documented within each investigation where relevant.

---

## 📚 Investigations

| Investigation | Category | Focus |
|---|---|---|
| [SOC138 - Suspicious XLSM File](./Malware/Suspicious-XLSM-File/) | Malware | Malicious Excel file, threat intelligence & suspected C2 |
| [SOC166 - JavaScript Detected in URL](./Web-Attacks/XSS-Attempt/) | Web Attack | XSS attempt & web request analysis |

More investigations will be added as I continue practicing SOC alert investigation and incident analysis.

---

## ⚠️ Disclaimer

These investigations are created for **educational and portfolio purposes**.

Some investigations are based on simulated security alerts and training environments. Conclusions are based on the telemetry and evidence available during each investigation.

Where evidence is incomplete, assumptions and hypotheses are explicitly identified rather than presented as confirmed facts.

No real-world confidential data, credentials, or sensitive organizational information is intentionally included.

---

## 👨‍💻 About

I am a **Cybersecurity student focused on Security Operations and SOC Analysis**, with a particular interest in:

- SOC Operations
- Incident Response
- Threat Detection
- Digital Forensics
- Threat Intelligence
- Malware Analysis
- SIEM

This repository is part of my practical cybersecurity portfolio and is intended to document my progression in investigating real-world-style security alerts.

---

## 📈 Continuous Learning

This repository is continuously updated with new investigations, techniques, and lessons learned.

The focus is not only on identifying malicious activity, but also on understanding:

> **What happened? Why did it happen? What evidence supports it? What happened next? And can we prove the impact?**
