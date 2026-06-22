# ChipQueue — Build Progress

**Project:** ChipQueue — real-time order queue display for Two Little Fish
**Repo:** https://github.com/undiescoverd/chip-check.git
**Local path:** `/Users/ianvincent/workspace/ChipCheck`
**PRD:** `queue-display-PRD.md` (kept in Downloads, not in repo)
**Build plan:** `queue-display-build-plan.md` (kept in Downloads, not in repo)

> **For any agent picking up:** Read this file first. It tells you which phase we're on, what's done, and what's blocked. Then read the PRD + the relevant phase from the build plan before continuing.

---

## How to use this file

- Work **one phase per session**.
- Before starting a phase, read the PRD and the matching phase from `queue-display-build-plan.md`.
- Tick the checkboxes as you verify each Definition of Done item.
- Update the **Current status** block at the top every time you finish a session.
- If a phase is partially done, note exactly where you stopped in the phase's **Notes** field.
- If you flag a deviation from the plan, record it in **Deviations from plan** at the bottom.

---

## Current status

**Active phase:** Phase 1 — Database & data layer (not started)
**Phase 0:** ✅ Complete and pushed to `origin/main`
**Blockers:** Supabase project(s) need to be created by Ian; `.env.local` needs real keys filled in before Phase 1 can be verified.

**Last commit:** `237fe18 Remove stray excalidraw.log from tracking; ignore it`
**Branch:** `main`

---

## Stack & key decisions (locked)

| Decision | Choice | Reason |
|---|---|---|
| Framework | Next.js 14 App Router + TypeScript | Ian's standard stack |
| Component library | NextUI v2 | Modern, premium feel, Framer Motion built in |
| Animation | Framer Motion (NextUI peer dep) | Already installed, no extra cost |
| Database + Realtime | Supabase | Real-time subs, free tier, Ian knows it |
| Styling | Tailwind v3 + NextUI theme | Standard |
| Hosting | Vercel | Standard |
| Icons | lucide-react | Standard |
| Order numbers | Stored as `text` | Preserves leading zeros to match receipt |
| Auth | Staff PIN via Route Handler | Lightweight, no user accounts needed |
| API keys | Supabase publishable/secret keys | Legacy anon/service_role deprecated |

---

## Environment variables

Local dev needs one set (dev project). Production keys go in Vercel dashboard only.

```
NEXT_PUBLIC_SUPABASE_URL=                    # dev project URL
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=        # sb_publishable_... (client, read-only)
SUPABASE_SECRET_KEY=                         # sb_secret_... (SERVER ONLY — never NEXT_PUBLIC_)
STAFF_PIN=                                   # server-side, gates the write API
NEXT_PUBLIC_READY_TIMEOUT_SECONDS=90         # how long ready orders stay on the display
```

- `.env.local` holds **dev** keys (gitignored, never committed).
- `.env.local.example` is the committed template.
- **Two Supabase projects** recommended: `chipqueue-dev` (local + Vercel Preview) and `chipqueue-prod` (Vercel Production only).
- The 6-hour stale-purge window is a **code constant**, not an env var.

---

## Phase 0 — Project setup & foundations ✅

**Goal:** A running Next.js app deployed to Vercel with Supabase connected. Nothing functional yet, just plumbing.

**Status:** ✅ Complete (commit `de203ab`, key rename `4f52232`, log cleanup `237fe18`)

### Definition of done
- [x] App runs locally (`npm run dev`)
- [x] NextUI Button renders correctly with no console errors
- [x] Supabase client imports without error (`lib/supabase.ts`)
- [x] Env vars set locally (`.env.local` with placeholders — Ian to fill real dev keys)
- [x] `.env.local.example` committed as template
- [ ] Deploys cleanly to Vercel — **pending Ian**: connect repo to Vercel, set env vars
- [ ] Env vars set in Vercel dashboard — **pending Ian**

