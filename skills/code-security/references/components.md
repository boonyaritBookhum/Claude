# Security Report — HTML Component Patterns

Read this file ONLY when generating the HTML report (Phase 3).
CSS classes are defined in `references/styles.css`.

## HTML Page Skeleton

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Security Audit Report — [Project Name]</title>
  <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet">
  <style>/* paste styles.css here */</style>
</head>
<body>
  <div class="cover"><!-- cover section --></div>
  <div class="wrapper">
    <div class="lang-tabs"><!-- language toggle --></div>
    <div id="lang-en" class="lang-section active"><!-- EN sections --></div>
    <div id="lang-th" class="lang-section"><!-- TH sections --></div>
  </div>
  <div class="footer"><!-- footer --></div>
  <script>/* paste shared JS here */</script>
</body>
</html>
```

## Shared JS (bottom of report)

```html
<script>
function switchLang(lang,btn){
  document.querySelectorAll('.lang-section').forEach(el=>el.classList.remove('active'));
  document.querySelectorAll('.lang-tab').forEach(el=>el.classList.remove('active'));
  document.getElementById('lang-'+lang).classList.add('active');
  btn.classList.add('active');
}
document.addEventListener('DOMContentLoaded',()=>{
  document.querySelectorAll('.code-window').forEach(w=>{
    const pre=w.querySelector('.code-content');
    const gutter=w.querySelector('.code-gutter');
    if(pre&&gutter){
      const lines=pre.textContent.split('\n').length;
      gutter.innerHTML=Array.from({length:lines},(_,i)=>`<div>${i+1}</div>`).join('');
    }
  });
});
</script>
```

## Component Cheat Sheet

**Cover:** `div.cover > div.logo-row(div.logo-box=🔐 + h2=project) + h1(span=accent) + div.subtitle + div.risk-badge.risk-critical|high|medium|low + div.meta-row(div.meta-item*)`
**Lang toggle:** `div.lang-tabs > button.lang-tab.active[onclick=switchLang('en',this)] + button.lang-tab[onclick=switchLang('th',this)]`
**Sections:** `div#lang-en.lang-section.active` (EN) + `div#lang-th.lang-section` (TH)
**Section header:** `div.section > div.section-title > div.icon + text`
**Finding card:** `div.finding-card.finding-critical|high|medium|low|info > span.sev.sev-X + strong(title) + span.loc(file:line) + div.code-window + div.finding-desc + div.finding-fix`
**Severity badge:** `span.sev.sev-critical|sev-high|sev-medium|sev-low|sev-info`
**Summary grid:** `div.summary-grid > div.scard.scard-critical|high|medium|low|info > div.num + div.lbl`
**Risk gauge:** `div.risk-gauge > div.score-num + div.score-label + div.score-bar > div.score-fill[style="width:X%;background:#color"]`
  Fill color per level: Critical `#dc2626`, High `#ea580c`, Medium `#ca8a04`, Low `#16a34a`
**Callout:** `div.callout.callout-success|warn|danger|info`
**Table:** `div.tbl-wrap > table > thead(th) + tbody(tr > td)` — location in `span.loc`
**Coverage grid:** `div.coverage-grid > div.cov-item.cov-clean|warn|critical > div.cov-head(div.cov-icon(emoji) + span.cov-name) + div.cov-status("✅ Clean"|"⚠️ N issues"|"🔴 N issues")`
**Secret value:** `span.secret-val` — e.g. `AKIA****WXYZ` (mask middle chars)
**File stats:** `div.file-stats > span(strong=N + " files scanned") + span(strong=N + " with findings") + span(strong=N + " clean")`
**File index:** `div.file-index > div.file-group > div.file-group-title("Files with Findings"|"Clean Files") + div.file-item.file-hit|file-warn|file-clean > span.file-icon(🔴|🟡|✅) + span.file-path(path/to/file.go) + span.file-count("3 findings"|"Clean")`
  — Group files: `.file-hit` = has CRITICAL/HIGH, `.file-warn` = has MEDIUM/LOW only, `.file-clean` = no findings. Sort by finding count desc.
**Timeline:** `div.timeline > div.tl-item > div.tl-title + ul > li`
**Footer:** `div.footer > p + p`

## Code Block — VS Code Style

```html
<div class="code-window">
  <div class="code-titlebar">
    <div class="code-dots"><span></span><span></span><span></span></div>
    <div class="code-filename">path/to/file.go:42</div>
  </div>
  <div class="code-body">
    <div class="code-gutter"></div>
    <pre class="code-content"><span class="t-var">query</span> := <span class="t-str">"SELECT * FROM users WHERE id = "</span> + <span class="t-danger">userID</span></pre>
  </div>
</div>
```

Use `.t-danger` (red bold) to highlight the vulnerable portion of each code snippet.

**Syntax token classes:** `.t-kw` keyword, `.t-str` string, `.t-cm` comment, `.t-fn` function, `.t-var` variable, `.t-num` number, `.t-danger` vulnerable code (red bold)
