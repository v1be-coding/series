# Runs land through auto-merge, judged by frozen tests

A run merges its own PR with no human click, using GitHub auto-merge, once one required check passes. GitHub holds the rule, not the agent. The agent can't bypass the check, and its token can't edit the workflow behind it. Café production still waits for human approval after the merge (ADR-0005).

The check is one chain of jobs: lint, type checks and unit tests; Infracost (ADR-0007); branch deploy; smoke test, frozen tests and evals; a human approval of the product checklist on the branch environment; and a final job that fails unless everything before it succeeded. The final job always runs, because GitHub counts a skipped job as passing.

A run can edit almost any file, including tests, so tests it can change prove little. Frozen tests close that gap. They're written during the Spec phase by a fresh session that reads only the spec, and they're merged before the run. CI takes them, with their config and helpers, from the PR's base commit, so an edit on the branch has no effect. Code owner review covers the frozen folder, so an edit can't reach `main` unnoticed either. A wrong frozen test is a spec failure and fails the attempt.

## Considered Options

- **A person merges every PR:** simple, but it adds a click to every run that checks nothing the required check and the checklist approval haven't already covered.
- **The agent merges when it judges the work done:** rejected, because agents report passes that didn't happen, and a gate needs to sit outside what it judges.
- **Tests the run may edit, with edits flagged for review:** rejected, because a flag only shows the problem, and it relies on someone reading it.
- **A hidden test suite:** rejected for now. Hidden tests lowered legitimate results in the one benchmark we found, and public failure logs could leak them.
- **Scaffolding merged before the run:** rejected. Stubs would give tests something to reach, but feature code would then exist before the run, which weakens Rule 1 and steers the design.
- **A throwaway reference build to prove the tests:** rejected for now, because it costs about a run. `/retro` tracks wrong frozen tests, so this can be revisited.

## Consequences

- Specs get longer. A test contract must name the routes, shapes, roles, visible text and any data that must already exist, because frozen tests can use nothing else.
- Freezing stops edits, not code written only to satisfy a test. The product checklist is the backstop, so it's best run with values the tests don't use.
- A run may stop early over a frozen test that looks wrong, and must not bend app behaviour to satisfy it. It still counts as a failed attempt, so stopping gains nothing.
- An attempt allows 5 failed CI rounds, counted by CI, because CI also depends on outside services.
- Migrations only add things, because reverting a merge doesn't undo one.
- Frozen tests have one retry, and a test that needed it is reported, so flakes stay visible.
