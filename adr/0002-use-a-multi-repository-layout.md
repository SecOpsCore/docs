# ADR-0002: Use a multi-repository layout

- **Status:** Accepted
- **Date:** 2026-11-13

## Context and problem statement

SecOpsCore consists of about ten deployable units (seven services and a web
dashboard, plus platform components), written in Java, Python and TypeScript. Around
them sit documentation, shared contracts (API specifications and event schemas) and
platform configuration (Helm charts, GitOps definitions, observability setup).

Source control has to support several goals at once. Each service must be built,
tested, versioned and released on its own (NFR-MNT-01). The agreements between
services must be explicit. The project must be easy to present on GitHub. And it is
built and maintained by one person.

How should the code be divided across Git repositories?

## Decision drivers

- Independent deployability and an independent release history per service
  (NFR-MNT-01).
- Contracts and platform configuration treated as first-class, versioned artifacts.
- Semantic Versioning per component, with plain tags such as `v1.2.0`.
- Small, fast CI runs that cover one service only.
- A layout that a reviewer visiting GitHub can understand quickly.
- Low coordination cost for a single developer.
- Learning value: experiencing the real trade-offs of delivering microservices.

## Considered options

1. **One repository, multi-module Maven project.** A parent `pom.xml` lists every
   service as a module.
2. **One repository (monorepo) with independent services.** Each service has its own
   build file and no shared parent. CI runs only for the folders that changed.
3. **Hybrid.** One repository for documentation, contracts and platform, and one
   repository per service.
4. **Multi-repository.** One repository per service, plus separate `docs`,
   `contracts` and `platform` repositories, all in one GitHub organization.

| Criterion                        | 1. Multi-module | 2. Monorepo            | 3. Hybrid | 4. Multi-repository |
|----------------------------------|-----------------|------------------------|-----------|---------------------|
| Independent release history      | Poor            | Medium (prefixed tags) | Good      | Good                |
| Languages other than Java        | Poor            | Good                   | Good      | Good                |
| Explicit contract management     | Weak            | Medium                 | Medium    | Strong              |
| Navigability for a visitor       | Good            | Good                   | Medium    | Needs extra effort  |
| Overhead for one developer       | Low             | Low                    | Medium    | Higher              |
| Realism of microservice practice | Low             | Medium                 | Medium    | High                |

## Decision outcome

Chosen option: **4, multi-repository under one GitHub organization.**

The organization is `SecOpsCore`. Repositories are named without a prefix, because
the organization already gives the context: `docs`, `contracts`, `platform`,
`ingestion-service`, `detection-service`, `incident-service`,
`notification-service`, `search-service`, `anomaly-service` and `dashboard`.
Repositories are created when they are needed, not all at once. The default branch is
`master`.

Why the other options lost:

- **Option 1** couples the services through the parent POM: shared versions, a shared
  build and a shared dependency set. It also covers only Java, so the Angular
  dashboard and the Python service would live outside the model anyway. A reviewer
  can reasonably read that coupling as a weakness in a microservices project.
- **Option 2** is the lowest-friction choice and a legitimate one. It loses because it
  hides the problems this project exists to practise: independent versioning,
  compatibility between separately released components, and one pipeline per service.
  Per-service tags also need prefixes such as `ingestion-service/v1.2.0`.
- **Option 3** is the closest alternative. It loses because contracts and platform
  configuration have different consumers and release rhythms, and putting them in one
  repository with the documentation mixes three different histories.

A GitHub organization with a profile page (the `.github` repository) and a `docs`
repository that links to everything gives a visitor one front door, which reduces the
main cost of this layout.

## Consequences

**Positive**

- Every repository has its own history, its own Semantic Versioning tags and its own
  changelog, so the independence of the services is visible in Git itself.
- CI is small and scoped. A change to one service never triggers builds of others.
- Contracts live in their own repository, so changing an interface is a deliberate,
  versioned act that consumers can adopt on their own schedule.
- Ownership boundaries are clear, and the layout matches how larger teams actually
  work.

**Negative**

- A change that spans services needs several commits in several repositories, in the
  right order. Contracts must change first.
- Some configuration is duplicated: CI workflows, Dockerfile conventions, lint and
  formatting settings. Reusable workflow definitions can reduce this, and the place to
  keep them is still undecided.
- Atomic cross-service refactoring is impossible. Version skew between services is
  possible and has to be managed.
- Each new repository needs its own setup (README, changelog, pipeline), which adds
  overhead for one developer.
- A visitor sees only one repository at a time, so the `docs` repository and the
  organization page have to do the work of presenting the whole.

## Related

- [ARCHITECTURE.md](../ARCHITECTURE.md), sections 2 and 4
- [REQUIREMENTS.md](../REQUIREMENTS.md), NFR-MNT-01
