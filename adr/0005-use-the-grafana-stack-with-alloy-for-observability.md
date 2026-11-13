# ADR-0005: Use the Grafana stack with Alloy for observability

- **Status:** Accepted
- **Date:** 2026-11-13

## Context and problem statement

Every service must expose health and metrics, write structured logs with a trace
identifier, and be traceable across services and message brokers (NFR-OBS-01 to
NFR-OBS-03). The platform must also define service level objectives and alert when
they are violated (NFR-OBS-04).

This decision concerns the **telemetry of the platform itself**: metrics, logs and
traces produced by the services and their infrastructure. It does not concern the
security events that the product processes. Those are stored and searched as described
in ADR-0004.

Which observability stack should collect, store and show the platform's telemetry?

## Decision drivers

- Metrics, logs and traces in one place, with easy movement between them.
- Vendor-neutral instrumentation through OpenTelemetry.
- A small memory and operational footprint on a single developer machine.
- Good integration with Spring Boot (Micrometer and OpenTelemetry) and with Kubernetes.
- Free and self-hosted.
- Learning value and relevance to current industry practice.

## Considered options

1. **Elastic Stack (ELK).** Elasticsearch, Logstash or Beats, Kibana, with APM for
   traces.
2. **Grafana stack with Alloy.** Prometheus for metrics, Loki for logs, Tempo for
   traces and Grafana for dashboards, with Alloy as the single collector.
3. **Grafana stack with separate agents.** The same backends, with one agent per signal
   (for example Promtail for logs). Promtail has been deprecated in favour of Alloy.
4. **A hosted commercial service** (for example Datadog or Grafana Cloud).

| Criterion                            | 1. Elastic Stack      | 2. Grafana stack with Alloy | 3. Grafana with separate agents | 4. Hosted service |
|--------------------------------------|-----------------------|-----------------------------|---------------------------------|-------------------|
| Metrics, logs and traces in one view | Good                  | Strong                      | Strong                          | Strong            |
| OpenTelemetry fit                    | Good                  | Strong                      | Medium                          | Strong            |
| Memory footprint locally             | High                  | Moderate                    | Moderate                        | None locally      |
| Full-text log search                 | Strong                | Limited (label-based)       | Limited (label-based)           | Strong            |
| Collectors to configure              | Several               | One                         | Several                         | One               |
| Free and self-hosted                 | Yes (licence caveats) | Yes                         | Yes                             | No                |

## Decision outcome

Chosen option: **2, the Grafana stack with Alloy as the single collector.**

| Component    | Role                                                                 |
|--------------|----------------------------------------------------------------------|
| Alloy        | Scrapes metrics, reads container logs and receives traces over OTLP. |
| Prometheus   | Stores metrics and evaluates alert rules.                            |
| Loki         | Stores logs, indexed by labels.                                      |
| Tempo        | Stores traces.                                                       |
| Grafana      | Dashboards and exploration across all three signals.                 |
| Alertmanager | Routes alerts from Prometheus.                                       |

The deciding factors are the small footprint, the single vendor-neutral collector and
the quality of the links between signals: from a metric panel to a trace, and from a
trace to the logs that carry its identifier. Alloy replaces a collection of
per-signal agents with one configuration.

Why the other options lost:

- **Option 1** is a strong stack and a very valuable skill, but it is heavier to run
  next to Kafka, MongoDB and an Elasticsearch node that already serves the product.
  Using one Elasticsearch deployment for both platform logs and product events would
  also mix two concerns.
- **Option 3** uses the same backends but keeps several agents, one of which is
  deprecated.
- **Option 4** is outside the constraint of free, self-hosted tools running locally.

**Boundary rule.** Elasticsearch is a datastore of the product and holds security
events only. Platform logs go to Loki. The two are not mixed.

## Consequences

**Positive**

- A lighter footprint than an Elastic-based stack, which matters on one machine.
- One collector and one dashboard tool for all three signals, with correlation through
  the trace identifier.
- Instrumentation stays vendor-neutral, so the backends can be replaced without
  changing the services.
- Prometheus and its query language are widely used in Kubernetes environments.

**Negative**

- Log search is label-based and weaker than full-text search in Elasticsearch. Finding
  a rare string across all logs is slower and less convenient.
- Three query languages and an Alloy configuration language must be learned.
- Several components run side by side: Alloy, Prometheus, Loki, Tempo, Grafana and
  Alertmanager.
- A single local Prometheus has no high availability and keeps data on local disk.
- The project then contains two search technologies, Elasticsearch for events and Loki
  for logs. The boundary rule above exists to keep that clear.

## Related

- [ARCHITECTURE.md](../ARCHITECTURE.md), section 8.3
- [REQUIREMENTS.md](../REQUIREMENTS.md), NFR-OBS-01 to NFR-OBS-04
- [ADR-0004](0004-use-mongodb-for-raw-events-and-elasticsearch-for-search.md)
