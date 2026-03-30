---
name: code-assessment
description: "Perform a comprehensive technical assessment and dependency analysis of a software project, then generate two beautiful bilingual (English/Thai) HTML reports: (1) Dependency Analysis Report with CVE scanning, version audit, and copy-paste upgrade commands, and (2) Technical Assessment Report with code quality scores, architecture diagram, OWASP Top 10 (2025) security audit, OWASP Top 10 CI/CD Security Risks assessment, CI/CD pipeline maturity assessment, and prioritized action plan. Use this skill whenever the user wants to audit, review, or assess a codebase — including dependency health checks, security audits, OWASP compliance, CI/CD pipeline review, CI/CD security risks, supply chain security, code quality reviews, technical debt analysis, upgrade planning, or any request to generate a technical assessment report. Also trigger when the user mentions 'scan dependencies', 'check for CVEs', 'code review report', 'technical debt report', 'upgrade plan', 'OWASP', 'CI/CD audit', 'pipeline security', 'supply chain', 'CICD-SEC', or wants to understand the health of their project. Works with Go, TypeScript, Angular, Node.js, Python, Java, Rust, and other tech stacks."
argument-hint: "[project-path or leave empty for current directory]"
---

# Code Assessment & Report Generator

Analyze source code at `$ARGUMENTS` (default: cwd). Generate two bilingual (EN/TH) HTML reports:
1. `dependency-analysis-report.html` — dependency audit, CVEs, upgrade commands
2. `technical-assessment-report.html` — code quality grades, architecture, security, action plan

## Files to SKIP (save tokens)

NEVER read: `node_modules/`, `vendor/`, `.git/`, `dist/`, `build/`, `out/`, `.next/`, `.angular/`, `coverage/`, `*.min.js`, `*.min.css`, `*.map`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `go.sum`, `.env*`, binary files, `__pycache__/`, `*.pb.go`, `*.generated.*`

ONLY read: manifests (`go.mod`, `package.json`, `requirements.txt`), source code (`.go`, `.ts`, `.js`, `.py`), configs (`tsconfig.json`, `angular.json`, `Dockerfile`), entry points (`main.go`, `main.ts`, `index.ts`), CI/CD configs (`.github/workflows/*.yml`, `Jenkinsfile`, `.gitlab-ci.yml`, `.circleci/config.yml`, `azure-pipelines.yml`, `bitbucket-pipelines.yml`, `.drone.yml`, `cloudbuild.yaml`).

---

## Phase 1: Discover & Analyze

**1a.** Find manifests (`go.mod`, `package.json`, `requirements.txt`, `Cargo.toml`, `pom.xml`). Identify subprojects.

**1b. Dependency analysis** (parallel agents per subproject):
- Extract deps + versions from manifests only
- **Skip** pure dev/tooling deps: `eslint`, `prettier`, `jest`, `mocha`, `@types/*`, `webpack`, `babel`, `typescript`, `husky`, `lint-staged`
- Web search `"<pkg> latest version"` and `"<pkg> CVE vulnerability"` per significant dep — **cap at 30 deps per subproject**; if more, prioritize: (1) runtime/direct deps, (2) deps with known vulnerability history (e.g. `express`, `axios`, `django`, `spring`), (3) largest version gaps
- Risk levels: CRITICAL (exploit CVE) → HIGH (DoS CVE, deprecated, EOL) → MEDIUM (many versions behind) → LOW (1-2 patches) → OK
- Generate upgrade commands: `go get pkg@ver`, `npm install pkg@ver`, etc.

**1c. Code quality** (parallel agents):
- **Cap at 40 source files per subproject.** Read in priority order: (1) entry points (`main.*`, `index.*`, `app.*`, `server.*`), (2) routers/controllers, (3) auth/security middleware, (4) database/ORM models, (5) utility/helper functions, (6) test files (sample only — 3 max). Stop once 40 files are read.
- Scan for: architecture, tests, security (secrets, XSS, CORS, injection, unbounded reads, leaks, missing timeouts/shutdown), performance, stability. Grade A+ to F.

