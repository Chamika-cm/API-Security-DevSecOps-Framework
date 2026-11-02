# STRIDE Threat Model & Risk Register - OWASP crAPI

## 1. Methodology
The Microsoft STRIDE framework is applied to identify, categorize, and mitigate security threats across the microservices architecture and trust boundaries of OWASP crAPI.

## 2. Risk Assessment Matrix
Risks are evaluated using a qualitative Likelihood x Impact matrix, categorized into Low, Medium, High, and Critical levels.

## 3. Top 10 Advanced Threat Register & Control Mapping

| ID | Threat Category (STRIDE) | Attack Scenario / Description | Potential Impact | Planned Security Control & Location |
| :--- | :--- | :--- | :--- | :--- |
| **TR-01** | Tampering / Authorization | **BOLA (Broken Object Level Authorization):** Manipulating object IDs in API requests to access another user's data. | Data Disclosure / Unauthorized Access | Implement owner-scoped server-side authorization checks in microservices. |
| **TR-02** | Elevation of Privilege | **BFLA (Broken Function Level Authorization):** Regular users invoking administrative endpoints without proper role verification. | Unauthorized Admin Actions | Enforce strict role-based access control (RBAC) middleware on admin routes. |
| **TR-03** | Spoofing | **Broken Authentication / JWT Flaws:** Forging JWT tokens via algorithm confusion (e.g., 'none' alg) or weak secrets. | Identity Impersonation / Full Takeover | Enforce explicit algorithm allowlists and robust cryptographic verification. |
| **TR-04** | Tampering | **Mass Assignment:** Modifying internal object properties (e.g., `is_admin: true`) via unbound API parameters. | Unauthorized Privilege Escalation | Use explicit request DTOs and allowlists to filter input parameters. |
| **TR-05** | Information Disclosure | **SSRF (Server-Side Request Forgery):** Forcing backend services to fetch malicious internal URLs or metadata. | Internal Network Exposure | Implement strict URL parsing, IP whitelisting, and egress filtering. |
| **TR-06** | Tampering / Injection | **NoSQL Injection:** Manipulating query logic via unfiltered user inputs in MongoDB endpoints. | Database Compromise / Data Leak | Use parameterized queries and ORM input sanitization. |
| **TR-07** | Information Disclosure | **Excessive Data Exposure:** API responses returning full database objects, exposing PII or sensitive keys. | Privacy Breach / Data Leakage | Define explicit response serialization schemas and field filtering. |
| **TR-08** | Denial of Service | **Unrestricted Resource Consumption (Rate Limiting):** High-volume automated requests overwhelming endpoints. | Service Unavailability (DoS) | Integrate rate-limiting and API gateway throttling middleware. |
| **TR-09** | Elevation of Privilege | **Container Misconfiguration:** Running microservice containers as root (UID 0) with elevated capabilities. | Host OS Compromise | Enforce non-root user execution (`USER 65532`) in Dockerfiles. |
| **TR-10** | Security Misconfiguration | **API Misconfigurations:** Exposed sensitive headers, improper CORS settings, or unnecessary HTTP methods. | Attack Surface Expansion | Enforce security headers, strict CORS policies, and automated hardening scripts. |