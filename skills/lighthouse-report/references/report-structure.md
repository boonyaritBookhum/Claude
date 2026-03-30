# Lighthouse Report — Section Structure

Read this file ONLY when generating the HTML report (Step 4).

---

## Report Sections (bilingual EN/TH)

1. **Executive Summary** — status, pages tested, score range, avg/min/max stats grid
2. **Understanding the Metrics** — table: FCP, LCP, INP, TBT, CLS, Speed Index with Good/Needs Work/Poor thresholds
   - INP thresholds: Good ≤200ms / Needs Work 201–500ms / Poor >500ms
   - Note: INP available in Lighthouse v10+ (replaced FID); omit row if no INP data found
3. **Scores Overview — Grouped by Menu** — pages by URL module:
   - Colored header row: group name + avg score badge + page count
   - Page rows indented with `&emsp;`
   - Group summary cards at top
   - "Full Report" column linking to original HTML files (relative paths)
4. **Root Cause Charts** (Chart.js):
   - Root Cause Doughnut — weighted % (Unused JS, Main-Thread, Slow LCP, CLS, Network, bfcache)
   - Radar — avg vs "Good" threshold for up to 6 metrics (LCP, FCP, TBT, CLS, Speed Index, INP); omit INP axis if no INP data
   - Performance Score Bar — horizontal bars sorted low→high, threshold lines at 50/90
   - LCP Distribution — histogram by LCP range
   - TBT Distribution — histogram by TBT range
   - Main-Thread Work Top 10 — stacked bars (JS Execution vs Other)
   - Unused JS Top 10 — horizontal bars with KiB labels
   - CLS by Page — horizontal bars, threshold lines at 0.1/0.25
   - INP by Page — horizontal bars, threshold lines at 200/500ms (only if INP data present)
   - **Skip any chart where all values are 0 or null across all pages.**
5. **Key Findings** — numbered analysis of ~7 major problems
6. **File-Level Deep Dive** — THE MOST IMPORTANT SECTION:
   - **A: Shared Problems** — per problematic file: name+size, stats grid (% unused, worst long task, scripting time, pages affected), "What is it" + "Why it's slow" (bilingual), green "SAFE FIX" box with numbered steps + risk level
   - **B: Per-Page Problems** — table: Page (grouped by module), Problem File/Element, Impact (CRITICAL/HIGH/MEDIUM + metrics), Safe Fix, Risk (Zero/Low/Medium). Pages with no issues: "No page-specific fix needed"
7. **Recommendations** — prioritized table: Priority tag, Area, Recommendation, Expected Impact
8. **Action Plan** — 4-phase: Quick Wins → Code Optimization → Runtime Performance → Advanced
9. **Code Examples — How to Fix** — include **only** examples relevant to problems found. Available samples:
   1. Code Splitting & Lazy Loading — Angular `loadChildren` + React `React.lazy()` (before/after)
   2. Optimize LCP — preconnect, preload, fetchpriority, defer, replace CSS `background-image` with `<img>`
   3. Reduce Main-Thread Work — Web Workers + `scheduler.yield()` chunking (before/after)
   4. Fix CLS — reserve space with min-height, skeleton loaders, explicit image width/height (before/after)
   5. Fix Accessibility — aria-label, form labels, heading order, color contrast (before/after)
   6. Fix Best Practices — replace `unload` with `pagehide`, enable bfcache (before/after)
   7. Analyze Bundle — webpack-bundle-analyzer, source-map-explorer, rollup-plugin-visualizer, angular.json budgets
   8. Fix INP — break up long tasks with `scheduler.yield()`, debounce input handlers, avoid synchronous layout/style recalculation in event callbacks (before/after) — include only if INP > 200ms found
