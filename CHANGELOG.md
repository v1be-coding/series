# Workflow changelog

Changes to how the series is built, newest first. Each episode's retro adds an entry when it changes the workflow.

## 2026-11-06: Foundation, phase E slice 2

- `v1be-infra` now owns each episode's production distribution, bucket, API shell and DNS, so a run can't reach the origin secret (ADR-0010).

## 2026-11-06: Foundation, phase E design

- Runs now land themselves through GitHub auto-merge, judged by one required check (ADR-0013).
- A new `/attempt` command follows `implement-spec`, then lands the PR and fixes what fails, up to 5 rounds.
- A new `/acceptance` command writes the test contract and frozen tests in the Spec phase, before any code exists.
- The product checklist moved before the merge, onto the branch environment.
- If production fails after a merge, the agent tries a fix PR first, then a revert. This covers café production too.
- `/setup-matt-pocock-skills` now runs once on the template, not for each episode.
- The template carries a pinned copy of the skills, because skills load only up to a repo's root.
- The bot's token gained Issues access, because the bot files specs and tickets.
- The Infracost key is an org Actions secret, the one exception to keeping secrets in SSM (ADR-0009).
- Portfolio CI roles no longer trust every branch (ADR-0011).

## 2026-11-06: Foundation, phase B

- The ADRs, glossary and series rules moved here from the working folder.
- Member accounts moved from a list of denied services to an allowlist (ADR-0007), so an episode's Setup now adds any new AWS service.
- Every change to an org repo now goes through a PR, including repos with no branch protection.
- Member accounts are limited to Singapore, apart from global services and ACM (ADR-0007).
