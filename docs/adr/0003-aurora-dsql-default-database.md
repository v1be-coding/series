# Aurora DSQL is the default database

Every episode uses Aurora DSQL unless its own ADR says otherwise. ADR-0002 rules out any database that needs a VPC, which leaves DSQL and DynamoDB. We chose DSQL for SQL. Café Copilot answers ad-hoc questions about café sales, and Ubos Na! forecasts from sales history. Both need GROUP BY and joins over data nobody designed access patterns for in advance, which DynamoDB can't do without precomputed rollups. SQL with Drizzle is also what coding agents write most reliably, and that matters for one-run episodes.

## Considered Options

- **DynamoDB:** cheaper at scale, has point-in-time recovery, and its 25 GB free tier isn't shared across accounts. We rejected it as the default because ad-hoc analytics is central to the café episodes. It stays available for simple, high-volume lookup workloads through an episode ADR.
- **RDS or Aurora Serverless v2 Postgres:** both need a VPC, so ADR-0002 rules them out on cost (NAT Gateway, or a minimum of about $21/month for RDS).

## Consequences

- The template repo provides the DSQL quirks once: a retry wrapper for optimistic-concurrency conflicts (SQLSTATE 40001), migrations that run one DDL statement per transaction, and the official `@aws/aurora-dsql-drizzle` adapter.
- Writes must be idempotent upserts keyed by client-generated IDs, because any write may be retried.
- There are no triggers or temp tables. A transaction can modify at most 3,000 rows and run at most 5 minutes.
- DSQL has no point-in-time recovery, only AWS Backup snapshots, so café production protects real data separately.
- The free tier (100k DPUs and 1 GB per month) is shared by every account in the AWS Organization.
- The template's per-IP limit and daily AI spend cap keep their counters in DynamoDB, in one table per target, as atomic counters whose rows expire by themselves. In DSQL, every request would add a write, writes from one IP would conflict with each other, and they would use the shared DPUs. These counters belong to the template, not to an episode's data, so they need no episode ADR. App data still lives in DSQL.
