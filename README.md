<p align="center">
  <img src="./apps/web/src/public/blanco-ilenia.png" alt="ILENIA" width="220" />
</p>

<h1 align="center">ILENIA-chat</h1>

<p align="center">
  Frontend and backend monorepo for the ILENIA conversational RAG platform, built to provide authenticated, multi-turn chat experiences with real-time streaming responses.
</p>

<p align="center">
  <a href="#"><img alt="Project Status" src="https://img.shields.io/badge/status-finished-1f9d55?style=for-the-badge"></a>
  <a href="https://next-generation-eu.europa.eu/"><img alt="NextGenerationEU" src="https://img.shields.io/badge/Funded%20by-NextGenerationEU-003399?style=for-the-badge"></a>
  <a href="https://www.surlabs.com/"><img alt="Developer Surlabs" src="https://img.shields.io/badge/Developer-Surlabs-00529B?style=for-the-badge"></a>
</p>

<p align="center">
  <a href="https://www.ehu.eus/"><img alt="UPV/EHU" src="https://img.shields.io/badge/UPV%2FEHU-University%20of%20the%20Basque%20Country-7a7a7a?style=for-the-badge"></a>
  <a href="http://www.hitz.eus/"><img alt="HiTZ Center" src="https://img.shields.io/badge/HiTZ-Basque%20Center%20for%20Language%20Technology-2563eb?style=for-the-badge"></a>
</p>

<p align="center">
  <img src="./apps/web/src/public/NEXTGENERATION_CONBANDERA_REDES_PRTR_BLANCO.png" alt="NextGenerationEU banner" width="720" />
</p>

---

Monorepo that provides a full-stack RAG (Retrieval-Augmented Generation) chat interface. Authenticated users can hold multi-turn conversations that are answered in real time by external RAG backends. The server auto-discovers each backend's available language/domain modes and routes queries accordingly; a mock provider is included for development without a live RAG backend.

> [!NOTE]
> To see detailed instructions for the Nós Project, check the `Nós branch` of the repo

## Project Structure

```
ilenia-rag-frontend/
├── apps/
│   ├── server/                       # API backend (Next.js + oRPC)
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── api/auth/         # Better Auth endpoints
│   │   │   │   └── rpc/              # oRPC handler (all RPC traffic)
│   │   │   ├── db/
│   │   │   │   ├── schema/           # Drizzle schema (auth, chat, message)
│   │   │   │   └── migrations/       # SQL migration files
│   │   │   ├── lib/
│   │   │   │   ├── auth.ts           # Better Auth configuration
│   │   │   │   ├── rag-adapter.ts    # RagProvider interface + factory
│   │   │   │   ├── rag-config.ts     # Multi-server credentials service
│   │   │   │   ├── rag-discovery.ts  # Capability polling & backend routing
│   │   │   │   ├── real-rag-provider.ts  # Production SSE client
│   │   │   │   └── mock-rag-provider.ts  # Development mock
│   │   │   └── routers/
│   │   │       ├── chat.ts           # Chat CRUD + streaming sendMessage
│   │   │       └── rag.ts            # RAG capabilities endpoint
│   │   ├── docker-compose.yml        # MySQL container for local dev
│   │   └── .env.example
│   └── web/                          # Frontend (Next.js + React 19)
│       ├── src/
│       │   ├── app/
│       │   │   ├── chat/             # Chat list and [id] conversation pages
│       │   │   └── page.tsx          # Root page
│       │   ├── components/
│       │   │   ├── ai-elements/      # Streaming chat UI (messages, sources, canvas…)
│       │   │   ├── layout/           # Sidebar, header
│       │   │   └── ui/               # shadcn/ui primitives (Radix-based)
│       │   ├── hooks/                # use-rag-capabilities, use-mobile
│       │   ├── lib/                  # oRPC client, auth client, query client
│       │   └── locales/              # i18n strings (es, eu, gl, va)
│       └── .env.example
├── docker-compose.yml                # Production deployment (db + server + web)
├── package.json                      # Workspace root (npm workspaces)
└── turbo.json                        # Turborepo task pipeline
```

## Tech Stack

