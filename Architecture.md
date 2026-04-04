# 🏗️ Ignisia — Architecture Document

> Cloud-Native Microservices Architecture | Podman-First | Event-Driven

---

## Table of Contents

1. [Architectural Principles](#architectural-principles)
2. [Microservices Inventory](#microservices-inventory)
3. [Communication Strategy](#communication-strategy)
4. [Event Catalog](#event-catalog)
5. [Data Isolation Strategy](#data-isolation-strategy)
6. [Real-Time Architecture (Rating Sessions)](#real-time-architecture-rating-sessions)
7. [Pod & Container Design](#pod--container-design)
8. [API Gateway Design](#api-gateway-design)
9. [Security Model](#security-model)
10. [Observability Stack](#observability-stack)
11. [Development Environment](#development-environment)
12. [Deployment Topology](#deployment-topology)

---

## Architectural Principles

Ignisia is designed around the following non-negotiable principles:

| Principle | Description |
|---|---|
| **One pod, one service** | Each microservice runs in its own Podman pod. No shared pods except infrastructure. |
| **One model, one service** | Each service owns its own data model and database schema. No cross-service schema sharing. |
| **API-first** | Every service exposes a versioned REST API. No direct service-to-service DB access. |
| **Event-driven async** | Business workflows that cross service boundaries use async events via RabbitMQ. |
| **Real-time where it matters** | Rating Sessions use WebSockets for live collaborative scoring. |
| **Polyglot by design** | Language choice is made per service based on the best fit. |
| **Observable by default** | Every service emits structured logs, metrics, and traces from day one. |
| **Fail gracefully** | Every service handles the unavailability of its dependencies without crashing. |

---

## Microservices Inventory

### Core Services

| # | Service | Language | Responsibility | Port |
|---|---|---|---|---|
| 1 | **API Gateway** | Node.js / TypeScript | Single entry point, routing, auth validation, rate limiting | 8080 |
| 2 | **Identity Service** | Node.js / TypeScript | User auth, JWT issuance, roles & permissions, teams | 3001 |
| 3 | **Idea Service** | Python / FastAPI | CRUD for ideas, maturity stage transitions, idea lifecycle | 3002 |
| 4 | **Feature Service** | Python / FastAPI | Feature CRUD, polarity, weight management, feature assignment to ideas | 3003 |
| 5 | **Rating Session Service** | Node.js / TypeScript | Session lifecycle, participant management, score submission & aggregation | 3004 |
| 6 | **Scoring Engine** | Python / FastAPI | Priority score computation, weight application, score history | 3005 |
| 7 | **Realtime Service** | Node.js / TypeScript | WebSocket server for live Rating Sessions, presence, score reveal | 3006 |
| 8 | **Notification Service** | Python / FastAPI | Email & in-app notifications for session invites, score results, stage changes | 3007 |
| 9 | **Backlog Service** | Go | Ordered backlog computation, sprint assignment, backlog snapshots | 3008 |
| 10 | **Audit Service** | Go | Immutable event log, change history for ideas, features, and scores | 3009 |

### Infrastructure Services (also run as pods)

| # | Service | Technology | Responsibility | Port |
|---|---|---|---|---|
| 11 | **Message Broker** | RabbitMQ | Async event bus between services | 5672 / 15672 |
| 12 | **Idea DB** | PostgreSQL | Database for Idea Service | 5432 |
| 13 | **Feature DB** | PostgreSQL | Database for Feature Service | 5433 |
| 14 | **Session DB** | PostgreSQL | Database for Rating Session Service | 5434 |
| 15 | **Identity DB** | PostgreSQL | Database for Identity Service | 5435 |
| 16 | **Backlog DB** | PostgreSQL | Database for Backlog Service | 5436 |
| 17 | **Audit DB** | PostgreSQL | Append-only store for Audit Service | 5437 |
| 18 | **Scoring DB** | PostgreSQL | Time-series score history for Scoring Engine | 5438 |
| 19 | **Cache** | Redis | Session tokens, rate limiting, real-time presence | 6379 |
| 20 | **Frontend** | React / Nginx | SPA served via Nginx | 80 / 443 |

---

## Communication Strategy

Ignisia uses two communication channels:

### 1. Synchronous REST (Request/Response)

Used when the caller needs an immediate response:
- User-facing operations (CRUD on ideas, features, sessions)
- Auth validation (gateway → Identity Service)
- Backlog reads (frontend → Backlog Service via Gateway)
- Score submissions during a session

All REST APIs follow OpenAPI 3.1 specification. Every service maintains its own `openapi.yaml`.

```
Client → API Gateway → Target Service → Response
```

### 2. Asynchronous Events (via RabbitMQ)

Used when an action triggers downstream effects in other services:
- Score submitted → Scoring Engine recomputes → Backlog Service reorders
- Session closed → Notification Service emails participants
- Idea stage changed → Audit Service logs → Notification Service alerts

```
Source Service → RabbitMQ Exchange → Queue → Consumer Service
```

Each service declares its own exchanges and queues using RabbitMQ topic exchanges. Services are consumers AND producers. No service calls another service's REST API for async workflows.

---

## Event Catalog

All events follow this envelope structure:

```json
{
  "event_id": "uuid",
  "event_type": "idea.stage.changed",
  "source_service": "idea-service",
  "timestamp": "ISO8601",
  "version": "1.0",
  "payload": { ... }
}
```

### Event Registry

| Event Type | Producer | Consumers | Trigger |
|---|---|---|---|
| `idea.created` | Idea Service | Audit Service | New idea submitted |
| `idea.updated` | Idea Service | Audit Service | Idea description updated |
| `idea.stage.changed` | Idea Service | Notification Service, Audit Service, Backlog Service | Maturity stage promoted or discarded |
| `feature.created` | Feature Service | Audit Service | New scoring feature created |
| `feature.weight.changed` | Feature Service | Scoring Engine, Audit Service | Feature weight updated globally |
| `feature.assigned` | Feature Service | Audit Service | Feature assigned to idea |
| `session.created` | Rating Session Service | Notification Service, Audit Service | Rating Session opened |
| `session.score.submitted` | Rating Session Service | Scoring Engine, Realtime Service, Audit Service | Participant submits score |
| `session.scores.revealed` | Rating Session Service | Realtime Service, Scoring Engine | All participants submitted |
| `session.closed` | Rating Session Service | Scoring Engine, Notification Service, Audit Service | Session finalized |
| `score.recalculated` | Scoring Engine | Backlog Service, Audit Service | Priority score updated for idea |
| `backlog.reordered` | Backlog Service | Realtime Service, Audit Service | Backlog rank changes after scoring |
| `sprint.assigned` | Backlog Service | Notification Service, Audit Service | Idea assigned to a sprint |

---

## Data Isolation Strategy

Each service owns its PostgreSQL instance. No service may query another service's database directly. Cross-service data needs are handled exclusively via REST or events.

### Schema Ownership

```
identity-db     → users, teams, roles, tokens
idea-db         → ideas, idea_tags, idea_comments, idea_links
feature-db      → features, idea_features (assignment only, no scores)
session-db      → rating_sessions, session_participants, session_scores
scoring-db      → score_history, priority_snapshots, weight_history
backlog-db      → backlog_entries, sprint_assignments, backlog_snapshots
audit-db        → audit_events (append-only, no deletes, no updates)
```

### Cross-Service Data Composition

When the frontend needs a composed view (e.g., an idea with its current score, assigned features, and last session result), the API Gateway or a dedicated **BFF (Backend for Frontend)** orchestrates calls to multiple services and composes the response. This is preferable to services calling each other.

---

## Real-Time Architecture (Rating Sessions)

Rating Sessions require live, collaborative scoring where participants see each other's submission status (but not scores) in real time, and then receive all scores simultaneously on reveal.

### Technology: WebSockets via Realtime Service

The Realtime Service maintains persistent WebSocket connections per session room. All participants in a session connect to the same room.

### Flow

```
1. Participant opens Rating Session in browser
2. Frontend connects: WebSocket → Realtime Service (room: session_id)
3. Realtime Service registers presence in Redis (SET session:{id}:presence)

4. Participant submits score:
   REST POST /sessions/{id}/scores → Rating Session Service
   → Rating Session Service publishes: session.score.submitted
   → Realtime Service consumes event, broadcasts to room:
     { type: "PARTICIPANT_SCORED", participant_id, feature_id }
     (score value is NOT broadcast yet — anti-anchoring)

5. Last participant submits:
   → Rating Session Service detects all submitted
   → publishes: session.scores.revealed
   → Realtime Service broadcasts to room:
     { type: "SCORES_REVEALED", scores: [ { participant_id, feature_id, score } ] }
   → Frontend reveals all scores simultaneously

6. Session closed:
   → Rating Session Service publishes: session.closed
   → Scoring Engine consumes, recomputes priority score
   → Backlog Service reorders
   → Realtime Service broadcasts: { type: "BACKLOG_UPDATED", new_ranking: [...] }
```

### WebSocket Message Types

| Direction | Message Type | Description |
|---|---|---|
| Server → Client | `PRESENCE_UPDATE` | Who is connected to the session room |
| Server → Client | `PARTICIPANT_SCORED` | A participant submitted (no score value) |
| Server → Client | `SCORES_REVEALED` | All scores visible |
| Server → Client | `SESSION_CLOSED` | Session finalized |
| Server → Client | `BACKLOG_UPDATED` | New priority ranking |
| Client → Server | `HEARTBEAT` | Keep connection alive |

---

## Pod & Container Design

Each service and each database runs in its own Podman pod. This mirrors the isolation model of Kubernetes pods for easy future migration.

### Pod Naming Convention

```
luminae-{service-name}-pod
```

### Example Pod Structure

```
luminae-idea-pod
├── luminae-idea-service   (Python / FastAPI container)
└── luminae-idea-db        (PostgreSQL container — sidecar pattern)

luminae-gateway-pod
└── luminae-gateway        (Node.js container — no local DB)

luminae-broker-pod
└── luminae-rabbitmq       (RabbitMQ container)

luminae-cache-pod
└── luminae-redis          (Redis container)
```

### Podman Network

All pods share a single Podman network: `luminae-net`. Services communicate by pod name via DNS resolution within this network.

```bash
# Create shared network
podman network create luminae-net

# Create a pod and attach to network
podman pod create --name luminae-idea-pod --network luminae-net

# Run containers inside the pod
podman run -d --pod luminae-idea-pod --name luminae-idea-db \
  -e POSTGRES_DB=luminae_ideas \
  -e POSTGRES_PASSWORD=... \
  postgres:16-alpine

podman run -d --pod luminae-idea-pod --name luminae-idea-service \
  -e DATABASE_URL=postgresql://localhost:5432/luminae_ideas \
  luminae/idea-service:latest
```

### Containerfile Conventions

Every service must include:
- A `Containerfile` (not `Dockerfile`) at the service root
- A non-root user for runtime
- A health check endpoint at `GET /health`
- Environment-based configuration (no config files committed)

---

## API Gateway Design

The API Gateway is the single entry point for all client traffic. It handles:

| Responsibility | Implementation |
|---|---|
| TLS termination | Nginx or Caddy in front of Gateway |
| JWT validation | Validates token signature, expiry, and claims on every request |
| Route mapping | Maps `/api/v1/ideas/*` → Idea Service, etc. |
| Rate limiting | Redis-backed token bucket per user |
| Request logging | Structured JSON logs with trace ID |
| CORS | Configured per environment |
| WebSocket proxying | `/ws/*` routes proxied to Realtime Service |

### Route Map

```
GET/POST   /api/v1/ideas/*           → idea-service:3002
GET/POST   /api/v1/features/*        → feature-service:3003
GET/POST   /api/v1/sessions/*        → rating-session-service:3004
GET        /api/v1/backlog/*         → backlog-service:3008
GET        /api/v1/scores/*          → scoring-engine:3005
POST       /api/v1/auth/*            → identity-service:3001
GET        /api/v1/audit/*           → audit-service:3009
WS         /ws/sessions/{id}         → realtime-service:3006
```

---

## Security Model

| Layer | Control |
|---|---|
| Transport | HTTPS everywhere, TLS 1.3 minimum |
| Authentication | JWT (RS256), issued by Identity Service, validated at Gateway |
| Authorization | Role-based (ADMIN, MODERATOR, CONTRIBUTOR, VIEWER) enforced per service |
| Secrets management | Environment variables via `.env` files (dev) → Podman secrets (staging/prod) |
| Network isolation | All services on private `luminae-net`, only Gateway exposed externally |
| Audit trail | Every write operation emits an `audit_event` — immutable, no deletes |
| Score integrity | Scores are locked after session reveal — no post-reveal edits allowed |

---

## Observability Stack

| Signal | Tool | Notes |
|---|---|---|
| Logs | Structured JSON → stdout | Aggregated via Fluentd or Loki |
| Metrics | Prometheus (scrape endpoint `/metrics` per service) | Custom metrics: score submissions, session latency, backlog reorder time |
| Tracing | OpenTelemetry → Jaeger | Trace IDs propagated via HTTP headers across all services |
| Health | `GET /health` per service | Returns service status + dependency status |
| Dashboards | Grafana | Backlog health, session activity, score distribution |

---

## Development Environment

### Prerequisites

```
podman >= 4.x
podman-compose >= 1.x
Node.js >= 20.x
Python >= 3.12
Go >= 1.22
```

### Running Locally

```bash
# Clone the repo
git clone https://github.com/your-org/luminae.git
cd luminae

# Start all infrastructure pods (broker, cache, databases)
podman-compose -f infra/podman-compose.yml up -d

# Start a specific service in development mode
cd services/idea-service
cp .env.example .env
uvicorn main:app --reload --port 3002

# Or start everything
podman-compose -f podman-compose.yml up -d
```

### Environment Variables (per service)

Each service reads configuration from environment variables. A `.env.example` file is committed for each service:

```
DATABASE_URL=postgresql://user:pass@localhost:5432/db_name
RABBITMQ_URL=amqp://guest:guest@localhost:5672
REDIS_URL=redis://localhost:6379
JWT_PUBLIC_KEY=...
SERVICE_PORT=3002
LOG_LEVEL=debug
ENVIRONMENT=development
```

---

## Deployment Topology

### Stage 1 — Local Development

All pods run on a single developer machine via Podman. `podman-compose` orchestrates startup order and network configuration.

### Stage 2 — Single Server (Self-Hosted)

All pods run on a single Linux server. Podman systemd units (`podman generate systemd`) manage pod lifecycle. Caddy handles TLS and reverse proxying.

### Stage 3 — Kubernetes (Future)

Each Podman pod maps 1:1 to a Kubernetes Pod / Deployment. Migration is straightforward because:
- Pod isolation is already enforced
- All configuration is environment-based
- Health checks are already implemented
- Service discovery uses DNS (compatible with Kubernetes services)

### Port Allocation Summary

```
8080  — API Gateway (public)
80    — Frontend via Nginx (public)
443   — TLS (handled by Caddy/Nginx in front)

3001  — Identity Service
3002  — Idea Service
3003  — Feature Service
3004  — Rating Session Service
3005  — Scoring Engine
3006  — Realtime Service (WebSocket)
3007  — Notification Service
3008  — Backlog Service
3009  — Audit Service

5432–5438 — PostgreSQL instances (internal only)
5672  — RabbitMQ AMQP (internal only)
15672 — RabbitMQ Management UI (dev only)
6379  — Redis (internal only)
```

---

*Ignisia Architecture Document — Living document, updated with each architectural decision.*
