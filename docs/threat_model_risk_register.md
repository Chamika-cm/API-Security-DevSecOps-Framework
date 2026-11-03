# STRIDE Threat Model & Risk Register

This document provides a comprehensive security threat model and risk assessment for the OWASP crAPI (Completely Ridiculous API) platform using the **STRIDE** methodology.

---

## 📊 Risk Assessment Matrix

Risks are categorized based on **Likelihood** and **Impact** using a standard $3 \times 3$ rating scale:

$$\text{Risk Score} = \text{Likelihood (1–3)} \times \text{Impact (1–3)}$$

* **Critical (7–9):** Immediate remediation required.
* **High (5–6):** High-priority remediation needed.
* **Medium (3–4):** Moderate priority; mitigate or accept with controls.
* **Low (1–2):** Low priority; monitor or address during scheduled updates.

---

## 📋 Threat Register

| ID | Threat Name & STRIDE Category | Description / Attack Scenario | Likelihood | Impact | Risk Level | Justification | Mitigation & Pipeline Location |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **TR-01** | **BOLA** <br>*(Tampering / Info Disclosure)* | Changing object identifiers (e.g., vehicle ID, order ID) in API requests to access/modify another user's private records. | High (3) | High (3) | **Critical (9)** | crAPI endpoints often accept sequential IDs without validating ownership against the active JWT session. | Implement owner-scoped server-side authorization checks in controllers (`crapi-identity` / `crapi-community`). Tested via Python Pytest (`tests/api/test_bola.py`). |
| **TR-02** | **BFLA** <br>*(Elevation of Privilege)* | A regular user invoking administrative or mechanic endpoints (e.g., `/api/v2/admin/...`) due to missing role checks. | High (3) | High (3) | **Critical (9)** | Standard user tokens can be re-routed to privileged admin routes without proper role validation middleware. | Enforce explicit server-side role verification (RBAC) middleware on all administrative routes. Tested via Pytest (`tests/api/test_bfla.py`). |
| **TR-03** | **Broken Auth & JWT Flaws** <br>*(Spoofing)* | Weak JWT signing secrets, lack of claim validation, or accepting unsigned/forged tokens (e.g., `none` algorithm). | High (3) | High (3) | **Critical (9)** | Weak secrets allow attackers to forge valid administrative tokens, leading to full account takeover. | Enforce strict cryptographic algorithm allowlists, strong rotating secrets managed via HashiCorp Vault, and claim validation. |
| **TR-04** | **Mass Assignment** <br>*(Tampering)* | Unsanitized JSON request properties being bound directly to internal database models (e.g., adding `is_admin: true`). | Medium (2) | High (3) | **High (6)** | Framework auto-binding features permit clients to modify protected user properties during profile updates. | Use strict request Data Transfer Objects (DTOs) and field allowlists. Reject or ignore unrequested parameters. |
| **TR-05** | **SSRF** <br>*(Info Disclosure / Elevation)* | Forcing backend services (like the mechanic finder) to fetch user-supplied URLs pointing to internal metadata or localhost ports. | Medium (2) | High (3) | **High (6)** | The application fetches external URLs to check vehicle parts, which can be manipulated to scan internal networks. | Apply destination URL/IP allowlists, strict DNS validation, and block private IP ranges (RFC 1918) in HTTP client configurations. |
| **TR-06** | **NoSQL / SQL Injection** <br>*(Tampering / Info Disclosure)* | Manipulating input parameters in API queries to bypass authentication or extract sensitive database contents. | Medium (2) | High (3) | **High (6)** | Unsanitized inputs processed directly in database queries allow attackers to manipulate query execution logic. | Use parameterized queries, ORM sanitization, and strict input type-checking across all microservice endpoints. |
| **TR-07** | **Excessive Data Exposure** <br>*(Information Disclosure)* | API responses returning full database objects (including password hashes or internal tokens) to the frontend client. | High (3) | Medium (2) | **High (6)** | Backend serializes entire entity models instead of filtered response DTOs, exposing sensitive PII. | Implement explicit response schemas and field allowlists. Validate response payloads in CI pipeline regression tests. |
| **TR-08** | **Unrestricted Resource Consumption** <br>*(Denial of Service)* | Lack of rate limiting on login, password reset, or lookup endpoints, allowing brute-force or resource exhaustion. | Medium (2) | Medium (2) | **Medium (4)** | Unthrottled endpoints enable automated scripts to spam requests, causing application slowdown or downtime. | Integrate rate-limiting middleware (e.g., Nginx ModSecurity / API Gateway rate-limiting) on high-frequency endpoints. |
| **TR-09** | **Container Misconfiguration** <br>*(Elevation of Privilege)* | Microservice Docker containers executing processes as root (UID 0) or retaining dangerous Linux capabilities. | High (3) | Medium (2) | **High (6)** | Default container setups run as root, increasing host compromise impact if a container breakout occurs. | Configure non-root user execution (`USER 65532`), drop unnecessary capabilities, and scan via Trivy & OPA/Conftest gates. |
| **TR-10** | **API Misconfiguration** <br>*(Tampering)* | Weak CORS policies, missing security headers, or exposing undocumented debug endpoints to the public internet. | Medium (2) | Low (1) | **Medium (2)** | Improper header configurations permit cross-origin attacks or information leakage about the tech stack. | Enforce secure HTTP headers (CSP, HSTS, X-Content-Type-Options), strict CORS origins, and disable debug flags in production. |