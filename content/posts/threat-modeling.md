+++
title = 'Threat Modeling with Rapid Risk Assessment - a low-effort and concise approach'
date = 2024-11-15T17:40:58+01:00
draft = false
description = "A practical guide to implementing low-effort threat modeling using Rapid Risk Assessment in your organization"
summary = "A practical guide to implementing threat modeling with Rapid Risk Assessment in your organization, including challenges, tailoring approaches, and example processes."
slug = "threat-modeling"
tags = ["AppSec", "Threat modeling"]
categories = ["appsec"]
+++

Organizations can undertake various activities to enhance security in the
development process, with threat modeling being one of the most impactful
activities in early stages of development. Other similar activities are risk
analysis or security architecture review. Threat modeling is sometimes mistaken
as vulnerability analysis too. 

Even though threat modeling is a valuable activity, a lot of organizations
struggle to implement it. The challenge is twofold: most developers lack
application security expertise, but more fundamentally, threat modeling requires
a completely different way of thinking. Instead of the constructive mindset of
building features and solving problems, teams need to think adversarially—like
an attacker looking for weaknesses. This unfamiliar perspective creates
uncertainty and insecurity around the entire activity.

In this article, I will share my own thoughts and experiences on implementing
threat modeling. This includes implementation challenges, tailoring approaches,
supportive tools, and a complete example process. Although the scope is
organization wide, this article should be useful for implementation in just one
team as well.

## Benefits of performing threat modeling regularly

- **Identify attack points**: Understand where your service is vulnerable to attacks
- **Catch architectural issues early**: Find design flaws before they become expensive to fix, or identify improvements in existing systems
- **Create valuable documentation**: Data-flow diagrams and threat scenarios help onboard new team members and provide quick context for management, security teams, and external auditors like pentesters. Also threat scenarios might be useful input to risk analysis 
- **Builds security awareness**: Teams develop security thinking that carries into daily work
- **Creates shared understanding**: Gets everyone on the same page about what's worth protecting
- **Prioritizes security work**: Helps teams distinguish between critical risks and theoretical edge cases, so they can focus where it matters

## What is threat modeling?
Threat modeling is a proactive approach to identifying and mitigating potential
security threats in services.

I often describe threat modeling as a more engaging form of risk analysis,
focusing on the aspects that are more fun and valuable to developers, while
streamlining the process.

Let's look at how [Microsoft](https://learn.microsoft.com/en-us/training/paths/tm-threat-modeling-fundamentals/)
defines it: 
> Threat modeling is an effective way to help secure your systems, applications, networks, and services. It's an engineering technique that identifies potential threats and offers recommendations to help reduce risk and meet security objectives earlier in the development lifecycle.

For those interested in a deeper dive, I recommend starting with Rapid Risk
Assessment (RRA) and STRIDE. RRA is a short and sweet method and STRIDE is the
most known method.

### A Note on Terminology

Technically, what I describe in this article is closer to **threat
identification** than comprehensive threat modeling. Traditional threat modeling
frameworks like STRIDE involve deep analysis of attack paths and detailed threat
categorization. The approach in this article—using Rapid Risk Assessment—is
actually a hybrid between risk assessment and threat modeling. It's simpler and
more focused on identifying threats that teams can act on.

So why do I call it threat modeling? Because for most organizations without
advanced AppSec maturity, this simpler approach is what I believe is best to
adopt. If you say "we're doing threat modeling," people understand it's about
security. If you say "threat identification," it sounds incomplete.

At the organization where I implemented this, we actually used "threat
identification" in the title internally. This was more accurate, but externally
and for this article, "threat modeling" communicates the purpose better.

If your organization struggles with heavyweight threat modeling processes, this
lighter approach might be exactly what you need. Just be aware: if you say
"threat modeling" to Security Advisory or risk management teams, they might
expect something different—either comprehensive STRIDE analysis or formal risk
assessments.

