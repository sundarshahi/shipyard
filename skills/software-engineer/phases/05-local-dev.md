# Phase 5: Local Dev Environment

## Objective

Generate everything needed to run the full stack locally. This includes Docker Compose for infrastructure, seed data scripts, a one-command dev setup script, and a root Makefile with all common commands.

## 5.1 — Docker Compose

Generate `docker-compose.dev.yml` at the project root:

```yaml
# Includes:
# - All application services (with hot-reload/watch mode)
# - PostgreSQL / MySQL (matching production DB)
# - Redis (matching production cache)
# - Message broker (Kafka/RabbitMQ matching production)
# - Mailhog / Mailpit (email testing)
# - LocalStack (if AWS services used) / GCP emulators
# - OpenTelemetry Collector + Jaeger (local tracing)
```

Requirements:
- Health checks on all services (depends_on with condition: service_healthy)
- Named volumes for data persistence across restarts
- Environment variable files (`.env.development`) — NOT committed, `.env.example` committed
- Port mapping that avoids conflicts (document in README)
- Hot-reload enabled for all application services

## 5.2 — Seed Data

Generate `scripts/seed-data.sh` at the project root:

```bash
#!/bin/bash
# Seeds the local database with realistic test data
# Usage: make seed   (or ./scripts/seed-data.sh)
#
# Creates:
# - 3 tenant organizations (free, pro, enterprise tiers)
# - 10 users per tenant (with various roles)
# - Realistic sample data for each domain entity
# - Admin super-user for testing
```

Requirements:
- Idempotent (safe to run multiple times — upserts, not inserts)
- Uses the same migration runner (runs migrations first if needed)
- Creates data that exercises all tenant tiers and role types
- Includes edge cases (long names, unicode, empty optional fields)
- Outputs created credentials and IDs for developer reference

## 5.3 — Dev Setup Script

Generate `scripts/dev-setup.sh` at the project root:

```bash
#!/bin/bash
# One-command local development setup
# Usage: ./scripts/dev-setup.sh
#
# Steps:
# 1. Check prerequisites (Docker, language runtime, tools)
# 2. Copy .env.example to .env.development (if not exists)
# 3. Start infrastructure (docker-compose up -d postgres redis kafka)
# 4. Wait for services to be healthy
# 5. Run database migrations
# 6. Seed development data
# 7. Install dependencies for all services
# 8. Print "Ready to develop" with service URLs
```

## 5.4 — Makefile

Generate `Makefile` at the project root:

```makefile
# Available commands:
# make setup          — First-time dev environment setup
# make up             — Start all services (docker-compose)
# make down           — Stop all services
# make logs           — Tail logs for all services
# make logs-<service> — Tail logs for one service
# make test           — Run all tests
# make test-unit      — Run unit tests only
# make test-int       — Run integration tests only
# make lint           — Lint all services
# make migrate-up     — Run pending migrations
# make migrate-down   — Rollback last migration
# make seed           — Seed development data
# make clean          — Remove containers, volumes, caches
# make build          — Build all service images
# make arch           — Import-boundary fitness function (exits non-zero on violation)
# make smoke-telemetry— Boot stack, scrape /metrics, assert RED instruments (exits non-zero on "No data")
# make security-scan  — osv-scanner + gitleaks + semgrep (exits non-zero on Critical/High)
# make flags-check    — Validate config/feature-flags.yaml (schema, stale/expired flags, safe defaults) — exits non-zero on violation
```

**This is the BASE Makefile (software-engineer owns it).** software-engineer EMITS the targets it owns here — including `make flags-check`, which validates `config/feature-flags.yaml` (the registry software-engineer owns per Phase 3.8): well-formed schema, every flag has a SAFE DEFAULT, and no flag is past its `removal_by` (stale-flag expiry) — exiting non-zero on any violation, wired as a required CI step.

**Other owner skills APPEND their gate targets to this same root Makefile** (no CI gate may call a target no skill emits):
- **qa** → `coverage-check`, `patch-coverage`
- **frontend-engineer** → `size-limit`, `build-frontend`
- **devops** → `docs-examples`

Each owner appends its own target; software-engineer does not pre-stub them. The base Makefile must be append-friendly (targets are additive, no clobbering).

Per-service Makefiles at `services/<name>/Makefile`:
```makefile
# make run     — Run this service locally (hot-reload)
# make test    — Run this service's tests
# make lint    — Lint this service
# make build   — Build this service
# make migrate — Run this service's migrations
```

## 5.5 — Environment Template

Generate `.env.example` at the project root with placeholder values for all required and optional environment variables documented in Phase 2 (section 2.7). Never commit `.env` or `.env.development`. Add both to `.gitignore`.

