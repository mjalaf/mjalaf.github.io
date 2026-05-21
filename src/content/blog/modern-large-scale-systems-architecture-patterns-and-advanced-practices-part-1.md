---
title: 'Modern Large-Scale Systems: Architecture, Patterns, and Advanced Practices - Parte 1: Fundamentos'
description: Este artículo explora el diseño y arquitectura de sistemas capaces de manejar millones de solicitudes diarias, abordando patrones, almacenamiento global, procesamiento de eventos y big data para lograr alta escalabilidad y disponibilidad.
pubDate: 2026-01-10
author: Martin Jalaf
tags:
- english
- arquitectura-software
- escalabilidad
- alta-disponibilidad
- big-data
- procesamiento-de-eventos
- almacenamiento-global
- patrones-arquitectonicos
series: 'Modern Large-Scale Systems: Architecture, Patterns, and Advanced Practices'
series_part: 1
series_total: 4
---

_Series "Modern Large-Scale Systems: Architecture, Patterns, and Advanced Practices" - Part 1 of 4_

## Table of Contents

- Introduction and context of large-scale systems
- Architectural patterns for high scalability and availability
- Designing global-scale data storage
- Strategies for continuous processing of infinite event streams
- Architectural patterns for big data
- Practical examples and case studies
- Limitations, trade-offs, and common challenges
- Comparison with alternative architectures
- Final recommendations and best practices

## Introduction and context of large-scale systems

"Large-scale systems" are applications and platforms designed to operate with high traffic volumes, large data volumes, and strict latency and availability requirements over long periods. It is not just about handling heavy load: it involves resilience to distributed failures, automated operation, continuous growth, and cost optimization. In practice, we are talking about platforms that process millions of daily events or requests, serve hundreds of thousands (or millions) of concurrent users, and accept complex links between services and data at a global scale.

### What characterizes a modern large-scale system?

- Distributed architecture: microservices, managed services, or event-based architectures that separate responsibilities and limit failure domains.
- Automated operation: deployments (CI/CD), auto-scaling, self-healing, and centralized configuration management.
- End-to-end observability: metrics, distributed tracing, and correlatable logs that allow diagnosing problems in real time.
- Designed for failures: tolerance to network partitions, controlled degradation, and circuit breakers.
- Partitioned data and understood consistency: sharding, geo-distributed replication, and consistency models chosen per use case.

### Main technical and operational challenges

- Load volatility: short spikes (flash crowds) that multiply average load. Example: 10M requests/day ≈ 116 rps average; with peak factor 10 => ~1.1k rps.
- Tail latency: queues and contention cause p99/p999 latencies to grow much more than the median; user experience is measured at those percentiles.
- Data fragmentation and coordination: partitioning, rebalancing, and maintenance operations without visible impact.
- Cascading failures and dependencies: latencies and errors in transit services can bring down consumers.
- Operational complexity: upgrades, schema migrations, resilience testing, and the human cost of 24/7 operation.

### Why design for scalability and high availability from the start

The cost of re-architecting a live system grows exponentially with its usage and data volume. Designing from the beginning with separation of responsibilities, contracts (APIs/Schemas), observability, and automation reduces technical debt and limits risk windows. Moreover, many problems (partitions, queue latency, regional failures) do not appear until a certain size — anticipating them enables practices such as failure testing (chaos engineering), canary deployments, and realistic SLOs.

### Relevant metrics and performance objectives

- Throughput: requests/sec, events/sec. Practical example: plan for the 95th percentile of peak traffic.
- Latency by percentile: p50, p90, p95, p99, p999. Prioritize reducing p99 in critical services.
- Availability and SLOs/SLAs: e.g., 99.95% (~21.6 minutes downtime per month) or 99.99% for critical services.
- Error rate: percentage of failed requests per unit time; linked to error budget.
- Capacity and utilization: CPU, memory, I/O, and real concurrency limits.
- RTO/RPO for recovery after incidents.

Designing with quantifiable objectives — SLOs, error budgets, latency limits — turns intuitions into measurable technical decisions and clear operational priorities.

*Diagram: Evolution and growth of large-scale systems*

```mermaid
flowchart LR
  A[Initial monolith] --> B[Increased load and data]
  B --> C{Scaling pains}
  C --> D[Contention and bottleneck]
  C --> E[High p99 latencies]
  C --> F[Slow and risky deployments]
  D --> G[Decomposition into services]
  E --> H[Caching and CQRS]
  F --> I[CI/CD automation and canary]
  G --> J[Partitioning and replication]
  H --> K[Stream processing]
  I --> L[Observability and SRE]
  J --> M[Geo-replication and chosen consistency]
  K --> N[Backpressure and resilience]
  L --> O[Failure testing (Chaos)]
  M --> P[Operation and governance]
  N --> P
  O --> P
  P --> Q[Continuous iteration and growth]
  style A fill:#f9f,stroke:#333,stroke-width:1px
  style Q fill:#dfd,stroke:#333,stroke-width:1px
```

## Architectural patterns for high scalability and availability

### Key patterns for horizontal scalability

The most effective patterns for horizontal scaling combine separation of responsibilities, stateless design, and data partitioning. Among them stand out:

