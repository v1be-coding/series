# Static frontends and Lambda APIs

Every episode's web frontend is a static build (Vite + React, Expo's web export, or a static Next.js export) served from S3 through CloudFront, and it talks to a Lambda API. Nothing renders on the server by default. With Terraform (ADR-0006), server rendering on Lambda would mean hand-wiring a third-party adapter that the agent is likely to get wrong. Nothing in the lineup needs it: the AI features are API calls, and the portfolio is mostly static content.

An episode needs an ADR for server rendering if it has dynamic pages that crawlers must read: search indexing of per-item pages, or link previews in Messenger or Facebook for URLs that only exist at runtime. A small Lambda that serves just the preview tags often covers the link-preview case without full server rendering.
