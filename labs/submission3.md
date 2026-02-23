## Task 1 — SSH Commit Signature Verification

### Summary: Benefits of Signing Commits for Security

Commit signing ensures that every commit in a repository can be verified as coming from a trusted author and that its contents have not been altered. By using SSH commit signing, developers create cryptographic proof of authorship and integrity. This helps prevent impersonation, unauthorized code injection, and supply chain attacks. Signed commits increase trust in collaboration, especially in distributed teams and open-source projects.

### Evidence of Successful SSH Key Setup and Configuration

An SSH key was generated using the Ed25519 algorithm and added to the Git hosting platform. Git was configured to use SSH signing by setting the signing key, enabling automatic signing for commits, and specifying the SSH format. A signed commit was created using the -S flag, and the repository platform displayed the commit as verified, confirming correct configuration.

Configuration steps included:

* Generating an SSH key pair
* Adding the public key to the Git hosting account
* Configuring Git (user.signingkey, commit.gpgSign, gpg.format)
* Creating and pushing a signed commit

### Analysis: Why Commit Signing Is Critical in DevSecOps Workflows

Commit signing is essential in DevSecOps because it establishes trust across the software delivery pipeline. It ensures that code changes originate from authenticated developers and have not been tampered with. This reduces the risk of malicious code entering the codebase, supports auditability, and strengthens supply chain security. Signed commits also enable automated policy enforcement in CI/CD pipelines, where only verified changes may be merged or deployed.