**1d. OWASP Top 10 (2025) audit** (parallel agent):
Systematically check code against all 10 categories (2025 edition):
- **A01:2025 Broken Access Control** — missing auth checks, IDOR, path traversal, CORS misconfig, privilege escalation, default-allow, SSRF (now consolidated here from old A10:2021), unvalidated URLs from user input, no allowlist for outbound requests, cloud metadata access risk
- **A02:2025 Security Misconfiguration** — default credentials, unnecessary features/ports, verbose errors in prod, missing security headers, permissive CORS, overly permissive cloud/container configs, unnecessary HTTP methods enabled, missing HSTS/CSP headers
- **A03:2025 Software Supply Chain Failures** _(NEW in 2025)_ — unvetted third-party dependencies, missing integrity verification (no lockfile or lockfile ignored), typosquatting risk, dependency confusion, unsigned packages, no SBOM, pulling from untrusted registries, missing vulnerability scanning in dependency pipeline. Cross-reference with dependency analysis results.
- **A04:2025 Cryptographic Failures** — weak/no encryption, hardcoded keys, weak hashing (MD5/SHA1), missing TLS, sensitive data in logs/URLs, insufficient key rotation, use of deprecated crypto algorithms, improper certificate validation
- **A05:2025 Injection** — SQL/NoSQL/OS/LDAP injection, XSS, template injection, unsafe deserialization of user input, header injection, expression language injection, ORM injection
- **A06:2025 Insecure Design** — missing rate limiting, no abuse-case modeling, lack of input validation patterns, missing business-logic controls, no threat modeling, missing security requirements in design, no defense-in-depth strategy
- **A07:2025 Authentication Failures** — weak password policy, missing MFA, session fixation, credential stuffing exposure, plaintext tokens, improper session management, weak credential recovery, brute-force susceptibility
- **A08:2025 Software or Data Integrity Failures** — unsigned updates, unverified CI/CD pipelines, unsafe deserialization, no integrity checks on artifacts, auto-update without verification, CDN/external resource integrity (missing SRI), tampered data accepted without validation
- **A09:2025 Logging & Alerting Failures** — no audit logs, missing login/access-denied logging, no alerting on suspicious activity, logs without context (who/what/when), log injection vulnerabilities, no centralized logging, missing real-time alerting, excessive false positives masking real threats
- **A10:2025 Mishandling of Exceptional Conditions** _(NEW in 2025)_ — unhandled exceptions exposing stack traces/internals, fail-open error handling (system grants access on error), missing error boundaries, crash-on-edge-case vulnerabilities, resource exhaustion from unhandled states, inconsistent error responses leaking info, missing circuit breakers, no graceful degradation under load

For each category: rate as PASS / WARN / FAIL with evidence (file:line). Generate overall OWASP compliance score (X/10 passed).

**1e. CI/CD pipeline assessment** (parallel agent):
Find and read CI/CD config files. Evaluate in two dimensions:

**Dimension 1: CI/CD Pipeline Maturity** — operational readiness:
- **Pipeline Security** — secrets in env vars vs vault/secrets manager, least-privilege tokens, no plaintext credentials in configs
- **Build Integrity** — pinned action/image versions (not `:latest`), reproducible builds, dependency lockfile enforced, SBOM generation
- **Test Automation** — unit/integration/e2e tests in pipeline, coverage gates, fail-on-test-failure, test parallelization
- **Code Quality Gates** — linter, formatter, SAST/DAST scanning, type checking, PR review requirements
- **Deployment Safety** — staging/canary/blue-green strategy, rollback mechanism, health checks, deploy approvals for prod
- **Supply Chain Security** — dependency scanning in pipeline (Dependabot/Renovate/Snyk), container image scanning, signed commits/artifacts
- **Branch Protection** — protected main branch, required reviews, no force push, status checks required
- **Monitoring & Observability** — deploy notifications, post-deploy smoke tests, alerting integration

Grade each area: ✅ Configured / ⚠️ Partial / ❌ Missing / ➖ N/A
Generate overall CI/CD maturity score: Beginner → Basic → Intermediate → Advanced → Expert

