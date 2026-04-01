# Claude Skills Workspace

Reusable AI agent skills for automated code analysis — works with GitHub Copilot, Claude, Cursor, and any capable LLM.

## Skills

| Skill | Output | Description |
|---|---|---|
| `code-assessment` | 2 HTML reports | Dependency CVE scan + technical assessment (code quality, architecture, OWASP 2025, CI/CD maturity) |
| `lighthouse-report` | 1 HTML report | Aggregates Lighthouse HTML files into a bilingual summary with Core Web Vitals and fix recommendations |
| `code-security` | 1 HTML report | Deep security audit — risk score (0–100), severity-graded findings, code evidence, remediation roadmap |

All reports are bilingual (English/Thai). Supports Go, TypeScript, Python, Java, Rust, .NET/C#, and more.

## Usage

### GitHub Copilot (VS Code)
Skills are auto-discovered in Agent mode. Just describe the task:
```
run code assessment on ./my-project
aggregate lighthouse reports in ./reports
security audit ./src
```

### Claude / Any LLM
Paste the skill file content as a system prompt, then append your request:
```
[contents of skills/code-assessment/SKILL.md]

Run the assessment on: /path/to/project
```

### Cursor
Copy skill body into `.cursor/rules/<skill-name>.mdc` — Cursor applies it automatically in Chat/Composer.

### Cline
Reference at runtime with `@file:skills/code-assessment/SKILL.md` before your request.

### Aider
```bash
aider --read skills/code-assessment/SKILL.md
```

## Structure
```
skills/
├── code-assessment/
├── lighthouse-report/
└── code-security/
```
