# Wiki Template

An Obsidian-friendly knowledge wiki designed to be maintained collaboratively with an LLM.

This repository separates immutable source material from agent-written knowledge pages, so the vault can grow into a structured research wiki instead of a loose note dump.

## What This Is

- `raw/` holds the original source material and is treated as read-only by the agent.
- The vault root and content folders hold generated wiki pages such as source summaries, entities, concepts, syntheses, and preserved query answers.
- `AGENTS.md` defines the schema, workflows, and conventions the agent should follow.

## Using This Vault

Open the repository in Obsidian and work with the agent against the conventions in `AGENTS.md`.

Typical prompts:

- `Ingest raw/articles/2026-04-30-example.md`
- `What does the wiki say about retrieval-augmented generation?`
- `Run a lint on the vault`

## Conventions

- Treat `raw/` as immutable.
- Prefer updating existing pages over creating redundant ones.
- Do not overwrite old facts on entity or concept pages without explicit correction.
- Flag contradictions and stale claims inline.
- Use descriptive kebab-case filenames.
- Use Markdown links for external URLs and `[[wikilinks]]` for internal pages.

## Structure

```text
.
├── AGENTS.md
├── Claude.md
├── README.md
├── index.md
├── log.md
├── raw/
│   ├── articles/
│   ├── papers/
│   ├── books/
│   ├── transcripts/
│   ├── clips/
│   └── assets/
├── sources/
├── entities/
├── concepts/
├── synthesis/
├── queries/
└── _templates/
    ├── source.md
    ├── entity.md
    ├── concept.md
    ├── synthesis.md
    ├── query.md
    └── note.md
```

## Core Files

- `AGENTS.md`: source of truth for page formats, workflows, and maintenance rules.
- `index.md`: content catalog for sources, entities, concepts, synthesis pages, and saved queries.
- `log.md`: append-only operational history of ingests, queries, and lint runs.
- `Claude.md`: points the coding agent at `AGENTS.md`.

## Page Types

- `sources/*.md`: one page per ingested raw source.
- `entities/*.md`: people, organizations, models, products, or projects.
- `concepts/*.md`: techniques, theories, terms, and architectures.
- `synthesis/*.md`: comparisons, overviews, and evolving theses.
- `queries/*.md`: preserved answers to questions worth keeping.

Every wiki page should include YAML frontmatter and use `[[wikilinks]]` for internal references.

## Workflow

### 1. Ingest a Source

1. Add a source file under `raw/`.
2. Read it and discuss key takeaways.
3. Create a page in `sources/`.
4. Create or update related `entities/` and `concepts/` pages.
5. Update any relevant `synthesis/` pages.
6. Refresh `index.md`.
7. Append the action to `log.md`.

### 2. Query the Wiki

1. Read `index.md` to locate relevant pages.
2. Read the relevant wiki pages first.
3. Answer using citations like `([[Source Title]], [[Entity Name]])`.
4. If the answer is worth preserving, save it under `queries/`.
5. Append the action to `log.md`.

### 3. Lint the Vault

Use a lint pass to look for:

- contradictions
- stale claims
- orphan pages
- missing pages
- broken wikilinks
- missing cross-references
- index drift
- underdeveloped topics

Lint reports belong in `synthesis/` and should also be summarized in `log.md`.

## Status

This template is initialized and ready for its first source ingest.

## Attribution

This template is adapted from ideas shared by Andrej Karpathy in [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).