## Threat modeling methods 
I've worked with [Rapid Risk Assessment (RRA)](https://infosec.mozilla.org/guidelines/risk/rapid_risk_assessment.html)
and implemented a tailored version for an organization. I haven't implemented
[STRIDE](https://owasp.org/www-community/Threat_Modeling_Process#stride), the
most common threat modeling method. Both methods share the same goal and way of
thinking—they differ in process and effort to get started.

According to [threat-modeling.com](https://threat-modeling.com/), other
frameworks include STRIDE, PASTA, and LINDDUN. Microsoft has their own tailored
STRIDE version.

Many methods follow similar stages:
1. Gather information about the service (often with a data-flow diagram)
2. Identify threats against the service
3. Implement mitigations (and verify them)

{{< alert "lightbulb" >}}
Rapid Risk Assessment is well explained with examples in [this video](https://www.rra.rocks/docs/podcasts).
{{< /alert >}}

### Risk Assessment vs. Threat Modeling: Where Does RRA Fit?

Rapid Risk Assessment is fundamentally a **risk assessment** method. Threat
modeling identifies *threats* (what could happen?), while risk assessment
evaluates *risks* (how likely and how bad?). RRA does both, but simplified.

This makes RRA a hybrid:
- **Like threat modeling**: thinking like an attacker, service-specific, creates
  actionable security issues
- **Like risk assessment**: evaluates likelihood and impact, produces risk-based
  decisions

RRA doesn't calculate formal risk scores—it focuses on identifying threat
scenarios and assessing worst-case impact. Mozilla designed RRA for any service
(infrastructure, websites, applications).

## Data-flow diagrams
Data-flow diagrams are commonly used in threat modeling, particularly with
methods like STRIDE and Microsoft's approach. Their primary purpose: **visualize
where data flows in your service**. This makes it easier to spot what's missing
and ensure all parts get considered during threat identification.

**Why they're valuable:**
- Makes scope explicit—even simple systems benefit from seeing "yes, it really is just these three components"
- Easier to spot what's missing: "wait, where does authentication happen?"
- Ensures all parts of the system get considered during threat identification
- Complex systems become clearer when broken down visually
- Useful documentation for onboarding and external reviews

**Creating the diagram:**
Create these **before the threat modeling session**, not during it. Ideally as
prep work before the first round, though not strictly required if time is
limited. Microsoft's approach works well: ask questions about the system (what
does it do? which services does it use? how do users access it? how does it
parse data?) and build the diagram from those answers.

Diagrams typically show processes, data stores, external entities, and data
flows. More advanced diagrams include trust zones (boundaries where security
context changes), which are valuable for identifying threats at boundaries—like
where user input enters your system or where you cross from your network to
external services. Start with clearly showing where data flows, then add trust
zones when they help clarify threat surfaces.

The [Microsoft Learn path](https://learn.microsoft.com/en-us/training/paths/tm-threat-modeling-fundamentals/)
explains data-flow diagrams well, including depth layers (how detailed to make
your diagrams). For most systems, layers 0 and 1 provide sufficient detail.

{{< alert "lightbulb" >}}
**Keep it simple.** The goal is understanding data flow to identify where
protection is needed. A one-minute review should give you a clear picture of
where data flows. Avoid unnecessary detail that obscures rather than clarifies.
{{< /alert >}}

**Tools:** [draw.io](https://draw.io) works well and is free. Microsoft suggests
[Visio](https://www.microsoft.com/en-US/microsoft-365/visio/flowchart-software)
and the [Threat Modeling Tool](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool).

## Common challenges for threat modeling implementation 
Most "normal" organizations who aren't very mature in regards to application
security face significant challenges when implementing threat modeling. These
challenges go beyond just technical knowledge—they touch on psychology,
organizational culture, and practical workflow integration.

### The Knowledge Gap and Mindset Shift
Threat modeling is hard for developers for two interconnected reasons: lack of
security expertise and an unfamiliar way of thinking.

Developers typically lack formal security training. They know how to build
secure features when given clear requirements, but identifying threats requires
different expertise. This knowledge gap creates insecurity—people don't like
feeling incompetent at their jobs.

Even more challenging is the mindset shift. Developers spend their days in
constructive mode—building features, solving problems, making things work.
Threat modeling demands the opposite: thinking destructively, like an attacker.
"How could I break this? What if someone tries to abuse this feature?" This
thinking doesn't come naturally and can feel uncomfortable.

This is why methods like RRA use approachable language. Instead of technical
questions like "What could an attacker do here?", RRA guides teams with "I am
worried that..." This simpler framing removes the pressure to think like a
security expert. Combined with predefined threat scenarios tailored to your
organization, it lowers the barrier to getting started.

### Practical Implementation Challenges
Beyond knowledge and mindset, there are practical hurdles:

- **Unclear scope**: It can be challenging to determine what's a realistic
  threat versus far-fetched scenarios
- **Missing practical guidance**: Teams get stuck making decisions during the
  session. Is this data "internal" or "confidential"? Is this threat realistic?
  Should we worry about this? Even when policies exist, if they're hard to use or
  require interpretation, teams waste time debating. Guidance needs to be
  practical, in-context, with clear examples
- **Process overwhelm**: Many threat modeling frameworks feel too heavyweight
  or academic for practical use
- **Time pressure**: Adding another meeting to already packed sprint schedules
  faces resistance
- **Lack of templates**: Without guidance and examples, teams don't know where
  to start
- **Fear of doing it wrong**: The combination of unfamiliarity and importance
  makes teams hesitant to begin

These challenges explain why threat modeling, despite its clear value, remains
difficult to implement successfully. The key to overcoming them is creating a
low-effort, practical process with clear boundaries and supportive tools.

## Choosing a method for your organization
No single threat modeling method suits all organizations. Choose a suitable base
method, then tailor it. If implementing across an organization, consider
allowing teams to adapt it further themselves—even if that means just 15 minutes
discussing threats during planning. Something is better than nothing. Ask
yourself:

- How critical are the services? Do they need extra attention to security?
- How mature is the organization in AppSec?
- What will dev teams prefer? Short and simple, or detailed and thorough?
- Do we have resources to support teams conducting threat modeling?
- What governing documents exist? (threat analysis, data classification, secure
  development procedures, etc.)
- Is there anything in the governing documents that affects the decision?

For most organizations without very critical services or advanced AppSec
maturity, I recommend starting with a low-effort method like Rapid Risk
Assessment (RRA) and tailoring it. Once teams are familiar with threat modeling
thinking, they often welcome more thorough processes.

I know of an organization that first tried implementing STRIDE—a comprehensive
and well-established method. Despite being a solid framework, it didn't get
adopted well enough. The organization then tried again with a simpler,
RRA-based approach, which gained better traction. Sometimes starting with the
"best" method isn't the best strategy; starting with what teams can actually
adopt is.

## Tailoring it to your organization

### Discover needs and pain points
Interview developers, security champions, risk management, and other
stakeholders. Figure out what threat modeling should solve for teams in your
organization, what the pain points are, and what possible gain creators there
are.

_There are many methods to discover needs and pain points. I learned about
"pains" and "gain creators" from the 
[Value Proposition Canvas](https://www.strategyzer.com/library/the-value-proposition-canvas)
framework at university._

### What helps streamline the process
These tools and resources make threat modeling easier to implement:

**Leverage existing governance (if available):**
- Secure development policies or security standards
- Data classification policies
- Threat analyses or risk appetite statements
- The organization's main threat scenarios

Aligning with these helps you design an appropriate process and demonstrate
value to stakeholders.

**Create practical guidance:**
- Process description with guided form
- Data classification examples (employee data = "internal", PHI = "critical") (if you are including a data table or similar)
- Starter set of threat scenarios for your context

**Provide support:**
- Contact person for questions
- Facilitator for teams' first sessions

Governance backing helps with adoption. I've worked with developers who were
rightfully critical of new processes—protecting the organization from time
wasters. Once they saw security requirements mandated this activity, they
shifted from questioning *whether* to do it to questioning *how*. That shift let
me focus on making it useful rather than justifying its existence.

### Create and test your draft
Create a draft that addresses needs and pain points and includes supportive
tools. For an example of what to create, see the [Example implementation](#example-implementation) 
section below, which includes both a process description and guided form.

{{< alert "coffee" >}}
If you're lucky, someone will be especially interested! Write about your work in
your organization's public space and invite feedback.
{{< /alert >}}

Then test with teams:
1. Review the draft with one or more potential users (preferably one who cares 
   a lot about user experience). Repeat until you think it is good enough. 
2. Guide one team through the activity
3. Gather feedback and improve
4. Second round: let someone from the team lead while you observe
5. Repeat with more teams until both you and the organization can stand behind
   the process

## Launching the activity!
Once you have a process the organization can stand behind, plan the launch.

**Build awareness before formal introduction:** Make threat modeling less
foreign by spreading awareness early. Involve security champions throughout
development—bring specific questions to their meetings for discussion, share
progress updates, and invite feedback. When ready, showcase the final process
in a security champion meeting. This gives champions time to mention it
informally in their teams, so the concept isn't completely new when formally
introduced. Champions often provide valuable feedback and can identify
potential adoption barriers you might have missed.

### Consider ownership and support
Who will facilitate threat modeling in each team? This decision significantly
impacts adoption and depends heavily on your organization's culture.

**Key principle:** Assign clear responsibility, typically to security champions
or tech leads. Their job: ensure threat modeling happens—either by facilitating
themselves or delegating it to someone else on the team.

**Important facilitation insight:** The person with the most security knowledge
shouldn't necessarily lead the session. When security experts facilitate, they
often dominate the conversation because they know more about the topic. Better
approach: have someone else lead while the security-knowledgeable person
participates and provides input when needed.

**Common ownership models:**
- **Security champions** ensure it's performed (facilitating or delegating)
- **Tech leads** take responsibility with security champion backup
- **Rotating facilitator** within the team (with clear fallback to champion/lead)
- **Security team support** initially, transitioning to team ownership (works for some cultures)

What matters most: someone owns it, they have support, and the team finds a
facilitation style that works for them.

### Communicate management support
Teams need to see that management stands behind threat modeling. The way you
announce and introduce the activity significantly impacts adoption.

**Have management announce it:** Upper management should introduce threat
modeling, explaining why the organization values it and expects teams to
perform it. The message: this has business value and is part of how we build
software here—not optional extra work (as long as that is correct).

**Show concrete support exists:**
- Time is explicitly allocated in sprint planning
- Teams aren't expected to fit it "in addition to" their regular work
- Training or facilitation support is available
- Management asks about it in reviews (showing it matters)

When teams see that management backs it with time and resources—not just
words—resistance may decrease.

### After launch
**Early wins matter:** Celebrate when teams find real issues or create useful
documentation. This builds momentum.

**Watch for warning signs:**
- Teams consistently skip or rush through it
- Sessions feel like box-checking exercises
- No actionable issues emerge
- Teams report it's confusing or too time-consuming

Do regular check-ins with teams to gather feedback and assess improvements. If
the process hinders more than helps, improve or discard it. A process that teams
resent is worse than no process at all.

## Example implementation
When implementing threat modeling, you'll typically create two main artifacts: a
process description and a guided form.

**Process description** - Documents the "why" and "how":
- Objective: What should threat modeling accomplish?
- Participants: Who should attend?
- Timing: When should teams do this? (e.g., before major releases, quarterly)
- Purpose and scope: Why spend time on this? What's in scope, what's out of
  scope?
- Links to supportive documents (data classification, risk appetite, etc.)

**Guided form** - The actual worksheet teams fill out during sessions.

{{< alert "lightbulb" >}}
**Don't copy this example directly.** Tailor it to your organization's needs and pain points discovered during interviews. What works for one organization may not fit yours.
{{< /alert >}}

**What to include depends on your context:**
- **Service metadata** (owners, classification): Include what helps your
  organization track and prioritize, skip what creates unnecessary overhead
- **Data dictionary**: Useful for understanding impact, but consider skipping it
  if your service inventory already captures this information and is easily
  accessible during the session
- **Data-flow diagram**: Helpful for visualizing flows, but not every method
  requires it. Create it before the session, not during

**Possible key elements for your form:**
- **Service description**: Provides context for everyone
- **Data types processed/stored**: Helps identify what to protect
- **Threat scenarios table**: The core activity—use "I am worried that..." framing
- **Instructions in each section**: Minimizes confusion and effort
- **Links to supporting documents**: Data classification policies, risk appetite statements, etc.
- **Wrapping-up checklist**: Schedule next meeting, create issues, inform stakeholders

The example below includes elements that worked for my implementation. Your
organization might need more, less, or different elements entirely.

<details class="example-form">
<summary>Example threat modeling form<br><small>Tailor to your needs, don't copy directly</small></summary>

### Threat Identification for "Super Cool API"
| | |
|-|-|
| **Service Owner(s)**            | Team Awesome  |
| **Risk Owner**                  | Mr. Risk Dude |
| **Service Data Classification** | Confidential  |
| **Service Description**         | A super cool API for sending automatic emails to employees and customers |
| | |

#### Data-flow Diagram (2 min)
_A **simple** illustration, e.g. a data-flow diagram to easily see how the data
flows. Add this another time, not during the process._

![Data Flow Diagram (DFD)](./illustration.png)

#### Data Dictionary (5-10 min)
<details>
<summary>Instructions</summary>

1. List the data types that are **processed** and/or **stored**. Keep the list short by grouping related data.
2. Classify the data according to your organization's **data classification** policy (or refer to [RRA's data classification](https://www.rra.rocks/docs/data_classification)).
3. Set the `Service Data Classification` value in the table at the top to the highest classification in this dictionary.

Remember to update this list when data types change!
</details>

| Data type | Classification | Comment |
|-|-|-|
| Employee data | Internal     | |
| User data     | Confidential | Includes dietary restrictions |
| User data id  | Internal     | |
| | | |

#### Threat Scenarios (10-30 min)
<details> 
<summary>Instructions</summary>

Find appropriate scenarios affecting **confidentiality**, **integrity** and **availability**. Make sure to keep it realistic, _a nation-state actor targeting your company blog might not be in scope_. Look at the [example scenarios](#example-scenarios) for inspiration. 

1. (10-25 min) Start by filling the **Scenario**, **Driver** and **Assessment** columns
2. When you are done, leave the **Preventive measures** column for now and head to the "Concluding" section 
</details>

| Scenario | Driver | Assessment | Preventive measures |
|----------|--------|------------|---------------------|
| | _A tip: think "I am worried that..."_ | _What's the worst that could happen if controls aren't implemented?_ | _Create an issue for each task. **Do not** find or discuss solutions here (no time!)_ |
| Unauthorized API access allows sending emails as any user | I am worried that since the authorization middleware isn't set by default on every endpoint, we may have forgotten to add it to some endpoints | An attacker could send phishing emails appearing to come from executives, potentially stealing credentials or causing financial fraud | [Jira issue #1337](some.link) |
| Database credential compromise with no accountability | We are using the same database password for all database users, and I don't think we log sufficiently to determine who accessed what | - Our customer data (including dietary restrictions) leaks and we end up in the news<br>- We cannot identify if it was an insider or external attacker<br>- GDPR violation and potential fines | - [Jira issue #4141 (create separate DB users per service)](link)<br>- [Jira issue #4242 (add comprehensive access logs)](link) |
| Service used as spam bot due to lack of rate limiting | I am worried we don't have rate limiting on the email sending endpoints | - Our email service gets blocklisted by major providers<br>- Legitimate emails don't reach customers<br>- Service abuse costs spike | [Jira issue #5001](some.link) |

#### Concluding (3 min) 
Appoint people to:
* Schedule the next meeting. If there wasn't enough time to discuss everything, schedule the next session sooner. Otherwise, plan to reconvene in approximately 3-6 months
* Create issues for scenarios in your issue management system and add their links in the **Preventive measures** column
* Prioritize the issues (or send them to whoever prioritizes)

</details>

### Implementation notes

One of my approaches has been to create a guided document in Confluence, which
worked well. We tried a Markdown version in repositories, but that had more
friction—harder to find previous sessions, version history was clunky.

**Ideal solution:** A web form that auto-fills known data (service name, owners,
*etc.) and manages versioning properly. This requires more development effort
*but significantly improves the experience.

## Reflections and lessons learned
The approach described in this article—creating a process document with a guided
form aligned to an organization's governance system—is based on my experience
implementing threat modeling. While some teams adopted the practice, I've learned
important lessons about the gap between having a process and making it truly
usable.

### What worked well
- **Low-effort framework**: Starting with RRA instead of a heavyweight method
  like STRIDE made adoption much easier, especially in form of simplifying a heavy
  process. Teams doing heavy risk analysis processes were especially interested
- **Guided templates**: Having a structured form with instructions reduced the
  "blank page" problem—teams knew exactly what information to fill in and what
  questions to answer, rather than staring at an empty document wondering where to
  start
- **Organizational alignment**: Connecting threat modeling to existing governance
  documents gave it legitimacy and developer approval among those who often questioned
  initiatives' value in the organization
- **"I am worried that..." framing**: This approachable language helped teams get
  started without feeling like security experts

### What I learned about pain points
Through implementing threat modeling and observing other organizations' attempts,
I've seen recurring challenges that go beyond process design:

- **The learning curve is steep**: One training session isn't enough. Teams need
  time and practice to adopt the "threat scenario" way of thinking. Even with
  supportive documents, teams often need hand-holding initially to understand what
  they are doing and why
- **Security terminology creates distance**: Jargon like "threat actor," "attack
  surface," or "vulnerability" makes the activity feel like security work rather
  than development work. This creates unnecessary psychological barriers
- **Data classification is often impractical**: Many organizations have data
  classification policies, but they're written for compliance officers, not
  developers. When policies aren't accessible or practical during threat modeling,
  teams waste time debating classifications instead of identifying threats 
- **Examples are critical but often undervalued**: Good threat scenario examples
  are what help developers understand what they're even looking for. Without
  effective examples integrated into the process, teams struggle to get started
- **Tooling significantly impacts adoption**: Static documents (Confluence,
  Markdown files) work but create friction. Previous threat scenarios either clutter
  new documents or disappear entirely. Version history helps but doesn't solve the
  fundamental workflow problem. Better tooling—like interactive web forms with proper
  versioning—would reduce friction considerably

### What I'd do differently next time
Reflecting on my implementation approach, here are the strategic choices I'd change:

1. **Invest more time in the discovery phase**: I talked to dev teams and security champions
   before implementing, but I'd now spend more time on structured interviews to
   systematically uncover pain points and preferences before designing anything
2. **Design for true self-sufficiency from day one**: I guided teams through their
   first session, then watched silently the second time (only jumping in when needed),
   but that's still treating symptoms. I'd invest the effort upfront to make the
   process so clear that teams don't need any security involvement at all. Only when
   teams can use it without help is it actually "easy enough." I haven't fully
   cracked this yet, and I'm curious how others have solved it
3. **Study and integrate examples more deeply**: I provided threat scenario examples,
   but I didn't spend enough time examining what makes examples truly effective or
   how to integrate them into the form itself. Better integration could have helped
   teams grasp the threat scenario mindset faster
4. **Advocate harder for proper tooling**: I worked within the constraints of
   Confluence and tried Markdown in Git, but I should have made a stronger business
   case for an interactive web form. The tooling choice significantly impacts
   adoption, and I underestimated that
5. **Build succession planning into the launch**: I didn't establish clear ownership
   for maintenance after my involvement ended. Someone needs to own iterating on
   the process based on ongoing feedback, or it stagnates

Thinking about threat modeling through the lens of pain points and pain relievers
(like the Value Proposition Canvas) has been eye-opening. Having a process isn't
enough—it needs to genuinely make the activity easier, not just possible. And not
even just easier, but easy enough. As I often say: 

> The activity must provide value. If it doesn't provide value, it must be improved or removed.

This is still a learning journey. If you're implementing threat modeling in your
organization, I encourage you to iterate, listen to your teams, and continuously
refine your approach based on what actually helps them succeed.

## Trainings for threat modeling
There is a [Microsoft Learn path](https://learn.microsoft.com/en-us/training/paths/tm-threat-modeling-fundamentals/) 
for threat modeling. The training explains what threat modeling is and how to
draw data-flow diagrams. They have separated the workflow in four stages: 
design, break, fix, and verify, with respective questions about the service that
are helpful for understanding the system and identifying threats. 

[Secure Code Warrior](https://www.securecodewarrior.com/), a training platform 
for secure coding, has a threat modeling course that might be worth exploring.

Some colleagues of mine participated in a training at the [Global AppSec conference](https://owasp.org/events/).

## Resources
- [rra.rocks](https://www.rra.rocks/)
- [Microsoft Learn path](https://learn.microsoft.com/en-us/training/paths/tm-threat-modeling-fundamentals/)
- [Mozilla's Rapid Risk Assessment (RRA)](https://infosec.mozilla.org/guidelines/risk/rapid_risk_assessment.html)

