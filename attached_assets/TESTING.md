# Pre-Deployment Testing Checklist

Quick validation before taking this into a real Sunday setup.

## Reported Bugs (pending fix)

### Warning Modal Readability — FIXED in v5.7.0
- [x] **Opaque backdrop** — dependency warning modal background fully obscures text behind it
- [x] **Warning text readable** — amber warning text, button labels clearly visible against background

### I/O Dante Edit Overflow — FIXED in v5.8.0
- [x] **Fields stay on screen** — editing Dante routing from/to fields doesn't overflow past right edge
- [x] **All controls accessible** — protocol dropdown and delete button visible without horizontal scroll

### Assignment Dropdown Styling — FIXED in v5.8.0
- [x] **Dark theme match** — custom AssignDropdown component with `#1C1A18` background
- [x] **Readable size** — 15px font, 10px 14px padding on each option
- [x] **Expanded view** — dropdown opens below button with full team roster, click-outside-to-close

### Notes Display — FIXED in v5.8.0
- [x] **Populated notes visible** — tasks with saved notes show preview below assignee (hidden on checked tasks)
- [x] **Detail text surfaced** — `item.detail` also shown above the fold, hidden on checked tasks
- [x] **Edit still in expand** — expanding the task still shows the full note editor

### Scrollbar Hidden — FIXED in v5.8.0
- [x] **No visible scrollbar** — CSS wildcard rules hide scrollbar on all containers

### Scroll Position Preservation — FIXED in v5.8.0
- [x] **Checklist position preserved** — scroll position saved per view on navigate, restored via requestAnimationFrame
- [x] **Dashboard position preserved** — viewStack navigation preserves scroll for all views
- [ ] **Works with swipe-back** — swipe-back gesture preserves scroll position same as back button

