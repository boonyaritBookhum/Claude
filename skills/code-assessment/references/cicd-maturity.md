# CI/CD Pipeline Maturity — Dimension 1 Checklist

Read this file when running Phase 1e (Dimension 1) of code-assessment.

Evaluate each area and grade: ✅ Configured / ⚠️ Partial / ❌ Missing / ➖ N/A

- **Pipeline Security** — secrets in env vars vs vault/secrets manager, least-privilege tokens, no plaintext credentials in configs
- **Build Integrity** — pinned action/image versions (not `:latest`), reproducible builds, dependency lockfile enforced, SBOM generation
- **Test Automation** — unit/integration/e2e tests in pipeline, coverage gates, fail-on-test-failure, test parallelization
- **Code Quality Gates** — linter, formatter, SAST/DAST scanning, type checking, PR review requirements
- **Deployment Safety** — staging/canary/blue-green strategy, rollback mechanism, health checks, deploy approvals for prod
- **Supply Chain Security** — dependency scanning in pipeline (Dependabot/Renovate/Snyk), container image scanning, signed commits/artifacts
- **Branch Protection** — protected main branch, required reviews, no force push, status checks required
- **Monitoring & Observability** — deploy notifications, post-deploy smoke tests, alerting integration

Generate overall CI/CD maturity score: Beginner → Basic → Intermediate → Advanced → Expert

## Note
- If no pipeline config found: report "No CI/CD pipeline detected", grade as "Beginner", provide setup recommendations. Mark all CICD-SEC risks as FAIL.
