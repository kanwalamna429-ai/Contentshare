# Universal Content Distribution Platform (UCDP)

A production-ready SaaS app to republish blog articles across 12 platforms from a single dashboard.

## Run & Operate

- `cd universal-content-distribution && ./node_modules/.bin/next dev --port 3000` — run the Next.js app (port 3000)
- `pnpm install --ignore-workspace` — install UCDP dependencies (must run from `universal-content-distribution/` with this flag)
- `pnpm --filter @workspace/api-server run dev` — run the API server (port 5000)
- `pnpm run typecheck` — full typecheck across all packages

## Stack

- **App**: Next.js 16 (Turbopack), React 19, TypeScript 5.9, TailwindCSS v4
- **Auth/DB**: Supabase (PostgreSQL + RLS + Edge Functions)
- **AI**: Google Gemini
- **Components**: Custom shadcn-style components (no CLI)
- **Deployment**: Vercel target
- **Workspace**: pnpm monorepo root, but UCDP is intentionally NOT in the workspace packages list

## Where things live

```
universal-content-distribution/
├── app/
│   ├── (auth)/login/        — login/signup page
│   ├── (dashboard)/         — all protected pages (layout + sidebar)
│   │   ├── dashboard/       — stats + activity
│   │   ├── connections/     — platform OAuth connections
│   │   ├── url-library/     — saved URLs
│   │   ├── campaigns/       — campaign list + new campaign wizard
│   │   ├── logs/            — distribution logs
│   │   └── settings/        — user settings
│   └── api/                 — all API routes
├── lib/
│   ├── platforms/           — 12 platform adapters (bluesky, mastodon, etc.)
│   ├── supabase/            — client.ts + server.ts
│   ├── encryption.ts        — AES-256-GCM for platform credentials
│   ├── gemini.ts            — Gemini AI content generation
│   ├── metadata.ts          — URL metadata extraction (cheerio)
│   └── scheduling.ts        — scheduling utilities
├── types/index.ts           — all TypeScript types + PLATFORM_CONFIGS
├── proxy.ts                 — auth proxy (Next.js 16 convention, replaces middleware.ts)
├── supabase/migrations/     — 001_initial_schema.sql (11 tables, RLS, triggers)
├── .npmrc                   — ignore-workspace=true, shamefully-hoist=true
└── SETUP.md                 — full setup guide
```

## Required Environment Variables

Add these in Replit Secrets:

| Variable | Description |
|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | From Supabase project Settings → API |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | From Supabase project Settings → API |
| `SUPABASE_SERVICE_ROLE_KEY` | From Supabase project Settings → API |
| `ENCRYPTION_KEY` | 32-char random string for credential encryption |
| `GEMINI_API_KEY` | Google AI Studio API key |

## Supported Platforms (12 total)

Bluesky, Mastodon, Misskey, Pixelfed, Dev.to, Hashnode, Reddit, Tumblr, Diigo, Raindrop.io, Pocket, Instapaper

## Architecture decisions

- UCDP is kept **outside** the pnpm workspace packages list to avoid version conflicts (React 19.2.4 vs workspace catalog 19.1.0, Zod v4 vs v3). Uses `--ignore-workspace` install + local `.npmrc`.
- `proxy.ts` (not `middleware.ts`) — Next.js 16 renamed the auth proxy convention.
- `serverExternalPackages: ['cheerio']` (not `experimental.serverComponentsExternalPackages`) — Next.js 16 moved this out of experimental.
- All platform credentials are encrypted at rest with AES-256-GCM before storing in Supabase.
- Content generation uses Gemini AI with platform-specific formatting (character limits, hashtag styles, thread support).

## Product

Users paste a URL → UCDP extracts metadata and content → Gemini AI generates platform-optimized variants → User selects which of 12 platforms to publish to → Campaign runs immediately or on schedule → Logs track every distribution.

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- **Always install with `pnpm install --ignore-workspace`** from within `universal-content-distribution/` — the root pnpm workspace has React/Zod version conflicts that put Next.js in `.ignored` otherwise.
- **Never add `universal-content-distribution` to `pnpm-workspace.yaml` packages** — it will break binary linking.
- Next.js 16 uses `proxy.ts` not `middleware.ts`, and exports `proxy` function not `middleware`.
- `allowedDevOrigins: ['*.pike.replit.dev', '*.replit.dev']` is required in `next.config.ts` for Replit preview to work.

## Pointers

- See `universal-content-distribution/SETUP.md` for full Supabase setup steps
- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
