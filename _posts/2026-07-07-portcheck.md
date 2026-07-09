---
layout: post
title: PortCheck
subtitle: Building a CBP-Shaped Cargo Screening System in Java, Because the Job Posting Was
cover-img: /assets/img/portcheck-cover.jpg
thumbnail-img: /assets/img/portcheck-thumb.png
share-img: /assets/img/portcheck-share.jpg
tags:
  [
    java,
    spring-boot,
    angular,
    postgresql,
    soap,
    rest,
    jwt,
    kubernetes,
    terraform,
    aws,
    ecs-fargate,
    ci-cd,
    github-actions,
    enterprise-java,
    interview-project,
  ]
author: Mark Truong
---

## So, Why Am I Doing This?

The target role sits on a CBP program, and the stack listed on the posting reads like a decade-old enterprise Java shop's greatest hits: Spring Boot, SOAP interop with a legacy system, Angular, raw SQL for reporting, Kubernetes, "Java 8 and higher" written like a warning label. Nothing about that stack is cutting-edge, and that's the point — it's exactly what actually runs at a federal agency, and a hiring panel there isn't going to be impressed by a Rust microservice.

So I built PortCheck: a simplified port-of-entry cargo screening workflow. Importers file entry manifests, a risk engine flags the suspicious ones, screening officers work a review queue, supervisors have override authority on the flags officers can't clear themselves. No real CBP data, no real CBP systems — just the shape of the problem, built with the exact stack the job wants, deep enough that someone who's actually worked entry processing wouldn't wince at the domain model.

## The Stack

- **Backend:** Java 17, Spring Boot 3.x — Spring Data JPA, Spring Security + JWT, Spring WS for SOAP
- **Database:** PostgreSQL 16, Flyway migrations, hand-written SQL (`JdbcTemplate`) for the reporting endpoint
- **Frontend:** Angular 18, standalone components, Angular Material, Chart.js
- **Containers:** Docker multi-stage builds, docker-compose for local dev
- **Orchestration:** Kubernetes manifests, verified end-to-end on minikube
- **Cloud (stretch):** Terraform — ECS Fargate + RDS Postgres behind an ALB
- **CI/CD:** GitHub Actions — `mvn verify` with a JaCoCo coverage gate, Angular lint/test, Docker build

## How It Actually Works

```
Screening Officer / Supervisor (browser)          Customs Broker (legacy SOAP client)
              │                                              │
           HTTPS                                    SOAP/XML SubmitManifestRequest
              │                                              │
              ▼                                              ▼
        ┌─────────────────────────────────────────────────────────┐
        │  nginx / Ingress / ALB  (same routing shape everywhere)  │
        │     /api, /ws  ──────────►  Spring Boot backend          │
        │     /          ──────────►  Angular SPA (static build)   │
        └─────────────────────────────────────────────────────────┘
                                      │
                        REST + SOAP → same ManifestService
                                      │
                                    JDBC
                                      │
                                      ▼
                        PostgreSQL 16 (Flyway-migrated)
```

The one deliberate architectural bet: **the router changes, the shape never does.** In docker-compose, `/api` and `/ws` are proxied by the frontend container's own nginx. In Kubernetes, the Ingress controller intercepts those same paths before they ever reach the frontend Service. On AWS, the ALB does it with listener rules. Same split, three different routers, and the Angular build itself never changes — `environment.prod.ts` ships with a same-origin `apiBaseUrl: ''` and doesn't care which of the three is actually terminating the connection.

A shipment's life: an importer (or a broker filing over legacy SOAP) submits a manifest, a rule pipeline scores it and attaches `RiskFlag`s, and it lands in an officer's queue ordered by severity and age. The officer clears it, holds it, or requests more information — every decision writes an immutable `ReviewAction`. An officer can clear a LOW or MEDIUM flag on their own; overriding a HIGH-severity flag requires a supervisor role, enforced with `@PreAuthorize` at the service layer so it holds no matter whether the request came in over REST or SOAP.

![Needs Review queue: a flagged manifest with two unresolved high-severity risk flags](/assets/img/portcheck-review-queue.png)

## The Interesting Bits

