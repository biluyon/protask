# `src/pages/` — Route-level views

**OVERVIEW:** Each file here is a route-level view rendered inside `App.tsx`'s `<Routes>`; they read from the Zustand store and handle GTD bucket rendering, DnD drop targets, and Google Calendar overlay.

## WHERE TO LOOK

| Route | File | Notes |
|-------|------|-------|
| `/` | `Today.tsx` | Today list + section board; `groupBy` section/project; DnD between custom `today_sections`; overdue bucket. Complex (~448 lines). |
| `/inbox` | `Inbox.tsx` | GTD Inbox + Someday; grouped by workspace; quick-capture input. |
| `/upcoming` | `Upcoming.tsx` | Date buckets: overdue, today, tomorrow, this week, next week, later; merges Google Calendar events. |
| `/week` | `Week.tsx` | Weekly planner board: 7 day columns + backlog; DnD assigns `scheduled_date` and `today_section`. (~356 lines) |
| `/scheduled` | — | Redirects to `/upcoming`. |
| `/someday` | — | Redirects to `/inbox`. |
| `/calendar` | `Calendar.tsx` | Month/week toggle; side Inbox/Someday panel; drag tasks onto days; two-way Google Calendar overlay. Largest (~491 lines). |
| `/workspaces` | `WorkspaceList.tsx` | Mobile tab workspace index with completion stats. |
| `/w/:wsId` | `Workspace.tsx` | Project list or board view per workspace; status filters; uses `SubprojectBoard` / `ProjectTable`. |
| `/settings` | `Settings.tsx` | App config, Google Calendar sign-in, JSON backup export. |
| `/guide` | `Guide.tsx` | Static user manual for GTD + project hierarchy. |

## PAGE CONVENTIONS

- **Route paths live in `App.tsx`.** Do not add a separate route config; import pages there as default components.
- **Default exports named `*Page`.** `App.tsx` imports `TodayPage` as `Today`, etc.
- **Store selectors:** read arrays/objects with `useShallow`; prefer fine-grained selectors for scalars.
- **Derived GTD buckets:** pages do not own bucket state. Inbox/Today/Scheduled/Someday are computed by selectors (`selInbox`, `selToday`, `selDated`, `selSomeday`) from `status`, `someday`, and `scheduled_date`.
- **Keyboard navigation:** call `useNavOrder(ids)` with the visible task order so `Shortcuts.tsx` can move focus via `data-navid`.
- **DnD sensors:** use `PointerSensor({ distance: 5 })` + `TouchSensor({ delay: 200, tolerance: 8 })`. Drop target IDs encode context (e.g., `date::sectionId` in `Week.tsx`, day keys in `Calendar.tsx`).
- **View state in `localStorage`:** persist toggles like `pd-todaygroup`, `pd-wsview`, `pd-calpanel` with fallbacks.
- **Google Calendar:** pages that show events (Today, Upcoming, Calendar) call `useGcal()` and run `gcal.init()` in `useEffect`; respect `gcal.status` before calling `ensureRange`.
