# ELIM Production App — Backlog

All planned work, organized by priority and category. Items move to CHANGELOG.md when shipped.

Last updated: 2026-03-29

---

## Priority 0: Bugs / Data Corrections

Active bugs and data accuracy issues.

### ATEM Name Correction
- **What:** App code and Signal Flow SVGs say "ATEM Mini Pro" in several places. Actual device is ATEM Television Studio Pro 4K.
- **Where:** Equipment Reference, checklist tasks cs-1b, cs-2, cs-1, signal flow SVGs
- **Fix:** Find-and-replace in `index.html`. No design decisions needed.

### I/O Line List Data Update
- **What:** `DEFAULT_IO_LINE_LIST` has 16 generic inputs, 5 outputs, 3 Dante routes. Real setup has 46 input lines (44 stage box + Wing 8 Bianca TB + Dante 47-48 ProPresenter), 19+ output lines with in-ear snake assignments.
- **Source:** Actual I/O list captured on-site 2026-03-08 (screenshots in project context)
- **Fix:** Replace `DEFAULT_IO_LINE_LIST` constant with real data, deploy + reset

### Post-Event Tasks Excluded from Timer
- **What:** The timer measures setup speed, but post-event tasks (teardown) inflate the total task count. Timer should stop counting when setup is "done" (pre-service checks complete), not when everything including teardown is checked.
- **Fix:** Exclude the Post-Event section from the timer's total/progress calculation. Post-event tasks still appear and function normally, just don't affect the setup timer metric.

### Timer Stop UX
- **What:** "How is the timer stopped when setup is done?" — this question came up during a live setup, indicating the timer behavior isn't obvious.
- **Current behavior:** Timer auto-pauses at zero (countdown from target time). There's no explicit "setup done" trigger.
- **Fix:** Evaluate whether the timer should auto-stop when pre-service checks hit 100%, or whether there should be a manual "Setup Complete" action. Ties into the post-event exclusion above.

---

## Priority 0: Already Fixed

### ~~Dependency Warning Modal Unreadable~~ — FIXED in v5.7.0
### ~~Dante I/O Fields Overflow Screen Edge~~ — FIXED in v5.8.0
### ~~Assignment Dropdown: White Background on Mobile~~ — FIXED in v5.8.0
### ~~Assignment Dropdown: Too Small to Read~~ — FIXED in v5.8.0
### ~~Populated Notes Hidden in Expand~~ — FIXED in v5.8.0
### ~~Visible Scrollbar~~ — FIXED in v5.8.0
### ~~Scroll Position Resets on Navigation~~ — FIXED in v5.8.0

---

## Priority 1: UX / Navigation

Strategic decisions about how the app should feel and flow. Discuss before implementing.

### ~~Navigation Model~~ — SHIPPED in v5.7.0
### ~~Reduce Visual Noise~~ — SHIPPED in v5.7.0 / v5.8.0

### Task Card Density
- Show assignee as plain text (not dropdown) by default — during setup, assignments are already set
- Only show assignment dropdown when task is unassigned or in an explicit "assign" mode
- Evaluate whether the expand arrow, note indicator, and power badge density is right
- **Add "All" assignment option** — for tasks everyone does (prayer, unloading, etc.). Currently these either stay unassigned or get assigned to one person arbitrarily.

### Section-Level Team Assignments
- Assign teams/people responsible for entire sections instead of individual tasks
- Came up during live setup as "Add teams responsible for sections"
- Evaluate scope: is this section-level default assignments, or just a visual label?

### Completion Feel
- Brief animation on task check-off
- Section completion celebration (subtle)
- 100% overall completion moment
- "You're done! 8 tasks still unassigned" proactive nudge when personal tasks are complete

---

## Priority 2: Checklist Task Additions

Add to the `CHECKLIST_DATA` constant in code, then deploy + reset.