- **REST and SOAP are two thin adapters over one service.** `ManifestController` and the SOAP `ManifestFilingEndpoint` both just call into `ManifestService` — a manifest filed by a legacy broker over SOAP runs through identical validation and risk screening as one filed through the Angular app. Contract-first, too: the XSD is the source of truth, and the WSDL and JAXB classes are generated from it, not hand-maintained alongside it.
- **The screening engine is a rule chain, not a conditional.** Each `ScreeningRule` — value deviation from a category baseline, country-of-origin watchlist hits, tariff-code mismatches, an auto-escalation rule for importers with repeat unresolved flags — is its own class, discovered via Spring DI and run as a pipeline. Adding a fifth rule means adding a fifth class; nothing existing gets touched or re-tested.
- **Raw SQL for the one query that actually needs it.** Every other repository is plain Spring Data JPA. The throughput report is hand-written `JdbcTemplate` SQL with three CTEs, because joining `review_actions` and `risk_flags` directly onto `entry_manifests` at the same level fans out — two review actions × three flags becomes six joined rows per manifest, silently inflating every `COUNT` and `AVG` upstream. Each CTE pre-aggregates to one row per manifest first, so the final join is a clean 1:1:1. The report also carries a window function per Project.md's spec — each row's port-level average time-to-decision (`AVG(...) OVER (PARTITION BY port_code)`) sitting alongside that week's own number, which is a window function wrapping an aggregate — legal in Postgres because windows evaluate after `GROUP BY`, but the kind of thing you only get right once you've been bitten by it before.
- **No NAT Gateway on the AWS deploy, on purpose.** Fargate tasks sit in the default VPC's public subnets with public IPs instead of behind a NAT Gateway — a real production setup wants that gateway, but it's also the single most common surprise line item on an AWS bill, and this is explicitly a tear-down-able demo, not a production posture. The tradeoff is written down, not silently taken.
- **CI found a bug that months of local Docker never could.** This machine's Docker Desktop has never been able to reach Testcontainers, so every Testcontainers-backed integration test ran for the very first time ever the moment this repo hit GitHub Actions — and it immediately surfaced a real test-isolation bug: a repository test assumed an empty table, but the shared Testcontainers Postgres runs Flyway's seed migration once per JVM, and that seed data (a FLAGGED manifest with the same entry number as the test's own fixture) leaked straight into the assertions. Fixed by clearing the table in `@BeforeEach`, safe because `@DataJpaTest` rolls back the whole transaction afterward. The same first CI run also caught GitHub's `ubuntu-latest` runners disabling the unprivileged user namespaces Chrome's sandbox needs — the Angular test job just crashed with "No usable sandbox!" until Karma got a CI-specific `--no-sandbox` launcher. Two bugs, invisible in any amount of local or dockerized testing, both surfaced by the exact moment the pipeline hit real infrastructure — which is a large part of why the pipeline exists at all.
- **AWS is real infrastructure, not a fake sandbox — and the README says exactly what that costs.** RDS is Free Tier eligible; ECS Fargate and the ALB are not, at all. Deploy, click through the login → file → flag → review → clear path as `jsmith`/`mgarcia`, and `terraform destroy` costs a few cents. Leaving it running costs roughly $30–45/month, almost entirely the ALB and two Fargate tasks. That number is in the README, not left for someone to discover on a bill.

## What I'd Do Differently With More Time

- Track more than one review action's worth of audit history end-to-end — REQUEST_INFO currently doesn't loop back into a second decision cleanly, and a real case-review workflow would.
- A proper market-calendar-style config for the watchlist and baseline tables instead of the current seed values, so the screening engine's thresholds are demonstrably tunable without a redeploy.
- Wire up a second environment (real staging RDS instead of the Testcontainers-only path) so the Flyway-seed-leak class of bug gets caught before a real CI push, not by one.
- OpenAPI/Swagger UI for the REST side to sit next to the WSDL, so both interop surfaces are self-describing.

None of these are things I didn't know how to do. They're things that weren't worth doing before the core claim — one service layer, two protocols, a screening pipeline that actually screens — was solid.

## Where I'm At Now

Seven phases, each landing as its own clean commit: foundation and schema, REST filing plus the screening engine, JWT auth and the review workflow, the SOAP interface, the Angular console, reporting plus Kubernetes, then the Terraform AWS deploy. Docs and a release workflow came right after. Then — fittingly, given the CBP setting — the first real push to GitHub Actions immediately failed twice, on a leaked seed-data assertion and a sandboxed Chrome headless launcher, both fixed within minutes because the failures were specific and reproducible instead of vague. That's the whole pitch of building the boring stack correctly: when something breaks, you know exactly why, and fixing it doesn't touch anything else.
