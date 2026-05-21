+++
title = 'Application Security in GitHub Enterprise — Introduction'
date = 2026-04-15T10:00:00+02:00
draft = false
description = "Many security programs look good on paper but fail when it matters. This series builds the foundations that separate real risk reduction from compliance theater — on GitHub Enterprise."
summary = "Dashboards, scanning tools, compliance boxes — and still no clear answers when something goes wrong. This series builds the foundations that make a security program actually work: a complete system inventory, risk tiers, and metrics that trigger action rather than generate reports."
slug = "appsec-program-github-intro"
series = "appsec_github"
relatedPosts = ["appsec-kpis", "github-enterprise-system-inventory"]
tags = ["GitHub", "AppSec", "Governance", "GHAS"]
categories = ["appsec"]
+++

{{< series "appsec_github" >}}

A lot of organizations have security programs that look good on paper: scanning
tools running, dashboards filled with numbers, compliance boxes ticked. And yet
when something goes wrong — a breach, an audit finding, a question from the
board — the answers aren't there. Which systems were affected? Who owns them?
Are they critical? Were they actually being scanned? Was the vulnerability
known? Security theater has a way of becoming visible at exactly the wrong
moment.

This series is about building something that isn't that. Not a compliance show,
but a program that actually reduces risk — and can demonstrate that it does.

The difference comes down to foundations. Without a complete, accurate inventory
of your systems, you don't know what you're protecting. Without risk tiers, you
apply the same pressure to a payment processor and an internal wiki — and teams
stop taking the requirements seriously. Without metrics tied to real thresholds,
you have reporting but no accountability. These aren't implementation details.
They're what separates a program that works from one that generates paperwork.

For developers, the practical difference is clear expectations, rules that
reflect the actual risk of their system, and direct communication of deviations
from expected practice. For management, it means knowing your most critical
systems are actually covered and being able to show an auditor that the policy
is enforced. 

This series builds that foundation on GitHub Enterprise: a system inventory,
risk tiers, KPIs that trigger action, and automation that enforces the rules
without requiring every developer to be a security expert.

## What This Series Covers

This series builds a practical AppSec program on GitHub Enterprise, one piece at
a time. Each post can stand alone, but the pieces build on each other — the
inventory is the foundation, the KPIs define what success looks like, the audit
scripts implements enforcement and notification mechanisms and each subsequent
post implements one of those KPIs. In the end, we discuss how to utilise the
entire system inventory.

**[AppSec KPIs]({{< ref "appsec-kpis" >}})**
defines four KPIs: inventory completeness, scan coverage, SLA compliance, and
secret exposure rate. Each has thresholds and a defined escalation path. With
the KPIs in place, we need to gather the data, starting with a system inventory.

**[Building a System Inventory in GitHub Enterprise]({{< ref "github-enterprise-system-inventory" >}})**
covers the foundation of the enire AppSec program. Custom Properties attach
structured metadata to every repository — data classification, business
criticality, internet exposure, and more. During an incident, this tells you
immediately which systems are in scope. During an audit, it's your evidence of
asset management. During regular controls, it is the data needed for
prioritizing effort where it is actually needed. Without this, the other KPIs
are measuring the wrong population. A good starting point is whether the repos
are running the necessary security scans.

**[Repository Defaults and Rulesets]({{< ref "github-enterprise-repository-hygiene" >}})**
covers the baseline defaults every repository should have — code review
enforcement, Dependabot, branch protection, no direct pushes to `main` — and
how to enforce them with Repository Rulesets so new repositories inherit them
automatically. Also covers how to handle exemptions for teams that have
legitimate reasons to deviate, without turning exceptions into the norm. This
is the hygiene baseline most compliance frameworks expect, applied without
relying on teams to remember.

**[Scan Coverage in GitHub Enterprise]({{< ref "scan-coverage-github-enterprise" >}})**
covers how to enforce tiered scanning requirements using GHAS Code Security
Configurations and Repository Rulesets. CodeQL and infrastructure scans for
critical systems, SCA and secret scanning for everything — including push
protection to catch credentials before they land in the repository. With the
proper scans in place, we can connect the scans with SLA compliance of fixing
findings.

**[SLA Compliance and Automated Escalation]({{< ref "sla-compliance-github-enterprise" >}})**
demonstrates how to track whether vulnerabilities are being fixed within agreed
timelines, and how to automate warnings, breach notifications, and escalation to
the CISO or responsible personnel using GitHub Actions. In addition, we can use
the risk profile of the systems to set and monitor good security hygiene in the
development process.

**[Handling Secret Exposures]({{< ref "handling-secret-exposures-github-enterprise" >}})**
covers the response process when a credential reaches a repository: push
protection as the first line, automated tracking issues and reminders when
something slips through, and rotation per credential type. This is the
implementation of the secret exposure rate KPI.