### New tasks to add:
- **Bring/charge batteries** — Pre-Setup or Pre-Service section. Currently `mc-1` says "Check in-ear and Mic batteries" but that's a check, not a "bring extras" task. Need an explicit "Ensure backup batteries are packed" in Pre-Setup.
- **Check batteries in drum click** — Pre-Setup or Mic Check section.
- **Add spotlight** — Camera Setup or Display Setup section. New equipment to set up.
- **Speaker/wall spacing check** — LED Wall Setup or Final Power-On. Reference: 25ft wall-to-chairs, 13ft carpet-to-chairs, 11.5ft speakers from actual wall corner.
- **Stage box wheel note** — Update `sb-1` detail/troubleshooting to include "Remove wheels for stability" or "Check if wheels should be removed."

### Existing task updates:
- **LED Wall power note** — Add "Power can be plugged in at any time" to LED Wall task detail/notes (lw-1, lw-2, or lw-3). Pending on-site verification (OPEN-ITEMS.md).
- **Camera setup updates** — Currently assumes 1 streaming camera. Real setup has 4 cameras (SDI IN 1-4). Camera Setup section needs task updates.

---

## Priority 3: Repairs Tracker Additions

Add to the `DEFAULT_REPAIRS` constant in code. Can also be added via in-app edit mode, but should be mirrored to code.

### New repair items:
- **Velcro for Resi and switch** — secure Resi encoder and switch in Wing case
- **Locktite: piano legs** — legs need threadlocker to stop loosening
- **Locktite: wall base** — wall base hardware needs threadlocker
- **Spray adhesive: wall boxes** — wall panel storage boxes need adhesive repair
- **Trailer: flooring reinforcement** — trailer floor needs reinforcement/repair
- **LED Wall cart wheels** — grease or replace wheels
- **LED Wall cart latching** — carts need to latch together during transport
- **LED Wall box stacking** — ensure no wheel boxes stacked on top of non-wheel boxes
- **Wing case re-rack and light** — re-rack equipment in Wing case, add light

---

## Priority 4: Code Quality / Robustness

### Data validation on load
- Add `validateChecklistData()` function that sanitizes data from Firebase/localStorage before use
- Clamp values, filter malformed records, return safe defaults for missing fields
- Prevents white-screen crashes from data corruption (real risk with open Firebase rules and 16 devices writing simultaneously)
- Pattern: ABC Cycle Tracker's `validateLogs()` / `validateState()`
- Effort: small — one validation function called once on load

### Schema versioning
- Add `elim3-schema-version` storage key with `runMigrations()` on load
- Detects stored version, runs upgrade logic for data format changes
- Enables non-destructive data format changes without full reset
- Becomes more valuable as persistent data (I/O config, repairs, history) grows
- Effort: small

### Error boundary with data export
- Add "Export Backup" button to existing `ErrorBoundary` component alongside "Tap to Reload"
- One-tap JSON export of all Firebase data keys for disaster recovery
- If app crashes and can't recover after reload, Sam can at least export data
- Pattern: ABC Cycle Tracker's error boundary
- Effort: small

### Remove dead weight:
- Delete `PlaceholderView` component and both placeholder nav entries (Equipment, Purchases)
- Remove placeholder routes from main app component

### Reduce duplication:
- Extract common edit toolbar pattern (used in Checklist, I/O, Repairs) into shared component
- Extract `JSON.parse(JSON.stringify())` deep clone calls into `deepClone()` utility (used 8+ times)

### Evaluate:
- Signal Flow SVGs (~200 lines of hand-crafted SVG) — could these be reference photos instead? Tradeoff: photos are easier to update but SVGs are always sharp and look professional. Code weight vs maintenance ease.

---

## Priority 5: App Features (Not Yet Built)

### Decibel Meter (Sam + Tyler only)
- Real-time SPL meter using Web Audio API `AnalyserNode` + phone microphone
- Restricted to Sam and Tyler profiles (sound check tool, not general volunteer feature)
- Displays live dB reading with rolling average
- Useful for sound check consistency ("is this level similar to last week?")
- Phone mics aren't calibrated for absolute dB — readings are relative. Can calibrate once against a real meter and store offset.
- Implementation: ~50-80 lines. New view accessible from dashboard (or only visible to Sam/Tyler). No external APIs or dependencies.
- Effort: small-medium

