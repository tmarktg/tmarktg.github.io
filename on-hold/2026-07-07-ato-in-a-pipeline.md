---
layout: post
title: ato-in-a-pipeline
subtitle: A DoD-Style DevSecOps Factory Where Every Gate Actually Blocks
cover-img: /assets/img/ato-in-a-pipeline-cover.jpg
thumbnail-img: /assets/img/ato-in-a-pipeline-thumb.png
share-img: /assets/img/ato-in-a-pipeline-share.jpg
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
    oscal,
    ansible,
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

So the brief I gave myself reads like something out of an RFP: build a CI/CD pipeline where the security gates are hard-blocking, not advisory, ship signed images with attached SBOMs, enforce policy at the Kubernetes admission layer, map every control back to a NIST 800-53 ID with real evidence — no aspirational rows — and then push past a static matrix into the machine-readable compliance-as-code and configuration-management story that a real cATO program actually runs on. The app riding through the pipeline is almost beside the point. The pipeline is the product; the app just exists to be shipped by it.

Boring name on purpose. No memes, no cutesy README. This had to read as program-ready to someone who reviews ATO packages for a living.

## The Stack

- **App:** FastAPI ("Readiness Board" — tracks deployable service statuses), 100% test coverage, non-root/read-only-fs container by design
- **CI:** GitHub Actions and GitLab CI, running the identical twelve-stage pipeline
- **Security gates:** Semgrep (SAST), Gitleaks (secrets), Trivy (CVE scanning), `ansible-lint`
- **Supply chain:** Syft (SBOM, SPDX + CycloneDX), Cosign (image signing)
- **Base image:** Red Hat UBI9, hardened against the DISA STIG profile via OpenSCAP
- **Infra:** Terraform against LocalStack (VPC, S3, DynamoDB)
- **Deploy:** Kubernetes (kind in CI) via Kustomize, admission-gated by 5 Kyverno `ClusterPolicy` resources
- **Monitoring:** scheduled `terraform plan -detailed-exitcode` drift detection
- **Compliance:** a NIST 800-53 rev5 matrix and an OSCAL `component-definition.json`, both generated from one YAML source of truth
- **Configuration management:** Ansible — idempotent STIG hardening against a live container, plus demo-toolchain provisioning

## How It Actually Works

The architecture is one straight line from commit to cluster, with a compliance matrix sitting underneath the whole thing:

![Architecture diagram: commit flows through GitLab CI security gates (SAST + ansible-lint, Secrets, CVE Scan, SBOM + Sign) to a registry, then through Kyverno admission control into Kubernetes; Terraform/LocalStack and a drift-detection agent feed continuous monitoring, Ansible provides idempotent STIG hardening and demo-env provisioning, and every stage maps into the NIST 800-53 compliance matrix.](/assets/img/ato-in-a-pipeline-architecture.svg)

Every push runs the same twelve stages in both GitHub Actions and GitLab CI. A commit that fails Semgrep, trips Gitleaks, or gets flagged CRITICAL by Trivy simply doesn't produce an image — there's no "warn and continue." The compliance stage regenerates the matrix and the OSCAL JSON from `compliance/controls.yaml` and fails the build if either drifts from what's committed, so the docs can't quietly go stale relative to the code. One that passes gets an SBOM attached and its digest signed with Cosign before it's allowed to touch a registry. From there, Kyverno stands at the cluster door: no signature verification against the committed public key, no admission. No `runAsNonRoot`, no admission. Wrong registry, `:latest` tag, missing resource limits — same story, five separate policies, five separate ways to get denied.

A scheduled job runs the drift check daily against the repo's own LocalStack infra, and a separate Ansible role can re-harden (and re-verify) a running container against the same STIG baseline the Dockerfile bakes in at build time. Every one of these controls is traced back to a NIST 800-53 control ID, with a link to the file or evidence artifact that actually backs it.

## The Interesting Bits

A few decisions I ended up defending to myself out loud while building this:

