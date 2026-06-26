# `src/lib`: Shared utilities, sync layer, and external integrations.

## WHERE TO LOOK

| Concern | File | Notes |
|---|---|---|
| Supabase client | `supabase.ts` | Anon-key client only. Never the service_role key. |
| Local-first sync | `sync.ts` | Serial `localStorage` outbox that flushes optimistic mutations to Supabase. |
| Google Calendar | `gcal.ts` | GIS OAuth, code exchange via `/api/google-token`, CRUD, and two-way drag reschedule. |
| Ordering | `position.ts` | `GAP = 1024`, `between()`, `rebalanced()`. |
| Task grouping | `group.ts` | Group-by helpers for table/board views plus a recursive checklist counter. |
| Date helpers | `dates.ts` | date-fns wrappers and Korean quick-capture parsing. |
| Mobile detection | `useIsMobile.ts` | React hook in `lib/` because the project has no `src/hooks/`. |

## CONVENTIONS

- **Anon Supabase only.** `supabase.ts` exports the anon-key client used by `sync.ts` and the stores.
- **Serial outbox.** Every optimistic mutation calls `enqueue()` in `sync.ts`. The queue is persisted under `pd-outbox-v1` and flushed one op at a time.
- **`flush()` on boot.** The module starts flushing on load, and `retryNow()` forces immediate retry after login or going back online.
- **Poison-op discard.** Sync ops that fail with PG codes `23503`, `23502`, `23514`, `22P02`, or `22001` are dropped so one bad op cannot block the queue.
- **Back-off for recoverable errors.** Non-poison failures stop the queue and schedule an exponential back-off retry.
- **GAP ordering.** Insert with `between(prev, next)`. If it returns `NaN`, rebuild positions with `rebalanced(count)`.
- **GCal auth flow.** Request a code with GIS `initCodeClient`, exchange it through the Vercel proxy, then keep a refresh token for silent re-auth.
- **Typed GCal failures.** `gcal.ts` returns `{ ok: false, reason: 'auth' | 'api_disabled' | 'error' }` so UI can ask for reconnect or enable the API.
- **Korean quick dates.** `dates.ts` parses strings like "내일", "다음주 월요일", and "5/20". Week helpers use Monday as the first day.
- **Hook in lib.** `useIsMobile.ts` subscribes to `matchMedia('(max-width: 767px)')` via `useSyncExternalStore`.

## ANTI-PATTERNS

- Do not call Supabase directly from components. Route writes through the store so the outbox stays in sync.
- Do not refetch after a failed sync push. Preserve local intent and let the queue retry or discard.
- Never import a service_role key or call Supabase admin APIs from this directory.
- Do not mutate `task.position` without collision checking. Always use `between()` and rebalance when needed.
