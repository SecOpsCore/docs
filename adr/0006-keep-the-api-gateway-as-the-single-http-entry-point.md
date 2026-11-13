# ADR-0006: Keep the API gateway as the single HTTP entry point

- **Status:** Accepted
- **Date:** 2026-11-13

## Context and problem statement

Two kinds of client reach the platform over HTTP: the analyst's dashboard and external
event sources. Both need TLS, token checks, rate limiting and routing to the right
service.

The first plan included a gateway, `api-gateway`, for these concerns. The architecture
draft proposed removing it in favour of a plain ingress with every service validating
tokens itself. This record decides to keep the gateway, as originally planned.

Should the platform keep a dedicated gateway service in front of its APIs, or rely on
the ingress and per-service checks?

## Decision drivers

- Authentication enforced in one place before any work reaches a service.
- Coarse rate limiting per client, before a request costs anything downstream.
- Routing and cross-cutting policy (headers, request limits, logging) defined once.
- Independent deployability: the gateway is its own repository and deployable unit.
- Rate limits per client, which also covers the per-source limit for event sources
  (FR-ING-06).
- Local footprint on one developer machine.
- Learning value: a gateway is a common interview topic.

## Considered options

1. **Ingress only.** The ingress terminates TLS and routes. Each service validates tokens
   and applies its own limits.
2. **Spring Cloud Gateway as the single HTTP entry point,** behind the ingress. It checks
   tokens, applies rate limits per client and routes. Services still validate and authorize.
3. **Gateway for the dashboard only.** Event sources reach ingestion directly through the
   ingress, with their own checks.
4. **A service mesh ingress gateway** with authentication and rate-limit policies.

| Criterion               | 1. Ingress only | 2. Gateway for all HTTP | 3. Gateway for dashboard | 4. Mesh gateway |
|-------------------------|-----------------|-------------------------|--------------------------|-----------------|
| Central token check     | No              | Yes                     | Partly                   | Yes             |
| Rate limits per client  | Limited         | Yes                     | Partly                   | Yes             |
| Later per-source quotas | Hard            | Easier                  | Medium                   | Easier          |
| Validation paths        | One per service | One at the edge         | Two, which must agree    | One at the edge |
| Local footprint         | Low             | Medium                  | Medium                   | High            |
| Learning value          | Low             | High                    | Medium                   | Medium          |

## Decision outcome

Chosen option: **2, Spring Cloud Gateway as the single HTTP entry point, behind the
ingress.**

- The **ingress** terminates TLS and routes traffic into the cluster.
- The **api-gateway** checks the token, applies the rate limit for the calling client
  and routes the request to the right service.
- **All rate limiting lives in the gateway.** The limit is keyed by the client identity
  in the token. Each event source is its own client, so the same mechanism is the
  per-source limit (FR-ING-06). Ingestion has no rate-limit logic and no Redis.
- **Every service still validates the token and enforces roles** (FR-IAM-03). The
  gateway is an early rejection point, not the authorization decision.

Why the other options lost:

- **Option 1** spreads policy across every service. Each one must implement the same
  checks, and a fix has to be made in every repository.
- **Option 3** gives two validation paths, one for sources and one for users, which
  must be kept in agreement. Two paths double the risk of a gap.
- **Option 4** is powerful, but a mesh adds a large footprint and a separate subject to
  learn. It remains a possible later topic (mTLS is still open, section 11 of
  ARCHITECTURE.md).

## Consequences

**Positive**

- External policy is visible and changeable in one repository.
- Rate limits stop abusive traffic before it reaches the services, and they live in
  one place.
- The security story is clear: one edge check, and a second check in each service.
- The gateway gives a concrete, interview-ready component to explain.

**Negative**

- One extra hop adds latency to every request.
- One more service to build, run, scale and monitor.
- If the gateway fails, external access stops. Two replicas reduce the risk, and the
  gateway holds no state.
- Tokens are checked twice, which costs a little CPU and needs signing keys in each
  service.
- Limits keyed by client identity require every event source to have its own client
  registration in Keycloak.
- Spring Cloud Gateway's reactive model has its own learning curve.

## Related

- [ARCHITECTURE.md](../ARCHITECTURE.md), sections 4.2 and 8.1
- [REQUIREMENTS.md](../REQUIREMENTS.md), FR-IAM-03, FR-ING-04, FR-ING-06, NFR-SEC-05
- [ADR-0008](0008-carry-the-original-payload-inside-the-normalized-event.md), the event that
  ingestion publishes behind the gateway
