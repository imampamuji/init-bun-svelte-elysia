---
name: init-bun-svelte-elysia
description: >-
  Scaffold a full-TypeScript Bun monorepo with SvelteKit frontend, ElysiaJS
  backend, Drizzle ORM, PostgreSQL, Docker Compose, .env.example, README, and a
  thin Makefile. Use when the user asks to init/scaffold a new project with Bun,
  Svelte/SvelteKit, Elysia, Drizzle, or this fullstack stack.
---

# Init Bun + SvelteKit + Elysia

Scaffold a **Bun workspaces monorepo**: SvelteKit FE, Elysia + Drizzle BE, Postgres via Docker Compose.

Prefer Indonesian replies when the user writes in Indonesian.

## Stack (fixed defaults)

| Layer | Choice |
|-------|--------|
| Runtime / PM | Bun + TypeScript |
| Layout | Monorepo: `apps/web`, `apps/api` |
| Frontend | SvelteKit (TypeScript) |
| Backend | ElysiaJS |
| ORM / DB | Drizzle ORM + PostgreSQL |
| Ops | `docker-compose.yml` (Postgres only), `.env.example`, `README.md`, thin `Makefile` |

Do **not** add auth, CI, shared packages, or test harness unless the user asks.

## Preconditions

1. Confirm target directory (cwd or path). Abort if it already has a `package.json` unless the user wants to overwrite.
2. Ask for project name (kebab-case). Use it for root `package.json` `name` and Docker DB name.
3. Ensure `bun` and `docker` are available; stop and report if missing.

## Layout to create

```
<project>/
├── apps/
│   ├── api/                 # Elysia + Drizzle
│   └── web/                 # SvelteKit
├── docker-compose.yml
├── Makefile
├── .env.example
├── .gitignore
├── package.json             # workspaces root
├── README.md
└── bun.lock                 # after bun install
```

## Workflow checklist

Copy and track:

```
Init progress:
- [ ] Root workspace + gitignore
- [ ] Docker Compose Postgres + .env.example
- [ ] apps/api (Elysia + Drizzle)
- [ ] apps/web (SvelteKit)
- [ ] Root scripts + Makefile
- [ ] README
- [ ] bun install + smoke check
```

### 1. Root workspace

`package.json`:

```json
{
  "name": "<project>",
  "private": true,
  "workspaces": ["apps/*"],
  "scripts": {
    "dev": "bun run --filter '*' dev",
    "dev:api": "bun run --filter @<project>/api dev",
    "dev:web": "bun run --filter @<project>/web dev",
    "db:generate": "bun run --filter @<project>/api db:generate",
    "db:migrate": "bun run --filter @<project>/api db:migrate",
    "db:push": "bun run --filter @<project>/api db:push",
    "db:studio": "bun run --filter @<project>/api db:studio"
  }
}
```

`.gitignore`: include `node_modules`, `.env`, `.env.local`, `dist`, `.svelte-kit`, `build`, `*.log`, `.DS_Store`.

Do **not** put app dependencies in the root `package.json`.

### 2. Docker Compose + env

`docker-compose.yml`: single `postgres:16-alpine` service, port `5432`, volume for data, healthcheck. Credentials from env (`POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`).

`.env.example` (and copy to `.env` for local smoke):

```env
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=<project>
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/<project>
API_PORT=3000
PUBLIC_API_URL=http://localhost:3000
```

Wire `DATABASE_URL` into `apps/api` only. SvelteKit may use `PUBLIC_API_URL` for client calls later; do not invent auth secrets.

### 3. Backend: `apps/api`

Package name: `@<project>/api`.

Minimal shape:

```
apps/api/
├── package.json
├── tsconfig.json
├── drizzle.config.ts
├── src/
│   ├── index.ts          # Elysia app + CORS + health
│   ├── db/
│   │   ├── index.ts      # drizzle client (postgres.js or bun sql driver)
│   │   └── schema.ts     # starter table (e.g. users id + created_at) OR empty export
│   └── routes/
│       └── health.ts
└── drizzle/              # after first generate (may be empty until migrate)
```

Dependencies (install inside `apps/api` or via workspace filter):

- runtime: `elysia`, `@elysiajs/cors`, `drizzle-orm`, `postgres` (or Bun-compatible Postgres driver)
- dev: `drizzle-kit`, `typescript`, `@types/bun`

Scripts: `dev` (`bun --watch src/index.ts`), `db:generate`, `db:migrate`, `db:push`, `db:studio`.

`drizzle.config.ts`: schema `./src/db/schema.ts`, out `./drizzle`, dialect `postgresql`, `dbCredentials.url` from `process.env.DATABASE_URL`.

Elysia entry: listen on `API_PORT` (default 3000), expose `GET /health` → `{ ok: true }`. Enable CORS for `http://localhost:5173`.

Keep Drizzle wiring ready; one tiny starter table is fine so `db:push` is demonstrable. No auth.

For Elysia + Drizzle type helpers later, see official `drizzle-typebox` notes; do **not** add it on init unless asked.

### 4. Frontend: `apps/web`

Prefer official scaffold, then normalize:

```bash
cd apps
bunx sv create web --template minimal --types ts --no-install
```

If the CLI prompts, choose: TypeScript, minimal template, no extra add-ons unless user asked. Then set package `name` to `@<project>/web`, ensure `dev` script uses Vite/SvelteKit defaults (`vite dev` / `bun --bun vite dev` as appropriate).

Add a simple home page that fetches `PUBLIC_API_URL/health` (or hardcode `http://localhost:3000/health` with a short comment to switch to env) so the monorepo has an end-to-end smoke path.

Do not embed Elysia inside SvelteKit routes; keep FE and BE as separate apps.

### 5. Makefile (thin)

Delegate to `bun` / `docker compose`. Do not duplicate business logic.

Targets:

| Target | Action |
|--------|--------|
| `install` | `bun install` |
| `up` | `docker compose up -d` |
| `down` | `docker compose down` |
| `logs` | `docker compose logs -f postgres` |
| `dev` | `bun run dev` |
| `dev-api` / `dev-web` | filtered scripts |
| `db-push` / `db-migrate` / `db-studio` | filtered db scripts |

Default goal: print help (`make` / `make help`).

### 6. README

Short, in Indonesian or English matching the user. Sections:

1. Stack
2. Prerequisites (`bun`, Docker)
3. Setup: copy `.env.example` → `.env`, `make install`, `make up`, `make db-push`, `make dev`
4. URLs: web `http://localhost:5173`, api `http://localhost:3000/health`
5. Useful Make / bun commands

### 7. Smoke check

1. `make install` (or `bun install` from root)
2. `make up` — wait until Postgres healthy
3. `make db-push` if schema exists
4. Start API briefly; `curl -s http://localhost:3000/health` should return ok
5. Report paths created and next commands; do not commit unless asked

## Defaults and escape hatches

- Ports: API `3000`, web `5173`, Postgres `5432`. If busy, ask before changing.
- Package scope: `@<project>/api` and `@<project>/web`.
- Skip Makefile only if the user says so; still keep root `package.json` scripts.
- Auth, Eden Treaty, shared `packages/*`, Vitest, ESLint, CI: **out of scope** until requested.

## Additional resources

- File templates and command snippets: [reference.md](reference.md)