## Output Structure

### Project Root Output (Deliverables)

```
services/
│   └── <service-name>/
│       ├── src/
│       │   ├── handlers/           # API route handlers
│       │   │   ├── health.ts
│       │   │   └── <resource>.ts
│       │   ├── services/           # Business logic
│       │   │   └── <resource>.service.ts
│       │   ├── repositories/       # Data access
│       │   │   └── <resource>.repository.ts
│       │   ├── models/             # Domain models
│       │   │   ├── entities/
│       │   │   ├── dto/
│       │   │   └── mappers/
│       │   ├── middleware/          # Auth, logging, rate limiting
│       │   │   ├── auth.middleware.ts
│       │   │   ├── logging.middleware.ts
│       │   │   ├── rate-limit.middleware.ts
│       │   │   ├── tenant.middleware.ts
│       │   │   └── error-handler.middleware.ts
│       │   ├── events/             # Event producers/consumers
│       │   │   ├── producers/
│       │   │   └── consumers/
│       │   ├── config/             # Service configuration
│       │   │   ├── index.ts
│       │   │   ├── database.ts
│       │   │   └── dependencies.ts
│       │   └── index.ts            # Entry point
│       ├── tests/
│       │   ├── unit/
│       │   │   ├── services/
│       │   │   └── mappers/
│       │   ├── integration/
│       │   │   └── repositories/
│       │   └── fixtures/
│       ├── Dockerfile
│       └── Makefile
libs/
│   └── shared/
│       ├── types/                  # Shared TypeScript types / proto-generated types
│       ├── errors/                 # Domain error definitions
│       ├── middleware/             # Reusable middleware (auth, tenant, logging)
│       ├── clients/               # Service-to-service + external API clients
│       │   ├── <service>-client.ts
│       │   └── <external>/
│       ├── events/                 # Event envelope, serialization, base consumer
│       ├── cache/                  # Cache-aside implementation
│       ├── resilience/             # Retry, circuit breaker, timeout wrappers
│       ├── feature-flags/          # Feature flag abstraction + backends
│       ├── observability/          # Tracing, metrics, logging setup
│       └── testing/                # Test helpers, factories, mocks
scripts/
│   ├── seed-data.sh               # Idempotent seed data loader
│   ├── dev-setup.sh               # One-command dev environment setup
│   └── migrate.sh                 # Migration runner wrapper
docker-compose.dev.yml             # Full local dev stack
.env.example                       # Template for local env vars
Makefile                           # Root-level dev commands
```

### Workspace Output (`drydock/software-engineer/`)

```
drydock/software-engineer/
├── implementation-plan.md
├── progress.md
└── logs/
    ├── build.log
    └── debug.log
```

## Validation Loop

Before marking the suite as complete:
- `make setup` runs successfully from a clean checkout
- `docker-compose up` starts all services with health checks passing
- `make seed` populates realistic test data
- `make test` runs all unit and integration tests green
- `make arch` exits 0 across all services (architecture boundaries hold)
- `make smoke-telemetry` exits 0 — **Telemetry end-to-end: boots the stack, hits an endpoint → a trace appears in Jaeger AND `/metrics` is populated with non-zero `http_requests_total` + the RED instruments** (no "No data" panels on first run; the check fails by exit code, see 3.10)
- `make security-scan` exits 0 across the assembled stack (osv-scanner + gitleaks + semgrep; non-zero on Critical/High)
- All services accessible at documented ports
- Developer can start coding within 5 minutes of running setup

## Quality Bar

- One-command setup: `make setup` does everything
- Idempotent: running setup twice does not break anything
- Documented: all ports, URLs, and credentials listed
- Clean: `.env.example` committed (placeholders only — secrets injected at runtime by the secret manager), `.env` gitignored
- Fast: infrastructure starts in under 60 seconds
- `make arch` is a required, non-skippable CI step (no `|| true` / `continue-on-error`)
- `make smoke-telemetry` is a required, non-skippable CI step (boots the stack, hits an endpoint, scrapes `/metrics`, asserts non-zero `http_requests_total` + RED instruments present; "No data" fails by exit code — see 3.10)
- **security-defaults checklist passes** across the assembled stack
- **BUILD-exit security scan**: `make security-scan` exits 0 — a concrete target (mirroring `make arch`) running osv-scanner (SCA) + gitleaks (secret scan) + semgrep (SAST) over the freshly written code; exits non-zero on any Critical/High finding and blocks suite completion. Required, non-skippable CI step (no `|| true` / `continue-on-error`)
