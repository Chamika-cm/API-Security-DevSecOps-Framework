# OWASP crAPI - STRIDE Threat Model & Security Risk Register

## 1. Overview & Methodology
This document defines the baseline security threat model for the OWASP crAPI application using the **STRIDE Methodology** (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, and Elevation of Privilege). Threats are mapped against identified system trust boundaries and assigned risk levels aligned with the project proposal scope.

---

## 2. STRIDE Threat Analysis Matrix

| ID | STRIDE Category | Target Component | Threat Scenario / Vulnerability | Risk Level | Mitigation Strategy / Controls |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **TM-01** | **Spoofing** | Identity Microservice | Weak JWT signature verification or weak signing secrets allowing attacker to forge tokens or spoof user identity. | **High** | Enforce explicit algorithm allowlists, signature/claim validation, and strong secret management via HashiCorp Vault. |
| **TM-02** | **Tampering** | Community API / Orders | Mass Assignment / API binding vulnerabilities allowing untrusted properties to mutate internal models. | **High** | Implement request DTOs, field allowlists, and reject unauthorized protected property updates. |
| **TM-03** | **Repudiation** | Database & App Layers | Insufficient API access logging for critical transactions (e.g., balance updates, password resets). | **Medium** | Enforce centralized security audit logging and forward container/app logs to Wazuh SIEM. |
| **TM-04** | **Information Disclosure** | Web Proxy / Vehicle API | Broken Object Level Authorization (BOLA) exposing other users' vehicle or private data endpoints. | **Critical** | Implement owner-scoped server-side authorization checks and test with separate user accounts. |
| **TM-05** | **Denial of Service** | API Gateway / Microservices | Lack of rate limiting or expensive nested queries causing resource exhaustion and unavailability. | **High** | Configure Nginx/ModSecurity rate limiting rules, timeout controls, and API gateway throttling. |
| **TM-06** | **Elevation of Privilege** | Identity & Admin Endpoints | Broken Function Level Authorization (BFLA) allowing normal users to invoke administrative or mechanic functions. | **Critical** | Enforce strict server-side role/function authorization checks and validate each role against privileged endpoints. |

---

## 3. Additional Project Security Threats Assessed

| Threat ID | Threat Category | Description & Attack Scenario | Potential Impact | Recommended Controls |
| :--- | :--- | :--- | :--- | :--- |
| **TM-07** | **SSRF** | A service fetching user-supplied URLs may request internal infrastructure or cloud metadata endpoints. | Internal network access and credential exposure. | Apply destination allowlists, DNS/IP validation, and egress restrictions. |
| **TM-08** | **Excessive Data Exposure** | API responses returning unneeded sensitive account fields or PII to the client. | Privacy impact and credential leakage. | Use explicit response schemas and field allowlists. |
| **TM-09** | **Race Conditions** | Concurrent requests exploiting non-atomic checks in booking, inventory, or coupon workflows. | Duplicate actions, inconsistent state, or business logic abuse. | Use atomic database operations, transactions, locking mechanisms, and bounded concurrency tests. |
| **TM-10** | **Container Privileges** | Root execution, privileged modes, or dangerous capabilities increasing compromise impact. | Potential host compromise and lateral movement across services. | Run containers as non-root, drop unnecessary capabilities, and scan configurations using Checkov/OPA. |

---

## 4. Remediation Planning Roadmap
- **Week 1:** Complete baseline trust boundaries, architecture mapping, and finalize the STRIDE threat model.
- **Week 2:** Implement automated Pytest checks, OWASP ZAP DAST scans, and Nginx/ModSecurity WAF rules to detect and block BOLA, BFLA, and injection flaws.
- **Week 3:** Configure HashiCorp Vault for secrets management, generate SBOMs via Syft, scan artifacts using Trivy, and enforce OPA policies.
- **Week 4:** Integrate Wazuh log forwarding, establish GitHub Actions security gates, and validate final remediation.