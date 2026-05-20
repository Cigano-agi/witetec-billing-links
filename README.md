# WiteTec Billing Links

> Generate a shareable payment URL in seconds — no integration required on the seller's end.

![Demo GIF em breve](https://placehold.co/800x400?text=Demo+GIF+em+breve)

---

## What it is

**WiteTec Billing Links** is a multi-service billing platform that lets sellers create shareable payment URLs without writing a line of code. Payers open the link, fill in their details, and the system processes the transaction through a hardened engine with idempotency, rate limiting, and LGPD-compliant PII protection built in.

---

## Why it exists

Most payment integrations require the seller to implement a backend, handle webhooks, and manage state. WiteTec Billing Links inverts that: sellers get a link, payers get a form, and the platform handles everything else. The architecture separates the public-facing API (Node.js) from the transaction engine (.NET 8) so each layer can be tested, scaled, and replaced independently.

---

## Architecture

```
Browser / React 18
      │
      ▼
Node-API (NestJS) — auth, idempotency, rate limiting
      │
      ├──► PostgreSQL 15 — billing links
      └──► dotnet-service (.NET 8) — transaction state machine
                │
                └──► Redis 7 — idempotency keys, rate limit counters
```

---

## Quick start

**Prerequisites:** Docker, Node.js 20+, .NET 8 SDK

```bash
# 1. Start infrastructure
docker compose up -d

# 2. Transaction engine
cd dotnet-service && dotnet run

# 3. API (new terminal)
cd node-api && cp ../.env.example .env && npm install && npm run start:dev

# 4. Frontend (new terminal)
cd frontend && npm install && npm run dev
```

| Service | URL |
|---------|-----|
| Frontend | http://localhost:5173 |
| Node-API | http://localhost:3000 |
| dotnet-service | http://localhost:5001 (internal) |

---

## API reference

### Authenticated (requires `Authorization: Bearer <token>`)

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/v1/billing-links` | Create a link |
| `GET` | `/v1/billing-links` | List seller's links |
| `PATCH` | `/v1/billing-links/:id` | Update description / amount |
| `DELETE` | `/v1/billing-links/:id` | Deactivate a link |
| `GET` | `/v1/billing-links/metrics` | Aggregated metrics |

### Public (no auth)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/v1/billing-links/public-info/:linkId` | Fetch link metadata |
| `POST` | `/v1/public/charge/:linkId` | Submit a payment |

---

## Security highlights

- **Seller isolation** — `seller_id` extracted from JWT only; never from the request body
- **PII protection** — payer name and CPF stripped from all error paths before logging (LGPD)
- **Idempotency** — Redis `SETNX` ensures duplicate submissions never create duplicate charges
- **Rate limiting** — 30 req/min per IP+link; excess returns `429` with `retry_after`
- **Correlation IDs** — single traceable ID spans Node-API and dotnet-service log streams

---

## Running tests

```bash
# Node-API — Jest (26 tests)
cd node-api && npm test

# dotnet-service — xUnit (7 tests)
cd dotnet-service-tests && dotnet test

# Frontend — Vitest (9 tests)
cd frontend && npm test
```

---

## Environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | `postgresql://postgres:postgres@localhost:5432/witetec` | PostgreSQL |
| `REDIS_URL` | `redis://localhost:6379` | Redis |
| `DOTNET_SERVICE_URL` | `http://localhost:5001` | Transaction engine URL |
| `JWT_SECRET` | `dev-secret-local` | Seller token secret |
| `RATE_LIMIT_PER_MINUTE` | `30` | Max charge requests per IP+link/min |
| `IDEMPOTENCY_TTL_SECONDS` | `86400` | Idempotency key TTL (24h) |

---

## License

Proprietary — WiteTec. All rights reserved.

---

*Built by [@cigano.agi](https://github.com/Cigano-agi) — join 2.800+ AI builders at [Athena.agi](https://athena.agi)*