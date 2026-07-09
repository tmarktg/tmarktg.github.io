---
layout: post
title: The Pipeline Is the Product
subtitle: Building a DoD-Style DevSecOps Factory Where Every Gate Actually Blocks
cover-img: /assets/img/ato-pipeline-cover.jpg
thumbnail-img: /assets/img/ato-pipeline-thumb.png
share-img: /assets/img/ato-pipeline-share.jpg
tags:
  [
    devsecops,
    kubernetes,
    kyverno,
    terraform,
    ci-cd,
    github-actions,
    gitlab-ci,
    cosign,
    sbom,
    supply-chain-security,
    nist-800-53,
    compliance-as-code,
    fastapi,
    python,
    dod,
    interview-project,
  ]
author: Mark Truong
---

## So, Why Am I Doing This?

I hold an active Secret clearance and I'm chasing work with defense contractors — Booz Allen, Leidos, GDIT, General Atomics, the usual names. Those shops don't hire on "I built a CRUD app." They hire on Platform One, Iron Bank, NIST RMF, and whether you understand that a security control that doesn't block anything isn't a control, it's a suggestion.

So I gave myself a brief that reads like something out of an RFP: build a CI/CD pipeline where the security gates are hard-blocking, not advisory, ship signed images with attached SBOMs, enforce policy at the Kubernetes admission layer, and map every single control back to a NIST 800-53 ID with real evidence — no aspirational rows. The app riding through the pipeline is almost beside the point. The pipeline is the product; the app just exists to be shipped by it.

Boring name on purpose. No memes, no cutesy README. This had to read as program-ready to someone who reviews ATO packages for a living.

## The Stack

- **App:** FastAPI ("Readiness Board" — tracks deployable service statuses), 100% test coverage, non-root/read-only-fs container by design
- **CI:** GitHub Actions and GitLab CI, running the identical ten-stage pipeline
- **Security gates:** Semgrep (SAST), Gitleaks (secrets), Trivy (CVE scanning)
- **Supply chain:** Syft (SBOM, SPDX + CycloneDX), Cosign (image signing)
- **Base image:** Red Hat UBI9, hardened against the DISA STIG profile via OpenSCAP
- **Infra:** Terraform against LocalStack (VPC, S3, DynamoDB)
- **Deploy:** Kubernetes (kind in CI) via Kustomize, admission-gated by 5 Kyverno `ClusterPolicy` resources
- **Monitoring:** scheduled `terraform plan -detailed-exitcode` drift detection
- **Compliance:** a hand-built NIST 800-53 rev5 matrix, every row backed by a real file or artifact

## How It Actually Works

The architecture is one straight line from commit to cluster, with a compliance matrix sitting underneath the whole thing:

```
commit
   │
   ▼
CI: lint → test → sast → secrets → terraform → build → scan → sbom → sign → publish
   │         (Semgrep)  (Gitleaks)   (plan)    (image)  (Trivy) (Syft) (Cosign)
   ▼
container registry (GHCR / GitLab)
   │
   ▼
Kyverno admission control (5 ClusterPolicies)
   │
   ▼
Kubernetes (kind) ── Terraform/LocalStack infra ── scheduled drift-check
   │
   ▼
NIST 800-53 compliance matrix (every stage above maps to a control + evidence link)
```

Every push runs the same ten stages in both GitHub Actions and GitLab CI. A commit that fails Semgrep, trips Gitleaks, or gets flagged CRITICAL by Trivy simply doesn't produce an image — there's no "warn and continue." One that passes gets an SBOM attached and its digest signed with Cosign before it's allowed to touch a registry. From there, Kyverno stands at the cluster door: no signature verification against the committed public key, no admission. No `runAsNonRoot`, no admission. Wrong registry, `:latest` tag, missing resource limits — same story, five separate policies, five separate ways to get denied.

A scheduled job runs the drift check daily against the repo's own LocalStack infra, and every one of these controls is traced back to a NIST 800-53 control ID in the compliance matrix, with a link to the file or evidence artifact that actually backs it.

## The Interesting Bits

A few decisions I ended up defending to myself out loud while building this:

