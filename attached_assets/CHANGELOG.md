# ELIM Production App — Changelog

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

**Bug fixes (pre-release, included in this version):**
- **Swipe-back exclusion zone** — touches starting in top-left area (top 80px, left 120px) no longer trigger swipe gesture; prevents conflict with back button taps
- **Progress bar clipping** — header spacer height increased to 48px, checklist top padding increased to 14px; progress bar no longer clips behind fixed header on scroll
- **Task field name bug** — corrected `task` → `text` field reference in task items throughout; was causing blank task display in some edit states

**Storage keys added:** `elim3-repairs`, `elim3-checklist-data`

**Version:** bumped to v5.6.0 in Settings About section

---

## v5.5.2 (2026-02-28) — Equipment Accuracy & Checklist Refinements

**Three Macs clarified (names restored):**
- Tyler's Mac: connects directly to Wing via USB-C for live recording (NOT on Dante network)
- Omar/Mitch's Mac: connects via CAT6 to 5-Port Switch (Dante) for stream mixing
- Glenn's Mac: connects via CAT6 to 5-Port Switch (Dante) for ProPresenter / video
- New task ws-5b: "Connect Tyler's ProTools Mac to Wing (USB-C direct)" added to Wing Setup section
- pt-1: updated to "Power on both ProTools Macs (Tyler's + Omar/Mitch's)"
- pp-1: updated to "Power on Glenn's ProPresenter Mac"
- ds-2, cs-2, dn-5, dn-6: updated to reference named Macs

**Behringer P1 / Pastor Monitor fully removed:**
- I/O outputs: Aux 3 "Pastor Monitor" row removed; replaced with "Wing XLR Out 8 → Bianca's IEM"
- Dante routing: Dante AVIO now routes to ATEM Mini Pro (Analog Audio IN Ch 1-2), not Pastor Monitor
- Audio SVG: ATEM Mini Pro replaces Pastor Monitor; Tyler's Mac branch added via USB-C direct; Bianca IEM added
- Network SVG: Pastor Monitor removed; ATEM Mini Pro added; Tyler's Mac noted as USB-C direct (not Dante)

**Camera Setup reordered:** cs-1b (ATEM setup) → cs-2 (ATEM USB) → cs-1 (Camera + SDI)
- cs-1b detail: set up Resi Encoder alongside ATEM (Control Ethernet → Resi LAN 1)

**WiFi Router dependency added:**
- ws-3: renamed to "Connect and setup WiFi Router with Wing" (power-on removed from this task)
- ws-4: gains `dependsOn: ["ws-2", "ws-3"]` — amber warning modal if Wing or router not yet setup

**Connection detail notes added to:** dn-2 (port 5 on switch + AES50 port A), dn-3 (Dante AVIO note), pt-2 (reboot note)

**Section numbering:** Checklist and My Tasks headers show "1. Pre-Setup", "2. LED Wall Setup", etc.

**Power Sequence:** Item 5 → "ProTools Macs (Tyler + Omar/Mitch)"; Item 7 → "Glenn's ProPresenter Mac"

**Signal Flow SVG updates:**
- Audio viewBox extended (680 → 730) to fit new elements
- Audio: "Omar/Mitch Mac (Stream Mix)" replaces generic label; Tyler's Mac added via USB-C branch; Bianca IEM added; ATEM replaces Pastor Monitor as AVIO destination
- Network: "Glenn's Mac (ProPresenter)" and "Omar/Mitch Mac (Stream Mix)" replace generic labels; ATEM replaces Pastor Monitor; Tyler's Mac noted as not on Dante

---

## v5.5.1 (2026-02-28) — Recovery + UX Fixes

**Recovered from v5.3.2 (lost during rebuild):**
- I/O auto-sort by channel number on blur (Inputs tab)
- Duplicate channel detection with red highlight styling (Inputs + Outputs)
- `sortInputsByChannel()`, `isDuplicateChannel()`, `handleChannelBlur()` helpers

