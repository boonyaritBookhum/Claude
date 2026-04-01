---
name: code-security
description: "Deep security audit of source code — bilingual (EN/TH) HTML report with CWE-mapped findings and remediation. Covers: hardcoded secrets, SQL/XSS/OS/template/LDAP injection, broken auth & IDOR, crypto weaknesses, path traversal, insecure deserialization, CORS/header misconfig, sensitive data exposure, ReDoS, Docker/container security, API security, file upload vulnerabilities. Trigger: security audit, vulnerability scan, secret scan, SAST, XSS, SQL injection, security hardening, pen test prep, Docker security, API security."
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

**Before starting agents**, read `references/vuln-patterns.md` once — its contents are loaded into shared context for all agents. **Agents must NOT re-read this file.**

> **Run all 7 scan agents in parallel** — launch simultaneously once file list and vuln-patterns are loaded.
> Every finding MUST include the relevant CWE ID from `references/vuln-patterns.md`.

### Scanning strategy (applies to ALL agents)

1. **Grep first** — use pattern matching (grep/ripgrep) to find candidate lines before reading files. Do NOT read files line-by-line.
2. **Cap**: process max **200 grep matches per agent**. If more matches found, prioritize: auth/security files > API handlers > config > utilities.
3. **Read only matched files** — open a file only when grep confirms a hit. Read ±20 lines around the match for context.
4. **Per-file limit**: max **500 lines per file read**. For larger files, read only the matched regions.
5. **Report format**: each finding = `{file, line, cwe, severity, evidence (1–3 line snippet)}`.

### Agent assignments

Each agent uses its corresponding section from pre-loaded `vuln-patterns.md` as the pattern reference. Do NOT repeat patterns already in that file — just apply them.

| Agent | Scope | vuln-patterns.md section |
|-------|-------|--------------------------|
| **A** | Secrets & Credentials (API keys, passwords, tokens, private keys, connection strings, cloud creds) | **Secrets** |
| **B** | Injection (SQL, OS cmd, XSS, SSTI, LDAP/NoSQL, header injection, SSRF) | **Injection** |
| **C** | Auth, Authorization & Session (missing auth, IDOR, broken access control, session, JWT, CSRF, timing) | **Auth & Session** + **CSRF** |
| **D** | Crypto, Data Exposure & Misconfiguration (weak crypto, hardcoded keys, TLS bypass, data in logs, CORS, headers, debug mode) | **Crypto** |
| **E** | Input Handling, File Upload & Logic (path traversal, deserialization, redirects, ReDoS, overflow, TOCTOU, file upload) | **Injection** (path/deser) + **File Upload** |
| **F** | Docker & Container Security (Dockerfile, compose, .dockerignore) | **Docker & Container Security** |
| **G** | API Security (rate limit, over-fetch, pagination, validation, GraphQL, mass assignment) | **API Security** |

**Severity hints** (agents assign preliminary severity; Phase 2 finalizes):
- Agent A: CRITICAL if live-looking key format, HIGH if generic literal password/token
- Agent F: read only `Dockerfile*`, `docker-compose*.yml`, `.dockerignore` — skip grep, read files directly (small set)
- Agent G: focus on route/handler/controller files only

---

## Phase 2: Consolidate & Score

> **Start Phase 2 ONLY after all 7 agents report completion.**

1. **Deduplicate** by `file:line + CWE` combination. If same file:line, same CWE from multiple agents: keep highest severity instance.
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

> **Start Phase 3 ONLY after Phase 2 scoring is complete.** Do NOT re-read `vuln-patterns.md` — CWE IDs are already in findings.

Read all three reference files **simultaneously (in parallel)** — located in the **same directory as this skill file**:
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