- **The gates had to actually block, with proof.** It's easy to claim a pipeline is "secure." So I deliberately committed a vulnerable dependency (`pyyaml==5.3.1`, CVE-2020-14343, CRITICAL) on a branch never merged to main, and separately a fake AWS credential, and captured both pipeline failures verbatim into `docs/evidence/`. Combined with the Kyverno admission-denial logs from Phase 4, a bad commit is now demonstrably blocked at three independent layers — CI scan, secrets gate, and cluster admission — which is exactly the "core interview artifact" the spec called for.
- **Iron Bank first, honestly documented when it didn't work.** I actually tried pulling from `registry1.dso.mil` before falling back to UBI9 — Platform One's SSO has no anonymous pull path, so the ADR just says that plainly and moves to the documented fallback, rather than pretending Iron Bank was never on the table.
- **STIG remediation, with the parts I couldn't fix written down.** OpenSCAP against the RHEL9 STIG profile went from 61 pass / 7 fail to 65 pass / 3 fail after four Dockerfile changes (umask policy, shell-init permissions, a tmpfiles override). The remaining three failures are structurally impossible for a container — FIPS mode needs a kernel boot flag no container can set, one's an artifact of the scanner's own install, and `/etc/resolv.conf` gets overwritten by Docker at runtime regardless of image contents. I'd rather explain why three checks can't pass than quietly ignore them.
- **Unfixed CVEs are tracked, not faked away.** Four CRITICAL CVEs in the base image have no upstream patch at all — `will_not_fix` or `fix_deferred` upstream, not "we didn't get to it." Trivy runs `--ignore-unfixed` on the blocking gate for exactly that reason, and the ADR spells out the distinction: a gate that fails on things no one can fix isn't a gate, it's noise people learn to ignore.
- **No Terraform-provisioned container registry, on purpose.** ECR is a LocalStack Pro (paid) feature. Provisioning it would mean either paying for infra on a project whose entire premise is zero cloud spend, or faking a registry that isn't real. Instead the real registry (GHCR/GitLab) is already wired up from Phase 1, and the ADR documents exactly how the LocalStack state backend maps onto a real S3 + DynamoDB backend in actual AWS.
- **The drift-detection agent is linked, not vendored.** I already have a separate LangGraph ReAct agent (Claude + pgvector RAG + Postgres checkpointer) that does detect → classify → remediate for Terraform drift. Wiring it into this repo's scheduled CI job would mean real API cost and a Postgres+pgvector dependency for a job that just needs `terraform plan -detailed-exitcode`. So the scheduled job stays free and boring, and the deeper story is one link away, with the reasoning for not vendoring it written into `docs/continuous-monitoring.md`.
- **The compliance matrix has no aspirational rows.** Every one of the 15 mapped NIST 800-53 controls links to a real file, pipeline line, or evidence artifact — and the matrix explicitly lists what it does *not* cover (account management, backups, incident response) because this repo genuinely doesn't implement them. A hiring manager who's reviewed real ATO packages will check the links; I'd rather have fewer rows that all hold up than a padded table.

## What I'd Do Differently With More Time

- Emit the compliance matrix as OSCAL component-definition JSON — it's scaffolded under `compliance/oscal/` as a stretch goal but never actually populated.
- Track a second, real Postgres-backed deployment target instead of only SQLite-in-dev — the app spec keeps that swap "trivial via env config," but I never actually exercised it end to end.
- Wire the LangGraph drift agent in as an optional, explicitly-gated CI job instead of purely a linked writeup, so the detect → classify → remediate story is demoable, not just described.
- Expand past 15 controls into the families I deliberately left out, particularly incident response — right now the matrix is honest about the gap, but honest isn't the same as complete.

None of these are things I didn't know how to do. They're things that weren't worth doing before the core claim — gates that actually block, mapped to real controls — was airtight.

## Where I'm At Now

The commit history tells its own story here: Phase 0 and Phase 1 landed together, then a quick fix for a bad Trivy action pin, a cosign key rotation, and then Phases 2 through 6 each as their own clean commit — hardened base image, Terraform/LocalStack, Kyverno enforcement, drift detection, compliance matrix. Then two closing passes: one to close a gap in the Definition of Done (the secrets-gate blocking evidence wasn't captured yet), and one to swap a Mermaid diagram for a proper SVG because it needed to look right, not just render.

Phase by phase, acceptance criteria before moving on, no scope creep into the app itself — it stayed frozen after Phase 0 exactly like the spec demanded. That's not the most exciting way to build something, but it's the way that produces a repo where every claim in the README has a file behind it. For the audience I built this for, that's the entire pitch.