**Dimension 2: OWASP Top 10 CI/CD Security Risks** — security-focused audit based on OWASP CI/CD project:
- **CICD-SEC-01: Insufficient Flow Control Mechanisms** — can code reach production without review/approval? Missing branch protection, no required reviewers, direct push to main, no merge gates
- **CICD-SEC-02: Inadequate Identity and Access Management** — overly permissive CI/CD user roles, shared service accounts, no RBAC for pipeline resources, missing MFA for CI/CD platform access
- **CICD-SEC-03: Dependency Chain Abuse** — dependency confusion risk, typosquatting exposure, no private registry configured, missing namespace scoping, auto-merge of dependency updates without review
- **CICD-SEC-04: Poisoned Pipeline Execution (PPE)** — pipeline configs modifiable by contributors (Direct PPE), build triggered by unreviewed code changes (Indirect PPE), 3rd-party pipeline triggers without validation (3rd-Party PPE)
- **CICD-SEC-05: Insufficient PBAC (Pipeline-Based Access Controls)** — pipeline jobs with excessive permissions, shared credentials across environments, no scoping of secrets per branch/env, pipeline can access production from feature branches
- **CICD-SEC-06: Insufficient Credential Hygiene** — hardcoded secrets in pipeline configs, unrotated tokens, secrets in logs/artifacts, credentials accessible to all pipeline stages, no secret scanning in CI
- **CICD-SEC-07: Insecure System Configuration** — CI/CD platform running with default settings, unnecessary plugins/features enabled, self-hosted runners without hardening, missing network segmentation
- **CICD-SEC-08: Ungoverned Usage of 3rd Party Services** — unvetted GitHub Actions/plugins, OAuth integrations with excessive scopes, 3rd-party services with write access to repo/pipeline, no inventory of CI/CD integrations
- **CICD-SEC-09: Improper Artifact Integrity Validation** — unsigned build artifacts, no checksum verification, artifacts pulled over insecure channels, no provenance tracking, container images without signatures
- **CICD-SEC-10: Insufficient Logging and Visibility** — no audit trail for pipeline changes, missing logs for secret access, no alerting on pipeline anomalies, inability to detect unauthorized pipeline modifications

For each CICD-SEC risk: rate as PASS / WARN / FAIL with evidence. Generate overall CI/CD security score (X/10 passed).

---

## Phase 2: Generate Reports

**When ready to generate**, read these reference files:
- `references/styles.css` — paste into `<style>` tag of both reports
- `references/components.md` — HTML component patterns (deferred: read only when starting to generate HTML)

### Shared JS (bottom of both reports)
```html
<script>
function switchLang(lang,btn){
  document.querySelectorAll('.lang-section').forEach(el=>el.classList.remove('active'));
  document.querySelectorAll('.lang-tab').forEach(el=>el.classList.remove('active'));
  document.getElementById('lang-'+lang).classList.add('active');
  btn.classList.add('active');
}
</script>
```

### Report 1: dependency-analysis-report.html

Structure (EN section, repeat in TH with Thai text):
1. **Cover** — logo-box=📦, link to technical report
2. **Audit Summary** — summary-grid (5 scard: CRITICAL/HIGH/MEDIUM/LOW/OK counts) + alert boxes
3. **Package Audit Table** — columns: Package, Project, Current(.ver-old), Latest(.ver-new), CVE/Risk, Urgency(.badge)
   - Group by subproject using `.cat-row` headers
4. **Upgrade Commands** — upgrade-block per urgency (CRITICAL→HIGH→MEDIUM)
5. **Code-Level Security Findings** — table: Severity, Project, Location, Finding (if deep scan)
6. **OWASP Top 10 (2025) Compliance** — owasp-score ring (X/10) + owasp-grid (10 items A01–A10:2025, each PASS/WARN/FAIL with brief finding)
7. **CI/CD Pipeline Assessment** — maturity-bar (level 1-5) + pipeline-flow (stages with status) + table: Area, Status, Finding, Recommendation
8. **OWASP Top 10 CI/CD Security Risks** — cicd-owasp-score ring (X/10) + owasp-grid (CICD-SEC-01 to CICD-SEC-10, each PASS/WARN/FAIL with brief finding)
9. **Upgrade Plan** — timeline: 🔴 Week 1, 🟡 Sprint 1, 🟠 Sprint 2-3, 🟢 Next Quarter

Thai section titles: สรุปผลการตรวจสอบ, ตารางวิเคราะห์ Package, คำสั่ง Upgrade, ปัญหาความปลอดภัยระดับโค้ด, การประเมิน OWASP Top 10 (2025), การประเมิน CI/CD Pipeline, ความเสี่ยงด้านความปลอดภัย CI/CD (OWASP Top 10), แผนการอัปเกรด
Thai timeline: สัปดาห์ที่ 1, Sprint 1 (2 สัปดาห์), Sprint 2–3, ไตรมาสถัดไป

### Report 2: technical-assessment-report.html

