---
name: code-security
description: "Perform a deep security audit of source code and generate a bilingual (EN/TH) HTML security report with CWE-mapped findings, code evidence, and remediation guidance. Covers: hardcoded secrets/credentials, injection flaws (SQL/XSS/OS/template/LDAP), broken auth & IDOR, cryptographic weaknesses, path traversal, insecure deserialization, CORS/header misconfig, sensitive data exposure, ReDoS, Docker/container security, API security (rate limiting/over-fetching/GraphQL), and file upload vulnerabilities. Trigger when user asks: security audit, find vulnerabilities, secret scan, security review, SAST, find XSS, find SQL injection, security hardening, pen test prep, check for secrets, Docker security, API security, file upload security."
argument-hint: "[project-path or leave empty for current directory]"
---

# Code Security Auditor

Perform a deep security audit of source code at `$ARGUMENTS` (default: cwd).
Generate one bilingual (EN/TH) HTML report: `security-audit-report.html`.

## Step 0: Validate input

- If `$ARGUMENTS` is provided, verify the path exists. If not, tell the user and stop.
- If `$ARGUMENTS` is empty, use cwd.
- If `security-audit-report.html` already exists, inform user it will be overwritten and proceed.

**Monorepo / root-directory detection:** If the resolved path is `.`, `/`, cwd, or a directory with **no manifest at the top level**, scan one level of sub-directories to find project roots:
1. List immediate sub-directories (skip: `.git`, `node_modules`, `vendor`, `dist`, `build`, `out`, `.next`, `.angular`, `coverage`, `__pycache__`).
2. A sub-directory is a **project root** if it contains any manifest: `go.mod`, `package.json`, `requirements.txt`, `pyproject.toml`, `Cargo.toml`, `pom.xml`, `*.csproj`, `Gemfile`, `Dockerfile`.
3. If 2+ project roots are found: scan each root independently (prioritize roots with security-sensitive names: api, auth, backend, server).
4. If only 1 project root is found: treat it as the scan target.
5. If no project roots found in sub-directories: scan the given path as-is (flat scan).
6. Tell the user which sub-directories were identified as project roots before starting the scan.

## Files to SKIP (save tokens)

NEVER read: `node_modules/`, `vendor/`, `.git/`, `dist/`, `build/`, `out/`, `.next/`, `.angular/`, `coverage/`, `*.min.js`, `*.min.css`, `*.map`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `bun.lockb`, `poetry.lock`, `Pipfile.lock`, `Gemfile.lock`, `go.sum`, binary files, `__pycache__/`, `*.pb.go`, `*.generated.*`, `*.snap`, `*.test.*`, `*_test.go`, `*.spec.*`

ONLY read: source code (`.go`, `.ts`, `.js`, `.py`, `.rs`, `.cs`, `.rb`, `.java`), configs (`Dockerfile*`, `docker-compose*.yml`, `.dockerignore`, `.env.example`, `nginx.conf`, `*.yaml`, `*.yml` excluding lock/generated), manifests (`go.mod`, `package.json`, `requirements.txt`, `pyproject.toml`, `Cargo.toml`, `pom.xml`, `*.csproj`, `Gemfile`).

Read in priority order: (1) auth/middleware/security files, (2) API handlers/controllers/routes, (3) database access/ORM/repositories, (4) file I/O handlers, (5) config/env files, (6) entry points, (7) utility/helper functions.

**Directory grouping:** Before scanning, classify every file into one of two groups based on its path:
- **Backend** — files under directories named: `backend`, `server`, `api`, `service`, `services`, `cmd`, `internal`, `pkg`, `app` (non-UI), `src` (when paired with a backend manifest), or any directory containing `.go`, `.py`, `.java`, `.cs`, `.rb`, `.rs` as the primary language
- **Frontend** — files under directories named: `frontend`, `client`, `web`, `ui`, `pages`, `components`, `views`, `public`, `static`, `assets`, or any directory containing `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html` as the primary type
- **Shared / Root** — config files, Docker files, manifests at the root level, or files that don't clearly belong to either group

Assign a `group` tag (Backend / Frontend / Shared) to each finding. Include the group in the report's file index and coverage map header. If the project has only one group (e.g. pure backend), omit the grouping label.

---

## Phase 1: Security Scan

**Before starting agents**, read `references/vuln-patterns.md` for language-specific vulnerability patterns.

> **Run all 7 scan agents in parallel** — launch simultaneously once file list and vuln-patterns are loaded.
> Every finding MUST include the relevant CWE ID from `references/vuln-patterns.md`.

**Agent A — Secrets & Credentials:**
Scan ALL files for hardcoded: API keys, passwords, tokens, private keys, connection strings, cloud credentials (AWS/GCP/Azure). Match against patterns in `references/vuln-patterns.md` Secrets section.
- CRITICAL: live-looking credential (matches known key format, non-placeholder value)
- HIGH: generic password/token variable assigned a literal string value

**Agent B — Injection Flaws:**
- SQL: string concat/interpolation in queries — no parameterized queries
- OS command: `exec`/`spawn`/`system` with user-controlled args
- XSS: unescaped user input rendered as HTML (`innerHTML`, `dangerouslySetInnerHTML`, `eval`)
- Template injection (SSTI): user input passed to template engine at render time
- LDAP/NoSQL: unsanitized user input in query operators or filter strings
- Header injection: user input written directly into HTTP response headers

