# Claude Skills Workspace

A collection of reusable GitHub Copilot agent skills for automated code analysis and reporting.

## Skills

### `code-assessment`
Performs a comprehensive technical assessment of a software project and generates two bilingual (English/Thai) HTML reports:
- **Dependency Analysis Report** — CVE scanning, version audit, and copy-paste upgrade commands
- **Technical Assessment Report** — code quality scores, architecture diagram, OWASP Top 10 (2025) security audit, CI/CD pipeline maturity assessment, and prioritized action plan

Supports Go, TypeScript, Angular, Node.js, Python, Java, Rust, and more.

### `lighthouse-report`
Reads Lighthouse HTML report files from a directory, aggregates scores across multiple pages, and generates a single bilingual (English/Thai) summary HTML report with:
- Performance, Accessibility, Best Practices, and SEO scores
- Core Web Vitals (FCP, LCP, TBT, CLS, INP etc.)
- File-level deep-dive (unused JS/CSS, render-blocking resources, layout shifts)
- Prioritized fix recommendations with code examples

### `code-security`
Performs a deep security audit of source code and generates a bilingual (English/Thai) HTML security report with:
- **5 parallel scan agents** — Secrets & Credentials, Injection Flaws, Auth/AuthZ, Crypto & Misconfig, Input Handling
- **Risk score** (0–100) with severity breakdown: CRITICAL / HIGH / MEDIUM / LOW / INFO
- Code evidence for every finding (file:line + snippet with vulnerable part highlighted)
- Secrets summary with masked values and immediate rotation guidance
- Security coverage map across 10 categories
- Prioritized remediation roadmap (Immediate → Sprint 1 → Sprint 2–3 → Backlog)

Supports Go, TypeScript, Angular, Node.js, Python, Java, Rust, .NET/C#, Ruby.

## Structure

```
skills/
├── code-assessment/     # Technical assessment & dependency audit skill
├── lighthouse-report/   # Lighthouse report aggregator skill
└── code-security/       # Deep security audit skill
```

---

## Usage Examples

### `code-assessment`

```
# Assess a specific project directory
run code assessment on ./my-go-api

# Assess current directory
audit this project and generate HTML reports

# With explicit argument
code-assessment: /Users/me/projects/backend
```

Output: `dependency-analysis-report.html` + `technical-assessment-report.html` in the target directory.

---

### `lighthouse-report`

First, export Lighthouse reports as HTML files into a directory:

```bash
# Export from Chrome DevTools: Lighthouse tab → Download report → HTML
# Or run via CLI:
npx lighthouse https://example.com --output html --output-path ./reports/home.html
npx lighthouse https://example.com/about --output html --output-path ./reports/about.html
npx lighthouse https://example.com/products --output html --output-path ./reports/products.html
```

Then invoke the skill:

```
# Pass directory containing the HTML files
aggregate lighthouse reports in ./reports

# Or with argument
lighthouse-report: ./reports
```

Output: `lighthouse-summary-report.html` in the same directory as source reports.

---

### `code-security`

```
# Audit a specific project
security audit ./my-api

# Check for secrets and vulnerabilities
find vulnerabilities in /path/to/project

# With explicit argument
code-security: ./src
```

Output: `security-audit-report.html` in the target directory with risk score (0–100), severity-graded findings, code evidence, and remediation roadmap.

---

## How to Use Skills by Vendor

### GitHub Copilot (VS Code)

Skills are natively supported as **agent customization files** (`.skill.md` / `SKILL.md`).

1. Open VS Code with the GitHub Copilot Chat extension installed.
2. Open the **Copilot Chat** panel and switch to **Agent** mode.
3. Copilot automatically discovers skill files in your workspace. Invoke a skill by describing the task — Copilot will match it to the skill's `description` field.

```
# code-assessment
run code assessment on ./my-project

# lighthouse-report
aggregate lighthouse reports in ./reports

# code-security
security audit ./my-api
```

> The `argument-hint` in each skill's frontmatter tells you what argument to pass (e.g. a directory path).

---

### Claude (Anthropic — claude.ai or API)

Claude does not auto-discover skill files. Copy the skill content and paste it as a **system prompt** or at the start of your conversation.

**Option A — Paste skill inline:**
1. Open the skill file (e.g. `skills/code-assessment/SKILL.md`).
2. Copy the full content (everything after the frontmatter `---`).
3. Paste it into your Claude conversation before your request:

