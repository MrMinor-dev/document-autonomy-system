# 9-Gate Scorecard

Use this to score any document for agentic readiness. Each gate is pass/fail. Score = (gates passed / 9) × 100, rounded to nearest whole number.

A passing score is 85+. Below 85 means an AI agent will either misretrieve the doc, misinterpret it, or act on stale information.

---

## The 9 Gates

### Gate 1 — YAML Frontmatter Present and Complete

**Check:** Does the document start with a YAML frontmatter block containing at minimum: `title`, `version`, `updated`, `purpose`?

```yaml
---
title: Content Compliance Master
version: 2.3
updated: 2026-01-29
purpose: Single source of truth for all content compliance rules and enforcement logic
---
```

**Fail if:** No frontmatter block, or block is missing any of the four required fields.

**Why it matters for agents:** Agents use frontmatter to determine whether a document is current and what it covers — without reading the full content. Missing frontmatter means the agent either skips the doc or reads it in full every time.

---

### Gate 2 — SSOT Status Declared

**Check:** Does the frontmatter include an `ssot-for` field (if this is an authoritative source) or a `consumer-of` field (if this document references another)?

```yaml
ssot-for: content-compliance          # This IS the source
consumer-of: CONTENT-COMPLIANCE-MASTER.md  # This references the source
```

**Fail if:** No SSOT declaration. Document is ambiguous about whether it's authoritative or a consumer.

**Why it matters for agents:** Without SSOT declaration, agents can't resolve conflicts when two documents cover the same topic. Which one wins? The registry declares the answer; the document confirms it.

---

### Gate 3 — Section Headers Are Concept-Named

**Check:** Do all H2 and H3 headers name a concept, not a topic or a section number?

| ❌ Fails | ✅ Passes |
|---------|----------|
| `## Section 1` | `## Enforcement Rules` |
| `## Overview` | `## Prohibited Content Categories` |
| `## Background` | `## Severity Tier Definitions` |
| `## Details` | `## Cascade Dependencies` |

**Fail if:** Any H2 or H3 is a generic label, a number, or a vague noun.

**Why it matters for agents:** Semantic search uses headers as retrieval anchors. A header named "Section 1" returns on any query that includes a section number. A header named "Enforcement Rules" returns when the agent is looking for enforcement logic.

---

### Gate 4 — Cross-References Use File Paths

**Check:** Does every reference to another document use a full relative file path — not a name, nickname, or description?

| ❌ Fails | ✅ Passes |
|---------|----------|
| "See the schema doc" | `AOS/Knowledge/SUPABASE-SCHEMA-MASTER.md` |
| "Per the compliance framework" | `AOS/Capabilities/Compliance/CONTENT-COMPLIANCE-MASTER.md` |
| "The main workflow" | `AOS/Workflows/content-pipeline/README.md` |

**Fail if:** Any cross-reference uses a name or description instead of a path.

**Why it matters for agents:** Agents follow references programmatically. A name requires a search to resolve. A path is a direct lookup. Names also drift — the doc gets renamed, the reference breaks silently.

---

### Gate 5 — No Ambiguous Pronouns

**Check:** Are pronouns like "it," "this," "that," and "the system" always followed by a specific referent within the same sentence?

| ❌ Fails | ✅ Passes |
|---------|----------|
| "It enforces the rules." | "The compliance workflow enforces the rules." |
| "This should be checked first." | "The severity tier should be checked first." |
| "That process handles exceptions." | "The exception-routing workflow handles exceptions." |

**Fail if:** Any paragraph contains a pronoun whose referent requires reading a previous sentence to resolve.

**Why it matters for agents:** Documents are chunked for semantic search. A chunk containing "it enforces the rules" with no prior context is uninterpretable. The agent either guesses the referent or retrieves the wrong content.

---

### Gate 6 — Chunking Boundaries at Logical Breaks

**Check:** Are sections ≤500 tokens (roughly ≤400 words)? Does each section contain exactly one logical unit — one concept, one procedure, or one reference table?

**Fail if:** Any section exceeds 500 tokens, or contains multiple distinct concepts that would be retrieved independently.

**Scoring guidance:** Score this gate based on the worst-offending section. If one section mixes three concepts in 800 tokens, the gate fails regardless of how well the rest is structured.

**Why it matters for agents:** Semantic search retrieves chunks, not documents. A 1,500-token chunk containing an enforcement rule, a severity definition, and a cascade dependency will retrieve on queries about all three — but return content about all three mixed together. Splitting them means each retrieval is precise.

---

### Gate 7 — Version and Last-Updated Present

**Check:** Is there a version number and a last-updated date in the frontmatter? Is the version number incremented when content changes?

```yaml
version: 2.3
updated: 2026-01-29
```

**Fail if:** No version number, no date, or version hasn't changed since the document was last substantively edited.

**Why it matters for agents:** Agents use version and date to determine whether a document is current enough to act on. A document without version information can't be compared to a previous state — drift goes undetected.

---

### Gate 8 — Registry Entry Exists

**Check:** Is this document registered in `SSOT-REGISTRY-MASTER.md` with scope, status, and cascade dependencies documented?

**Fail if:** Document claims to be an SSOT but has no registry entry. Or document is a high-priority consumer with no registry reference.

**Why it matters for agents:** The registry is the index. An unregistered document is invisible to any agent that starts with the registry before searching. Cascade dependencies are only enforced if the registry knows the dependency exists.

---

### Gate 9 — Cascade Dependencies Documented

**Check:** For any SSOT document: are all downstream documents that depend on this one listed explicitly — either in the frontmatter or in a dedicated cascade section?

```yaml
cascade-dependencies:
  - AOS/Skills/content-compliance-skill/SKILL.md
  - AOS/Workflows/content-pipeline/README.md
  - HAIOS/claude-instructions.md
```

**Fail if:** SSOT document has no cascade dependencies listed. Or cascade list is known to be incomplete.

**Why it matters for agents:** When an SSOT is updated, the agent needs to know what else to update. Without an explicit cascade list, the agent either updates nothing (drift) or searches the entire knowledge base for references (slow, incomplete).

---

## Scoring Reference

| Score | Interpretation |
|-------|----------------|
| 100 (9/9) | Fully agentic-ready |
| 88 (8/9) | Production standard — one structural gap, acceptable for most docs |
| 77 (7/9) | Usable but retrieval quality will be inconsistent |
| 66 (6/9) | Significant gaps — agent will misretrieve or misinterpret |
| <66 | Not agentic-ready — requires optimization before indexing |

**Production target:** 85+ (8/9 gates minimum).

---

## How to Use

1. Open the document you want to score
2. Work through each gate in order — pass or fail, no partial credit
3. Record which gates failed
4. Fix failing gates (see optimization notes in each gate)
5. Re-score — a properly optimized document should reach 85+ in one pass

The 9-gate system was validated across 4 documents ranging from 200 lines to 2,800 lines. All 4 scored exactly 88/100 after optimization, confirming the gates test structural properties rather than content volume.