- **The gates had to actually block, with proof.** It's easy to claim a pipeline is "secure." So I deliberately committed a vulnerable dependency (`pyyaml==5.3.1`, CVE-2020-14343, CRITICAL) on a branch never merged to main, and separately a fake AWS credential, and captured both pipeline failures verbatim into `docs/evidence/`. Combined with the Kyverno admission-denial logs from Phase 4, a bad commit is now demonstrably blocked at three independent layers — CI scan, secrets gate, and cluster admission — exactly the "core interview artifact" the spec called for.
- **Iron Bank first, honestly documented when it didn't work.** I actually tried pulling from `registry1.dso.mil` before falling back to UBI9 — Platform One's SSO has no anonymous pull path, so the ADR just says that plainly and moves to the documented fallback, rather than pretending Iron Bank was never on the table.
- **STIG remediation, with the parts I couldn't fix written down.** OpenSCAP against the RHEL9 STIG profile went from 61 pass / 7 fail to 65 pass / 3 fail after four Dockerfile changes (umask policy, shell-init permissions, a tmpfiles override). The remaining three failures are structurally impossible for a container — FIPS mode needs a kernel boot flag no container can set, one's an artifact of the scanner's own install, and `/etc/resolv.conf` gets overwritten by Docker at runtime regardless of image contents. I'd rather explain why three checks can't pass than quietly ignore them.
- **Build-time hardening isn't the same claim as running-system hardening — so Phase 8 says so twice.** A Dockerfile `RUN` proves a setting was correct the moment the image was built; it says nothing about a container that's been running for months. `ansible/roles/stig_hardening` re-expresses the same four STIG fixes as idempotent tasks against a *live* container via the `community.docker.docker` connection plugin — no SSH, no VM, docker exec under the hood. The target is deliberately Phase 2's pre-hardening base image, not this project's own already-hardened one, because pointing Ansible at an already-compliant container would report `changed=0` on the first run and prove nothing about idempotency. First run changes 6 tasks; the immediate rerun reports `changed=0`; the filesystem state gets checked directly, independent of Ansible's own report. Three separate pieces of evidence for one claim, because "idempotent" is a claim worth actually demonstrating, not asserting.
- **The OSCAL file is generated, not hand-authored — and had to be made boring on purpose.** Phase 6 explicitly declined to hand-write a second, JSON-shaped copy of the compliance matrix, since an OSCAL file nobody validates is exactly the kind of aspirational artifact this project is trying to avoid. Phase 7 makes it real: both the matrix and `component-definition.json` generate from one `controls.yaml`, and CI fails on any drift. That only works if regenerating from an unchanged source produces byte-identical output every time, which meant hunting down two non-deterministic fields — UUIDs (`uuid.uuid5` off a stable string instead of `uuid.uuid4`, so the same control always gets the same UUID) and `metadata.last-modified` (the source file's own git commit timestamp instead of `datetime.now()`, which would fail the drift check on literally every run). Validation runs by parsing the generated file back through `compliance-trestle`'s Pydantic models rather than reaching for NIST's Java `oscal-cli` or `trestle validate` — both real options I tried first and rejected for reasons written into ADR 0006, not just picked because they were harder.
- **`component-definition`, not a system-security-plan.** OSCAL has a model for "here's the full authorization package" (SSP) and a narrower one for "here's what this component implements" (component-definition). This repo's own non-goals say plainly it isn't a real ATO, so generating an SSP would misrepresent what the project claims. A component-definition makes the same true, narrower claim the matrix already makes — machine-readable this time.
- **Unfixed CVEs are tracked, not faked away.** Four CRITICAL CVEs in the base image have no upstream patch at all — `will_not_fix` or `fix_deferred` upstream, not "we didn't get to it." Trivy runs `--ignore-unfixed` on the blocking gate for exactly that reason: a gate that fails on things no one can fix isn't a gate, it's noise people learn to ignore.
- **No Terraform-provisioned container registry, on purpose.** ECR is a LocalStack Pro (paid) feature. Provisioning it would mean either paying for infra on a project whose entire premise is zero cloud spend, or faking a registry that isn't real. Instead the real registry (GHCR/GitLab) is already wired up from Phase 1, and the ADR documents exactly how the LocalStack state backend maps onto a real S3 + DynamoDB backend in actual AWS.
- **The drift-detection agent is linked, not vendored — and Molecule got the same treatment.** I already have a separate LangGraph ReAct agent (Claude + pgvector RAG + Postgres checkpointer) that does detect → classify → remediate for Terraform drift; wiring it into this repo's scheduled CI job would mean real API cost and a Postgres+pgvector dependency for a job that just needs `terraform plan -detailed-exitcode`, so it stays linked. Same instinct killed Molecule for the Ansible role — it's the standard tool for apply-twice-and-check-idempotency, but adding its own scenario directory and driver config to test a two-role playbook that already has three independent pieces of idempotency evidence felt like testing infrastructure to test the thing that's already just Docker. Both decisions, and the reasoning for not vendoring/adding them, are written down rather than left implicit.
- **The compliance matrix has no aspirational rows — and neither does Ansible's mapping.** Every one of the 15 mapped NIST 800-53 controls links to a real file, pipeline line, or evidence artifact, and the matrix explicitly lists what it does *not* cover (account management, backups, incident response) because this repo genuinely doesn't implement them. The Ansible `demo_env` role — real engineering value, since `make demo` used to silently assume `kind`/`kubectl`/`kustomize`/`helm` were already installed — isn't mapped to any control at all, because it provisions developer tooling, not a security setting, and mapping it to CM-2 just to inflate the control count would be exactly the padding the matrix rules out elsewhere.

## What I'd Do Differently With More Time

- Deepen the CI git checkout past `fetch-depth: 1` so the OSCAL `last-modified` timestamp resolves to the actual last commit that touched `controls.yaml`, not just the tip commit — GitLab's `GIT_DEPTH: 50` already gets this right; GitHub Actions doesn't yet.
- Wire the LangGraph drift agent in as an optional, explicitly-gated CI job instead of purely a linked writeup, so the detect → classify → remediate story is demoable, not just described.
- Bring in Molecule once the Ansible surface grows past two roles — the deferral was scope-appropriate at two roles, not a permanent stance.
- Expand past 15 controls into the families I deliberately left out, particularly incident response — right now the matrix is honest about the gap, but honest isn't the same as complete.

None of these are things I didn't know how to do. They're things that weren't worth doing before the core claim — gates that actually block, mapped to real controls, regenerated and re-verified rather than asserted once — was airtight.

## Where I'm At Now

The commit history tells its own story here: Phase 0 and Phase 1 landed together, then a quick fix for a bad Trivy action pin, a cosign key rotation, and then Phases 2 through 6 each as their own clean commit — hardened base image, Terraform/LocalStack, Kyverno enforcement, drift detection, compliance matrix. A gap-closing commit filled in missing secrets-gate evidence, then the Mermaid diagram got swapped for a proper SVG because it needed to look right, not just render. Then two more full phases on top of what I'd originally scoped as done: OSCAL compliance-as-code, and Ansible configuration management — plus two small follow-up commits making sure the architecture diagram's own alt-text didn't go stale the moment Ansible joined the picture.

Phase by phase, acceptance criteria before moving on, no scope creep into the app itself — it stayed frozen after Phase 0 exactly like the spec demanded. That's not the most exciting way to build something, but it's the way that produces a repo where every claim in the README has a file behind it, and where finishing the original plan didn't mean the project was done — it meant it was time to ask what a real program would build next. For the audience I built this for, that's the entire pitch.
