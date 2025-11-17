+++
title = 'Threat Modeling with Rapid Risk Assessment - a low-effort and concise approach'
date = 2024-11-15T17:40:58+01:00
draft = false
tags = ["AppSec", "Threat modeling", "Rapid Risk Assessment (RRA)", "STRIDE"]
+++

Organizations can undertake various activities to enhance application security,
with threat modeling being one of the most impactful activities in the early
stages of development. Other similar activities are risk analysis or security
architecture review, and some times mistaken as vulnerability analysis. 

Even though threat modeling is a valuable activity, a lot of organizations
struggle to implement it and make it good enough for development teams to
perform it reglarly.

In this article, I will share my own thoughts and experiences on implementing
the threat modeling activity in organizations. Although the scope is
organization wide, this article should be useful for implementation in just one
team as well.

## Benefits of performing threat modeling regularly
First, I would like to list the main benefits of the threat modeling activity.

- Gain understanding of the attack points of your application
- Avoid possible reimplementation if threat modeling is performed before
  implementation
- Find architectural mistakes or improvements if threat modeling is performed
  after implementation
- Valuable documentation for both newcomers to the team, and management, the
  data protection department or other entities that might want to understand
  your system quickly, such as pentesters. (Documentation showing among other
  things where the data flows, how important the data is, and a simple
  architecture drawing)

## What is threat modeling?
Threat modeling is a proactive approach to identifying and mitigating potential
security threats in applications.

I often describe threat modeling as a more engaging form of risk analysis,
focusing on the aspects that are more fun and valuable to developers, while
streamlining the process.

Let's look at how [Microsoft](https://learn.microsoft.com/en-us/training/paths/tm-threat-modeling-fundamentals/)
defines it: 
> Threat modeling is an effective way to help secure your systems, applications, networks, and services. It's an engineering technique that identifies potential threats and offers recommendations to help reduce risk and meet security objectives earlier in the development lifecycle.

For those interested in a deeper dive, I recommend starting with Rapid Risk
Assessment (RRA) and STRIDE. RRA is a short and sweet method and STRIDE is the
most known method. 

