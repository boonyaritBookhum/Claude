---
name: lighthouse-report
description: "Aggregate Lighthouse HTML report files and generate a bilingual (EN/TH) summary HTML report with charts, file-level analysis, and fix examples. Trigger: Lighthouse reports, performance audit, Core Web Vitals, page speed, combine multiple Lighthouse HTML files."
argument-hint: "[directory-path]"
---

# Lighthouse Report Analyzer

Read all Lighthouse HTML report files from `$ARGUMENTS` (default: `lighthouse-report/` in cwd).

---

## Step 0: Validate input

- If `$ARGUMENTS` is provided, verify the directory exists. If not, tell the user and stop.
- If `$ARGUMENTS` is empty, look for `lighthouse-report/` in cwd. If it does not exist either, ask the user to provide the directory path and stop.
- If `lighthouse-summary-report.html` already exists in the target directory, inform the user it will be overwritten and proceed.

## Step 1: Discover report files

Glob `*.html` in target directory (exclude any existing `lighthouse-summary-report.html`).

- If **0 HTML files** found → tell the user "No Lighthouse report files found in `<path>`" and stop.
- If any file fails to parse or does not contain Lighthouse embedded JSON, **skip it**, note it by name, and continue with the rest.

## Step 2: Extract data (single pass, parallel agents)

> **Token efficiency:** Lighthouse HTML files are 2–5 MB. Do NOT read the full file.

**JSON extraction strategy** (per file):
1. Search for `window.__LIGHTHOUSE_JSON__` in the **last 30% of the file** (JSON blob is near the end). Use offset-based reading — start from 70% of file length.
2. If not found, search for `<script type="application/json">` in the last 30%.
3. If neither found in the last 30%, try the **first 50KB** as fallback.
4. If still not found → **skip the file**, note it as unparseable. Do NOT read the entire file.
5. Extract only the JSON object. Skip all HTML markup, CSS, SVG, other scripts.

**Batching:** 1–8 files → 1 agent. 9–16 → 2 agents (8 each). 17+ → ceil(N/8) agents, last batch takes the remainder. Each agent reads its assigned files and extracts ALL data in one pass:

**Basic metrics:**
- URL — `"requestedUrl"` or `"finalUrl"` in embedded JSON
- Category scores — Performance, Accessibility, Best Practices, SEO
  - Raw value from `"score"` field is **0.0–1.0**. Multiply by 100 to get the 0–100 display score. Round to nearest integer.
- **Core Web Vitals** (LCP, CLS, INP — these 3 define the green/orange/red pass/fail badges in the report):
  - **LCP** — audit key `"largest-contentful-paint"`; `numericValue` ms → divide by 1000 for seconds. Thresholds: ≤2.5s Good, ≤4.0s Needs Work, >4.0s Poor.
  - **CLS** — audit key `"cumulative-layout-shift"`; `numericValue` is dimensionless. Thresholds: ≤0.1 Good, ≤0.25 Needs Work, >0.25 Poor.
  - **INP** — audit key `"interaction-to-next-paint"`; `numericValue` ms. Thresholds: ≤200ms Good, ≤500ms Needs Work, >500ms Poor. Present in Lighthouse v10+ only; skip gracefully if absent.
- **Lab metrics** (diagnostic, not CWV — displayed as supplementary data):
  - **FCP** — audit key `"first-contentful-paint"`; `numericValue` ms → seconds. Thresholds: ≤1.8s Good, ≤3.0s Needs Work, >3.0s Poor.
  - **TBT** — audit key `"total-blocking-time"`; `numericValue` ms. Thresholds: ≤200ms Good, ≤600ms Needs Work, >600ms Poor.
  - **TTI** — audit key `"interactive"`; `numericValue` ms → seconds.
  - **Speed Index** — audit key `"speed-index"`; `numericValue` ms → seconds.
  - Use `displayValue` as-is when available for all metrics above.
- Top audit failures — opportunities with potential savings
- Diagnostics — main-thread work, JS execution time, unused JS size, network payload

**File-level detail (same pass):**

Extract in priority order. **Cap at top 10 items per audit key** (sorted by wastedBytes or impact). If an audit key has no items or score=1, skip it.

| Priority | Audit key | Extract |
|----------|-----------|---------|
| **P1 — Core** | `"unused-javascript"` (fallback: `"reduce-unused-javascript"`) | file URL, totalBytes, wastedBytes |
| **P1** | `"render-blocking-resources"` | items |
| **P1** | `"largest-contentful-paint-element"` | DOM element, selector, timing |
| **P2 — Important** | `"unused-css-rules"` | file URLs, wasted bytes |
| **P2** | `"third-party-summary"` | transfer sizes |
| **P2** | `"mainthread-work-breakdown"` | categories |
| **P2** | `"layout-shift-elements"` | DOM elements, shift scores |
| **P2** | `"bf-cache"` | failure reasons (actionable only) |
| **P3 — Supplementary** | `"dom-size"` | total elements, max depth, max children |
| **P3** | `"long-tasks"` | JS files with longest blocking tasks |
| **P3** | `"font-display"` | fonts without `font-display: swap` |
| **P3** | `"modern-image-formats"` / `"uses-optimized-images"` | wasteable bytes |

If token budget is tight, extract P1 + P2 only. P3 is optional but provides richer detail.

## Step 3: Analyze and group

1. **Group by URL path module** — use the first path segment (e.g., `/well-database/*`, `/admin-setting/*`).
   - If a URL has no meaningful path segment (e.g., `/`, `/about`, `/contact`), assign it to a group called **"Root / Single Pages"**.
   - If ALL URLs fall into root paths, skip grouping and treat the entire set as one group.
   - Compute avg score per group.
2. **Shared problems** — files on ALL pages (shared bundle, polyfills, external scripts, fonts). Highest priority.
3. **Per-page problems** — unique to specific pages (large DOM, CLS elements, page-specific chunks).
4. **Fix safety classification:**
   - **Zero risk**: preload hints, preconnect, image format, font-display: swap, CSS min-height
   - **Low risk**: async/defer scripts, removing unused CSS links, lazy-loading routes
   - **Medium risk**: code splitting, removing polyfills, SSR, framework-specific optimizations (e.g., zoneless Angular, React Server Components, Vue async components)

## Step 4: Generate HTML report

> **Start Step 4 ONLY after Step 3 analysis is complete.**

Read all three reference files **simultaneously (in parallel)** — located in the **same directory as this skill file** (not the report directory):
- `references/styles.css` — paste into `<style>` tag
- `references/components.md` — HTML component patterns, Chart.js setup, shared JS
- `references/report-structure.md` — full section structure and code example list

Create `lighthouse-summary-report.html` in the same directory as source reports.

## Step 5: Confirm completion

Tell user:
- Output file path
- Files analyzed (and any skipped files with reason)
- Module groupings with avg scores (table)
- Top 3 shared issues
- Top 3 per-page issues
- Risk level summary (Zero/Low/Medium counts)
