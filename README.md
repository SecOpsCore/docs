# SecOpsCore Documentation

Central documentation for **SecOpsCore**, a security monitoring platform
(a small SIEM) built to practice and demonstrate modern backend engineering,
DevOps and secure software development.

This repository holds the architecture, the design decisions, the security
documentation and the operational runbooks. Each service lives in its own
repository and keeps only short, service-specific docs that link back here.

## What SecOpsCore does

SecOpsCore ingests security events, stores and indexes them, detects threats
and anomalies, raises alerts and lets an analyst triage them as incidents.

```mermaid
flowchart LR
    SRC(["📥 Security sources"])
    ING["⚙️ Ingestion service"]
    KFK[("⚡ Event stream")]
    ML["🤖 Anomaly scoring"]
    TI["🛡️ Detection"]
    HUM(["👤 Security analyst"])
    ALR(["🚨 Alerts"])
    INC["📋 Incidents"]
    NOT["📨 Notifications"]
    DSH["📊 Dashboard"]
    DB[("🔎 Event storage<br/>and search")]
    
    SRC --> ING
    ING --> KFK
    KFK --> ML
    KFK --> TI
    KFK --> HUM
    KFK --> DB
    ML --> ALR
    TI --> ALR
    ALR --> INC
    INC --> NOT
    HUM --> DSH
    DSH --> INC
    DSH --> DB

    classDef source fill:#d9f0ff,stroke:#0088cc,color:#003b66
    classDef pipeline fill:#f1f5f9,stroke:#64748b,color:#1e293b
    classDef stream fill:#fff0d9,stroke:#ff8800,color:#663300
    classDef detection fill:#f2e6ff,stroke:#9966ff,color:#4b2680
    classDef alert fill:#e0ffe0,stroke:#00aa44,color:#005522
    classDef dashboard fill:#ffe6f0,stroke:#ff3388,color:#881144
    classDef storage fill:#ccfff5,stroke:#00aa99,color:#005544

    class SRC,HUM source
    class ING pipeline
    class KFK stream
    class ML,TI detection
    class ALR,INC,NOT alert
    class DSH dashboard
    class DB storage
```

## Technology overview

| Area                         | Technologies                                                                         |
|------------------------------|--------------------------------------------------------------------------------------|
| Backend services             | Java, Spring Boot                                                                    |
| CI/CD and DevSecOps          | GitHub Actions, SAST, dependency and container scanning, DAST                        |
| Containers and orchestration | Docker, Kubernetes (local with kind), Helm, Argo CD                                  |
| Data                         | PostgreSQL (relational), MongoDB (raw events), Elasticsearch (search), Redis (cache) |
| Frontend                     | Angular                                                                              |
| Machine learning             | Python, FastAPI, scikit-learn (anomaly detection)                                    |
| Messaging                    | Apache Kafka (event stream), RabbitMQ (notifications)                                |
| Identity and secrets         | Keycloak (OIDC/OAuth2), HashiCorp Vault                                              |
| Observability                | Micrometer, Prometheus, Grafana, Loki, Tempo, Alloy, OpenTelemetry                   |

The reasoning behind each choice is recorded in the Architecture Decision
Records (ADRs) under `docs/adr/`.

## Repositories

| Repository             | Purpose                                           | Status      |
|------------------------|---------------------------------------------------|-------------|
| `anomaly-service`      | ML-based anomaly detection                        | Planned     |
| `contracts`            | OpenAPI specifications and event schemas          | Planned     |
| `dashboard`            | Angular web interface                             | Planned     |
| `detection-service`    | Rule-based threat detection                       | Planned     |
| `docs`                 | Central documentation (this repository)           | In progress |
| `incident-service`     | Alerts and incident management                    | Planned     |
| `ingestion-service`    | Receives and validates security events            | Planned     |
| `notification-service` | Alert delivery                                    | Planned     |
| `platform`             | Helm charts, Argo CD, observability configuration | Planned     |
| `search-service`       | Event search and analytics                        | Planned     |

## Documentation index

| Document                                    | Description                                                  | Status  |
|---------------------------------------------|--------------------------------------------------------------|---------|
| `./API_GUIDELINES.md`                       | REST conventions                                             | Planned |
| `./ARCHITECTURE.md`                         | System context, containers and data flow                     | Planned |
| `./CODING_STANDARDS.md`                     | Conventions per language                                     | Planned |
| `./DATA_MODEL.md`                           | Data models per datastore                                    | Planned |
| `./DEPLOYMENT.md`                           | Kubernetes and GitOps flow                                   | Planned |
| `./DEVELOPMENT.md`                          | Local development setup                                      | Planned |
| `./EVENT_CATALOG.md`                        | Kafka topics and event schemas                               | Planned |
| `./GIT_WORKFLOW.md`                         | Branching, commits, versioning and tagging                   | Planned |
| `./GLOSSARY.md`                             | Domain terms                                                 | Planned |
| `./INCIDENT_RESPONSE.md`                    | Incident handling and postmortems                            | Planned |
| `./OBSERVABILITY.md`                        | Metrics, logs and traces                                     | Planned |
| `./README.md`                               | This file                                                    | Done    |
| `./REQUIREMENTS.md`                         | Functional and non-functional requirements                   | Done    |
| `./ROADMAP.md`                              | Phases and milestones                                        | Planned |
| `./SLOs.md`                                 | Service level objectives                                     | Planned |
| `./TESTING_STRATEGY.md`                     | Test pyramid and tooling                                     | Planned |
| `adr/0001-record-architecture-decisions.md` | Why and how decisions are recorded                           | Done    |
| `learning/`                                 | Concept notes on the technologies used                       | Planned |
| `ml/`                                       | Model card, datasets and evaluation                          | Planned |
| `runbooks/`                                 | Operational runbooks                                         | Planned |
| `security/`                                 | Threat model, secure coding, DevSecOps, secrets, authN/authZ | Planned |

## Conventions

- **Default branch:** `master`.
- **Commits:** [Conventional Commits](https://www.conventionalcommits.org/),
  one commit per incremental change.
- **Versioning:** [Semantic Versioning](https://semver.org/). Each repository
  is versioned and tagged independently.
- **Changelog:** [Keep a Changelog](https://keepachangelog.com/) format.
- **Diagrams:** Mermaid only, no ASCII art. The style rules are defined in
  `docs/GIT_WORKFLOW.md` and `ARCHITECTURE.md` once they exist.

The full details will live in `docs/GIT_WORKFLOW.md`.

## Project status

The project is in its documentation phase. No service code exists yet.
