# LLM Wiki Agent Schema

> This file tells the LLM agent how to structure, maintain, and operate this wiki.
> It is the single source of truth for wiki conventions and workflows.
> Update it as conventions evolve.

---

## 1. Architecture

Three layers. The agent never confuses them.

| Layer           | Path                               | Role                                                                                         | Mutability                         |
| --------------- | ---------------------------------- | -------------------------------------------------------------------------------------------- | ---------------------------------- |
| **Raw sources** | `raw/`                             | Immutable source documents (articles, papers, clips, images, transcripts). Agent reads only. | Human curates. Agent never writes. |
| **Wiki**        | Vault root (`./*.md`, `./**/*.md`) | Agent-generated knowledge pages. Summaries, entities, concepts, synthesis.                   | Agent writes. Human reviews.       |
| **Schema**      | `AGENTS.md` (this file)            | Conventions, workflows, page types.                                                          | Human and agent co-evolve.         |

---

## 2. Directory Structure

```
/
├── AGENTS.md              ← This file. Agent reads on every session start.
├── index.md               ← Content catalog. Agent updates on every ingest.
├── log.md                 ← Chronological operation log. Append-only.
├── raw/                   ← Immutable sources.
│   ├── articles/
│   ├── papers/
│   ├── books/
│   ├── transcripts/
│   ├── clips/
│   └── assets/            ← Local images and attachments.
├── sources/               ← Wiki pages: one per ingested source.
├── entities/              ← Wiki pages: people, orgs, models, products.
├── concepts/              ← Wiki pages: techniques, architectures, theories, terms.
├── synthesis/             ← Wiki pages: comparisons, overviews, evolving theses.
├── queries/               ← Wiki pages: answers to good questions, filed back into wiki.
└── _templates/
    ├── source.md
    ├── entity.md
    ├── concept.md
    ├── synthesis.md
    └── query.md
```

---

## 3. Page Types & Conventions

Every wiki page must have YAML frontmatter. Every wiki page must use wikilinks (`[[Page Name]]`) liberally.

### 3.1 Source Page (`sources/*.md`)

Created once per ingested raw document.

```yaml
---
type: source
title: "Article Title"
source: "raw/articles/2026-04-26-article-slug.md"
author: "Author Name"
date: 2026-04-26
tags: [tag1, tag2]
entities: [Entity One, Entity Two]
concepts: [Concept One, Concept Two]
summary: "One-line summary of the source."
status: ingested
---
```

**Body structure:**

- `## Summary` — 3-5 paragraph summary in your own words.
- `## Key Takeaways` — Bullet points of the most important claims.
- `## Entities Mentioned` — Links to entity pages.
- `## Concepts Discussed` — Links to concept pages.
- `## Connections` — Links to related sources or synthesis pages.
- `## Raw Notes` — Verbatim excerpts worth keeping, with blockquotes.

### 3.2 Entity Page (`entities/*.md`)

Created for any person, organization, model, product, or project mentioned across sources.

```yaml
---
type: entity
name: "Entity Name"
aliases: ["Alt Name", "Abbreviation"]
category: person | org | model | product | project
tags: []
first_seen: 2026-04-26
last_updated: 2026-04-26
sources: [Source Title One, Source Title Two]
---
```

**Body structure:**

- `## Overview` — What this entity is, in 2-3 sentences.
- `## Key Facts` — Bullet points of verified facts.
- `## Related Entities` — Wikilinks.
- `## Related Concepts` — Wikilinks.
- `## Sources` — Links to source pages where this entity appears.
- `## Contradictions / Open Questions` — Flag if sources disagree about this entity.

**When updating an entity:**

- Append new facts. Do not overwrite old facts unless explicitly correcting an error.
- Update `last_updated`.
- Append new sources to the `sources` list.
- If a new source contradicts an old fact, note it in `## Contradictions / Open Questions`.

### 3.3 Concept Page (`concepts/*.md`)

Created for techniques, architectures, theories, methodologies, or domain terms.

```yaml
---
type: concept
name: "Concept Name"
aliases: ["Alt Name"]
domain: "e.g. machine-learning, psychology, business"
tags: []
first_seen: 2026-04-26
last_updated: 2026-04-26
sources: []
---
```

**Body structure:**

- `## Definition` — Clear definition in your own words.
- `## Explanation` — Deeper explanation, analogies, examples.
- `## Related Concepts` — Wikilinks.
- `## Related Entities` — Wikilinks.
- `## Sources` — Links to source pages.
- `## Contradictions / Evolving Understanding` — Note if understanding of this concept has changed over time.

### 3.4 Synthesis Page (`synthesis/*.md`)

Created for comparisons, overviews, evolving theses, or topic deep-dives.

```yaml
---
type: synthesis
title: "Synthesis Title"
tags: []
last_updated: 2026-04-26
sources: []
---
```

**Body structure:**

- `## Thesis / Main Argument` — The core claim or overview.
- `## Evidence` — Points drawn from specific sources, with citations.
- `## Connections` — How this ties together multiple entities/concepts/sources.
- `## Open Questions` — What remains unclear or needs further investigation.

### 3.5 Query Page (`queries/*.md`)

Created when an answer to a question is worth preserving.

```yaml
---
type: query
question: "The exact question asked"
date: 2026-04-26
tags: []
sources_used: []
---
```

**Body structure:**

- `## Answer` — The synthesized answer, with citations to wiki pages and raw sources.
- `## Reasoning` — How the answer was derived.
- `## Related Pages` — Wikilinks to relevant source/entity/concept/synthesis pages.

---

## 4. Workflows

### 4.1 Ingest

Triggered when a new source is added to `raw/`.

**Step-by-step:**

