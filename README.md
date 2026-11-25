# [Brand] — AI Yard Design for Elk Grove Homeowners

Upload a photo of your yard, pick a style and your must-haves (drought-tolerant, low maintenance, pet-friendly), and get an AI design concept plus a ballpark cost range. If you like it, request quotes from licensed (CSLB C-27) local landscapers you choose.

> **Status:** 🚧 In development — Milestone A (public demo) targeted for January 2027.
> **Live demo:** _coming soon_

This is also a hands-on AWS architecture project. Every design choice is recorded in a decision log (see [`docs/adr/`](docs/adr/)), and the whole stack is defined as infrastructure-as-code with AWS CDK.

---

## What it does

1. **Upload:** the homeowner verifies their email and uploads one yard photo (JPEG/PNG, ≤ 10 MB) directly to S3.
2. **Analyze:** Claude Haiku 4.5 on Amazon Bedrock confirms it's a yard and lists what's in it.
3. **Redesign:** Stability AI *Control Structure* on Bedrock restyles the photo while keeping the yard's layout.
4. **Estimate:** deterministic code (not the LLM) turns the homeowner's square footage and a local rate table into a low–high cost range.
5. **Request quotes:** the homeowner picks up to 3 licensed landscapers and consents; each gets the lead by email.

## Architecture

![Architecture diagram](docs/architecture.png)
<!-- TODO: export the diagram to docs/architecture.png -->

| Layer | Services |
| --- | --- |
| Edge | Amazon CloudFront, AWS WAF, Amazon S3 (static site, private via OAC), Route 53, ACM |
| API | Amazon API Gateway (HTTP API), AWS Lambda (Python) |
| Async AI pipeline | Amazon SQS (+ DLQ), Lambda worker, Amazon Bedrock, Amazon Rekognition |
| Data | Amazon DynamoDB (on-demand, TTL), Amazon S3 (media bucket with lifecycle rules) |
| Email | Amazon SES |
| Guardrails | AWS Budgets (+ budget action), Cost Anomaly Detection, CloudWatch alarms, SSM Parameter Store |
| IaC | AWS CDK v2 (Python) |

Region: **us-west-2 (Oregon)**.

## Cost and abuse controls

Image generation is the only meaningful per-request cost (~$0.08 per design), so it sits behind several layers:

- WAF rate limiting and CAPTCHA at the edge
- API Gateway route throttling
- Verified email required before upload
- Per-email daily quota and a global daily cap (DynamoDB conditional writes)
- SQS event-source maximum concurrency of 2 workers
- AWS Budgets alerts, plus a budget action that blocks Bedrock calls (kill switch)

Measured monthly cost: _TBD after the demo goes live._

## Privacy

- EXIF/GPS metadata is stripped from every upload.
- Original uploads are deleted after 30 days and generated designs after 90 days (S3 lifecycle rules).
- Leads are shared only with the landscapers the homeowner selects.
- Uploads are never used to train models.

## Repository layout

```
.
├── infra/            # AWS CDK app (Python) — stacks for site, API, pipeline, guardrails
├── services/         # Lambda function code (Python)
├── web/              # Static front end
├── docs/
│   ├── adr/          # Architecture decision records
│   └── architecture.png
└── README.md
```

## Getting started (developer)

Prerequisites: Python 3.12, Node.js LTS, AWS CLI v2, AWS CDK v2, Docker Desktop, and AWS credentials for an account with Bedrock access in us-west-2.

```bash
cd infra
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cdk bootstrap aws://<ACCOUNT_ID>/us-west-2   # first time only
cdk diff
cdk deploy --all
```

## Decision log

See [`docs/adr/`](docs/adr/). New decisions start from [`0000-template.md`](docs/adr/0000-template.md).

## Disclaimer

[Brand] is a referral service, not a contractor. Designs are AI-generated concepts and cost ranges are rough planning estimates, not bids. Homeowners contract with and pay licensed contractors directly.

## Author

Liqiang (Grant) Guan · [LinkedIn](https://linkedin.com/in/guangrant)
