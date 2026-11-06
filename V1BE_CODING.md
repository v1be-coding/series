# V1be Coding

_Spec to code. Iterate on the spec, never on the code._

A build series about one skill: turning an idea into a spec and issues clear
enough that an AI agent builds a working, deployed app in one run. Every app
has an AI feature inside it, and every episode makes the workflow a little
better. The finished apps live on the portfolio site.

## Rules

1. **Spec, then one run.** Grill → spec → tickets → frozen tests, then `/attempt`
   builds and lands it in one run. No hand edits, no "fix this" prompts.
2. **Green means deployed.** Every acceptance criterion passes, including a
   passing smoke test on the attempt's branch environment and again on
   production after the merge. Criteria are either automated (frozen tests,
   evals with thresholds) or a pass/fail product checklist written into the
   spec ahead of time, which I check on the branch environment before the
   merge. Anything about the real domain must be a smoke test. Subjective
   "taste" items are scored 1–5 and never gate.
3. **A failed run goes back to the spec.** A run fixes its own CI failures, up
   to 5 failed rounds counted by CI. It fails when those run out, when I reject
   an approval, or when it stops over a frozen test that looks wrong, which it
   must do instead of bending the app to pass. A production failure gets a fix
   PR with the same limit, then a revert. Log what the spec missed as a comment
   on the spec issue, fix the spec, and rerun on a fresh branch. At most 3
   attempts; publish the result honestly either way. Fixing setup (keys, DNS)
   is allowed but logged, and so is my ruling that an outage doesn't count
   against an attempt.
4. **One day per episode** is the target. Scope is cut to fit it, and the real
   time is recorded.
5. **Every app has an AI feature**, with rate limiting and a daily spend cap
   when it's public, backed by a monthly spend limit on the episode's own
   Anthropic workspace.
6. **No real data in public repos.** Realistic fake seed data instead; real café
   data lives only in the `collective-cafe-prod` AWS account. Secrets are
   scanned in pre-commit and CI, and never pass through a chat.
7. **Retro after every episode** (`/retro`), and the lessons go into the
   workflow changelog below.

## Foundation

Built once, before episode 0. Every episode runs on the same default stack:
Terraform on AWS (Singapore), Lambda with no VPC, Aurora DSQL, and static
frontends behind Cloudflare and CloudFront. The expected bill is about $0–2 a
month, with cost guards that stop runaway spend.

- **`v1be-infra`** holds the AWS accounts (`collective-cafe-prod` and
  `portfolio`), the GitHub deploy roles, the cost guards, the state buckets and
  the list of episodes.
- **The template repo** is where every episode starts. It comes with CI,
  pre-commit hooks, git guardrails, the smoke test, Lambda bundling, the
  CloudFront module and a pinned copy of the skills.
- **Everything lives in the `v1be-coding` GitHub org.**

The words are defined in [GLOSSARY.md](./GLOSSARY.md), and the reasons for
each choice are in [docs/adr/](./docs/adr/).

## Workflow (stock skills, plus `/attempt` and `/acceptance`)

| Phase | Steps |
| ----- | ----- |
| Setup | Claude adds the episode to `v1be-infra` in a PR. I merge it, run `org` in Windows Sandbox, which creates the repo from the template with its rules, and approve the CI apply (DNS, certificate, generated secrets, CI role). I set the Anthropic workspace's monthly limit in the Console and enter third-party keys through a `wizard` prompt straight into SSM |
| Spec  | `research` or `field-reports` as needed → `/grill-with-docs` → `/to-spec` → `/to-tickets` → `/acceptance` in a fresh session, which writes the test contract and frozen tests. I review them and merge |
| Code  | `/attempt` on a fresh branch follows `implement-spec` and marks the PR ready. CI runs the required check: unit tests, Infracost, branch environment, smoke test, frozen tests, evals, then the product checklist, which waits for my approval. The agent fixes failures, and GitHub merges when the check passes |
| Ship  | Merge to `main` deploys production and reruns the smoke test (a café episode deploys its demo first, then production waits for my approval). The agent watches this through, then I publish the episode page on the portfolio |
| After | `/retro`, workflow changelog entry |

## Each episode publishes

- **The repo:** spec and tickets as GitHub Issues, glossary, ADRs, tests, CI
- **The live app**
- **A portfolio page:** description, demo GIF, links, scorecard, a "How it
  works" section in my own words, and a retro section
- **Scorecard:** time to green, attempts, tests passing on attempt 1, build
  cost, model used, what the spec missed, frozen tests that turned out wrong.
  `/retro` reads these from GitHub where it can. Time to green leaves out time
  spent waiting for my approvals, and tests passing on attempt 1 means frozen
  tests on that attempt's first CI run

## Lineup

| #   | Episode                              | What it is                                                    | Platform                        |
| --- | ------------------------------------ | ------------------------------------------------------------- | ------------------------------- |
| 0   | Portfolio site                       | Series home + "Ask about Emiliano" chat                       | Web                             |
| 1   | AInakupo!                            | LLM tracing, cost and evals; later episodes report through it | Web + npm                       |
| 2   | Kahit Saan                           | The barkada picks where to eat; an agent proposes places      | Expo, web first                 |
| 3   | Ulam?                                | Fridge photo → what to cook, step by step                     | Expo, web first                 |
| 4   | Collective POS                       | Replaces the café's Google Sheets POS; works offline          | Android tablet (direct install) |
| 5   | Collective Cafe website + Ate \_\_\_ | Café site on its own domain + table QR menu assistant         | Web                             |
| 6   | Café Copilot                         | Ask Claude about the café's sales over MCP                    | MCP server                      |
| 7   | Ubos Na!                             | Predicts when supplies run out; restock list for Shopee       | Extends Collective POS          |
| 8   | Patok!                               | Drink photo → Facebook and Instagram post drafts              | Expo, web first                 |
| —   | Blackmoor Inn                        | AI murder-mystery RPG with an MCP detective eval              | Web + MCP                       |
