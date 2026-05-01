# ML Sec Ops 🍃

> Practical sec lab repository focused on WebAppSec, API Security, secure coding, container hardening and vulnerability analysis.

## 📍 Overview

`ML Sec Ops` is a hands-on security engineering repository that documents a sequence of laboratory works around modern application security and DevSecOps practices.

The repository is focused on practical security analysis rather than abstract theory: vulnerability research, CVE triage, exploit reproduction, insecure API behavior, secure coding review, Docker container hardening and image vulnerability scanning.

The main goal is to show a clear security workflow:

⬩ identify the attack surface  
⬩ reproduce vulnerable behavior in a controlled local environment  
⬩ map findings to OWASP categories  
⬩ document impact and root cause  
⬩ propose mitigation and hardening steps  

## 🔸 Main Security Areas

- Web application vulnerabilities
- SQL Injection analysis
- REST API security testing
- OWASP API Security Top 10 mapping
- Broken Object Level Authorization / IDOR
- Broken Object Property Level Authorization
- Debug endpoint exposure
- Stored Cross-Site Scripting
- Secure coding review for Python web apps
- Django security model and session handling
- Docker container hardening
- Non-root container runtime configuration
- Vulnerability scanning with Trivy
- CVE research and vulnerability classification

## 🔹 Repo Structure

```text
ML-Sec-Ops/
├── 1 lab/
│   └── Web vulnerability research and SQL injection practice
│
├── 2 lab/
│   └── REST API security, OWASP API Top 10 and vulnerable API analysis
│
├── 3 lab/
│   └── XSS research, PHP CVEs and Stored XSS exploitation
│
├── 4 lab/
│   └── Secure coding review, Flask issues and Django security model
│
├── 5 lab/
│   └── Docker hardening, non-root containers and Trivy vulnerability scan
│
└── README.md
```

## 🔸 Lab Index

### 🔹 Lab 1 — Web Vulnerabilities and SQL Injection

The first lab focuses on the role of web vulnerabilities in real-world compromise scenarios and includes practical SQL Injection testing.

Key points:

- research of web vulnerability statistics from security reports;
- OWASP Top 10 category analysis;
- manual SQL Injection probing;
- database enumeration through vulnerable query parameters;
- extraction of sensitive data through UNION-based SQLi;
- documentation of exploitation flow and final findings.

Main security topics:

`SQLi`, `OWASP Top 10`, `web attack surface`, `database enumeration`, `input validation`, `query injection`.

---

### 🔹 Lab 2 — REST API Security and OWASP API Top 10

The second lab is focused on REST API security and API-specific weakness categories.

It includes CVE research related to REST API flaws and practical analysis of a vulnerable Dockerized API application.

Covered findings:

- public endpoint inventory exposure;
- unauthenticated access to user data;
- object-level authorization failure;
- debug endpoint exposure;
- sensitive data disclosure through API responses;
- mapping findings to OWASP API Security Top 10.

Main security topics:

`REST API`, `BOLA`, `IDOR`, `Broken Object Property Level Authorization`, `Security Misconfiguration`, `Improper Inventory Management`, `debug endpoint`, `authorization bypass`.

---

### 🔹 Lab 3 — XSS Research and Stored XSS Exploitation

The third lab focuses on Cross-Site Scripting, especially Stored XSS in PHP-based components and a local vulnerable application.

The practical part demonstrates how user-controlled input is stored and later rendered without proper output encoding, which allows browser-side JavaScript execution.

Covered work:

- CVE research for PHP components affected by XSS;
- analysis of vulnerable parameters and affected functionality;
- local Stored XSS reproduction;
- payload execution through a comment field;
- discussion of impact and prevention techniques.

Main security topics:

`Stored XSS`, `output encoding`, `HTML escaping`, `client-side code execution`, `payload persistence`, `Content Security Policy`.

---

### 🔹 Lab 4 — Secure Coding and Django Security Model

The fourth lab is focused on secure coding review for Python web applications and built-in security mechanisms in Django.

The code review part identifies common implementation mistakes:

- hardcoded Flask `SECRET_KEY`;
- insecure password field handling;
- missing validation;
- plaintext password storage;
- incorrect exception handling;
- missing error logging.

