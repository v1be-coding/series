# Protecting real café data

Real data lives only in the `collective-cafe-prod` AWS account. Every public demo, every non-café episode and every branch environment lives in `portfolio` and uses seed data. The CI roles in `portfolio` have no access to `collective-cafe-prod`. Café production deploys only from `main` of the café repos, and every deploy needs a human approval, even code-only ones, because staff use the system while serving customers. GitHub enforces the `main` rule through a deployment branch rule on the `production` environment, because the token AWS checks doesn't include the branch. The café repos stay public, since required reviewers need that on GitHub's free plan. The café demo in `portfolio` deploys automatically and is where the smoke test proves each build.

Aurora DSQL has no point-in-time recovery (ADR-0003), so we cover the gap in two layers:

- A weekly AWS Backup snapshot of the café backend, kept for **14 days**, plus one before each café production deploy that runs a migration, kept for **7 days**. DSQL backups are full copies, so we keep only a few. A backup may take up to an hour just to start, so only migrations, the deploys that can damage data, wait for one. The newest snapshot is always younger than the tablet's window, which is why a weekly one is needed even when deploys are rare.
- The POS tablet keeps synced sales locally for **15 days**. After a restore, the up to 7 days of sales lost since the last snapshot can be refilled from the tablet.

Menu and supply changes aren't on the tablet, so a restore can lose up to a week of them. A problem noticed more than 14 days later can't be fixed from a snapshot. We accepted both to keep the setup lean, since menu changes are rare and easy to redo.
