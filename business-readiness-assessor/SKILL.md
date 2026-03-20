---
name: business-readiness-assessor
description: >
  Evaluate whether an AI agent, chatbot, or automation system can be successfully built for a
  specific business. Performs structured intake of business requirements and scores readiness
  across 5 dimensions: domain coverage, technical feasibility, data availability, risk profile,
  and complexity. Use this skill whenever someone wants to assess if AI is right for their
  business, evaluate a client for an AI project, scope an AI build, do a feasibility study
  for automation, or figure out what kind of AI agent a business needs. Also trigger when the
  user is onboarding a new client for AI services, building a proposal, or needs to determine
  if a business has enough data and infrastructure to support an AI deployment.
---

# Business Readiness Assessor

Systematically evaluate whether an AI agent can be successfully built for a specific business. This skill encodes a structured intake process and 5-dimension scoring system proven on real client engagements.

## When to Use This

This skill is for the critical "should we build this?" decision. Before investing time building an AI agent, chatbot, or automation system for a client (or yourself), run a readiness assessment. It catches showstoppers early — missing data, unrealistic expectations, high-risk domains — before you've burned hours on a doomed project.

The output is a clear READY / PARTIALLY_READY / NOT_READY verdict with specific scores, knowledge gaps, and risk flags.

## Step 1: Business Intake

Gather the business information needed for assessment. Start by collecting the required fields, then probe for optional fields that improve assessment quality.

### Required Fields (must have all 6)

| Field | What to Collect | Example |
|---|---|---|
| `business_name` | Company or project name | "CloudMetrics" |
| `industry` | Sector — determines which knowledge domains matter | "saas", "ecommerce", "healthcare", "finance", "support", "trading", "marketing" |
| `description` | 1-3 sentences: what the business does and what the AI agent should do | "B2B SaaS analytics platform. Agent should handle tier-1 support tickets and onboarding questions." |
| `objectives` | Specific goals the agent should achieve (at least 1) | ["Reduce support ticket volume by 40%", "Automate onboarding FAQ responses"] |
| `constraints` | Limitations — must include budget (`max_daily_spend_usd`) | {"max_daily_spend_usd": 10, "no_pii_storage": true} |
| `success_metrics` | Measurable outcomes to evaluate the agent (at least 1) | ["First response time < 30 seconds", "Customer satisfaction > 4.0/5"] |

### Optional Fields (improve assessment quality)

| Field | What It Tells You | Why It Matters |
|---|---|---|
| `data_sources` | What data the agent can access | No data = no useful agent |
| `tools` | APIs and platforms to integrate with | Integration complexity |
| `brand_voice` | How the agent should communicate | Voice matching difficulty |
| `target_market` | Who the agent serves | Audience expectations |
| `competitors` | Who they compete with | Competitive intelligence needs |
| `evaluation_scenarios` | Test cases the client cares about | Defines "done" clearly |
| `escalation_rules` | When to hand off to a human | Risk management |
| `operating_hours` | When the agent should be active | 24/7 vs business hours |
| `existing_tools` | Current tech stack | Integration surface |

### Intake Interview Guide

Walk through these questions with the client or user. The goal is to fill in as many fields as possible before running the assessment.

**Opening questions:**
1. "What does your business do, in a sentence or two?"
2. "What problem would an AI agent solve for you? What task eats the most time?"
3. "What does success look like? If the agent works perfectly in 3 months, what's different?"

**Data and infrastructure:**
4. "Where does your business knowledge live? (docs, website, CRM, Slack, etc.)"
5. "Do you have an existing knowledge base or FAQ? How large is it?"
6. "What tools and platforms do you use daily?"

**Constraints and risks:**
7. "What's your budget for AI API costs? (in dollars per day or month)"
8. "Are there things the agent absolutely must NOT do? (e.g., make financial commitments, access PII)"
9. "What industry regulations apply? (HIPAA, GDPR, financial compliance, etc.)"

**Scope:**
10. "Should this start small (FAQ bot) or handle complex workflows (multi-step processes)?"
11. "Who are the end users? Internal team, customers, or both?"
12. "When should the agent hand off to a human?"

