+++
title = 'AppSec KPIs'
date = 2026-05-15T10:00:00+02:00
draft = false
description = "A practical set of AppSec KPIs for organizations using GitHub Enterprise, with thresholds, escalation paths, and an honest look at what each metric actually measures."
summary = "Not every AppSec metric is worth tracking. This post covers four KPIs that give a genuine picture of your security posture — inventory completeness, scan coverage, SLA compliance, and secret exposure — along with thresholds and escalation paths you can act on."
slug = "appsec-kpis"
series = "appsec_github"
relatedPosts = ["github-enterprise-system-inventory"]
tags = ["AppSec", "Governance", "GHAS", "GitHub", "KPI"]
categories = ["appsec"]
+++

{{< katex >}}

{{< series "appsec_github" >}}

Good KPIs (Key Performance Indicators) answer a specific question, have a clear
owner, and trigger a defined response when they cross a threshold. They're also
surprisingly hard to find good guidance on for application security. Published
lists tend to stay at the *track your vulnerabilities* level without the
thresholds or escalation paths that make a metric useful in practice.

This post covers four KPIs that work as a foundation for organizations building
their AppSec program around a system inventory in GitHub. They might not be the
only KPIs worth tracking, and probably not enough on their own for a mature
program.  They are directly measurable from the GitHub data the inventory gives
you, and each has a defined action when it slips. A few other metrics worth
considering later are listed [at the end of this post](#other-kpis-to-consider).

The four are ordered by dependency: scan coverage doesn't make sense without a
system inventory to measure against, and SLA compliance only reflects reality
when scan coverage is high enough that the findings you're tracking are
representative. Get them in sequence. They depend on the `business-criticality`
and `data-classification` properties being filled in accurately.

{{< alert "lightbulb" " " >}}
Take one step at a time. Finish up a KPI before moving on to the next.
{{< /alert >}}

## At a Glance

| # | KPI | What it measures | Healthy target | First escalation |
|---|---|---|---|---|
| 1 | [Inventory Completeness](#kpi-1-inventory-completeness) | Repositories with all required Custom Properties set | ≥ 95% | Repo owner notified weekly until filled in |
| 2 | [Scan Coverage](#kpi-2-scan-coverage) | Repositories with the scans their tier requires | 100% per tier | Repo owner notified weekly; flagged to AppSec if unresolved after 14 days |
| 3 | [SLA Compliance](#kpi-3-sla-compliance) | Findings remediated within their SLA | ≥ 95% high-risk, ≥ 80% low-risk | Owner warned at 75% of SLA elapsed |
| 4 | [Secret Exposure Rate](#kpi-4-secret-exposure-rate) | Push-blocked vs. post-commit secret detections | Push-blocks dominate, trend down | Immediate response on every post-commit detection |

## KPI 1: Inventory Completeness

**What it measures**: The percentage of active, non-archived repositories that
have all required Custom Properties filled in with an explicit value.

**Why it matters**: During an incident, an incomplete inventory means you
don't know what you're protecting. During an audit, it's a gap with no
workaround. Every other KPI depends on the system inventory completeness. It is
necessary in order to know which systems that need more attention in regards to
security, so to prioritize correctly. 

**How to calculate it**:

$$\text{Completeness} = \frac{\text{repos with all required properties set}}{\text{total active repos}} \times 100$$

Required properties are the ones that are useful to your organization. For the
sake of an example in this post, I use the properties `business-criticality`,
`data-classification`, `internet-facing`.

**Escalation path per repository**:

The goal is to get missing properties filled in, not to punish teams. The
escalation path should be supportive, with automation doing the chasing rather
than a person sending manual reminders.

1. **Properties missing**: The repository owner receives an automated weekly
   notification listing which properties are unset and linking to where and how
   to fill them in.
2. **Still missing after 3 weeks**: The security responsible (AppSec, CISO, or 
   similar, depending on your organization). is notified and reaches out to the
   team directly — to help, not to reprimand. Some teams genuinely don't know
   what value to pick; a short conversation usually resolves it faster than more
   emails.

{{< alert "lightbulb" "Automate the chasing" >}}
This escalation path can be automated with a scheduled GitHub Actions workflow.
A weekly job queries the API for repos with missing properties, identifies the
owning team for each repository, and posts a notification to that team's
channel. A monthly job sends a report to the application security management How
to implement this is covered in [Building a System Inventory in GitHub
Enterprise]({{< ref "github-enterprise-system-inventory" >}}).
{{< /alert >}}

**Score thresholds for governance reporting**:

| Score | Interpretation |
|---|---|
| ≥ 95% | On track |
| 85–95% | Needs attention — review which teams are lagging and why |
| < 85% | Escalate — systemic problem, likely requires AppSec involvement |

The target is 100%. Anything below 95% is a governance problem, not a
technical one.

## KPI 2: Scan Coverage

**What it measures**: The percentage of active repositories that have the
correct scanning tools enabled, based on their criticality tier.

**Why it matters**: Scan coverage tells you whether you're actually looking for
vulnerabilities in the right places. A repository with both source code and a
Dockerfile should use both code scanning and IaC-scanning. Measuring only the
amount of findings in the source code provides a false sense of safety.

The requirements differ by risk tier. The examples here use the two-tier model
defined in the
[series intro]({{< ref "appsec-program-github-intro#two-risk-tiers" >}}):
high-risk repositories get the full GHAS set, low-risk ones get a lighter
baseline.

**Coverage requirements by tier**:

| Risk tier | Required scanning |
|---|---|
| High | SAST (CodeQL or equivalent), Infrastructure as Code (IaC) scanning, secret scanning, push protection, SCA |
| Low (baseline) | Secret scanning, SCA |

{{< alert "circle-info" "On SAST tooling" >}}
CodeQL covers most common languages well, but not all.
For repositories where CodeQL doesn't support the primary language, the
requirement is "an equivalent SAST tool integrated into the CI pipeline with
results pushed to GitHub Security." The spirit of the requirement is that
static analysis runs on every PR — the specific tool is secondary.
{{< /alert >}}

**How to calculate it**: Calculate separately per tier, since the requirements
differ. A combined percentage hides the more important number: coverage on your
highest-risk systems.

$$\text{Coverage (tier)} = \frac{\text{repos in tier with all required tools active}}{\text{total repos in tier}} \times 100$$

**Target and escalation**:

The target is 100% across both tiers — every active repository should have the
scans its tier requires. Missing scans aren't a per-incident emergency, but
they are a gap that needs closing on a predictable cadence, similar to missing
inventory properties.

1. **Scans missing**: The repository owner receives an automated weekly
   notification listing which required scans are not active, with links to how
   to enable them. Owners have 14 days to bring the repository into compliance.
2. **Still missing after 14 days**: Included in the monthly report to the
   application security responsible. High-risk repositories are listed first
   and flagged for follow-up that month; low-risk repositories are listed as
   standard backlog.

The tier doesn't change *whether* a gap is reported — it changes *how urgently*
it's followed up.

## KPI 3: SLA Compliance

**What it measures**: The percentage of open vulnerabilities that are being
remediated within the agreed SLA, and the percentage that have breached it.
Open vulnerabilities are findings from any security scanner, such as third-party
scanning, code scanning, and IaC-scanning.

**Why it matters**: Finding vulnerabilities is only half the job. SLA compliance
tells you whether findings are actually being resolved, and at what pace. For
auditors, it's the evidence that your security program has teeth. It's not just
alerts going into a queue nobody acts on.

**The SLA matrix**:

SLAs should reflect two dimensions: the severity of the vulnerability and the
risk tier of the affected system (as defined in the [series intro]({{< ref
"appsec-program-github-intro#two-risk-tiers" >}})). A Critical CVE in a payment
system is not the same urgency as a Critical CVE in a development tool.

| Severity | High risk | Low risk |
|---|:---:|:---:|
| **Critical** | 3 days | 10 days |
| **High** | 10 days | 30 days |
| **Medium** | 30 days | 90 days |
| **Low** | 90 days | Best effort |

These are starting points — adjust them to match your organization's risk
appetite and your teams' actual capacity. An SLA you can't consistently meet
is worse than no SLA: it normalizes breach as the expected state.

**Escalation path**:

A vulnerability approaching or breaching SLA should trigger automatic
notifications — not a dashboard that someone has to remember to check.

1. **Warning (75% of SLA elapsed)**: Notify the repository owner. Give them
   time to act before the breach.
2. **Breach (100% of SLA elapsed)**: Notify both the owner and the application
   security responsible for the organization. The owner has had their chance.
3. **Extended breach (+50% past SLA), Medium and Low severity**: Tracked by
   the application security responsible and included in the monthly report.
   Repeated patterns from the same team warrant a direct conversation, not
   another notification.
4. **Extended breach (+50% past SLA), Critical and High severity**: Escalate
   to the CISO. At this point it's a governance issue, not a team issue.

**Thresholds**:

Track compliance per tier separately, because the two tiers answer different
questions.

For **high-risk repositories**, SLA compliance is a direct risk indicator. A
breach means a known vulnerability is sitting unpatched on a system that
matters.

| High-risk SLA compliance | Interpretation |
|---|---|
| ≥ 95% | Healthy — monitor regularly |
| 85–95% | Warning — investigate which teams are lagging |
| < 85% | Escalate — systemic capacity or prioritization problem |

For **low-risk repositories**, SLA compliance is more of a hygiene and culture
signal. A single breach on an internal tool isn't a crisis, but a consistent
pattern tells you a team is missing either the competence, the understanding, or
the security mindset to handle findings as part of normal work. For a product
organisation, it's a sign that your engineering quality isn't where it should
be. For a consultancy, it says the same about the standard of the engineers
you're placing with clients. Basic hygiene and engineering quality should apply
everywhere, including most low-risk repos, so that good practice is the default
rather than the exception.

| Low-risk SLA compliance | Interpretation |
|---|---|
| ≥ 80% | Acceptable — track as trend |
| 70–80% | Worth a conversation with the team about capacity or SLA realism |
| < 70% | Either the team needs help, or the SLAs are set too aggressively |

An overall combined percentage hides both signals. Reporting them separately
keeps the high-risk number honest and the low-risk number useful.

## KPI 4: Secret Exposure Rate

**What it measures**: The number of secrets detected in the codebase, split by
*when* they were caught — blocked at push time versus discovered after the fact.

**Why it matters**: Leaked credentials in source code have caused public
breaches at companies large enough to make headlines. Uber, Toyota, Samsung,
and many others. They remain a recurring finding in penetration tests and
security audits. The two numbers in this KPI answer two different questions.

*Push-blocked* tells you whether the preventive controls you've paid for are
actually doing work. Every blocked push is a credential that would otherwise
have reached the repository and required rotation, exposure assessment, and
possibly an incident response. Each of which costs real engineering time.

*Post-commit discoveries* tell you how often prevention fails and the
organization is left cleaning up. A low and stable number means the
controls are working; a rising number means they aren't, regardless of what
the tooling budget looks like.

**How to track it**:

Both numbers come from the GitHub Security tab. Push-blocks are recorded by
GitHub Secret Protection; post-commit discoveries come from secret scanning.
Both signals only cover repositories with GHAS enabled for the gathered
overview.

There's no fixed target for the absolute numbers. Codebase size and
developer workflows affect volume. What matters is the ratio and the
trend:

- Push-blocks should make up the majority of detections. If post-commit
  discoveries consistently outnumber them, push protection isn't active
  everywhere it should be, go back to KPI 2.
- Total detections should trend down over time as developers and tooling
  move credentials into secret managers and environment variables.
- Any increase in post-commit discoveries warrants investigation. Is a team
  bypassing push protection? Is a new integration introducing credentials
  into code?

Every post-commit secret discovery should trigger an immediate response: rotate
the credential, assess the exposure window, and review whether it was accessed.
The full process with rotation, branch handling, edge cases, and the automation
that drives the workflow below, is covered in
[Handling Secret Exposures in GitHub Enterprise]({{< ref "handling-secret-exposures-github-enterprise" >}}).

{{< alert "lightbulb" "Scaling back when the numbers stay healthy" >}}
GHAS is a per-committer cost, and full Secret Protection coverage across every
repository may not always be justifiable. If the numbers stay healthy over a
long period and budget pressure is real, it can be reasonable to redefine which
repositories keep push protection and rely on post-commit scanning plus the
response process for the rest. The trade-off is effort: post-commit detections
always mean rotation work, where push-blocks mean none.
{{< /alert >}}

**Remediation tracking**:

A detected secret that nobody acts on is the same as no detection at all. The
process should make handling visible and overdue handling hard to ignore:

1. **On detection**: An automated workflow opens a tracking issue in the
   affected repository (or posts to the team's channel) listing what was
   exposed and the required steps: rotate, assess exposure, document
   resolution. The GitHub security alert remains the source of truth; the
   issue is for team visibility and audit trail.
2. **Weekly reminder** as long as the alert is open and the issue isn't
   closed with a resolution comment. Closing without a comment doesn't count
   — the comment is the audit trail.
3. **After 14 days unhandled**: Included in the monthly report to the
   application security responsible, with high-risk repositories flagged
   first, matching the pattern used for the other KPIs.

{{< alert "circle-info" "Rotation is the default, not history rewriting" >}}
It's tempting to "just remove it from the history" with `git filter-repo` or
similar, especially for private low-risk repositories or short-lived feature
branches. This is rarely enough on its own — the secret has typically already
been copied into CI logs, build artifacts, container images, mirrors, or
developer clones, and history rewriting requires a force-push that breaks
existing clones and open PRs. Rotation invalidates the credential everywhere
at once; history rewriting is at best a cleanup step on top.
[Handling Secret Exposures]({{< ref "handling-secret-exposures-github-enterprise" >}})
covers the edge cases where the trade-off is more nuanced.
{{< /alert >}}

## Putting It Together

These four KPIs build on each other. The order matters:

1. **Inventory completeness** must be ≥ 95% before the other metrics are
   trustworthy. Gaps in the inventory mean gaps in every downstream KPI.
2. **Scan coverage** tells you what percentage of the inventory you're actually
   monitoring. A system you're not scanning is a system you can't measure.
3. **SLA compliance** and **secret exposure rate** measure whether controls are
   working in practice, not just whether they're configured.

A monthly review of all four, with owners identified for each threshold breach,
is the operational cadence that keeps the program honest.

## Other KPIs to Consider

There's no universal set of "correct" AppSec KPIs. What's useful depends on
your organization's size, risk profile, regulatory exposure, and where your
program is in its maturity. Once the four foundational KPIs are running
smoothly, these are worth considering — but each one assumes a process or
dataset that isn't necessarily in place on day one:

- **Threat modeling coverage** — % of high-risk systems with documented threats
  within the last 12 months. Useful once you have an established threat
  modeling practice; meaningless before that.
- **Push protection bypass rate** — How often developers override secret push
  protection, and with what justification. Low and stable is healthy. A rising
  trend usually means one of two things: push protection is configured too
  aggressively and producing false positives, or the override flow is being
  treated as a formality. Either is worth investigating before it normalizes.
- **Mean time to remediate (MTTR)** — Median days from detection to fix for
  Critical and High severity findings. Read as a *trend* rather than a verdict:
  if SLA compliance is healthy, MTTR mostly tells you whether you're getting
  faster or slower over time. A sudden change — in either direction — is worth
  understanding. Requires consistent timestamps and enough volume for the
  median to be meaningful.

Pick what answers a question you actually need answered. A KPI nobody acts on
is just another row in a spreadsheet.

---

With the numbers we are going to track in place, let's continue by setting up
the system inventory.
