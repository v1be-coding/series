# Lambda only, outside any VPC

Every episode runs on AWS Lambda outside a VPC. Realtime features use API Gateway WebSockets. There are no containers, always-on servers or private networks. Traffic is near zero, and a Lambda outside a VPC reaches the internet (the Claude API, for example) for free. A Lambda inside a VPC needs a NAT Gateway at about $43/month in ap-southeast-1, which on its own is more than the whole series is meant to cost. This rules out server-style databases that only live inside a VPC (RDS, Aurora provisioned), so the default database has to be reachable over a public, IAM-authenticated endpoint.

## Consequences

- An episode that needs a long-lived process or a VPC resource must record why in its own ADR.
- Lambda's 15-minute limit caps any single request; long eval suites run in CI instead.