### Completeness Score

Calculate intake completeness as: `(filled_required + filled_optional) / total_fields * 100`

- 80%+ = COMPLETE — ready for full assessment
- 50-79% = PARTIAL — can assess but with caveats
- <50% or missing required fields = INCOMPLETE — need more information before assessing

## Step 2: Run the 5-Dimension Assessment

Score each dimension 1-5. The scores determine the overall readiness verdict.

### Dimension 1: Domain Coverage (1-5)

Does the builder have enough knowledge about this industry and domain to build a useful agent?

| Score | Meaning | Example |
|---|---|---|
| 5 | Deep expertise, many relevant knowledge items | SaaS support agent and you've built 10 SaaS tools |
| 4 | Strong coverage, minor gaps fillable | Healthcare FAQ bot, you have medical content but need compliance review |
| 3 | Moderate — can support basic agent, gaps need filling | E-commerce returns bot, you understand the domain but need product-specific data |
| 2 | Thin coverage, significant learning needed | Legal contract analyzer, limited legal domain knowledge |
| 1 | Near-zero coverage, would produce an uninformed agent | Nuclear safety compliance bot with no nuclear domain knowledge |

**How to evaluate:** Check what relevant knowledge, experience, or reference material exists for this domain. If building for a client, check what content they can provide.

### Dimension 2: Technical Feasibility (1-5)

Can the requested capabilities actually be built with available tools?

| Score | Meaning | Example |
|---|---|---|
| 5 | All capabilities achievable with standard tools (APIs, RAG, browser automation) | FAQ chatbot using ChromaDB + Claude |
| 4 | Mostly achievable, one capability needs custom work | Support bot + CRM integration requiring custom API connector |
| 3 | Core features work, some need workarounds | Multi-language bot where translation layer adds complexity |
| 2 | Significant capabilities require tools that don't exist yet | Real-time video analysis agent |
| 1 | Core capability is technically impossible or years away | AGI-level reasoning required for the task |

**How to evaluate:** Map each requested capability to a concrete technical approach. If you can't describe how to build it, it's a feasibility risk.

### Dimension 3: Data Availability (1-5)

Are the data sources the agent needs actually accessible?

| Score | Meaning | Example |
|---|---|---|
| 5 | All data specified, accessible, and in usable format | Company wiki in markdown + API access to CRM |
| 4 | Most data available, minor gaps fillable | Website content + most docs, but some are in private Notion pages |
| 3 | Some sources available, others need configuration or permission | CRM data exists but needs API setup, some docs in locked Drive folders |
| 2 | Critical data exists but extraction is difficult | Knowledge is in people's heads, minimal documentation |
| 1 | Essential data doesn't exist or is completely inaccessible | No written content, no API access, no documentation |

**How to evaluate:** For each data source listed, verify: Does it exist? Can you access it? Is it in a format you can ingest?

### Dimension 4: Risk Profile (1-5, where 5 = low risk)

How high-stakes are the agent's tasks? Higher stakes = more risk = lower score.

| Score | Meaning | Example |
|---|---|---|
| 5 | Low stakes — errors are easily corrected, no harm potential | Internal FAQ bot, content recommendation |
| 4 | Minor stakes — errors cause inconvenience, not damage | Customer support triage (human reviews escalations) |
| 3 | Moderate stakes — needs supervision, errors have cost | Order processing automation, scheduling |
| 2 | High stakes — errors could cause financial or reputational harm | Financial advice, legal document generation |
| 1 | Critical stakes — errors could cause physical harm or legal liability | Medical diagnosis, safety-critical systems |

**How to evaluate:** Ask: "What's the worst thing that happens if the agent gives a wrong answer?" The answer determines the score.

### Dimension 5: Complexity (1-5, where 5 = simple)

How complex is the agent to build and maintain?

