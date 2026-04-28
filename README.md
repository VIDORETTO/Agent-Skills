<div align="center">

# 🧠 Agent Skills

**A curated library of SKILL.md files for Claude — turning the AI into a specialist on demand.**

[![Skills](https://img.shields.io/badge/skills-20+-6366f1?style=for-the-badge&logo=buffer&logoColor=white)](https://github.com/VIDORETTO/Agent-Skills)
[![Claude](https://img.shields.io/badge/Built%20for-Claude%20AI-cc785c?style=for-the-badge&logo=anthropic&logoColor=white)](https://claude.ai)
[![License](https://img.shields.io/badge/license-MIT-22c55e?style=for-the-badge)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-3b82f6?style=for-the-badge)](https://github.com/VIDORETTO/Agent-Skills/pulls)

</div>

---

## 📖 What is this?

**Agent Skills** is a growing collection of structured `SKILL.md` files designed to extend and specialize Claude's behavior in specific domains. Each skill acts as a **modular expert module** — when loaded into Claude's context, it transforms the AI into a focused specialist with deep, opinionated guidance on how to tackle a particular type of task.

Think of it as a **skills marketplace for AI agents**: instead of relying on generic prompting, you load a skill and Claude behaves like a senior engineer, security auditor, UX specialist, or whatever domain the skill covers.

---

## ⚡ How It Works

Each skill follows a consistent structure:

```
skill-name/
└── SKILL.md       # The skill definition file
```

A `SKILL.md` typically contains:

- **Trigger conditions** — when Claude should activate this skill
- **Methodology** — a structured, opinionated approach to the task
- **Frameworks & checklists** — domain-specific tools Claude should apply
- **Output format** — how results should be structured and delivered
- **Edge cases & anti-patterns** — common mistakes to avoid

When you reference or inject a `SKILL.md` into Claude's system prompt or conversation context, Claude operates within that expert framework for the duration of the task.

---

## 🗂️ Skills Catalog

> This catalog is updated as new skills are added. Each skill targets a specific domain and problem type.

### 🔐 Security & Trust

| Skill | Description |
|---|---|
| [`security-vulnerability-audit-remediation-backlog-guardian`](./security-vulnerability-audit-remediation-backlog-guardian/) | Full security audit of codebases — identifies vulnerabilities, scores severity, and generates a prioritized remediation backlog |
| [`authentication-authorization-access-control-guardian`](./authentication-authorization-access-control-guardian/) | Deep review of auth flows, JWT handling, RBAC policies, OAuth, and session management |
| [`abuse-pattern-detector`](./abuse-pattern-detector/) | Identifies business logic abuse gaps — promo exploitation, referral farming, free trial abuse, and other platform attack vectors |

### 🏗️ Architecture & Backend

| Skill | Description |
|---|---|
| [`backend-architecture-scalability-guardian`](./backend-architecture-scalability-guardian/) | Reviews and designs scalable backend systems — microservices, queues, caching strategies, and fault tolerance patterns |
| [`api-design-contract-consistency-guardian`](./api-design-contract-consistency-guardian/) | Audits REST/GraphQL API design for contract consistency, versioning, error handling, and developer experience |
| [`database-structure-query-optimization-specialist`](./database-structure-query-optimization-specialist/) | Analyzes schemas, indexes, query plans, and N+1 problems — optimizes for performance at scale |
| [`observability-logging-monitoring-guardian`](./observability-logging-monitoring-guardian/) | Designs observability stacks — structured logging, distributed tracing, alerting, and SLO/SLA frameworks |
| [`vps-production-agent`](./vps-production-agent/) | Manages production VPS environments — deployment pipelines, process supervision, Nginx, SSL, and server hardening |

### 💰 SaaS & Product

| Skill | Description |
|---|---|
| [`billing-subscription-plan-enforcement-guardian`](./billing-subscription-plan-enforcement-guardian/) | Designs and audits billing systems — Stripe integration, plan enforcement, metered usage, and webhook reliability |
| [`saas-readiness-user-centric-transformation-guardian`](./saas-readiness-user-centric-transformation-guardian/) | Transforms internal tools or MVPs into production-ready SaaS products with proper multi-tenancy, isolation, and UX |
| [`conversion-upgrade-flow-specialist`](./conversion-upgrade-flow-specialist/) | Optimizes free-to-paid conversion flows — upgrade triggers, paywall UX, and pricing page psychology |
| [`onboarding-user-activation-optimizer`](./onboarding-user-activation-optimizer/) | Designs onboarding flows that maximize activation — aha moments, progress mechanics, and drop-off analysis |
| [`admin-panel-internal-operations-architect`](./admin-panel-internal-operations-architect/) | Designs internal admin panels — role management, audit logs, bulk operations, and ops dashboards |

### 🎨 Frontend & UX

| Skill | Description |
|---|---|
| [`frontend-ux-refinement-specialist`](./frontend-ux-refinement-specialist/) | Audits and improves frontend UX — interaction design, accessibility, performance perception, and visual hierarchy |
| [`landing-page-marketing-site-conversion-specialist`](./landing-page-marketing-site-conversion-specialist/) | Builds and reviews landing pages with conversion-first thinking — copy, CRO, social proof, and A/B strategy |
| [`seo-geo-discoverability-specialist`](./seo-geo-discoverability-specialist/) | Covers technical SEO, GEO (Generative Engine Optimization), structured data, and AI-era discoverability |

### 🧹 Code Quality & Docs

| Skill | Description |
|---|---|
| [`code-hygiene-maintainability-guardian`](./code-hygiene-maintainability-guardian/) | Enforces clean code principles — naming conventions, complexity limits, SOLID, dead code detection, and refactoring |
| [`codebase-documentation-developer-onboarding-writer`](./codebase-documentation-developer-onboarding-writer/) | Writes comprehensive developer docs — architecture overviews, ADRs, API references, and onboarding guides |
| [`performance-optimization-specialist`](./performance-optimization-specialist/) | Profiles and optimizes application performance — Core Web Vitals, bundle size, server response times, and memory leaks |

### 🤖 AI & RAG

| Skill | Description |
|---|---|
| [`rag-best-practices`](./rag-best-practices/) | Builds production-grade RAG pipelines — chunking strategies, embedding models, retrieval tuning, and grounded generation |

---

## 🚀 Getting Started

### Option 1 — System Prompt Injection

Copy the contents of the desired `SKILL.md` and paste it into Claude's system prompt before starting your task:

```
[System]
<skill>
{paste SKILL.md contents here}
</skill>

You are now operating as a specialist. Apply the skill framework to all tasks in this session.
```

### Option 2 — Claude.ai Projects

1. Create a new **Project** in Claude.ai
2. Add the `SKILL.md` file(s) as **Project Knowledge**
3. Every conversation in that project will have the skill active automatically

### Option 3 — Claude Code / API

Reference the skill file path in your agent configuration:

```python
with open("./security-vulnerability-audit-remediation-backlog-guardian/SKILL.md") as f:
    skill = f.read()

system_prompt = f"""
You are an expert agent. Apply the following skill to your work:

{skill}
"""
```

### Option 4 — Multi-Skill Stacking

Skills can be combined for complex tasks. Load multiple skills when a task spans domains:

```
[System]
You have access to the following skills:
<skill name="security">...</skill>
<skill name="api-design">...</skill>

Apply both frameworks when auditing this API for security and design quality.
```

---

## 🏛️ Skill Anatomy

Every skill in this repository follows this structure:

```markdown
---
name: skill-name
description: One-line trigger description for the agent routing system
---

## Purpose
What this skill is for and when to use it.

## Trigger Conditions
Keywords, task types, or contexts that should activate this skill.

## Methodology
Step-by-step framework the AI must follow.

## Output Format
How results should be structured and delivered to the user.

## Anti-Patterns
Common mistakes this skill helps avoid.
```

---

## ➕ Adding a New Skill

This repository grows continuously. To add a skill:

1. **Create a folder** with a descriptive kebab-case name:
   ```
   mkdir my-new-skill-name
   ```

2. **Create the `SKILL.md`** following the anatomy above. Be opinionated — the value is in the framework, not generic advice.

3. **Update this README** — add your skill to the catalog table in the appropriate category.

4. **Open a Pull Request** with a short description of what the skill does and what problems it solves.

### Naming Convention

Skill folder names should follow this pattern:

```
{domain}-{specialization}-{role}
```

Examples:
- `database-query-optimization-specialist`
- `frontend-accessibility-guardian`
- `payment-fraud-detection-analyst`

---

## 🗺️ Roadmap

Skills currently planned or in progress:

- [ ] `test-coverage-quality-guardian` — TDD, coverage gaps, snapshot testing strategy
- [ ] `data-pipeline-etl-reliability-guardian` — batch jobs, idempotency, failure recovery
- [ ] `email-deliverability-lifecycle-specialist` — SPF/DKIM, sequences, churn prevention
- [ ] `mobile-app-ux-performance-guardian` — React Native / Flutter performance and UX
- [ ] `ai-prompt-engineering-specialist` — system prompt design, few-shot examples, evals
- [ ] `growth-analytics-metrics-architect` — funnel tracking, cohort analysis, event schemas

> Have an idea for a skill? Open an issue or submit a PR.

---

## 🤝 Contributing

Contributions are welcome! A good skill is:

- **Specific** — targets a well-defined problem domain
- **Opinionated** — provides a real framework, not vague guidance
- **Actionable** — Claude can follow it step-by-step to produce real outputs
- **Transferable** — useful across different codebases and contexts

Please keep skills focused. One skill = one domain. Avoid creating "do everything" skills.

---

## 📄 License

MIT — use freely, attribute if you share.

---

<div align="center">

**Built by [VIDORETTO](https://github.com/VIDORETTO) · Growing continuously**

*If this saved you time, leave a ⭐*

</div>
