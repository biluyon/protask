# src/components/

## OVERVIEW
Reusable React UI surface for Protask: navigation, task rows, modals, popovers, context menus, drag-and-drop surfaces, and domain-specific views under `project/` and `workspace/`.

## STRUCTURE
- Top-level: shared, page-agnostic components used across multiple routes.
- `project/`: views scoped to a single project.
- `workspace/`: multi-project or board-level views.

## WHERE TO LOOK
| Concern | File | Notes |
|---|---|---|
| Sidebar + folder DnD + mobile drawer | `Sidebar.tsx` | Includes `SyncDot` status indicator. |
| Inline task tree row | `TaskRow.tsx` | Checklist tree, project picker, schedule chip. |
| Quick capture modal | `QuickCapture.tsx` | Bound to `Ctrl+K`. |
| Global keyboard shortcuts | `Shortcuts.tsx` | Shortcut matrix; consumes `data-navid` targets. |
| Task detail editor | `TaskDetail.tsx` | Full-page or modal task form. |
| Imperative modal host | `DialogHost.tsx` | Driven by `dialogStore`; don't open modals ad-hoc. |
| Nested JSON checklist | `Checklist.tsx` | Renders the `tasks.checklist` tree. |
| Authentication UI | `Login.tsx` | Google sign-in entry point. |
| Generic modal shell | `Modal.tsx` | Backdrop, focus trap, portal wrapper. |
| Schedule / plan popover | `PlanPopover.tsx` | Date picker + bucket planner. |
| Project badge/chip | `ProjectChip.tsx` | Color-coded project label. |
| Task right-click menu | `TaskContextMenu.tsx` | Paired with `TaskRow`. |
| Google Calendar event modal | `GcalEventModal.tsx` | GCal event details + edit proxy. |
| Notion-style grouped table | `project/ProjectTable.tsx` | Group headers + row DnD. |
| Kanban board | `workspace/SubprojectBoard.tsx` | Column + card drag surfaces. |

## CONVENTIONS
- `data-navid` attributes mark navigable rows for global keyboard nav via `Shortcuts`.
- Drop identifiers use typed prefixes: `folder:` (sidebar folders), `grp:` (task groups), `sec:` (sections / board columns).
- Context menus use the shared `useContextMenu` hook and `MenuItem` components.
- Hover-only actions use `group-hover:visible touch:visible` so touch users can still reach them.
- Store selections returning arrays or objects must use `useShallow`.
- UI strings are mixed Korean and English; keep existing labels consistent.

## ANTI-PATTERNS
- Don't mutate refs during render. Use `useEffect` for imperative side effects.
- Don't inline shell components in new files; `App.tsx` already owns the shell layout.
- Don't create ad-hoc modal markup. Open modals through `DialogHost` and `dialogStore`.
- Don't invent new DnD prefix schemes; extend `folder:`, `grp:`, `sec:` only when needed.
