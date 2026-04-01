# Security Report — Section Structure

Read this file ONLY when generating the HTML report (Phase 3).

---

## security-audit-report.html

Structure (EN section — see "TH Section" at the bottom for Thai strings and translation rules):

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
   - If multiple groups detected: show `div.file-group-badge.badge-backend|badge-frontend|badge-shared` header before each group's files (🖥️ Backend / 🌐 Frontend / 📁 Shared)
   - Group 1 "Files with Findings": `.file-hit` (CRITICAL/HIGH) and `.file-warn` (MEDIUM/LOW only) — show file path + finding count badge, sorted by count desc
   - Group 2 "Clean Files": `.file-clean` — show file path + "Clean" badge
   - Each file path links to its first finding (anchor `#file-N`) — use `file-icon` 🔴 for CRITICAL/HIGH, 🟡 for MEDIUM/LOW, ✅ for clean

8. **Security Coverage Map** — if multiple groups detected: show `div.cov-group-label` (🖥️ Backend / 🌐 Frontend / 📁 Shared) as a header before each group's grid, with a separate `div.coverage-grid` per group. Single-group projects: no label, one grid. 13 cells per grid:
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

---

## TH Section — Translation Reference

**Rule:** In `#lang-th`, translate general UI text and labels to Thai. Keep in English always:
- Severity ratings: CRITICAL / HIGH / MEDIUM / LOW / INFO
- Risk levels: Low / Medium / High / Critical
- CWE IDs, file paths, code snippets
- All security vulnerability category names (e.g. Authentication & Authorization, Cryptographic Issues, Sensitive Data Exposure, Security Misconfiguration, Insecure Deserialization, Secrets & Credentials — see Coverage Grid below)
- Technical security terms: XSS, SQL Injection, IDOR, JWT, CSRF, CORS, SSTI, LDAP, ReDoS, MIME, DTO, API, Credential, Secret, Sprint, Backlog

### Section Titles
| EN | TH |
|---|---|
| Executive Summary | สรุปผู้บริหาร |
| Critical & High Findings | ช่องโหว่ระดับวิกฤตและสูง |
| Medium Findings | ช่องโหว่ระดับปานกลาง |
| Low & Info Findings | ช่องโหว่ระดับต่ำและข้อมูล |
| Secrets & Credentials | ข้อมูลลับและ Credential |
| Files Scanned | ไฟล์ที่ตรวจสอบ |
| Security Coverage Map | แผนที่การตรวจสอบ |
| Remediation Roadmap | แผนการแก้ไข |

### Stats Row
| EN | TH |
|---|---|
| files scanned | ไฟล์ที่ตรวจสอบ |
| findings | ช่องโหว่ที่พบ |
| scope | ขอบเขต |

### File Index
| EN | TH |
|---|---|
| Files with Findings | ไฟล์ที่มีปัญหา |
| Clean Files | ไฟล์ที่ปลอดภัย |
| Clean | ปลอดภัย |
| N findings | N ปัญหา |

### Coverage Grid Categories
Keep all 13 category names in English — they are security technical terms:
Secrets & Credentials / SQL / NoSQL Injection / XSS & Template Injection / OS Command Injection / Path Traversal / Authentication & Authorization / Cryptographic Issues / Sensitive Data Exposure / Security Misconfiguration / Insecure Deserialization / Docker / Container Security / API Security / File Upload Security

### Coverage Status
| EN | TH |
|---|---|
| ✅ Clean | ✅ ปลอดภัย |
| ⚠️ N issues | ⚠️ N ปัญหา |
| 🔴 N issues | 🔴 N ปัญหา |
| N/A — No Docker files | N/A — ไม่มีไฟล์ Docker |
| N/A — No upload handlers | N/A — ไม่มี upload handler |

### Risk Score Labels
Keep risk level names (Low / Medium / High / Critical) in English. Only the gauge label can use Thai:
| EN | TH |
|---|---|
| Risk Score | คะแนนความเสี่ยง |

### Remediation Roadmap
| EN | TH |
|---|---|
| 🔴 Immediate (Today) | 🔴 ดำเนินการทันที (วันนี้) |
| 🟠 Sprint 1 (1–2 weeks) | 🟠 Sprint 1 (1–2 สัปดาห์) |
| 🟡 Sprint 2–3 (1 month) | 🟡 Sprint 2–3 (1 เดือน) |
| 🟢 Backlog (Long term) | 🟢 Backlog (ระยะยาว) |

### Secrets Section
| EN | TH |
|---|---|
| Type | ประเภท |
| File:Line | ไฟล์:บรรทัด |
| Value | ค่า |
| Rotate all exposed secrets immediately | เปลี่ยน credential ที่รั่วไหลทั้งหมดทันที |

### Callout Messages
| EN | TH |
|---|---|
| No critical vulnerabilities found | ไม่พบช่องโหว่ระดับวิกฤตหรือสูง |
| No issues found | ไม่พบปัญหาในหมวดนี้ |
