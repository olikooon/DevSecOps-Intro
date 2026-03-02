# Lab 4 – SBOM and SCA Toolchain Comparison

## Target Application
Container image: OWASP Juice Shop

Tools evaluated:
- Syft (SBOM generation)
- Grype (vulnerability scanning)
- Trivy (all-in-one SBOM + SCA solution)

---

# 1. Accuracy Analysis

## 1.1 Package Detection Comparison

| Category | Count |
|----------|-------|
| Packages detected by both tools | 1126 |
| Packages only detected by Syft | 13 |
| Packages only detected by Trivy | 9 |

### Analysis

Package detection overlap between Syft and Trivy is very high.  
Out of the total detected packages, 1126 were identified by both tools.

Unique detections:
- Syft-only packages: 13
- Trivy-only packages: 9

This indicates:

- Strong consistency in dependency discovery
- Minor differences likely caused by:
  - Ecosystem-specific analyzers
  - Handling of base image layers
  - Transitive dependency parsing
  - Metadata normalization differences

Overall, package detection accuracy is comparable and reliable across both tools.

---

## 1.2 Vulnerability Detection Overlap

| Category | Count |
|----------|-------|
| CVEs found by Grype | 117 |
| CVEs found by Trivy | 116 |
| Common CVEs | 29 |

### Analysis

Although total vulnerability counts are similar (117 vs 116), the overlap is relatively low (29 shared CVEs).

This suggests:

- Different vulnerability databases
- Different matching logic (CPE vs package metadata)
- Differences in vendor advisory mapping
- Different handling of unfixed vulnerabilities

The low overlap indicates that using only one scanner may result in blind spots.

---

# 2. Tool Strengths and Weaknesses

## Syft + Grype

### Strengths

- Clear separation of SBOM generation and vulnerability scanning
- Modular pipeline design
- SBOM can be reused across multiple scanners
- Strong compatibility with compliance workflows
- Good transparency in vulnerability matching

### Weaknesses

- Requires managing two tools instead of one
- Slightly more complex CI/CD setup
- No built-in secret scanning
- No integrated license compliance scanning

---

## Trivy (All-in-One)

### Strengths

- Single tool for SBOM, vulnerabilities, licenses, and secrets
- Easy CI/CD integration
- Minimal configuration required
- Broader ecosystem coverage
- Faster initial setup

### Weaknesses

- Less modular
- SBOM tightly coupled to scanning workflow
- Harder to swap vulnerability engines independently
- May abstract some detection logic

---

# 3. Use Case Recommendations

## When to Choose Syft + Grype

- Enterprise environments requiring SBOM reuse
- Regulatory compliance (SBOM export, archiving)
- Multi-scanner validation workflows
- Air-gapped or offline scanning environments
- Advanced supply chain security pipelines

## When to Choose Trivy

- Small to medium teams
- Quick CI/CD integration
- DevSecOps pipelines requiring simplicity
- Projects needing integrated secrets + license scanning
- Cloud-native container scanning

## High-Security Environments

For critical production systems, combining tools is recommended due to vulnerability database differences and low CVE overlap.

---

# 4. Integration Considerations

## CI/CD Integration

### Syft + Grype

- Requires two pipeline steps:
  1. Generate SBOM (Syft)
  2. Scan SBOM (Grype)
- More flexible for artifact storage
- SBOM can be archived and reused
- Better for long-term auditability

### Trivy

- Single pipeline step
- Faster onboarding
- Built-in support for:
  - Vulnerability scanning
  - License compliance
  - Secret detection
- Easier GitHub Actions / GitLab CI integration

---

## Operational Aspects

| Factor | Syft + Grype | Trivy |
|--------|--------------|--------|
| Setup complexity | Medium | Low |
| Modularity | High | Low |
| SBOM reuse | Yes | Limited |
| Secrets scanning | No | Yes |
| License scanning | Limited | Yes |
| Maintenance overhead | Moderate | Low |

---

# Final Conclusion

- Package detection accuracy between tools is very high.
- Vulnerability detection overlap is low, indicating complementary coverage.
- Trivy is ideal for simplicity and fast DevSecOps integration.
- Syft + Grype is better suited for modular, compliance-driven environments.
- For maximum security visibility, multi-scanner validation is recommended.

Overall, both toolchains are production-ready, but selection should depend on organizational maturity, compliance requirements, and pipeline architecture.