```
[paste skill content here]

Now run the assessment on: /path/to/project
```

**Option B — API system prompt:**
```python
import anthropic, pathlib

# Choose the skill you want to invoke:
# skills/code-assessment/SKILL.md
# skills/lighthouse-report/SKILL.md
# skills/code-security/SKILL.md
skill = pathlib.Path("skills/code-assessment/SKILL.md").read_text()

client = anthropic.Anthropic()
response = client.messages.create(
    model="claude-opus-4-5",
    system=skill,
    messages=[{"role": "user", "content": "Assess: /path/to/project"}],
    max_tokens=8096,
)
```

---

### Cursor

Cursor supports custom instructions via `.cursor/rules/` (MDC format) or the legacy `.cursorrules` file.

**Option A — Rules directory (recommended):**
1. Create a `.mdc` file per skill in `.cursor/rules/` in your project root:
   - `.cursor/rules/code-assessment.mdc`
   - `.cursor/rules/lighthouse-report.mdc`
   - `.cursor/rules/code-security.mdc`
2. Paste the skill body into each file (strip or keep YAML frontmatter — Cursor ignores unknown keys).
3. Cursor will apply the matching rule automatically when you open Chat or Composer.

**Option B — `.cursorrules`:**
```bash
cat skills/code-assessment/SKILL.md >> .cursorrules
```

Then in Cursor Chat:
```
# code-assessment
Run a full code assessment on this project

# lighthouse-report
Aggregate lighthouse reports in ./reports

# code-security
Security audit ./src
```

---

### Windsurf (Codeium)

Windsurf uses **Rules** stored in `.windsurf/rules/` or a `AGENTS.md` / `.windsurfrules` file.

1. Copy the skill body into `.windsurfrules` or a per-skill file under `.windsurf/rules/`:
   - `.windsurf/rules/code-assessment.md`
   - `.windsurf/rules/lighthouse-report.md`
   - `.windsurf/rules/code-security.md`
2. Open the **Cascade** panel and type your request:

```
# code-assessment
Perform a technical assessment and generate the HTML reports

# lighthouse-report
Aggregate lighthouse reports in ./reports and generate a summary

# code-security
Run a deep security audit on this project
```

Windsurf Cascade will follow the skill instructions for multi-step tool use.

---

### Cline (VS Code Extension)

Cline supports **custom instructions** via its settings and references local files with `@file`.

**Option A — Custom instructions:**
1. Open Cline settings → **Custom Instructions**.
2. Paste the skill content as the default instruction set.

**Option B — Reference at runtime:**
```
# code-assessment
@file:skills/code-assessment/SKILL.md
Run a code assessment on ./my-project

# lighthouse-report
@file:skills/lighthouse-report/SKILL.md
Aggregate lighthouse reports in ./reports

# code-security
@file:skills/code-security/SKILL.md
Security audit ./src
```

---

### Aider

Aider loads custom prompts from files using `--read` or inline with `/add`.

```bash
# code-assessment
aider --read skills/code-assessment/SKILL.md
# In REPL: > Run a full code assessment on ./my-project

# lighthouse-report
aider --read skills/lighthouse-report/SKILL.md
# In REPL: > Aggregate lighthouse reports in ./reports

# code-security
aider --read skills/code-security/SKILL.md
# In REPL: > Security audit ./src
```

---

### Continue (VS Code / JetBrains Extension)

Continue supports **slash commands** and **context providers**. Add the skill as a slash command:

1. Edit `~/.continue/config.json`:

```json
{
  "slashCommands": [
    {
      "name": "code-assessment",
      "description": "Run technical assessment and generate HTML reports",
      "prompt": "<paste skill body here>"
    },
    {
      "name": "lighthouse-report",
      "description": "Aggregate Lighthouse HTML reports into a summary",
      "prompt": "<paste skill body here>"
    },
    {
      "name": "code-security",
      "description": "Deep security audit with severity-graded findings and risk score",
      "prompt": "<paste skill body here>"
    }
  ]
}
```

2. In Continue Chat, invoke with:

```
/code-assessment ./my-project
/lighthouse-report ./reports
/code-security ./src
```

---

### General Pattern (Any LLM)

For any AI tool not listed above, the universal approach is:

1. **Copy** the skill file body (everything below the `---` frontmatter).
2. **Prepend** it to your prompt as instructions.
3. **Append** your specific request with the target path.

The skill files are plain Markdown — they work as instructions for any capable LLM.