**[Putting the GitHub System Inventory to Work]({{< ref "github-enterprise-system-inventory-in-practice" >}})**
brings it all together. With the inventory, controls, and scanning in place,
GitHub's repository filtering and the Security Overview dashboard become your
query interface for the moments that matter: answering which systems are
affected when a new CVE drops, scoping the blast radius during an incident, and
producing audit evidence on demand. All without pulling a security team into
every question. This is where the foundation pays off, and where any gaps in
your inventory become visible.

The order of the posts isn't arbitrary. Accurate system metadata enables
correct targeting of controls. Baseline repository hygiene makes enforcement
predictable. That enables meaningful scan coverage, which enables measurable
SLA compliance. A gap at the bottom propagates upward, which is why inventory
and hygiene come first.

Throughout the series, you will read about GitHub Enterprise and GitHub Advanced
Security (GHAS), three different budget models, system metadata and risk tiers.
Read the remaining of this introductional post to understand these core
components of the AppSec framework that you should understand before moving on.

## Why GitHub Enterprise

This series focuses on GitHub Enterprise because the controls live close to the
code. Security configuration, scanning, and policy enforcement all sit in the
same platform developers already use day to day, which lowers the friction of
integrating security into the development workflow. Compared to the alternatives
I have worked with, GitHub Enterprise makes that integration better for
development teams, even with budget constraints. The tooling also maps directly
to the framework in this series: Custom Properties, Code Security
Configurations, Repository Rulesets, and the Security Overview dashboard. The
principles apply elsewhere, but the implementation is GitHub-specific.

Custom Properties, Required Properties enforcement, and Repository Rulesets
all require **GitHub Enterprise** (Cloud or Server 3.11+). For sufficient
control of system risks, **GitHub Advanced Security (GHAS)**, or similar
functionality, must be added on top. Not every organization can afford
that across every repository, so the next section walks through three
realistic models.

## The Budget Question: Three Models

The tooling model you choose affects what you can automate, what you can
measure, and what you can show an auditor. It's worth being deliberate about
it before configuring anything.

{{< tabs >}}
{{< tab "GHAS on Everything" >}}
Every repository gets full GitHub Advanced Security coverage.

**What you get**:
- Code scanning, secret scanning, and advanced Dependabot features on all repos (basic Dependabot alerts and security updates are available without GHAS — GHAS adds auto-triage rules, grouping, and dependency review)
- The Security Overview dashboard — a unified view across the organization
- Full automation potential for SLA tracking, owner notifications, and policy enforcement
- Pull request feedback on new vulnerabilities before they merge

**Best for**: Organizations where the cost of a breach outweighs the licensing
cost, or where compliance mandates full coverage.

**Risk**: If all findings surface at once without a triage strategy, teams get
overwhelmed. Use the criticality properties (configured in the
[system inventory post]({{< ref "github-enterprise-system-inventory" >}})) to
prioritize the rollout.

**Rough cost** (100 engineers, assuming 80 have pushed to a GHAS-enabled repo
in the last 90 days): 100 × ~$21 = ~$2,100 for GitHub Enterprise + 80 × ~$49
= ~$3,920 for GHAS ≈ **$6,000/month**.

{{< alert "circle-info" >}}
GitHub Advanced Security is now sold as two separate add-ons: 
*GitHub Code Security* (code scanning, advanced Dependabot features, dependency
review, security overview) and *GitHub Secret Protection* (secret scanning, push
protection). Both are priced per active committer.
{{< /alert >}}
{{< /tab >}}

{{< tab "Hybrid GHAS" >}}
GHAS is enabled only on repositories tagged as business-critical or high-risk.

**What you get**:
- Strong coverage where the business says it matters most
- Custom Properties define what "critical" means; rulesets target those repos automatically

**What you miss**:
- Security blind spots on "non-critical" repos — in real attacks these are
  common lateral movement targets. A low-criticality repo with a production
  deploy key or write access to a critical pipeline is not actually low risk
- No unified Security Overview across all repos
- SLA automation only covers GHAS-enabled repos

**Best for**: Organizations with a well-maintained criticality classification
and genuine budget constraints. Only works if someone is responsible for keeping
classifications accurate.

**Risk**: Attackers don't respect your criticality classifications. This model
requires discipline and honest reassessment of what "low criticality" actually
means.

**Rough cost** (100 engineers, GHAS on ~25% of repos): 100 × ~$21 = ~$2,100
for GitHub Enterprise. Assuming ~30 engineers have pushed to those GHAS-enabled
repos in the last 90 days: 30 × ~$49 = ~$1,470 for GHAS ≈ **$3,600/month**.
{{< /tab >}}

