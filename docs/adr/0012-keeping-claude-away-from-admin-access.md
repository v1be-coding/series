# Keeping Claude away from admin access

Claude writes all the Terraform, so its code must never run beside an admin login. Claude works in a dev container, the user runs `org` and `accounts` in Windows Sandbox, and GitHub Actions runs `edge` and `episodes` after a merge and the user's approval. Episode repos are created by the `org` stack from the Sandbox, so CI holds no GitHub token.

| Environment | Runs | Holds |
| ------------------- | --------------------------------- | ------------------------------------------- |
| Dev container       | Claude                            | `v1be-bot`'s token and `claude-readonly`    |
| Windows Sandbox     | `org`, `accounts`, by the user    | The user's admin, GitHub owner logins       |
| GitHub Actions      | `edge`, `episodes`                | Cloudflare token and OIDC roles, on protected environments |

`org` can't move to CI, because a role in the management account sits outside every SCP, and a leak from it could remove the guardrails. `accounts` creates the login CI uses, so it can't move either.

`v1be-bot` can open PRs but can't edit workflows or merge in `v1be-infra`. The workflow that runs the required check comes from the PR, so a bot that could edit workflows could switch the check off. `claude-readonly` can view `portfolio` and read its logs and view the café account. It has no access to the management account and is denied secret values, state and KMS decrypt.

This separation doesn't stop bad code once it's merged. Terraform runs merged code beside its tokens, so the user reading the diff is the real gate. The separation limits what a mistake or prompt injection can reach before that point, and it means the management account is never reachable from anything Claude wrote without the user running it in the Sandbox.

## Considered Options

- **Claude runs Terraform with the user's SSO session:** the simplest, but Claude's code would run beside an admin login on the user's PC.
- **The user runs all Terraform:** removes Claude from the login, but the code still runs beside admin access on Windows, and it would make every Setup a manual run.
- **WSL2 as the separation:** rejected because it shares the Windows files and the user's account by default, and it ranked sixth of the options researched.
- **Terraform in CI creates episode repos:** rejected because it needs a GitHub token able to create and change repos, sitting beside merged code.
- **The user creates episode repos by hand:** rejected because settings and environments would become by-hand steps too, against ADR-0006.
- **One required approval on `main`:** rejected as the merge rule because the bot can't push workflows and the user can't approve their own PR, so no workflow change could land. Only the user can merge instead, which is also stricter on Claude.

## Consequences

- This repo's CI now plans and applies, reversing the earlier rule that it only checks code. Pull requests still never run `terraform plan`.
- Adding an episode takes one Sandbox run, so that step needs the user at their PC.
- The user commits workflow changes themselves, from Claude's draft.
- `edge` and `episodes` state moves to `v1be-infra-state` in `portfolio`, so CI never touches the management account (ADR-0011).
- An SCP on both member accounts denies `iam:CreateUser` and `iam:CreateAccessKey`, because people use Identity Center and CI uses OIDC. Long-lived keys would outlast the sessions this separation relies on.
- The separation only works if the user's PC is sound, so UAC prompts stay on.
- Windows Sandbox doesn't shield itself from the host account. `wsb exec` runs commands in a running Sandbox, including as SYSTEM, with no prompt, even with UAC on Always notify. So the dev container comes before every admin login, and Claude never runs on Windows beside a logged-in Sandbox.
