# Secrets in SSM Parameter Store, scoped per episode

Secrets are SSM Parameter Store SecureString parameters, encrypted with the AWS-managed key and stored under one path per episode (`/<episode>/...`). The template repo's Lambda module grants `ssm:GetParameter` on its own episode's path only, so one episode can never read another's keys. Branch environments share their episode's secrets, and café production's secrets live only in `collective-cafe-prod`. CI and Lambdas can only read them. Values are written during Setup in one of two ways:

- **Generated secrets** (internal signing keys): `v1be-infra` creates them with Terraform's `random_password`, and nobody needs to see them. They do sit in Terraform state, which `v1be-infra`'s CI can read.
- **Third-party keys** (Google, Facebook; Anthropic needs none, see below): the human creates them in the provider's console and enters them through a `wizard` prompt that writes straight to SSM. They never pass through a chat with Claude, because a transcript would keep them.

Anthropic is the exception, since 2026-11-17: there is no Anthropic key at all.

- Each episode's production Lambda asks AWS STS for a token, and Anthropic's workload identity federation swaps it for a short-lived Claude token. Anthropic recommends this over long-lived keys for cloud workloads.
- Anthropic trusts one exact role ARN, through a rule on its side. The episode's boundary lets roles ask only for Anthropic-audience tokens lasting 5 minutes or less.
- What Setup stores is four IDs that aren't secret, in `/<episode>/third-party/anthropic-federation`.
- A leak leaves nothing reusable to copy, and there's nothing to rotate.
- If federation ever fails, the fallback is a service account key, written to SSM like any other third-party key.

Each episode's path has two branches: `/<episode>/third-party/...` for keys from outside vendors, and `/<episode>/internal/...` for secrets we generate. An SCP on both member accounts stops Identity Center sessions and `v1be-infra`'s CI roles reading the `third-party` branch directly. Terraform never needs those values, so the block costs nothing. It can't cover `internal`, because Terraform reads those on every refresh. The SCP isn't absolute, since an admin could work around it, but doing so takes several visible steps.

The Cloudflare origin header is the one secret that isn't in SSM. A CloudFront Function has no network access and can't call SSM, so `v1be-infra` writes the value straight to the Cloudflare transform rule and a CloudFront KeyValueStore (ADR-0010). It's also a single series-wide value. Cloudflare can only add a fixed header value, so one value per episode would need one transform rule per episode, and the free plan allows 10 per domain in total. A leaked value would let someone skip the Cloudflare edge for every episode, but the app-level rate limits and spend caps still apply behind it.

The Infracost key is the other exception. It's a single GitHub organization Actions secret, entered through a `wizard`, and only the Infracost step receives it. It's series-wide and used only in CI, and a leak would expose a pricing key, not an account.

Secrets Manager is the dedicated service and offers the same IAM scoping, but it costs $0.40 per secret per month. At 10–15 secrets that's the series' entire monthly budget. Its one extra feature is automatic rotation, and none of our API keys support it. A secret that needs rotation can use Secrets Manager through an episode ADR.
