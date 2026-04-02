# ELIM Production App — Backlog

All planned work, organized by priority and category. Items move to CHANGELOG.md when shipped.

Last updated: 2026-04-01

---

## Priority 0: Active Bugs

(none)

---

## Priority 0: Fixed

### ~~cs-1b Task Detail Factually Wrong~~ — FIXED in v5.11.0
### ~~ATEM Name Correction~~ — FIXED in v5.9.0
### ~~I/O Line List Data Update~~ — FIXED in v5.9.0
### ~~Post-Event Timer Exclusion~~ — FIXED in v5.9.0
### ~~Timer Stop UX~~ — FIXED in v5.9.0 (auto-stop at 100% + manual Setup Complete in Settings)
### ~~I/O Line List Persistence~~ — FIXED in v5.10.0 (useStorage with draft-based editing)
### ~~Dependency Warning Modal Unreadable~~ — FIXED in v5.7.0
### ~~Dante I/O Fields Overflow~~ — FIXED in v5.8.0
### ~~Assignment Dropdown Styling~~ — FIXED in v5.8.0
### ~~Populated Notes Hidden~~ — FIXED in v5.8.0
### ~~Visible Scrollbar~~ — FIXED in v5.8.0
### ~~Scroll Position Resets~~ — FIXED in v5.8.0 (retry-based restore added in v5.9.0)
### ~~Task Detail Text Overflow~~ — FIXED in v5.11.0 (line clamp 5 + word-break)

---

## Priority 1: UX Improvements

### Completion Feel
- Brief animation on task check-off
- Section completion celebration (subtle)
- 100% overall completion moment
- "You're done! X tasks still unassigned" proactive nudge when personal tasks complete

---

## Priority 1: Completed

### ~~I/O Page Redesign~~ — SHIPPED in v5.11.0
Shipped as: 3 tabs (Stage Box, Wing, Dante) with Inputs/Outputs sub-toggle on Stage Box. Stage Box inputs grouped into 6 sections (Drums, Instruments, Talk Backs, Vocals, Wireless, Crowd) with "Other" fallback. Outputs grouped into IEM/Infrastructure by destination. In-ear snake pill badge (🎧). Cross-tab search. Edit gated to Sam and Bri. Compact pencil edit button. Custom IODropdown replacing native selects.
### ~~Navigation Model~~ — SHIPPED in v5.7.0
### ~~Reduce Visual Noise~~ — SHIPPED in v5.7.0 / v5.8.0
### ~~My Tasks Collapsible Sections~~ — SHIPPED in v5.9.0

---

## Priority 2: Checklist Task Corrections

(none active — cs-1b fixed in v5.11.0)

---

## Priority 2: Completed

### ~~cs-1b Detail Fix~~ — FIXED in v5.11.0
### ~~Drum click batteries~~ — SHIPPED in v5.9.0 (mc-3)
### ~~Stage spotlight~~ — SHIPPED in v5.9.0 (cs-3)
### ~~Speaker/wall spacing reference~~ — SHIPPED in v5.9.0 (sb-4 detail)
### ~~Stage box wheel stability~~ — SHIPPED in v5.9.0 (sb-1 troubleshooting)
### ~~4-camera split~~ — SHIPPED in v5.9.0 (cs-1, cs-4, cs-5, cs-6)
### ~~Glenn → functional naming~~ — SHIPPED in v5.9.0

---

## Priority 3: Technical Debt

### Photo Blob Split
- **What:** All photos stored as base64 strings in a single `elim3-photos` JSON blob. Each upload rewrites the entire blob. At scale (30+ photo slots, 50-100KB each), this means 2-3MB downloaded on every app load across 16 devices.
- **Fix:** Split into individual storage keys (`elim3-photo-{slot}`). Migration: read old blob, write individual keys, delete blob. Don't move to Firebase Storage (adds SDK weight and auth complexity) — individual RTDB keys are fine at this scale.
- **Risk:** Biggest structural scalability issue. Current auto-resize (800×600, 70% JPEG) buys time but doesn't solve the architecture.

### Data Validation on Load
- **What:** Every `useStorage` call trusts whatever comes back from Firebase. If a value is corrupted or wrong type, the app white-screens. Open Firebase rules + 16 concurrent writers = one malformed write from a crash.
- **Fix:** Add `sanitize(key, data, schema)` function called inside `useStorage` after parsing. Each key gets a simple schema: `elim3-checks` = `Record<string, boolean>`, `elim3-assign` = `Record<string, string>`, etc. Malformed fields dropped, missing fields get defaults.
- **Effort:** Small — ~30-40 lines, called once per key on load.

### Schema Versioning
- **What:** No way to detect and migrate data format changes non-destructively. Currently handled with full resets, but as persistent data grows (I/O config, repairs, history, photos), resets become more expensive.
- **Fix:** Add `elim3-schema-version` (integer, starts at 1). On load, if stored version < current version, run `migrations` array of functions sequentially. Start simple — just having the version number in place enables future migrations.
- **Effort:** Small.

