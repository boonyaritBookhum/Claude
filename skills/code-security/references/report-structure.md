# Security Report — Section Structure

Read this file ONLY when generating the HTML report (Phase 3).

---

## security-audit-report.html

Structure (EN section, repeat in TH with Thai text):

1. **Cover** — logo-box=🔐, project name from path, scan date, risk level badge (`.risk-critical|high|medium|low`)

2. **Executive Summary:**
   - Risk score gauge: large number (0–100) with color-coded fill bar and risk level label
   - Severity distribution: `div.summary-grid` — 5 scards: CRITICAL / HIGH / MEDIUM / LOW / INFO with counts
   - Stats row: `div.stats-row` — files scanned, total findings, scan scope description
   - One-paragraph risk narrative (bilingual)
   - If zero CRITICAL/HIGH: show `callout-success` "No critical vulnerabilities found"

3. **Critical & High Findings** — for each finding (ordered CRITICAL first, then HIGH):
   - `div.finding-card.finding-critical|high[id="file-N"]` containing:
     - `span.sev.sev-critical|high` + finding title (bold) + `span.cwe` (e.g. CWE-89)
     - Location: `span.loc` with `file:line`
     - Code snippet: VS Code style `div.code-window` — 1–3 lines, highlight vulnerable part with `.t-danger`
     - `div.finding-what` — **What is it / คืออะไร** (1 sentence, bilingual)
     - `div.finding-why` — **Why dangerous / ทำไมถึงอันตราย** (1 sentence, bilingual)
     - `div.finding-fix` — **Remediation / การแก้ไข** — concrete fix: numbered steps or brief before/after
   - If no CRITICAL/HIGH findings: `div.callout.callout-success`

4. **Medium Findings** — `div.tbl-wrap > table`:
   - Columns: #, Severity badge, CWE (`.cwe`), File:Line (`.loc`), Issue, Remediation (brief)
   - If none: `div.callout.callout-success`

5. **Low & Info Findings** — `div.tbl-wrap > table`:
   - Columns: #, Severity badge, CWE (`.cwe`), File:Line (`.loc`), Issue
   - If none: `div.callout.callout-success`

6. **Secrets & Credentials** — dedicated section (highest urgency):
   - Alert callout if any CRITICAL secrets found
   - Table: Type (API Key/Password/Token/Private Key), File:Line, Masked Value (`.secret-val` — show `AKIA****WXYZ` format)
   - Action box `callout-danger`: rotate all exposed secrets immediately, add patterns to `.gitignore`, migrate to secrets manager (Vault/AWS Secrets Manager/GitHub Secrets)
   - If no secrets found: `callout-success`

7. **Files Scanned** — file-stats (total / with findings / clean) + file-index:
   - Group 1 "Files with Findings": `.file-hit` (CRITICAL/HIGH) and `.file-warn` (MEDIUM/LOW only) — show file path + finding count badge, sorted by count desc
   - Group 2 "Clean Files": `.file-clean` — show file path + "Clean" badge
   - Each file path links to its first finding (anchor `#file-N`) — use `file-icon` 🔴 for CRITICAL/HIGH, 🟡 for MEDIUM/LOW, ✅ for clean

8. **Security Coverage Map** — `div.coverage-grid` (13 cells):
   - Secrets & Credentials
   - SQL / NoSQL Injection
   - XSS & Template Injection
   - OS Command Injection
   - Path Traversal
   - Authentication & Authorization
   - Cryptographic Issues
   - Sensitive Data Exposure
   - Security Misconfiguration
   - Insecure Deserialization
   - Docker / Container Security
   - API Security
   - File Upload Security
   - Each cell: `.cov-clean` (icon + name + "✅ Clean") / `.cov-warn` (icon + name + "⚠️ N issues") / `.cov-critical` (icon + name + "🔴 N issues")
   - Icons: 🔑 Secrets, 💉 SQL Injection, 🖥️ XSS, ⌨️ OS Command, 📂 Path Traversal, 🔒 Auth, 🔐 Crypto, 📋 Data Exposure, ⚙️ Misconfig, 📦 Deserialization, 🐳 Docker, 🌐 API, 📤 File Upload
   - If no Dockerfile found: Docker cell shows `.cov-clean` with "N/A". If no upload handlers found: File Upload cell shows `.cov-clean` with "N/A".

9. **Remediation Roadmap** — `div.timeline` (4 items):
   - 🔴 **Immediate (Today)**: rotate exposed secrets, fix CRITICAL injection flaws, add missing auth on exposed endpoints, fix Docker root user & exposed secrets
   - 🟠 **Sprint 1**: fix HIGH auth/IDOR issues, add input validation/parameterized queries, fix JWT issues, add rate limiting, fix file upload validation
   - 🟡 **Sprint 2–3**: address MEDIUM findings, add security headers, improve crypto, fix path traversal, harden Docker images (multi-stage, pin versions), add API pagination & schema validation
   - 🟢 **Backlog**: LOW/INFO hardening, SAST integration in CI/CD, dependency scanning, security testing, container scanning, API versioning

Thai section titles: สรุปผู้บริหาร, ช่องโหว่ระดับวิกฤตและสูง, ช่องโหว่ระดับปานกลาง, ช่องโหว่ระดับต่ำ, ข้อมูลลับและ Credential, ไฟล์ที่ตรวจสอบ, แผนที่การตรวจสอบ, แผนการแก้ไข
Thai roadmap labels: ดำเนินการทันที (วันนี้), Sprint 1 (1–2 สัปดาห์), Sprint 2–3 (1 เดือน), Backlog (ระยะยาว)
