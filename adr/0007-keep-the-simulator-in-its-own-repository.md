# ADR-0007: Keep the simulator in its own repository

- **Status:** Accepted
- **Date:** 2026-11-13

## Context and problem statement

The simulator generates normal event traffic and labelled attack scenarios. It serves
demos, tests and the training and evaluation data for the anomaly model (FR-SIM-01 to
FR-SIM-03). Architecturally it behaves like an external event source, so it sits
outside the product in the system context.

Where its code lives decides how it is versioned, how visible it is, and how it is kept
in step with the event contracts.

Should the simulator be a separate repository, part of the platform repository, part of
the contracts repository, or spread across the services' tests?

## Decision drivers

- Keep the simulator outside the product's deployable units.
- Make it a visible consumer of the public event contracts.
- Version its generated datasets independently of the services.
- Avoid duplicating event-generation logic in several places.
- Keep the overhead for a single developer proportionate.

## Considered options

1. **A dedicated `simulator` repository.**
2. **Inside the `platform` repository,** as `tools/simulator`.
3. **Inside the `contracts` repository,** as an example consumer.
4. **Inside each service's test code.**

| Criterion                              | 1. Own repository | 2. Platform folder      | 3. Contracts folder    | 4. Service tests |
|----------------------------------------|-------------------|-------------------------|------------------------|------------------|
| Seen as an external source             | Yes               | No, looks like plumbing | Partly                 | No               |
| Consumes contracts as a client         | Yes               | Yes                     | Mixed with definitions | Partly           |
| Independent datasets and releases      | Yes               | No                      | No                     | No               |
| Usable end to end against the platform | Yes               | Yes                     | Yes                    | Hard             |
| Repository overhead                    | One more          | None                    | None                   | None             |

## Decision outcome

Chosen option: **1, a dedicated `simulator` repository.**

The simulator is an actor in the system context, not part of the platform. It consumes
the public event contract the way any real source does. Its labelled attack data is a
reusable artifact for tests and for training the anomaly model, and it benefits from its
own versioning and changelog.

Why the other options lost:

- **Option 2** would make the simulator look like platform infrastructure, which
  understates its role as a client.
- **Option 3** mixes definitions of the contract with a client that changes faster.
- **Option 4** duplicates generation logic across services and cannot run end to end
  against the platform for demos.

## Consequences

**Positive**

- The simulator's status as an external source is explicit.
- It can be pointed at any version of the platform, which is a useful test of contract
  compatibility.
- It shows a contract consumer, a common interview theme.
- Generated datasets have their own release history and can be referenced by the anomaly
  model's documentation.

**Negative**

- One more repository to maintain: README, changelog and pipeline, for a tool that is
  not part of the product.
- It can drift from the contracts. Mitigation: it consumes a versioned contract artifact,
  not a copy of the schemas.
- Its language is undecided. A Python implementation shares tooling with
  `anomaly-service`, and a Java one shares it with the services. Either choice adds a
  dependency to the stack.
- A visitor may read it as a test tool. That is accurate, and the README should say so
  plainly.

## Related

- [ARCHITECTURE.md](../ARCHITECTURE.md), sections 3 and 4.3
- [REQUIREMENTS.md](../REQUIREMENTS.md), FR-SIM-01 to FR-SIM-03
- [ADR-0002](0002-use-a-multi-repository-layout.md), the repository layout this follows
