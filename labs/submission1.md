# Triage Report — OWASP Juice Shop

## Scope & Asset
- Asset: OWASP Juice Shop (local lab instance)
- Image: bkimminich/juice-shop:v19.0.0
- Release link/date: https://github.com/juice-shop/juice-shop/releases/tag/v19.0.0 — 2023-10-xx
- Image digest (optional): <sha256:2765a26de7647609099a338d5b7f61085d95903c8703bb70f03fcc4b12f0818d>

## Environment
- Host OS: Windows (amd64)
- Docker Desktop: 4.47.0 (206054)
- Docker Engine: 28.4.0
- Container OS: linux/amd64 (Docker Desktop VM)

## Deployment Details
- Run command used:

```bash
docker run -d --name juice-shop \
  -p 127.0.0.1:3000:3000 \
  bkimminich/juice-shop:v19.0.0
```

- Access URL: http://127.0.0.1:3000
- Network exposure: 127.0.0.1 only  
  - [x] Yes  
  - [ ] No

---

## Health Check

### Page load
- Juice Shop homepage loads successfully in browser.


Screenshot evidence:
![juice-shop-home.png](screenshots%2Fjuice-shop-home.png)

```text
labs/screenshots/juice-shop-home.png
```

---

### API check

Command used:

```bash
curl -s http://127.0.0.1:3000/api/Products
```

Output (first lines):

```json
{"status":"success","data":[
  {"id":1,"name":"Apple Juice (1000ml)","description":"The all-time classic.","price":1.99},
  {"id":2,"name":"Orange Juice (1000ml)","description":"Made from oranges hand-picked by Uncle Dittmeyer.","price":2.99},
  {"id":3,"name":"Eggfruit Juice (500ml)","description":"Now with even more exotic flavour.","price":8.99}
]}
```

This confirms the REST API is responding correctly.

---

## Surface Snapshot (Triage)

- Login/Registration visible:  
  - [x] Yes  
  - [ ] No  
  Notes: Login button and registration link appear on the landing page.

- Product listing/search present:  
  - [x] Yes  
  - [ ] No  
  Notes: Product catalog is visible immediately after loading the app.

- Admin or account area discoverable:  
  - [x] Yes  
  - [ ] No  
  Notes: Account menu exists; admin features may be hidden behind authentication.

- Client-side errors in console:  
  - [ ] Yes  
  - [x] No  
  Notes: No critical JavaScript errors observed during initial triage.

- Security headers (quick look — optional):

Command:

```bash
curl -I http://127.0.0.1:3000
```

Notes (observed headers):
- X-Content-Type-Options: nosniff (present)
- X-Frame-Options: SAMEORIGIN (present)
- HSTS: not present (expected for HTTP)
- CSP: not present in response
- Access-Control-Allow-Origin: * (very permissive)

---

## Risks Observed (Top 3)

1) **Intentionally vulnerable application**
   - Juice Shop contains OWASP Top 10 vulnerabilities by design (SQLi, XSS, auth flaws).

2) **No HTTPS/TLS in default deployment**
   - Traffic is unencrypted, which would allow interception in real deployments.

3) **Potential exposure if port binding changes**
   - If deployed with `-p 3000:3000` instead of localhost binding, the service becomes publicly reachable.