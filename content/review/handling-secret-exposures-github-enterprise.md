+++
title = 'Handling Secret Exposures in GitHub Enterprise'
date = 2026-06-02T15:05:00+02:00
draft = false
description = "How to respond when a secret reaches your GitHub repository — prevention, rotation, tracking, and the edge cases that make 'just delete the commit' a bad default."
summary = "Push protection stops most secrets before they land. When one slips through, the response needs to be automatic and visible — not a private Slack message someone hopes to remember. This post covers the full workflow: detection, rotation, tracking, and when (if ever) history rewriting is appropriate."
slug = "handling-secret-exposures-github-enterprise"
series = "appsec_github"
relatedPosts = ["appsec-kpis", "appsec-program-github-intro"]
tags = ["GitHub", "AppSec", "GHAS", "Secret Scanning"]
categories = ["appsec"]
+++

{{< series "appsec_github" >}}

A secret in a repository is a credential that lives on more machines than you
can name. By the time secret scanning flags it, it has typically been copied
into CI logs, build artifacts, container images, mirrors, and every developer's
local clone. Rotation is the only response that addresses all of those at once.
Everything else — issue tracking, branch cleanup, history rewriting — is
supporting work around the rotation.

This post covers the full response: prevention, the automated workflow that
makes handling visible, rotation per credential type, and the edge cases where
the trade-offs are more nuanced than the general rule suggests. It's the
implementation post for the secret exposure rate KPI defined in
[AppSec KPIs]({{< ref "appsec-kpis" >}}).

## Prevention First: Push Protection

The cheapest secret to handle is the one that never leaves the developer's
machine. **Push protection** rejects pushes containing recognized secret
patterns before they reach the server. GitHub Secret Protection ships with
detectors for hundreds of credential types — cloud provider keys, API tokens,
private keys — and updates them continuously.

Enable push protection on every repository as part of the scan coverage
baseline (see [KPI 2]({{< ref "appsec-kpis#kpi-2-scan-coverage" >}})). Track
how often it's bypassed using the
[push protection bypass rate]({{< ref "appsec-kpis#other-kpis-to-consider" >}})
metric — a rising trend usually means either too many false positives or a
team treating the override as a formality.

A successful push protection block is a non-event from a security perspective:
the secret never reached the server, no rotation is needed, no incident occurs.
That's the goal. Everything below this section is about what to do when
prevention fails.

## The Response Workflow

When secret scanning detects a credential that *did* reach the repository, the
response needs to be automatic and visible. Relying on someone seeing an email
or remembering to check the Security tab doesn't scale and doesn't survive an
audit.

### On Detection

A GitHub Actions workflow listens for `secret_scanning_alert` events and:

1. **Opens a tracking issue** in the affected repository, listing what was
   exposed (alert URL, type of secret, file path, commit reference) and the
   required steps — rotate, assess exposure, document the resolution in a
   comment.
2. **Notifies the team channel** with a link to both the alert and the
   tracking issue.
3. **Assigns the issue** to the repository's primary maintainer team using the
   same team-lookup mechanism described in
   [Building a System Inventory in GitHub Enterprise]({{< ref "github-enterprise-system-inventory" >}}).

The GitHub security alert remains the source of truth. The issue is for team
visibility, conversation, and audit trail — closing the alert without a
documented resolution is what we want to make difficult.

### Weekly Reminders

A scheduled workflow runs weekly:

- Finds open secret scanning alerts older than seven days.
- Checks whether the corresponding tracking issue is still open or was closed
  without a resolution comment.
- Re-pings the team channel with a "still open" reminder.

Closing the issue without writing what happened doesn't count. The comment is
the audit trail — "Rotated 2026-06-03, no evidence of use in CloudTrail" is a
valid resolution; clicking close is not.

### 14-Day Escalation

If the alert is still open after 14 days, the next monthly report to the
application security responsible includes it. High-risk repositories are listed
first and flagged for follow-up that month; low-risk repositories appear in the
standard backlog. This matches the escalation pattern used for inventory
completeness and scan coverage — predictable, supportive, and impossible to
ignore for long.

## Rotation Per Credential Type

"Rotate the credential" sounds simple. In practice, the steps vary by
credential type, and a runbook per type saves time during the rare cases that
actually matter.

