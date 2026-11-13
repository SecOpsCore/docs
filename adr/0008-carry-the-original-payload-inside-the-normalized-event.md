# ADR-0008: Carry the original payload inside the normalized event

- **Status:** Accepted
- **Date:** 2026-11-13

## Context and problem statement

Every accepted event must keep its original form (FR-STO-01). Downstream services work
on a normalized form of the event: detection evaluates it, and search indexes it.
`search-service` stores the original payload in MongoDB (ADR-0004), so it needs the
original as well as the normalized fields.

How should the original payload travel from ingestion to the service that stores it?

## Decision drivers

- No accepted event may be lost or half-stored (NFR-REL-01).
- Ingestion keeps working when downstream services are down (NFR-REL-03).
- Each datastore keeps a single owner (database per service).
- Few moving parts, and no new problems that need extra machinery.
- Learning value of a well-known event design pattern.

## Considered options

1. **Carry the original payload inside the normalized event.** One message per event
   (event-carried state transfer).
2. **Publish the original and the normalized event as two messages.**
3. **Store the original first in a raw store owned by ingestion, then publish the
   normalized event with a reference** (the claim check pattern).
4. **Have ingestion call search-service synchronously** to store the original.

| Criterion                  | 1. Inside the event | 2. Two messages     | 3. Reference                                                 | 4. Synchronous call                          |
|----------------------------|---------------------|---------------------|--------------------------------------------------------------|----------------------------------------------|
| Atomic publish             | Yes                 | No                  | Ordered writes, orphans possible                             | No                                           |
| Ingestion depends on       | Kafka only          | Kafka only          | Kafka and the raw store                                      | Kafka and search-service                     |
| Ownership of the raw store | search-service      | search-service      | Moves to ingestion                                           | Broken: ingestion writes into search's store |
| Copies of the payload      | One extra, in Kafka | One extra, in Kafka | None extra                                                   | None extra                                   |
| Extra machinery needed     | None                | Joining two streams | Read API, retention coupling, shared normalizer for rebuilds | None, but tight coupling                     |

## Decision outcome

Chosen option: **1, carry the original payload inside the normalized event.**

Ingestion validates and normalizes the event, then publishes one message that holds the
normalized fields and the original payload unchanged. `search-service` stores the
original in MongoDB and indexes the normalized fields in Elasticsearch. Nothing else in
the system needs to know where the original lives.

Why the other options lost:

- **Option 2** is not atomic. One message can be published without the other, and
  consumers would have to join two streams.
- **Option 3** avoids the extra copy, and it is a sound pattern for large payloads. At
  this project's scale it adds more than it saves: a second datastore in the ingestion hot
  path, a change of owner for the raw store, orphaned payloads after a failed publish,
  retention that must be kept equal between stores, a read API for originals, and a shared
  normalizer to rebuild the search index. It was briefly adopted and then reversed before
  being committed, for these reasons.
- **Option 4** puts a synchronous dependency on search in the ingestion path, so a search
  outage would stop ingestion, and it breaks data ownership.

## Consequences

**Positive**

- One atomic publish per event. An event is either fully accepted or not at all.
- Ingestion depends only on Kafka, and stays stateless.
- Ownership stays clean: `search-service` owns both MongoDB and Elasticsearch, as ADR-0004
  decided.
- The search index can be rebuilt from MongoDB, which holds both the original and what is
  needed to normalize it again.
- The design is simple to explain and to test.

**Negative**

- Messages are larger, roughly double the normalized event alone. At the target of 1,000
  events per second this is well within a local broker's capacity, but it costs disk for
  the 7-day retention.
- Every consumer receives the original payload, even those that ignore it.
- Unmasked originals sit in Kafka for its retention period, in addition to MongoDB. Kafka
  is in the data zone and topics are readable only by their consumers, but the exposure is
  real and must be stated in the threat model.
- If payloads ever become very large, this decision should be revisited, and the claim
  check pattern (option 3) is the natural replacement.

## Related

- [ADR-0004](0004-use-mongodb-for-raw-events-and-elasticsearch-for-search.md), the stores
  this event feeds
- [ARCHITECTURE.md](../ARCHITECTURE.md), sections 5.1, 6, 7.2 and 8.1
- [REQUIREMENTS.md](../REQUIREMENTS.md), FR-STO-01, NFR-REL-01, NFR-REL-03, NFR-SEC-08
