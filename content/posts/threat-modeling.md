+++
title = 'Threat Modeling'
date = 2024-11-15T17:40:58+01:00
draft = true
tags = ["AppSec", "Threat modeling", "Rapid Risk Assessment (RRA)", "STRIDE"]
+++

There are many activities that may be launched to improve application security
in an organization. Threat modeling is one of them. Others are CI pipelines with
security checks, or a vulnerability management system, creating standards or
maybe development guides. There are so many activitites! 

Threat modeling might be one of the most important activities in application
security. Still, a lot of organizations struggle to implement it and make it
valuable enough for development teams. I have done a lot of thinking and some
research to figure out why this is, and I have my hypothesis' on this. However,
I won't dive into that. Anyways, make sure to do your homework and ensure that
you deliver something valuable in the first version. It is paramount to ask
developers for their needs, their pain points, how it can be valuable for them
to make this activity stick.

<details>
<summary>Are you an AppSec personell, or a security architect or anyone that has
been appointed to implement activities to dev teams? Open the drop-down and read
this ^_^ </summary>

I want to emphasize one important thing, a misinterpretation I've seen and that
I think is _dangerous_ for the important work of making AppSec a more common
practice among development. If you are a person who is hired to improve
application security at a company, you might think that the dev teams aren't
delivering secure enough services (or in other words - not good enough
services). No matter how true that is, do **not** say that when approaching the
dev teams. Tell them that you see how **skilled** they are and that you are 
**excited to work with them**, that you will need **their help** to create
something that works for them. Then ask them what they need help with or how
they think things could be done differently to improve the process of delivering
services with security in mind.

Honestly, just think about it. Your expertise is security. The development
landscape is shifting all the time and new development process methods,
technology and expectations from developers changes all the time. Developers
expected expertise is most likely so much wider than yours, and here you come
adding to their workload. Not only are they expected to know their programming
language, they have to get domain knowledge, know pipelines, ways of rolling out
their services, cloud hosting, several kinds of testing, architecture, all
should be good quality, they are already behind on technical debt due to
prioritization, _and now they must at a deeper level consider_... security!! 

Your security work usually isn't as business critical as what the dev teams
already are doing. They are creating what gives direct value to the company.
You are there to assist them, not to save the world from their mistakes. You are
definitely not there to tell devs that they suck in security by saying that they
aren't good enough at the moment and make it seem like you are the hero assigned
by the company to save the world and fix their mistakes. I am 100% sure that the
company did not hire you to do that. _(Yes, I've experienced this and after
joining tons of security meetups it seems like this additude is a little more
common than it should be.)_

It is OK to think you are a hero. I think you are! Doing this amazing work to 
protect users and organizations. Just keep it to yourself and make the
developers feel like heros. And yes, imo they are too!

In other words, your job is to help dev teams get the proper tooling to deliver
more predictable and secure enough services. Tooling that should already have
been there for the teams from the get-go. Tools that are necessary for dev teams
to do what the job they are expected to do. Tools that are quick, easy, takes as
little effort as possible. Preferably that removes workload. Find their pain
points and use your expertise to figure out what tools the organization is
missing that would be helpful. If that is your approach, they are more likely to
welcome you.  Show them that you are there for them, not for management to tick
of some compliance checks.

</details>

## What is threat modeling?
This is a question I've gotten a lot. In the beginning I wasn't really able to 
answer it. It is not very common and security people working in consulting seems
to struggle to understand the value in it. Which is understandable, since it 
remains an activity that we don't see in most organizations (yet!).

I like to say that threat modelling is like a fun risk analysis - apologizing to
everyone who likes risk analysis - where you take out all the boring stuff and
rather spend time on the fun part!

It is also a valuable for newcomers in teams to get to understand the service
they are working with, as it provides a good overview of how the service works,
where the data flows, and how critical the service is.

Let's quote [Microsoft](https://learn.microsoft.com/en-us/training/paths/tm-threat-modeling-fundamentals/): 
> Threat modeling is an effective way to help secure your systems, applications, networks, and services. It's an engineering technique that identifies potential threats and offers recommendations to help reduce risk and meet security objectives earlier in the development lifecycle.

Threat modeling can be done both at system level and service level.

I'll leave it to that, and then you can google a bit if you want to learn more
about it in depth and move on to different threat modeling methods out there.

## Threat modeling methods 
There is no such thing as one method that rules all. What's paramount for this
activity to work, is to tailor it to the organization and allow teams to do it
in a way that works for them. __So that it gives value.__ It is better to use 15
minutes to discuss threats during planning than doing nothing. Although, I would
advice to use a little more thorough process.