1. **Read the source** in `raw/`.
2. **Discuss key takeaways** with the user. Ask: what surprised you? What should we emphasize? What connects to existing wiki pages?
3. **Create a source page** in `sources/` using the template.
4. **Identify entities** mentioned in the source. For each entity:
   - If an entity page exists, update it with new facts and add the source.
   - If not, create a new entity page.
5. **Identify concepts** mentioned in the source. For each concept:
   - If a concept page exists, update it.
   - If not, create a new concept page.
6. **Update relevant synthesis pages** if the source changes or supports an existing thesis.
7. **Update `index.md`** — add the new source page, update entity/concept listings.
8. **Append to `log.md`**:

   ```markdown
   ## [2026-04-26] ingest | Article Title

   - Source: `raw/articles/article-slug.md`
   - Pages created: [[Article Title]], [[Entity One]], [[Concept One]]
   - Pages updated: [[Existing Entity]], [[Existing Synthesis]]
   ```

**Rules:**

- One source at a time unless user explicitly requests batch.
- Never overwrite old facts on entity/concept pages — append and flag contradictions.
- Always use wikilinks when referencing other wiki pages.
- File names: kebab-case. Example: `sources/2026-04-26-article-title.md`.

### 4.2 Query

Triggered when the user asks a question.

**Step-by-step:**

1. **Read `index.md`** to identify relevant pages.
2. **Read the relevant wiki pages** (source, entity, concept, synthesis).
3. **Synthesize an answer** with inline citations like `([[Source Title]], [[Entity Name]])`.
4. **Discuss the answer** with the user. Refine if needed.
5. **If the answer is worth preserving**, ask the user: "Should I file this as a query page?"
   - If yes, create a page in `queries/` using the template.
   - Link it from relevant synthesis or concept pages.
6. **Append to `log.md`**:

   ```markdown
   ## [2026-04-26] query | What is X and how does it relate to Y?

   - Pages read: [[Concept X]], [[Entity Y]], [[Source Z]]
   - Filed: [[queries/what-is-x-and-y]] (optional)
   ```

**Rules:**

- Prefer reading wiki pages over raw sources. The wiki is the compiled knowledge.
- If the wiki is insufficient, note the gap and suggest sources to ingest.
- Citations must point to wiki pages, not just raw sources.

### 4.3 Lint

Triggered when the user asks for a health check, or periodically.

**Checklist:**

- [ ] **Contradictions** — Scan entity and concept pages for facts that contradict each other. Flag them.
- [ ] **Stale claims** — Identify claims superseded by newer sources. Mark with `> ⚠️ Stale: superseded by [[Newer Source]]`.
- [ ] **Orphan pages** — Find pages with zero inbound wikilinks. Suggest links from other pages or ask if they should be deleted.
- [ ] **Missing pages** — Find concepts or entities mentioned in wikilinks that have no corresponding page. Create stub pages or note them in a `TODO` section.
- [ ] **Broken wikilinks** — Find wikilinks that don't resolve to existing files.
- [ ] **Missing cross-references** — Suggest connections between pages that should link to each other.
- [ ] **Index accuracy** — Ensure `index.md` reflects the actual state of the wiki.
- [ ] **Data gaps** — Note topics that are under-developed and suggest sources or web searches.

**Output:**

- Write findings to `synthesis/lint-report-YYYY-MM-DD.md`.
- Append summary to `log.md`.
- Present top 3-5 issues to the user with suggested fixes.

---

## 5. Special Files

### 5.1 `index.md`

Content-oriented catalog. Updated on every ingest.

Structure:

```markdown
# Wiki Index

Last updated: 2026-04-26

## Sources

- [[Article Title]] — One-line summary. `raw/articles/slug.md`

## Entities

- [[Entity Name]] — What/who they are. Category: person.

## Concepts

- [[Concept Name]] — What it means. Domain: domain.

## Synthesis

- [[Synthesis Title]] — What it covers.

## Queries

- [[Query Title]] — Question answered.
```

### 5.2 `log.md`

Chronological, append-only. Use consistent prefixes for parseability.

```markdown
# Wiki Log

## [2026-04-26] ingest | Article Title

...

## [2026-04-26] query | Question asked

...

## [2026-04-26] lint | Health check

...
```

---

## 6. Formatting Rules

- **File names**: kebab-case, descriptive, include date for sources (`YYYY-MM-DD-slug.md`).
- **Wikilinks**: Always use `[[Page Name]]` for internal links. Do not use Markdown URLs for wiki pages.
- **External links**: Use standard Markdown `[text](url)` for external references.
- **Frontmatter**: Every wiki page must have YAML frontmatter with at minimum `type`, `title`/`name`, and `last_updated`.
- **Blockquotes for excerpts**: When quoting raw sources, use `>` blockquotes with attribution.
- **Contradictions**: Flag with `> ⚠️ Contradiction: [[Source A]] claims X, but [[Source B]] claims Y.`
- **Stale claims**: Flag with `> ⚠️ Stale: superseded by [[Newer Source]].`

---

## 7. Agent Session Startup

At the beginning of every session, the agent should:

1. Read `AGENTS.md` (this file).
2. Read `index.md` to understand the current state of the wiki.
3. Read the last 5 entries of `log.md` to understand recent activity.
4. Ask the user: "What would you like to do? Ingest a source, query the wiki, or run a lint?"

---

## 8. Tips

- Obsidian is the IDE; the agent is the programmer; the wiki is the codebase.
- The user has Obsidian open on one side and the agent on the other. The user browses in real time.
- When in doubt, ask the user before creating or deleting pages.
- Prefer updating existing pages over creating new ones if the content fits.
- The wiki should feel like a living document — not a dump of summaries.