**Recovered from v5.4.0 (lost during rebuild):**
- Pre-Setup reordered: ps-3 (Unload) → ps-1 (Pray) → ps-2 (Assign)
- Stage Box reordered: sb-1, sb-4, sb-6, sb-5, sb-2, sb-3
- Task text updates: ws-3, ds-3, pp-5, psc-1, dn-2
- Equipment depersonalized (Glenn's/Mitch's/Omar's references removed)
- POWER_SEQUENCE fixes (items 5, 6, 7, 9)
- cs-1b "Setup ATEM Mini Pro" added to Camera Setup section
- Dependency warning system: dn-3 depends on dn-1, pt-3 depends on pt-2
- Amber warning modal in CheckItem with Continue/Cancel

**New in v5.5.1:**
- Behringer P1 removed from I/O data and signal flow SVGs
- LED Scaler removed from ds-1 task text and Video signal flow SVG
- Network/Dante SVG viewBox height reduced (480 → 360)
- Video SVG viewBox reduced (580 → 510) with streaming callout repositioned
- Login page scroll fixed (100dvh + overflow hidden on login container)
- Header padding reduced (12px → 8px top, spacer 50 → 44px)
- RepairsView component added with 6 prioritized maintenance items (hardcoded; upgraded to CRUD in v5.6.0)
- Back button enlarged — arrow + title text act as one tap target
- Version bumped to v5.5.1

---

## v5.5.0 (2026-02-28) — Signal Flow Diagrams + Header & Navigation Fixes

**Signal Flow diagram view:**
- Three static SVG diagrams: Audio, Video, Network/Dante
- Audio chain: Stage Inputs → Midas DL251 → Behringer Wing → fork to analog (FOH/Subs/IEMs) + Dante (5-Port Switch → ProTools/Dante AVIO → Behringer P1)
- Video chain: Streaming Camera → ATEM Mini Pro → ProPresenter Mac → fork to LED Wall path (Scaler → Transmitter → Wall) + Confidence Monitor
- Network chain: Wing Dante Expansion → 5-Port Switch → three endpoints (ProTools, Dante AVIO → P1, ProPresenter)
- WiFi Router sidebar showing iPad wireless control
- Live Streaming callout: ProPresenter → Internet → Resi
- SVG helper components (Box, Arrow, Label, HLine, VLine) for consistent rendering
- Color-coded sections: orange (audio), purple (video), green (network)
- Compact viewBoxes: Audio 570×680, Video 480×580, Network 480×480

**Fixed header positioning:**
- Header now `position: fixed; top: 0` with safe-area-inset as internal padding
- Background extends behind iOS status bar (no gap above header)
- Removed duplicate safe-area-inset-top from body padding (`padding-top: 0`)
- Changed `overflow-x: hidden` → `overflow-x: clip` for better iOS compatibility

**Smooth swipe-back gesture:**
- Replaced threshold-only swipe with follow-finger drag gesture
- Real-time `translateX` tracking via direct DOM manipulation (no re-render lag)
- On release: >35% screen width completes navigation with slide-out animation
- Below threshold: springs back to original position with ease-out
- Opacity reduces during drag for depth effect
- `[data-no-swipe]` attribute excludes specific elements (photo scroll areas)

**Minor:**
- Power Sequence item 6: "Monitor" → "Confidence Monitor"

## v5.4.0 (2026-02-28) — Task Organization & Dependency Warnings

**I/O Line List improvements:**
- Inputs tab auto-sorts by channel number on blur (numeric then alphabetic)
- Duplicate channel detection with red styling on inputs and outputs tabs
- Visual indicator only, doesn't block saving

**Task reorders:**
- Pre-Setup: Unload trailer → Pray → Assign roles
- Stage Box: Setup box → Setup speakers → Attach antennae → Power on → Run cables → Setup mics

**Task text updates:**
- ws-3: "Connect WiFi Router to Wing then power on"
- ds-3: "Power on Confidence Monitor"
- pp-5: "Test Pastor / Mami media videos"
- psc-1: "Transpose button set correctly for keyboardist (Michael-ON / Sunny-OFF)"
- dn-2: Removed "(right-most port)" from task text

**Equipment depersonalization:**
- Removed names from all equipment references (Glenn's, Mitch's, Omar/Mitch's)
- "ProPresenter Mac" and "ProTools Mac" throughout
- POWER_SEQUENCE also updated
- "LED Transmitter" without "(Small box)"

**New task:**
- cs-1b: "Setup ATEM Mini Pro" added to Camera Setup section

**Dependency warnings:**
- dn-3 (Power on Switch) warns if dn-1 (Plug in AVIO cabling) not complete
- pt-3 (Open ProTools Session) warns if pt-2 (Check Dante) not complete
- Amber warning modal: "⚠️ {task name} isn't complete yet. Continue anyway?"
- User can always proceed after confirmation
- No hard blocking

**Technical:**
- `dependsOn` property added to tasks
- Dependency checking in CheckItem component
- Warning modal with amber styling

## v5.3.2 (2026-02-28) — I/O Auto-Sort & Duplicate Prevention

**Inputs tab auto-sort:**
- Channel list automatically sorts by channel number after editing
- Numeric channels sort numerically (1, 2, 10 instead of 1, 10, 2)
- Non-numeric channels (like "L/R Main") sort alphabetically at the end
- Sorting triggers on blur (when you finish editing the channel number)
- Uses `parseInt()` for numeric sorting, falls back to string comparison

**Duplicate prevention (Inputs & Outputs):**
- Red background and text when channel number is duplicate
- Visual feedback appears immediately while typing
- Prevents confusion from multiple channels with same number
- Works on both Inputs and Outputs tabs
- Dante tab unaffected (no channel numbers)

**Technical:**
- New `sortInputsByChannel()` helper function
- New `isDuplicateChannel()` validation function
- `handleChannelBlur()` triggers auto-sort on inputs tab
- Deep clone pattern maintained for state updates

## v5.3.1 (2026-02-28) — Editable I/O Line List

**Editable I/O configuration:**
- "✏️ Edit I/O" button toggles edit mode in I/O Line List view
- All channel fields editable inline: labels, types, notes, destinations, channel numbers
- Channel numbers accept any text input (not just numbers) — allows "L/R Main", "Aux 1", etc.
- Add channels: + button at bottom of each tab (Inputs, Outputs, Dante)
- Remove channels: × button per row in edit mode (instant removal, no confirmation)
- Type dropdown for inputs: XLR, DI/XLR, Line, 1/4"
- Protocol dropdown for Dante routes: Dante, Analog, AES50, USB
- Changes save immediately to Firebase via `elim3-io-config` storage key
- New channels default to last channel number for inputs (user must edit), "Aux N" for outputs

**Reset to defaults:**
- Reset button with two-tap confirmation ("Reset" → "Confirm Reset" + "×")
- Restores original hardcoded `DEFAULT_IO_LINE_LIST`
- Exits edit mode after reset

**Storage & sync:**
- New storage key: `elim3-io-config`
- Defaults to `DEFAULT_IO_LINE_LIST` constant (16 inputs, 5 outputs, 3 Dante routes)
- User edits override defaults
- Real-time cross-device sync via Firebase
- Deep clone used for state updates to ensure React re-renders

**Technical:**
- Extracted `DEFAULT_IO_LINE_LIST` constant from hardcoded `IO_LINE_LIST`
- `IOListView` uses `useStorage` hook for lazy loading (not in main app loaded check)
- View mode: clean read-only display (unchanged from v5.3.0)
- Edit mode: inline inputs replace static text

## v5.3.0 (2026-02-28) — Crew Focused Views + Unassigned Awareness + UX Refinements

**Crew focused views (replaces filtered checklist):**
- Tapping a crew member name in Crew Status opens a focused task list showing only their tasks grouped by section — same format as My Tasks
- Header shows "{Name}'s Tasks" with timer in sticky header
- Cleaner than previous approach of showing full checklist with filter bar
- Tasks assigned to you still get orange highlight even in someone else's focused view

**Unassigned task awareness:**
- "Unassigned tasks" row at bottom of Crew Status on dashboard with progress bar and count
- Separated from crew list with divider line and dashed border styling
- Taps through to focused list of all unassigned tasks grouped by section (header: "Unassigned Tasks")
- Unassigned task cards: dashed border, lighter background, italic dimmed text — visually distinct from assigned tasks
- Section headers show "{N} unassigned" in italic when section has unassigned incomplete tasks
- Sections where all incomplete tasks are unassigned: title goes italic/muted with dimmed icon
- Empty state: "All tasks are assigned — Every task has an owner."

**Dashboard hero redesign:**
- Stacked vertical layout: progress zone on top, timer zone below, separated by divider
- Progress zone tappable → navigates to full checklist (no auto-expand, clean overview)
- 48px centered percentage, subtitle "X of Y tasks · View all →", full-width progress bar
- Timer zone with stopPropagation — timer controls don't accidentally navigate to checklist
- Timer picker X button moved to top-right of panel
- History stat line shown below timer

**Header redesign:**
- Logo left (32px ElimLogo), username truly centered via absolute positioning with ▾ dropdown indicator
- Tapping centered name opens user selection (replaces "Switch" button)
- ⚙️ gear icon top-right navigates to Settings
- Removed "Signed in as" label and orange "Switch" button

**Settings restructured:**
- Reset checklist moved to top of Settings (was on dashboard hero)
- Order: Reset → Team Roster → Setup History → About
- History keeps last 52 sessions (was 20) — covers ~1 year of weekly setups

**Swipe-back gesture expanded:**
- Now works from anywhere on screen (was left 30px edge only)
- 80px minimum horizontal distance, must be 2x vertical movement
- Photo scroll areas tagged with `data-no-swipe` to prevent conflicts

**Section header separators:**
- Thin warm-toned 1px line between collapsed section headers for visual separation
- Line disappears when section is expanded

**Power sequence badge:**
- Moved from inline in task text to right side of card
- Enlarged to 16px font with more padding, vertically centered in card

**Navigation cards:**
- Removed Checklist and Settings cards from dashboard grid (access via hero and header ⚙️)
- Final 3×2 grid: Power Sequence, I/O Line List, Signal Flow, Equipment, Repairs, Purchases

**Desktop scroll fix:**
- `overflow-x: hidden` moved from `html, body, #root` to `body` only — fixes scroll wheel on desktop browsers

**Bug fixes:**
- Fixed "Dante AVIÓ" → "Dante AVIO" (accented Unicode Ó replaced with plain ASCII O)
- Background color reverted from `#1A1918` back to original `#111010`

## v5.2.0 (2026-02-27) — Countdown Timer + Weekly Reset + SVG Brand Mark + UX Polish

**Countdown timer (was v5.1.2 roadmap item):**
- Tap dashboard timer → "⏱ Set Timer" opens inline picker
- Presets: 30, 45, 60, 90 minutes, plus custom input (1–300 min)
- Displays remaining time counting down with color transitions: green (>50%) → yellow (25–50%) → red (<25%)
- No flashing — calm color shift only
- "change" link below active countdown to swap the target
- "Remove countdown (use count-up)" option to revert to elapsed timer
- Auto-pauses and shows "⏰ Time!" when countdown reaches zero
- Countdown target persists to Firebase — setting on one phone shows on all devices
- Checklist header and My Tasks header both show countdown with matching colors
- `timerColor()` helper function for color computation

**Weekly reset (was v5.3 roadmap item — moved up by request):**
- "New Week" button on dashboard hero area (below progress bar)
- Two-tap confirmation: first tap shows "Clear all progress?" with "Yes, Reset" (red) + "Cancel"
- Clears: checkboxes, completions, timer
- Preserves: assignments, task notes, photos, roster
- Saves lightweight history record before clearing: `{date, pct, elapsed, tasks, total}`
- Keeps last 20 weeks of history
- Dashboard shows last week's stats: "Last: 2026-02-26 — 100% in 1:12:34"
- Storage key: `elim3-history`

**SVG brand mark:**
- Replaced all 5 text-based `»E` instances with proper SVG `ElimLogo` component
- Two right-pointing geometric chevron strokes + block E with three arms, inside circle
- Sharp corners on both chevrons (`strokeLinecap="square"`, `strokeLinejoin="miter"`) and E (no border-radius)
- Inner content scaled to 82% via `<g transform>` for breathing room inside circle
- Used at 32px (dashboard header), 56px (loading state), 76px (login screen)
- Pre-React HTML loading screen uses matching inline SVG
- Old `.bolt` CSS class removed

**Background color:**
- Briefly lightened to `#1A1918`, reverted to original `#111010` after review

**Dashboard header redesign:**
- Left: SVG brand mark (32px)
- Center: "Signed in as" label + user name (18px, centered)
- Right: orange "Switch" button (replaces small "Glenn ▾" dropdown)

**Timer in My Tasks:**
- Timer now visible in sticky header for both Checklist and My Tasks views
- Checking a box in My Tasks auto-starts the timer

**Solid triangle expand arrows:**
- Section expand: text chevrons → `▶`/`▼` solid triangles (12px, 28px min-width hit area)
- Task expand: solid triangles at 10px
- Troubleshooting expand: same treatment
- Much more visible and tappable than previous `›`/`⌄` characters

**Back swipe gesture:**
- `useBackSwipe` hook detects right swipe from left 30px edge
- Navigates back to dashboard from any subpage
- Works alongside ← button

**Other UX:**
- Checklist Reset button removed from checklist header (New Week on dashboard is single reset path)
- "tap to view →" hint added to Crew Status section header on dashboard
- Login screen logo centered with flexbox (was off-center with `margin: auto`)

**Deferred items (discussed, not implemented):**
- Section-level assignments — deferred for later assessment
- Crew status vs checklist redundancy — resolved in v5.3.0 with focused crew views

## v5.1.1 (2026-02-26) — Readability + Crew Filter + Timer Persistence
**Readability overhaul — all text sizes increased for mobile usability:**
- Task text: 13px → 16px (recommended mobile body text)
- Section headers: 14px → 16px, icons 17px → 20px
- Section expand arrows: 10px → 20px, switched to `⌄`/`›` chevrons with dedicated min-width so they don't get squeezed
- Task-level expand arrows: 10px → 16px
- Checkboxes: 24px → 28px with larger checkmark for better thumb tap targets
- Assign dropdowns: 11px → 14px
- Detail blocks, troubleshooting Q&A, notes: all 14px
- Section labels (Troubleshooting, Notes, Crew Status, Team Roster): 11px → 12px
- I/O channel labels, Dante routes, power sequence: 14–16px
- Login buttons, Settings description, filter bar, empty states: all bumped
- Only remaining 10–11px text: tiny badges ("YOU" pills, type tags)
- Section header row padding increased for larger tap targets

**Checklist header restructured:**
- Timer and Reset button moved from checklist body into sticky header bar (inline with ← and title)
- Progress bar now spans full width with percentage alongside
- Progress bar slightly thicker (6px → 8px)
- Header title: 17px → 18px, back arrow: 20px → 22px

**Crew filter auto-expand:**
- When filtering by crew member from dashboard, sections with their assigned tasks auto-expand
- Filtered person's tasks get orange left border highlight (same as "your" tasks)
- Manual collapse state resets when filter changes so auto-expand recalculates
- "Show All" reverts to only highlighting your own tasks

**Timer persistence:**
- Timer state now persisted to Firebase via `elim3-timer` storage key
- Stores `startedAt` timestamp + `accumulated` seconds — survives page refresh on any device
- Timer syncs across devices in real time (all phones see the same timer)
- Reset button clears timer along with checkboxes
- Auto-start on first checkbox still works

**Data safety note:**
- Redeploying (`firebase deploy`) only replaces app code (`index.html`). All team data (checkboxes, assignments, notes, photos, roster, timer) lives in Firebase Realtime Database and is never affected by code deployments.

## v5.1.0 (2026-02-26) — Warm Night Theme + UX Simplification
**Theme overhaul — "Warm Night" design philosophy:**
- 3-state color system: orange (your tasks), green (completed), neutral warm grey (everything else)
- Background: #0B0D11 → #111010 (warmer black)
- Text: cool greys → warm creams (#F0E6D8, #B8A894, #7A6E60)
- All rgba(255,255,255,...) → warm-tinted equivalents
- Softer accent: #E8893A → #D4763A, green #10B981 → #5CB87A
- Per-person team colors removed from all UI except Settings roster management

**Removed (clutter reduction):**
- Team progress strip from checklist header
- Per-person filter buttons from checklist
- Section timers (per-section stopwatches)
- Completion timestamps ("✓ Glenn · 7:02 AM")
- SectionTimer component (dead code)
- TeamStrip component (dead code)
- Per-section colored progress bars (now neutral)
- "Welcome, {name}" subtitle in header

**Added:**
- Crew names clickable on dashboard → opens checklist filtered to that person
- Filter indicator bar in checklist ("Showing: Omar" + "Show All" button)
- `initialFilter` prop for checklist navigation from dashboard

**Changed:**
- Section headers: cleaner layout with count/✓ right-aligned, progress bar inside expanded area
- My Tasks: section count → green ✓ when all tasks in section complete
- "YOU" badge only shows when you have incomplete tasks (not after finishing)
- CheckItem: removed completedBy prop, simplified to 3-state visual

**Fixed:**
- React hooks ordering bug — `useCallback` for `navigate` was after early returns, causing crash when clicking a username on login screen (React error #310)

**Hardened:**
- Firebase init wrapped in try/catch — app renders even if Firebase CDN blocked
- `_ls` safe localStorage wrapper — all localStorage calls wrapped in try/catch for environments that block storage APIs
- App degrades gracefully: Firebase unavailable → localStorage only → in-memory only

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
- Deploy command: `firebase deploy` from `C:\Users\sammh\elim-production\`

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