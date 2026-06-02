+++
title = 'Putting the GitHub System Inventory to Work'
date = 2026-05-20T10:00:00+02:00
draft = false
description = "How to use GitHub Custom Properties for vulnerability prioritization, breach response, and audit evidence — and what the inventory looks like under scrutiny."
summary = "Once the system inventory is in place, GitHub's repository filtering becomes your query interface. This post covers the practical use cases: vulnerability prioritization, breach response scoping, audit preparation, and what to show an auditor when they ask whether you know what systems you have."
slug = "github-enterprise-system-inventory-in-practice"
series = "appsec_github"
relatedPosts = ["appsec-program-github-intro", "github-enterprise-system-inventory"]
tags = ["GitHub", "AppSec", "Governance", "GHAS"]
categories = ["appsec"]
+++

{{< series "appsec_github" >}}

With Custom Properties in place, GitHub's repository filtering is your query
interface. The same three properties — `business-criticality`,
`data-classification`, and `internet-facing` — show up across every use case
below. That's by design: a small, well-maintained schema pays dividends across
multiple processes.

## Using the Inventory

**Vulnerability prioritization**: Filter `business-criticality: mission-critical`
combined with `internet-facing: yes` to define your highest-priority remediation
scope. These are the repositories where an unpatched vulnerability has the
shortest path to real impact.

**GDPR breach response**: Filter `data-classification: restricted` or
`confidential` to scope which systems are in play during a data breach. Within
minutes of an incident you can answer "which repositories handle restricted
data?" — without searching through spreadsheets or asking five different team
leads.

**Audit preparation**: Export repos with `dpia-completed: no` and
`data-classification: confidential` to generate a DPIA remediation list. The
filter runs in seconds; the remediation takes longer.

**Attack surface review**: Filter `internet-facing: yes` combined with
`authentication-type: basic-auth` or `api-key` to find systems potentially
needing authentication hardening before a penetration test.

**Incident response**: Filter `hosting-platform: aws` to immediately see all
affected systems during a cloud provider outage — no manual triage required.

## What This Looks Like in an Audit

When an auditor asks whether you know what systems you have and how sensitive
they are, you want to answer with evidence — not a spreadsheet someone updated
six months ago.

With this setup you can:

- **Show filtered repository views** by classification, criticality, or owner,
  directly in GitHub — live, not exported
- **Demonstrate required properties** are enforced at the organizational level,
  not just requested
- **Point to a single source of truth** maintained by the teams who own the
  systems, updated as part of normal repository management
- **Show change history** — the GitHub audit log records when properties were
  set and by whom

This covers the asset management control in **ISO 27001 A.5.9**, supports **SOC
2 CC6.1**, and gives a structured foundation for **GDPR Article 30** records of
processing activities — particularly when paired with `dpia-completed` and
`data-classification`.

---

*This is the final post in the series on building an AppSec program in GitHub Enterprise.
Start from the beginning with the [series introduction]({{< ref "appsec-program-github-intro" >}}).*
