# ADR-0001: Use a serverless architecture instead of Lightsail

- **Status:** Proposed
- **Date:** 2026-11-24
- **Decider:** Grant Guan

## Context

- Traffic will be low and spiky: roughly 150 designs/month during the demo, roughly 600/month at launch.
- Budget: about $500 cash plus $100–200 in AWS credits. Time: 3–5 hours a week.
- The project must also serve as a portfolio piece for the AWS Solutions Architect Associate path, so the architecture needs to be explainable in interviews.
- The dominant cost is the AI image call (~$0.08 per design), which is the same on any hosting option.

## Options considered

| Option | Pros | Cons | Est. monthly cost (demo) |
| --- | --- | --- | --- |
| A. Serverless: S3 + CloudFront, API Gateway, Lambda, SQS, DynamoDB | Near-zero idle cost; scales per request; no OS patching; managed throttling and concurrency caps | More services to learn and wire together | ~$15 (≈$12 of it Bedrock) |
| B. Lightsail instance running web + API | One server, simple mental model, fixed price | Idle cost every month; OS patching, TLS renewal, backups; single point of failure; abuse limits must be written in app code | ~$7 instance + ~$12 Bedrock |
| C. Amplify Hosting + Lambda | Git-push deploys | Hides the S3/CloudFront/OAC wiring I need to understand and explain | Similar to A |

## Decision

We chose **A (serverless)** because, at this traffic, the non-AI infrastructure costs almost nothing, and managed services (SQS concurrency limits, API throttling, WAF) give real-time cost and abuse controls without running a server.

## Consequences

- **Good:** no idle compute; nothing to patch; the async queue caps spend; the architecture covers the services and patterns the Solutions Architect exam tests.
- **Bad / trade-offs:** more moving parts; cold starts on Lambda (acceptable for this use case); I must learn CDK.
- **Follow-ups:** ADR-0002 (CDK vs SAM), ADR-0004 (S3 + CloudFront vs Amplify), ADR-0005 (async generation via SQS).

## Well-Architected pillars touched

- [x] Operational excellence
- [x] Security
- [x] Reliability
- [ ] Performance efficiency
- [x] Cost optimization
- [x] Sustainability

## Revisit when

Sustained traffic would make an always-on container cheaper than per-request pricing, or a feature needs long-running compute beyond Lambda's limits.

## Interview summary (30 seconds)

> Traffic is low and spiky and the budget is tight, so I wanted zero idle cost and no servers to patch. The AI call costs the same anywhere, so the choice came down to operations and control. Serverless let me cap spend with SQS concurrency limits and API throttling instead of writing that logic myself. The trade-off is more services to wire together, which I manage with CDK.