## Threat modeling methods 
I have previously worked with [Rapid Risk Assessment
(RRA)](https://infosec.mozilla.org/guidelines/risk/rapid_risk_assessment.html)
and implemented a tailored version for an organization. What I have not done is
to implement the typical threat modeling method
[STRIDE](https://owasp.org/www-community/Threat_Modeling_Process#stride). The
way of thinking during the process is the same for both methods. The goal is
the same as well. The difference is the road getting there. Both for the
process itself, and the effort needed to get started with it. Microsoft has
their own tailored version of STRIDE.

Examples of other frameworks are Attack Trees, PASTA, DREAD and CVSS.

These stages are common for all the methods:
1. Gather information about the service and draw a data-flow diagram
2. Use the diagram and knowledge of the service to find threats against the
   service
3. Implement the mitigations from previous step (and verify it)

{{< alert "lightbulb" >}}
Rapid Risk Assessment is well explained with examples in [this video](https://www.rra.rocks/docs/podcasts).
{{< /alert >}}

## Choosing and tailoring threat modeling for your organization/team
No single threat modeling method suits all organizations and all of their
teams. Tailoring the approach is key to effective threat modeling. Also, if
implementing threat modeling for an entire organization, consider allowing
teams to tailor it further themselves, enabling them to adopt methods that
provide **real value**. Even if their way is using 15 minutes to discuss
threats during planning, it is better than nothing. Depending on the risk level
of the system, it might just be sufficient (usally not though!).

When setting up threat modeling for the first time, start by asking yourself:

- How critical are the services? Do they need extra attention to security?
- How mature is my organization? Is it naturally ready to use a more detailed
  framework?
- What will the dev teams prefer? A short and concise, or a longer and detailed
  framework?
- Do we have dedicated resources to plan and help teams conducting threat
  modeling activities?
- What governing documents support or can be integrated to the process
  description? E.g. A threat analysis showing who expected attackers are,
  Secure Development Procedure describing the process for threat modeling, etc.

If you're dealing with military grade systems that needs tons of security, then
maybe it is better to go for a detailed framework. Then you have proper
reasoning, and most likely have enough budget to make the process painless by
assigning dedicated resources to both continuously improve the activity and
help teams out when conducting it. 

As long as the organization doesn't have very critical services that require
extra attention to security, or the organization isn't a very mature one in
regards to AppSec, I recommend starting with a short and concise, low-effort,
method like Rapid Risk Assessment (RRA) and tailor it to the organization/team.
When teams are familiar to the threat modeling way of thinking, they often
welcome more involved processes and some suggest doing more thorough processes
themselves. Rapid Risk Assessment is the one we continue with in this article.

Next, ask for the pain points, and how to relieve them. Do developers think
threat modeling seems useful? If not, why? What makes threat modeling hard for
development teams in the organization? 

{{< alert "lightbulb" >}}
If you are lucky, there will be some especially interested with many opinions! 
Write a post about your work in the company public place and let everyone know 
that you are open for feedback and suggestions.
{{< /alert >}}

After doing you research, create a draft that avoids as many pain points as
possible and includes supportive tools (see section [Supportive tools to
streamline the process](#supportive-tools-to-streamline-the-process)). Go to
section [Process description](#process-description) and [Guided form](#guided-form---the-process-itself)
to see an example of a tailored version of Rapid Risk Assessment.

Get feedback continuously and ask whenever new questions pops up. Ask one or
more of the people that has gotten involved so far for feedback. Then you
should be ready for test runs! Maybe guide one team through the activity the
first time. Implement the feedback you gathered and do another test round.
This time, make one person in the team lead the meeting. Tag along, but be a
fly on the wall. That way, it will be clearer which parts needs improvement.

Repeat the feedback process with a subset of teams or developers until you are
confident in your delivery, and both you and the organization can stand behind
the activity.

## Launching the activity!
By launching a new activity, you are making team's development process more
involved and stressful. Especially in regards to threat modeling. With a
process the organization can stand behind, it is time to plan the launch!

There are many possible approaches. Consider assigning a responsible in each
team, or create a process to ensure the team feels enough ownership to perform
the activity. If you have security champions, maybe they should get training
and guide their teams? Maybe during the launch phase, you should watch and
assist? Should you guide them the first time? Is it natural that the tech leads
gets this training and they have backing from security champions in form of
providing more security knowledge? Figure out what suits the organization
best.

I recommend the upper management to make an announcement about the activity to
show that this is what the organization wants and that development teams are
expected to perform the activity. Showing that this is not some charity work
that teams can do because some security people told them. It actually has value
to the business. Upper management communication including "why" also helps if
there is general resistance for implementing new activities. 

After launching the activity, do regular check-ins with teams to assess
improvements and gather feedback. As an AppSec admin, remember that if the
process hinders more than helps, it's time for improvement or discarding the
process. 

{{< alert "coffee" >}}
Do dev teams have a way to provide feedback on the activities in your
organization?
{{< /alert >}}

## Supportive tools to streamline the process
To streamline the process and make it low-effort, organizations should develop
supportive tools for threat modeling, such as

- a process description for the threat modeling process, accompanied with a
  guided form to perform the process
- a data classification table with examples, e.g. employee data is classified
  as "internal", while Protected Health Information (PHI) is classified as
  "critical"
- a minimum set of threat scenarios that the organization's services should
  think through _(this very useful for orgs where teams aren't used to threat
  modeling)_
- a threat analysis, who are likely to attack? This document might make it
  easier to understand why to defend the system
- a description of the organization's risk appetite, how much risk is the
  organization willing to take?
- a designated contact person for assistance and clarifications
- a person to introduce the process the first time(s)

I don't expect all developers to read these documents. And I don't think anyone
should expect that. However, experience show that some prefer having proper
documentation with proper reasoning and boundaries.

I have experienced teams stuck discussing whether a threat scenario is is
scope for their company, and also discussing whether a data type is "internal"
or "protected". Both times, they seemed annoyed and I ended 
be classified as internal or confidential. Instead of having this uncertainty,
define and document the boundaries. With well-made processes and boundaries, 
negative vibes are avoided 

## Data-flow diagrams
Data-flow diagrams are integral to most threat modeling activities. It is a
helpful drawing of where the data flows in the service you are creating a threat
model for. The drawing often includes trust zones and has different shapes to
represent processes, data stores, external entities, data-flow and trust
boundaries. 

The [Microsoft Learn path](https://learn.microsoft.com/en-us/training/paths/tm-threat-modeling-fundamentals/)
is nice to learn more about the diagrams. Microsoft outlines four depth layers
for data-flow diagrams, and I concur that most data-flow diagrams should include
both layers 0 and 1:

| Depth layer | Title   | Description |
|-------------|---------|-------------|
| 0           | System  | Starting point for any system. Data-flow diagram contains major system parts with enough context to help you understand how they work and interact with each other. |
| 1           | Process | Focus on data-flow diagrams for each part of the system by using other data-flow diagrams. Use this layer for every system, especially if it handles sensitive data. The context at this layer helps you identify threats and ways to reduce or eliminate risks more efficiently. |

All four depth layers can be found in Learning path's first module
"Introduction to threat modeling", at [Step 1 - Design](https://learn.microsoft.com/en-us/training/modules/tm-introduction-to-threat-modeling/2-step-1-design-phase)
in the section "Diagram layers".

{{< alert "lightbulb" >}}
__Focus on making it easy to understand how the data flows to see where we need
to ensure protection.__ Keep the diagram simple; avoid overcrowding it with
unnecessary details. For short and sweet threat modeling processes, the goal
should be that by looking at it for a minute, you should have control of where
the data flows.
{{< /alert >}}

I've usually used [draw.io](https://draw.io) to create these diagrams. Microsoft
suggests their own tools [Visio](https://www.microsoft.com/en-US/microsoft-365/visio/flowchart-software) 
and [Threat Modeling Tool](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool).

## Process description
In addition to a guided form that dev teams use when conducting the threat 
modeling activity (see section [Guided form - The process itself](#guided-form---the-process-itself)), 
I believe a process description is useful. 

A good process description describes what the objective is, who the
participants should and when should the activity be performed. As well as the
purpose and scope. Why should the teams use their time on this? What should be
focused on, and what should teams refrain from using time on? 

Other supportive documents are also suitable here.

## Guided form - The process itself 
In order to go in further details on possible ways to tailor your threat
modeling process, I would like to bring up an example process and walk us
through it. It is based on RRA.

<details>
<summary>Example threat modeling form</summary>

_My example is heavily based of Mozilla's RRA, but modified to how I
like it. You can use it as inspiration tailoring the process to your org._

### Threat modeling for "Super cool API"
| | |
|-|-|
| **Service owner(s)**            | Team Awesome  |
| **Risk owner**                  | Mr. Risk Dude |
| **Service data classification** | Confidential  |
| **Service description**         | A super cool API for sending automatic emails to employees and customers |
| | |

#### Data-flow diagram (2 min)
_A **simple** illustration, e.g. a data-flow diagram to easily see how the data
flows. Add this another time, not during the process._

![Data Flow Diagram (DFD)](./illustration.png)

#### Data dictionary (5-10 min)
<details>
<summary>Instructions</summary>

1. List the data types that are **processed** and/or **stored**. Make the list short and by grouping data.
2. Classify the data according to the organization's **data classification** policy (or take a look at [RRA's data classification](https://www.rra.rocks/docs/data_classification)).
3. Set the `Service Data Classification` value in the table at the top of this document to the highest classification in this data dictionary. 

Any changes? Remember to update the list!
</details>

| Data type | Classification | Comment |
|-|-|-|
| Employee data | Internal     | |
| User data     | Confidential | Includes dietary restrictions |
| User data id  | Internal     | |
| | | |

#### Threat scenarios (10-30 min)
<details> 
<summary>Instructions</summary>

Find appropriate scenarios affecting **confidentiality**, **integrity** and **availability**. Make sure to keep it realistic, _an EMP bomb might not be in scope_. Look at the [example scenarios](#example-scenarios) for inspiration. 

1. (10-25 min) Start by filling the **Scenario**, **Driver** and **Assessment** columns
2. When you are done, leave the **Preventive measures** column for now and head to the "Concluding" section 
</details>

| Scenario | Driver | Assessment | Preventive measures |
|----------|--------|------------|---------------------|
| | _An advice is to think "I am worried that..."_ | _What's the worst that may happen if controls aren't implemented?_ | _Create an issue for each task, **do not** find or discuss solutions here (no time!)_ | |
| An attacker sends phishing emails | I am worried that since the authorization middleware isn't set by default on every endpoint, we have forgotten to do it on some endpoints | The company looses tons of money because the CFO was phished during through our service | [Jira issue #1337](some.link) |
| An attacker leaks the database, and we aren't be able to find out how | We are using the same database password for all database users. Also, I do not think we log sufficiently to say for sure where the attack originated | - Our customer data leaks and we end up in the news<br>- We aren't able to find an insider who did it due to lack of logging | - [Jira issue #4141 (create several DB users)](link)<br>- [Jira issue #4242 (add accountability logs)](link) |

#### Concluding (3 min) 
Appoint people to:
* Schedule the next meeting. Was there too little time to discuss? Don't wait to long until next time. Otherwise, it can be a good idea to schedule it in approximately 3-6 months
* Create issues for scenarios in your issue management system and add their link in the **Preventive measures** column
* Prioritize the issues (or send them to whomever prioritizes)

---
</details>

One of my approaches has been to create a guided document that teams fill
during the process, like the example above. This was done in Confluence, which
worked well. We tried with a Markdown version in the repositories too, but that
did not work as well.

{{< alert "lightbulb" >}}
I think the best process is using a web form, but that requires a more time and
resources. Even better if the form auto-fills whatever data is already known to the
organization. For instance the service name, service owner, risk owner, etc.
{{< /alert >}}

In the example, I have included information that is useful for thinking about
threat scenarios and keeping it minimal, such as the service description, data
types that is processed by the service (what to protect) and the data-flow
diagram showing where the data flows (where to protect). Also, I think every
section should include instructions to minimize confusion and effort.

{{< alert "lightbulb" >}}
When describing the process, consider linking to supporting documents. E.g.
the internal data classification.
{{< /alert >}}

The **threat modeling** part of this activity might be the most difficult,
especially in the beginning. This is when finding potential security issues in
the service. In the Microsoft Learn path, two different ways of thinking are
brought up, focusing on **protecting the service** or **understanding the
attacker**. I like this approach, however I experience that most dev teams like
to focus on protecting the system. Microsoft also states that _"Microsoft
product engineers mostly focus on protecting the system. Penetration testing
teams focus on both."_ (Wonder if an offensive security person has been working
on this process at Microsoft:) ).

When coming up with scenarios, I like add a scenario description, the driver 
for the scenario (whats the reason behind bringing it up) and thinking about how
bad it might get. The latter to get an understanding of how important it is to
fix the issue. For the scenarios, thinking "I am worried that..." works like a
charm for many people. 

{{< alert "lightbulb" >}}
I recommend providing a list of examples scenarios or your organization's base
scenarios. This way, it is easier to get the threat modeling way of thinking
started. 
{{< /alert >}}

When wrapping up, the process can consist of activities such as: 
- Setting up a new meeting (it is easy to forget)
- Create issues for all scenarios to start doing research on finding security controls and fix the threats
- Inform the risk owner about the scenarios, it might be useful for risk analysis
- Inform the project manager (or whomever prioritizes issues) about the new issues

## Trainings for threat modeling
There is a [Microsoft Learn path](https://learn.microsoft.com/en-us/training/paths/tm-threat-modeling-fundamentals/) 
for threat modeling. The training explains what threat modeling is and how to
draw data-flow diagrams. They have separated the workflow in four stages, 
design, break, fix and verify, with respective questions about the service that
is great to get 

[Secure Code Warrior](https://www.securecodewarrior.com/), a training platform 
for secure coding has a threat modeling course. I am about to test it soon.

Some colleagues of mine participated in a training at the [Global AppSec conference](https://owasp.org/events/).

## Example scenarios
[maybe coming soon ^_^]

## Resources
- [rra.rocks](https://www.rra.rocks/)
- [Microsoft Learn path](https://learn.microsoft.com/en-us/training/paths/tm-threat-modeling-fundamentals/)
- [Mozilla's Rapid Risk Assessment (RRA)](https://infosec.mozilla.org/guidelines/risk/rapid_risk_assessment.html)

