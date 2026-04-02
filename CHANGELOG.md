# ELIM Production App — Changelog

## v5.11.0 (2026-04-01) — I/O Page Redesign + Polish

**I/O tab restructure — organized by physical device:**
- Tabs reduced from 4 to 3: Stage Box, Wing, Dante
- Outputs tab removed — outputs now live under Stage Box tab (both inputs and outputs are physically on the Midas DL251)
- Stage Box tab has an Inputs/Outputs sub-toggle (pill-style segmented control) — defaults to Inputs

**Stage Box input grouping:**
- `STAGE_BOX_GROUPS` constant defines 6 channel-range groups: Drums (1-12), Instruments (13-25), Talk Backs (26-27), Vocals (28-36), Wireless (37-40), Crowd (41-44)
- Group headers render inline with channel count — uppercase label, thin divider line, count badge
- "Other" fallback group catches user-added channels outside defined ranges
- Channels matched by numeric parsing of `ch` field; non-numeric channels fall to Other

**Output grouping (IEM vs Infrastructure):**
- `isIEM()` helper filters by `destination` containing "in-ear" or "in ear" (case-insensitive)
- "In-Ear Monitors" group: IEM transmitters and wired in-ear channels
- "Infrastructure" group: subs, mains, unused channels

**In-ear snake pill badge:**
- `inEarSnake` field now renders as a prominent pill badge: `🎧 {value}` (e.g., "🎧 Michael", "🎧 Snake 7")
- Styled with accent glow background, accent border, 12px bold text, 20px border-radius
- Replaces previous 11px mono text (`Snake: {value}`)

**Cross-tab I/O search:**
- 🔍 search icon in toolbar opens search input (matches checklist search pattern)
- Search filters across all sections simultaneously: Stage Box inputs, Wing direct inputs, outputs, Dante routes
- Results grouped by section with headers and match counts
- Normal tab/sub-toggle view hidden during active search
- "No matches" empty state when search finds nothing

**I/O edit permissions:**
- Edit mode (✏️ pencil) restricted to Sam and Bri (was: anyone could edit)
- `currentUser` passed as new prop to `IOListView`
- Non-authorized users see only search icon, no edit button

**I/O edit button restyled:**
- Full-width "✏️ Edit I/O" button replaced with compact pencil icon (✏️) matching checklist edit pattern
- Edit mode shows "✓ Done" + "Cancel" buttons (compact, not full-width)

