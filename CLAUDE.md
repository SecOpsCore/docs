# CLAUDE.md: SecOpsCore working agreement

This file defines how Claude works on SecOpsCore. Follow it in every session.

## Project

SecOpsCore is a security monitoring platform (a small SIEM) built by one developer
as a portfolio and learning project. It exists to demonstrate backend engineering,
DevOps and secure development.

- Priority: quality and explanation over speed. Time is not a constraint.
- Explain the trade-offs behind each choice, so the author learns them.
- The project is strictly solo. Do not create open-source community files.

## Repositories

- GitHub organization: `SecOpsCore`.
- Local parent folder: `sec-ops-core`. It is a plain folder, not a Git repository.
- Each repository has its own Git root, and no repository contains another one.
- Repositories: `docs`, `contracts`, `platform`, `ingestion-service`,
  `detection-service`, `incident-service`, `notification-service`,
  `search-service`, `anomaly-service` (Python), `dashboard` (Angular).
- Default branch: `master`.
- Create a repository only when it is needed, in the order a real project would follow.

## Development order

Follow the order a real project would use:

1. Documentation and contracts first.
2. A walking skeleton: one service running end to end, locally.
3. Features, one vertical slice at a time.
4. Platform, observability and DevSecOps hardening, then ML.

## Working method

- Write one document or change at a time. Do not start the next one until the user says so.
- Before writing a document, confirm its scope and check where it fits in the README index.
- After writing a document, update the README documentation index (status `Planned` to `Done`).
  Update any other document that references it, as described under "Keeping documents consistent".
- Give the user the file as a download, the exact placement path, and the exact Git commands.
- Use `git status` before committing. Stage named files only, never `git add .`.
- Never commit IDE or OS files (`.idea/`, `*.iml`, `.DS_Store`). Keep `.gitignore` current.
- Do not create empty placeholder folders for planned documents.
- When a decision is structural or hard to reverse, propose it and wait for approval.

## Commits

Use Conventional Commits: `type: imperative summary`, lowercase, no trailing period.

- Types: `docs`, `feat`, `fix`, `chore`, `test`, `build`, `ci`, `refactor`, `perf`.
- Each incremental change gets its own commit.
- A document change and its README index update are two separate commits.
- Add a body when the reason is not obvious from the summary.
- When implementing a requirement, reference its ID in the body, for example `FR-DET-03`.
- When Claude creates a commit itself, add the `Co-Authored-By` trailer that the session specifies.

Examples:

```
docs: add ADR-0001 record architecture decisions
docs: mark REQUIREMENTS as done in README index
docs: add requirements
```

## Versioning and tags

- Semantic Versioning, applied to each repository independently.
- Claude decides when a commit is ready and proposes when to tag, with the reasoning.
  Never create a tag without the user's approval.
- The `docs` repository is tagged `v0.1.0` when the first coherent set exists:
  README, REQUIREMENTS, ARCHITECTURE and the first ADRs.
- Repositories with a `CHANGELOG.md` follow Keep a Changelog. Add each behavior or
  documentation change to `Unreleased` in the same change.

## Documentation rules

Diagrams:
- Mermaid only. Never ASCII art.
- Use `flowchart LR` for data flow. Use C4 for architecture, and sequence diagrams for interactions.
- Node shapes: stadium for external actors and events, rectangle for services,
  cylinder for brokers and datastores.
- Separate node declarations from relationships: declare all nodes first, then define all connections in
  a separate section, styling last. Never combine node declarations and relationships on the same line.
- Palette classes: `source` (blue), `pipeline` (gray), `messaging` (amber),
  `storage` (teal), `processing` (violet), `output` (green), `frontend` (pink).
  Reuse the same class for the same role in every diagram.
- Overview diagrams show roles only. Do not put topic names or technology names on the
  nodes. Topic names belong in `docs/EVENT_CATALOG.md`. Technologies belong in `ARCHITECTURE.md`.

Writing:
- Headings in sentence case, without trailing punctuation.
- Tables for inventories and status. Prose for reasoning. Pad every table cell with trailing
  spaces so that the columns line up in a plain-text editor too.
- Link to an ADR for every decision, rather than explaining it inline.
- Explain concepts for the learner: what it is, why it is used here, and the trade-off.

Architecture Decision Records:
- Location: `docs/adr/NNNN-kebab-case-title.md` in the `docs` repository for
  cross-repository decisions. Single-service decisions go in that service's `docs/adr/`.
- Numbers are four digits, sequential, and never reused.
- Sections, in order: title, status and date, context and problem statement,
  decision drivers, considered options, decision outcome, consequences.
- Consequences must include honest negatives.
- Once accepted, an ADR's content is never rewritten. Only its status changes, for
  example to `Superseded by ADR-0012`. A new decision gets a new ADR that links to the old one.

Requirements:
- Functional: `FR-<AREA>-NN`, for example `FR-DET-03`.
- Non-functional: `NFR-<CATEGORY>-NN`, for example `NFR-SEC-02`.
- IDs are never reused. A removed requirement is marked withdrawn, not deleted.
- Priorities use MoSCoW: Must, Should, Could.

## Keeping documents consistent

When a change affects other documents, update them in separate commits, and list
them in your reply. Examples:

- New document: add a row to the README documentation index.
- Changed requirement: update `REQUIREMENTS.md`, and any architecture document or ADR that cites it.
- New or changed service: update the README repositories table.
- New technology decision: write an ADR, and add the technology to the README table if it is new.
- Milestone or phase progress: update `ROADMAP.md`.
- Event topic or schema change: update `contracts` first, then `docs/EVENT_CATALOG.md`.

## Scope

Excluded from this project: `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SUPPORT.md`,
`SECURITY.md`, issue and pull request templates, and `CODEOWNERS`.

Included: `docs/learning/` concept notes for each major technology.

## Current state

Update this section whenever progress changes.

As of 2026-11-13:
- GitHub organization `SecOpsCore` exists, with a `.github` profile placeholder.
- `docs` repository: `README.md`, `docs/adr/0001-record-architecture-decisions.md` and
  `REQUIREMENTS.md` are done and pushed. `GLOSSARY.md` is drafted and awaits commit confirmation.
- Next: `ARCHITECTURE.md`, then ADRs for decisions already made:
  multi-repository layout, Kafka plus RabbitMQ, MongoDB plus Elasticsearch, and the Grafana stack.
- No service code exists yet.