- Microservices: divide the domain into small, independently deployable and scalable services. Allows adjusting replicas by load (CPU/req/s) and applying auto-scaling (HPA in Kubernetes).
- Event-driven architecture: decouples emitters and consumers via queues or topics (Kafka, Pulsar). Facilitates parallelism and backpressure.
- CQRS (Command Query Responsibility Segregation) + Event Sourcing: separation of write path (commands) and read path (queries). Append-only writes (event store) scale via partitions; reads use replicated and optimized projections.
- Sharding (data partitioning): split dataset into independent partitions (by customer_id, range, or hash) to distribute I/O and storage.
- Replication: replicas in different zones/regions for availability and read latency.

Each pattern addresses a different bottleneck: microservices for CPU/latency per endpoint; events and CQRS for asynchronous business throughput; sharding for storage capacity and operations per second.

### High availability and fault tolerance

Implementing HA involves combining capabilities at infrastructure and design levels: cross-AZ/region replicas, load balancers, health checks, retries, and failover automation.

- Replicas and quorum: distributed systems use replication factor R (e.g., R=3). Typical scheme: N=3 replicas, writes W=2, reads Rq=2 to guarantee availability and consistency/eventual convergence (W+Rq > N). Adjust W/R according to SLOs.
- Leader election and consensus: for operations requiring a leader (shard coordination, commits), use etcd/Consul/Zookeeper or Raft-based DBs (etcd, Consul, CockroachDB) to avoid split-brain.
- Stateless services + sticky state in storage: keep services stateless to scale and put state in replicated/partitioned stores.
- Cross-region: asynchronous replication between regions for reduced RTO; combine local reads with writes in primary region or use conflict resolution for multi-master.

Operational strategies: readiness/liveness probes (K8s), circuit breakers (Hystrix/Resilience4j), bulkheads, and timeouts determine resilience visible to clients.

### Data partitioning and replication: role in scalability

Partitioning (sharding) and replication are complementary:

- Sharding increases horizontal capacity by distributing storage and CPU among independent nodes. Practical example: start with 16 logical shards and scale to 256/1024 virtual nodes using consistent hashing to minimize redistribution.
- Replication ensures availability and read speed. One replica per AZ and read replica in read-intensive scenarios reduce latency.

Concrete decisions: if N=3 replicas, a common policy is W=2 (write to 2 nodes) and R=1/2 (read from 1 replica for low latency, or 2 for higher consistency). For loads with hotspots, use salted hashing or dynamic re-sharding (vnode) and split/merge ranges.

### Cost in complexity and maintainability

These patterns introduce operational and cognitive complexity:

- Larger failure surface: more components (message broker, event store, orchestrator) to monitor.
- Eventual consistency: CQRS/event sourcing simplify scaling but add debugging challenges (replays, projection rebuilding) and maintaining transactional invariants.
- Schema evolution: event versioning and projection migrations require tools and discipline (schema compatibility, upcasters).
- Essential observability: distributed traces (OpenTelemetry), metrics, and structured logs. More expensive testing: end-to-end tests, chaos tests, and replay validation.

### Practical recommendations

- Prefer microservices when the domain justifies operational independence; avoid microservices by default.
- Use CQRS/event sourcing when auditability, replays, and high business complexity are needed; for simple CRUD cases, it adds unnecessary complexity.
- Design sharding with metrics: establish correct shard criteria (cardinality, hotspots) and support rebalancing without downtime.
- Automate failover and regularly verify quorum policies and latencies among replicas.

The right combination depends: stateless services + multi-AZ replication for low-latency HTTP endpoints; sharding + replicas + event-driven for massive throughput and intensive writes.

*Diagram: Microservices with replication and sharding*

```mermaid
flowchart LR
  Client-->API[API Gateway]
  API-->LB[Load Balancer]
  subgraph Services
    direction TB
    S1[Service A (stateless) \n replicas x N]
    S2[Service B (stateless) \n replicas x M]
  end
  LB-->S1
  LB-->S2
  S1-->ShardRouter[Shard Router]
  S2-->ShardRouter
  subgraph DataLayer
    direction LR
    Shard1[Shard 1 \n Primary (AZ1) \n Replica (AZ2)]
    Shard2[Shard 2 \n Primary (AZ2) \n Replica (AZ1)]
    ShardN[Shard N \n Primary (AZ3) \n Replica (AZ1)]
  end
  ShardRouter-->Shard1
  ShardRouter-->Shard2
  ShardRouter-->ShardN
  note right of Shard1: Replication factor = 3 (example)
  note right of Shard2: Reads from local replicas for low latency
```

*Diagram: Event flow in CQRS and Event Sourcing*

```mermaid
flowchart LR
  Command -->|1| WriteModel[Command Handler / Aggregate]
  WriteModel -->|append| EventStore[(Event Store)]
  EventStore -->|publish| EventBus[(Event Bus / Kafka)]
  EventBus --> ProjectionService[Projector / Materializer]
  ProjectionService --> ReadModel[(Read DB / Index)]
  Query -->|reads| ReadModel
  EventBus --> Saga[Long-running workflow (Saga)]
  Saga -->|dispatch| Command
  subgraph Notes
    direction TB
    E1[Event sourcing: source of truth = events]
    E2[CQRS: separate write/read models]
  end
  EventStore -.-> E1
  ReadModel -.-> E2
```

_Next part: 2/4_
