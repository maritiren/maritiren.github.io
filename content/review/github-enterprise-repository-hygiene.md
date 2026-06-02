+++
title = 'Repository Defaults and Rulesets in GitHub Enterprise'
date = 2026-04-29T10:00:00+02:00
draft = false
description = "The baseline every repository should have, and how to use Repository Rulesets to enforce it across the organisation without per-repo configuration."
summary = "A short, opinionated baseline of repository defaults (code review, branch protection, Dependabot, push restrictions) and how to enforce them with Repository Rulesets so every repository inherits them automatically."
slug = "github-enterprise-repository-hygiene"
series = "appsec_github"
relatedPosts = ["appsec-program-github-intro", "github-enterprise-system-inventory"]
tags = ["GitHub", "AppSec", "Governance", "GHAS"]
categories = ["appsec"]
+++

{{< series "appsec_github" >}}

{{< alert "triangle-exclamation" "Work in progress" >}}
This post is an early draft
{{< /alert >}}

A repository without enforced defaults is a repository that depends on every
developer remembering to enable the right settings. In a small team that can
work. At organisation scale it doesn't. Defaults should to be applied
automatically, deviations should to be deliberate, and someone should to be able
to see which repositories are out of line.

This post maps out the settings worth configuring in every repository. The
settings fall into three areas: branch protection rules enforced through
Repository Rulesets, security features in the repository Security tab, and
Actions permissions. The final sections explain how to enforce branch protection
and how to handle exemptions.

In-depth coverage of individual features (scan coverage, SLA enforcement,
secrets management) for `high` tiered repos are covered in the other posts in
this series. Here the focus is the common baseline that applies to all
repositories regardless of risk tier.

## Scenarios that Would be Prevented by Good Defaults

The defaults in this post are not theoretical. They are based on best practices
and would have prevented several scenarios I've experienced on my own.

I have seen code review skipped on a project because the team said it wasn't
worth it. That resistance is often a sign that the team lacks knowledge in
good development hygiene, and that knowledge gap is an organisational risk in
its own right.

I have seen a helper team join a project that had no push protection and no
required reviews on the default branch. They pushed unreviewed infrastructure
changes directly to main without the owners controlling what was added. The
cleanup took longer than it would have for the owners to do it properly from the
start.

I've been involved with a project that got compromised through a third-party
dependency with a known vulnerability that had been sitting there for months,
because no alert had ever fired and no one had thought to look. And honestly, 
which AppSec Engineer haven't been? Or at least read about a bunch of similar
cases it.

All three were preventable with settings that cost nothing to enable.

## The Baseline Every Repository Should Have

Every repository has settings spread across three places in GitHub: branch
protection rules enforced via Rulesets, security features in the Security
tab, and Actions permissions. The tables below cover each area with a
priority rating.

**Essential** means nearly every organisation should have this enabled.
Skipping it is a risk, not a deliberate trade-off. **Recommended** is a
strong default with some legitimate exceptions. **Nice-to-have** is worth
enabling where the overhead is low.

### Branch Protection

These controls are enforced through Repository Rulesets. The next section
covers how to apply them across all repositories at once.

| Setting | Priority |
|---------|----------|
| Require at least one approval before merging | Essential |
| Dismiss stale approvals when new commits are pushed | Essential |
| Block direct pushes to the default branch | Essential |
| Block force-pushes on the default branch | Essential |
| Block deletion of the default branch | Essential |
| Require CI status checks to pass before merge | Essential |
| Restrict ruleset bypass to a named list | Essential |
| Delete head branches automatically after merge | Recommended |
| Require linear history | Recommended |
| Require signed commits | Nice-to-have |

Code review is one of the highest-value activities for code quality. The only
defensible case for skipping it is a solo developer with no reviewer available.
AI-assisted review is a useful addition but not a substitute: the PR author owns
the suggestions, using AI-generated feedback from a code review is just the same
as using AI-generated code for creating the feature. It does not replace a
human-being checking for any mistakes or misunderstandings their team-mate (and
their AI assistant) might have. 

Not dismissing stale reviews when new commits are pushed allows for sneaking in
unreviewed changes. 

Blocking direct pushes without requiring reviews is incomplete; requiring
reviews without blocking direct pushes is unenforceable. The two rules only work
together.

Automatically deleting head branches after merge is mostly a hygiene and
housekeeping improvement. It keeps the branch list manageable and reduces stale
noise, but it is not a core control, which is why it sits at Recommended.