### Notes
- Scaffolded with `create-next-app@14` (NOT `@latest`) to get Next 14 + Tailwind v3 — `@latest` now ships Next 16 + Tailwind v4, which breaks the NextUI v2 setup the plan is built around.
- Three NextUI gotchas handled: `framer-motion` peer dep installed, Tailwind content glob includes `./node_modules/@nextui-org/theme/dist/**/*.{js,ts,jsx,tsx}`, `NextUIProvider` wraps app in `layout.tsx`.
- Switched from legacy Supabase keys (`anon`/`service_role`) to new publishable/secret keys (`sb_publishable_...`/`sb_secret_...`). RLS policies still target the `anon` Postgres role — only the key format changed, roles are unchanged.
- NextUI packages show deprecation notices pointing to `@heroui/*`; they still work. Sticking with NextUI v2 per the plan.
- `excalidraw.log` was stray editor temp; now gitignored.

### What Ian still needs to do for Phase 0
1. Create **two** Supabase projects: `chipqueue-dev` and `chipqueue-prod`.
2. Fill `.env.local` with **dev** project keys (publishable + secret + URL + a `STAFF_PIN`).
3. Connect the repo to Vercel; set env vars (Production → prod keys, Preview → dev keys).
4. Confirm a clean Vercel deploy.

---

## Phase 1 — Database & data layer ⬜

**Goal:** The `orders` table exists (via a version-controlled migration), Realtime is on, the browser can read it, and all writes go through a server-side Route Handler using the secret key.

**Status:** ⬜ Not started

### Definition of done
- [ ] Running `0001_init.sql` against a fresh Supabase project creates the table, indexes, and RLS policy
- [ ] Browser (publishable key) can read orders and receive Realtime updates
- [ ] Browser (publishable key) **cannot** insert/update — a direct write attempt is rejected by RLS
- [ ] A write through the Route Handler **with** the correct PIN succeeds
- [ ] A write through the Route Handler **without**/with a wrong PIN returns 401 and does nothing
- [ ] Secret key is server-only — not present in any client bundle
- [ ] Realtime shows as enabled in Supabase dashboard
- [ ] TypeScript types compile without errors

### Notes
- Use env var names: `SUPABASE_SECRET_KEY` (not `SUPABASE_SERVICE_ROLE_KEY`), `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` (not `NEXT_PUBLIC_SUPABASE_ANON_KEY`).
- `lib/supabaseAdmin.ts` (server-only, secret key) + `lib/supabase.ts` (browser, publishable key) — already have the latter.
- Migration RLS policy `for select to anon` stays as-is — publishable key resolves to the `anon` role.
- Route Handler at `app/api/orders/route.ts` validates PIN + uses secret-key client for all writes.
- `purgeStaleOrders()` — 6h code constant, not env var.

---

## Phase 2 — Staff view (cashier + packer share this) ⬜

**Goal:** A working staff screen — add orders via keypad, see the live list, mark ready, recall, clear. Syncs in real time across both tablets.

**Status:** ⬜ Not started

### Definition of done
- [ ] Type `0022`, hit Add → appears in list with amber Preparing chip
- [ ] List is ordered oldest-first (next to serve at top)
- [ ] Open `/staff` in two browser tabs simultaneously → action on one tab appears on the other in < 1.5s
- [ ] Mark Ready: card flips to green Ready chip
- [ ] Recall: green card returns to amber
- [ ] Clear: single card disappears from list
- [ ] Clear All: confirm modal, then board empties
- [ ] Duplicate **active** number triggers Modal warning; a reused-but-cleared number does not
- [ ] Marking an already-ready order ready again does nothing bad
- [ ] Survives browser refresh — state reloads from Supabase
- [ ] Touch targets comfortable on a tablet

### Notes
- Optimistic UI on the acting tablet; Realtime reconciles the other tablet.
- PIN gate UI is **not** this phase — it's Phase 5. Phase 2 builds the functional staff screen; the Route Handler still enforces the PIN server-side from Phase 1, so writes will need a PIN. For Phase 2 testing, either send a test PIN or stub the PIN entry simply. (Phase 5 adds the proper PIN entry screen.)

---

## Phase 3 — Customer display (TV + phone) ⬜

**Goal:** The public-facing screen. Big, clear, real-time. Works on a wall TV and on customer phones via the same URL.

**Status:** ⬜ Not started