**Custom IODropdown component:**
- New `IODropdown` component replaces native `<select>` elements in I/O edit mode
- Dark-themed dropdown panel matching `AssignDropdown` visual pattern: `#1C1A18` background, hover highlights, checkmark on current selection, click-outside-to-close
- Used for Stage Box input type selector (XLR, DI/XLR, Line, 1/4") and Dante protocol selector (Dante, Analog, AES50, USB)

**Bug fixes:**
- cs-1b task detail corrected: "connect ATEM Ethernet to AX55 router LAN port for remote control. Resi gets internet directly from venue ethernet (separate network)" — was incorrectly referencing Resi Encoder LAN 1
- Task detail text overflow fixed: line clamp increased from 2 to 5 lines for full visibility of setup instructions; `wordBreak: "break-word"` and `overflowWrap: "break-word"` added to prevent horizontal overflow on mobile

**Version:** bumped to v5.11.0

---

## v5.10.0 (2026-03-31) — I/O Line List Persistence

**I/O config now persists across refreshes and syncs to all devices:**
- `useStorage("elim3-io-config", DEFAULT_IO_LINE_LIST)` owned by `ElimProductionApp`, passed as props to `IOListView`
- Removed the cleanup `useEffect` line that was actively deleting `elim3-io-config` from Firebase/localStorage on every load
- Added `c9` to the `loaded` gate so the app waits for I/O config before rendering

**Draft-based editing with confirm-on-save:**
- Tapping "Edit I/O" clones live config into a local draft state
- All edits modify the draft only — live state (and other devices) are unaffected
- "Done Editing" opens a confirm modal: "Save I/O changes? This syncs to all devices."
- "Save" writes draft to `useStorage` → Firebase; "Discard" drops the draft
- "Cancel" button available during editing for quick exit without modal

**Removed:**
- Reset-to-defaults button and its confirm flow removed entirely from IOListView
- `confirmingReset` state removed

**Version:** bumped to v5.10.0

---

## v5.9.0 (2026-03-30) — I/O Data Update, ATEM Correction, Timer UX, Checklist Tasks

**I/O Line List data replacement:**
- Stage Box: 16 generic channels → 44 real channels (drums 1-12, instruments 13-25, talk backs 26-27, vocals 28-36, wireless 37-40, crowd 41-44)
- Wing Direct Inputs: new section with 2 entries (Wing 8 = Bianca TB, Dante 47-48 = ProPresenter PC)
- Outputs: 5 generic → 18 real entries with `inEarSnake` field (Sunny P1 through Wing 7)
- Dante Routing: corrected to reference "Bianca's ProPresenter Mac" and "ATEM Television Studio Pro 4K"
- IOListView tabs updated: "Stage Box", "Wing", "Outputs", "Dante"
- IOListView render code updated to handle new `wingInputs` section and `inEarSnake` display

**ATEM name correction:**
- All "ATEM Mini Pro" references replaced throughout codebase
- Checklist tasks cs-1b and cs-1 say "ATEM Television Studio Pro 4K"
- Three SVG Box labels say "ATEM TV Studio 4K" (space-constrained)
- Zero occurrences of "ATEM Mini Pro" remain

**Post-event timer exclusion:**
- Post-Event section (pe-1/pe-2/pe-3) excluded from dashboard progress calculations and timer
- Main app `totalItems` and `allItems` filter out `sec.id !== "post-event"`
- `newWeek` function also excludes post-event from history records
- ChecklistView's own `totalItems` counts ALL tasks including post-event (intentional — full checklist shows true completion)

**Glenn → functional naming:**
- 6 references renamed from "Glenn's ProPresenter Mac" to "ProPresenter Mac" (tasks dn-6, cs-2, ds-2, pp-1, Power Sequence #7, Network SVG)
- ss-3 troubleshooting reference "Glenn has walkthrough for setup" preserved (refers to the person, not equipment)

**P2 checklist task additions:**
- mc-3: "Check batteries in drum click" added to Mic Check section
- cs-3: "Setup stage spotlight" added to Camera Setup section with detail
- sb-4: Speaker spacing reference added as detail text (25ft/13ft/11.5ft)
- sb-1: Wheel stability troubleshooting entry added ("Lock the caster wheels")
- Camera Setup expanded from 1 generic camera task to 4 specific tasks: cs-1 (BlackMagic wide-angle, ATEM IN 1), cs-4 (handheld via Hollyland, ATEM IN 2), cs-5 (DJI, ATEM IN 3), cs-6 (BlackMagic 6K, ATEM IN 4)

**Timer auto-stop at 100%:**
- `autoStoppedRef` tracks whether auto-stop has fired this session
- `useEffect` watches `checks` against `allItems`/`totalItems` — when all non-post-event tasks are checked, timer auto-pauses
- One-way gate: unchecking a task after auto-stop does NOT restart timer
- Ref reset when timer is manually started or checklist is reset

**Manual "Setup Complete" in Settings:**
- `setupComplete` callback logs current progress to history without clearing checkboxes or assignments
- Timer freezes at current elapsed value (visible, not zeroed)
- Two-tap confirm UI in Settings below Reset Checklist
- Sets `autoStoppedRef.current = true` to prevent redundant auto-stop

**Scroll position retry fix:**
- Replaced single `requestAnimationFrame` scroll restore with retry loop (10 attempts, 5px tolerance)
- Handles async-loaded views (I/O, Repairs) where content renders after initial scroll attempt
- Early return for `saved === 0` (no retry needed for top-of-page)

**My Tasks collapsible sections:**
- `manualExpand` state tracks user toggle overrides
- Completed sections (all tasks checked) auto-collapse to single green header line with ✓
- Tap any section header to toggle expand/collapse
- Collapsed sections get tighter margin (4px vs 14px)
- Header turns green with 0.7 opacity when all done
- ▶/▼ expand indicators on right side of headers

**Version:** bumped to v5.9.0

---

## v5.8.0 (2026-03-28) — P0 Bug Fixes + Note Surfacing

Five Priority 0 bugs resolved plus task card information density improvements.

**Bug fixes:**
- **Hidden scrollbar** — CSS rules added (`scrollbar-width: none` + `::-webkit-scrollbar { display: none }`) on wildcard selector; covers main page and inner scrollable containers
- **Dante I/O edit overflow** — added `minWidth: 0` to from/to input fields and `flexShrink: 0` to delete button in Dante routing edit row; inputs now shrink properly on narrow screens
- **Custom assignment dropdown** — replaced native `<select>` with `AssignDropdown` component; dark-themed dropdown list (`#1C1A18` background), 15px text, click-outside-to-close, "Unassign" option when assigned, checkmark on current selection; matches Warm Night theme
- **Scroll position preservation** — `scrollPositions` ref stores `window.scrollY` per view on navigate/goBack; restored via `requestAnimationFrame` on view change; session-only (not persisted across refreshes)

**Task card improvements:**
- **Detail text surfaced above the fold** — `item.detail` now renders below assignee row (outside expand), capped at 5 lines with `-webkit-line-clamp`; hidden on checked tasks to save vertical space
- **Note preview above the fold** — populated `taskNote` shows below detail with 📌 indicator, capped at 2 lines; hidden when task is expanded (full editing UI visible) or checked
- **Detail removed from expand area** — no longer duplicated inside the expanded section

**Navigation:**
- **View stack** — `viewStack` array replaces simple `view` state; `navigate()` pushes or returns to existing position; `goBack()` pops; enables proper breadcrumb-style back navigation

**Component changes:**
- New `AssignDropdown` component (defined near `CheckItem`)
- `CheckItem` render order restructured: checkbox → task name → assignee → detail → note preview → [expand: photos, troubleshooting, note editing]

**Version:** bumped to v5.8.0

---

## v5.7.0 (2026-03-28) — Audit Fixes + Performance

Zero-Principles audit identified 26 issues across data integrity, volunteer safety, and performance. This release addresses the top 15.

**Data integrity fixes:**
- **Firebase write retry** — failed writes now tracked in `_pendingWrites` Set; incoming `elim-sync` events ignored for pending keys; retry with exponential backoff; prevents stale Firebase data from silently overwriting local changes on spotty venue WiFi
- **Removed localStorage→Firebase re-seed** — `storage.get` fallback path no longer pushes stale localStorage data to Firebase on reconnect; prevents clobbering newer data from other devices
- **`JSON.parse(null)` crash guard** — both parse sites in `useStorage` (initial `get` and `elim-sync` handler) now discard null values instead of passing them to `setVal`; prevents white-screen crash if a Firebase node is deleted
- **Orphaned data cleanup on task deletion** — `removeTask` now deletes corresponding entries from `checks`, `assignments`, `taskNotes`, and `completions`
- **Orphaned data cleanup on roster removal** — removing a team member clears all their task assignments before removing them from roster; prevents blank/ghost dropdown entries
- **Accurate task counter** — `totalChecked` now computed by intersecting with live task IDs (`allItems.filter(i => checks[i.id]).length`) instead of counting all keys in `checks` object; prevents 103% progress after task deletion
- **CHECKLIST_VERSION stamp** — version string on `CHECKLIST_DATA` constant. On app load, if stored `elim3-checklist-data` has a different version, the stored value is deleted (forcing reload from fresh constants). This prevents structural drift between code-defined tasks and user-modified data — e.g. if a task ID changes in code but old data still has the previous ID, the version bump forces a clean reset.

**UX safety improvements:**
- **Timer auto-start threshold** — timer only auto-starts when `totalChecked >= 2` (was: on first checkbox); prevents accidental single tap from starting the timer and corrupting setup time history
- **Dependency warning modal readable** — modal card background changed from `S.card` (nearly transparent) to `#1C1A18` (opaque dark)

**Error handling:**
- **React Error Boundary** — new `ErrorBoundary` class component wraps the full app at mount point (`root.render`); any render-time exception shows "Something went wrong — Tap to Reload" screen instead of permanent white screen

**Performance improvements:**
- **Memoized `allItems`** — wrapped in `useMemo` keyed on `checklistData`; no longer recomputed on every 1-second timer tick
- **`React.memo` on `CheckItem`** — prevents 59 task cards from re-rendering every second due to timer updates in parent
- **Photos removed from load gate** — `c5` (photos loaded flag) removed from `const loaded = ...` gate; app renders immediately, photos load lazily when available
- **Hoisted static constants** — `cards` array moved from inside `DashboardView` to module scope; `CARD_BASE_STYLE` hoisted from inside `CheckItem` render to module scope

**Bucket B — UX improvements (shipped same day):**
- **Default My Tasks view** — app opens to My Tasks instead of Dashboard; Dashboard accessible via back button
- **Search toggle** — checklist search bar hidden behind 🔍 icon; expands on tap, collapses when empty + blurred
- **Placeholder cards removed from dashboard** — Equipment and Purchases cards removed; dashboard nav grid reduced to essential items
- **"Last setup" stat removed from dashboard hero** — still accessible in Settings → Setup History

**Storage keys added:** `elim3-pending-reset` (transient, deleted after reset completes)

**Version:** bumped to v5.7.0

---

## v5.6.0 (2026-02-28) — Repairs CRUD + Checklist Edit Mode + Search + Audio SVG Redesign

**Repairs View — fully editable, Firebase-persisted:**
- `useStorage("elim3-repairs", DEFAULT_REPAIRS)` replaces hardcoded `REPAIRS_LIST`
- `DEFAULT_REPAIRS` constant holds original 6 items as reset target
- Search field always visible above list — filters by item name or issue text (case-insensitive)
- Edit mode toggle ("✏️ Edit Repairs" / "✓ Done Editing") matches I/O list pattern exactly
- Inline editing in edit mode: item name field, issue description field, priority dropdown per item
- Priority dropdown styled with per-priority color (red/orange/green) matching badge colors
- ✕ delete button per item in edit mode (immediate, no confirmation)
- "+ Add Repair" button at bottom in edit mode — hidden when search is active
- Reset to defaults: two-tap confirm flow ("Reset" → "Confirm Reset" + "✕ cancel")
- Item count shown below controls (e.g. "3 items of 6")
- Cross-device sync via Firebase real-time listeners

**Checklist View — search toolbar + edit mode + task CRUD:**
- Search toolbar added between progress bar and section list
- Real-time filtering: sections with no matches hidden; sections with matches auto-expand
- Edit mode (✏️ toggle button right of search field): all sections auto-expand for editing
- Search + edit combo: matched sections expand and show edit controls
- Edit mode: ✕ delete button per task (left of card), immediate removal
- Edit mode: tap task text to inline-edit — Enter key or blur saves, Escape cancels
- Edit mode: "+ Add Task" button at bottom of each expanded section (hidden during search)
- New tasks assigned timestamp-based IDs (e.g. `task-1709123456789`) to prevent collision
- Edit mode toggle clears any in-progress inline edit or add state on exit

**Checklist data — now Firebase-persisted:**
- Checklist structure stored in `elim3-checklist-data` (defaults to `CHECKLIST_DATA` constant)
- All original task IDs preserved — existing checks, assignments, notes, photos unaffected
- `ElimProductionApp` owns `checklistData` state via `useStorage`; `allItems` derived live from it
- `DashboardView` receives `allItems` prop (crew status, your tasks count now use live data)
- `ChecklistView` and `MyTasksView` receive `checklistData` prop; fall back to static `CHECKLIST_DATA` if prop absent
- `newWeek` history record uses live `checklistData` for accurate task totals
- Module-level `ALL_ITEMS` constant retained as fallback; `CheckItem` dependency lookup still works for all built-in tasks

**Audio Signal Chain SVG redesigned:**
- Tyler's Mac moved to left sidebar branching directly off Wing — mirrors WiFi Router sidebar on right
- Eliminates long diagonal/vertical line that previously cut through Omar/Mitch Mac and IEM boxes
- Bianca IEM now directly below IEM Transmitters in a clean vertical flow (no crossing lines)
- ViewBox reduced from 780px to 660px height — less whitespace, tighter layout
- Both main branches (left analog, right Dante) now terminate at approximately the same vertical level

**Bug fixes (pre-release, included in this version):**
- **Swipe-back exclusion zone** — touches starting in top-left area (top 80px, left 120px) no longer trigger swipe gesture; prevents conflict with back button taps
- **Progress bar clipping** — header spacer height increased to 48px, checklist top padding increased to 14px; progress bar no longer clips behind fixed header on scroll
- **Task field name bug** — corrected `task` → `text` field reference in task items throughout; was causing blank task display in some edit states

**Storage keys added:** `elim3-repairs`, `elim3-checklist-data`

**Version:** bumped to v5.6.0

---

## v5.5.2 (2026-02-28) — Equipment Accuracy & Checklist Refinements

**Three Macs clarified (names restored):**
- Tyler's Mac: connects directly to Wing via USB-C for live recording (NOT on Dante network)
- Omar/Mitch's Mac: connects via CAT6 to 5-Port Switch (Dante) for stream mixing
- Glenn's Mac: connects via CAT6 to 5-Port Switch (Dante) for ProPresenter / video (later renamed to "ProPresenter Mac" in v5.9.0)
- New task ws-5b: "Connect Tyler's ProTools Mac to Wing (USB-C direct)" added to Wing Setup section
- pt-1: updated to "Power on both ProTools Macs (Tyler's + Omar/Mitch's)"
- pp-1: updated to "Power on Glenn's ProPresenter Mac"
- ds-2, cs-2, dn-5, dn-6: updated to reference named Macs

**Behringer P1 / Pastor Monitor fully removed:**
- I/O outputs: Aux 3 "Pastor Monitor" row removed; replaced with "Wing XLR Out 8 → Bianca's IEM"
- Dante routing: Dante AVIO now routes to ATEM (Analog Audio IN Ch 1-2), not Pastor Monitor
- Audio SVG and Network SVG updated accordingly

**Camera Setup reordered:** cs-1b (ATEM setup) → cs-2 (ATEM USB) → cs-1 (Camera + SDI)

**WiFi Router dependency added:**
- ws-4: gains `dependsOn: ["ws-2", "ws-3"]` — amber warning modal if Wing or router not yet setup

**Section numbering:** Checklist and My Tasks headers show "1. Pre-Setup", "2. LED Wall Setup", etc.

**Version:** bumped to v5.5.2

---

## v5.5.1 (2026-02-28) — Recovery + UX Fixes

Recovered lost changes from v5.3.2 and v5.4.0 that were dropped during a rebuild attempt (v4). Added Repairs view, back button improvements, header cleanup.

**Version:** bumped to v5.5.1

---

## v5.5.0 (2026-02-28) — Signal Flow Diagrams + Header & Navigation Fixes

Three static SVG diagrams (Audio, Video, Network/Dante) with color-coded sections. Fixed header positioning with safe-area-inset. Smooth follow-finger swipe-back gesture.

**Version:** bumped to v5.5.0

---

## v5.4.0 and earlier

See previous changelog entries for I/O auto-sort, duplicate detection, task reorders, equipment depersonalization, dependency warnings, countdown timer, weekly reset, SVG brand mark, dynamic roster, Firebase deployment, brand alignment, and the original v3→v5 surgical upgrade path.
