+++
title = 'Why Mixing CRA Criticality Levels with Internal Risk Classifications Is a Bad Idea'
date = 2025-11-02T08:46:34+01:00
draft = false 
slug = "trivy-pipeline"
tags = ["AppSec", "CRA", "Governance"]
categories = ["appsec"]
+++

**TL;DR:** The EU Cyber Resilience Act (CRA) and your internal security risk
levels answer completely different questions. Don't merge them. Keep them
separate, track them separately, and apply them together when needed.

---

## Let's Talk About Two Different Questions

When you're building software in your organization, you're probably asking:

**"How much security effort should we put into this system?"**

This is your **internal risk classification** question. It's about operational
security, risk management, and how you allocate resources. Maybe you follow ISO
27001, maybe you have your own framework, but the principle is the same: assess
the risk, apply appropriate controls.

But if you're **selling software products in the EU**, there's a completely
different question:

**"What legal compliance obligations apply when we put this product on the
market?"**

This is what the **EU Cyber Resilience Act (CRA)** addresses. It's not about
*your* risk assessment—it's about *legal obligations* for products with digital
elements sold in the EU market.

**These are fundamentally different questions.** And that's why mixing the
answers is a mistake.

---

## What Is the CRA, Anyway?

The **Cyber Resilience Act** is EU regulation coming into force in 2024/2025
(phased implementation through 2027). Think of it as the "CE marking" regime for
software—similar to how physical products need safety certifications, software
products now need cybersecurity compliance.

**The CRA applies to:**
- ✅ Products with digital elements **sold, licensed, or distributed commercially** in the EU
- ✅ SaaS, IoT devices, software packages, embedded systems
- ✅ Free products if they're part of a commercial activity

**The CRA does NOT apply to:**
- ❌ Software developed exclusively for internal use within your organization
- ❌ Pure open-source projects (with no commercial entity behind them)
- ❌ Research prototypes not yet released to market

**Key CRA requirements include:**
- Secure by design and by default
- Software Bill of Materials (SBOM)
- Vulnerability handling (24-hour disclosure for exploited vulnerabilities)
- Security updates for at least 5 years
- CE marking and EU Declaration of Conformity
- Technical documentation
- Much more depending on your product category

**Why it matters:**
- **Penalties:** Up to €15M or 2.5% of global annual turnover for non-compliance
- **Market access:** Can't sell in EU without compliance
- **Supply chain:** Your customers may require CRA compliance

---

## Internal Risk Classification: A Different Beast

Most organizations classify their systems based on **internal risk**—regardless
of whether they sell anything. This is good practice. Even if you're not
pursuing ISO 27001 certification, the principles make sense:

**Internal risk classification typically considers:**
- **Data sensitivity:** What data does this system handle? (PII, PHI, trade
  secrets, financial data?)
- **Business impact:** What happens if this system is compromised? (Operational
  disruption, reputation damage, legal liability?)
- **Access patterns:** Who uses it? (Internal only, partners, public?)
- **Regulatory requirements:** Any compliance obligations? (GDPR, HIPAA,
  financial regulations?)

**Example internal classification scheme:**
- **Critical:** Clinical trial system (handles PHI, regulatory compliance,
  patient safety)
- **High:** Customer-facing portal (handles PII, payment data, brand reputation)
- **Medium:** Internal analytics tool (handles confidential business data)
- **Low:** Internal wiki (handles general business information)

**The goal:** Apply security controls proportionate to risk. Critical systems
get MFA, encryption, penetration testing, 24/7 monitoring. Low-risk systems get
basic hygiene.

---

## Scope Comparison: CRA vs. Internal Classification

Here's where the confusion starts—and why mixing them is problematic:

### Internal Classification Scope

**Applies to:** **Every system built by your organization** that processes
anything of value

This includes:
- Internal tools (HR systems, wikis, dashboards)
- Research prototypes
- Data processing pipelines
- CI/CD infrastructure
- Partner collaboration platforms
- Customer-facing products
- **Everything** your employees build

**Typical coverage:** 100% of your software portfolio

---

### CRA Scope

**Applies to:** **Products with digital elements sold/distributed commercially
in the EU**

This includes:
- Commercial software products
- SaaS offerings
- IoT devices with software
- Embedded systems
- Mobile apps in app stores (if sold/licensed)
- Open-source software **if** maintained by a commercial entity for commercial
  purposes

This **excludes:**
- Internal tools (not products)
- Research software (not yet on market)
- Pure collaboration (not distribution)
- Non-commercial open source
- Services (consulting, training) without distributing software

**Typical coverage for most organizations:** 5-20% of your software portfolio

---

### Visual Comparison

