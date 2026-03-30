# OWASP Top 10 CI/CD Security Risks Checklist

Read this file when running Phase 1e (Dimension 2) of code-assessment.

- **CICD-SEC-01: Insufficient Flow Control Mechanisms** — can code reach production without review/approval? Missing branch protection, no required reviewers, direct push to main, no merge gates
- **CICD-SEC-02: Inadequate Identity and Access Management** — overly permissive CI/CD user roles, shared service accounts, no RBAC for pipeline resources, missing MFA for CI/CD platform access
- **CICD-SEC-03: Dependency Chain Abuse** — dependency confusion risk, typosquatting exposure, no private registry configured, missing namespace scoping, auto-merge of dependency updates without review
- **CICD-SEC-04: Poisoned Pipeline Execution (PPE)** — pipeline configs modifiable by contributors (Direct PPE), build triggered by unreviewed code changes (Indirect PPE), 3rd-party pipeline triggers without validation (3rd-Party PPE)
- **CICD-SEC-05: Insufficient PBAC (Pipeline-Based Access Controls)** — pipeline jobs with excessive permissions, shared credentials across environments, no scoping of secrets per branch/env, pipeline can access production from feature branches
- **CICD-SEC-06: Insufficient Credential Hygiene** — hardcoded secrets in pipeline configs, unrotated tokens, secrets in logs/artifacts, credentials accessible to all pipeline stages, no secret scanning in CI
- **CICD-SEC-07: Insecure System Configuration** — CI/CD platform running with default settings, unnecessary plugins/features enabled, self-hosted runners without hardening, missing network segmentation
- **CICD-SEC-08: Ungoverned Usage of 3rd Party Services** — unvetted GitHub Actions/plugins, OAuth integrations with excessive scopes, 3rd-party services with write access to repo/pipeline, no inventory of CI/CD integrations
- **CICD-SEC-09: Improper Artifact Integrity Validation** — unsigned build artifacts, no checksum verification, artifacts pulled over insecure channels, no provenance tracking, container images without signatures
- **CICD-SEC-10: Insufficient Logging and Visibility** — no audit trail for pipeline changes, missing logs for secret access, no alerting on pipeline anomalies, inability to detect unauthorized pipeline modifications
