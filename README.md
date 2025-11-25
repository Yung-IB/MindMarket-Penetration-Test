# Penetration Test Report: MindMarket Application

**Author:** Olaniyi Ibrahim Abiodun  
**Date:** September 24, 2025
**Target Application:** [MindMarket](https://mindmarket-project.vercel.app/)

---

## 1. Executive Summary

I conducted a comprehensive penetration test against the MindMarket web application to identify and assess security vulnerabilities across its infrastructure and application layers. The assessment utilized a multi-phased methodology including network reconnaissance, passive traffic analysis, threat intelligence lookups, and in-depth manual application testing.

**Overall Risk Posture:** **Informational**.
**Key Findings:** No critical vulnerabilities were successfully exploited. However, the complexity of the back-end API and its authentication system presents a significant area for potential future vulnerabilities.
**Primary Recommendation:** Security efforts should focus on a rigorous, ongoing assessment of the back-end API.

---

## 2. Methodology and Scope

### Scope
The assessment covered the following assets:
* **Front-end:** `https://mindmarket-project.vercel.app/`.
* **Back-end API:** `https://mind-market-api.onrender.com/` (Discovered during testing).

### Tools & Techniques
1.  **Network Reconnaissance (Nmap):** To identify the network footprint, open ports, and service versions.
2.  **Passive Analysis (Wireshark):** To monitor traffic for anomalies and verify encryption.
3.  **Application Mapping (Burp Suite):** To map structure, analyze API endpoints, and test for flaws like injection and XSS.
4.  **Threat Intelligence (VirusTotal):** To scan public domains for reputation.

---

## 3. Technical Findings

### 3.1 Network Reconnaissance (Nmap)
I performed multiple scan types (`-Pn`, `-sC`, `-sL`, `-sV`, `-vv`) to analyze the host.
<img width="1919" height="839" alt="-Pn scan" src="https://github.com/user-attachments/assets/4503e7f6-580b-4b81-9048-d92b7cfe8b03" />
<img width="1913" height="1020" alt="-sC scan" src="https://github.com/user-attachments/assets/ee93106c-6b8d-4948-aabd-b1673a7b52a4" />
<img width="1176" height="400" alt="-sL scan" src="https://github.com/user-attachments/assets/7a21c3d2-9266-486b-b408-fd55aaa6c9b1" />
<img width="1919" height="1026" alt="-sV scan" src="https://github.com/user-attachments/assets/33f36b1b-276e-435b-8adc-32bc14c4f45d" />
<img width="1919" height="1023" alt="-v -v scan" src="https://github.com/user-attachments/assets/afe1aef0-6e12-4a78-8f6a-6e6a9ce502a0" />

**Port Scanning:** Identified four open TCP ports: **21 (FTP)**, **80 (HTTP)**, **443 (HTTPS)**, and **587 (Submission)**.
**Firewall Status:** 996 ports were reported as "filtered," confirming a robust firewall is in place.
**Service Detection:**
      **Port 21:** Identified as **ProFTPD**.
      **Port 80:** Identified as **CollabNet Git http server**.
      **TTL Analysis:** TTL values (64 vs 113/114) suggest a mix of operating systems (Linux and Windows/Network devices).

> **Recommendation:** Verify that the open ports (FTP/SMTP) are essential. Perform targeted vulnerability research on the specific ProFTPD version.

### 3.2 Application Architecture (Burp Suite)
Burp Suite revealed a segregated architecture where the Vercel host serves the UI, but critical functions are handled by a separate back-end API.
<img width="1720" height="905" alt="Burpsuite sitemap 2" src="https://github.com/user-attachments/assets/6b2a0c41-dc9c-46db-b986-0da320b633c4" />

* **Front-end:** `mindmarket-project.vercel.app`
* **Back-end:** `mind-market-api.onrender.com` (Primary attack surface).

### 3.3 Authentication Mechanism (Burp Suite)
The application uses **JSON Web Tokens (JWT)** for stateless authentication.
<img width="1720" height="902" alt="Sitemap sighin" src="https://github.com/user-attachments/assets/039e18fe-8468-4ee1-b76e-a3902fbafaeb" />
<img width="1718" height="902" alt="Burpsuite sitemap" src="https://github.com/user-attachments/assets/57921c9d-6365-42d7-a43a-e1af8765f451" />

* **Evidence:** A successful login POST request returns a JSON body containing a "token" string.
* **Risk:** While standard, JWTs require testing for signature validation and broken access control.

> **Recommendation:** Decode tokens to check for sensitive data and test for IDOR vulnerabilities.

### 3.4 Server Response Handling (Burp Suite)
I analyzed the server's response handling for both valid and invalid resources on the front-end to ensure no information leakage occurred.
<img width="1716" height="900" alt="Repeater" src="https://github.com/user-attachments/assets/435001c5-c0d8-450d-8e1e-0e59fd25c026" />
<img width="1719" height="898" alt="Repeater error" src="https://github.com/user-attachments/assets/bb01d3ac-7ee0-4cb3-9968-06caa3ff6555" />

* **Evidence:**
    * A GET request to the root (`/`) correctly returns a **200 OK** response.
    * A GET request to a non-existent path (`/sign-up.html`) correctly returns a standard **404 Not Found** error page.
* **Analysis:** The server handles requests for non-existent pages gracefully. It does not leak sensitive information like file paths or server diagnostics.

> **Status:** **Informational** (No remediation required).


### 3.5 Traffic Analysis (Wireshark)
Passive analysis confirmed that communication between the client and server is conducted over **TLSv1.2**.
<img width="1919" height="1078" alt="Wireshark 3" src="https://github.com/user-attachments/assets/932a73a3-8566-4eef-8e9b-2deb953c835b" />
<img width="1919" height="1078" alt="Wireshark 2" src="https://github.com/user-attachments/assets/31177a08-77f0-4451-8f86-e4d221b58f88" />
<img width="1919" height="1078" alt="Wireshark 4" src="https://github.com/user-attachments/assets/217f07a2-773f-46f4-9bb8-39d93b65be51" />
<img width="1919" height="1078" alt="Wireshark" src="https://github.com/user-attachments/assets/e30b19d6-dda6-4f6c-bf36-6f6fe134f5bb" />

* **Security:** Application data is encapsulated within encrypted records, protecting against Man-in-the-Middle (MitM) attacks.

### 3.6 Threat Intelligence (VirusTotal)
<img width="1919" height="977" alt="Virustotal Score" src="https://github.com/user-attachments/assets/b0afefc2-d9ed-4b61-a8cf-c9e2f9476735" />
<img width="1917" height="874" alt="Virustotal Details" src="https://github.com/user-attachments/assets/c5fdc5cb-25ff-4340-bcb3-99439a1a50a1" />

* **Detection Score:** **0/98** (Clean).
* **Reputation:** Domains are categorized as "information technology" and are not associated with malicious activity.

---

## 4. Conclusion and Recommendations

The MindMarket application exhibits a strong security posture at the network and transport layers, with a hardened perimeter and proper encryption. The most significant discovery was the segregated back-end API (`mind-market-api.onrender.com`).

### Final Recommendations
Future security efforts should be prioritized on a deep-dive, manual penetration test of the **back-end API**. This should include:
1.  **Broken Access Control (IDOR)**.
2.  **JWT Implementation Flaws**.
3.  **Injection Vulnerabilities**.

---

*Disclaimer: This penetration test was conducted for educational and assessment purposes. The authors are not responsible for misuse of this information.*
