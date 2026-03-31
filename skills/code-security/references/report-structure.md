# Security Report — Section Structure

Read this file ONLY when generating the HTML report (Phase 3).

---

## security-audit-report.html

Structure (EN section, repeat in TH with Thai text):

1. **Cover** — logo-box=🔐, project name from path, scan date, risk level badge (`.risk-critical|high|medium|low`)

2. **Executive Summary:**
   - Risk score gauge: large number (0–100) with color-coded fill bar and risk level label
   - Severity distribution: `div.summary-grid` — 5 scards: CRITICAL / HIGH / MEDIUM / LOW / INFO with counts
   - Stats row: files scanned, total findings, scan scope description
   - One-paragraph risk narrative (bilingual)
   - If zero CRITICAL/HIGH: show `callout-success` "No critical vulnerabilities found"

3. **Critical & High Findings** — for each finding (ordered CRITICAL first, then HIGH):
   - `div.finding-card.finding-critical|high` containing:
     - `span.sev.sev-critical|high` + finding title (bold)
     - Location: `span.loc` with `file:line`
     - Code snippet: VS Code style `div.code-window` — 1–3 lines, highlight vulnerable part with `.t-danger`
     - **What is it** (1 sentence, bilingual EN label + Thai label)
     - **Why it's dangerous** (1 sentence, bilingual)
     - **Remediation** — concrete fix: numbered steps or brief before/after (no full code blocks needed)
   - If no CRITICAL/HIGH findings: `div.callout.callout-success`

4. **Medium Findings** — `div.tbl-wrap > table`:
   - Columns: #, Severity badge, File:Line (`.loc`), Issue, Remediation (brief)
   - If none: `div.callout.callout-success`

5. **Low & Info Findings** — `div.tbl-wrap > table`:
   - Columns: #, Severity badge, File:Line (`.loc`), Issue
   - If none: `div.callout.callout-success`

6. **Secrets & Credentials** — dedicated section (highest urgency):
   - Alert callout if any CRITICAL secrets found
   - Table: Type (API Key/Password/Token/Private Key), File:Line, Masked Value (`.secret-val` — show `AKIA****WXYZ` format)
   - Action box `callout-danger`: rotate all exposed secrets immediately, add patterns to `.gitignore`, migrate to secrets manager (Vault/AWS Secrets Manager/GitHub Secrets)
   - If no secrets found: `callout-success`

7. **Security Coverage Map** — `div.coverage-grid` (10 cells):
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
   - Each cell: `.cov-clean` (✅ Clean) / `.cov-warn` (⚠️ N issues) / `.cov-critical` (🔴 N issues)

8. **Remediation Roadmap** — `div.timeline` (4 items):
   - 🔴 **Immediate (Today)**: rotate exposed secrets, fix CRITICAL injection flaws, add missing auth on exposed endpoints
   - 🟠 **Sprint 1**: fix HIGH auth/IDOR issues, add input validation/parameterized queries, fix JWT issues
   - 🟡 **Sprint 2–3**: address MEDIUM findings, add security headers, improve crypto, fix path traversal
   - 🟢 **Backlog**: LOW/INFO hardening, SAST integration in CI/CD, dependency scanning, security testing

Thai section titles: สรุปผู้บริหาร, ช่องโหว่ระดับวิกฤตและสูง, ช่องโหว่ระดับปานกลาง, ช่องโหว่ระดับต่ำ, ข้อมูลลับและ Credential, แผนที่การตรวจสอบ, แผนการแก้ไข
Thai roadmap labels: ดำเนินการทันที (วันนี้), Sprint 1 (1–2 สัปดาห์), Sprint 2–3 (1 เดือน), Backlog (ระยะยาว)