{{< tab "GHE + DefectDojo" >}}
Use GitHub Enterprise for the system inventory (Custom Properties), and
[Defect Dojo](https://www.defectdojo.org/) — the free, open-source OWASP
vulnerability management platform — for aggregating scan results from CI
pipelines.

**What you get**:
- System inventory in GitHub via Custom Properties
- Basic Dependabot alerts and security updates
- Centralized vulnerability management in Defect Dojo, independent of GitHub plan
- SLA tracking, deduplication, and reporting built into Defect Dojo

**What you miss**:
- No native GitHub integration — findings don't appear in the Security tab or PR
  checks
- Easy-to-setup pipelines for code scanning and secret scanning. These must be
  created and centralized on your own.
- CI pipelines must be wired up to push results to Defect Dojo manually
- PR-level feedback requires custom tooling

**Best for**: Organizations with existing Defect Dojo investment, or where
GitHub isn't the only SCM.

**Risk**: The operational overhead is easy to underestimate. Many teams set it
up, fill it with findings, and gradually stop maintaining it. If no one owns it
properly, the data goes stale.

**Rough cost**: ~$2,100/month for GHE + ~$150/month for hosting ≈ **$2,250/month**
in direct costs — but the real cost includes engineering time to build and
maintain CI pipelines, write integration tooling, and keep Defect Dojo running.
That ongoing effort is often underestimated and rarely shows up in the initial
budget proposal.
{{< /tab >}}
{{< /tabs >}}

### At a Glance

| | GHAS on All | Hybrid GHAS | GHE + Defect Dojo |
|---|:---:|:---:|:---:|
| Approx. monthly cost (100 engineers) | ~$6,000 | ~$3,600 | ~$2,250 |
| Operational overhead | Low | Low | High |
| Full repo coverage | Yes | Partial | Depends on CI |
| Secret scanning (native) | Yes | Partial | No |
| PR-level feedback (native) | Yes | Partial | No |
| Unified security view | GitHub | Partial | Defect Dojo |
| SLA automation (native) | Yes | Partial | No |
| Audit-ready out of the box | Yes | Partial | Medium |

{{< alert "circle-info" "Pricing is illustrative" >}}
GitHub Enterprise is publicly listed at ~$21/user/month (billed annually). GHAS
pricing requires a quote from GitHub sales and varies by contract. An active
committer is anyone who has pushed a commit to a GHAS-enabled repository within
the last 90 days. Use these numbers for relative comparison, not budgeting.
{{< /alert >}}

**Recommendation**: GHAS on everything gives the best visibility with the
lowest operational overhead. The hybrid model is a legitimate fallback — but
only with a process for keeping criticality classifications accurate. Defect
Dojo is meant as an example of a free tool to aggregate findings data. It suits
best organizations that has a security team that can delegate tasks to teams. 

This series uses the **hybrid model**: high-risk systems get GHAS, low-risk
systems don't. The risk tiers below decide which is which.

## System Metadata: Three Properties

The risk tiers, KPIs, and rulesets across this series all build on three
pieces of metadata attached to every repository:

| Property | What it captures | Values |
|---|---|---|
| `business-criticality` | How important the system is to the business | `mission-critical`, `business-critical`, `important`, `non-critical` |
| `data-classification` | Sensitivity of the data the system handles | `public`, `internal`, `confidential`, `restricted` |
| `network-exposure` | How reachable the system is on the network | `open-internet`, `vpn-only`, `restricted-access`, `local-only` |

These are implemented as GitHub
[Custom Properties](https://docs.github.com/en/organizations/managing-organization-settings/managing-custom-properties-for-repositories-in-your-organization)
at the organization level. Teams fill in the values on each of their
repositories, and the values become queryable across the org for filtering,
reporting, and targeting rulesets. The
[system inventory post]({{< ref "github-enterprise-system-inventory" >}})
covers the full schema, value definitions, and how to enforce that the values
are actually set.

Together, these three answer the questions that drive every downstream
decision: *how much does this matter, what does it touch, and who can reach
it?*

## Two Risk Tiers

The examples in this series use two risk tiers: **high** and **low**. A
repository is high risk if *any* of these are true:

- `business-criticality` is `mission-critical` or `business-critical`
- `data-classification` is `confidential` or `restricted`
- `network-exposure` is `open-internet`

Everything else is low risk. High-risk repositories get the full set of GHAS
controls (SAST, IaC scanning, secret scanning with push protection, SCA).
Low-risk repositories get a lighter baseline (secret scanning, SCA).

Two tiers keep licensing predictable and the rules easy to communicate, and
they're enough to drive meaningfully different controls without drowning teams
in nuance. A third tier can be useful later, for example to separate
`mission-critical` payment systems from internal tools that happen to be
internet-facing. Start with what fits your budget and reporting needs, and
refine when the tiers stop telling you something useful.

---

With the tooling model, metadata, and risk tiers in place, the next step is
deciding what to measure. The KPI post defines the metrics that tell you whether
the program is working, and the rest of the series implements them.
