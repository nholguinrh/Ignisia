Project structure:
ignisia/
├── shared/              ← Event catalog + types shared across services
├── docs/                ← Architecture + README docs
├── infra/               ← Infrastructure-only compose (DBs, broker, cache)
├── podman-compose.yml   ← Full stack compose
├── CONTRIBUTING.md      ← Getting started guide for devs
└── services/
    ├── api-gateway/     ← Fully implemented (Fastify, JWT proxy, rate limiting)
    ├── identity-service/← Fully implemented (see below)
    └── [8 other services] ← Scaffolded with .env.example + README