If you are an AppSec admin, please note that if the process is more in the way
than the teams thinks it gives value, then it should be improved! Figure out the
pain points and relieve them, check in with the teams and see if it got better.
Do it in iterations and then you should find something that works for your
organization. But, please, do your homework, gather information and put in
effort to create a process you believe is valuable when you launch it. By
launching a new activity, you are already making team's development process more
involved and stressful. Try not to lose their faith straight away. 

I have previously worked with [Rapid Risk Assessment (RRA)](https://infosec.mozilla.org/guidelines/risk/rapid_risk_assessment.html) 
and implemented a tailored version of RRA for an organization. What I have not
done is to implement the typical threat modeling method
[STRIDE](https://owasp.org/www-community/Threat_Modeling_Process#stride). 
The way of thinking during the process is the same for both methods. The goal is
the same as well. The difference is how detailed the road to get there is. Also,
how much effort is needed to get started with it. Microsoft also has their own
tailored version of STRIDE.

{{< alert "lightbulb" >}}
Rapid Risk Assessment is very well explained with examples in [this video](https://www.rra.rocks/docs/podcasts).
{{< /alert >}}

What is common for all the methods is that they do these stages:
1. Gather information about the service and draw a data-flow diagram
2. Use the diagram and knowledge of the service to find threats against the service
3. Implement the mitigations from previous step (and verify it)

{{< alert "comment" >}}
I believe that most organizations aren't mature enough for involved threat
modeling processes. To begin with, the process should to be low-effort and short
in order to be valuable enough for dev teams in less mature organizations. When
teams are familiar to the threat modeling way of thinking, they often welcome
more involved processes and some suggest doing more thorough processes
themselves.
{{< /alert >}}

To make the process low-effort, every organization should create tools
supporting threat modeling, such as
- a process description for the threat modeling process, accompanied with a guided form to perfom the process
- a data classification table with examples, e.g. employee data is categorized as "internal" 
- a minimum set of threat scenarios that the organization's services should cover _(this is the most valuable for orgs where teams aren't used to threat modeling)_
- a threat analysis, who are likely to attack? 
- a description of the organization's risk appetite, how much risk is the organization willing to take?
- a person to contact for help or clarifications
- a person to introduce the process the first time(s)

Honestly, I don't expect all devs to read these documents. And I don't think 
anyone should expect that. However, some times teams come to a point where they
get uncertain. Uncertainties is a big no-no and should be eliminated! Let's say
a team is stuck discussing whether a threat scenario is in scope for the company
or not. Or, whether their application should be classified as internal or
confidential because a piece of data they don't quite know how to classify.
Instead of having teams discussing without really knowing, which in turn can
give the process negative vibes, give them the tools to look it up. 

To conclude this section about what method to chose, I've so far always adviced
to start with a short and sweet method such as the Rapid Risk Assessment and
tailor it to suit the organization.

## Data-flow diagrams
Most threat model activities include a data-flow diagram. It is a helpful
drawing of where the data flows in the service you are creating a threat model
for. The drawing often includes trust zones and has different shapes to
represent processes, data stores, external entities, data-flow and trust
boundaries. 

The [Microsoft Learn path](https://learn.microsoft.com/en-us/training/paths/tm-threat-modeling-fundamentals/)
is nice to learn more about the diagrams. It also brings up what depth of detail
the drawing should have, by listing four depth layers. According to them, and I
agree, most data-flow diagrams should contains both layers 0 and 1. They also 
point out that you should reach out to your security team in case you are
unsure. They describe the depth layers 0 and 1 as follows:

| Depth layer | Title   | Description |
|-------------|---------|-------------|
| 0           | System  | Starting point for any system. Data-flow diagram contains major system parts with enough context to help you understand how they work and interact with each other. |
| 1           | Process | Focus on data-flow diagrams for each part of the system by using other data-flow diagrams. Use this layer for every system, especially if it handles sensitive data. The context at this layer helps you identify threats and ways to reduce or eliminate risks more efficiently. |

To see all four depth layers, check out the Learning path's first module
"Introduction to threat modeling", at [Step 1 - Design](https://learn.microsoft.com/en-us/training/modules/tm-introduction-to-threat-modeling/2-step-1-design-phase)
in the section "Diagram layers".

{{< alert "lightbulb" >}}
__Focus on making it easy to
understand how the data flows to see where we need to ensure protection.__ Keep
the diagram simple, don't add to many parts to it. At least for short and sweet
threat modeling processes, the goal should be that by looking at it for a
minute, you should have control of where the data flows.
{{< /alert >}}

I've usually used [draw.io](https://draw.io) to create these diagrams. Microsoft
suggests their own tools [Visio](https://www.microsoft.com/en-US/microsoft-365/visio/flowchart-software) 
and [Threat Modeling Tool](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool).

## Process description
I think two documents are necessary when implementing threat modelling across an
organization. First, the process description, and then the guided form that
teams fill when performing the process. In this section, I will summarize what
I've learned from creating such a description. 



## An example of a simple threat modeling process
In order to go in further details on possible ways to tailor your threat
modeling process, I would like to bring up an example process and walk us
through it. 

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
| An attacker sends phishing emails | I am worried that sinc the authorization middleware isn't set by default on every endpoint, we have forgotten to do it on some endpoints | The company looses tons of money because the CFO was phished during through our service | [Jira issue #1337](some.link) |
| An attacker leaks the database, and we aren't be able to find out how | We are using the same database password for all database users. Also, I do not think we log sufficiently to say for sure where the attack origined | - Our customer data leaks and we end up in the news<br>- We aren't able to find an insider who did it due to lack of logging | - [Jira issue #4141 (create several DB users)](link)<br>- [Jira issue #4242 (add accountability logs)](link) |

#### Concluding (3 min) 
Appoint people to:
* Schedule the next meeting. Was there too little time to discuss? Don't wait to long until next time. Otherwise, it can be a good idea to schedule it in approximately 3-6 months
* Create issues for scenarios in your issue management system and add their link in the **Preventive measures** column
* Prioritise the issues (or send them to whomever prioritizes)

---
</details>

What I like to do is to create a guided document that teams fill during the
process. At the top of this document, I put a **table containing high level
information** about the service, such as the service owner(s), risk owner, service
data classification, as well as a short description of the service.  In some
cases, I like to add the business criticality of the service, if the
organization has some business critical services that are not processing critical
data.  These pieces of information already says a lot about the service, both for
the team and for others that might need information about the service.

Then comes the **data-flow diagram**, used to illustrate how the data flows
through the service. The data is what we want to protect, so this diagram shows
what points of the service might need more protections. These are spots that you
can focus more on when utilizing the diagram later. 

Next, I like to add a **table of data types** that the service is process or
store, and their data classification. To make it short and sweet, I prefer
grouping data. For instance, instead of adding every data point for a user,
write "user data" and the highest classification for the user data. 

{{< alert "lightbulb" >}}
When describing how to fill in the data types table, it is appropriate to add a
link to the data classification document
{{< /alert >}}

Then comes the **threat modeling** part of this activity, finding potential
security issues in the service. In the Microsoft Learn path, two different ways
of thinking are brought up, focusing on **protecting the service** or
**understanding the attacker**. I like this approach, however I experience that
most dev teams like to focus on protecting the system. Microsoft also states
that _"Microsoft product engineers mostly focus on protecting the system.
Penetration testing teams focus on both."_ (Wonder if an offensive security
person has been working on this process at Microsoft:) ).

When coming up with scenarios, I like add a scenario description, the driver 
for the scenario (whats the reason behind bringing it up) and thinking about how
bad it might get. The latter to get an understanding of how important it is to
fix the issue. For the scenarios, thinking "I am worried that..." works like a
charm for many people. 

This part of the threat modeling process is probably the place you want to sit
down and think very hard on what method is best for your teams. Do you want to 
go with STRIDE? Do you want to go with the [RRA way](https://www.rra.rocks/docs/threat_scenarios),
or the Marit (me) way? Maybe a mix between all of them? In my opinion it is all
about finding the easiest approach for the teams is that still provides value.
Just remember that to begin with, the teams needs to learn the way of thinking.
Get a little warmed up. Then they are more likely to want to move on to a more
thorough method.

{{< alert "lightbulb" >}}
I recommend providing a list of examples scenarios or your organization's base
scenarios. This way, it is easier to get the threat modeling way of thinking
started. 
{{< /alert >}}

When you have a list of scenarios, the process is almost done. Then wrapping up 
the process can consist of activities such as: 
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

## Resources
- [rra.rocks](https://www.rra.rocks/)
- [Microsoft Learn path](https://learn.microsoft.com/en-us/training/paths/tm-threat-modeling-fundamentals/)
- [Mozilla's Rapid Risk Assessment (RRA)](https://infosec.mozilla.org/guidelines/risk/rapid_risk_assessment.html)
