# Architecture Overview

## Goals

This proof of concept demonstrates a minimal but realistic network test harness that can:
- authenticate users through Google
- let users define and run simulated network tests
- execute tests asynchronously through a queue-backed worker
- persist lifecycle events and final results
- notify users through an optional webhook callback

## Architectural model

The POC follows a **control-plane / execution-plane** split.

### Control plane
The control plane owns user interaction, orchestration, and persistence.

Responsibilities:
- authenticate users with Google
- persist user profiles and optional webhook URLs
- store test definitions, topology JSON, and test configuration JSON
- create test runs and enqueue execution requests
- show run status, lifecycle events, and final summaries

Primary components:
- Next.js portal
- PostgreSQL

### Execution plane
The execution plane owns asynchronous test execution.

Responsibilities:
- consume queued test run requests
- claim queued runs safely and idempotently
- load topology and configuration data from PostgreSQL
- simulate the network test
- persist lifecycle events and final results
- attempt webhook delivery after completion

Primary components:
- ElasticMQ in local development / Amazon SQS in AWS
- Rust harness worker

## Component responsibilities

### Next.js portal
- hosts the user-facing UI
- handles Google authentication
- creates and manages user-owned test definitions
- creates test runs and enqueues queue messages containing IDs only
- renders test run status, event history, and result summaries

### PostgreSQL
- stores durable application state
- acts as the source of truth for users, test definitions, test runs, test run events, and webhook delivery records
- stores topology/configuration payloads and execution outcomes

### Queue layer
- decouples user-triggered run creation from background execution
- enables local development with ElasticMQ
- maps cleanly to Amazon SQS in production

### Rust harness
- runs as a background worker
- long-polls the queue for work
- loads run context from PostgreSQL
- executes the simulated topology test
- records events, results, and webhook outcomes

### Webhook delivery
- sends an optional POST callback after test completion
- should never be the source of truth for test status
- must be recorded independently for auditability and troubleshooting

## End-to-end flow

1. A user signs in through the portal.
2. The portal persists or refreshes the user profile in PostgreSQL.
3. The user creates a test definition using a stored topology and test configuration.
4. The portal creates a `test_runs` row with status `QUEUED`.
5. The portal sends a queue message containing the run and ownership identifiers.
6. The Rust harness consumes the message and claims the queued run.
7. The harness loads topology/configuration data from PostgreSQL and executes the simulation.
8. The harness writes lifecycle events and the final result summary.
9. If configured, the harness attempts webhook delivery and records the outcome.
10. The portal displays status, history, and results from PostgreSQL.

## Local deployment architecture

Local development is designed for fast iteration with production-like seams:

- **Portal**: local Next.js application container or process
- **Database**: local PostgreSQL container
- **Queue**: ElasticMQ container exposing an SQS-compatible endpoint
- **Harness**: local Rust worker container or process

Why this works well locally:
- queue integration is exercised early
- the worker remains decoupled from the portal
- the persistence model matches the eventual production design
- only the queue/provider swap changes materially between local and AWS

## AWS deployment architecture

The intended production path preserves the same logical boundaries:

- **Portal**: ECS service or equivalent container-hosted web app
- **Database**: Amazon RDS for PostgreSQL
- **Queue**: Amazon SQS
- **Harness**: ECS worker service polling the queue
- **Secrets**: AWS Secrets Manager
- **Access control**: IAM roles for portal and harness workloads

This keeps the POC migration path straightforward:
- ElasticMQ settings become SQS settings
- local containers map to deployed services
- PostgreSQL-backed workflows remain unchanged

## Repository layout rationale

```text
.
├── apps/
│   ├── harness/
│   └── portal/
├── docs/
├── infra/
└── tasks/
```

### `apps/portal`
Planned home for the Next.js portal app, including UI routes, API routes, authentication, and portal-specific utilities.

### `apps/harness`
Planned home for the Rust worker, queue consumer logic, execution engine, and persistence integration.

### `docs`
Home for architecture notes, ADRs, rollout notes, and other design documentation.

### `infra`
Reserved for infrastructure planning and eventual IaC artifacts or environment-specific deployment notes.

### `tasks`
Local workflow scratch space plus tracked lessons learned that shape future agent behavior.
