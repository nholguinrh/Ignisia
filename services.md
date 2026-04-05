Identity Service — fully implemented with:

POST /api/v1/auth/register — auto-creates account (controlled by ALLOW_REGISTRATION env var)

POST /api/v1/auth/login — timing-attack-safe password check

POST /api/v1/auth/refresh — rotating refresh tokens (one-time use, stored as SHA-256 hashes)

POST /api/v1/auth/logout — revokes all refresh tokens

GET  /api/v1/auth/me — returns current user from JWT

GET  /api/v1/users — lists team members

RS256 key pair generation utility (npm run generate:keys)

PostgreSQL migration with users, teams, refresh_tokens tables

RabbitMQ event publisher wired up and ready
