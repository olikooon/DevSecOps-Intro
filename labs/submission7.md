## Task 1 — Image Vulnerability & Configuration Analysis

### 1.1 Vulnerability Scanning — Docker Scout

**Summary:**

- 118 vulnerabilities in 48 packages  
- CRITICAL: 11  
- HIGH: 65  
- MEDIUM: 30  
- LOW: 5  

---

### 1.2 Top Critical/High Vulnerabilities (Short)

**1. CVE-2026-22709 — `vm2` (CRITICAL)**  
- Sandbox bypass → attacker can run code on host  
- **Impact:** full Remote Code Execution (RCE)

**2. CVE-2023-37903 — `vm2` (CRITICAL)**  
- OS command execution via crafted input  
- **Impact:** run system commands, no fix available

**3. CVE-2023-37466 — `vm2` (CRITICAL)**  
- Code injection via async flaws  
- **Impact:** full process compromise

**4. CVE-2023-46233 — `crypto-js` (CRITICAL)**  
- Weak crypto (uses MD5 instead of SHA-256)  
- **Impact:** passwords easier to crack

**5. CVE-2021-44906 — `minimist` (CRITICAL)**  
- Prototype pollution  
- **Impact:** bypass auth, change app behavior

---

### 1.3 Snyk Comparison (Short)

- OS layer: 6 issues (Node.js, OpenSSL)  
- npm layer: 47 issues  

**Important findings:**
- `vm2` → RCE (no safe fix)
- `marsdb` → code injection
- `multer` → crashes & leaks
- `lodash` → prototype pollution
- `jsonwebtoken` → auth bypass
- `sequelize` → SQL injection

**Comparison:**
- Both tools found same critical issues  
- **Snyk:** better dependency tracing  
- **Docker Scout:** better CVE details  

---

### 1.4 Dockle Configuration

**Findings (simplified):**

- No HEALTHCHECK → container health not monitored  
- No Docker Content Trust → image integrity not verified  
- Extra `.DS_Store` files → minor info leak  
- Password check skipped (normal for this image)

**Good:**
- Runs as non-root (`UID 65532`)

---

### 1.5 Security Assessment

**Runs as root?**  
→ No (good)

**Main improvements:**
1. Replace or remove `vm2` (critical risk)
2. Update `crypto-js`
3. Update `lodash`
4. Update `jsonwebtoken`
5. Add `HEALTHCHECK`
6. Enable Docker Content Trust
7. Remove unnecessary files
8. Update Node.js version

---

## Task 2 — Docker Host Security Benchmarking

### 2.1 Result

- Tool failed to run (Docker Desktop limitation)

**Reason:**
- Uses Windows + Docker Desktop  
- No access to `/var/run/docker.sock` like Linux

---

### 2.2 Expected Results (Summary)

**Good:**
- Container runs as non-root → PASS  
- No privileged containers → PASS  

**Problems:**
- No HEALTHCHECK → FAIL  
- No `no-new-privileges` (in default) → FAIL  
- Weak network isolation → WARN  
- No AppArmor → WARN  

---

## Task 3 — Deployment Security Analysis

### 3.1 Key Differences (Short)

| Setting | Default | Hardened | Production |
|--------|--------|---------|-----------|
| Capabilities | full | none | minimal |
| Memory/CPU | unlimited | limited | limited |
| PIDs | unlimited | unlimited | 100 |
| Restart | no | no | on-failure |
| Security flags | none | basic | strongest |

---

### 3.2 Security Flags (Simple)

**a) `--cap-drop=ALL`**
- Removes all privileges  
- Prevents privilege escalation  
- `NET_BIND_SERVICE` added for low ports  

**b) `no-new-privileges`**
- Blocks setuid privilege escalation  
- Stops attacker from gaining higher rights  

**c) Memory & CPU limits**
- Prevent DoS via resource exhaustion  
- Protect host from crashes  

**d) `--pids-limit=100`**
- Stops fork bombs  
- Limits number of processes  

**e) Restart policy**
- Restarts on crash (max 3 times)  
- Improves reliability  

---

### 3.3 Answers


**1. Which profile for DEVELOPMENT? Why?**

The **Default profile** (`juice-default`) is appropriate for development. Developers often need to execute arbitrary commands inside the container for debugging (`docker exec`), access system tools, and test features that may temporarily require elevated permissions. Resource limits and strict capability restrictions slow down iteration and can produce confusing failures that obscure the actual bug being investigated. Security hardening should be introduced gradually as the application moves toward staging and production.

**2. Which profile for PRODUCTION? Why?**

The **Production profile** (`juice-production`) should be used. It applies the principle of least privilege across multiple layers: all capabilities are dropped except those strictly needed, privilege escalation via setuid is blocked, memory and CPU are bounded to prevent resource exhaustion and DoS, the PID limit prevents fork bombs, and the restart policy ensures resilience against transient crashes while limiting the blast radius of persistent failures. The `docker inspect` output confirms all these settings are active and the container runs successfully (`"Status": "running"`).

**3. What real-world problem do resource limits solve?**

Resource limits solve the **noisy neighbor** and **denial-of-service containment** problems. In a shared Kubernetes cluster or a multi-service VM, a single misbehaving container (due to a bug, traffic spike, or deliberate attack) can starve other containers of CPU and memory. Resource limits enforce isolation: each service gets a bounded share of host resources, and a breach in one container cannot cascade into a full host failure. This is fundamental to the blast radius containment principle in production deployments.

**4. If an attacker exploits Default vs. Production, what actions are blocked in Production?**

| Attack Action | juice-default | juice-production |
|--------------|--------------|-----------------|
| Fork bomb / spawn unlimited processes | Allowed | Blocked (`--pids-limit=100`) |
| Escalate via setuid binary (`su`, `passwd`) | Allowed | Blocked (`no-new-privileges`) |
| ARP spoofing / raw packet injection | Allowed (`CAP_NET_RAW` retained) | Blocked (`--cap-drop=ALL`) |
| Mount filesystems | Allowed (`CAP_SYS_ADMIN` retained) | Blocked |
| Load kernel modules | Allowed (`CAP_SYS_MODULE` retained) | Blocked |
| Exhaust host memory | Allowed (no limit) | Blocked (512 MB cap) |
| Persistent crash loop | Container stays dead | Restarts up to 3 times, then stops |

In the default profile, a successful exploit inside the container can quickly lead to host-level compromise. In the production profile, the attacker is constrained to the application's process space with no viable path to privilege escalation.

**5. What additional hardening would you add?**

- Read-only filesystem  
- User namespace mapping  
- Seccomp profile  
- AppArmor/SELinux  
- Network isolation  
- Image signing  
- Minimal base image  