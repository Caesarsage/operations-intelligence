# Ops Intelligence Platform

A multi-tenant, BYOK (Bring Your Own Key) intelligence platform that ingests events from communication channels, analyzes them with AI, and publishes actionable insights.

## What It Does

1. **Connect** communication systems (Gmail, Outlook/IMAP, Slack, Zendesk, Jira)
2. **Filter** messages by subject, sender, date, labels, or custom queries
3. **Analyze** using your own LLM API keys (OpenAI, Anthropic, Gemini, or local Ollama)
4. **Publish** structured analysis to Google Sheets
5. **Monitor** trends, recurring issues, and severity patterns from a live dashboard

**Core pipeline**: Ingest -> Normalize -> Analyze -> Act -> Learn

## BYOK Model

Users supply their own API keys — the platform never locks you into a single AI vendor:

| Provider | Models | Auth |
|----------|--------|------|
| OpenAI | GPT-4o, GPT-4o-mini | API key |
| Anthropic | Claude Sonnet, Claude Haiku | API key |
| Google Gemini | Gemini 2.0 Flash, Gemini 2.5 Pro | API key |
| Ollama / vLLM | Llama 3.2, Mistral, any local model | None (local) |

All keys encrypted at rest with AES-256-GCM envelope encryption.

## Tech Stack

- **Backend**: Go 1.24, Postgres 16 (pgvector), Redis 7, RabbitMQ 3, S3/MinIO
- **Frontend**: Next.js 15, TypeScript, Tailwind CSS, shadcn/ui, Recharts
- **Auth**: JWT (access + refresh tokens)
- **Async**: RabbitMQ queue-driven workers for analysis, export, and workflow jobs
- **Encryption**: AES-256-GCM envelope encryption for all BYOK credentials

## Quickstart

```bash
# 1. Start infrastructure
cd backend
docker-compose up -d   # postgres, redis, rabbitmq, minio

# 2. Run database migrations
make migrate-up

# 3. Start backend API (port 8080)
make run-api

# 4. Start async worker
make run-worker

# 5. Start frontend (port 3000)
cd ../frontend
npm install
npm run dev
```

## Spec Documents

| Document | Description |
|----------|-------------|
| [architecture.md](architecture.md) | Multi-tenant system architecture, data model, RBAC, tenant isolation, security architecture, multi-LLM design, deployment topology |
| [roadmap.md](roadmap.md) | V1/V2/V3 product roadmap with build execution order |
| [backend-mvp-go.md](backend-mvp-go.md) | Backend MVP spec: module responsibilities, component specs (Postgres, Redis, RabbitMQ, S3, pgvector), API contract, auth/secrets, multi-LLM provider spec, webhook mode |
| [frontend-spec.md](frontend-spec.md) | Frontend spec: page routing, auth flow, dashboard analytics, component conventions, API client architecture |

## Project Structure

```
ops-intelligence-platform-specs/
├── README.md                    # This file
├── architecture.md              # System architecture spec
├── roadmap.md                   # Product roadmap
├── backend-mvp-go.md           # Backend technical spec
├── frontend-spec.md            # Frontend technical spec
├── backend/                     # Go backend implementation
│   ├── cmd/api/                # HTTP API server
│   ├── cmd/worker/             # Async job worker
│   ├── internal/platform/      # Shared infra (db, cache, queue, ai, crypto)
│   ├── internal/connectors/    # Connector framework (Gmail, etc.)
│   ├── internal/ingestion/     # Event ingestion pipeline
│   ├── internal/analysis/      # AI analysis module
│   ├── internal/issue/         # Issue clustering + trends
│   ├── internal/exports/       # Google Sheets export
│   ├── internal/oauth/         # OAuth flows
│   ├── migrations/             # SQL migrations
│   └── deployments/            # Docker configs
└── frontend/                    # Next.js frontend
    ├── app/                    # App Router pages
    ├── components/             # UI components
    └── lib/                    # API client, auth utilities
```

## Environment Variables

### Backend

| Variable | Required | Description |
|----------|----------|-------------|
| `API_ADDR` | No | Server listen address (default `:8080`) |
| `DB_URL` | No | Postgres connection string (in-memory fallback if absent) |
| `AMQP_URL` | No | RabbitMQ connection string (in-memory fallback) |
| `REDIS_ADDR` | No | Redis address (no caching fallback) |
| `S3_BUCKET` | No | S3 bucket name |
| `S3_ENDPOINT` | No | S3/MinIO endpoint URL |
| `MASTER_ENCRYPTION_KEY` | Yes | Base64-encoded 32-byte key for envelope encryption |
| `JWT_SECRET` | Yes | Secret for JWT token signing |
| `OPENAI_API_KEY` | No | Platform-default OpenAI key (users override with BYOK) |
| `ANTHROPIC_API_KEY` | No | Platform-default Anthropic key |
| `GEMINI_API_KEY` | No | Platform-default Gemini key |
| `OLLAMA_BASE_URL` | No | Ollama server URL (default `http://localhost:11434`) |
| `GOOGLE_CLIENT_ID` | Yes | Google OAuth client ID |
| `GOOGLE_CLIENT_SECRET` | Yes | Google OAuth client secret |

### Frontend

| Variable | Required | Description |
|----------|----------|-------------|
| `NEXT_PUBLIC_API_BASE_URL` | Yes | Backend API URL (default `http://localhost:8080`) |
