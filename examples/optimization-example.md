# Optimization Example — 45→88

A real document section before and after 9-gate optimization. The section is from an operational document that scored 45/100 before optimization and 88/100 after. The content is illustrative; the structural problems and fixes are exact.

---

## Before Optimization (45/100)

Gates failed: 1 (no frontmatter), 2 (no SSOT declaration), 3 (generic headers), 4 (name references), 5 (ambiguous pronouns), 9 (no cascade dependencies)

```markdown
# Compliance Framework

## Overview

This document covers the rules we use for content compliance. It should be checked
before publishing anything. The system enforces these automatically but there are
some cases where it needs human review.

See the brand guidelines for tone requirements. The workflow handles the rest.

## Rules

Products must not make health claims. This is an FTC requirement. Prohibited
categories include: medical devices, supplements with efficacy claims, and anything
that implies FDA approval.

When it detects a violation, it blocks publishing and logs the issue. The team 
should review these within 24 hours.

## Severity

There are three levels. Critical means block immediately. High means flag for
review. Low means log only.
```

---

## After Optimization (88/100)

Gates passed: 1, 2, 3, 4, 5, 6, 7, 8, 9 (8 of 9 — Gate 6 borderline on the severity section, acceptable)

```markdown
---
title: Content Compliance Master
version: 1.0
updated: 2025-11-13
purpose: Single source of truth for all content compliance rules, severity tiers, and enforcement logic
ssot-for: content-compliance
cascade-dependencies:
  - AOS/Skills/content-compliance-skill/SKILL.md
  - AOS/Workflows/content-pipeline/README.md
---

# Content Compliance Master

## Prohibited Content Rules

Products must not make health claims. FTC regulations prohibit efficacy claims
for medical devices, dietary supplements, and any product that implies FDA approval
or medical endorsement.

Prohibited categories:
- Medical devices with therapeutic claims
- Dietary supplements with efficacy claims
- Products implying FDA approval or clinical validation

Reference: `AOS/Capabilities/Compliance/FTC-RULES-MASTER.md`

## Enforcement Behavior

The content compliance workflow blocks publishing when a prohibited category is
detected. The content compliance workflow logs the violation with severity tier,
product ID, and detection timestamp. Violations flagged as Critical or High require
human review within 24 hours before the product can be resubmitted.

Reference: `AOS/Workflows/content-pipeline/compliance-node/README.md`

## Severity Tier Definitions

| Tier | Definition | Enforcement Action |
|------|------------|-------------------|
| Critical | Direct health claim or FDA implication | Block immediately, no override |
| High | Implied efficacy, ambiguous claim | Block + flag for human review |
| Low | Borderline language, contextual risk | Log only, publish proceeds |

Severity tier assignment is handled by `AOS/Skills/content-compliance-skill/SKILL.md`.
Human override is permitted for High tier only. Critical tier has no override path.

## Tone and Brand Alignment

Content tone requirements are governed by `TGT/Knowledge/BRAND-MASTER.md`.
Tone compliance is a separate check from content compliance. Both checks must pass
before publishing proceeds.
```

---

## What Changed and Why

| Gate | Before | After | Why |
|------|--------|-------|-----|
| Gate 1 — Frontmatter | None | Full YAML block with 5 fields | Agent can't determine currency or scope without frontmatter |
| Gate 2 — SSOT | None | `ssot-for: content-compliance` | Without declaration, agents can't resolve conflicts with overlapping docs |
| Gate 3 — Headers | "Overview", "Rules", "Severity" | "Prohibited Content Rules", "Enforcement Behavior", "Severity Tier Definitions" | Generic headers retrieve on any query; concept headers retrieve on the right query |
| Gate 4 — References | "See the brand guidelines", "the workflow" | Full file paths to BRAND-MASTER.md and compliance-node | Path is a direct lookup; name requires a search that may return the wrong doc |
| Gate 5 — Pronouns | "it detects", "it should be checked", "the team" | "The content compliance workflow detects", "Violations require human review" | Chunks are retrieved without context. "It" has no referent in a standalone chunk. |
| Gate 9 — Cascades | None | 2 downstream dependencies listed | When this doc changes, SKILL.md and the workflow README both need review — cascade rule makes that automatic |

Gates 6, 7, and 8 passed before and after (document had reasonable section lengths, was versioned, and was registered). The 5 structural fixes moved the score from 45 to 88.

---

## Key Observation

The content didn't change. The compliance rules, severity tiers, and enforcement behavior are identical before and after. What changed is the structure — headers, references, frontmatter, pronoun resolution. The same information went from unreliable for agent retrieval to production-grade.

This is why "just keep docs updated" isn't enough. A doc can be perfectly current and still fail an agent on retrieval. Agentic readability is a separate design requirement.
