# ELIM Production App — Changelog

## Documentation Update (2026-03-29) — Field Data + Hardware + Doc Consolidation

No code changes. Major documentation update incorporating on-site field data from March 2026 setup, hardware build decisions, and consolidation of multiple planning documents.

**ELIM-PROJECT.md — comprehensive rewrite:**
- Added "Purpose" section (why the app exists — verification at scale, knowledge transfer, venue consistency)
- Equipment Reference fully corrected: ATEM Television Studio Pro 4K (was "ATEM Mini Pro"), all SDI port assignments documented (IN 1-4, IN 8, OUT 8), Bianca's Mac connections (Anker hub, UltraStudio, dual-network), Hollyland receiver, 4 cameras (was 1), Wing 8 = Bianca TB
- Added "Network Architecture" section documenting three separate networks (Dante, Production Control, Internet) with device assignments and troubleshooting guidance
- Added full ATEM network configuration (192.168.0.240 static, Mac at .100, AX55 router) with revert procedure
- Added "Hardware Organization" section: Wing case contents (including confirmed Resi placement), Pelican 1660 contents, cable bundle definitions, table cable management
- Added cable color code system (7 colors with from/to assignments)
- Added wall spacing reference measurements (25ft wall-to-chairs, 13ft carpet-to-chairs, 11.5ft speakers-to-wall)
- Added "Hardware Decisions Log" with rationale and rejected ideas
- Updated deployment section: Replit as primary dev environment, `firebase deploy --only hosting`
- Updated dashboard layout: 2×2 nav grid (was 3×2, changed in v5.7.0)
- Updated file structure to reflect Replit workspace
- Added clarification about Firebase/code sync workflow (in-app edits lost on deploy+reset if not mirrored to code)

**OPEN-ITEMS.md — major update:**
- 7 items resolved with dates and resolutions: Wing XLR Input 8, ATEM SDI ports (partial), Resi SDI input, ATEM-to-Resi ethernet, Resi placement, Router WiFi signal, Speaker/wall spacing
- Remaining ATEM SDI unknowns (IN 5-7, OUT 1-7) consolidated into single item
- Added new open items: PA speaker power draw, LED wall caster mounting, confidence monitor dimensions, Pelican full contents inventory, 3D stacking cup fit, cable run distances, Bianca's Mac multi-network verification

**BACKLOG.md — restructured and expanded:**
- Added P0 items: ATEM name correction, I/O line list data update (46 inputs/19 outputs from field screenshots), post-event timer exclusion, timer stop UX
- Added P1 item: section-level team assignments (from field notes)
- Added P2 items: drum click batteries, spotlight, camera setup updates for 4 cameras
- Added P4 items from ABC app pattern analysis: data validation on load, schema versioning, error boundary with data export
- Added P5 item: decibel meter (Web Audio API, Sam+Tyler only)
- Added "Evaluated and Rejected" section: Shazam (use the app), song key detection (unreliable for polyphonic audio)
- Added P6 items: stage power distribution, church-wide production standards (laminated references)

---

## v5.7.0 (2026-03-28) — Audit Fixes + Performance

Zero-Principles audit identified 26 issues across data integrity, volunteer safety, and performance. This release addresses the top 15.

**Data integrity fixes:**
- **Firebase write retry** — failed writes now tracked in `_pendingWrites` Set; incoming `elim-sync` events ignored for pending keys; single 5-second retry attempted; prevents stale Firebase data from silently overwriting local changes on spotty venue WiFi
- **Removed localStorage→Firebase re-seed** — `storage.get` fallback path no longer pushes stale localStorage data to Firebase on reconnect; prevents clobbering newer data from other devices
- **`JSON.parse(null)` crash guard** — both parse sites in `useStorage` (initial `get` and `elim-sync` handler) now discard null values instead of passing them to `setVal`; prevents white-screen crash if a Firebase node is deleted
- **Orphaned data cleanup on task deletion** — `removeTask` now deletes corresponding entries from `checks`, `assignments`, `taskNotes`, and `completions`
- **Orphaned data cleanup on roster removal** — removing a team member clears all their task assignments before removing them from roster; prevents blank/ghost dropdown entries
- **Accurate task counter** — `totalChecked` now computed by intersecting with live task IDs (`allItems.filter(i => checks[i.id]).length`) instead of counting all keys in `checks` object; prevents 103% progress after task deletion
- **Atomic `newWeek` reset** — writes `elim3-pending-reset` flag to Firebase before clearing; if interrupted (phone sleeps, network drops), next app load completes the reset automatically via `useEffect` on `loaded`

