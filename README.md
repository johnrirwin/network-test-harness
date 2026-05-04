# Network Test Harness

A proof-of-concept network test harness built around a **control-plane / execution-plane** architecture.

## POC architecture

The system is split into two major concerns:

- **Control plane**: the user-facing portal and persistence layer used to register users, define tests, schedule runs, and review results.
- **Execution plane**: the worker that consumes queued test requests, simulates the network test, persists lifecycle events/results, and optionally delivers webhooks.

## Core components

| Component | Role |
| --- | --- |
| Next.js portal | Google login, registration, test definition management, run scheduling, and result viewing |
| PostgreSQL | Source of truth for users, test definitions, test runs, events, and webhook deliveries |
| ElasticMQ | Local SQS-compatible queue for development |
| Amazon SQS | Production queue for test run dispatch |
| Rust harness | Execution worker that consumes queue messages and runs the network simulation |
| Webhook delivery | Optional callback to a user-provided endpoint after test completion |

## Deployment model

### Local development
- Next.js portal runs locally for the control plane UI and API routes.
- PostgreSQL stores users, test definitions, runs, events, and webhook delivery records.
- ElasticMQ provides an SQS-compatible queue endpoint for local development.
- Rust harness consumes queued test requests and simulates execution against persisted topology/config data.

### AWS target
- Portal runs on ECS or an equivalent container platform.
- PostgreSQL is hosted on Amazon RDS.
- Amazon SQS replaces ElasticMQ.
- Rust harness runs as an ECS worker service.
- Secrets live in AWS Secrets Manager and access is scoped with IAM roles.

## Repository structure

```text
.
├── AGENTS.md
├── README.md
├── apps/
│   ├── harness/
│   └── portal/
├── docs/
│   └── architecture.md
├── infra/
└── tasks/
```

### Directory responsibilities
- `apps/portal`: Next.js portal application
- `apps/harness`: Rust execution worker
- `docs`: architecture, design, and implementation notes
- `infra`: deployment and infrastructure planning artifacts
- `tasks`: local workflow notes and tracked lessons

## Architecture details

See [`docs/architecture.md`](docs/architecture.md) for:
- control-plane / execution-plane responsibilities
- local and AWS deployment architecture
- system data flow
- repository layout rationale
