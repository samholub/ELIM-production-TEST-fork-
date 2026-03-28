# ELIM Production App — Backlog

All planned work, organized by priority and category. Items move to CHANGELOG.md when shipped.

Last updated: 2026-03-28

---

## Priority 0: Bugs / Polish

Active bugs and visual issues that affect usability.

### ~~Dependency Warning Modal Unreadable~~ — FIXED in v5.7.0
~~The amber dependency warning overlay doesn't sufficiently obscure the text behind it. The background is `rgba(0,0,0,0.6)` which lets checklist text bleed through, making the warning text hard to read. Needs a more opaque backdrop or a solid card background.~~

### Dante I/O Fields Overflow Screen Edge
When editing Dante routing entries in the I/O Line List, the from/to input fields and protocol dropdown expand past the right edge of the screen. Needs `overflow` handling or layout adjustment on the edit row.

### Assignment Dropdown: White Background on Mobile
The `<select>` dropdown opens with native OS white styling, which clashes with the dark Warm Night theme. Needs a custom dropdown component that matches the app's color system — dark background, cream text, properly sized tap targets.

### Assignment Dropdown: Too Small to Read
The dropdown is compact (14px, tight padding). When tapped, the native picker shows names but the in-task display is cramped. Should expand into a larger, more readable selector when activated — bigger text, more padding, clearer name display.

### Populated Notes Hidden in Expand
When a task has a saved note, it's only visible after expanding the task (📌 icon is the only hint). Populated notes should surface above the fold — display under the assignee name in a compact, styled format matching the existing note aesthetic. The expand area retains the edit functionality.

### Visible Scrollbar
The right-side scrollbar is visible on the app. Should be hidden via CSS (`scrollbar-width: none` / `::-webkit-scrollbar { display: none }`).

### Scroll Position Resets on Navigation
When navigating from a screen (e.g., Checklist → task detail or Dashboard → My Tasks) and returning via back button or swipe, the scroll position resets to the top. Should preserve and restore scroll position for each view.

---

## Priority 1: UX / Navigation Rethink

These are strategic decisions about how the app should feel and flow. Should be discussed before implementing.

### ~~Navigation Model~~ — DONE in v5.7.0 (Option A selected)
~~The dashboard is designed for a team-lead perspective (overall progress, crew status), but most users are individual crew members who just want their task list. The current flow is: Open → Dashboard → Tap "Your Tasks" → See tasks. That's one unnecessary hop.~~

App now opens directly to My Tasks. Dashboard accessible via back arrow.

### Reduce Visual Noise
- ~~**Remove Equipment placeholder** — nav card leads to "Coming soon" screen. Dead weight.~~ DONE in v5.7.0 — moved to Settings
- ~~**Remove Purchases placeholder** — same. Add both back when they're actually built.~~ DONE in v5.7.0 — moved to Settings
- ~~**Nav grid:** 3×2 (6 cards, 2 dead) → 2×2 (4 working cards). Every card does something.~~ DONE in v5.7.0
- ~~**Hide checklist search bar** behind a search icon toggle — 59 tasks across 15 sections is scrollable in 3 seconds. Search bar always occupies space for a problem that barely exists.~~ DONE in v5.7.0
- **Move checklist edit mode to Settings** (or behind long-press) — admin function that 15 of 16 team members will never use. Same for I/O edit and Repairs edit. *(Partially addressed: edit mode now restricted to Sam via `currentUser === "Sam"` check)*
- ~~**Remove "Last setup" stat from dashboard hero** — interesting for post-mortem, not during setup. Already lives in Settings → Setup History.~~ DONE in v5.7.0

### Task Card Density
- Show assignee as plain text (not dropdown) by default — during setup, assignments are already set
- Only show assignment dropdown when task is unassigned or in an explicit "assign" mode
- Evaluate whether the expand arrow, note indicator, and power badge density is right
- **Add "All" assignment option** — for tasks everyone does (prayer, unloading, etc.). Currently these either stay unassigned or get assigned to one person arbitrarily.

### Completion Feel
- Brief animation on task check-off
- Section completion celebration (subtle)
- 100% overall completion moment
- "You're done! 8 tasks still unassigned" proactive nudge when personal tasks are complete

---

## Priority 2: Checklist Task Additions

These should be added to the `CHECKLIST_DATA` constant in code, then deployed + reset.

### New tasks to add:
- **Bring/charge batteries** — Pre-Setup or Pre-Service section. Currently `mc-1` says "Check in-ear and Mic batteries" but that's a check, not a "bring extras" task. Need an explicit "Ensure backup batteries are packed" in Pre-Setup.
- **Speaker/wall spacing check** — LED Wall Setup or Final Power-On. Verify physical spacing between speakers and LED wall panels.
- **Stage box wheel note** — Update `sb-1` detail/troubleshooting to include "Remove wheels for stability" or "Check if wheels should be removed."

### Existing task updates:
- **LED Wall power note** — Add "Power can be plugged in at any time" to LED Wall task detail/notes (lw-1, lw-2, or lw-3)

---

## Priority 3: Repairs Tracker Additions

These should be added to the `DEFAULT_REPAIRS` constant in code. Can also be added via in-app edit mode, but should be mirrored to code.

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

## Priority 4: Code Quality / Simplification

### Remove dead weight:
- ~~Delete `PlaceholderView` component and both placeholder nav entries (Equipment, Purchases)~~ Partially done in v5.7.0 — placeholder cards moved from dashboard to Settings; `PlaceholderView` component can still be deleted once Settings rows replace it
- Remove placeholder routes from main app component

### Reduce duplication:
- Extract common edit toolbar pattern (used in Checklist, I/O, Repairs) into shared component
- Extract `JSON.parse(JSON.stringify())` deep clone calls into `deepClone()` utility (used 8+ times)

### Evaluate:
- Signal Flow SVGs (~200 lines of hand-crafted SVG) — could these be reference photos instead? Tradeoff: photos are easier to update but SVGs are always sharp and look professional. Code weight vs maintenance ease.

---

## Priority 5: App Features (Not Yet Built)

### Dynamic Signal Flow SVGs from I/O Data
The Signal Flow diagrams are currently ~200 lines of hardcoded SVG. The I/O Line List already contains the routing information (inputs, outputs, Dante paths). The SVGs could be generated dynamically from that data, so editing the I/O list automatically updates the signal flow diagrams. This would eliminate the need to maintain two sources of truth for the same routing information. Significant architectural change — evaluate complexity vs benefit.

### Equipment Inventory View
Full equipment inventory with serial numbers, condition tracking, who-has-what. Currently a placeholder row in Settings.

### Purchases View
Want/need purchase tracking list. Currently a placeholder row in Settings.

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

These are real-world improvements that the app should track or support but aren't code changes.

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
- Laptop stands for production Macs
- Cabling organization and management
- Power distribution cleanup
- Desk slide-out drawers for equipment

### Stage Box Height
Check whether stage box height is appropriate or needs risers/adjustment.

---

## Notes

- Items in Priority 1-2 should be discussed before implementing — they affect the app's core UX
- Items in Priority 3-4 can be implemented independently without design decisions
- Priority 5 features should be built one at a time, not all at once
- Priority 6 items are tracked here so they don't get lost, but they're physical/process work
