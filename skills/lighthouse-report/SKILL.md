---
name: lighthouse-report
description: "Read Lighthouse HTML report files from a directory, analyze performance/accessibility/best-practices scores and metrics, then generate a beautiful summary HTML report with charts, file-level deep-dive analysis, and code fix examples in both Thai and English. Use this skill when the user mentions Lighthouse reports, performance audits, Core Web Vitals analysis, page speed reports, or wants to aggregate multiple Lighthouse HTML files into a single summary."
argument-hint: "[directory-path]"
---

# Lighthouse Report Analyzer

Read all Lighthouse HTML report files from `$ARGUMENTS` (default: `lighthouse-report/` in cwd).

---

## Step 1: Discover report files

Glob `*.html` in target directory (exclude any existing `lighthouse-summary-report.html`).

## Step 2: Extract data (single pass, parallel agents ~8 files each)

Launch parallel agents (subagent_type `general-purpose`). Each agent reads assigned files and extracts ALL data in one pass:

**Basic metrics:**
- URL — `"requestedUrl"` or `"finalUrl"` in embedded JSON
- Category scores — Performance, Accessibility, Best Practices, SEO (`"score"` near category names)
- Core Web Vitals — FCP, LCP, TBT, CLS, Speed Index, TTI (`"numericValue"`, `"displayValue"`)
- Top audit failures — opportunities with potential savings
- Diagnostics — main-thread work, JS execution time, unused JS size, network payload

**File-level detail (same pass):**
- Unused JS — `"reduce-unused-javascript"` / `"unused-javascript"`: file URL, totalBytes, wastedBytes
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

1. **Group by URL path module** — e.g., `/well-database/*`, `/admin-setting/*`. Compute avg score per group.
2. **Shared problems** — files on ALL pages (shared bundle, polyfills, external scripts, fonts). Highest priority.
3. **Per-page problems** — unique to specific pages (large DOM, CLS elements, page-specific chunks).
4. **Fix safety classification:**
   - **Zero risk**: preload hints, preconnect, image format, font-display: swap, CSS min-height
   - **Low risk**: async/defer scripts, removing unused CSS links, lazy-loading routes
   - **Medium risk**: code splitting, removing polyfills, zoneless Angular, SSR

## Step 4: Generate HTML report

**When ready to generate**, read `references/design.md` for CSS design system, code block patterns, code examples, Chart.js structure, and JS.

Create `lighthouse-summary-report.html` in the same directory as source reports.

### Report Sections (bilingual EN/TH)

1. **Executive Summary** — status, pages tested, score range, avg/min/max stats grid
2. **Understanding the Metrics** — table: FCP, LCP, TBT, CLS, Speed Index with Good/Needs Work/Poor thresholds
3. **Scores Overview — Grouped by Menu** — pages by URL module:
   - Colored header row: group name + avg score badge + page count
   - Page rows indented with `&emsp;`
   - Group summary cards at top
   - "Full Report" column linking to original HTML files (relative paths)
4. **Root Cause Charts** (Chart.js):
   - Root Cause Doughnut — weighted % (Unused JS, Main-Thread, Slow LCP, CLS, Network, bfcache)
   - Radar — avg vs "Good" threshold for 5 metrics
   - Performance Score Bar — horizontal bars sorted low→high, threshold lines at 50/90
   - LCP Distribution — histogram by LCP range
   - TBT Distribution — histogram by TBT range
   - Main-Thread Work Top 10 — stacked bars (JS Execution vs Other)
   - Unused JS Top 10 — horizontal bars with KiB labels
   - CLS by Page — horizontal bars, threshold lines at 0.1/0.25
5. **Key Findings** — numbered analysis of ~7 major problems
6. **File-Level Deep Dive** — THE MOST IMPORTANT SECTION:
   - **A: Shared Problems** — per problematic file: name+size, stats grid (% unused, worst long task, scripting time, pages affected), "What is it" + "Why it's slow" (bilingual), green "SAFE FIX" box with numbered steps + risk level
   - **B: Per-Page Problems** — table: Page (grouped by module), Problem File/Element, Impact (CRITICAL/HIGH/MEDIUM + metrics), Safe Fix, Risk (Zero/Low/Medium). Pages with no issues: "No page-specific fix needed"
7. **Recommendations** — prioritized table: Priority tag, Area, Recommendation, Expected Impact
8. **Action Plan** — 4-phase: Quick Wins → Code Optimization → Runtime Performance → Advanced
9. **Code Examples — How to Fix** — 7 before/after code samples (see `references/design.md`)

## Step 5: Confirm completion

Tell user: file path, files analyzed, module groupings with avg scores (table), top 3 shared issues, top 3 per-page issues, risk level summary (Zero/Low/Medium counts).
