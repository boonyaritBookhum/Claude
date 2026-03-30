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
- Core Web Vitals (FCP, LCP, TBT, CLS, etc.)
- File-level deep-dive (unused JS/CSS, render-blocking resources, layout shifts)
- Prioritized fix recommendations with code examples

## Structure

```
skills/
├── code-assessment/     # Technical assessment & dependency audit skill
└── lighthouse-report/   # Lighthouse report aggregator skill
```
