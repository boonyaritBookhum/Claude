---
name: code-assessment
description: "Generate two bilingual (EN/TH) HTML reports: (1) Dependency Analysis — CVE scanning, version audit, upgrade commands, (2) Technical Assessment — code quality grades, architecture, OWASP Top 10 (2025), CI/CD maturity, OWASP CI/CD Security Risks, action plan. Trigger: audit, assess, scan dependencies, check CVEs, code review, technical debt, OWASP, CI/CD audit, pipeline security. Works with Go, TypeScript, Angular, Node.js, Python, Java, Rust, .NET/C#, Ruby."
argument-hint: "[project-path or leave empty for current directory]"
---

# Code Assessment & Report Generator

Analyze source code at `$ARGUMENTS` (default: cwd). Generate two bilingual (EN/TH) HTML reports:
1. `dependency-analysis-report.html` — dependency audit, CVEs, upgrade commands
2. `technical-assessment-report.html` — code quality grades, architecture, security, action plan

## Step 0: Validate input

- If `$ARGUMENTS` is provided, verify the path exists. If not, tell the user and stop.
- If `$ARGUMENTS` is empty, use cwd.
- If `dependency-analysis-report.html` or `technical-assessment-report.html` already exist in the target directory, inform the user they will be overwritten and proceed.

## Files to SKIP (save tokens)

NEVER read: `node_modules/`, `vendor/`, `.git/`, `dist/`, `build/`, `out/`, `.next/`, `.angular/`, `coverage/`, `*.min.js`, `*.min.css`, `*.map`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `bun.lockb`, `poetry.lock`, `Pipfile.lock`, `Gemfile.lock`, `go.sum`, `.env*`, binary files, `__pycache__/`, `*.pb.go`, `*.generated.*`

ONLY read: manifests (`go.mod`, `package.json`, `requirements.txt`, `pyproject.toml`, `Pipfile`, `Cargo.toml`, `pom.xml`, `build.gradle`, `*.csproj`, `Gemfile`), source code (`.go`, `.ts`, `.js`, `.py`, `.rs`, `.cs`, `.rb`, `.java`), configs (`tsconfig.json`, `angular.json`, `Dockerfile`, `nuget.config`), entry points (`main.go`, `main.ts`, `index.ts`, `Program.cs`, `Startup.cs`, `main.rs`, `lib.rs`), CI/CD configs (`.github/workflows/*.yml`, `Jenkinsfile`, `.gitlab-ci.yml`, `.circleci/config.yml`, `azure-pipelines.yml`, `bitbucket-pipelines.yml`, `.drone.yml`, `cloudbuild.yaml`).

---

## Phase 1: Discover & Analyze

**1a.** Find manifests (`go.mod`, `package.json`, `requirements.txt`, `pyproject.toml`, `Pipfile`, `Cargo.toml`, `pom.xml`, `build.gradle`, `*.csproj`, `Gemfile`). Identify subprojects. **Cap at 10 subprojects** — if more exist, prioritize: (1) those named `api`, `server`, `backend`, `core`, `app`, (2) subprojects with the most source files. Inform the user which were skipped.
- If **no manifests found at all**, tell the user "No supported project manifests found in `<path>`" and stop.

> **Run 1b, 1c, 1d, and 1e in parallel** — launch all four as simultaneous agents once manifests and file lists are identified. Do not wait for one to finish before starting the next.