| Score | Meaning | Example |
|---|---|---|
| 5 | Simple FAQ/lookup agent — one data source, one output type | "Answer questions about our pricing page" |
| 4 | Moderate — 2-3 data sources, some logic branching | "Answer support questions from our docs and route complex ones to humans" |
| 3 | Multi-tool workflow — needs integrations, conditional logic | "Monitor CRM, draft responses, update tickets, escalate based on rules" |
| 2 | Complex orchestration — multiple agents, real-time data, complex state | "Manage multi-step sales pipeline with competitor tracking and automated follow-ups" |
| 1 | Extremely complex — multi-agent system, real-time processing, custom ML | "Autonomous trading system with regime detection and portfolio optimization" |

## Step 3: Calculate Overall Readiness

Use the scores to determine the verdict:

```
average_score = sum(all_5_scores) / 5
min_score = lowest individual score

if average_score >= 4.0 AND min_score >= 3:
    verdict = "READY"
elif average_score >= 2.5 AND min_score >= 2:
    verdict = "PARTIALLY_READY"
else:
    verdict = "NOT_READY"
```

**Interpreting the verdict:**

- **READY** — Green light. The project has strong fundamentals across all dimensions. Proceed to build.
- **PARTIALLY_READY** — Proceed with caution. Identify the weakest dimensions and create a mitigation plan before building. Common mitigations: fill knowledge gaps with targeted research, simplify scope to reduce complexity, add human-in-the-loop for high-risk areas.
- **NOT_READY** — Don't build yet. One or more dimensions are critically weak. Address the blockers first: gather more data, reduce scope, or decline the project.

## Step 4: Generate the Assessment Report

Structure the output as a clear, actionable report:

```markdown
# Readiness Assessment: [Business Name]

## Verdict: [READY / PARTIALLY_READY / NOT_READY]
**Average Score:** [X.X/5]

## Dimension Scores
| Dimension | Score | Summary |
|---|---|---|
| Domain Coverage | X/5 | [One-line reasoning] |
| Technical Feasibility | X/5 | [One-line reasoning] |
| Data Availability | X/5 | [One-line reasoning] |
| Risk Profile | X/5 | [One-line reasoning] |
| Complexity | X/5 | [One-line reasoning] |

## Knowledge Gaps (if any)
- [Specific topic that needs to be learned or researched]

## Risk Flags (if any)
- [Specific concerns requiring human attention]

## Recommended Agent Type
[FAQ bot / Support assistant / Workflow agent / Multi-agent system]

## Recommended Next Steps
1. [First action based on the assessment]
2. [Second action]
3. [Third action]
```

## Industry-Specific Knowledge Domains

Different industries require different knowledge coverage. Use this mapping to evaluate Domain Coverage:

| Industry | Key Knowledge Domains |
|---|---|
| SaaS | Agent architecture, API design, cost optimization |
| E-commerce | API design, coding patterns, CRM integration |
| Marketing | CRM, API design, cost optimization |
| Healthcare | Security guardrails, API design, knowledge management |
| Finance | Trading/financial, security guardrails, API design |
| Support | CRM, knowledge management, agent architecture |
| Trading | Trading domain, API design, cost optimization |

## Common Patterns and Anti-Patterns

**Patterns that predict success:**
- Client has a large, organized knowledge base (docs, FAQs, courses)
- Clear, measurable success metrics defined upfront
- Low-to-moderate risk domain (errors don't cause harm)
- Client willing to start with a simple scope and expand
- Existing API access to required data sources

**Red flags that predict failure:**
- "We want the AI to do everything" (undefined scope)
- No existing documentation or knowledge base
- High-stakes domain with no human review layer
- Client expects 100% accuracy from day one
- Critical data is in people's heads, not written down
- Budget is insufficient for the required API usage

## Adapting the Framework

This assessment framework works for any AI agent project, not just chatbots. Adjust the dimensions based on what you're building:

- **Chatbots:** Weight Domain Coverage and Data Availability heavily — the bot is only as good as its knowledge base
- **Workflow automation:** Weight Technical Feasibility and Complexity — integration complexity is the main risk
- **Decision support:** Weight Risk Profile heavily — wrong recommendations can cause real damage
- **Content generation:** Weight Domain Coverage and brand_voice — output quality depends on domain understanding
