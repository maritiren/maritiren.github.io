+++
title = 'SLA Compliance and Automated Escalation in GitHub Enterprise'
date = 2026-06-09T18:15:00+02:00
draft = false
description = "How to track whether vulnerabilities are being fixed within agreed timelines, and how to automate warnings, breach notifications, and escalation."
summary = "An SLA matrix is only useful if breaches are actually visible. This post covers how to track remediation timelines per severity and risk tier in GitHub Enterprise, and how to automate the escalation path from warning to CISO."
slug = "sla-compliance-github-enterprise"
series = "appsec_github"
relatedPosts = ["appsec-program-github-intro", "appsec-kpis", "scan-coverage-github-enterprise"]
tags = ["GitHub", "AppSec", "GHAS", "SLA"]
categories = ["appsec"]
+++

{{< series "appsec_github" >}}

{{< alert "triangle-exclamation" "Work in progress" >}}
This post is an early draft. The structure and examples below are the
starting point and will be expanded.
{{< /alert >}}

The [AppSec KPIs]({{< ref "appsec-kpis#kpi-3-sla-compliance" >}}) post
defines the SLA matrix: severity crossed with risk tier, with concrete
remediation timelines for each cell. This post is the implementation side.
*How* to track those timelines in GitHub Enterprise, and *how* to make
breaches visible without expecting anyone to remember to check a dashboard.

This post depends on:

- An accurate system inventory
  ([Building a System Inventory]({{< ref "github-enterprise-system-inventory" >}}))
  so the risk-tier dimension of the SLA matrix is reliable.
- Scan coverage in place
  ([Scan Coverage]({{< ref "scan-coverage-github-enterprise" >}}))
  so the findings being tracked are representative of the actual codebase.

Without those, SLA compliance numbers look fine while the real risk is
invisible.

## What This Post Will Cover

To be expanded:

- Mapping the SLA matrix to GHAS alert severities (code scanning, secret
  scanning, Dependabot).
- A scheduled workflow that walks open security alerts, calculates time
  remaining against the SLA, and notifies on warning and breach thresholds.
- Escalation paths: owner at warning, AppSec at breach, CISO on extended
  breach for Critical/High.
- Reporting compliance per tier and per team in the monthly summary.
- Handling findings that genuinely can't be remediated within SLA: risk
  acceptance, compensating controls, and how to record them so they don't
  silently dominate the breach numbers.