**Volunteer safety fixes:**
- **Task deletion confirmation** — `window.confirm()` required before `removeTask`; prevents accidental one-tap task deletion that syncs to all devices
- **Edit mode restricted to Sam** — ✏️ button in `ChecklistView` only renders when `currentUser === "Sam"`; prevents other volunteers from accidentally entering edit mode and seeing delete buttons
- **Roster removal confirmation** — updated confirm dialog to warn that assignments will be cleared
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
- **Placeholder cards removed from dashboard** — Equipment and Purchases cards moved to Settings → "Coming Soon" section; dashboard nav grid reduced from 3×2 to 2×2
- **Last-setup stat removed from dashboard hero** — still accessible in Settings → Setup History

**Storage keys added:** `elim3-pending-reset` (transient, deleted after reset completes)

**Version:** bumped to v5.7.0

---

## v5.8.0 (2026-03-28) — P0 Bug Fixes + Note Surfacing

Five Priority 0 bugs resolved plus task card information density improvements.

**Bug fixes:**
- **Hidden scrollbar** — CSS rules added (`scrollbar-width: none` + `::-webkit-scrollbar { display: none }`) on wildcard selector; covers main page and inner scrollable containers
- **Dante I/O edit overflow** — added `minWidth: 0` to from/to input fields and `flexShrink: 0` to delete button in Dante routing edit row; inputs now shrink properly on narrow screens
- **Custom assignment dropdown** — replaced native `<select>` with `AssignDropdown` component; dark-themed dropdown list (`#1C1A18` background), 15px text, click-outside-to-close, "Unassign" option when assigned, checkmark on current selection; matches Warm Night theme
- **Scroll position preservation** — `scrollPositions` ref stores `window.scrollY` per view on navigate/goBack; restored via `requestAnimationFrame` on view change; session-only (not persisted across refreshes)

**Task card improvements:**
- **Detail text surfaced above the fold** — `item.detail` now renders below assignee row (outside expand), capped at 2 lines with `-webkit-line-clamp`; hidden on checked tasks to save vertical space
- **Note preview above the fold** — populated `taskNote` shows below detail with 📌 indicator, capped at 2 lines; hidden when task is expanded (full editing UI visible) or checked
- **Detail removed from expand area** — no longer duplicated inside the expanded section

**Navigation:**
- **View stack** — `viewStack` array replaces simple `view` state; `navigate()` pushes or returns to existing position; `goBack()` pops; enables proper breadcrumb-style back navigation

**Component changes:**
- New `AssignDropdown` component (defined near `CheckItem`)
- `CheckItem` render order restructured: checkbox → task name → assignee → detail → note preview → [expand: photos, troubleshooting, note editing]

**Version:** bumped to v5.8.0

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

**I/O Line List — fully editable, Firebase-persisted:**
- `useStorage("elim3-io-config", DEFAULT_IO_LINE_LIST)` replaces hardcoded data
- `DEFAULT_IO_LINE_LIST` constant holds original values as reset target
- Edit mode toggle ("✏️ Edit" / "✓ Done Editing") in header
- Inline editing in edit mode: channel label, type, and notes all editable
- ✕ delete button per channel in edit mode (immediate, no confirmation)
- "+ Add Channel" button at bottom of each section in edit mode
- Reset to defaults: two-tap confirm flow ("Reset" → "Confirm Reset" + "✕ cancel")
- Dante routing section: editable from/to text fields + protocol dropdown
- All sections now show row counts (e.g. "16 channels")
- Cross-device sync via Firebase real-time listeners

**Version:** bumped to v5.6.0

---

## v5.5.0 (2026-02-27) — Power Sequence + Countdown Timer + I/O Line List + Signal Flow

**Power Sequence:**
- 10-step reference with specific equipment names — power on 1→10, off 10→1
- Teardown reminder card displayed after all steps
- Badge on task cards: amber shield with step number for tasks linked to power sequence
- Data: `POWER_SEQUENCE` array with `step`, `name`, `taskId` per entry

**Countdown Timer:**
- Configurable via presets (30/45/60/90 min) or custom input
- Visual: green→yellow→red color transition based on remaining percentage
- Auto-pause at zero (does not go negative)
- Synced across devices via Firebase (`elim3-timer` key)
- State: `{startedAt, accumulated, targetMinutes}` — handles pause/resume by accumulating elapsed time
- Timer visible in both Dashboard hero and header bar