"Require CI status checks to pass before merge" at the baseline level means
build and unit test checks, not necessarily SAST or vulnerability scan results.
We come back to those in later posts. What's important to mention here, is that
one should never enforce a quality gate without the possibility to bypass it. A
blocking security scan or linter might stop emergency fixes and become very
costly. The bypass should be narrow, named, and logged.

Linear history (squash or rebase merges) is partly a developer workflow choice,
but it has real operational value: `git bisect` works reliably on a linear
graph, and incident timelines are much easier to trace when each commit is a
self-contained logical change. I recommend it, but I believe it is up to the
team what they prefer.

### Security Features

These settings are configured in the repository **Security** tab, or at
organisation level in GitHub Enterprise settings. Anything marked GHAS
requires a GitHub Advanced Security licence for private repositories; for
public repositories it is included for free.

| Setting | Priority | Default (private repo) | Licence |
|---------|----------|----------------------|---------|
| Dependency graph | Essential | Off | Free |
| Dependabot alerts | Essential | Off | Free |
| Dependabot security updates | Essential | Off | Free |
| Secret scanning | Essential | Off | GHAS (or add your own GitHub Action) |
| Push protection | Essential | Off | GHAS |
| Code scanning (CodeQL default setup) | Recommended | Off | GHAS |
| Dependency review on PRs | Recommended | Off | GHAS |
| Dependabot version updates | Recommended | Off | Free |
| Private vulnerability reporting | Recommended | Off | Free |

Dependabot alerts and security updates are related but different. Alerts
tell you a dependency has a known vulnerability. Security updates open a
pull request to fix it. There is almost no reason to enable alerts without
also enabling security updates. Version updates are broader: they open PRs
to update all dependencies to their latest versions on a schedule, regardless
of known CVEs. They require a `dependabot.yml` file and generate more PR
volume, which is why they land at Recommended rather than Essential.

Push protection blocks pushes containing known secrets before they land in
the repository. Secret scanning alerts after the fact. Both are worth
having; if you can only enable one, push protection prevents the problem
rather than detecting it after it has already landed in history.

### Actions Settings

These settings live in the repository **Settings → Actions** tab and can
also be set at organisation level.

| Setting | Priority | Default |
|---------|----------|---------|
| Restrict allowed Actions to GitHub and Marketplace-verified creators | Essential | All allowed |
| Set default `GITHUB_TOKEN` permissions to read-only | Essential | Read and write |
| Require approval for fork PR workflows | Essential for private/internal, Recommended for public OSS | Off |
| Pin Actions to a full commit SHA in workflows | Recommended | Convention only |

GitHub offers three options for which Actions can run in your repositories:
all actions from any public source, actions from GitHub and
Marketplace-verified creators, or an explicit allowlist of approved actions.
"Verified creators" means publishers with a verified badge in the GitHub
Marketplace. Restricting to GitHub and verified creators is a low-friction
baseline control with meaningful supply chain risk reduction, which is why it
is marked Essential.

The default `GITHUB_TOKEN` with read and write permissions means any
workflow can modify repository content, open issues, or push commits,
intentionally or not. Setting it to read-only and requiring workflows to
declare what they need follows least privilege and limits blast radius. This
is also low-friction and high-impact, so it is Essential.

Fork workflow approval is where context matters: private and internal
repositories should treat it as Essential, while public OSS repositories may
need a faster contributor flow and can classify it as Recommended with clear
compensating controls.

Pinning an action to a tag (`uses: some-action@v2`) means the workflow runs
whatever the maintainer has pointed that tag at. Pinning to a full commit
SHA ensures the workflow only runs the exact code you reviewed, but it adds
maintenance overhead, which is why it stays at Recommended.

To enforce these baselines, we are going to look further into repository
rulesets.

## Repository Rulesets in Brief

Rulesets are GitHub Enterprise's mechanism for defining policy rules once at
the organisation level and applying them to many repositories at once. They
have largely replaced per-repository branch protection rules, which required
configuration on every single repository.

A ruleset has three parts:

- **Target**: which repositories and which branches it applies to. Can be
  all repositories, repositories matching a name pattern, repositories with
  specific Custom Property values, or a manual list.
- **Rules**: what is enforced. Required reviews, status checks, push
  restrictions, signed commits, linear history, and so on.
- **Enforcement status**: `active` (rules are enforced), `evaluate`
  (violations are logged but not blocked), or `disabled`.

