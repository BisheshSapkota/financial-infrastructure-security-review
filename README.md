# Passive Web Infrastructure Security Assessment: Enterprise Financial Gateway

## 1. Project Context & Scope
This repository details the findings of a passive infrastructure configuration audit conducted against the public-facing web architecture of a Tier-1 commercial banking institution. 

The objective of this assessment was to evaluate edge-security perimeter controls, Web Application Firewall (WAF) effectiveness, and cross-origin resource sharing (CORS) policies across corporate, payment processing, and internal staging subdomains.

* **Target Scope**: `://target-bank.com`
* **Methodology**: Passive Reconnaissance, Security Header Fingerprinting, Threat-Modeling
* **Post-Assessment Classification**: Medium-High Risk

---

## 2. Core Vulnerability Matrix (Top Findings)

### Finding 01: Enterprise WAF Bypass via X-Forwarded-For Mutation
* **Severity**: Medium (CVSS 5.3)
* **Affected Host**: Administrative panel subdomain
* **The Flaw**: The edge Web Application Firewall (WAF) was configured to implicitly trust upstream internal proxy headers. By spoofing an internal loopback address within an external request header, security blocklists could be entirely circumvented.
* **Proof of Concept (PoC)**:
  ```bash
  # Standard request containing a sensitive path is blocked by the perimeter WAF:
  curl -I https://://target-bank.com/.git/HEAD -> [403 Forbidden]

  # Injecting an internal loopback bypasses the filtering mechanism completely:
  curl -I https://://target-bank.com/.git/HEAD -H 'X-Forwarded-For: 127.0.0.1' -> [200 OK]
  ```
* **Impact**: Circumvention of enterprise WAF protection mechanisms. This allows an external actor to perform directory brute-forcing against restricted administrative pathways, potentially accessing exposed version control repositories (`.git`) or deployment backups.

### Finding 02: Permissive CORS Wildcard Configuration on Production Gateways
* **Severity**: High (CVSS 7.5)
* **Affected Host**: Main corporate landing endpoint
* **The Flaw**: The web server explicitly evaluated incoming cross-origin requests and returned an `Access-Control-Allow-Origin: *` wildcard response header combined with relaxed resource verification rules.
* **Impact**: Violates core Same-Origin Policy (SOP) boundaries. A malicious third-party site could build authenticated cross-origin calls to perform data exfiltration from active user sessions.

### Finding 03: Public Exposure of Staging & Integration Sandbox Environments
* **Severity**: Medium (CVSS 5.3)
* **Affected Hosts**: 4 distinct development/staging subdomains
* **The Flaw**: Multiple internal staging platforms and payment integration sandboxes were left accessible to the public internet without IP whitelisting, VPN verification arrays, or strict access controls.
* **Impact**: Provides malicious actors with an unrestricted sandbox environment to test and map functional exploits against upcoming pre-production logic without alerting core operations logging systems.

### Finding 04: RFC 1918 Private IP Address Leakage in Policy Headers
* **Severity**: Low (CVSS 3.7)
* **Affected Host**: Production landing host
* **The Flaw**: The `Content-Security-Policy` (CSP) header explicitly included an internal, private Class A IP network address block (`10.xx.xx.xx`) inside its `frame-ancestors` directive configuration.
* **Impact**: Leaks internal corporate network topology, aiding attackers in mapping internal infrastructure zones for multi-stage network lateral movement.

---

## 3. Professional Remediation Architecture
To mitigate the combined risks identified during the audit, the following defensive layers were recommended:
1. **WAF Policy Hardening**: Disable the 'Trust X-Forwarded-For' parameter for all public external traffic zones on the proxy layout. Enforce origin logging exclusively through untrusted layer-3 source validation.
2. **Staging Perimeter Isolation**: Restrict all pre-production zones to private RFC 1918 spaces accessible exclusively via corporate VPN connections or strict IP allowlists.
3. **Explicit CORS Whitelisting**: Replace the global dynamic wildcard response token with strict server-side validation against an explicit array of verified origin domains.

---
*Disclaimer: This assessment was executed strictly using passive analysis, security header verification techniques, and public-facing directory mapping tools with zero active database degradation, scraping, or system exploitation performed. It is presented here exclusively for computer science portfolio and engineering evaluation purposes.*
