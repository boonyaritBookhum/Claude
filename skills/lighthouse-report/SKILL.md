---
name: lighthouse-report
description: "Aggregate Lighthouse HTML report files from a directory and generate a bilingual (EN/TH) summary HTML report with charts, file-level deep-dive, and fix examples. Trigger when user mentions: Lighthouse reports, performance audit, Core Web Vitals, page speed, or wants to combine multiple Lighthouse HTML files."
argument-hint: "[directory-path]"
---

# Lighthouse Report Analyzer

Read all Lighthouse HTML report files from `$ARGUMENTS` (default: `lighthouse-report/` in cwd).

---

## Step 0: Validate input

- If `$ARGUMENTS` is provided, verify the directory exists. If not, tell the user and stop.
- If `$ARGUMENTS` is empty, look for `lighthouse-report/` in cwd. If it does not exist either, ask the user to provide the directory path and stop.
- If `lighthouse-summary-report.html` already exists in the target directory, inform the user it will be overwritten and proceed.
- If no HTML files found in target directory → tell the user "No Lighthouse report files found in `<path>`" and stop.

## Step 1: Discover report files

Glob `*.html` in target directory (exclude any existing `lighthouse-summary-report.html`).

- If **0 HTML files** found → tell the user "No Lighthouse report files found in `<path>`" and stop.
- If any file fails to parse or does not contain Lighthouse embedded JSON, **skip it**, note it by name, and continue with the rest.

## Step 2: Extract data (single pass, parallel agents)

> **Token efficiency:** Lighthouse HTML files are 2–5 MB of HTML/CSS/JS. Do NOT read the full file.
> Extract only the embedded JSON data blob:
> - Search for `window.__LIGHTHOUSE_JSON__` — its value is the full report JSON object
> - Or look for `<script type="application/json">` near the end of the file
> - Extract that JSON blob only. Skip all HTML markup, CSS, SVG, and other script tags.
> - This reduces token cost by **10–50× per file**.

**Batching:** 1–8 files → 1 agent. 9–16 → 2 agents (8 each). 17+ → ceil(N/8) agents, last batch takes the remainder. Each agent reads its assigned files and extracts ALL data in one pass:

**Basic metrics:**
- URL — `"requestedUrl"` or `"finalUrl"` in embedded JSON
- Category scores — Performance, Accessibility, Best Practices, SEO
  - Raw value from `"score"` field is **0.0–1.0**. Multiply by 100 to get the 0–100 display score. Round to nearest integer.
- Core Web Vitals — extract from `"numericValue"` and `"displayValue"`:
  - **FCP, LCP, TTI, Speed Index** — `numericValue` is in **milliseconds**; divide by 1000 for seconds display. Use `displayValue` as-is when available.
  - **TBT** — `numericValue` is in **milliseconds**; display as ms.
  - **CLS** — `numericValue` is dimensionless (no unit); display as a decimal (e.g. `0.25`).
- Top audit failures — opportunities with potential savings
- Diagnostics — main-thread work, JS execution time, unused JS size, network payload

**File-level detail (same pass):**
- Unused JS — try `"unused-javascript"` first (Lighthouse ≥ v9); fall back to `"reduce-unused-javascript"` (v8 and earlier): file URL, totalBytes, wastedBytes
- Unused CSS — `"unused-css-rules"`: file URLs, wasted bytes
- Render-blocking — `"render-blocking-resources"` items
- Third-party code — `"third-party-summary"` with transfer sizes
- Main-thread breakdown — `"mainthread-work-breakdown"` categories
- LCP element — `"largest-contentful-paint-element"`: DOM element, selector, timing breakdown
- Layout shift elements — `"layout-shift-elements"`: DOM elements with shift scores
- DOM size — `"dom-size"`: total elements, max depth, max children
- Long tasks — `"long-tasks"`: JS files with longest blocking tasks
- bfcache failures — `"bf-cache"`: failure reasons (actionable vs not)
- Font display — `"font-display"`: fonts without `font-display: swap`
- Image optimization — `"modern-image-formats"` / `"uses-optimized-images"`: wasteable bytes

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

**When ready to generate**, read these reference files (in order) — located in the **same directory as this skill file** (not the report directory):
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