### Dynamic Signal Flow SVGs from I/O Data
The Signal Flow diagrams are currently ~200 lines of hardcoded SVG. The I/O Line List already contains the routing information. The SVGs could be generated dynamically from that data, so editing the I/O list automatically updates the signal flow diagrams. Significant architectural change — evaluate complexity vs benefit.

### Equipment Inventory View
Full equipment inventory with serial numbers, condition tracking, who-has-what. Currently a placeholder card on dashboard.

### Purchases View
Want/need purchase tracking list. Currently a placeholder card on dashboard.

### Teardown Checklist
Reverse of setup with teardown-specific tasks. Power sequence already has teardown guidance but no checklist.

### Setup Time Analytics
Track improvement week over week. History data exists (Settings → Setup History) but no visualization or trend analysis.

### Photo Comparison
This week's photos vs reference photos side-by-side. Useful for checking that setup matches expected configuration.

### Training Mode
Expanded instructions + video links for new volunteers. Could be a toggle that shows more detail per task.

### Live Notifications
Push notification when someone completes a task (via Firebase listeners). Would require service worker.

### Full Offline Caching
Service worker for loading app without any connectivity. Currently requires initial internet connection.

### Section-Level Assignments
Assign entire sections to a person instead of individual tasks. Assessing whether this is needed — assignments persist across resets, so the pain is mostly first-time setup.

---

## Priority 6: Physical / Process Improvements (Not App Features)

### ELIM SOP / Process Implementation
Standard operating procedures for setup, teardown, and troubleshooting. Could live as a document outside the app, or be integrated as expanded task details.

### Training Priority List
Training materials to create or organize:
- Dante Controller walkthrough (Glenn's video)
- Sunny's production overview videos
- ProPresenter operation
- ProTools operation
- Blackmagic encoder / ATEM operation

### Production Booth Consolidation
Physical workspace improvements:
- Laptop stands for production Macs (Headliner Noho stands purchased)
- Cabling organization and management
- Power distribution cleanup

### Stage Power Distribution
- 50ft 12/3 extension cords paired with 6-outlet surge strips at each endpoint
- Five positions: LED wall, PA towers (×2), stage box, production table
- Pending: PA speaker power draw measurement (OPEN-ITEMS.md) determines whether each tower needs one or two circuits

### Church-Wide Production Standards
The following should be documented as standalone reference materials (laminated cards, case lid references) independent of the app:
- Cable color code system (7 colors, documented in ELIM-PROJECT.md)
- Three-network architecture diagram
- Power sequence (on 1-10, off 10-1)
- Pelican case and Wing case packing checklists (taped inside case lids)

---

## Evaluated and Rejected

### Shazam-like Song Identification
- Requires external API (AudD ~$10/mo, ACRCloud). Adds subscription cost, API key, and network dependency.
- Shazam already exists as a free app on the same phone. Solves zero Sunday morning problems.
- **Rejected:** use Shazam app instead.

### Song Key Signature Detection
- Browser-based pitch detection (FFT/autocorrelation) works for single instruments but fails for polyphonic live worship music with drums, crowd, and multiple instruments.
- The dominant frequency in a full mix is usually bass/kick, not the tonal center.
- If someone needs to know the key, they have chord charts or can ask. This is a setlist/planning problem, not a production setup problem.
- **Rejected:** wrong tool for the problem.

---

## Notes

- Items in Priority 0 can be implemented immediately — they're data corrections or clear bug fixes
- Items in Priority 1 should be discussed before implementing — they affect the app's core UX
- Items in Priority 2-3 can be implemented independently without design decisions
- Priority 4 items improve reliability/maintainability with no user-facing change
- Priority 5 features should be built one at a time, not all at once
- Priority 6 items are tracked here so they don't get lost, but they're physical/process work
