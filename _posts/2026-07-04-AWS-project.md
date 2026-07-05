---
layout: post
title: Six Stocks, Zero Servers
subtitle: What a Project Taught Me About Serverless
cover-img: /assets/img/aws-project-cover.jpg
thumbnail-img: /assets/img/aws-project-thumb.png
share-img: /assets/img/aws-project-share.jpg
tags:
  [
    aws,
    serverless,
    lambda,
    dynamodb,
    terraform,
    eventbridge,
    api-gateway,
    s3,
    python,
    infrastructure-as-code,
    ci-cd,
    github-actions,
    cloud-architecture,
    interview-project,
  ]
author: Mark Truong
---

## So, Why Am I Doing This?

The build was straightforward on paper: build something that pulls stock data and shows it off, using serverless AWS. No hand-holding on architecture, no starter repo — just a prompt and a deadline. The kind of assignment that's really testing two things at once: can you design cloud infrastructure that doesn't fall over, and can you make decisions about the boring-but-important stuff (cost, security, idempotency) without someone telling you to.

So that's what I optimized for. Not "impress them with complexity," but "ship something a senior engineer would actually sign off on."

## The Stack

- **Backend:** Python Lambdas — no framework, just `boto3` and `urllib3`
- **Data:** DynamoDB
- **Infra:** Terraform, split into modules (`api_gateway`, `dynamodb`, `eventbridge`, `lambda`, `s3_frontend`)
- **Frontend:** plain HTML/CSS/vanilla JS on an S3 static site, with config injected by Terraform at deploy time
- **External API:** Massive.com for OHLC stock data
- **CI/CD:** GitHub Actions

## How It Actually Works

The architecture is deliberately boring, in the best way:

```
EventBridge (daily cron, ~5PM ET)
        │
        ▼
Lambda: ingestion  ──► Secrets Manager (Massive.com API key)
        │             ──► Massive API (stock OHLC data)
        ▼
   DynamoDB (stock-movers table)
        │
        ▼
Lambda: retrieval  ◄── API Gateway (GET /movers)  ◄── S3 static site (frontend)
```

Every trading day, EventBridge fires the ingestion Lambda, which pulls the previous day's open/close bar for AAPL, MSFT, GOOGL, AMZN, TSLA, and NVDA from Massive.com. For each ticker it computes `((close - open) / open) * 100`, then writes only the single biggest mover — by absolute percent change — to DynamoDB. No servers running 24/7 waiting for a cron to hit; nothing is provisioned until the moment it's needed.

On the read side, a retrieval Lambda sits behind API Gateway and serves the last 7 days of movers via `GET /movers`, which the S3-hosted frontend renders into a table.

## The Interesting Bits

A few decisions I ended up defending to myself out loud while building this:

- **Idempotent writes for free.** The DynamoDB partition key is `date`. Re-run ingestion twice in one day and it just overwrites — no dedup logic, no "have I already run today" check needed.
- **Weekends and holidays handle themselves.** Instead of building a market-calendar lookup, I lean on Massive's `/prev` (previous-bar) endpoint. On a day with no trading, it just returns no data, and the Lambda skips the write. Zero calendar logic, zero maintenance when NYSE adds a new holiday.
- **Rate limits are handled, not hoped away.** The ingestion Lambda retries on HTTP 429 with exponential backoff, up to 3 attempts per ticker.
- **Remote Terraform state, encrypted, locked down.** State lives in S3 — not on my laptop — encrypted at rest with AES-256, with all four public-access blocks enabled. Overkill for a solo project? Maybe. But it's the "pretend a teammate is going to `terraform apply` this next week" habit, and habits like that are exactly what a engineer is checking for.
- **Free Tier, mostly.** Everything fits comfortably in AWS Free Tier except Secrets Manager, which runs about $0.40/month after the trial period. The README calls this out explicitly, along with the SSM Parameter Store swap that would make the whole thing fully free.
- **Scan instead of Query — on purpose.** The retrieval Lambda does a `Scan` with a date filter instead of a `Query` against an index. That's usually a code smell, but with at most 365 items a year, a full scan costs nothing meaningful. I noted in the README that a GSI would be the right call at real scale — I'd rather show I know the shortcut I'm taking and why it's fine here than pretend I didn't take one.
- **CI actually gates infra changes.** GitHub Actions runs `terraform plan` on every PR and auto-applies on merge to main. Infrastructure-as-code, not click-ops with a Terraform label slapped on it.

## What I'd Do Differently With More Time

The README already has an honest list of trade-offs, because I'd rather write them down than have someone find them for me:

- Swap Secrets Manager for SSM Parameter Store to get to true $0/month.
- Add a GSI on the DynamoDB table once the dataset outgrows "scan is fine."
- Track more than the single top mover per day — right now, second place gets thrown away entirely, which is a little wasteful if anyone ever wants historical context beyond "who won."
- More tickers, more sectors — six was enough to prove the pattern, not enough to be genuinely useful for anyone tracking a real portfolio.

None of these are things I didn't know how to do. They're things that weren't worth doing yet, which is its own kind of engineering judgment.

## Where I'm At Now

Looking back at the commit history, this project reads nothing like my usual rabbit holes. It's five or six small, incremental commits — initial implementation, fix the gitignore, add the remote backend, fix the README wording, fix the architecture diagram arrows. Built it, then hardened it. No chaos, no 2 AM scope creep. Just a scoped problem, solved, then polished until I was comfortable putting my name on it.

It didn't get me a "look how clever I am" story. It got me a small, real, currently-running piece of infrastructure that costs less than a coffee a month and hasn't needed a single manual intervention since I deployed it. Sometimes that's the whole point of the exercise.
