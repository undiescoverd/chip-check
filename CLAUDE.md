# ChipQueue — Claude Code / Agent Guide

**Project:** ChipQueue — real-time order queue display for Two Little Fish (chippy pilot)
**Repo:** https://github.com/undiescoverd/chip-check.git
**Local path:** `/Users/ianvincent/workspace/ChipCheck`
**Stack:** Next.js 14 (App Router, TS) + Supabase (Postgres + Realtime) + Tailwind v3 + NextUI v2 + Vercel

---

## Read this first, every session

1. **`PROGRESS.md`** — the single source of truth for where we are. It has the current phase, per-phase Definition of Done checklists, notes, blockers, and deviations. Update it at the end of every session.
2. **`queue-display-PRD.md`** (in Ian's Downloads, not in repo) — the product spec.
3. **`queue-display-build-plan.md`** (in Ian's Downloads, not in repo) — the phased build plan with task-level detail.

> If you don't have the PRD/build plan in context, ask Ian to paste the relevant phase before starting work.

## Working rhythm

- **One phase per session.** Don't start the next phase until the current one's Definition of Done is ticked in `PROGRESS.md`.
- Start each session by reading `PROGRESS.md` to see the current status and any blockers.
- At the end of each session: update `PROGRESS.md` (tick boxes, update current status, note where you stopped), commit, and push.
- If the build plan gets something wrong, **flag it** (record in `PROGRESS.md` → Deviations) rather than working around it silently.

## Locked decisions (don't relitigate)

| Decision | Choice |
|---|---|
| Framework | Next.js 14 App Router + TypeScript |
| Component library | NextUI v2 (NOT HeroUI — `@nextui-org/*` despite deprecation notices) |
| Styling | Tailwind v3 + NextUI theme (`tailwind.config.ts`, not Tailwind v4 CSS config) |
| Database + Realtime | Supabase |
| Hosting | Vercel |
| Icons | lucide-react |
| Order numbers | Stored as `text` (preserves leading zeros) |
| Auth | Staff PIN validated server-side in a Route Handler |
| API keys | Supabase **publishable/secret** keys (`sb_publishable_...`/`sb_secret_...`), NOT legacy `anon`/`service_role` |

## Environment variables

```
NEXT_PUBLIC_SUPABASE_URL=                    # dev project URL
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=        # sb_publishable_... (client, read-only)
SUPABASE_SECRET_KEY=                         # sb_secret_... (SERVER ONLY — never NEXT_PUBLIC_)
STAFF_PIN=                                   # server-side, gates the write API
NEXT_PUBLIC_READY_TIMEOUT_SECONDS=90         # how long ready orders stay on the display
```

- `.env.local` holds **dev** project keys (gitignored). One set is enough for local dev.
- `.env.local.example` is the committed template.
- **Two Supabase projects:** `chipqueue-dev` (local + Vercel Preview) and `chipqueue-prod` (Vercel Production only). Ian creates these.
- The 6-hour stale-purge window is a **code constant**, not an env var.

## Architecture invariants (from PRD §11/§12)

- **Read path:** browser uses publishable key → `SELECT` + Realtime only. RLS policy `for select to anon` (publishable key resolves to `anon` role).
- **Write path:** all writes go through a Next.js Route Handler (`app/api/orders/route.ts`) using the **secret key** (`lib/supabaseAdmin.ts`, server-only) + PIN validation. The browser never writes directly.
- **No hard deletes** — clearing is a soft delete (`cleared = true`).
- **`ready` orders auto-clear from the display** after 90s (display-layer concern). **`preparing` orders never auto-clear** — only the 6h stale purge can touch them.
- **Optimistic UI** on the acting tablet; Realtime reconciles the other tablet. Target latency < 1.5s.
- Supabase is the single source of truth — no order state in localStorage. The only browser storage exception is `sessionStorage` for the staff PIN (Phase 5).

## Deviations from the original plan (already agreed — don't revert)

1. Supabase publishable/secret keys instead of legacy anon/service_role (Ian's request; legacy deprecated).
2. Pinned to `create-next-app@14` — `@latest` ships Next 16 + Tailwind v4 which breaks NextUI v2 setup.
3. Phase 5 will use a `POST /api/auth/verify` endpoint for PIN-screen feedback (Route Handler remains the real gate).

## Commands

```
npm run dev      # local dev server
npm run build    # production build (also typechecks)
npm run lint     # eslint
```

## Git

- Branch: `main`
- Remote: `origin` → https://github.com/undiescoverd/chip-check.git
- Never commit `.env.local` (gitignored). Never commit secrets.
- Commit messages: concise, match repo style (see `git log --oneline`).

## NextUI gotchas (Phase 0, already handled — keep these if regenerating config)

1. `framer-motion` must be installed as an explicit peer dep.
2. Tailwind `content` array must include `./node_modules/@nextui-org/theme/dist/**/*.{js,ts,jsx,tsx}` or components won't style.
3. `NextUIProvider` must wrap the entire app in `app/layout.tsx`.
