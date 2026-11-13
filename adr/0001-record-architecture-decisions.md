# ADR-0001: Record architecture decisions

- **Status:** Accepted
- **Date:** 2026-11-13

## Context and problem statement

SecOpsCore is made of many repositories and many technology choices:
messaging, datastores, observability, security tooling and deployment. Each
choice was made for specific reasons, against specific alternatives, under
specific constraints.

Reasons fade quickly. Without a written record, a later reader (including the
author, months later) can see *what* was chosen but not *why*, and cannot tell
whether a decision is still valid or was a deliberate trade-off. This is
especially costly in a project whose purpose is to demonstrate and learn from
engineering decisions.

How should significant decisions be recorded so that the reasoning stays
visible, reviewable and tied to the code history?

## Decision drivers

- The reasoning must live next to the code and be versioned with it.
- A decision must be easy to find, link to and cite from other documents.
- History must be preserved: a changed decision must not erase the old one.
- The format must be light enough to be written for every significant decision.
- The format should be a recognised convention, not a custom one.

## Considered options

1. **No formal record.** Rely on commit messages, code and memory.
2. **A wiki or external tool.** Write decisions in a separate system.
3. **A single "decisions" document** in the repository, edited over time.
4. **One Architecture Decision Record (ADR) per decision**, as a Markdown file
   in the repository.

## Decision outcome

Chosen option: **4, one ADR per decision, stored in Markdown in the
repository**, using a format based on
[MADR](https://adr.github.io/madr/) (Markdown Architectural Decision Records).

Option 1 loses the reasoning. Option 2 separates the reasoning from the code
and its history. Option 3 invites rewriting, which destroys the record of what
was believed at the time. Option 4 keeps each decision as a small, immutable,
linkable unit with its own status and history.

## Consequences

**Positive**

- The reasoning behind each decision is preserved and reviewed through the same
  commits as the code.
- Decisions can be referenced precisely (`ADR-0004`) from documents, pull
  requests and code comments.
- A reversed decision leaves a trail: the old record is marked superseded and
  points to its replacement.
- Writing the record forces the alternatives to be named and compared.

**Negative**

- Every significant decision costs some writing time.
- Records can become stale if their status is not maintained.
- Deciding what counts as "significant" requires judgement.

## Conventions

### Location and naming

- ADRs live in `docs/adr/` of the repository they affect.
- Decisions that span several repositories live in the central `docs`
  repository. Decisions limited to one service live in that service's
  repository.
- File names follow `NNNN-title-in-kebab-case.md`, for example
  `0004-use-kafka-for-the-event-stream.md`.
- Numbers are sequential, zero-padded to four digits, and **never reused**,
  even if a record is later rejected or superseded.

### Format

Each record contains these sections, in this order:

1. **Title:** `# ADR-NNNN: Short imperative title`
2. **Status and date**
3. **Context and problem statement:** the situation and the forces at play
4. **Decision drivers:** the criteria that matter
5. **Considered options:** the realistic alternatives, including doing nothing
6. **Decision outcome:** the chosen option and why the others lost
7. **Consequences:** positive and negative results, stated honestly

### Status lifecycle

| Status | Meaning |
|---|---|
| Proposed | Written, not yet decided |
| Accepted | Decided and in force |
| Rejected | Considered and not adopted (kept for the record) |
| Deprecated | No longer relevant, with no replacement |
| Superseded by ADR-NNNN | Replaced by a newer decision |

### Immutability

Once an ADR is accepted, its content is not rewritten. Only the status line
changes, for example to `Superseded by ADR-0012`. If the decision changes, a new
ADR is written that links back to the old one.

### When to write one

Write an ADR when a decision is hard to reverse, affects several components, or
has real alternatives worth comparing. Examples: the choice of a datastore, a
messaging technology, an authentication model, the repository structure. Do not
write one for routine choices, such as a library version or a naming detail.
