# Adam Skills

Open-source Claude Code skills built by [Adam](https://github.com/christiandigital) — an autonomous AI agent system.

These skills encode battle-tested workflows from real client engagements and production autonomous agent operations. They work with Claude Code, Cowork, and any agent that supports the Skills standard.

## Skills

### 1. Community Chatbot Builder
Build RAG-powered AI chatbots for online communities (Skool, Circle, Discord, Mighty Networks). Covers the full pipeline from content scraping to deployment — voice analysis, ChromaDB ingestion, model routing, sync automation, and crash-proof hosting.

**Use when:** You want to build an AI assistant that knows a community's actual content and speaks in its voice.

### 2. Business Readiness Assessor
Evaluate whether an AI agent can be successfully built for a specific business. Structured intake process with 5-dimension scoring: domain coverage, technical feasibility, data availability, risk profile, and complexity.

**Use when:** You need to decide "should we build this?" before investing time on an AI project.

### 3. Agent OS (Autonomous Agent Operating System)
Build production-grade autonomous AI agents with self-healing, zombie detection, tiered thinking, and structured operational loops. Encodes patterns from a system running 100+ daily execution windows across 25+ scheduled tasks.

**Use when:** You want your AI agent project to run reliably in production without constant human supervision.

## Installation

Each skill is a standalone directory with a `SKILL.md` file. To use with Claude Code:

```bash
# Clone the repo
git clone https://github.com/christiandigital/adam-skills.git

# Copy the skill you want to your project's .claude/skills/ directory
cp -r adam-skills/community-chatbot-builder /your/project/.claude/skills/
```

Or reference directly in your Claude Code configuration.

## How Skills Work

Skills are markdown files that extend AI coding agents with domain-specific knowledge and workflows. When triggered, the agent reads the SKILL.md and follows its instructions to complete the task.

Each skill contains:
- **Frontmatter** (name + description) — determines when the skill triggers
- **Instructions** — step-by-step workflow the agent follows
- **Templates** — reusable structures, checklists, and patterns
- **Best practices** — hard-won lessons from production use

## Contributing

These skills are open-source. If you use one and find improvements, PRs are welcome.

## License

MIT
