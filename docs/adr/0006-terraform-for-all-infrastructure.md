# Terraform for all infrastructure

Terraform defines the Foundation (AWS accounts, deploy roles, cost guards) and every episode. Its Cloudflare and GitHub providers also manage DNS records and the GitHub organization's settings, so no part of the infrastructure is set up by hand-clicking in a console. We first chose SST (ADR-0004) for its short TypeScript config, then dropped it before building anything because reviews raised concerns about how actively it's maintained. Terraform is the most widely used IaC tool, coding agents know it best, and `terraform plan` gives a readable preview before every change.

## Considered Options

- **AWS CDK:** TypeScript, maintained by AWS, and it packages Lambdas for you. We rejected it because it deploys through CloudFormation, which is slow and can get stuck in failed-rollback states. That's a bad failure mode for an episode that has to finish in one run.
- **AWS CLI scripts:** these aren't infrastructure as code. They keep no state, so there's no diff, no partial update and no clean teardown of branch environments.

## Consequences

- A few steps stay by hand because nothing can do them for us: signing up for each service, enabling IAM Identity Center (which creates the Organization that Terraform then imports), and setting each Anthropic workspace's monthly limit in the Console.
- The Foundation creates one Terraform state bucket per AWS account, plus `v1be-infra-state` in `portfolio` for the stacks CI runs (ADR-0012).
- The template repo provides the Lambda bundling step (esbuild to a zip) that SST would have handled.
- Server rendering would need a hand-wired adapter, so frontends are static by default (ADR-0008).