**Agent C — Auth, Authorization & Session:**
- Missing authentication on sensitive endpoints (no middleware guard, no inline check)
- IDOR: object IDs from user input used in queries without ownership verification
- Broken access control: privileged operations without role/permission checks
- Insecure session: missing `HttpOnly`/`Secure`/`SameSite` cookie flags, no expiry
- Weak password handling: plaintext storage, MD5/SHA1 without salt, no stretch
- JWT issues: `alg:none`, weak secret, missing expiry/signature validation
- Timing attack: `==` comparison for tokens instead of constant-time compare
- CSRF: state-changing endpoints (POST/PUT/PATCH/DELETE) without CSRF token validation or `SameSite` cookie attribute; check for anti-CSRF middleware or framework-level protection

**Agent D — Crypto, Data Exposure & Misconfiguration:**
- Weak/broken crypto: MD5/SHA1 for security, DES/3DES/RC4, ECB mode
- Hardcoded crypto keys, IVs, or nonces
- Disabled TLS verification: `InsecureSkipVerify`, `verify=False`, `NODE_TLS_REJECT_UNAUTHORIZED=0`
- Sensitive data in logs: passwords, tokens, PII in log/print statements
- Permissive CORS: `Access-Control-Allow-Origin: *` combined with credentials
- Missing security headers: CSP, HSTS, X-Frame-Options, X-Content-Type-Options
- Debug/dev mode flags enabled in production configs

**Agent E — Input Handling, File Upload & Logic:**
- Path traversal: user-controlled paths without sanitization (no `basename`, no allowlist)
- Unsafe deserialization: `pickle.loads`, `yaml.load` without SafeLoader, Java `ObjectInputStream`, `BinaryFormatter`
- Unvalidated redirects: user-controlled redirect URLs without allowlist
- ReDoS: catastrophic-backtracking regex patterns applied to user input
- Integer overflow in security-critical calculations (token expiry, quota checks)
- Race condition (TOCTOU): check-then-use pattern on files or shared state
- **File upload**: no file type validation (MIME/magic bytes), no size limit, saving to web-accessible path, using unsanitized original filename, no content validation (SVG XSS, zip bombs). Match against `references/vuln-patterns.md` File Upload section.

**Agent F — Docker & Container Security:**
Scan `Dockerfile`, `docker-compose.yml`, `.dockerignore` and container-related configs. Match against `references/vuln-patterns.md` Docker section.
- Running as root (no `USER` instruction or `USER root`)
- Unpinned base image tags (`:latest` or no tag)
- Secrets in `ARG`/`ENV`/`environment:` as literal values
- `COPY . .` without `.dockerignore` — leaks `.env`, `.git`, secrets
- `privileged: true`, `network_mode: host`, Docker socket mount
- No multi-stage build, no `HEALTHCHECK`, unnecessary tools in prod image
- No resource limits in compose (`mem_limit`, `cpus`)

**Agent G — API Security:**
Scan route handlers, controllers, middleware for API-specific issues. Match against `references/vuln-patterns.md` API section.
- Missing rate limiting on auth/public endpoints
- Data over-fetching: returning full DB objects without DTO/serializer
- Missing pagination: unbounded `SELECT *` / `.find({})` without `LIMIT`
- No request body schema validation (no joi/zod/pydantic/class-validator)
- GraphQL: introspection enabled in prod, no query depth/complexity limit
- Sensitive data in URL query params (tokens in GET requests)
- Bulk endpoints without size limits
- Mass assignment: accepting arbitrary keys from request body

---

## Phase 2: Consolidate & Score

After all agents complete:
1. **Deduplicate** findings that overlap across agents (same file:line)
2. **Assign final severity** — CRITICAL / HIGH / MEDIUM / LOW / INFO:
   - CRITICAL: directly exploitable, RCE/data breach possible (live secret, unmitigated SQL injection, no auth on admin endpoint)
   - HIGH: exploitable with moderate effort, significant impact (IDOR, weak crypto on sensitive data, SSTI)
   - MEDIUM: exploitable under specific conditions (XSS in low-privilege field, weak hashing for non-passwords, missing CSRF)
   - LOW: defense-in-depth / hardening (missing security header, verbose error, non-critical weak config)
   - INFO: informational, no direct exploitability
3. **Risk score** (0–100): min(CRITICAL×10 + HIGH×5 + MEDIUM×2 + LOW×1, 100)
4. **Risk level**: 0–20 = Low | 21–50 = Medium | 51–80 = High | 81–100 = Critical

---

## Phase 3: Generate Report

**When ready to generate**, read these reference files (in order) — located in the **same directory as this skill file**:
- `references/styles.css` — paste into `<style>` tag
- `references/components.md` — HTML component patterns + shared JS
- `references/report-structure.md` — full section structure

Create `security-audit-report.html` in `$ARGUMENTS` (or cwd).

**Conditional sections:**
- No findings in a category → show green "No issues found" callout, do not skip the section
- Zero CRITICAL/HIGH → show congratulatory callout in Executive Summary
- Secrets section: mask values — show first 4 + `****` + last 4 chars only (e.g. `AKIA****WXYZ`)
- No Dockerfile/docker-compose found → Docker coverage cell: `.cov-clean` with "N/A — No Docker files"
- No file upload handlers found → File Upload coverage cell: `.cov-clean` with "N/A — No upload handlers"

---

## Important Notes

- Report actual code evidence (file:line + 1–3 line snippet) for **every** finding — never fabricate
- Do NOT modify or fix any code automatically — report findings only
- Thai: natural Thai, keep technical terms (XSS, SQL Injection, IDOR, JWT, CSRF) in English
- Self-contained HTML — CSS inline in `<style>`, only inline JS

## Completion

Tell user: output file path, files scanned, finding counts by severity (CRITICAL / HIGH / MEDIUM / LOW / INFO), risk score (X/100), risk level, top 3 most critical findings summary.
