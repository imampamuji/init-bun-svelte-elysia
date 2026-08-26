# Reference: init-bun-svelte-elysia

Concrete file stubs. Prefer these when scaffolding; adapt names to `<project>`.

## docker-compose.yml

```yaml
services:
  postgres:
    image: postgres:16-alpine
    restart: unless-stopped
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-postgres}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-postgres}
      POSTGRES_DB: ${POSTGRES_DB:-app}
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-postgres} -d ${POSTGRES_DB:-app}"]
      interval: 5s
      timeout: 5s
      retries: 10

volumes:
  pgdata:
```

## Makefile

```makefile
.PHONY: help install up down logs dev dev-api dev-web db-push db-migrate db-studio

help:
	@echo "install     bun install"
	@echo "up          docker compose up -d"
	@echo "down        docker compose down"
	@echo "logs        postgres logs"
	@echo "dev         run all workspace dev scripts"
	@echo "dev-api     API only"
	@echo "dev-web     web only"
	@echo "db-push     drizzle push"
	@echo "db-migrate  drizzle migrate"
	@echo "db-studio   drizzle studio"

install:
	bun install

up:
	docker compose up -d

down:
	docker compose down

logs:
	docker compose logs -f postgres

dev:
	bun run dev

dev-api:
	bun run dev:api

dev-web:
	bun run dev:web

db-push:
	bun run db:push

db-migrate:
	bun run db:migrate

db-studio:
	bun run db:studio
```

## apps/api/package.json

```json
{
  "name": "@project/api",
  "version": "0.0.1",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "bun --watch src/index.ts",
    "start": "bun src/index.ts",
    "db:generate": "drizzle-kit generate",
    "db:migrate": "drizzle-kit migrate",
    "db:push": "drizzle-kit push",
    "db:studio": "drizzle-kit studio"
  },
  "dependencies": {
    "@elysiajs/cors": "^1.2.0",
    "drizzle-orm": "^0.39.0",
    "elysia": "^1.2.0",
    "postgres": "^3.4.5"
  },
  "devDependencies": {
    "@types/bun": "latest",
    "drizzle-kit": "^0.30.0",
    "typescript": "^5.7.0"
  }
}
```

Pin versions to whatever `bun add` resolves at scaffold time; the ranges above are guidance only.

## apps/api/drizzle.config.ts

```ts
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  schema: "./src/db/schema.ts",
  out: "./drizzle",
  dialect: "postgresql",
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

## apps/api/src/db/index.ts

```ts
import { drizzle } from "drizzle-orm/postgres-js";
import postgres from "postgres";
import * as schema from "./schema";

const url = process.env.DATABASE_URL;
if (!url) throw new Error("DATABASE_URL is required");

const client = postgres(url);
export const db = drizzle(client, { schema });
```

## apps/api/src/db/schema.ts

```ts
import { pgTable, serial, timestamp, text } from "drizzle-orm/pg-core";

export const notes = pgTable("notes", {
  id: serial("id").primaryKey(),
  body: text("body").notNull(),
  createdAt: timestamp("created_at", { withTimezone: true }).defaultNow().notNull(),
});
```

## apps/api/src/index.ts

```ts
import { Elysia } from "elysia";
import { cors } from "@elysiajs/cors";

const port = Number(process.env.API_PORT ?? 3000);

const app = new Elysia()
  .use(
    cors({
      origin: ["http://localhost:5173"],
    }),
  )
  .get("/health", () => ({ ok: true }))
  .listen(port);

console.log(`api listening on http://localhost:${app.server?.port}`);
```

## SvelteKit create flags

Prefer non-interactive when the CLI allows:

```bash
bunx sv create apps/web --template minimal --types ts --no-add-ons --no-install
```

If flags differ by CLI version, run with defaults closest to: minimal + TypeScript + no add-ons, then rename package to `@<project>/web`.

## Smoke curl

```bash
curl -s http://localhost:3000/health
# {"ok":true}
```