For the full reference, see the
[GitHub Docs on Repository Rulesets](https://docs.github.com/en/enterprise-cloud@latest/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets).
The posts in this series focus on *how* to use rulesets for specific goals,
not the complete configuration surface.

{{< alert "lightbulb" "Always start in evaluate mode" >}}
Before flipping a ruleset to `active`, run it in `evaluate` mode for at least
a week. Violations are recorded in the ruleset insights without blocking any
work. This is how you find out which existing workflows the new rules would
break. Bots that push directly to `main`, automation that force-pushes,
release tags created outside the review process: these show up in insights
before the rule starts blocking real work.
{{< /alert >}}

## Enforcing the Baseline

Rulesets can target all repositories in the organisation or a scoped subset
defined by Custom Properties. Regardless of the end goal, starting with a
pilot group is worth considering: it lets you validate the ruleset against
real workflows before it affects everyone, and it turns up edge cases while
the blast radius is still small.

The [system inventory post]({{< ref "github-enterprise-system-inventory" >}})
covers how to define Custom Properties for departmental ownership. A ruleset
targeting `department` = `your-department` applies only to repositories
tagged with that value. 

A practical rollout sequence:

1. Create the ruleset targeting a pilot department or group. Set the branch
   target to the default branch (GitHub Rulesets support `~DEFAULT_BRANCH`
   as a pattern that follows each repository's configured default, so you
   don't need to hardcode `main`).
2. Run in `evaluate` mode for at least a week and review the insights.
3. Convert legitimate violations into allowlist entries or compliance work.
4. Flip to `active` for the pilot.
5. Expand to additional departments once you are confident in the configuration.

Expanding scope should be a deliberate step; the rules should not spread
silently to teams that have not been prepared.

This ruleset is the foundation. Every other ruleset in this series (scanning
enforcement, signed commits for critical systems, secret push protection) adds
*on top* of this baseline, not in place of it.

## Stricter Defaults for Critical Systems

The baseline applies to every repository. Create a second ruleset targeting
repositories in the `high` risk tier, using the combination of
`business-criticality`, `data-classification`, and `internet-facing` Custom
Properties covered in the system inventory series.  On top of the baseline,
consider:

- **Require review from a code owner.** An approval from any available
  reviewer is less valuable than one from someone with domain knowledge for
  the specific code being changed. CODEOWNERS files map directory paths to
  responsible teams; when required, the PR cannot merge until a member of
  the right team has approved.
- **Block force-pushes on release branches** such as `release/*`, in addition
  to the default branch.
- **Restrict bypass to trusted teams with clear ownership.** Team-level bypass
  can work well when the team is explicitly accountable and membership is
  reviewed. Named-individual bypass is most useful for the subset of systems
  with strict compliance requirements.

Two required approvals and signed commits are worth considering for the subset
of repositories that are both mission-critical and subject to compliance or
regulatory requirements. For most organisations, the overhead of operationalising
commit signing for all developers, service accounts, and CI pipelines outweighs
the practical security benefit at the broader high tier.

A repository that gets re-tagged falls under this ruleset the moment its
property changes. No additional admin work needed.

## Handling Exemptions

Some repositories will have legitimate reasons to deviate. A release-automation
service account that needs to push tags. A repository under active migration
where strict controls would block routine work.

The right way to handle these is *named, time-bound exceptions*, not
disabling rules for everyone. Two mechanisms help:

- **Bypass list on the ruleset.** Add the specific account or team that needs
  to bypass, with a comment explaining why. Review the list quarterly.
- **Repository exclusion via Custom Property.** Add a property such as
  `ruleset-exempt`, and exclude repositories with that value from the
  ruleset's target. The exemption becomes visible both in the property and
  in the ruleset configuration, and it's easy to audit. (TODO: Tenk mer over denne)

Archived repositories do not need to be listed as exceptions. GitHub makes
archived repositories read-only at the platform level, so no pushes are
possible and ruleset rules cannot be triggered.

Every exemption should leave a trail.

With the baseline in place across all repositories and stricter rules layered on
top for critical systems, the foundation is set. The next question is whether
the right scans are actually running on the repositories that need them. A
ruleset can require a CI check to pass before merging, but it cannot require
that the check exists in the first place. Ensuring scan coverage is a separate
concern, and the [next post]({{< ref "scan-coverage-github-enterprise" >}})
covers how to enforce it with Code Security Configurations and Repository
Rulesets targeted by Custom Properties.