**I/O Line List (static):**
- Stage Box Inputs: 16 channels (ch 1-16, XLR/DI/Line types)
- Main Outputs: 5 outputs (FOH, Sub, Aux 1-2, XLR Out 8)
- Dante Routing: 5 routes with from/to/channels/protocol
- Expandable sections with channel count badges

**Signal Flow SVGs:**
- Three static diagrams: Audio Signal Chain, Video Signal Chain, Network/Dante
- Helper sub-components: `Box`, `Arrow`, `Label`, `HLine`, `VLine`
- Audio: viewBox 570×660, color-coded orange (analog) / purple (digital) / green (Dante)
- Video: viewBox 480×510
- Network: viewBox 480×360
- Responsive SVG scaling, always sharp on any device

**Other changes:**
- Unassigned tasks: dashed border, lighter background, italic dimmed text throughout
- Section headers show unassigned count
- Dashboard "Unassigned Tasks" row at bottom of crew status
- `cardStyle()` function resolves CSS property conflicts (border vs borderLeft)
- Timer in header visible on Dashboard, Checklist, My Tasks, Crew Tasks views

**Version:** bumped to v5.5.0

---

## v5.4.0 (2026-02-27) — Swipe Back + Crew Focused Views

**Swipe-back gesture:**
- `useBackSwipe` custom hook
- Follow-finger: content `translateX` tracks touch position in real-time via refs (not state)
- 35% screen width threshold to trigger navigation
- Header zone exclusion: top 80px, left 120px (prevents conflict with browser gestures)
- `data-no-swipe` attribute excludes photo scroll areas
- Performance: direct DOM manipulation via refs, no React re-renders during gesture

**Crew focused views:**
- Tap crew member name on Dashboard → see just their tasks (filtered `MyTasksView`)
- `MyTasksView` accepts `targetUser` prop for filtering
- `__unassigned__` special value filters to tasks with no assignee
- Back navigation returns to Dashboard with correct scroll position

**Version:** bumped to v5.4.0

---

## v5.3.0 (2026-02-27) — Setup History + Timer Sync + Logo

**SVG Brand Mark:**
- `ElimLogo` component: two right-pointing chevrons + block E
- Sharp corners, geometric style, scaled 82% inside circle
- Used on login screen and loading screen

**Setup History:**
- Last 52 weeks stored in `elim3-history`
- Record: `{date, pct, elapsed, tasks, total}`
- Displayed in Settings with date, progress bar, percentage, time

**Timer Improvements:**
- Timer now syncs across devices via Firebase
- Timer auto-starts on second checkbox (threshold = 2, prevents accidental starts)

**Version:** bumped to v5.3.0

---

## v5.0.4 (2026-02-26) — Hardening Pass

**Hardened:**
- Firebase init wrapped in try/catch — app renders even if Firebase CDN blocked
- `_ls` safe localStorage wrapper — all localStorage calls wrapped in try/catch for environments that block storage APIs
- App degrades gracefully: Firebase unavailable → localStorage only → in-memory only

**Changed:**
- Section headers: cleaner layout with count/✓ right-aligned, progress bar inside expanded area
- My Tasks: section count → green ✓ when all tasks in section complete
- "YOU" badge only shows when you have incomplete tasks (not after finishing)
- CheckItem: removed completedBy prop, simplified to 3-state visual

