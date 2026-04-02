# ELIM Production App — Changelog

## v5.12.0 (2026-04-02) — dB Meter + Defensive Batch

**dB Meter (Sam/Tyler only):**
- New `DecibelMeterView` component accessible from a conditional dashboard card (only visible to Sam and Tyler)
- Web Audio API pipeline: `getUserMedia` with AGC, noise suppression, and echo cancellation explicitly disabled for accurate readings
- dBFS reading from time-domain RMS via `getFloatTimeDomainData` (Float32Array, no quantization loss)
- A-weighted (dBA) reading via `getFloatFrequencyData` with IEC 61672 A-weighting formula applied per FFT bin
- Estimated SPL: dBA + 94 dB reference offset + user-adjustable calibration offset
- `fftSize = 4096`, `smoothingTimeConstant = 0` (own EMA smoothing at 0.85 coefficient)
- Peak hold with 2-second hold then ~3 dB/sec decay, tracked independently for both dBFS and dBA
- Horizontal meter bar with green/yellow/red color zones and peak marker line
- Calibration controls: +/- buttons to adjust SPL estimate against Wing meters
- Expandable "How This Works" guide explaining dBFS, dBA, Est. SPL, peak behavior, calibration procedure, and accuracy notes
- Proper cleanup: stops media tracks, closes AudioContext, cancels animation frame on unmount
- Permission-denied state with retry button
- View route: `view === "db-meter"`, title "dB Meter"

**Data validation on load:**
- `SCHEMA_TYPES` constant maps each storage key to its expected type (`record-boolean`, `record-string`, `record-number`, `array`, `object`)
- `sanitize(key, data, initial)` function validates parsed data against schema after JSON parse
- For `record-*` types: drops individual entries where value type doesn't match, logs count to console
- For `array` / `array-string`: confirms Array type, filters non-string entries for array-string
- For `object`: confirms non-null, non-array object type
- Falls back to `initial` value if top-level type is wrong
- Wired into both `useStorage` read paths: initial Firebase load and real-time sync handler

**Schema versioning scaffold:**
- `SCHEMA_VERSION = 1` constant, `MIGRATIONS = []` array (empty — scaffold only)
- `runMigrations()` async function: reads `elim3-schema-version`, runs any migrations with index ≥ stored version, writes new version
- Migration functions receive `(get, set)` helpers for reading/writing storage keys
- Triggered once after all `useStorage` hooks report loaded, guarded by `migrationsRun` ref
- Enables future non-destructive data migrations (e.g., photo blob split)

**Error boundary data export:**
- "Export Backup" button added to ErrorBoundary alongside "Tap to Reload"
- Reads all 9 `elim3-*` keys + `elim3-schema-version` from Firebase/localStorage
- Builds JSON object with `exportedAt` timestamp and error string
- Creates downloadable `elim-backup-YYYY-MM-DD.json` via Blob URL
- Green styling to distinguish from amber reload button
- Error text updated to mention export option

**Version:** bumped to v5.12.0

---

## v5.11.0 (2026-04-01) — I/O Page Redesign + Bug Fixes

**I/O Page redesign (render-side only, no data model changes):**
- Reduced from 4 tabs to 3: Stage Box (with Inputs/Outputs sub-toggle), Wing, Dante
- Stage Box Inputs: 6 named channel groups (Drums 1-12, Instruments 13-25, Talk Backs 26-27, Vocals 28-36, Wireless 37-40, Crowd 41-44) with inline group headers and channel counts
- Stage Box Outputs: IEM/Infrastructure split — outputs with destination containing "in-ear" grouped under "In-Ear Monitors", remainder under "Infrastructure"
- In-ear snake pill badge: `inEarSnake` field rendered as prominent accent-colored pill with 🎧 prefix (replaces small text line)
- Cross-tab search: search field searches across all tabs simultaneously, showing grouped results with section headers
- Edit mode restricted to Sam and Bri (`currentUser === "Sam" || currentUser === "Bri"`)
- `IODropdown` custom component replaces native `<select>` for type and protocol fields (matches dark theme)
- Compact pencil edit button (✏️) replaces verbose "✏️ Edit I/O" text
- `renderGroupHeader` helper for consistent section dividers

**Bug fixes:**
- cs-1b detail text corrected: ATEM Ethernet connects to AX55 router LAN port for remote control; Resi gets internet directly from venue ethernet (separate network)
- Task detail text overflow: added `WebkitLineClamp: 5`, `wordBreak: "break-word"`, `overflowWrap: "break-word"` to detail and note preview elements

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
- Reset via `newWeek` clears the `autoStoppedRef` flag
- One-way gate: unchecking a task does NOT restart the timer

**Manual "Setup Complete":**
- Button in Settings stops timer and saves current progress to history
- Saves `{date, pct, elapsed, tasks, total}` record to history
- Two-tap confirmation to prevent accidental taps
- Does NOT clear checkboxes or assignments — only stops timer and logs progress

**Other:**
- My Tasks collapsible sections: completed sections auto-collapse to a single green ✓ header line
- `manualExpand` state per section — tap any header to toggle; manual override remembered per session
- Auto-collapse default: sections with all tasks done collapse; incomplete sections expand
- Scroll position retry: up to 10 attempts with 5px tolerance for async-loaded views

**Version:** bumped to v5.9.0

---

## v5.8.0 (2026-03-01) — Bug Fixes, UX Polish