### Definition of done
- [ ] `/display` shows live queue — updates in real time as staff use `/staff`
- [ ] Order numbers readable from across the room on a TV
- [ ] Ready animation fires when an order is marked ready
- [ ] `ready` orders auto-disappear 90s after going ready
- [ ] `preparing` orders stay put no matter how long they wait
- [ ] Busy board (15+ orders) lays out without clipping off-screen
- [ ] Looks clean and readable on a phone screen
- [ ] Empty state looks intentional, not broken
- [ ] Connection indicator works — shows reconnecting if WiFi drops briefly
- [ ] `?sound=1` plays a chime when new Ready orders appear

### Notes
- Auto-clear of `ready` orders is **display-only** (visually filter); the 6h DB purge from Phase 1 is the hard cleanup.
- Realtime connections cap at 24h — connection indicator + auto-reconnect handles overnight drops.

---

## Phase 4 — QR code & customer phone access ⬜

**Goal:** Customers can scan a QR code to open the live display on their own phone.

**Status:** ⬜ Not started

### Definition of done
- [ ] QR code scans correctly and opens `/display` on a phone
- [ ] Printable QR PNG asset exists
- [ ] `/order/[number]` route exists and doesn't 404

### Notes
- `npm install qrcode @types/qrcode`.
- `/order/[number]` is a v2 stub — simple "coming soon" page.

---

## Phase 5 — Hardening & polish ⬜

**Goal:** Robust enough to leave running unattended through a busy service. Looks intentional.

**Status:** ⬜ Not started

### Definition of done
- [ ] `/staff` requires PIN — wrong PIN rejected
- [ ] Writes rejected at server level without valid PIN
- [ ] All errors show toast, no silent failures
- [ ] No uncaught errors in a 1-hour soak test with active orders
- [ ] Kiosk doc written and tested on an actual tablet + TV

### Notes
- PIN entry UX added here (the write API was PIN-gated from Phase 1 — this is the front-end gate + `sessionStorage` session handling).
- Deviation agreed: use a minimal `POST /api/auth/verify` endpoint for immediate PIN-screen feedback rather than relying on the first write's 401. The Route Handler remains the real enforcement point.
- `purgeStaleOrders()` runs on every app load + 30-min interval while display is open.
- Two Little Fish theming is optional — neutral by default.

---

## Phase 6 — Pilot at Two Little Fish ⬜

**Goal:** Run it live through a real busy service. Gather feedback. Fix friction.

**Status:** ⬜ Not started

### Definition of done
- [ ] Full service run completed without staff confusion
- [ ] No system errors or crashes during service
- [ ] At least one customer observed using the display unprompted
- [ ] v2 feedback list written up

### Notes
- Hardware setup: Tablet 1 (cashier) + Tablet 2 (packer) on `/staff`; TV on `/display?sound=0`; QR laminated on counter.
- Use the **production** Supabase project + Vercel Production env vars for the pilot.

---

## Deviations from plan

1. **Supabase key system:** Plan/PRD reference legacy `anon`/`service_role` keys. Switched to new `sb_publishable_...`/`sb_secret_...` keys per Ian's request (legacy deprecated). Env var names updated accordingly. RLS `to anon` policies unchanged — publishable key still resolves to the `anon` Postgres role.
2. **Next.js version:** `create-next-app@latest` now ships Next 16 + Tailwind v4, which breaks NextUI v2 setup. Pinned to `create-next-app@14` to match the plan (Next 14 + Tailwind v3).
3. **Phase 5 PIN verify endpoint:** Plan says "lightweight `POST /api/auth` check, or the first write's 401 response." Agreed to build a minimal `POST /api/auth/verify` for clean PIN-screen feedback — Route Handler remains the real gate.

---

## Open pilot questions (from PRD §16 — confirm before/during Phase 6)

- [ ] Does the packer station have reliable WiFi, or will it need a range extender?
- [ ] Will the TV be a smart TV with a usable browser, or does it need a Fire TV / Chromecast / mini PC?
- [ ] Does the shop want their branding on the display for the pilot, or keep it neutral? (Default: neutral)
- [ ] What's the realistic peak order count in a 30-min window? (Sanity-checks the busy-board layout. Build assumes 20+.)
