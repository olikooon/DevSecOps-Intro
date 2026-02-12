# Lab 2 — Threat Modeling with Threagile

## Task 1 — Baseline Threat Model

**Objective:** Generate a baseline threat model for OWASP Juice Shop and analyze the risk posture.

**Generated Baseline Outputs:**  
- `labs/lab2/baseline/report.pdf` — full PDF report including diagrams  
- `labs/lab2/baseline/risks.json` — exported risk list  
- `labs/lab2/baseline/stats.json`, `technical-assets.json` — additional metadata  
- Data-flow & data-asset diagrams (PNG)

### Top 5 Risks (Baseline)

| Severity | Category | Asset | Likelihood | Impact | Composite Score       |
|----------|---------|-------|-----------|--------|-----------------------|
| Elevated | Cross-Site Scripting | Juice Shop Application | Likely (3) | Medium (2) | 332|
| Elevated | Missing Authentication | Juice Shop Application | Likely (3) | Medium (2) | 332|
| Elevated | Missing Authentication Second Factor | Direct/Reverse Proxy | Unlikely (2) | Medium (2) | 422|
| Medium | Server-Side Request Forgery | Juice Shop Application | Likely (3) | Low (1) | 331 |
| Medium | Unencrypted Communication | Direct to App | Likely (3) | High (3) | 243 |

**Analysis:**  
- The baseline model highlights several critical web application risks, including XSS and missing authentication protections.  
- Communication with the app over HTTP (unencrypted) and unencrypted storage of sensitive data is evident.  
- Diagrams show trust boundaries from **Internet → Host → Container Network** with user browsers and optional reverse proxy.

---

## Task 2 — Secure Variant & Risk Comparison

**Objective:** Apply security controls to reduce risk in the model and compare results.

### Secure Model Changes

1. User Browser → Direct to App: set `protocol: https`  
2. Reverse Proxy → all communication links: set `protocol: https`  
3. Persistent Storage: set `encryption: transparent`  

**Secure variant saved as:** `labs/lab2/threagile-model.secure.yaml`  

**Generated Secure Outputs:**  
- `labs/lab2/secure/report.pdf`  
- `labs/lab2/secure/risks.json`  
- Diagrams (PNG) and metadata

---

### Risk Category Delta Table (Baseline vs Secure)

| Category | Baseline | Secure | Δ |
|---|---:|---:|---:|
| container-baseimage-backdooring | 1 | 1 | 0 |
| cross-site-request-forgery | 2 | 2 | 0 |
| cross-site-scripting | 1 | 1 | 0 |
| missing-authentication | 1 | 1 | 0 |
| missing-authentication-second-factor | 2 | 2 | 0 |
| missing-build-infrastructure | 1 | 1 | 0 |
| missing-hardening | 2 | 2 | 0 |
| missing-identity-store | 1 | 1 | 0 |
| missing-vault | 1 | 1 | 0 |
| missing-waf | 1 | 1 | 0 |
| server-side-request-forgery | 2 | 2 | 0 |
| unencrypted-asset | 2 | 1 | -1 |
| unencrypted-communication | 2 | 0 | -2 |
| unnecessary-data-transfer | 2 | 2 | 0 |
| unnecessary-technical-asset | 2 | 2 | 0 |

---

### Delta Run Explanation

**Specific changes made:**

- Enforced HTTPS on all user-to-app and proxy-to-app communications  
- Applied transparent encryption to persistent storage  

**Observed results:**

- **Unencrypted communication** risk dropped from 2 → 0 (Δ = -2)  
- **Unencrypted asset** risk dropped from 2 → 1 (Δ = -1)  
- Other categories remain unchanged since no structural changes were made  

**Analysis:**

- Enforcing HTTPS mitigates risks associated with credentials and session data in transit.  
- Encrypting persistent storage reduces exposure of sensitive data if the storage medium is compromised.  
- The risk landscape shows how simple technical controls significantly reduce critical attack surfaces without affecting application functionality.  
- Diagrams comparing baseline vs secure clearly show encrypted communication links and encrypted storage symbols.
