# init-bun-svelte-elysia

Agent skill untuk scaffold monorepo fullstack:

**Bun + TypeScript** · **SvelteKit** (FE) · **ElysiaJS + Drizzle ORM** (BE) · **PostgreSQL** · Docker Compose · `.env.example` · README · Makefile tipis

Skill ini mengikuti format [Agent Skills](https://skills.sh/) (`SKILL.md`), jadi bisa dipakai di banyak coding agent.

## Kompatibel dengan

Satu perintah install bisa menargetkan agent yang terpasang di mesinmu, misalnya:

- [Cursor](https://cursor.com/)
- [Claude Code](https://claude.ai/code)
- [OpenAI Codex](https://openai.com/codex/)
- OpenCode, Windsurf, Cline, Gemini CLI, dan agent lain yang didukung [skills CLI](https://github.com/vercel-labs/skills)

Lihat daftar agent: [vercel-labs/skills](https://github.com/vercel-labs/skills#supported-agents).

## Install

Butuh Node.js (`npx`).

### Semua agent yang terdeteksi

```bash
npx skills add imampamuji/init-bun-svelte-elysia
```

### Global (user-level)

```bash
npx skills add imampamuji/init-bun-svelte-elysia -g
```

### Agent tertentu

```bash
# Cursor
npx skills add imampamuji/init-bun-svelte-elysia -a cursor -g

# Claude Code
npx skills add imampamuji/init-bun-svelte-elysia -a claude-code -g

# Codex
npx skills add imampamuji/init-bun-svelte-elysia -a codex -g

# Beberapa sekaligus
npx skills add imampamuji/init-bun-svelte-elysia -a cursor -a claude-code -a codex -g
```

### Manual

Salin folder ini ke lokasi skill agent-mu, misalnya:

| Agent | Path (global, contoh) |
|-------|------------------------|
| Cursor | `~/.cursor/skills/init-bun-svelte-elysia/` atau `~/.agents/skills/` |
| Claude Code | `~/.claude/skills/init-bun-svelte-elysia/` |
| Codex | `~/.codex/skills/init-bun-svelte-elysia/` |

Pastikan di dalamnya ada `SKILL.md` (dan `reference.md`).

## Cara pakai

1. Install skill (lihat di atas), lalu buka proyek/folder tujuan di agent.
2. Minta agent menjalankan skill ini. Contoh prompt:

```text
Pakainya skill init-bun-svelte-elysia.
Init proyek monorepo di folder ini, nama: my-app.
```

Prompt lain yang biasanya memicu skill:

```text
Scaffold fullstack Bun + SvelteKit + Elysia + Drizzle + Postgres
```

```text
Init project Bun Svelte Elysia
```

3. Agent akan menanyakan/memakai nama proyek (`kebab-case`), lalu membuat layout kira-kira:

```text
my-app/
├── apps/
│   ├── api/          # Elysia + Drizzle
│   └── web/          # SvelteKit
├── docker-compose.yml
├── Makefile
├── .env.example
├── package.json
└── README.md
```

4. Setelah scaffold selesai, alur lokal yang umum:

```bash
cp .env.example .env
make install
make up
make db-push
make dev
```

- Web: `http://localhost:5173`
- API health: `http://localhost:3000/health`

Default skill **tidak** menyertakan auth, CI, atau shared package. Minta terpisah jika perlu.

Detail langkah agent ada di [`SKILL.md`](./SKILL.md); template file di [`reference.md`](./reference.md).

## Referensi tech stack

| Layer | Docs |
|-------|------|
| Bun (runtime, workspaces) | https://bun.com/docs |
| Bun workspaces | https://bun.com/guides/install/workspaces |
| SvelteKit | https://svelte.dev/docs/kit |
| ElysiaJS | https://elysiajs.com/ |
| Elysia + Drizzle | https://elysiajs.com/integrations/drizzle |
| Drizzle ORM | https://orm.drizzle.team/docs/overview |
| Drizzle + PostgreSQL | https://orm.drizzle.team/docs/get-started/postgresql-new |
| PostgreSQL | https://www.postgresql.org/docs/ |
| Docker Compose | https://docs.docker.com/compose/ |

## Belajar Agent Skills

Konsep format `SKILL.md` dan cara install di berbagai agent: **Programmer Zaman Now (PZN)**

- [YouTube: Agent Skills](https://www.youtube.com/watch?v=H9n2eYPX4wg)

Ekosistem:

- [skills.sh](https://skills.sh/)
- [vercel-labs/skills (CLI)](https://github.com/vercel-labs/skills)
