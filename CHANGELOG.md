# Changelog

All notable changes to this repository are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the
repository uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] - 2026-11-13

First documentation baseline.

### Added

- `ARCHITECTURE.md` describing the system in C4 levels, runtime flows, data ownership,
  messaging, cross-cutting concepts and deployment.
- `CHANGELOG.md`, this file.
- `CLAUDE.md` with the working agreement for the project.
- `GLOSSARY.md` with domain, messaging, security and observability terms.
- `README.md` with the architecture overview and the documentation index.
- `REQUIREMENTS.md` with functional and non-functional requirements and their IDs.
- ADR-0001: how decisions are recorded.
- ADR-0002: multi-repository layout under one GitHub organization.
- ADR-0003: Kafka for events and RabbitMQ for notification tasks.
- ADR-0004: MongoDB for raw events and Elasticsearch for search.
- ADR-0005: the Grafana stack with Alloy for observability.
- ADR-0006: the API gateway as the single HTTP entry point, with all rate limits in it.
- ADR-0007: the simulator in its own repository.
- ADR-0008: the original payload carried inside the normalized event.

[Unreleased]: https://github.com/SecOpsCore/docs/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/SecOpsCore/docs/releases/tag/v0.1.0