**1b. Dependency analysis** (parallel agents per subproject):
- Extract deps + versions from manifests only
- **Skip** pure dev/tooling deps: `eslint`, `prettier`, `jest`, `mocha`, `@types/*`, `webpack`, `babel`, `typescript`, `husky`, `lint-staged`
- Web search `"<pkg> latest version"` and `"<pkg> CVE vulnerability"` per significant dep — **cap at 30 deps per subproject**; if more, prioritize: (1) runtime/direct deps, (2) deps with known vulnerability history (e.g. `express`, `axios`, `django`, `spring`), (3) largest version gaps
- Risk levels: CRITICAL (exploit CVE) → HIGH (DoS CVE, deprecated, EOL) → MEDIUM (many versions behind) → LOW (1-2 patches) → OK
- Generate upgrade commands per package manager:
  - Go: `go get pkg@ver`
  - Node.js/npm: `npm install pkg@ver`
  - Node.js/yarn: `yarn add pkg@ver` / Node.js/pnpm: `pnpm add pkg@ver` / Node.js/bun: `bun add pkg@ver`
  - Python/pip: `pip install pkg==ver`
  - Python/poetry: `poetry add pkg@ver`
  - Python/uv: `uv add pkg==ver`
  - Rust: edit `Cargo.toml` version then `cargo update -p pkg`
  - Java/Maven: `mvn versions:use-dep-version -Dincludes=group:artifact -DdepVersion=ver`
  - Java/Gradle: edit `build.gradle` dependency version
  - .NET/C#: `dotnet add package pkg --version ver`
  - Ruby/Bundler: edit `Gemfile` version then `bundle update pkg`

**1c. Code quality** (parallel agents):
- **Cap at 40 source files per subproject.** Read in priority order: (1) entry points (`main.*`, `index.*`, `app.*`, `server.*`, `Program.cs`, `Startup.cs`), (2) routers/controllers, (3) auth/security middleware, (4) database/ORM models, (5) utility/helper functions, (6) test files (sample only — 3 max). Stop once 40 files are read.
- Scan for: architecture, tests, security (secrets, XSS, CORS, injection, unbounded reads, leaks, missing timeouts/shutdown), performance, stability. Grade A+ to F.

**1d. OWASP Top 10 (2025) audit** (parallel agent):
Read `references/owasp-2025.md` for the full 10-category checklist. Systematically check code against all categories (2025 edition).
For each category: rate as PASS / WARN / FAIL with evidence (file:line). Generate overall OWASP compliance score (X/10 passed).

**1e. CI/CD pipeline assessment** (parallel agent):
Find and read CI/CD config files. Evaluate in two dimensions:

**Dimension 1: CI/CD Pipeline Maturity** — read `references/cicd-maturity.md` for the full 8-area checklist.

**Dimension 2: OWASP Top 10 CI/CD Security Risks** — security-focused audit based on OWASP CI/CD project:
Read `references/cicd-sec.md` for the full CICD-SEC-01 to CICD-SEC-10 checklist.
For each CICD-SEC risk: rate as PASS / WARN / FAIL with evidence. Generate overall CI/CD security score (X/10 passed).

---

## Phase 2: Generate Reports

**When ready to generate**, read all three reference files **simultaneously (in parallel)**:
- `references/styles.css` — paste into `<style>` tag of both reports
- `references/components.md` — HTML component patterns + shared JS
- `references/report-structure.md` — full section structure for both reports

Create both HTML files in `$ARGUMENTS` (or cwd).

**Conditional sections:** Skip or collapse sections with no data:
- No CVEs found → show "No known CVEs detected" callout instead of empty CVE table
- No CI/CD config found → show "No CI/CD pipeline detected" alert, skip pipeline-flow diagram, set maturity to Beginner
- No code-level security findings → show success callout instead of empty issues table
- Zero OWASP failures → show congratulatory callout instead of detailed failure list

---

## Important Notes

- Only report CVEs verified via web search — never fabricate CVE IDs
- Thai: natural Thai, keep technical terms (CVE, DoS, XSS) in English
- Reports link to each other via relative filename
- Self-contained HTML — CSS inline in `<style>`, no external JS dependencies (only inline scripts)
- Severity: Critical=patch before deploy, High=this sprint, Medium=plan, Low=nice-to-have

## Completion

Tell user: file paths, subprojects analyzed, deps audited, CVE counts, OWASP 2025 score (X/10 passed), CI/CD maturity level, CI/CD security score (X/10 CICD-SEC passed), top 3 urgent findings, grades, open in browser for language toggle.
