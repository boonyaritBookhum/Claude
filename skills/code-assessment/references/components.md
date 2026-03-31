# HTML Component Patterns

Read this file ONLY when generating HTML reports (Phase 2).
CSS classes are defined in `references/styles.css`.

## HTML Page Skeleton

**dependency-analysis-report.html:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Dependency Analysis Report — [Project]</title>
  <style>/* paste styles.css here */</style>
</head>
<body>
  <div class="cover"><div class="wrapper"><!-- cover (dep pattern) --></div></div>
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

**technical-assessment-report.html** (CVE Banner and Report Nav are ALWAYS VISIBLE — outside lang-sections):
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Technical Assessment Report — [Project]</title>
  <style>/* paste styles.css here */</style>
</head>
<body>
  <div class="cover"><div class="wrapper"><!-- cover (tech pattern with .top-row) --></div></div>
  <div class="wrapper">
    <div class="cve-banner"><!-- CVE summary: always visible, before lang tabs --></div>
    <div class="report-nav"><!-- links to both reports: always visible --></div>
    <div class="lang-tabs"><!-- language toggle --></div>
    <div id="lang-en" class="lang-section active"><!-- EN sections --></div>
    <div id="lang-th" class="lang-section"><!-- TH sections --></div>
  </div>
  <div class="footer"><!-- footer --></div>
  <script>/* paste shared JS here */</script>
</body>
</html>
```

## Shared JS (bottom of both reports)

```html
<script>
function switchLang(lang,btn){
  document.querySelectorAll('.lang-section').forEach(el=>el.classList.remove('active'));
  document.querySelectorAll('.lang-tab').forEach(el=>el.classList.remove('active'));
  document.getElementById('lang-'+lang).classList.add('active');
  btn.classList.add('active');
}
</script>
```

## Component Cheat Sheet

**Cover (tech):** `div.cover > div.wrapper > div.top-row(logo-row + report-link) + h1(span=accent) + div.subtitle + div.meta-row(meta-item*)`
**Cover (dep):** `div.cover > div.wrapper > div.logo-row + h1(span=accent) + div.subtitle + div.meta-row(meta-item*) + div.report-link > a[href="technical-assessment-report.html"]` (link to tech report, no .top-row)
**Lang toggle:** `div.lang-tabs > button.lang-tab.active[onclick=switchLang('en',this)]` + `button.lang-tab[onclick=switchLang('th',this)]`
**Sections:** `div#lang-en.lang-section.active` (EN) + `div#lang-th.lang-section` (TH)
**Section header:** `div.section > div.section-title > div.icon + text`
**Alert:** `div.alert.alert-critical|alert-high|alert-info`
**Callout:** `div.callout.callout-success|callout-warn|callout-danger|callout-info`
**Score cards (counts):** `div.summary-grid > div.scard.scard-critical|high|medium|low|ok > div.num + div.lbl`
**Score cards (grades):** `div.score-grid > div.score-card > div.val(B+) + div.lbl`
**CVE count cards:** `div.cve-grid > div.cve-card.cc-critical|high|medium|low|ok > div.num + div.lbl`
**CVE banner:** `div.cve-banner > div.icon + div > h4 + p(span.cve-pill + strong + text)`
**Badge:** `span.badge.b-critical|b-high|b-medium|b-low|b-ok|b-eol`
**CVE pill:** `span.cve-pill` (monospace purple)
**Table:** `div.tbl-wrap > table > thead(th) + tbody(tr > td)` — use `.cat-row` for group headers, `.ver-old`/`.ver-new` for versions
**Card:** `div.card > h3 + content`
**Checklist:** `ul.checklist > li > div.ci.ci-ok|ci-warn|ci-fail + span`
**Arch diagram:** `div.arch-box` (pre-formatted dark bg, use `.hl`=yellow `.grn`=green `.blu`=blue `.pnk`=pink `.red`=red for colored text)
**Sub cards:** `div.sub-grid > div.sub-card > div.tag + h4 + p`
**Upgrade cmd:** `div.upgrade-block > h4 + pre(span.cmt=comment .cmd=command .pkg=package .ver=version)`
**Timeline:** `div.timeline > div.tl-item > div.tl-title + ul > li`
**Report nav:** `div.report-nav > a.report-btn.active + a.report-btn`
**OWASP grid:** `div.owasp-grid > div.owasp-item.owasp-pass|owasp-warn|owasp-fail > div.owasp-id + div.owasp-name + div.owasp-status`
**OWASP score ring:** `div.owasp-score > div.ring > span.ring-num + small + div.ring-label`
**CI/CD pipeline:** `div.pipeline-flow > div.pipe-stage.pipe-ok|pipe-warn|pipe-fail|pipe-na > div.pipe-icon + div.pipe-label + div.pipe-status`
**CI/CD maturity:** `div.maturity-bar > div.maturity-fill[data-level="1-5"] + div.maturity-labels > span*5`
**CI/CD OWASP score ring:** `div.owasp-score > div.ring > span.ring-num + small + div.ring-label` (reuse owasp-score pattern with "CI/CD Security" label)
**CI/CD OWASP grid:** reuse `div.owasp-grid > div.owasp-item` pattern with CICD-SEC-01..10 IDs
**Footer:** `div.footer > p + p`
