# Workflow changelog

Changes to how the series is built, newest first. Each episode's retro adds an entry when it changes the workflow.

## 2026-11-06: Foundation, phase B

- The ADRs, glossary and series rules moved here from the working folder.
- Member accounts moved from a list of denied services to an allowlist (ADR-0007), so an episode's Setup now adds any new AWS service.
- Every change to an org repo now goes through a PR, including repos with no branch protection.
- Member accounts are limited to Singapore, apart from global services and ACM (ADR-0007).