**Bucket A — Bug fixes (shipped same day):**
- **Dependency warning modal** — backdrop opacity increased to 0.6 for readability
- **Scroll position preservation** — retry-based restore for async-loaded views
- **Scrollbar hidden** — CSS `scrollbar-width: none` + `::-webkit-scrollbar { display: none }`
- **Assignment dropdown** — custom dark-themed dropdown replaces native `<select>`: 15px font, 10px 14px padding, click-outside-to-close, unassign option, ✓ indicator
- **Populated notes surfaced** — note preview (📌) shown below assignee without expanding; hidden on checked tasks; hidden when task expanded (full editor shown instead)
- **Dante I/O field overflow** — edit fields constrained with `minWidth: 0` + `width: 100%` in flex containers

**Bucket B — UX improvements (shipped same day):**
- **Default My Tasks view** — app opens to My Tasks instead of Dashboard; Dashboard accessible via back button
- **Search toggle** — checklist search bar hidden behind 🔍 icon; expands on tap, collapses when empty + blurred
- **Placeholder cards removed from dashboard** — Equipment and Purchases cards removed; dashboard nav grid reduced to essential items
- **"Last setup" stat removed from dashboard hero** — still accessible in Settings → Setup History

**Storage keys added:** `elim3-pending-reset` (transient, deleted after reset completes)

**Version:** bumped to v5.8.0 (originally released as v5.7.0 with Bucket A, bumped for Bucket B)

---

## v5.7.0 (2026-02-28) — Repairs CRUD + Checklist Edit Mode + Search + Audio SVG Redesign

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
- New tasks assigned timestamp-based IDs (e.g. `task-1709123456789`)

**Audio Signal Chain SVG redesign:**
- Full restructure of audio chain to accurately represent main, IEM, sub output paths
- Tyler's Mac positioned on left sidebar (USB-C direct) — separated from Dante branch
- WiFi Router positioned on right sidebar — control path, not audio
- Clear left/right fork: analog outputs (FOH, Subs, IEMs, Bianca IEM) vs Dante path (Switch → Macs → AVIO → ATEM)
- All label and box sizes adjusted for readability on mobile
- Viewbox expanded to 570×660 to accommodate added detail

**Version:** bumped to v5.7.0

---

## v5.6.0 (2026-02-27) — Architecture: Offline Resilience + Sync Robustness

**Smart `prevRef` diffing for offline merges:**
- Added `prevRef` tracking per `useStorage` instance
- Save function compares new value against `prevRef` using key-level diff
- Partial object changes (< total keys changed) route through `storage.merge()` → Firebase `.update()`
- Full replacements route through `storage.set()` → Firebase `.set()`
- Reduces Firebase write surface and minimizes merge conflicts during offline/online transitions

**`_pendingWrites` Set for blocking stale sync events:**
- New `_pendingWrites` global Set tracks keys with in-flight Firebase writes
- Firebase `.on('value')` listener skips `elim-sync` dispatch for keys in `_pendingWrites`
- Prevents stale Firebase data from reverting a user's local change before the write completes
- Write success clears the key from `_pendingWrites`; retry failure also clears to allow listener to resume

**`storage.merge()` method:**
- New method on `window.storage` API
- Reads current value, applies field-level changes, writes merged result
- Smart routing: if Firebase value is a native object, uses `.update(fields)` for atomic field merge; if it's a legacy JSON string, uses `.set(fullObj)` for full replacement
- Read-path normalization: handles both old JSON string format and new native object format transparently

**Firebase write retry with exponential backoff:**
- Both `storage.set()` and `storage.merge()` use retry loop: 4 attempts, 2s initial delay, doubles up to 30s max
- Failed writes tracked via `_pendingWrites` — incoming sync events ignored during retry
- `_pendingWrites` cleared on final retry failure (one-line fix in catch handler's exhausted-retries branch)
- Sync error dispatched on final failure: `elim-sync-status` event with `status: 'error'`
- Sync success dispatched on write completion: `elim-sync-status` event with `status: 'ok'`

**`storage.get()` fallback path fix:**
- localStorage fallback no longer pushes its value back to Firebase
- Prevents stale cache from clobbering newer remote data written by other devices while this device was offline

**Sync error indicator:**
- 🔴 icon appears in header when a Firebase write fails after all retries
- Cleared when the next write succeeds

**Offline awareness:**
- Firebase `.info/connected` listener monitors device connectivity
- Amber "Offline — changes saved locally" banner appears at top of app when offline
- Banner hides when connectivity restores

**CHECKLIST_VERSION stamp:**
- Added `CHECKLIST_VERSION` constant to prevent structural data drift
- Stored version checked on load; stale data triggers re-seed from code

**Unassigned task styling:**
- Dashed border, lighter background, italic dimmed text for unchecked unassigned tasks
- `isUnassigned` computed from `!assignee && !checked` in `CheckItem`
- Section headers show unassigned count
- Dashboard shows "Unassigned tasks" row with dashed border below crew list

**Version:** bumped to v5.6.0

---

## v5.5.2 (2026-02-27) — Data Fidelity Rebuild

**Full data audit and correction pass:**
- All 15 sections, 62 tasks — every word verified against original spreadsheet, physical setup, and actual connections
- All photo slots verified
- All troubleshooting Q&As checked for accuracy
- Power sequence verified against actual power-on/off order with specific equipment names
- I/O Line List preserved (16 inputs, 5 outputs, 3 Dante routes — pre v5.9.0 data update)

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