| Feature/Aspect            | Internal Risk Classification                  | CRA Compliance                               |
|---------------------------|----------------------------------------------|---------------------------------------------|
| **Purpose**              | Operational security, risk management        | Legal compliance for EU market              |
| **Scope**                | All systems processing data of value         | Products with digital elements sold in the EU|
| **Applies to**           | Internal tools, customer-facing products, etc.| Commercial software, SaaS, IoT devices, etc.|
| **Excludes**             | N/A                                          | Internal tools, research software, non-commercial open source |
| **Typical Coverage**     | 100% of software portfolio                   | 5-20% of software portfolio                 |
| **Key Requirements**     | Varies by organization                       | Secure by design, SBOM, vulnerability handling, security updates, CE marking, technical documentation |
| **Penalties for Non-compliance** | Varies by regulation/framework (e.g., GDPR, HIPAA) | Up to €15M or 2.5% of global annual turnover |
| **Market Access Impact** | N/A                                          | Cannot sell in EU without compliance        |
| **Supply Chain Impact**  | N/A                                          | Customers may require CRA compliance        |

---

## Why Mixing Them Causes Trouble

1. **Different Goals:** One aims to protect your organization, the other to meet legal requirements.
2. **Varied Scopes:** Internal classifications cover all systems, CRA only applies to products sold in the EU.
3. **Diverse Requirements:** Security controls vs. legal compliance measures.
4. **Inconsistent Penalties:** Varying consequences for non-compliance.

---

## Why You Can't Map Them 1:1

**Key insight:** CRA is a **subset** of your internal scope. But the
classification criteria are **completely different**.

Let's look at some real-world examples:

### Example 1: Internal Clinical Trial System

**Internal Classification:** Critical
- Handles PHI (protected health information)
- Regulatory compliance (FDA, EMA)
- Patient safety impact
- **Security requirements:** Encryption, MFA, audit logs, penetration testing, incident response, disaster recovery

**CRA Classification:** Not applicable
- Internal use only
- Not sold as a product
- Not distributed outside organization
- **CRA requirements:** None

**Can you map "Critical" to CRA?** No. This Critical system has zero CRA obligations.

---

### Example 2: Simple To-Do List App (SaaS Product)

**Internal Classification:** Low
- No sensitive business data processed (basic task lists, no financial/health data)
- Low operational impact to your company if compromised
- Minimal regulatory requirements
- **Security requirements:** Basic security hygiene, code review, dependency scanning

**CRA Classification:** Default Category (but still has requirements)
- Not listed in Annex I or III, but is a product with digital elements
- Must comply with CRA base requirements
- **CRA requirements:** CE marking, SBOM, vulnerability disclosure within 24
  hours if exploited, security updates, technical documentation, self-assessment

**Can you map "Low" to CRA Default?** No. Even though both seem "low priority,"
the classification criteria are completely different - internal risk (business
impact to you) vs. product function type (CRA categorization).

---

### Example 3: Operating System

**Internal Classification:** Medium
- Used internally for development work
- Handles confidential source code
- Moderate business impact if compromised
- **Security requirements:** Standard security controls, access management, patching

**CRA Classification:** Important Product - Class II
- If selling/distributing the OS too, otherwise no CRA obligations
- Operating systems are explicitly listed in Annex III
- Highest CRA compliance level (short of "Critical Product")
- **CRA requirements:** All CRA Important requirements + additional enhanced requirements

**Can you map "Medium" to CRA?** No. The internal risk is Medium, but the CRA
obligations are the highest level.

---

## The Problem with Merging Them

When organizations try to merge CRA categories with internal risk levels, bad
things happen:

### Problem 1: Misclassification

**Scenario:** You label your internal Critical system as "Critical"
- ❌ **Wrong:** It's not a CRA Critical Product (it's internal!)
- ❌ **Waste:** You're applying product compliance requirements to internal tools
- ❌ **Confusion:** Legal team sees "CRA Critical" and panics unnecessarily

### Problem 2: Under-securing Internal Systems

**Scenario:** You map CRA Default Category to "Low Risk"
- ❌ **Wrong:** CRA Default doesn't mean low risk internally
- ❌ **Dangerous:** You under-invest in security for systems handling sensitive data
- ❌ **Gap:** You miss internal security requirements because you only look at CRA

### Problem 3: Legal Exposure

**Scenario:** You self-determine CRA categories without legal review
- ❌ **Wrong:** CRA classification requires legal expertise (not technical)
- ❌ **Risky:** Misclassification = non-compliance = penalties
- ❌ **Audit fail:** Auditors will ask "Where's the legal determination?"

### Problem 4: Process Breakdown

**Scenario:** You try to use one classification for two purposes
- ❌ **Wrong:** Security teams care about risk, legal teams care about compliance
- ❌ **Confusion:** No one knows if "High" means high security risk or high CRA category
- ❌ **Complexity:** Combined matrix becomes unmaintainable (5 internal levels × 4 CRA categories = 20 combinations)

## Conclusion: Keep Them Separate

Mixing up internal risk classifications with CRA compliance can lead to
inadequate security postures or unnecessary compliance burdens. Keep them
separate, understand the unique requirements and implications of each, and
apply them as needed based on your specific context and activities.

I see CRA just as any other legal entity or regulation. They are added on
top of or overrules the internal framework.