| Layer | Technology |
|---|---|
| **Monorepo** | Turborepo, npm workspaces |
| **Frontend** | Next.js 16, React 19, TailwindCSS 4, shadcn/ui, Radix UI |
| **State / data fetching** | TanStack Query 5, Vercel AI SDK 5 |
| **API layer** | oRPC 1.8 (end-to-end type-safe RPC, streaming support) |
| **Backend** | Next.js 16 (API routes only), Pino logger, OpenRouter SDK |
| **Auth** | Better Auth 1.3 (email/password, Drizzle adapter) |
| **ORM / DB** | Drizzle ORM, MySQL 8 (mysql2 driver) |
| **Deployment** | Docker Compose (three services: db, server, web) |
| **Tooling** | TypeScript 5, Biome 2 (lint + format) |

## Local Setup & Deployment

### System requirements

- **Node.js** 20 or higher (`npm` 11+)
- **Docker** (for the MySQL database container)

### Local development

```bash
# 1. Install dependencies (from monorepo root)
cd ilenia-rag-frontend
npm install

# 2. Configure environment variables
cp apps/server/.env.example apps/server/.env   # fill in required values
cp apps/web/.env.example apps/web/.env         # fill in NEXT_PUBLIC_SERVER_URL

# 3. Start MySQL container
cd apps/server && docker compose up -d && cd ../..

# 4. Apply DB schema
cd apps/server && npx drizzle-kit migrate && cd ../..

# 5. Start all apps (server :3000, web :3001)
npm run dev
```

Open [http://localhost:3001](http://localhost:3001) for the web app.
The API is available at [http://localhost:3000](http://localhost:3000).

#### Key environment variables (`apps/server/.env`)

| Variable | Description |
|---|---|
| `DATABASE_URL` | MySQL connection string |
| `BETTER_AUTH_SECRET` | Random secret for session signing |
| `BETTER_AUTH_URL` | Public URL of the server app |
| `CORS_ORIGIN` | Allowed origin for the web app |
| `RAG_PROVIDER` | `mock` (default) or `real` |
| `RAG_SERVERS` | Comma-separated RAG backend URLs (required when `real`) |
| `RAG_MASTER_URL` | Master backend URL for `/configure` calls (required when `real`) |
| `RAG_SERVER_CREDENTIALS` | `user:pass` pairs matching `RAG_SERVERS` order |

`apps/web/.env` requires two variables:

| Variable | Description |
|---|---|
| `NEXT_PUBLIC_SERVER_URL` | Public URL of the server app (browser-accessible) |
| `SERVER_INTERNAL_URL` | Internal URL used by Next.js server-side rewrites. Leave unset in local dev; set to `http://server:3000` in Docker production |

### Production deployment (Docker Compose)

```bash
# From the monorepo root (ilenia-rag-frontend/)
cp apps/server/.env.example .env   # single .env file consumed by all services
# Edit .env with production values

docker compose up -d --build
```

This starts three containers:
- `ilenia-db-prod` — MySQL 8 (port 3326)
- `ilenia-server-prod` — API server (port 3000), runs migrations on startup
- `ilenia-web-prod` — Web frontend (port 3001)

## Troubleshooting

**MySQL container not ready**
The server waits for the database to be healthy before starting. If migrations fail on production startup, check that the `ilenia-db-prod` container is in `healthy` state:
```bash
docker compose ps
```

**`RAG_SERVERS` required error on startup**
Starting with `RAG_PROVIDER=real` without defining `RAG_SERVERS` or `RAG_MASTER_URL` will throw an initialization error. Either set the required variables or switch to `RAG_PROVIDER=mock` for local development.

**TLS errors connecting to RAG backends**
In development, TLS verification is automatically disabled (`NODE_TLS_REJECT_UNAUTHORIZED=0`). In production, ensure all RAG backends have valid certificates.

**CORS errors from the frontend**
Verify that `CORS_ORIGIN` in `apps/server/.env` exactly matches the URL serving the frontend (including protocol and port).

**Chats not persisting / session errors**
`BETTER_AUTH_SECRET` must remain the same value across server restarts. Changing it invalidates all existing sessions.
