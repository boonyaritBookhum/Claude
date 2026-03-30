# Lighthouse Report — HTML Component Patterns

Read this file ONLY when generating the HTML report (Step 4).
CSS classes are defined in `references/styles.css`.

## External Dependencies (in `<head>`)

```html
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.7/dist/chart.umd.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.2.0/dist/chartjs-plugin-datalabels.min.js"></script>
```

## Shared JS (bottom of file)

```html
<script>
function switchLang(lang,btn){
  document.querySelectorAll('.lang-section').forEach(el=>el.classList.remove('active'));
  document.querySelectorAll('.lang-tab').forEach(el=>el.classList.remove('active'));
  document.getElementById('lang-'+lang).classList.add('active');
  btn.classList.add('active');
}
document.addEventListener('DOMContentLoaded', () => {
  // 1. Populate .code-gutter with line numbers from .code-content
  document.querySelectorAll('.code-window').forEach(w => {
    const pre = w.querySelector('.code-content');
    const gutter = w.querySelector('.code-gutter');
    if (pre && gutter) {
      const lines = pre.textContent.split('\n').length;
      gutter.innerHTML = Array.from({length: lines}, (_, i) => `<div>${i + 1}</div>`).join('');
    }
  });
  // 2. Add Copy button to each .code-titlebar
  document.querySelectorAll('.code-titlebar').forEach(bar => {
    const btn = document.createElement('button');
    btn.className = 'copy-btn';
    btn.textContent = 'Copy';
    btn.onclick = () => {
      const code = bar.closest('.code-window').querySelector('.code-content').textContent;
      navigator.clipboard.writeText(code).then(() => {
        btn.textContent = 'Copied!';
        btn.classList.add('copied');
        setTimeout(() => { btn.textContent = 'Copy'; btn.classList.remove('copied'); }, 2000);
      });
    };
    bar.appendChild(btn);
  });
  // 3. Initialize ALL Chart.js charts with data from analysis
  //    - Register ChartDataLabels plugin
  //    - scoreColor helper for conditional coloring
  //    - Custom plugins for threshold lines on bar charts
});
</script>
```

## Component Cheat Sheet

**Cover:** `div.cover > h1(span=accent) + div.subtitle + div.meta-row(div.meta-item*)`
**Lang toggle:** `div.lang-tabs > button.lang-tab.active[onclick=switchLang('en',this)] + button.lang-tab[onclick=switchLang('th',this)]`
**Sections:** `div#lang-en.lang-section.active` (EN) + `div#lang-th.lang-section` (TH)
**Section header:** `div.section > div.section-title > div.icon + text`
**Stats grid:** `div.stats-grid > div.stat-card > div.num + div.lbl`
**Score circle:** `span.score-circle.green|orange|red`
**Table:** `div.tbl-wrap > table > thead(th) + tbody(tr > td)` — use `.group-row` for module group headers
**Chart:** `div.chart-grid > div.chart-card > h4 + canvas`
**Finding:** `div.finding > span.finding-num + text`
**File card:** `div.file-card > h4 + div.file-stats(div.file-stat > div.val + div.lbl) + bilingual text + div.safe-fix(h5 + ol + span.risk-tag)`
**Impact:** `span.badge.impact-critical|impact-high|impact-medium` (inline)
**Priority:** `span.priority-tag.p-critical|p-high|p-medium|p-low`
**Risk tag:** `span.risk-tag.risk-zero|risk-low|risk-medium`
**Timeline:** `div.timeline > div.tl-item > div.tl-title + ul > li`
**Footer:** `div.footer > p + p`

**Code block (VS Code style):**
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

**Before/After pattern:**
```html
<div class="ba-grid">
  <div class="ba-panel bad"><div class="ba-badge">✗ BEFORE — desc</div><div class="code-window">...</div></div>
  <div class="ba-panel good"><div class="ba-badge">✓ AFTER — desc</div><div class="code-window">...</div></div>
</div>
```

## Chart.js Data Structure

Embed page data as JS array:
```javascript
const pages = [
  { name: 'page-name', perf: 30, fcp: 2.2, lcp: 4.5, inp: 320, tbt: 640, cls: 0.300, tti: 5.1, si: 7.5, mt: 3.0, js: 1.5, unusedJS: 2678, payload: 6069 },
  // fcp/lcp/tti/si = seconds | tbt/inp = ms | cls = unitless | mt/js = seconds | unusedJS/payload = KiB
  // inp: null if Lighthouse < v10 | tti: may be absent in newer Lighthouse versions
  // ... sorted by perf ascending
];
```

**Color helpers** — use these when coloring individual metric values in charts and tables:
```javascript
// For Lighthouse category scores (0–100)
function scoreColor(v) { return v >= 90 ? '#0cce6b' : v >= 50 ? '#ffa400' : '#ff4e42'; }

// For individual metrics — pass numericValue as stored in pages[]
function lcpColor(s)  { return s <= 2.5  ? '#0cce6b' : s <= 4.0  ? '#ffa400' : '#ff4e42'; } // seconds
function fcpColor(s)  { return s <= 1.8  ? '#0cce6b' : s <= 3.0  ? '#ffa400' : '#ff4e42'; } // seconds
function tbtColor(ms) { return ms <= 200 ? '#0cce6b' : ms <= 600 ? '#ffa400' : '#ff4e42'; } // ms
function clsColor(v)  { return v <= 0.1  ? '#0cce6b' : v <= 0.25 ? '#ffa400' : '#ff4e42'; } // unitless
function inpColor(ms) { return ms == null ? '#94a3b8' : ms <= 200 ? '#0cce6b' : ms <= 500 ? '#ffa400' : '#ff4e42'; } // ms; null (Lighthouse <v10) → gray
```
