<div align="center">

# 🔍 Best Option Research Specialist

**The generalist "best X for Y" skill — turn any vague recommendation request into a ranked, defensible answer.**

[![Skill](https://img.shields.io/badge/skill-best--option--research--specialist-6366f1?style=for-the-badge)](#)
[![Generalist](https://img.shields.io/badge/scope-generalist-22c55e?style=for-the-badge)](#)
[![Method](https://img.shields.io/badge/method-triangulation-cc785c?style=for-the-badge)](#)

</div>

---

## 📖 What is this?

**Best Option Research Specialist** is a generalist research skill for answering *any* "best X for Y" question — skills, tools, libraries, SaaS, frameworks, courses, hardware, anything.

Instead of returning a single SEO listicle, it **triangulates 4 independent source tiers** and only recommends what converges across them:

| Tier | Source type | What it tells you |
|---|---|---|
| **1. Real users** | Reddit, HN, Quora, forums | Ground truth — pain, complaints, "I switched because..." |
| **2. Market proof** | X/Twitter, Product Hunt, GitHub stars, G2/Capterra | Velocity — hype vs sustained use |
| **3. Expert reviews** | Benchmarks, blogs, Gartner, newsletters | Structured comparison (flag sponsored bias) |
| **4. Official specs** | Docs, pricing, changelogs, repos | Verification — price, freshness, maintenance |

The result is not "it depends" — it's a **ranked Top 3-5 with trade-offs, a clear winner, and who should pick what**.

---

## ⚡ When to Use

Trigger this skill when the user says:

- "pesquise as melhores X para Y"
- "qual a melhor X para Y"
- "best X for Y"
- "top picks for X"
- "recomendação de X"

Works for **any domain** — not limited to skills, marketing, or code.

Examples:
- "melhores skills de marketing para SEO"
- "best JS framework for solo founder em 2026"
- "melhor note-taking app para dev"
- "top RAG tools for small team"

---

## 🧭 Methodology (7 steps)

1. **Parse** — extract X (category), Y (use case), constraints (price, stack, language)
2. **Expand queries** — generate 5-8 variations with `site:` operators and temporal filters
3. **Multi-source sweep** — query all 4 tiers (≥3 web_search + 2 web_extract)
4. **Dynamic evaluation framework** — derive criteria weights from Y (cheap → price HIGH, enterprise → security HIGH)
5. **Score & cross-check** — 6-10 candidates → 3-5 finalists; downrank SEO-only or abandoned options
6. **Synthesize & rank** — TL;DR verdict, ranked list with pros/cons, comparison table, honorable mentions, sources
7. **Verify** — check official pricing/features; never hallucinate

---

## 📦 Output Format

**TL;DR Verdict** — one sentence: winner + runner-up + who should pick which.

**Ranked Top 3-5** — each with one-liner, 2-3 pros, 2-3 cons, best-for persona, price, freshness, 1-2 source links.

**Comparison table** — criteria vs candidates (compact).

**Avoid list** — what NOT to pick and why.

**Sources grouped by tier with dates** — e.g., "avaliado em ago/2026".

---

## 🔎 Search Operators

- `site:reddit.com "X" "best"`
- `site:github.com X stars:>1000`
- `X vs Y site:medium.com OR site:dev.to`
- `X pricing 2026`
- Add `2026` / `after:2025` for recency

---

## 🚫 Anti-Patterns

This skill explicitly avoids:

- Recommending from a single "top 10" blog
- Ignoring forums (where failure modes live)
- Hiding sponsored/affiliate bias
- Recommending abandoned tools (checks last update)
- Hallucinating pricing or features

---

## 🤖 Usage

```markdown
Load this skill when the user asks for "best X for Y".

System prompt:
<skill>
{paste SKILL.md contents here}
</skill>

Then follow the 7-step workflow and deliver a ranked, cited answer.
```

Stack with other skills when the request is domain-specific — e.g., combine with `seo-geo-discoverability-specialist` for "best SEO tools".

---

## 📁 Files

```
best-option-research-specialist/
├── SKILL.md   # The specialist framework (this skill)
└── README.md  # This file
```

---

> Part of [Agent-Skills](https://github.com/VIDORETTO/Agent-Skills) — a curated library of SKILL.md files for Claude.
