# ADR-0003: Use Kafka for events and RabbitMQ for notifications

- **Status:** Accepted
- **Date:** 2026-11-13

## Context and problem statement

The platform moves two different kinds of messages.

1. **Event data.** A high volume of security events and the alerts derived from them.
   Several services need the same data independently (detection, anomaly scoring,
   search, incident handling). Order matters per entity. A service that was down, or a
   search index that has to be rebuilt, must be able to read the data again.
2. **Notification tasks.** A small number of messages, each of which exactly one
   worker must complete. A failed task needs retries with a delay and a place to park
   it if it keeps failing (FR-NOT-02).

Which messaging technology, or technologies, should carry these two kinds of traffic?

## Decision drivers

- Throughput of at least 1,000 events per second on a laptop (NFR-PERF-01).
- No loss of an accepted event, and tolerance of duplicate delivery (NFR-REL-01,
  NFR-REL-02).
- Ingestion must keep working when a downstream service is down (NFR-REL-03).
- Parallel processing without losing per-entity order (NFR-SCA-02).
- Replay: a new or recovered consumer reads past events.
- Windowed processing for detection rules.
- Per-message acknowledgement, delayed retry and dead-lettering for tasks.
- Operational weight on a single developer machine.
- Learning value, including choosing the right tool for each job.

## Considered options

1. **Kafka only.** Events and notification tasks both go through Kafka. Notification
   retries use retry topics and a dead-letter topic.
2. **RabbitMQ only.** Events and tasks both go through RabbitMQ queues. RabbitMQ
   stream queues can give log-like behaviour.
3. **Kafka for events and RabbitMQ for notification tasks.**
4. **A lighter broker for everything** (for example NATS JetStream or Redis Streams).

| Criterion                                     | 1. Kafka only          | 2. RabbitMQ only                       | 3. Kafka and RabbitMQ | 4. Lighter broker |
|-----------------------------------------------|------------------------|----------------------------------------|-----------------------|-------------------|
| Replay and several independent readers        | Strong                 | Possible with stream queues            | Strong                | Possible          |
| Per-key order with parallel consumers         | Strong (partitions)    | Needs plugins or a single consumer     | Strong                | Varies            |
| Windowed stream processing                    | Strong (Kafka Streams) | No built-in equivalent                 | Strong                | Limited           |
| Per-message acknowledgement and delayed retry | Awkward (retry topics) | Strong (TTL and dead-letter exchanges) | Strong                | Medium            |
| Components to operate                         | One                    | One                                    | Two                   | One               |
| Fit to each workload                          | Good for events only   | Good for tasks only                    | Good for both         | Adequate for both |

## Decision outcome

Chosen option: **3, Kafka for events and alerts, RabbitMQ for notification tasks.**

The boundary rule is simple and applies to every future message:

> Data that several services need, in order, and that must be replayable goes through
> **Kafka**. A task that exactly one worker must complete, with retries and a place for
> failures, goes through **RabbitMQ**.

Kafka is the backbone of the data plane. It gives a durable, ordered, partitioned log
that any number of consumer groups read at their own pace. It also makes Kafka Streams
available for the windowed rules in `detection-service`, and it makes the search
index rebuildable by replay. RabbitMQ handles the notification path: per-message
acknowledgement, delayed retries through time-to-live queues and dead-letter
exchanges, and a parking queue for messages that keep failing.

Why the other options lost:

- **Option 1** works, and it is the simplest operationally. But delayed retry of a
  single task is awkward on a log, because a retry topic is a delay mechanism rather
  than a per-message feature. It also leaves RabbitMQ's model, the standard answer to
  task queues, unused.
- **Option 2** handles notifications very well but has no natural counterpart to
  partition-keyed ordering with parallel consumers or to windowed stream processing,
  which the detection rules depend on.
- **Option 4** reduces operational weight but gives up the Kafka ecosystem, which is
  the most widely used event streaming platform and the most valuable to practise.

## Consequences

**Positive**

- Each workload gets the tool designed for it, and the boundary between them is easy
  to state and to test.
- Events are retained after consumption, so a recovered service catches up and the
  search index can be rebuilt by replay.
- Detection rules can use windowed, stateful stream processing with exactly-once
  semantics inside Kafka Streams.
- Notification delivery gets proper retry, backoff and dead-lettering without custom
  code.
- The project demonstrates both messaging models and the reasoning for choosing
  between them.

**Negative**

- Two brokers must be run, secured, monitored and upgraded, and both consume memory on
  a developer machine.
- Developers need two client libraries and two mental models.
- Delivery is at-least-once in both systems, so every consumer must be idempotent.
- Moving a message from the database to RabbitMQ needs the outbox pattern to avoid a
  gap between a committed change and a published message.
- A single local Kafka broker is a single point of failure and cannot demonstrate
  replication.
- The boundary can blur over time if new message types are placed by habit instead of
  by the rule.

If RabbitMQ turns out to carry only the notification path and its operating cost
dominates, a later ADR can supersede this one with a design that uses Kafka retry
topics instead.

## Related

- [ARCHITECTURE.md](../ARCHITECTURE.md), sections 4.4 and 7
- [REQUIREMENTS.md](../REQUIREMENTS.md), NFR-PERF-01, NFR-REL-01 to NFR-REL-03,
  NFR-SCA-02, FR-NOT-02
