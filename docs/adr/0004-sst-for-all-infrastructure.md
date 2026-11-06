---
status: superseded by ADR-0006
---

# SST for all infrastructure, as an experiment

Both the Foundation (accounts, deploy roles, budgets) and every episode app are defined with SST v3. We're testing a tool that's new to us. The bet is that SST's short TypeScript config is easier for a coding agent to get right in one run: it handles Next.js deploys, branch environments, linked resources and secrets, all of which Terraform or CDK would need hand-wired. Terraform is the most widely used IaC tool and has more training data, so this bet can lose.

## Revisit when

A retro finds a failed attempt whose cause was SST itself (its config, a deploy quirk, or the agent misusing its APIs) rather than the spec. Two such episodes means re-evaluating, with Terraform or CDK as the alternatives.