### "All" Assignment Option
- [ ] **"All" in dropdown** — assignment dropdown includes "All" option
- [ ] **"All" styling** — tasks assigned to "All" have appropriate visual treatment (not orange "yours" unless you're logged in)
- [ ] **"All" in My Tasks** — tasks assigned to "All" appear in everyone's My Tasks view

---

## v5.8.0 Field Validation

### Custom Assignment Dropdown
- [ ] **Tap to open** — tapping assignee button opens dark-themed dropdown list
- [ ] **Tap to select** — tapping a name assigns and closes dropdown
- [ ] **Tap outside to close** — tapping anywhere outside the dropdown dismisses it
- [ ] **Unassign option** — "Unassign" option appears only when a name is already assigned
- [ ] **Current selection marked** — currently assigned name shows with ✓ prefix and accent color
- [ ] **No z-index clipping** — dropdown list is not clipped by parent card containers
- [ ] **Works in My Tasks view** — dropdown functions same in My Tasks as in full Checklist

### Detail + Note Surfacing
- [ ] **Detail visible without expanding** — tasks with `item.detail` show it below assignee row
- [ ] **Note preview visible** — tasks with saved notes show 📌 preview below detail
- [ ] **2-line clamp** — long detail/note text truncates with ellipsis at 2 lines
- [ ] **Hidden when checked** — both detail and note preview disappear when task is checked off
- [ ] **Note preview hides on expand** — expanding a task hides the preview (full editor shown instead)
- [ ] **Detail stays on expand** — detail text remains visible when task is expanded

### Scroll Position
- [ ] **Preserved on navigate** — scroll to middle of checklist, tap back, re-enter → same position
- [ ] **Preserved on swipe-back** — swipe gesture preserves position same as back button
- [ ] **Fresh on new view** — navigating to a view for the first time starts at top

### View Stack Navigation
- [ ] **Back button works** — tapping ← goes to previous view, not always dashboard
- [ ] **Deep navigation** — Dashboard → Checklist → Crew Tasks → back → back returns to Dashboard
- [ ] **Swipe back matches button** — swipe gesture navigates same as ← button

---

## v5.7.0 Field Validation

### Data Integrity — Firebase Write Retry
- [ ] **Offline write preserved** — turn on airplane mode, check a box, turn off airplane mode; checkbox should persist (not revert)
- [ ] **No stale overwrite** — check a box on Phone A while Phone B is offline; bring Phone B online; Phone A's change should appear on Phone B (not be overwritten)
- [ ] **Retry fires** — check a box with WiFi off, then reconnect within 10 seconds; the write should reach Firebase after reconnect

### Data Integrity — Task Deletion Cleanup
- [ ] **Confirm dialog** — in edit mode (as Sam), tapping ✕ shows "Delete this task? This cannot be undone." confirmation
- [ ] **Cancel preserves task** — tapping Cancel leaves the task intact
- [ ] **Orphan cleanup** — delete a previously-checked, assigned task with notes; verify `checks`, `assignments`, `taskNotes` no longer contain that task ID (check Firebase console)
- [ ] **Progress accurate** — delete 2 checked tasks; dashboard percentage should not exceed 100%

### Data Integrity — Atomic Reset
- [ ] **Normal reset works** — Settings → Reset Checklist clears checkboxes, timer, completions; preserves assignments, notes, photos
- [ ] **History saved** — after reset, new entry appears in Setup History
- [ ] **Interrupted reset recovers** — (hard to test manually) if `elim3-pending-reset` exists in Firebase on load, app clears checks/completions/timer automatically

### Volunteer Safety — Edit Mode Gate
- [ ] **Sam sees edit button** — log in as Sam, go to Checklist; ✏️ button visible next to search
- [ ] **Others don't see it** — log in as any other name; ✏️ button NOT visible in Checklist
- [ ] **Edit mode still works** — as Sam, enter edit mode; can add, edit, and delete tasks

### Volunteer Safety — Roster Removal
- [ ] **Confirmation dialog** — in Settings, removing a member shows confirm with "assignments will be cleared" message
- [ ] **Assignments cleared** — remove a member who has assigned tasks; their tasks should become unassigned
- [ ] **Member gone** — removed member no longer appears in login screen or assignment dropdowns

### Volunteer Safety — Timer Threshold
- [ ] **First box doesn't start timer** — check one box; timer should NOT auto-start
- [ ] **Second box starts timer** — check a second box; timer should auto-start
- [ ] **Uncheck doesn't stop** — uncheck one of the two boxes; timer should keep running

### Error Handling
- [ ] **Error boundary renders** — (hard to test manually) if a component throws, should see "Something went wrong — Tap to Reload" instead of white screen
- [ ] **Reload button works** — tapping "Tap to Reload" refreshes the page

### Performance
- [ ] **App loads without waiting for photos** — on slow connection, app should be interactive before photos finish loading
- [ ] **No visible jank** — scroll through full checklist with timer running; should be smooth on mobile

### JSON Null Guard
- [ ] **Null doesn't crash** — (test via Firebase console) set `elim3-checks` to `null` in Firebase; app should not white-screen; should fall back to empty state

---

## v5.6.0 Field Validation

### Repairs View — CRUD & Search
- [ ] **Search visible** — search input is always visible at top of Repairs view (not hidden behind edit mode)
- [ ] **Search filters by name** — type part of an item name (e.g. "LED"), only matching items shown
- [ ] **Search filters by issue** — type part of an issue description, matching items shown
- [ ] **Search case-insensitive** — "led", "LED", "Led" all return the same results
- [ ] **Item count updates** — count line below controls reflects filtered count (e.g. "2 items of 6")
- [ ] **Edit mode toggle** — "✏️ Edit Repairs" button enters edit mode; "✓ Done Editing" exits
- [ ] **Inline name edit** — in edit mode, item name field is editable as text input
- [ ] **Inline issue edit** — in edit mode, issue description field is editable
- [ ] **Priority dropdown** — in edit mode, priority dropdown present; changing it updates badge color and label immediately
- [ ] **Delete item** — ✕ button removes item immediately with no confirmation dialog
- [ ] **Add repair button visible** — "+ Add Repair" appears at bottom of list in edit mode
- [ ] **Add repair hidden during search** — "+ Add Repair" not shown when search input has text
- [ ] **Add repair works** — tap "+ Add Repair", new item appears with default name/issue/medium priority
- [ ] **New item editable** — newly added item's name and issue fields are immediately editable
- [ ] **Reset button** — "Reset" button visible next to edit toggle
- [ ] **Reset confirm flow** — tap "Reset" → "Confirm Reset" + "✕" buttons appear
- [ ] **Cancel reset** — tap "✕" cancels and returns to single Reset button
- [ ] **Reset works** — tap "Confirm Reset", list restores to original 6 items, exits edit mode
- [ ] **Edits persist** — edit an item name, exit edit mode, refresh page — edit persists
- [ ] **Cross-device sync** — edit an item on Phone A, change appears on Phone B within 1-2 seconds
- [ ] **Priority colors correct** — Urgent = red badge, Soon = orange/amber badge, Low = green badge

### Checklist — Search Toolbar
- [ ] **Search field visible** — "Search tasks..." input visible between progress bar and section list
- [ ] **Edit button visible** — ✏️ button to the right of search field
- [ ] **Search filters tasks** — type a keyword, only tasks containing that text shown
- [ ] **Empty sections hidden** — sections with no matching tasks collapse/hide during search
- [ ] **Sections auto-expand** — sections containing matching tasks expand automatically
- [ ] **Case-insensitive** — search works regardless of capitalization
- [ ] **Clear search** — clearing the search field returns to normal view
- [ ] **Normal expand behavior restored** — after clearing search, manual expand/collapse works normally

### Checklist — Edit Mode
- [ ] **Edit mode toggle** — tap ✏️ to enter edit mode; tap "✓ Done" to exit
- [ ] **Sections expand in edit mode** — entering edit mode auto-expands all sections
- [ ] **Delete task** — ✕ button appears to the left of each task card in edit mode; tap removes task immediately
- [ ] **Inline task edit** — tap a task's text in edit mode, inline input appears pre-filled with task text
- [ ] **Enter saves edit** — pressing Enter while editing task text saves the change
- [ ] **Blur saves edit** — tapping away from the edit input saves the change
- [ ] **Escape cancels edit** — pressing Escape returns to original text without saving
- [ ] **Add task button** — "+ Add Task" button appears at bottom of each expanded section in edit mode
- [ ] **Add task hidden during search** — "+ Add Task" not shown when search input has text
- [ ] **Add task flow** — tap "+ Add Task", text input appears with "Add" and "✕" buttons
- [ ] **Add task saves** — type a name and tap "Add" or press Enter; new task appears in section
- [ ] **Add task cancel** — tap "✕" dismisses input without adding anything
- [ ] **New task appears** — newly added task is visible in the section immediately
- [ ] **Edit mode exit clears state** — exiting edit mode dismisses any open inline input or add form

### Checklist Data Persistence & Migration
- [ ] **Existing checks survive** — all previously checked boxes remain checked after upgrade to v5.6.0
- [ ] **Existing assignments survive** — task assignments all intact; no orphaned assignments
- [ ] **Existing notes survive** — per-task notes all intact
- [ ] **Existing photos survive** — photo attachments all intact
- [ ] **Added task persists** — add a task, refresh page — task still there
- [ ] **Added task syncs** — add a task on Phone A, appears on Phone B within 1-2 seconds
- [ ] **Deleted task persists** — delete a task, refresh — task gone
- [ ] **Dashboard counts correct** — total task count on dashboard hero updates if tasks are added/removed
- [ ] **My Tasks unaffected** — My Tasks only shows assigned tasks; custom unassigned tasks don't appear

### Audio Signal Flow SVG
- [ ] **Tyler's Mac in left sidebar** — Tyler's Mac box appears on the left side of the diagram, branching from Wing (mirrors WiFi Router on right side)
- [ ] **No crossing lines** — Tyler's Mac branch doesn't cross through Omar/Mitch Mac or IEM Transmitter boxes
- [ ] **Bianca IEM vertical flow** — Bianca IEM box appears directly below IEM Transmitters in clean vertical column
- [ ] **No overlap** — all boxes have clear separation, no overlapping text or boxes
- [ ] **All labels visible** — Omar/Mitch Mac, Tyler's Mac, Bianca IEM, ATEM Mini Pro all labeled correctly
- [ ] **Smaller viewBox** — diagram is less tall than before (660px instead of 780px), less whitespace
- [ ] **No clipping** — all boxes and labels visible on iPhone SE screen width
- [ ] **Scrolling smooth** — can scroll through all three SVG sections without jank

### Swipe-Back Exclusion Zone
- [ ] **Back button tap works** — tapping the ← back button in the header always navigates back, no swipe interference
- [ ] **Swipe still works elsewhere** — swipe-back gesture triggers normally when starting from middle of screen
- [ ] **Header area excluded** — starting a swipe in the top-left corner (where ← lives) does NOT trigger swipe gesture
- [ ] **Right-side header unaffected** — swiping from title or right side of header still works normally

### Progress Bar / Header Spacing
- [ ] **No clipping** — progress bar in checklist view not hidden behind fixed header when at top of scroll
- [ ] **Correct spacing** — appropriate gap between header and first content on all views
- [ ] **Smooth scroll** — scrolling up doesn't cause progress bar to clip or disappear under header

---

## v5.5.2 Field Validation

### Three Macs & Equipment Names
- [ ] **Tyler's task (ws-5b) visible** — "Connect Tyler's ProTools Mac to Wing (USB-C direct)" appears in Wing Soundboard Setup
- [ ] **ws-5b detail shows** — expanded detail says Tyler's Mac is NOT through the Dante switch
- [ ] **dn-5 names Omar/Mitch** — task references "Omar/Mitch's ProTools Mac"
- [ ] **dn-6 names Glenn** — task references "Glenn's ProPresenter Mac"
- [ ] **pt-1 references both Macs** — "Power on both ProTools Macs (Tyler's + Omar/Mitch's)"
- [ ] **pp-1 names Glenn** — "Power on Glenn's ProPresenter Mac"
- [ ] **ds-2 names Glenn** — "Plug Confidence Monitor into Glenn's ProPresenter Mac"
- [ ] **cs-2 names Glenn** — "Plug ATEM USB cable into Glenn's ProPresenter Mac"

### Camera Setup Order
- [ ] **cs-1b first** — "Setup ATEM Mini Pro" is the first task in Camera Setup section
- [ ] **cs-2 second** — "Plug ATEM USB cable into Glenn's ProPresenter Mac"
- [ ] **cs-1 last** — "Plug in camera power and run SDI/HDMI Extender to ATEM Mini Pro"
- [ ] **cs-1b detail shows** — Resi Encoder note visible when task is expanded

### WiFi Router Dependency (ws-4)
- [ ] **Amber warning triggers** — checking ws-4 "Power on WiFi Router" before ws-2 or ws-3 shows amber modal
- [ ] **Continue/Cancel works** — can proceed past warning or cancel
- [ ] **No warning if prerequisites done** — checking ws-4 after ws-2 + ws-3 are complete has no warning
- [ ] **ws-3 text correct** — says "Connect and setup WiFi Router with Wing" (not "then power on")

### I/O Line List
- [ ] **No "Aux 3 Pastor Monitor"** — Aux 3 row gone from Outputs tab
- [ ] **Bianca's IEM visible** — "Wing XLR Out 8 → Bianca's IEM" in Outputs
- [ ] **Dante routing updated** — AVIO connects to ATEM Mini Pro (Analog Audio IN Ch 1-2), not Pastor Monitor
- [ ] **Tyler's Mac route noted** — USB-C direct route visible in Dante routes

### Section Numbers
- [ ] **Numbers in Checklist** — sections read "1. Pre-Setup", "2. LED Wall Setup", "3. Stage Box Setup", etc.
- [ ] **Numbers in My Tasks** — My Tasks grouped section headers also numbered
- [ ] **Numbers in Crew Tasks** — same numbering when viewing a crew member's tasks

### Signal Flow SVGs
- [ ] **Audio: no Pastor Monitor box** — removed entirely, no arrow pointing to it
- [ ] **Audio: Tyler's Mac** — shown branching from Wing via USB-C (separate from switch)
- [ ] **Audio: ATEM Mini Pro** — shown as Dante AVIO destination (Analog Audio IN Ch 1-2)
- [ ] **Audio: Bianca IEM** — shown from Wing XLR Out 8
- [ ] **Audio: Omar/Mitch Mac labeled** — "Omar/Mitch Mac (Stream Mix)"
- [ ] **Audio: no clipping** — all boxes visible on narrow screens
- [ ] **Network: Glenn's Mac labeled** — "Glenn's Mac (ProPresenter)"
- [ ] **Network: Omar/Mitch Mac labeled** — "Omar/Mitch Mac (Stream Mix)"
- [ ] **Network: no Pastor Monitor** — ATEM Mini Pro shown instead
- [ ] **Network: Tyler note** — small text note that Tyler's Mac is USB-C direct, not on Dante

### Power Sequence View
- [ ] **Item 5** — "ProTools Macs (Tyler + Omar/Mitch)"
- [ ] **Item 7** — "Glenn's ProPresenter Mac"

---

## v5.5.1 Field Validation

### Login & Header
- [ ] **Login page doesn't scroll** — user selection screen is fixed height, no scrolling
- [ ] **Header padding reduced** — header sits slightly higher/tighter than v5.5.0
- [ ] **Back button hit area** — tapping anywhere on ← arrow + section title text navigates back (not just the arrow)

### RepairsView
- [ ] **Repairs has CRUD** — edit mode, search, add/remove/edit all functional (v5.6.0 upgrade)
- [ ] **Priority badges color-coded** — High = red, Medium = orange, Low = green

### I/O Duplicate Detection (recovered from v5.3.2)
- [ ] **Inputs auto-sort** — after editing a channel number and tapping away, inputs reorder numerically
- [ ] **Duplicate inputs highlighted red** — if two inputs share a channel number, both rows show red styling
- [ ] **Duplicate outputs highlighted red** — same behavior on Outputs tab

### Dependency Warnings (recovered from v5.4.0)
- [ ] **dn-3 warns** — checking "Power on the Switch" before dn-1 is done shows amber modal
- [ ] **pt-3 warns** — checking "Open ProTools Session" before pt-2 is done shows amber modal
- [ ] **Amber modal style** — warning has orange/amber border, not red

---

## v5.5.0 Field Validation

### Signal Flow Diagrams
- [ ] **All three diagrams render** — Audio (orange), Video (purple), Network (green) sections visible
- [ ] **No clipping** — all boxes and labels visible on iPhone SE, standard, and Pro Max
- [ ] **SVG text readable** — equipment names and connection labels legible at all screen widths
- [ ] **Labels don't overlap** — connection type labels (e.g. "AES50 (CAT6)") positioned to right of arrows, not overlapping
- [ ] **Boxes don't overlap** — minimum clearance between equipment boxes at fork points
- [ ] **Scrolling smooth** — smooth scroll through all three sections
- [ ] **Color coding correct** — orange section header for Audio, purple for Video, green for Network
- [ ] **WiFi Router sidebar** — small sidebar box connected to Behringer Wing in Audio chain
- [ ] **Live Streaming callout** — green bordered box at bottom of Video chain
- [ ] **Equipment names match** — all names match actual equipment (Midas DL251, ATEM Mini Pro, etc.)

### Header Positioning
- [ ] **No gap above header** — header background extends behind iOS status bar (no visible gap)
- [ ] **No gap below header** — content starts immediately below header, no double spacing
- [ ] **Header stays fixed** — header remains at top during scroll on all views
- [ ] **Blur effect works** — semi-transparent header with backdrop blur during scroll
- [ ] **Safe area correct** — works on both notched iPhones (14/15/16) and non-notched devices

### Swipe-Back Gesture
- [ ] **Follow-finger tracking** — content moves with finger in real-time (no lag)
- [ ] **Completing swipe** — dragging >35% screen width navigates back with slide-out animation
- [ ] **Partial swipe springs back** — dragging <35% springs content back to original position
- [ ] **Opacity effect** — content slightly fades during drag
- [ ] **No interference** — swipe doesn't trigger on `[data-no-swipe]` elements (photo scroll areas)
- [ ] **Works from Signal Flow** — can swipe back from Signal Flow to dashboard
- [ ] **Back button still works** — ← button in header still navigates back

### Minor Updates
- [ ] **Power Sequence item 6** — says "Confidence Monitor" (not "Monitor")

---

## Critical Path (do these first)

- [ ] **Open https://elim-production.web.app on phone** — does it load without errors?
- [ ] **Select a user** — does the name selection screen work with SVG logo centered?
- [ ] **Check a box** — does it save and persist after page refresh?
- [ ] **Assign a task** — does the dropdown work, does assignment persist?
- [ ] **Switch users** — does a different user see the same checkboxes?
- [ ] **Smart expand** — assign tasks to yourself in 2-3 sections. Close and reopen checklist. Do only those sections auto-expand?
- [ ] **Manual override** — collapse an auto-expanded section. Does it stay collapsed?

## Cross-Device Sync (critical — test with two phones)

- [ ] **Real-time checkbox sync** — check a box on Phone A, does it appear on Phone B within 1-2 seconds?
- [ ] **Assignment sync** — assign a task on Phone A, does Phone B see the assignment?
- [ ] **Notes sync** — add a note on one device, does it appear on the other?
- [ ] **Photo sync** — take a photo on one device, does it appear on the other? (may be slow due to base64 size)
- [ ] **Crew status dashboard** — do progress bars update across devices?
- [ ] **Timer sync** — start timer on Phone A, does Phone B show the same running time? Pause on Phone B, does Phone A stop?
- [ ] **Countdown sync** — set a 45-min countdown on Phone A. Does Phone B show the same countdown with matching colors?
- [ ] **Offline fallback** — turn off WiFi on one phone, check a box, turn WiFi back on. Does it sync?

## Editable I/O Line List (v5.3.1)

- [ ] **View mode default** — I/O Line List opens in read-only view (same as v5.3.0)?
- [ ] **Edit button** — "✏️ Edit I/O" button visible below tabs?
- [ ] **Enter edit mode** — tap "✏️ Edit I/O", inline inputs appear for all fields?
- [ ] **Channel number edits** — can edit channel numbers (text input, no up/down arrows), accepts any text like "L/R Main"?
- [ ] **Label edits** — can edit channel labels, changes save immediately?
- [ ] **Notes edits** — can edit notes field?
- [ ] **Type dropdown** — inputs tab shows type dropdown (XLR, DI/XLR, Line, 1/4")?
- [ ] **Protocol dropdown** — Dante tab shows protocol dropdown (Dante, Analog, AES50, USB)?
- [ ] **Add channel** — "+ Add Input/Output/Route" button appears at bottom in edit mode?
- [ ] **Add works** — tap + button, new blank channel appears in list?
- [ ] **Remove channel** — × button on each row in edit mode?
- [ ] **Remove works** — tap ×, channel disappears immediately (no confirmation)?
- [ ] **Exit edit mode** — tap "✓ Done Editing", inputs become read-only text again?
- [ ] **Persistence** — edit a label, exit edit mode, refresh page — edit persists?
- [ ] **Reset button** — "Reset" button visible next to "Edit I/O"?
- [ ] **Reset confirm** — tap Reset → "Confirm Reset" + "×" buttons appear?
- [ ] **Reset works** — tap "Confirm Reset", all channels restore to defaults?
- [ ] **Cross-device sync** — edit I/O on Phone A, changes appear on Phone B within 1-2 seconds?
- [ ] **All 3 tabs editable** — Inputs, Outputs, and Dante tabs all support editing?

## I/O Auto-Sort & Duplicate Prevention (v5.3.2)

- [ ] **Auto-sort on blur** — edit an input channel number to "5", blur the field, does the list re-sort with channel 5 in numeric position?
- [ ] **Numeric sorting** — create channels 1, 10, 2, 3, 20. Do they sort as 1, 2, 3, 10, 20 (not 1, 10, 2, 20, 3)?
- [ ] **Non-numeric at end** — create channels 1, 2, "L/R Main", 3. Does "L/R Main" sort to the end?
- [ ] **Sort preserves data** — after auto-sort, verify all labels, notes, types stay with correct channel numbers?
- [ ] **Duplicate detection inputs** — change channel 5 to match channel 3's number. Does it show red background immediately?
- [ ] **Duplicate detection outputs** — change "Aux 1" to match "L/R Main". Does it show red background?
- [ ] **Duplicate styling** — red background on channel number badge, red text color?
- [ ] **No duplicate false positives** — editing a channel number to a unique value shows normal styling?
- [ ] **Dante unaffected** — Dante tab has no auto-sort or duplicate checks (no channel numbers)?

## Task Organization & Dependency Warnings (v5.4.0)

### Task Reorders
- [ ] **Pre-Setup order** — ps-3 (Unload), ps-1 (Pray), ps-2 (Assign)?
- [ ] **Stage Box order** — sb-1, sb-4, sb-6, sb-5, sb-2, sb-3?

### Task Text Updates
- [ ] **ds-3** — says "Power on Confidence Monitor"?
- [ ] **pp-5** — says "Test Pastor / Mami media videos"?
- [ ] **psc-1** — says "Transpose button set correctly for keyboardist (Michael-ON / Sunny-OFF)"?

### Dependency Warnings
- [ ] **dn-3 warning** — try to check dn-3 without checking dn-1 first, warning appears?
- [ ] **pt-3 warning** — try to check pt-3 without checking pt-2 first, warning appears?
- [ ] **Warning modal** — amber colored, shows dependency task name, has Continue and Cancel buttons?
- [ ] **Continue works** — clicking Continue checks the box despite warning?
- [ ] **Cancel works** — clicking Cancel closes modal without checking box?
- [ ] **No warning when dependency met** — check dn-1 first, then dn-3 checks without warning?
- [ ] **Other tasks unaffected** — no warnings on tasks without dependsOn property?

## Crew Focused Views (v5.3.0)

- [ ] **Tap crew name** — on dashboard, tap a crew member in Crew Status. Does it open "{Name}'s Tasks" focused view?
- [ ] **Focused content** — only that person's assigned tasks shown, grouped by section?
- [ ] **Timer in header** — countdown/timer visible in sticky header of focused view?
- [ ] **Your tasks highlighted** — if viewing someone else's tasks, tasks also assigned to you still get orange highlight?
- [ ] **Empty state** — if a crew member has no tasks, does it show "No tasks assigned to {Name}"?
- [ ] **Back navigation** — back button and swipe both return to dashboard?

## Unassigned Task Awareness (v5.3.0)

- [ ] **Unassigned row** — "Unassigned tasks" row visible at bottom of Crew Status with progress bar?
- [ ] **Visual separation** — divider line and dashed border distinguish it from crew rows above?
- [ ] **Tap unassigned** — opens "Unassigned Tasks" focused view showing only unassigned tasks?
- [ ] **Unassigned card styling** — unassigned task cards have dashed border, lighter background, italic dimmed text?
- [ ] **Section header count** — sections with unassigned tasks show "{N} unassigned" in italic?
- [ ] **All-unassigned section** — when all incomplete tasks in a section are unassigned, title goes italic/muted with dimmed icon?
- [ ] **Assign clears styling** — assigning a previously unassigned task makes it look normal (solid border, full opacity)?
- [ ] **All assigned state** — when every task has an owner, unassigned row shows ✓ or focused view says "All tasks are assigned"?

## Dashboard Hero (v5.3.0)

- [ ] **Stacked layout** — percentage on top, timer below, separated by divider line?
- [ ] **Progress tap** — tapping progress zone (percentage, subtitle, progress bar) navigates to full checklist?
- [ ] **No auto-expand** — checklist opens with all sections collapsed when navigated from hero?
- [ ] **Timer isolated** — tapping timer controls doesn't accidentally navigate to checklist?
- [ ] **Timer picker** — X button in top-right of picker panel, presets and custom input work?
- [ ] **History line** — "Last: {date} — {pct}% in {time}" visible below timer?

## Header (v5.3.0)

- [ ] **Layout** — logo left, centered username with ▾, ⚙️ gear right?
- [ ] **Tap name** — tapping centered username returns to user selection screen?
- [ ] **Tap gear** — tapping ⚙️ navigates to Settings?
- [ ] **No "Switch" button** — orange "Switch" button removed?
- [ ] **No "Signed in as"** — label removed, just the name?

## Settings (v5.3.0)

- [ ] **Reset at top** — Reset Checklist section is first thing in Settings?
- [ ] **Two-tap reset** — "🔄 Reset Checklist" → "Clear all progress?" → "Yes, Reset" / "Cancel"?
- [ ] **Reset preserves** — after reset, assignments, notes, photos, roster still intact?
- [ ] **History saved** — after reset, new entry appears in Setup History?
- [ ] **52 sessions** — history stores up to 52 entries?
- [ ] **Order** — Reset → Team Roster → Setup History → About?
- [ ] **Gear access** — Settings accessible via ⚙️ in header (no dashboard card)?

## Section Header Separators (v5.3.0)

- [ ] **Collapsed sections** — thin line visible between collapsed section headers?
- [ ] **Expanded sections** — line disappears when section is expanded?
- [ ] **Consistent** — all collapsed sections show the separator?

## Power Sequence Badge (v5.3.0)

- [ ] **Right side** — ⚡ badge with number on right side of task card (not inline with text)?
- [ ] **Size** — badge is clearly visible (16px font)?
- [ ] **Centered** — badge vertically centered in the card?
- [ ] **Correct tasks** — only tasks with power sequence numbers show the badge?

## Navigation Cards (v5.3.0)

- [ ] **3×2 grid** — Power Sequence, I/O Line List, Signal Flow, Equipment, Repairs, Purchases?
- [ ] **No Checklist card** — checklist access only via hero "View all →"?
- [ ] **No Settings card** — settings access only via header ⚙️?

## Back Swipe (v5.6.0 — updated)

- [ ] **Follow-finger** — content follows finger position in real-time during swipe?
- [ ] **Complete swipe** — dragging past 35% of screen width navigates back with slide-out animation?
- [ ] **Snap back** — dragging less than 35% springs content back to original position?
- [ ] **Header zone excluded** — swipe starting in top-left corner (top 80px, left 120px) does NOT trigger gesture?
- [ ] **Back button reliable** — ← button in header always navigates back, never intercepted by swipe?
- [ ] **Photo areas excluded** — swiping in photo scroll areas doesn't trigger back navigation?
- [ ] **Mid-screen swipe works** — swipe-back still triggers normally from center or bottom of screen?

## Desktop Browser (v5.3.0)

- [ ] **Scroll wheel** — mouse scroll wheel works in desktop browser?
- [ ] **Horizontal scroll blocked** — no unwanted horizontal scrolling?

## SVG Brand Mark

- [ ] **Login screen** — SVG logo (76px) centered on screen, sharp chevrons pointing right, block E, circle with breathing room?
- [ ] **Loading state** — SVG logo (56px) centered while app loads?
- [ ] **Dashboard header** — SVG logo (32px) left-aligned, recognizable at small size?
- [ ] **Sharp corners** — chevron tips and E corners are all sharp (no rounded edges)?
- [ ] **Consistent** — all 3 logo sizes look the same, no visual differences?

## Countdown Timer

- [ ] **No target set** — dashboard shows count-up time with "⏱ Set Timer" link?
- [ ] **Timer picker** — tapping "Set Timer" opens inline picker with 30/45/60/90 presets + custom input?
- [ ] **Set preset** — tap "60m", timer area shows "60:00" countdown?
- [ ] **Color green** — countdown above 50% remaining shows green text?
- [ ] **Color yellow** — countdown between 25-50% remaining shows yellow/amber text?
- [ ] **Color red** — countdown below 25% remaining shows red text?
- [ ] **Auto-pause at zero** — when countdown reaches 0:00, timer stops and shows "⏰ Time!"?
- [ ] **Change target** — "change" link opens picker to set a new target?
- [ ] **Remove countdown** — "Remove countdown (use count-up)" option reverts to elapsed timer?
- [ ] **Custom input** — can enter a custom number (e.g., 75 minutes) and it works?
- [ ] **Checklist header** — countdown with matching colors visible in sticky header on Checklist view?
- [ ] **My Tasks header** — countdown with matching colors visible in sticky header on My Tasks view?
- [ ] **Persists** — set countdown, refresh page — countdown resumes with correct remaining time?
- [ ] **Cross-device** — set countdown on Phone A, Phone B shows same countdown?

## Warm Night Theme

- [ ] **4-state colors** — your tasks orange border, completed green, unassigned dashed/dim, others neutral?
- [ ] **No per-person colors** — checklist and dashboard use only orange/green/dashed/neutral (no indigo, pink, cyan)?
- [ ] **Background** — dark warm tone (#111010)?
- [ ] **Warm text** — cream-colored text, not cool grey?
- [ ] **No team strip** — checklist has no scrollable name pills at top?
- [ ] **No filter buttons** — checklist has no per-person filter row?
- [ ] **No timestamps** — completed tasks don't show "✓ Glenn · 7:02 AM"?

## Feature Validation

- [ ] **Photo upload** — expand a task with a 📷 slot, take a photo. Does it save?
- [ ] **Troubleshooting** — expand a task with 🔧, tap a question. Does the answer show?
- [ ] **Notes** — expand a task, tap "Add a note", type something, save. Does it persist?
- [ ] **My Tasks** — shows only your assigned tasks grouped by section?
- [ ] **My Tasks green ✓** — when all tasks in a section are complete, shows ✓ instead of count?
- [ ] **"YOU" badge** — appears on sections where you have incomplete tasks, disappears when done?
- [ ] **Power Sequence** — all 10 items in order?
- [ ] **I/O Line List** — inputs, outputs, Dante tabs all render?

## Dynamic Roster

- [ ] **Settings page** — accessible from header ⚙️ icon?
- [ ] **Add member** — can add a new name to the roster?
- [ ] **Remove member** — can remove a member (with confirmation)?
- [ ] **Remember-me** — close and reopen app, are you still logged in as the same user?
- [ ] **Roster syncs** — roster changes appear on other devices?

## Readability

- [ ] **Task text** — comfortable reading size, not squinting on phone (should be 16px)?
- [ ] **Checkboxes** — easy to tap with thumb, not too small (28px)?
- [ ] **Section expand arrows** — clearly visible solid triangles, not squeezed against count text?
- [ ] **Assign dropdowns** — readable text inside dropdown, easy to tap (14px)?
- [ ] **Troubleshooting / notes text** — readable when expanded, not tiny (14px)?
- [ ] **Section headers** — title, count, unassigned label, and arrow all clearly readable with spacing?

## Mobile-Specific

- [ ] **Scroll performance** — smooth scrolling through full checklist?
- [ ] **Tap targets** — checkboxes, dropdowns, expand arrows all hittable with thumb?
- [ ] **Add to Home Screen** — works as standalone app (no browser chrome)?
- [ ] **Screen stays on** — important during setup, remind users to adjust auto-lock

## Known Limitations

1. **Photos are base64 in Firebase** — works fine for a handful of photos but large photos may be slow to sync. Auto-resize (800×600, 70% quality) keeps this manageable.
2. **No offline load** — if phone has never loaded the app before and has no internet, it can't load. Add to Home Screen while online to cache the page.
3. **No auth** — anyone with the URL can access the app. Fine for a small trusted team.
4. **Firebase free tier limits** — 10GB bandwidth/month, 1GB database storage. More than enough for this team size.
5. **Data persists across deploys** — redeploying the app only replaces code, never touches stored data. Checkboxes, assignments, notes, photos, roster, timer, history, repairs, and checklist structure all survive any number of redeploys.
6. **Custom checklist tasks don't support dependsOn** — the dependency warning system only works for built-in tasks with `dependsOn` properties. Tasks added via edit mode in the checklist will never show dependency warnings.
7. **Edit mode gate is name-based** — edit mode is restricted to `currentUser === "Sam"`. This is a weak gate (any volunteer could select "Sam" at login) but prevents accidental entry by the other 15 users. A stronger gate (passcode, settings toggle) is in the backlog.
8. **Concurrent note edits overwrite** — the entire `elim3-notes` object is written on every save. If two devices edit different tasks' notes simultaneously, last-write-wins applies to the whole object.

## Known Open Items

Open items requiring on-site investigation are tracked in **OPEN-ITEMS.md**.
Planned features and improvements are tracked in **BACKLOG.md**.