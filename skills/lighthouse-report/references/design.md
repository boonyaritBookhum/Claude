# Lighthouse Report — Design & Code Examples Reference

Read this file ONLY when generating the HTML report (Step 4).

## External Dependencies (in `<head>`)

```html
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.7/dist/chart.umd.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.2.0/dist/chartjs-plugin-datalabels.min.js"></script>
```

## Design System

- **Language toggle**: `body.th` class toggles `.lang-en` / `.lang-th` spans
- **Layout**: white cards (`.card`) on `#f0f2f5` background
- **Score colors**: green (90-100), orange (50-89), red (0-49) → `.score-green`, `.score-orange`, `.score-red`
- **Responsive**: `@media(max-width:768px)` breakpoints
- **Fonts**: JetBrains Mono for code, Segoe UI + Noto Sans Thai for body
- **Print**: `@media print` for cards and header

## Code Block — VS Code Style

```html
<div class="code-window">
  <div class="code-titlebar">
    <div class="code-dots"><span></span><span></span><span></span></div>
    <div class="code-filename">filename.ts</div>
  </div>
  <div class="code-body">
    <div class="code-gutter"></div>
    <pre class="code-content">...code with syntax highlighting spans...</pre>
  </div>
</div>
```

**Styling:** dark bg `#1e1e2e` Catppuccin Mocha, macOS dots (red/yellow/green), gutter `#181825`, shadow `0 4px 24px rgba(0,0,0,0.18)`, copy button via JS on DOMContentLoaded.

**Syntax token classes:**
- `.t-kw` keyword — purple `#cba6f7`
- `.t-str` string — green `#a6e3a1`
- `.t-cm` comment — gray italic `#6c7086`
- `.t-fn` function — cyan `#89dceb`
- `.t-tag` HTML tag — blue `#89b4fa`
- `.t-at` attribute — yellow `#f9e2af`
- `.t-num` number — orange `#fab387`
- `.t-typ` type — pink `#f38ba8`

**Before/After pattern:**
```html
<div class="ba-grid">
  <div class="ba-panel bad"><div class="ba-badge">✗ BEFORE — desc</div><div class="code-window">...</div></div>
  <div class="ba-panel good"><div class="ba-badge">✓ AFTER — desc</div><div class="code-window">...</div></div>
</div>
```

## Required Code Examples (7 sections)

1. **Code Splitting & Lazy Loading** — Angular `loadChildren` + React `React.lazy()` (before/after)
2. **Optimize LCP** — preconnect, preload, fetchpriority, defer, replace CSS `background-image` with `<img>`
3. **Reduce Main-Thread Work** — Web Workers + `scheduler.yield()` chunking (before/after)
4. **Fix CLS** — reserve space with min-height, skeleton loaders, explicit image width/height (before/after)
5. **Fix Accessibility** — aria-label, form labels, heading order, color contrast (before/after)
6. **Fix Best Practices** — replace `unload` with `pagehide`, enable bfcache (before/after)
7. **Analyze Bundle** — webpack-bundle-analyzer, source-map-explorer, rollup-plugin-visualizer, angular.json budgets

## Chart.js Data Structure

Embed page data as JS array:
```javascript
const pages = [
  { name: 'page-name', perf: 30, fcp: 2.2, lcp: 4.5, tbt: 640, cls: 0.300, si: 7.5, mt: 3.0, js: 1.5, unusedJS: 2678, payload: 6069 },
  // ... sorted by perf ascending
];
```

## JavaScript (bottom of file)

```javascript
function switchLang(lang) {
  document.body.classList.toggle('th', lang === 'th');
  document.querySelectorAll('.lang-toggle button').forEach(btn => {
    btn.classList.toggle('active', btn.textContent.trim() === lang.toUpperCase());
  });
}
document.addEventListener('DOMContentLoaded', () => {
  // 1. Populate .code-gutter with line numbers from .code-content
  // 2. Add Copy button to each .code-titlebar
  // 3. Initialize ALL Chart.js charts with data from analysis
  //    - Register ChartDataLabels plugin
  //    - scoreColor helper for conditional coloring
  //    - Custom plugins for threshold lines on bar charts
});
```