### Error Boundary Data Export
- **What:** Current error boundary only has "Tap to Reload." If the app crashes and can't recover after reload, Sam can't access the data.
- **Fix:** Add "Export Backup" button alongside "Tap to Reload" that dumps all Firebase data keys as JSON.
- **Effort:** Small — one button, one function.

---

## Priority 4: Code Quality

### Remove dead weight
- Delete `PlaceholderView` component if still present
- Remove any remaining placeholder routes from main app component

### Reduce duplication
- Extract `JSON.parse(JSON.stringify())` deep clone calls into `deepClone()` utility (used 8+ times)
- Extract common edit toolbar pattern (used in Checklist, I/O, Repairs) into shared component

### Evaluate
- Signal Flow SVGs (~200 lines of hand-crafted SVG) — keep as-is. Photos are easier to update but SVGs are always sharp and professional. The maintenance burden is low since the signal chain changes rarely.

---

## Priority 5: Future Features

### Teardown Checklist
- Reverse of setup with teardown-specific tasks
- Power sequence already has teardown guidance but no checklist
- Design question: should teardown tasks share state with setup tasks or be independent? Independent is safer — teardown shouldn't accidentally clear setup progress.

### Service Worker (Full Offline Caching)
- Currently the app requires internet for first load
- A service worker with cache-first strategy would make the app truly offline-capable after first install
- Complex: cache versioning, update detection, iOS Safari quirks
- Dedicated session, not a side task

---

## Evaluated and Deferred

### Task Card Density (Part A: Assignee as Plain Text)
- Show assignee as plain text by default; dropdown only when unassigned or in assign mode
- **Deferred:** Sam decided to leave current behavior for now. Reassess later as low priority.

### "All" Assignment Option
- Add "All" to assignment dropdown for communal tasks (prayer, unloading)
- **Deferred:** Only 2-3 tasks would use it. Not worth the complexity. Assign shared tasks to crew lead if the dashed styling is distracting.

### Unassigned Task Styling Change
- Change unassigned tasks from dashed/dim to solid neutral styling
- **Deferred:** Sam prefers to assign shared tasks to himself rather than change styling.

---

## Evaluated and Rejected

### Dynamic Signal Flow SVGs from I/O Data
Building a graph layout engine to auto-position SVG boxes from I/O data. Multi-hundred-line undertaking in a single-file app. The I/O list changes maybe twice a year. Manual SVG updates take 10 minutes. ROI deeply negative.

### Equipment Inventory View
Full equipment inventory with serial numbers, condition tracking. A shared Google Sheet does this better. Not a setup-day problem.

### Purchases View
Purchase tracking list. Apple Notes or a shared spreadsheet handles this. Not a setup-day problem.

### Section-Level Team Assignments
Assign teams to entire sections. Creates a second assignment system that conflicts with per-task assignments. Individual task assignments persist across resets. The "first-time setup is tedious" problem happens once per roster change.

### Setup Time Analytics
Trend visualization from history data. Export `elim3-history` JSON from Firebase console and use a spreadsheet. Not worth in-app code weight.

### Live Notifications
Push notifications via service worker. 16 people in the same room can communicate verbally. The crew status dashboard already shows who's done.

### Photo Comparison (Side-by-Side)
This week vs reference photos. Reference photos already live in task expand slots. A laminated card is a better training tool.

### Training Mode
Expanded instructions + video links. Belongs in a Google Doc or printed SOP, not inline in a speed-optimized checklist.

### In-App Decibel Meter
Web Audio API SPL meter restricted to Sam/Tyler. Phone mics are uncalibrated. NIOSH SLM app or dedicated SPL meter does this better. Wrong tool for a setup checklist app.

### Shazam-like Song Identification
Requires external API + subscription. Shazam already exists as a free app. Zero Sunday morning value.

### Song Key Signature Detection
Browser-based pitch detection fails for polyphonic live music. If someone needs the key, they have chord charts. Setlist/planning problem, not a production setup problem.

---

## Priority 6: Physical / Process Improvements (Not App Features)

### Repairs Tracker Items
New items to add to `DEFAULT_REPAIRS` constant:
- Velcro for Resi and switch
- Locktite: piano legs
- Locktite: wall base
- Spray adhesive: wall boxes
- Trailer: flooring reinforcement
- LED Wall cart wheels
- LED Wall cart latching
- LED Wall box stacking
- Wing case re-rack and light

### Church-Wide Production Standards
Should be documented as standalone reference materials (laminated cards, case lid references) independent of the app:
- Power sequence (on 1-10, off 10-1)
- Three-network architecture diagram
- Pelican case and Wing case packing checklists (taped inside case lids)
- Cable color code reference (when implemented)

### Stage Power Distribution
- 50ft 12/3 extension cords + 6-outlet surge strips at 5 positions
- Pending: PA speaker power draw measurement (Kill-A-Watt)

---

## Notes

- P0: Fix immediately — actively wrong information
- P1: Discuss before implementing — affects app UX
- P2: Can be done independently — data corrections
- P3: Technical debt — improves reliability with no user-facing change
- P4: Code quality — no urgency
- P5: Build one at a time, not all at once
- P6: Physical/process work, tracked here so it doesn't get lost
