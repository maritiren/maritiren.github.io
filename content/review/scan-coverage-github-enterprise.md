+++
title = 'Scan Coverage in GitHub Enterprise'
date = 2026-06-09T18:14:00+02:00
draft = false
description = "How to enforce scanning requirements across repositories using GHAS Code Security Configurations and Repository Rulesets."
summary = "Different systems need different scans. This post shows how to enforce scan coverage with Code Security Configurations and Repository Rulesets targeted by Custom Properties."
slug = "scan-coverage-github-enterprise"
series = "appsec_github"
relatedPosts = ["github-enterprise-repository-hygiene", "appsec-kpis", "github-enterprise-system-inventory"]
tags = ["GitHub", "AppSec", "GHAS", "Scanning"]
categories = ["appsec"]
+++

{{< series "appsec_github" >}}

{{< alert "triangle-exclamation" "Work in progress" >}}
This post is an early draft. The structure and examples below are the
starting point and will be expanded.
{{< /alert >}}

The [AppSec KPIs]({{< ref "appsec-kpis#kpi-2-scan-coverage" >}}) post defines
what scan coverage should look like as a metric and how compliance is
calculated. This post covers the implementation: *how* to make those
requirements stick in GitHub Enterprise.

The mechanism is two-part: Code Security Configurations enable the right GHAS
tooling, and Repository Rulesets enforce scanning on pull requests. Both are
targeted using Custom Properties, so re-classifying a repository automatically
updates which configuration and rules apply.

This post walks through creating configurations for each risk tier, adding
rulesets that enforce the required checks, handling internet-facing repositories
that need stricter enforcement regardless of criticality, and defining a
bypass path for urgent fixes.

## The Two Mechanisms

[**Code Security Configurations**](https://docs.github.com/en/enterprise-cloud@latest/code-security/concepts/security-at-scale/organization-security#about-security-configurations)
are organisation-level templates that enable GHAS features — code scanning,
secret scanning, Dependabot — on a targeted set of repositories. They replace
per-repository manual enablement.

[**Repository Rulesets**](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)
define branch and merge policies at the organisation level. Required status
checks, merge restrictions, and bypass policies are all configured here.

## Prerequisites

This post depends on:

- An accurate system inventory
  ([Building a System Inventory]({{< ref "github-enterprise-system-inventory" >}}))
  with `business-criticality`, `data-classification`, and `network-exposure`
  set on all repositories.
- Baseline repository controls
  ([Repository Defaults and Rulesets]({{< ref "github-enterprise-repository-hygiene" >}}))
  already in place. Scan coverage rulesets are added on top of that baseline.

## Tier Requirements

Use the same two-tier model as the KPI post:

| Tier | Required scanning |
|---|---|
| High risk | Code scanning (CodeQL or equivalent), IaC scanning, secret scanning with push protection, dependency review/SCA |
| Low risk | Secret scanning, dependency review/SCA |

## Setting Up Enforcement

The steps below set up the enforcement of scan coverage: first enable the right
tooling on the right repositories, then add merge enforcement on top. Skipping
the first step means the required scanning checks won't exist for the rulesets
to depend on.

### Step 1: Create Code Security Configurations

In your organization's {{< org-link path="settings/security_products/configurations" text="Code Security Configurations" >}}, create two configurations:

1. `ghas-high-risk`: enable code scanning, secret scanning, and Dependabot.
2. `ghas-low-risk`: enable secret scanning and Dependabot baseline.

Target them with Custom Property filters so repository classification controls
which configuration applies.

### Step 2: Add Rulesets for Enforcement

Create one ruleset per tier. Each ruleset targets the matching Custom Property
values and defines what must hold true before a pull request can merge.

| Rule | High risk | Low risk |
|---|:---:|:---:|
| Code scanning check must complete | Yes | No |
| Dependency review check must complete | Yes | Yes |
| Block on new secret introduced | Yes | Yes |
| Block on new **critical** vulnerability introduced | Yes | No |
| Block on new **high** vulnerability introduced | Optional | No |

Typical status check names to require in the ruleset:

- `CodeQL` (or your equivalent SAST workflow check)
- `Dependency Review`

TODO: Add screenshots and rulesets shown more in detail

### Step 3: Define Bypass for Urgent Fixes

Rulesets support a built-in bypass list: specific teams or roles that are
allowed to merge even when rules would otherwise block. This is configured
directly in the ruleset settings.

Bypass activity is visible in the ruleset's **Insights** tab and can be
included in the monthly AppSec report, giving the AppSec team visibility
into whether bypass is being used responsibly or becoming a habit.

For accountability, a GitHub Actions workflow triggered on merge can detect
whether a bypass occurred and automatically open an issue on the repository
asking the team to document their reasoning. Most teams will treat a bypass
sparingly once they know it generates a follow-up. Teams that bypass
repeatedly are a signal for the AppSec engineer to reach out directly.

{{< alert "circle-info" >}}
The follow-up issue workflow requires a small custom GitHub Actions
implementation. The bypass detection itself uses the ruleset Insights API or
the pull request event payload.
{{< /alert >}}

### Rollout Recommendation

Start rulesets in `evaluate` mode, review misses for 1 to 2 sprints, then
switch to enforcement.

## What This Gives You

With configurations and rulesets in place, scan coverage becomes measurable
and enforceable. You have control over which repositories have the required
tooling active and which don't. _That visibility is what makes the KPI
meaningful_. Repositories that fall outside the automated targeting still
show up as gaps, which gives AppSec a clear list to work through, often
requiring direct contact with the team to understand what's blocking them.

This is also the point where the {{< org-link base="https://github.com/orgs" path="security/coverage" text="Security Overview" >}} dashboard becomes
genuinely useful: you can filter by tier, exposure, or team and see at a
glance which repositories are covered, which have open findings, and where
the gaps are. How to use that view for incident scoping, audit evidence, and
continuous monitoring is covered in
[Putting the System Inventory to Work]({{< ref "github-enterprise-system-inventory-in-practice" >}}).

With scanning in place, the next step is tracking whether the findings it
surfaces are actually being fixed. That's covered in
[SLA Compliance and Automated Escalation]({{< ref "sla-compliance-github-enterprise" >}}).