Structure (EN section, repeat in TH):
1. **Cover** — logo-box=🛢️, link to dependency report
2. **CVE Banner** — cve-banner (before lang tabs, always visible)
3. **Report Nav** — buttons linking both reports
4. **Executive Summary** — callout-success + score-grid (8 grades: Code Quality, Architecture, Dependency Health, Security, OWASP Compliance, CI/CD Maturity, CI/CD Security, Stability) + cve-grid
5. **Architecture** — arch-box (ASCII diagram with colored spans) + sub-grid cards
6. **Code Quality Strengths** — card per subproject with checklist (ci-ok/ci-warn)
7. **CVE Table** — columns: CVE(.cve-pill), Package, Project, Vulnerable Ver(red), Fixed Ver(green), Impact
8. **Code Issues** — columns: #, Severity(.badge), Location(code+small), Issue & Fix(strong+em)
9. **OWASP Top 10 (2025) Compliance** — owasp-score ring (X/10 categories passed) + owasp-grid (A01–A10:2025, each with PASS/WARN/FAIL status) + expandable findings per category with file:line evidence + callout linking to https://owasp.org/Top10/2025/ + note changes from 2021 (A03 Supply Chain & A10 Exceptional Conditions are new)
10. **CI/CD Pipeline Assessment** — maturity-bar (Beginner→Expert) + pipeline-flow diagram (Source→Build→Test→Security→Deploy stages) + detailed table: Area, Status(✅/⚠️/❌/➖), Current State, Recommendation + callout for top 3 CI/CD improvements
11. **OWASP Top 10 CI/CD Security Risks** — cicd-owasp-score ring (X/10 risks passed) + owasp-grid (CICD-SEC-01 to CICD-SEC-10, each with PASS/WARN/FAIL status and evidence) + detailed table: Risk ID, Risk Name, Status, Finding, Recommendation + callout linking to https://owasp.org/www-project-top-10-ci-cd-security-risks/
12. **Dependency Health** — cross-project comparison table + callout-warn linking to dep report
13. **Stability** — 2 cards: "Currently Stable ✓" (list) + "Stability Risks ⚠️" (bold+list)
14. **Action Plan** — timeline: 🔴 Week 1 (CVEs+critical OWASP fails+critical CI/CD-SEC risks+critical code), 🟡 Sprint 1 (deprecated+high fixes+CI/CD security hardening+PPE mitigation), 🟠 Sprint 2-3 (framework+OWASP compliance+CI/CD maturity+supply chain security), 🟢 Next Quarter (long-term+full OWASP 2025 compliance+CI/CD SEC optimization+full audit trail)

Thai section titles: บทสรุปผู้บริหาร, สถาปัตยกรรมระบบ, จุดแข็งด้านคุณภาพโค้ด, ช่องโหว่ความปลอดภัย, ปัญหาคุณภาพโค้ด, การประเมิน OWASP Top 10 (2025), การประเมิน CI/CD Pipeline, ความเสี่ยงด้านความปลอดภัย CI/CD (OWASP Top 10), สรุป Dependency, เสถียรภาพการทำงาน, แผนการดำเนินการ

---

## Important Notes

- Read `references/styles.css` once when generating — paste as `<style>` content
- Only report CVEs verified via web search — never fabricate CVE IDs
- Web search for real latest versions
- Thai: natural Thai, keep technical terms (CVE, DoS, XSS) in English
- Reports link to each other via relative filename
- Self-contained HTML — CSS inline in `<style>`, only JS is `switchLang()`
- Severity: Critical=patch before deploy, High=this sprint, Medium=plan, Low=nice-to-have
- OWASP Top 10: reference **2025 edition** (latest). Link to https://owasp.org/Top10/2025/ for context. Note the 2 new categories: A03 Software Supply Chain Failures and A10 Mishandling of Exceptional Conditions. SSRF is now part of A01. Do not fabricate findings — only report what is evidenced in code.
- OWASP CI/CD Security Risks: reference OWASP Top 10 CI/CD Security Risks project. Link to https://owasp.org/www-project-top-10-ci-cd-security-risks/ for context. Evaluate CICD-SEC-01 through CICD-SEC-10. Do not fabricate findings — only report what is evidenced in pipeline configs and code.
- CI/CD: if no pipeline config found, report as "No CI/CD pipeline detected" with recommendation to set one up. Still grade as "Beginner" with full recommendations. For OWASP CI/CD risks, mark all as FAIL with recommendation to establish pipeline first.

## Completion

Tell user: file paths, subprojects analyzed, deps audited, CVE counts, OWASP 2025 score (X/10 passed), CI/CD maturity level, CI/CD security score (X/10 CICD-SEC passed), top 3 urgent findings, grades, open in browser for language toggle.
