# State Management

State is split across four focused Zustand stores; the domain store owns almost all app data and UI navigation state.

## WHERE TO LOOK

| Store | File | Responsibility |
|---|---|---|
| Domain | `store.ts` | Workspaces, folders, phases, projects, tasks, today sections, keyboard nav, undo, derived selectors |
| Auth | `authStore.ts` | Optional Google sign-in, session readiness, RLS-aware `ready` flag |
| Google Calendar | `gcalStore.ts` | OAuth token lifecycle, calendar/event cache, two-way GCal writes |
| Dialog | `dialogStore.ts` | Imperative `promptDialog()` / `confirmDialog()` host |

## CONVENTIONS

- `useStore` is a 584-line monolithic domain store. Keep new CRUD here unless it clearly belongs to auth, GCal, or dialogs.
- Every mutating action is optimistic: call `set()` to update local state first, then `enqueue()` to persist through the serial sync outbox.
- Selectors that return arrays or objects must use `useShallow` in components to avoid extra renders.
- Derived task buckets are computed, not stored. Use `bucketOf(task)` or the exported selectors (`selInbox`, `selToday`, `selOverdue`, `selScheduled`, `selWeek`, `selDated`, `projectStats`) instead of rewriting filtering logic.
- Keyboard navigation lives in `useStore`: `navOrder`, `navKind`, `hoverTaskId`, `quickFocus`, `tabNav`, `sidebarFocus`. Pages register their order with `useNavOrder(ids, kind)` and tab switching with `useViewTabs(keys, active, setActive)`.
- Undo is limited to task changes. `undoStack` is capped at 50 entries. `suppressUndo` prevents recursive entries when `undo()` itself calls mutators.
- `fetchAll()` sets `loaded: true` once. After the first load, it skips refetching while the sync outbox has pending ops so optimistic local edits are not overwritten.
- Ordering uses `GAP = 1024` positions. Reorder mutators call `rebalance(ids, field)` or `reorderProjects(ids)` when gaps collapse.
- `addTask()` clones recurring tasks on completion via `nextOccurrence()` and resets checklist trees with `resetCk()`.
- IDs come from `nid('prefix')`. Valid prefixes in this layer include `ws`, `fd`, `ph`, `pr`, `t`, `sec`, `ck`.

## ANTI-PATTERNS

- Do not call Supabase directly from components or pages; route writes through the store so `sync.ts` enqueues them.
- Do not refetch after a failed sync push; local intent wins and the outbox retries or discards poison ops.
- Avoid adding non-domain UI state to `store.ts`; use `dialogStore` for modals and local state for ephemeral UI.
