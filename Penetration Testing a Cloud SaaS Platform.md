# Executive summary

Over two consecutive 2‑month internship projects at **Dotcom**, I worked on SOC 2 security compliance for a SaaS cloud platform: penetration testing and infrastructure hardening.

---

# Penetration Testing (2 months)

## 1) Recap

- Discovered and reported a high‑severity authentication vulnerability with PoC and remediation plan delivered to developers.  
- Conducted OSINT, network/service enumeration, and vulnerability assessments across APIs, containers, and web apps.  
- Performed manual and automated API fuzzing; uncovered abnormal behaviors, misconfigurations, and leaks of sensitive information/source code.  
- Tested password hashing strength and recommended improvements.  
- Assessed Docker/LXC containers for escape/privilege escalation risks and advised hardening.  
- Produced detailed vulnerability documentation and coordinated fixes with developers and leadership.

## 2) Goals

- Responsibly identify exploitable weaknesses in development and staging environments.  
- Produce actionable, prioritized remediation guidance to improve SOC 2 risk posture.

## 3) Methodology

- **Framework:** PDCA (Plan–Do–Check–Act): scoping & checklist → scanning & exploitation → validate PoCs → deliver remediation & retest.  
- **Test types:** Black‑box OSINT and enumeration, API & web app testing, authentication/JWT testing, fuzzing; Gray‑box configuration review.  
- **Engagement:** Black/Gray box scenarios and rules of engagement agreed with the board.

## 4) Findings

- Session invalidation flaw (JWT lifecycle) — sessions remained valid after logout in certain cases.  
- Services exposed version/debug info (Nginx and others), aiding exploit development.  
- API endpoints vulnerable to fuzzing — errors revealed sensitive data and source code.  
- Container image vulnerabilities and insecure config increased escape risk.

## 5) Tools

Proxmox, Kali Linux, Nmap, curl, Burp Suite, Microsoft Restler, Gobuster, sqlmap, hashcat, Grype, docker‑escape‑tool, SSH tools, Trivy, Greenbone/OpenVAS, Hydra, John the Ripper, hash‑identifier, jwtbreaker, jwt_checker, jwtcat, jwt.io, Caido, XSStrike, LinkFinder, Nikto, OWASP ZAP, wafw00f, Shodan, recon‑ng, censys.io, whatweb, wappalyzer, custom bash scripts.

## 6) Key activities

### a) OSINT
- Systematic collection of exposed services, assets, subdomains, and versions.  
- Built an attack surface inventory and prioritized targets.

### b) Active reconnaissance
- IP/scan techniques to evade flagging (rotation, masking, TOR) where appropriate under scope.  
- Repeated Nmap scans and version detection; firewall fingerprinting.

### c) API & web application testing
- Endpoint discovery (Gobuster, LinkFinder), automated + manual fuzzing (Burp, Restler, custom curl), and OWASP Top 10 testing (SQLi, XSS, SSRF, request smuggling…).
- Used sqlmap, XSStrike, Nikto, ZAP, Burp and custom tooling.

### d) Authentication & password hygiene
- Identified hashing algorithms and ran controlled cracking to assess strength (hashcat, John).  
- Performed controlled brute‑force tests on authorized endpoints to validate lockouts and rate limits (Hydra, John).

### e) Container / LXC testing (Proxmox VE)
- Reviewed architecture and rebuilt a local lab to mirror the environment.  
- Tested images/configs for escape and privilege escalation (Trivy, Grype, OpenVAS); recommended unprivileged runtimes, non‑root execution, read‑only filesystems.

### f) SSH & infrastructure checks
- Controlled SSH brute testing to validate password strength, rate limits, key usage and IP bans.

## 7) PoCs & mitigations delivered

- PoCs for session invalidation and fuzzing findings used to validate fixes on dev servers.  
- Recommendations and example configs for JWT handling, token revocation patterns, and session lifecycle controls.  
- Remediation steps for API input validation, parameterized DB access, and improved error handling.  
- CI/CD pipeline changes suggested to include image scanning (Grype) and image hardening.

## 8) Proven abilities

- End‑to‑end capability: offensive testing, defensive engineering changes, and verification of fixes.  
- Methodical, audit‑oriented approach aligned with PDCA and SOC 2.  
- Automation & scale: scripts and PoCs to automate fuzzing and repetitive tasks.  
- Continuous research: CVE monitoring and learning new components (e.g., Proxmox internals).  
- Strong communication: presented technical plans and outcomes to leadership and cross‑functional teams.  
- Collaborative & independent: worked with the board, devs, and independently to drive remediation.

---

# Infrastructure Hardening & SOC 2 (2 months)

## Recap

- Built SOC 2‑aligned security checklists and remediation roadmaps tailored to company infrastructure.  
- Hardened Nginx reverse proxies and PM2 (rate limiting, Fail2ban, secure headers, zero‑trust enforcement).  
- Secured Docker/LXC containers (unprivileged runtimes, non‑root execution, read‑only filesystems) and automated tasks via bash scripts.  
- Implemented Port Knocking to conceal services and reduce brute‑force exposure.  
- Tracked emerging CVEs from forums and integrated mitigations into the roadmap.  
- Presented risks, timelines, and remediation priorities to executives and engineers.

## Key activities

- Compiled SOC 2‑mapped checklists and prioritized remediation by risk/impact.  
- Implemented proxy and process hardening (Nginx, PM2) and enforced secure headers and rate limits.  
- Automated container hardening and scanning in CI pipelines; recommended moving privileged containers to unprivileged where feasible.  
- Deployed Port Knocking automation and validated reduced brute‑force exposure.  
- Maintained a CVE tracking practice and fed actionable items into the remediation roadmap.

## Outcome

- Improved defense-in-depth posture across reverse proxies, application processes, and container hosts.  
- Clear, SOC 2‑ready documentation and remediation plans for auditors and engineering teams.

---

*Document prepared by: [Your Name]*