**Fixed:**
- React hooks ordering bug — `useCallback` for `navigate` was after early returns, causing crash when clicking a username on login screen (React error #310)

## v5.0.3 (2026-02-26) — Dynamic Roster + Remember Me
**Added:**
- Dynamic team roster management in Settings view (add/remove/reorder members)
- Remember-me: device remembers last selected user across sessions
- Settings nav card on dashboard (⚙️)
- `elim3-roster` storage key for persistent roster
- DEFAULT_ROSTER constant with original 6 members + colors

**Changed:**
- Team members derived from roster storage instead of hardcoded TEAM_MEMBERS array
- Login screen uses 2-column grid when roster has 7+ members
- All components receive `teamMembers` prop from roster

## v5.0.2 (2026-02-26) — Firebase Deployment
**Added:**
- Firebase Realtime Database for cross-device sync (replaces localStorage-only storage)
- Real-time listeners: changes on one phone appear on all others within ~1 second
- `elim-sync` custom event system bridges Firebase → React state updates
- localStorage cache layer for fast initial render + offline fallback
- Firebase Hosting deployment at https://elim-production.web.app
- PWA manifest for Add to Home Screen on all devices

**Storage architecture:**
- Write path: update React state → write localStorage cache → write Firebase → Firebase triggers real-time listener on other devices
- Read path: try Firebase first → fall back to localStorage cache → throw if neither exists
- Offline: all writes save to localStorage immediately, push to Firebase when connectivity returns

**Infrastructure:**
- Firebase project: `elim-production`
- Database URL: `https://elim-production-default-rtdb.firebaseio.com`
- Hosting URL: `https://elim-production.web.app`
- Database rules: open read/write (sufficient for small trusted team)
- Deploy command: `firebase deploy` from Replit terminal

**Preserved:**
- All app code, features, data, and brand styling identical to v5.0.1
- Storage keys unchanged: `elim3-checks`, `elim3-assign`, `elim3-comp`, `elim3-notes`, `elim3-photos`
- `window.storage` API interface unchanged (app code doesn't know about Firebase)

## v5.0.1 (2026-02-26) — Brand Alignment
**Changed:**
- Rebranded color scheme from indigo/purple to warm orange to match The Well / ELIM Arrival brand
- Primary accent: #818CF8 → #E8893A (warm orange)
- Secondary accent: #C084FC → #F0B060 (golden orange)
- Gradient: indigo→purple → orange→gold
- All glows, borders, hover states, active indicators updated to orange family
- Replaced ⚡ bolt emoji with styled »E brand mark (circle + logotype) on login and loading screens
- Glenn's team color (#818CF8) preserved — now distinct from app accent

**Preserved:**
- All functionality, data, sections, tasks identical to v5
- All other team colors, section colors, and functional colors unchanged

## v5 (2026-02-26) — Smart Checklist + Team Awareness
**Added** (surgical additions to v3, no data changes):
- Smart auto-expand: sections with your assigned tasks or in-progress auto-expand on load
- Manual collapse override: tap to expand/collapse any section, sticks for session
- "YOU" badge on section headers where you have assigned tasks
- Colored left border on tasks assigned to you (uses your team color)
- Team progress strip: scrollable crew status row at top of checklist
- Crew status grid on dashboard: per-person progress bars in 3×2 layout
- Global setup timer: on dashboard (large, tappable) and checklist header
- Timer auto-starts on first checkbox
- TEAM_COLORS constant for per-person color coding
- ALL_ITEMS helper for cross-section progress calculations

**Preserved** (unchanged from v3):
- All 15 sections, 58 tasks — every word of task text identical
- All photo slots with labels
- All troubleshooting Q&As
- Per-task notes
- Section timers
- Team filter
- Power sequence (10 steps with specific equipment names)
- I/O Line List (16 inputs, 5 outputs, 3 Dante routes)
- My Tasks view
- All 8 dashboard nav cards including Repairs and Purchases placeholders

## v4 (2026-02-26) — Rebuild Attempt (superseded)
- Attempted clean rebuild with new features
- Lost data fidelity: task text shortened, troubleshooting answers abbreviated, photo slots renamed
- Dropped Repairs and Purchases nav cards
- **Superseded by v5** which took the correct approach of editing v3 surgically

## v3 (2026-02-26) — Mission Control
- Mission Control dashboard with 8 nav cards
- Full checklist with all 15 sections
- Photo upload with auto-resize (800×600, JPEG 70%)
- Troubleshooting expandable Q&A per task
- Per-task persistent notes
- Per-task team member assignment
- Section timers (per-section stopwatch)
- Team filter (filter checklist by person)
- My Tasks view (personal task queue grouped by section)
- Power Sequence reference (10-step order)
- I/O Line List (inputs, outputs, Dante routing)
- Placeholder views for Signal Flow, Equipment, Repairs, Purchases
- User selection screen (no auth)
- Shared persistent storage

## v2 (2026-02-26) — Feature Build
- Added photo documentation system
- Added troubleshooting guides
- Updated power sequence (removed P1 Monitor and LED Scaler)
- Refined checklist content from original spreadsheet

## v1 (2026-02-26) — Initial Build
- Basic checklist from ELIM_Production_Setup_Checklist.xlsx
- 15 sections with checkbox tracking
- Power sequence reference
- Dark theme UI