The Django part covers framework-level protections:

- XSS protection;
- CSRF protection;
- SQL Injection protection through ORM usage;
- clickjacking protection;
- Host header validation;
- session middleware;
- secure session cookie settings.

Main security topics:

`secure coding`, `secret management`, `password hashing`, `CSRF`, `Django sessions`, `HttpOnly`, `Secure`, `SameSite`, `ORM safety`.

---

### 🔹 Lab 5 — Docker Container Hardening and Trivy Scan

The fifth lab focuses on basic Docker container security and vulnerability scanning.

The original container was checked with `whoami` and was found to run as `root`. A hardened image was then created with a dedicated non-root user and safer runtime recommendations.

Covered work:

- Docker image inspection;
- root vs non-root container runtime check;
- custom Dockerfile creation;
- non-root user configuration;
- runtime hardening flags;
- Trivy vulnerability scan;
- analysis of HIGH and CRITICAL findings;
- security conclusions for the selected base image and dependencies.

Main security topics:

`Docker hardening`, `non-root container`, `least privilege`, `Trivy`, `CVE scanning`, `Alpine packages`, `Python dependencies`, `container runtime security`.

## 🔸 Technical Stack

- Docker
- Python
- Flask
- Django
- REST API
- SQL
- Linux shell
- Trivy
- NVD CVE database
- OWASP Top 10
- OWASP API Security Top 10
- Git / GitHub

## 🔹 Security Workflow

The labs follow a practical security workflow:

```text
1. Recon / Research
   └── collect CVE data, reports, OWASP categories and known weakness patterns

2. Local Environment Setup
   └── run vulnerable apps in Docker containers

3. Manual Testing
   └── interact with endpoints, parameters, forms and runtime behavior

4. Exploit Reproduction
   └── validate SQLi, XSS, BOLA / IDOR and misconfiguration issues

5. Root Cause Analysis
   └── identify why the vulnerability exists at code, config or architecture level

6. Mitigation
   └── document secure alternatives, access control, encoding, validation and hardening

7. Security Reporting
   └── summarize findings, impact and remediation steps
```

## 🔸 Example Findings


| Area            | Example Finding                                        | Security Category           |
| --------------- | ------------------------------------------------------ | --------------------------- |
| Web Application | SQL Injection through unsanitized request parameter    | Injection                   |
| REST API        | User profile access without object-level authorization | BOLA / IDOR                 |
| REST API        | Public debug endpoint exposing sensitive user data     | Security Misconfiguration   |
| XSS             | Stored JavaScript payload executed from saved comment  | Stored XSS                  |
| Secure Coding   | Hardcoded secret key in Flask application code         | Secret Management Failure   |
| Secure Coding   | Plaintext password handling                            | Insecure Credential Storage |
| Docker          | Container running as root                              | Least Privilege Violation   |
| Docker          | Vulnerable packages detected by Trivy                  | Vulnerable Dependencies     |


## 🔹 Hardening Notes

Security improvements documented across the labs include:

⬩ validate and normalize user-controlled input
⬩ avoid raw SQL string concatenation
⬩ use parameterized queries or ORM abstractions
⬩ enforce authentication and authorization on every sensitive endpoint
⬩ check object-level access on every resource request
⬩ remove debug endpoints from deployed builds
⬩ never expose plaintext passwords in API responses
⬩ store secrets outside source code
⬩ hash passwords with dedicated password hashing algorithms
⬩ encode output before rendering user-controlled data in HTML
⬩ enable CSRF protection for state-changing requests
⬩ configure secure session cookies
⬩ run containers as non-root users
⬩ drop unnecessary Linux capabilities
⬩ use read-only filesystems where possible
⬩ regularly scan images and dependencies for known CVEs

## 🔸 Purpose

To be honest it's just DevSecOps practice.

It demonstrates not only that a vulnerability can be reproduced, but also that the finding can be classified, explained and connected to concrete remediation steps.

The emphasis is on practical engineering thinking:

```text
vulnerability → exploitation path → impact → root cause → mitigation → hardening
```

