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

## How to Use Skills by Vendor

### GitHub Copilot (VS Code)

Skills are natively supported as **agent customization files** (`.skill.md` / `SKILL.md`).

1. Open VS Code with the GitHub Copilot Chat extension installed.
2. Open the **Copilot Chat** panel and switch to **Agent** mode.
3. Copilot automatically discovers skill files in your workspace. Invoke a skill by describing the task — Copilot will match it to the skill's `description` field.

```
@workspace run code assessment on ./my-project
```

> The `argument-hint` in each skill's frontmatter tells you what argument to pass (e.g. a directory path).

---

### Claude (Anthropic — claude.ai or API)

Claude does not auto-discover skill files. Copy the skill content and paste it as a **system prompt** or at the start of your conversation.

**Option A — Paste skill inline:**
1. Open `skills/code-assessment/SKILL.md`.
2. Copy the full content (everything after the frontmatter `---`).
3. Paste it into your Claude conversation before your request:

```
[paste skill content here]

Now run the assessment on: /path/to/project
```

**Option B — API system prompt:**
```python
import anthropic, pathlib

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
1. Create `.cursor/rules/code-assessment.mdc` in your project root.
2. Paste the skill content (strip the YAML frontmatter, or keep it — Cursor ignores unknown frontmatter).
3. Cursor will apply the rule automatically when you open Chat or Composer.

**Option B — `.cursorrules`:**
```bash
cat skills/code-assessment/SKILL.md >> .cursorrules
```

Then in Cursor Chat:
```
Run a full code assessment on this project
```

---

### Windsurf (Codeium)

Windsurf uses **Rules** stored in `.windsurf/rules/` or a `AGENTS.md` / `.windsurfrules` file.

1. Copy the skill body into `.windsurfrules` or `.windsurf/rules/code-assessment.md`.
2. Open the **Cascade** panel and type your request:

```
Perform a technical assessment and generate the HTML reports
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
@file:skills/code-assessment/SKILL.md

Run a code assessment on ./my-project
```

---

### Aider

Aider loads custom prompts from files using `--read` or inline with `/add`.

```bash
# Pass the skill as a read-only context file
aider --read skills/code-assessment/SKILL.md

# Then in the aider REPL:
> Run a full code assessment on this project and generate the HTML reports
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
```

---

### General Pattern (Any LLM)

For any AI tool not listed above, the universal approach is:

1. **Copy** the skill file body (everything below the `---` frontmatter).
2. **Prepend** it to your prompt as instructions.
3. **Append** your specific request with the target path.

The skill files are plain Markdown — they work as instructions for any capable LLM.
