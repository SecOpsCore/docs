# ADR-0004: Use MongoDB for raw events and Elasticsearch for search

- **Status:** Accepted
- **Date:** 2026-11-13

## Context and problem statement

The platform must keep every accepted event in its original form (FR-STO-01), delete
events after a configurable retention period (FR-STO-02), and let analysts search them
by text and fields, filter by time and source, and see aggregations such as counts over
time (FR-STO-03 to FR-STO-05). Searches over the last 24 hours should answer within
one second at the 95th percentile (NFR-PERF-03).

The events come from different sources, so their payloads differ in shape and change
over time. The original payload is also evidence: it must not be altered. The data used
for searching, on the other hand, can be reshaped and re-created freely.

Which datastore, or datastores, should hold raw events and serve search?

## Decision drivers

- Storing payloads of varying shape without forcing them into a fixed schema.
- Keeping the original event unchanged, separate from derived data.
- Simple retention by age.
- Fast full-text search, field filters and aggregations over time ranges.
- The ability to rebuild the search structures if they are lost or their design
  changes.
- Memory and operational cost on a single developer machine.
- Learning value of working with more than one kind of datastore.

## Considered options

1. **PostgreSQL only.** Events as `jsonb` documents with GIN indexes and built-in
   full-text search.
2. **MongoDB only.** Events as documents, with text indexes and the aggregation
   pipeline.
3. **Elasticsearch only.** One store as both record and search index.
4. **MongoDB for the raw events and Elasticsearch for the search index.**
5. **Kafka as the record and Elasticsearch for search,** with no separate raw store.

| Criterion                              | 1. PostgreSQL only   | 2. MongoDB only    | 3. Elasticsearch only       | 4. MongoDB and Elasticsearch | 5. Kafka and Elasticsearch |
|----------------------------------------|----------------------|--------------------|-----------------------------|------------------------------|----------------------------|
| Flexible payload shape                 | Good (`jsonb`)       | Strong             | Good, but mappings can grow | Strong                       | Good                       |
| Original data kept separate from index | No                   | No                 | No                          | Yes                          | Partly (limited retention) |
| Simple retention by age                | Manual jobs          | Strong (TTL index) | Good (lifecycle rules)      | Strong                       | Topic retention only       |
| Full-text search and relevance         | Basic                | Basic              | Strong                      | Strong                       | Strong                     |
| Aggregations over time at volume       | Medium               | Medium             | Strong                      | Strong                       | Strong                     |
| Rebuild the search structure           | Not needed           | Not needed         | Needs an external source    | Yes, from the raw store      | Only within retention      |
| Datastores to run                      | One (already needed) | One                | One                         | Two                          | One                        |

## Decision outcome

Chosen option: **4, MongoDB for raw events and Elasticsearch for the search index.**
Both are owned by `search-service`, which is the only service that touches them.

- **MongoDB is the record of what arrived.** It stores the original payload together
  with the source and the receive time, in a document model that tolerates any payload
  shape. A TTL index on the receive time implements retention.
- **Elasticsearch is a derived search index.** It holds the normalized events in an
  inverted index built for full-text search, field filters and aggregations. It is a
  read model: if it is lost or its mapping changes, it is rebuilt from MongoDB or by
  replaying the event stream. Elasticsearch is never the only copy of any data.

Why the other options lost, and an honest note on the choice:

- **Option 1** is the simplest and would probably be enough at this project's data
  volume. It loses mainly on search quality and aggregation speed at scale, and because
  it would remove the opportunity to work with the datastores a real SIEM uses.
- **Option 2** has no strong answer to relevance-ranked full-text search or fast
  aggregations.
- **Option 3** is the most realistic for a SIEM, but using Elasticsearch as the only
  system of record is discouraged. A mapping mistake is fixed by re-indexing, and
  re-indexing needs the original data to come from somewhere else.
- **Option 5** has a retention limited to the topic's, and it makes the evidence
  depend on the broker's configuration.

A single store would be simpler to run. The two-store design is justified by the clear
separation between evidence and search, and, openly, by the learning goal of practising
polyglot persistence.

## Consequences

**Positive**

- Raw events are stored unchanged in a flexible model, with retention handled by the
  database itself.
- Search is fast and expressive, and it scales independently of the raw store.
- The search index can be dropped and rebuilt at any time, which makes mapping
  changes and recovery routine.
- The design demonstrates read models, eventual consistency and polyglot persistence.

**Negative**

- Two datastores must be run, secured, backed up and monitored.
- The index lags the raw store slightly. Indexing failures must be retried and
  dead-lettered, and a reconciliation check may be needed to detect drift.
- The same data is stored twice, which costs disk space.
- Elasticsearch uses a lot of memory. Locally it runs as a single node with a capped
  heap, and it is the heaviest component of the platform.
- Retention must be kept in step across both stores: a TTL index in MongoDB and a
  lifecycle rule in Elasticsearch.
- Both products use licences that differ from permissive open source. That is
  acceptable for local, non-commercial use but should be reviewed before any
  commercial use. OpenSearch is a permissively licensed alternative to Elasticsearch.

## Related

- [ARCHITECTURE.md](../ARCHITECTURE.md), sections 6.1 to 6.4
- [REQUIREMENTS.md](../REQUIREMENTS.md), FR-STO-01 to FR-STO-05, NFR-PERF-03
