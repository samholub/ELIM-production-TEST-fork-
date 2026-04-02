# ELIM Production App — Backlog

All planned work, organized by priority and category. Items move to CHANGELOG.md when shipped.

Last updated: 2026-04-02

---

## Priority 0: Active Bugs

(None)

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

(No active P1 items)

---

## Priority 1: Completed

### ~~Navigation Model~~ — SHIPPED in v5.7.0
### ~~Reduce Visual Noise~~ — SHIPPED in v5.7.0 / v5.8.0
### ~~My Tasks Collapsible Sections~~ — SHIPPED in v5.9.0
### ~~I/O Page Redesign~~ — SHIPPED in v5.11.0
(3 tabs, 6 input groups, IEM/Infrastructure output split, 🎧 pill badge, cross-tab search, edit restricted to Sam/Bri, IODropdown, compact pencil edit)

---

## Priority 2: Checklist Task Corrections

(No active P2 items)

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
- **Fix:** Split into individual storage keys (`elim3-photo-{slot}`). Migration: read old blob, write individual keys, delete blob. Use the `SCHEMA_VERSION` / `MIGRATIONS` scaffold (shipped in v5.12.0) for the first real migration.
- **Risk:** Biggest structural scalability issue. Current auto-resize (800×600, 70% JPEG) buys time but doesn't solve the architecture. No photos currently uploaded, so migration is trivial right now.
- **Scope:** ~60-80 lines across 4-5 code locations. Custom `usePhotoStorage` hook, migration function, loaded gate update, error boundary backup update. Full session, not a quick fix.

---

## Priority 3: Completed

### ~~Data Validation on Load~~ — SHIPPED in v5.12.0
(`sanitize()` function with `SCHEMA_TYPES` in `useStorage` read path)

### ~~Schema Versioning~~ — SHIPPED in v5.12.0
(`SCHEMA_VERSION = 1`, `MIGRATIONS` array scaffold, `runMigrations()`)

### ~~Error Boundary Data Export~~ — SHIPPED in v5.12.0
("Export Backup" button dumps all `elim3-*` keys as downloadable JSON)

---

## Priority 4: Code Quality

### ~~Remove dead weight~~ — DONE (verified v5.12.0)
PlaceholderView removed, no dead routes or placeholder nav cards remain.

### Reduce duplication
- Extract `JSON.parse(JSON.stringify())` deep clone calls into `deepClone()` utility (8 instances across ChecklistView and IOListView)
- **Status:** Low value — cosmetic cleanup only, no behavior change. Skip unless doing a broader refactor pass.

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

### Completion Feel (Animations)
Check-off animations, section celebrations, 100% completion moment. Evaluated and rejected — the progress bar, section auto-collapse, and timer auto-stop already provide sufficient completion feedback. Animations add code weight and rendering overhead for zero Sunday morning value. This is a task-oriented tool used while holding cables, not a consumer engagement app.

### Shazam-like Song Identification
Requires external API (AudD, $5/1000 requests after 300 free). Shazam already exists as a free app on every phone. Zero overlap with setup coordination. Fails the Sunday morning test.

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
