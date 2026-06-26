# PROJECT KNOWLEDGE BASE

**Generated:** 2026-06-26
**Commit:** N/A (not a git checkout)
**Branch:** N/A

## OVERVIEW

Protask — a self-hosted, single-instance workspace/project/GTD task manager. Vite 8 + React 19 + TypeScript 6 + Tailwind v4 + Zustand + Supabase. Local-first PWA with optimistic edits flushed through a serial sync outbox.

## STRUCTURE

```
/
├── src/               # React SPA
│   ├── components/    # UI components (keyboard nav, DnD, menus)
│   ├── pages/         # Route-level views (Today, Calendar, Workspace, ...)
│   ├── lib/           # Utilities, sync layer, Supabase client, GCal
│   ├── store/         # Zustand stores
│   ├── types.ts       # Shared domain types
│   ├── App.tsx        # Router + shell layout
│   └── main.tsx       # React bootstrap
├── api/               # Vercel serverless function (Google token proxy)
├── mcp/               # Python MCP server for AI-agent board access
├── scripts/           # Build helpers + one-shot migration
├── supabase/          # SQL migrations
└── public/            # PWA icons + static assets
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Bootstrap | `src/main.tsx` → `src/App.tsx` | Router, auth gate, shell |
| Domain state | `src/store/store.ts` | Monolithic 584-line Zustand store; all CRUD + undo + selectors |
| Auth state | `src/store/authStore.ts` | Optional Google sign-in + RLS support |
| Dialog state | `src/store/dialogStore.ts` | Imperative modal host |
| GCal state | `src/store/gcalStore.ts` | Token + connection status |
| Offline sync | `src/lib/sync.ts` | localStorage outbox; serial flush; poison-op discard |
| Supabase client | `src/lib/supabase.ts` | Anon key; same tables as MCP/migration |
| Positioning | `src/lib/position.ts` | `GAP = 1024`, `between()`, `rebalanced()` |
| Domain types | `src/types.ts` | `Task`, `Project`, `Workspace`, `Bucket`, enums |
| Google Calendar | `src/lib/gcal.ts` | OAuth + CRUD + two-way drag |
| DnD grouping | `src/lib/group.ts` | Drop-area helpers |
| Keyboard nav | `src/components/Shortcuts.tsx` | Global shortcut matrix |
| Quick capture | `src/components/QuickCapture.tsx` | `Ctrl+K` capture modal |
| Sidebar | `src/components/Sidebar.tsx` | Nav + folders + mobile drawer |
| Task row | `src/components/TaskRow.tsx` | Inline checklist tree |
| Calendar | `src/pages/Calendar.tsx` | Month/week + GCal overlay |
| Today | `src/pages/Today.tsx` | List + section board |
| DB schema | `supabase/migrations/*.sql` | Run in numeric order |
| Vercel API | `api/google-token.ts` | Google OAuth token exchange proxy |
| MCP server | `mcp/project_board_mcp_v2.py` | FastMCP tools over Supabase REST |
| Legacy migration | `scripts/migrate_blob.py` | Blob boards → normalized tables |

## CONVENTIONS

- **ESM** only; `"type": "module"`. No CommonJS.
- **Relative imports** everywhere — no `@/` path aliases configured.
- **Type-only imports** required (`verbatimModuleSyntax: true`).
- **Unused locals/parameters** are TS errors; prefix discardables with `_`.
- **Korean UI strings and comments** mixed with English identifiers.
- **IDs** use `nid('prefix')` → 12-char alphanumerics (mirrored in Python MCP).
- **Ordering** uses `GAP = 1024` with `between()`/`rebalanced()`.
- **Optimistic writes**: every store mutation does `set()` then `enqueue()`.
- **Zustand selectors** use `useShallow` for array/object returns.
- **Tailwind v4** via `@tailwindcss/vite`; no PostCSS config.

## ANTI-PATTERNS (THIS PROJECT)

- Do not mutate refs during render — use `useEffect`.
- Do not refetch after a failed sync push; preserve local intent.
- Poison sync ops (FK/not-null/check/type-mismatch PG codes) are discarded, not retried.
- Never commit secrets; keys live in `.env` only.
- Never expose the Supabase `service_role` key in the frontend.
- Do not point untrusted users at a shared instance — single-tenant by design.

## UNIQUE STYLES

- **Single-instance**: no login by default; one Supabase instance = one user (or trusted group).
- **GTD buckets are derived**: `inbox`/`today`/`scheduled`/`someday`/`done` computed from `status` + `someday` + `scheduled_date`.
- **Checklist as nested JSON** inside `tasks.checklist`, not a separate table.
- **Two-way Google Calendar overlay**: drag GCal events to reschedule in Google.
- **Today section board**: alternate board view alongside the task list.
- **Per-workspace Excalidraw canvas** for overview whiteboards.
- **`data-navid` attributes** target rows for global keyboard navigation.

## COMMANDS

```bash
npm run dev       # Vite dev server
npm run build     # tsc -b && vite build
npm run lint      # eslint .
npm run preview   # vite preview
```

## NOTES

- No test framework, formatter, or path aliases are configured.
- `tsconfig.node.json` only covers `vite.config.ts`; `api/google-token.ts` has no type-checking.
- `api/` and `mcp/` are separate runtimes documented here but kept minimal; see root `README.md` for deployment and Private mode setup.
- Migration `0007_project_archive_folders.sql` exists on disk but is not listed in README quick-start.