| Credential type | Primary action | Verification |
|---|---|---|
| Cloud provider key (AWS, GCP, Azure) | Disable the key, issue a new one | Check provider audit log for use of the old key after exposure |
| Database password | Rotate the user's password, update consumers | Connection logs for unexpected source IPs during the exposure window |
| OAuth client secret / app token | Revoke and reissue from the provider's developer console | Provider's audit log or token activity feed |
| Signing key (GPG, code signing, JWT) | Revoke, publish revocation, generate a new key | Check for signatures or tokens issued after exposure |
| Webhook secret | Rotate at both sender and receiver | Sender's delivery log for replay attempts |
| Personal Access Token | Revoke from GitHub user settings | GitHub audit log for use of the token |

For each, the team's runbook should also cover *who has the authority to
rotate* — for shared service credentials, that's not always the same person
who pushed the secret.

## Why Not Just Rewrite History?

It is tempting, especially for private low-risk repositories or short-lived
feature branches, to reach for `git filter-repo` or BFG and "make the commit
go away." This is rarely sufficient as a primary response, for three reasons.

**The credential has already left the repository.** Anywhere CI ran on the
affected commit, the secret was likely loaded into an environment variable and
written to a build log. Container images built from the commit contain the
credential. Forks made before the rewrite still have it. Developer clones
still have it. GitHub's own reflog and search index retain references for some
time after deletion. History rewriting fixes the repository; it does not fix
the world.

**Rewriting history breaks downstream work.** A force-push invalidates every
existing clone, every open PR against the affected branch, and every CI run
in flight. For a busy repository this turns a secret incident into a
half-day of confused teams asking why their branches are gone.

**Rotation is faster and more definitive.** Disabling a cloud key takes
seconds. Verifying that you've found every copy of a committed secret takes
hours and ends in uncertainty. Rotation moves the question from "did I clean
up everywhere?" to "is the old credential still accepted?" — and the answer
to the second is a definite no.

History rewriting has its place as a *cleanup step* after rotation, especially
for repositories that will be made public. But it is not a substitute for
rotation, and treating it as one is the most common mistake teams make with
secret incidents.

## Edge Cases

The general rule — rotate, then track, then optionally clean up — covers most
cases. A few situations deserve a more deliberate decision.

**Secret pushed to a personal feature branch, no CI ran, no fork exists, no
one else fetched.** Technically the exposure window is small. Practically: it
has still been transmitted to GitHub's infrastructure, indexed by secret
scanning, and logged. For low-value credentials in a private repository this
*can* be a judgement call, but the time spent making that judgement is usually
greater than the cost of rotating. Default to rotation.

**The detected "secret" is a test value or example credential.** False
positives happen. Mark the alert as a false positive in the GitHub UI with a
comment explaining what the value is and why it's safe. The tracking issue can
be closed referencing the false positive marking. Recurring false positives
from the same pattern are worth a custom pattern exclusion rather than
repeated dismissals.

**The credential is no longer in use anywhere.** Still rotate — or in this
case, formally revoke. A credential that "isn't used" but still authenticates
is a credential that can still be used. Document the revocation in the
tracking issue.

**The credential is shared across many services and rotating it is
expensive.** This is a planning problem disguised as a security problem. The
short-term answer is still to rotate; the longer-term answer is to move toward
per-service credentials so a single exposure has a bounded blast radius. Track
this as an architectural follow-up, separate from the incident itself.

## Tying It Back to the KPI

The secret exposure rate KPI tracks two numbers — secrets blocked at push
versus secrets discovered post-commit. The process above is what gives the
second number meaning. Without an automated workflow that opens issues,
sends reminders, and escalates, "secrets discovered" is just an alert in a
dashboard nobody reads. With it, you have a measurable, auditable response —
and a strong incentive for teams to make sure push protection catches things
in the first place.

---

With detection, response, and rotation in place, the program has the controls
to catch problems and the workflows to handle them. The remaining piece is
using the data: querying the inventory and the security findings together to
answer the questions that actually come up — incident scoping, audit
evidence, and "which systems are affected by this new CVE?" That's covered
in [Putting the GitHub System Inventory to Work]({{< ref "github-enterprise-system-inventory-in-practice" >}}).
