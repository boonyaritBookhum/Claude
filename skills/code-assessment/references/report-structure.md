# Report Structure

Read this file ONLY when generating HTML reports (Phase 2).

---

## Report 1: dependency-analysis-report.html

Structure (EN section, repeat in TH with Thai text):
1. **Cover** — logo-box=📦, link to technical report
2. **Audit Summary** — summary-grid (5 scard: CRITICAL/HIGH/MEDIUM/LOW/OK counts) + alert boxes
3. **Package Audit Table** — columns: Package, Project, Current(.ver-old), Latest(.ver-new), CVE/Risk, Urgency(.badge)
   - Group by subproject using `.cat-row` headers
4. **Upgrade Commands** — upgrade-block per urgency (CRITICAL→HIGH→MEDIUM)
5. **Code-Level Security Findings** — table: Severity, Project, Location, Finding (if deep scan)
6. **OWASP Top 10 (2025) Compliance** — owasp-score ring (X/10) + owasp-grid (10 items A01–A10:2025, each PASS/WARN/FAIL with brief finding)
7. **CI/CD Pipeline Assessment** — maturity-bar (level 1-5) + pipeline-flow (stages with status) + table: Area, Status, Finding, Recommendation
8. **OWASP Top 10 CI/CD Security Risks** — cicd-owasp-score ring (X/10) + owasp-grid (CICD-SEC-01 to CICD-SEC-10, each PASS/WARN/FAIL with brief finding)
9. **Upgrade Plan** — timeline: 🔴 Week 1, 🟡 Sprint 1, 🟠 Sprint 2-3, 🟢 Next Quarter

Thai section titles: สรุปผลการตรวจสอบ, ตารางวิเคราะห์ Package, คำสั่ง Upgrade, ปัญหาความปลอดภัยระดับโค้ด, การประเมิน OWASP Top 10 (2025), การประเมิน CI/CD Pipeline, ความเสี่ยงด้านความปลอดภัย CI/CD (OWASP Top 10), แผนการอัปเกรด
Thai timeline: สัปดาห์ที่ 1, Sprint 1 (2 สัปดาห์), Sprint 2–3, ไตรมาสถัดไป

---

## Report 2: technical-assessment-report.html

Structure (EN section, repeat in TH):
1. **Cover** — logo-box=🛢️, link to dependency report
2. **CVE Banner** — cve-banner (before lang tabs, always visible)
3. **Report Nav** — buttons linking both reports
4. **Executive Summary** — callout-success + score-grid (8 grades: Code Quality, Architecture, Dependency Health, Security, OWASP Compliance, CI/CD Maturity, CI/CD Security, Stability) + cve-grid
5. **Architecture** — arch-box (ASCII diagram with colored spans) + sub-grid cards
6. **Code Quality Strengths** — card per subproject with checklist (ci-ok/ci-warn)
7. **CVE Table** — columns: CVE(.cve-pill), Package, Project, Vulnerable Ver(red), Fixed Ver(green), Impact
8. **Code Issues** — columns: #, Severity(.badge), Location(code+small), Issue & Fix(strong+em)
9. **OWASP Top 10 (2025) Compliance** — owasp-score ring (X/10 categories passed) + owasp-grid (A01–A10:2025, each with PASS/WARN/FAIL status) + expandable findings per category with file:line evidence + callout linking to https://owasp.org/Top10/2025/ + note changes from 2021 (A03 Supply Chain & A10 Exceptional Conditions are new)
10. **CI/CD Pipeline Assessment** — maturity-bar (Beginner→Expert) + pipeline-flow diagram (Source→Build→Test→Security→Deploy stages) + detailed table: Area, Status(✅/⚠️/❌/➖), Current State, Recommendation + callout for top 3 CI/CD improvements
11. **OWASP Top 10 CI/CD Security Risks** — cicd-owasp-score ring (X/10 risks passed) + owasp-grid (CICD-SEC-01 to CICD-SEC-10, each with PASS/WARN/FAIL status and evidence) + detailed table: Risk ID, Risk Name, Status, Finding, Recommendation + callout linking to https://owasp.org/www-project-top-10-ci-cd-security-risks/
12. **Dependency Health** — cross-project comparison table + callout-warn linking to dep report
13. **Stability** — 2 cards: "Currently Stable ✓" (list) + "Stability Risks ⚠️" (bold+list)
14. **Action Plan** — timeline: 🔴 Week 1 (CVEs+critical OWASP fails+critical CI/CD-SEC risks+critical code), 🟡 Sprint 1 (deprecated+high fixes+CI/CD security hardening+PPE mitigation), 🟠 Sprint 2-3 (framework+OWASP compliance+CI/CD maturity+supply chain security), 🟢 Next Quarter (long-term+full OWASP 2025 compliance+CI/CD SEC optimization+full audit trail)

Thai section titles: บทสรุปผู้บริหาร, สถาปัตยกรรมระบบ, จุดแข็งด้านคุณภาพโค้ด, ช่องโหว่ความปลอดภัย, ปัญหาคุณภาพโค้ด, การประเมิน OWASP Top 10 (2025), การประเมิน CI/CD Pipeline, ความเสี่ยงด้านความปลอดภัย CI/CD (OWASP Top 10), สรุป Dependency, เสถียรภาพการทำงาน, แผนการดำเนินการ
