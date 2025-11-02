# 🛡️ LetsDefend: SQL Injection Payload Detection and Triage

## 🚨 Project Overview

This repository documents a **Security Operations Center (SOC) Tier 1 Analyst's** workflow in identifying, analyzing, and triaging a **Classic SQL Injection (SQLi) attempt** against a simulated web application. The project highlights crucial skills in log analysis, threat intelligence correlation, URL decoding, and initial incident assessment to determine the severity and necessary next steps.

The scenario demonstrates detection of the classic `' OR 1=1 -- -` payload, which is designed to bypass authentication or extract data by making the SQL query logic always true.

---

## 🎯 Key Learning Objectives

* **Threat Intelligence Correlation:** Using open-source and internal tools (like a simulated ArcSight/Splunk integration with VirusTotal) to enrich alerts.
* **Log Analysis & Investigation:** Analyzing web server access logs to identify suspicious HTTP status codes and patterns (e.g., successful vs. error responses).
* **Payload Decoding:** Utilizing tools to decode URL-encoded attacks and reveal the true malicious payload.
* **Incident Triage & Escalation:** Determining the success of the attack and deciding whether to escalate the case to a higher security tier (Tier 2/3).
* **OWASP Top 10 Relevance:** Demonstrating detection for **Injection** attacks, which consistently ranks as a critical risk on the OWASP Top 10 list.

---

## 💻 Tools and Technologies Used

| Category | Tool / Resource | Purpose in Project |
| :--- | :--- | :--- |
| **Log Management** | SIEM (Simulated ArcSight/Splunk) | Centralized logging and alert generation. |
| **Vulnerability Scanning** | OWASP ZAP (Mentioned) | Used previously for web application vulnerability discovery. |
| **Threat Intelligence** | VirusTotal, ArcSight Threat Intel | Validating the reputation of the Source IP address. |
| **Analysis Utility** | Online URL Decoder | Unmasking the encoded attack payload from the URL request. |
| **Defense System** | Endpoint Security (Simulated) | Checking local process activity and file changes on the victim. |
| **Vulnerability Type**| SQL Injection (SQLi) | The specific web application attack vector being investigated. |

---

## 🔎 Case Management Walkthrough: SQL Injection Payload

### 1. Initial Alert and Enrichment

* **Alert Triggered:** A high-severity alert was generated for a potential **SQL Injection Payload Detection** targeting a web application at IP `172.16.17.18`.
* **Source IP Analysis:** The Source IP address was queried against **VirusTotal** and **ArcSight Threat Intelligence**.
    * **Result:** The IP was flagged as **malicious**, specifically associated with phishing activity. This increased the alert's priority.
* **Requested URL (Encoded):**
    ```
    [https://172.16.17.18/search/?q=%22%20OR%201%20%3D%201%20--%20-](https://172.16.17.18/search/?q=%22%20OR%201%20%3D%201%20--%20-)
    ```

### 2. Payload Decoding and Identification

To understand the attacker's intent, the analyst used an **Online URL Decoder** tool.

* **Decoded Payload (Classic SQLi):**
    ```
    [https://172.16.17.18/search/?q=](https://172.16.17.18/search/?q=)" OR 1 = 1 -- -
    ```
    * **Analysis:** This payload is a classic **Boolean-based SQL Injection** attempt. The `' OR 1=1` statement is intended to bypass authentication checks or make a conditional query always true. The `-- -` (or `-- `) sequence comments out the rest of the legitimate SQL query.

### 3. Log Analysis and Triage

The analyst performed a timeline analysis in the log management system to gauge the attack's effectiveness, focusing on HTTP **Response Status Codes**.

| Request Type | Request URL (Partial) | Response Status Code | Implication |
| :--- | :--- | :--- | :--- |
| **Normal** | `https://172.16.17.18/` | `200 OK` | Successful, normal web traffic. |
| **Probe (Initial)** | `https://172.16.17.18/search/?q='` | `500 Internal Server Error` | The single quote caused a database error, confirming an SQL syntax vulnerability exists. |
| **Exploit** | `...q=" OR 1 = 1 -- -` | `500 Internal Server Error` | The injection payload failed to execute successfully. |

### 4. Conclusion on Attack Success

* **Key Indicator:** The exploit attempt consistently returned a **`500 Internal Server Error`** and the **file size of the response remained unchanged** compared to the initial single-quote probe.
* **Determination:** The attack was identified as a **malicious SQL Injection attempt**, but the injection was likely **unsuccessful** (or the attacker failed to extract data) because the application returned a server-side error rather than a modified, successful response (e.g., a `200 OK` with altered content).

### 5. Final Case Documentation and Triage Action

1.  **Attack Classification:** Confirmed attempted SQL Injection (OWASP Top 10: Injection).
2.  **Success Assessment:** Injection attempt was **unsuccessful** based on the consistent `500` status code and unchanging response size.
3.  **Endpoint Security Check:** Verified that no unusual process creation or file changes were observed on the endpoint security platform, reinforcing the conclusion of an unsuccessful exploit.
4.  **Case Notes:** Added the full decoded request URL and a detailed analysis.
5.  **Escalation Decision:** **No Tier 2/3 escalation required.** The immediate threat was contained by the application's error handling, and the initial triage provided sufficient data for security engineering to address the underlying vulnerability.

---

## 💡 Mitigation & Prevention

As a follow-up, the web development and security engineering teams should address the underlying vulnerability exposed by the `500` status code, which confirms the application is susceptible to SQL syntax errors.

* **Parameterization:** Implement **Prepared Statements (Parameterized Queries)** to ensure all user input is treated as data, not executable code.
* **Input Validation:** Use strict **Allow-listing** for input validation to reject malicious characters.
* **Error Handling:** Configure the application to present **generic error pages** instead of verbose database errors to prevent information leakage.
* **WAF (Web Application Firewall):** Ensure a WAF is in place with updated rules to block known SQLi signatures.
