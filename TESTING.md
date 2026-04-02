# Pre-Deployment Testing Checklist

Quick validation before taking this into a real Sunday setup. Current as of **v5.12.0**.

---

## Quick Smoke Test (5 minutes)

Do these first. If any fail, stop and fix before Sunday.

- [ ] **App loads** — open https://elim-production.web.app on phone, no white screen or errors
- [ ] **Select a user** — name selection screen works, SVG logo centered
- [ ] **Check a box** — saves and persists after page refresh
- [ ] **Assign a task** — dropdown opens, selection persists after refresh
- [ ] **Cross-device sync** — check a box on Phone A, appears on Phone B within 1-2 seconds
- [ ] **My Tasks loads** — app opens to My Tasks view showing only your assigned tasks
- [ ] **Timer works** — start timer, verify it counts up and syncs to second device
- [ ] **I/O Line List loads** — three tabs (Stage Box, Wing, Dante) render with data
- [ ] **Back navigation** — back button and swipe-back both return to previous view

---

## Sync & Data Integrity

### Cross-Device Sync
- [ ] **Checkbox sync** — check a box on Phone A, appears on Phone B within 1-2 seconds
- [ ] **Assignment sync** — assign a task on Phone A, Phone B sees the assignment
- [ ] **Notes sync** — add a note on one device, appears on the other
- [ ] **Photo sync** — take a photo on one device, appears on the other (may be slow due to base64)
- [ ] **Crew status sync** — progress bars update across devices
- [ ] **Timer sync** — start timer on Phone A, Phone B shows same running time; pause on B, A stops
- [ ] **Countdown sync** — set a 45-min countdown on Phone A, Phone B shows same countdown with matching colors
- [ ] **I/O sync** — edit an I/O label on Phone A, change appears on Phone B after save

### Offline & Recovery
- [ ] **Offline write preserved** — airplane mode on, check a box, airplane mode off → checkbox persists (not reverted)
- [ ] **No stale overwrite** — check a box on Phone A while Phone B is offline; bring Phone B online; Phone A's change should appear on Phone B (not be overwritten)
- [ ] **Write retry** — check a box with WiFi off, reconnect within 10 seconds; write reaches Firebase after reconnect
- [ ] **Offline banner** — offline warning banner appears when device loses connectivity
- [ ] **Sync error indicator** — 🔴 appears when Firebase writes fail after retries

### Data Validation
- [ ] **Invalid type rejected** — (via Firebase console) set `elim3-checks` to a string like `"bad"`; app falls back to empty object `{}`, no white screen
- [ ] **Invalid entries dropped** — (via Firebase console) add a non-boolean entry to `elim3-checks` like `"fake-id": "hello"`; on load, that entry is dropped; valid boolean entries preserved
- [ ] **Array type enforced** — set `elim3-history` to an object `{}`; app falls back to empty array `[]`
- [ ] **Console warnings** — open browser console; corrupted data logs `sanitize: ...` warnings

### Schema Versioning
- [ ] **Version stored** — after first load, `elim3-schema-version` exists in Firebase with value `1`
- [ ] **No re-run** — subsequent loads do not log "Running migration" (already at current version)

---

## Navigation & Gestures

### View Stack
- [ ] **Back button works** — tapping ← goes to previous view, not always dashboard
- [ ] **Deep navigation** — Dashboard → Checklist → Crew Tasks → back → back returns to Dashboard
- [ ] **Swipe back matches button** — swipe gesture navigates same as ← button

### Swipe-Back Gesture
- [ ] **Follow-finger tracking** — content moves with finger in real-time during drag
- [ ] **Complete swipe** — dragging past 35% screen width navigates back with slide-out animation
- [ ] **Partial swipe snaps back** — dragging less than 35% springs content back
- [ ] **Header zone excluded** — swipe starting in top-left corner (top 80px, left 120px) does NOT trigger gesture
- [ ] **Photo areas excluded** — swiping in photo scroll areas doesn't trigger back navigation

### Scroll Preservation
- [ ] **Scroll restored** — scroll deep into Checklist, navigate to I/O, come back; scroll position restored
- [ ] **Dashboard starts at top** — returning to Dashboard always starts at top
- [ ] **Async views restore** — scroll deep into I/O List, navigate away and back; position restored despite async load

---

## Timer & Progress

### Timer Basics
- [ ] **Start/pause** — tap timer to toggle running state
- [ ] **Count-up mode** — timer counts up by default (no target set)
- [ ] **Syncs across devices** — running state and elapsed time match on all devices

### Countdown Timer
- [ ] **Presets work** — tap "⏱ Set Timer", select 30/45/60/90 minutes
- [ ] **Custom value** — enter a custom number, press "Set"
- [ ] **Color transition** — green above 50%, yellow 25-50%, red below 25%
- [ ] **Auto-pause at zero** — countdown stops automatically when it reaches 0:00
- [ ] **Timer text shows countdown** — displays remaining time, not elapsed
- [ ] **Remove countdown** — "Remove countdown" link switches back to count-up mode
- [ ] **Target persists** — set a 60-min countdown, refresh page → still showing countdown

### Timer Auto-Stop
- [ ] **Stops at 100%** — check all non-post-event tasks; timer auto-pauses
- [ ] **One-way gate** — uncheck a task after auto-stop; timer does NOT restart
- [ ] **Reset clears gate** — reset checklist, then timer can auto-stop again on next 100%

### Post-Event Exclusion
- [ ] **Dashboard excludes post-event** — progress percentage on dashboard does NOT count Post-Event section tasks
- [ ] **Timer excludes post-event** — timer auto-stop triggers at 100% of non-post-event tasks, not 100% of all tasks
- [ ] **Checklist includes post-event** — ChecklistView's own progress bar counts ALL tasks including post-event (intentional asymmetry)

### Manual Setup Complete
- [ ] **Button in Settings** — "Setup Complete" button visible in Settings
- [ ] **Two-tap confirm** — requires confirmation before executing
- [ ] **Stops timer** — timer pauses when Setup Complete is confirmed
- [ ] **Saves to history** — current progress saved to Setup History
- [ ] **No clearing** — checkboxes, assignments, notes remain intact

---

## Dashboard

### Hero Area
- [ ] **Stacked layout** — percentage on top, timer below, separated by divider line
- [ ] **Progress tap** — tapping progress zone navigates to full checklist
- [ ] **Timer isolated** — tapping timer controls doesn't navigate to checklist

### Crew Status
- [ ] **Per-person progress** — each crew member with progress bar
- [ ] **You sorted first** — your name appears at top of crew status list
- [ ] **Unassigned row** — "Unassigned tasks" row visible at bottom with progress bar
- [ ] **Tap crew name** — opens focused view of that person's tasks
- [ ] **Tap unassigned** — opens focused view showing only unassigned tasks

### Nav Cards
- [ ] **1×3 grid** — I/O Line List, Power Sequence, Signal Flow
- [ ] **No Checklist card** — checklist access only via hero "View all →"
- [ ] **No Settings card** — settings access only via header ⚙️

### dB Meter Card
- [ ] **Visible to Sam** — log in as Sam, dB Meter card appears below nav grid
- [ ] **Visible to Tyler** — log in as Tyler, dB Meter card appears
- [ ] **Hidden from others** — log in as any other name, no dB Meter card

---

## Checklist

### Display
- [ ] **15 sections** — Pre-Setup through Post-Event, numbered (1-15)
- [ ] **Section collapse** — sections collapse/expand with solid triangle arrows
- [ ] **Collapsed separators** — thin line visible between collapsed section headers
- [ ] **Smart auto-expand** — sections with your incomplete tasks auto-expand
- [ ] **Manual override** — collapse an auto-expanded section, it stays collapsed
- [ ] **"YOU" badge** — appears on sections where you have incomplete tasks, disappears when done

### Task Cards
- [ ] **Checkbox + text** — 28px checkbox with 16px task text
- [ ] **Assignment dropdown** — dark-themed dropdown, click-outside-to-close
- [ ] **Unassign option** — appears only when a name is already assigned
- [ ] **Current selection marked** — assigned name shows with ✓ prefix and accent color
- [ ] **Power sequence badge** — ⚡ badge with number on right side of relevant task cards
- [ ] **Detail visible** — tasks with `item.detail` show text below assignee without expanding (hidden when checked)
- [ ] **Detail overflow** — long detail text clamps at 5 lines with word-break
- [ ] **Note preview** — tasks with saved notes show 📌 preview below detail (hidden when checked)
- [ ] **Note preview hides on expand** — expanding a task hides the preview (full editor shown instead)
- [ ] **Unassigned styling** — unassigned unchecked tasks show dashed border, lighter background, italic dimmed text
- [ ] **Assign clears styling** — assigning a previously unassigned task restores normal solid styling

### Search
- [ ] **🔍 toggle** — search icon toggles search field visibility
- [ ] **Search filters tasks** — type a keyword, only matching tasks shown
- [ ] **Empty sections hidden** — sections with no matching tasks collapse/hide during search
- [ ] **Sections auto-expand** — sections containing matching tasks expand automatically
- [ ] **Case-insensitive** — search works regardless of capitalization
- [ ] **Clear restores normal** — clearing search field returns to normal view with manual expand/collapse

### Edit Mode (Sam only)
- [ ] **Sam sees edit button** — log in as Sam, ✏️ button visible next to search icon
- [ ] **Others don't see it** — log in as any other name, ✏️ button NOT visible
- [ ] **Enter/exit** — tap ✏️ to enter edit mode, tap "✓ Done" to exit
- [ ] **Sections expand** — entering edit mode auto-expands all sections
- [ ] **Delete task** — ✕ button appears; tap shows confirmation dialog, then removes task
- [ ] **Inline edit** — tap task text, inline input appears pre-filled; Enter saves, Escape cancels, blur saves
- [ ] **Add task** — "+ Add Task" button at bottom of each section in edit mode
- [ ] **Add task hidden during search** — "+ Add Task" not shown when search has text
- [ ] **Orphan cleanup** — delete a checked, assigned task with notes; verify `checks`, `assignments`, `taskNotes` no longer contain that task ID (check Firebase console)
- [ ] **Progress accurate** — delete 2 checked tasks; dashboard percentage doesn't exceed 100%

### Dependency Warnings
- [ ] **Warning triggers** — checking a task with unmet `dependsOn` shows amber modal
- [ ] **Modal content** — shows dependency task name, has Continue and Cancel buttons
- [ ] **Continue works** — clicking Continue checks the box despite warning
- [ ] **Cancel works** — clicking Cancel closes modal without checking box
- [ ] **No false warnings** — checking a task after its dependency is met produces no warning
- [ ] **Only dependsOn tasks** — tasks without `dependsOn` property never trigger warnings

---

## My Tasks

- [ ] **Default view** — app opens to My Tasks after login
- [ ] **Shows only your tasks** — only tasks assigned to you, grouped by section
- [ ] **Section numbers** — section headers are numbered same as in checklist
- [ ] **Green ✓ on completion** — when all tasks in a section are complete, header shows ✓ instead of count
- [ ] **Auto-collapse** — completed sections collapse to a single green ✓ header line
- [ ] **Manual expand** — tap any header to toggle; override remembered per session
- [ ] **Timer in header** — countdown with matching colors visible in sticky header

---

## Crew Focused Views

- [ ] **Tap crew name** — on dashboard, tap a crew member → opens "{Name}'s Tasks" focused view
- [ ] **Focused content** — only that person's assigned tasks shown, grouped by section
- [ ] **Timer in header** — countdown visible in sticky header
- [ ] **Your tasks highlighted** — if viewing someone else's tasks, tasks also assigned to you still get orange highlight
- [ ] **Empty state** — crew member with no tasks shows "No tasks assigned to {Name}"
- [ ] **Back navigation** — back button and swipe both return to dashboard
- [ ] **Unassigned view** — tapping "Unassigned tasks" row shows only unassigned tasks

---

## I/O Line List

### Display
- [ ] **Three tabs** — Stage Box, Wing, Dante
- [ ] **Stage Box sub-toggle** — Inputs / Outputs toggle below tabs (only visible on Stage Box tab, not during search)
- [ ] **44 inputs** — all stage box channels render in Inputs sub-view
- [ ] **6 input groups** — Drums (1-12), Instruments (13-25), Talk Backs (26-27), Vocals (28-36), Wireless (37-40), Crowd (41-44) with section headers and counts
- [ ] **Wing Direct Inputs** — Bianca TB and ProPresenter Dante entries
- [ ] **Outputs — IEM group** — outputs with "in-ear" destination grouped under "In-Ear Monitors" header
- [ ] **Outputs — Infrastructure group** — subs and mains grouped under "Infrastructure" header
- [ ] **18 outputs total** — all output entries render across both groups
- [ ] **Dante** — 5 routes render
- [ ] **Channel numbers** — inputs sorted numerically (1, 2, 3... not 1, 10, 2)
- [ ] **Type badges** — input type shown (XLR, DI/XLR, Line, 1/4")
- [ ] **In-ear snake pill badge** — `inEarSnake` value rendered as accent pill with 🎧 prefix on relevant output entries

### Cross-Tab Search
- [ ] **🔍 toggle** — search icon toggles search field visibility
- [ ] **Search spans all tabs** — results from Stage Box, Wing, Outputs, and Dante shown grouped with section headers
- [ ] **Result count** — total match count shown at top
- [ ] **Sub-toggle hidden** — Stage Box Inputs/Outputs toggle hidden during search
- [ ] **Case-insensitive** — search works regardless of capitalization
- [ ] **No matches message** — shows "No matches for ..." when nothing found

### Draft-Based Editing (Sam + Bri only)
- [ ] **Edit button visible** — log in as Sam or Bri, ✏️ button visible
- [ ] **Others don't see it** — log in as any other name, ✏️ button NOT visible
- [ ] **Enter edit mode** — tap ✏️, inline inputs appear for all fields
- [ ] **Channel number edits** — can edit channel numbers (text input, accepts any text)
- [ ] **Label edits** — can edit channel labels
- [ ] **Notes edits** — can edit notes field
- [ ] **Type dropdown** — `IODropdown` component for type field (XLR, DI/XLR, Line, 1/4")
- [ ] **Protocol dropdown** — `IODropdown` component for Dante protocol field (Dante, Analog, AES50, USB)
- [ ] **Add channel** — "+ Add" button at bottom in edit mode; new blank channel appears
- [ ] **Remove channel** — × button on each row in edit mode; removes immediately
- [ ] **Confirm modal on save** — tapping "✓ Done" triggers confirm modal before writing
- [ ] **Save writes to Firebase** — confirming save persists changes to all devices
- [ ] **Discard option** — can discard draft changes without saving
- [ ] **Cancel button** — exits edit mode without modal
- [ ] **All tabs editable** — Stage Box (Inputs and Outputs), Wing, and Dante tabs all support editing

### Auto-Sort & Duplicate Detection
- [ ] **Auto-sort on blur** — editing a channel number and blurring re-sorts list numerically
- [ ] **Non-numeric at end** — entries like "L/R Main" sort after numeric entries
- [ ] **Sort preserves data** — labels, notes, types stay with correct channel numbers after sort
- [ ] **Duplicate inputs highlighted** — two inputs sharing a channel number show red styling
- [ ] **Duplicate outputs highlighted** — same behavior on Outputs sub-view
- [ ] **No false positives** — unique channel numbers show normal styling
- [ ] **Dante unaffected** — Dante tab has no auto-sort or duplicate checks

---

## dB Meter (Sam + Tyler only)

### Access
- [ ] **Card visible** — log in as Sam or Tyler, dB Meter card visible on dashboard below nav grid
- [ ] **Card hidden** — log in as any other name, no dB Meter card
- [ ] **View loads** — tap dB Meter card, meter view opens
- [ ] **Back navigation** — back button and swipe return to dashboard

### Microphone
- [ ] **Permission prompt** — first visit prompts for microphone access
- [ ] **Denied state** — if denied, shows "Microphone access denied" with retry button
- [ ] **Retry works** — tapping retry re-requests permission
- [ ] **AGC disabled** — verify via browser dev tools that `autoGainControl: false` is set on the media stream

### Readings
- [ ] **dBFS displayed** — large centered number showing current level in dBFS
- [ ] **dBA displayed** — A-weighted reading shown in secondary row
- [ ] **Est. SPL displayed** — estimated SPL shown in secondary row (dBA + 94 + cal offset)
- [ ] **Meter bar** — horizontal bar fills left-to-right tracking dBFS level
- [ ] **Color zones** — bar green below -20dB, yellow -20 to -6, red above -6
- [ ] **Peak marker** — thin vertical line on bar at peak position
- [ ] **Peak values** — peak dBFS and peak dBA shown as numbers
- [ ] **Scale labels** — -60, -40, -20, 0 labels below the bar
- [ ] **Responsive** — readings update smoothly in real-time
- [ ] **Silent room** — quiet room reads around -40 to -50 dBFS (not stuck at -60)
- [ ] **Loud clap** — clapping near phone spikes the meter noticeably

### Peak Hold
- [ ] **Peak captures** — loud sound sets peak marker; it stays for ~2 seconds
- [ ] **Peak decays** — after 2 seconds, peak marker slowly falls toward current level

### Calibration
- [ ] **+/- buttons work** — tapping + increases offset by 1, - decreases by 1
- [ ] **Offset shown** — "Cal: +X dB" or "Cal: -X dB" displayed
- [ ] **SPL adjusts** — Est. SPL changes immediately when offset changes
- [ ] **Session only** — offset resets on page reload (not persisted)

### Guide
- [ ] **Toggle works** — "How This Works" expands/collapses guide section
- [ ] **Content present** — dBFS, A-Weighted, Est. SPL, Peak, calibration, accuracy sections all render

### Cleanup
- [ ] **Navigate away** — go back to dashboard; no audio continues (mic stops, no console errors)
- [ ] **Re-enter** — navigate back to dB Meter; starts fresh with new mic stream

---

## Signal Flow Diagrams

- [ ] **Three diagrams render** — Audio (orange), Video (purple), Network (green)
- [ ] **No clipping** — all boxes and labels visible on iPhone SE width
- [ ] **Text readable** — equipment names and connection labels legible
- [ ] **No overlap** — boxes have clear separation, no overlapping text
- [ ] **Scrolling smooth** — smooth scroll through all three sections
- [ ] **ATEM name correct** — diagrams show "ATEM TV Studio 4K" (not "ATEM Mini Pro")
- [ ] **Equipment names match** — all names match actual equipment (Midas DL251, Behringer Wing, etc.)
- [ ] **Tyler's Mac** — shown branching from Wing via USB-C (separate from Dante switch)
- [ ] **Bianca IEM** — shown from Wing XLR Out 8

---

## Power Sequence

- [ ] **10 items in order** — Stage Box through Speakers
- [ ] **Equipment names correct** — Item 5 = "ProTools Macs (Tyler + Omar/Mitch)", Item 7 = "ProPresenter Mac"
- [ ] **Teardown note** — reverse order (10→1) guidance present

---

## Photos, Notes & Troubleshooting

- [ ] **Photo upload** — expand a task with 📷 slot, take a photo; it saves and persists
- [ ] **Photo sync** — photo taken on one device appears on the other
- [ ] **Notes** — expand a task, add a note, save; it persists and syncs
- [ ] **Troubleshooting** — expand a task with 🔧, tap a question; answer toggles open

---

## Settings & Roster

### Layout
- [ ] **Section order** — Reset Checklist → Setup Complete → Team Roster → Setup History → About

### Reset
- [ ] **Two-tap reset** — "🔄 Reset Checklist" → confirmation → "Yes, Reset" / "Cancel"
- [ ] **Reset preserves** — assignments, notes, photos, roster intact after reset
- [ ] **History saved** — new entry appears in Setup History after reset
- [ ] **52 sessions max** — history stores up to 52 entries
- [ ] **Interrupted reset recovers** — if `elim3-pending-reset` exists in Firebase on load, app completes the reset automatically

### Team Roster
- [ ] **Add member** — can add a new name to the roster
- [ ] **Remove member** — confirmation dialog shows blast-radius warning listing affected assignments
- [ ] **Assignments cleared** — removed member's tasks become unassigned
- [ ] **Member gone** — removed member no longer in login screen or assignment dropdowns
- [ ] **Roster syncs** — changes appear on other devices

### Login
- [ ] **Remember-me** — close and reopen app, still logged in as same user
- [ ] **Tap name to switch** — tapping centered username in header returns to user selection
- [ ] **Settings via gear** — ⚙️ in header navigates to Settings

---

## Header & Chrome

- [ ] **Layout** — logo left, centered username with ▾, ⚙️ gear right
- [ ] **No gap above header** — header background extends behind iOS status bar
- [ ] **Header stays fixed** — remains at top during scroll on all views
- [ ] **Blur effect** — semi-transparent header with backdrop blur during scroll
- [ ] **Safe area correct** — works on notched iPhones and non-notched devices
- [ ] **Back button hit area** — tapping arrow + title text navigates back (not just the arrow)

---

## SVG Brand Mark

- [ ] **Login screen** — SVG logo (76px) centered, sharp chevrons pointing right, block E
- [ ] **Loading state** — SVG logo (56px) centered while app loads
- [ ] **Dashboard header** — SVG logo (32px) left-aligned, recognizable at small size
- [ ] **Sharp corners** — chevron tips and E corners are all sharp (no rounded edges)
- [ ] **Consistent** — all 3 logo sizes look the same

---

## Warm Night Theme

- [ ] **4-state colors** — your tasks = orange border, completed = green, unassigned = dashed/dim, others = neutral
- [ ] **Background** — dark warm tone (#111010)
- [ ] **Warm text** — cream-colored text (#F0E6D8), not cool grey
- [ ] **No per-person colors** — checklist and dashboard use only orange/green/dashed/neutral
- [ ] **No timestamps** — completed tasks don't show completion times

---

## Mobile & Performance

- [ ] **Scroll performance** — smooth scrolling through full checklist with timer running
- [ ] **Tap targets** — checkboxes, dropdowns, expand arrows all hittable with thumb (44px minimum)
- [ ] **Add to Home Screen** — works as standalone app (no browser chrome)
- [ ] **App loads without waiting for photos** — interactive before photos finish loading
- [ ] **Desktop scroll wheel** — mouse scroll works in desktop browser
- [ ] **No horizontal scroll** — no unwanted horizontal scrolling

---

## Error Handling

- [ ] **Error boundary renders** — if a component throws, shows "Something went wrong" instead of white screen
- [ ] **Reload button works** — tapping "Tap to Reload" refreshes the page
- [ ] **Export backup button** — "Export Backup" button visible alongside reload button
- [ ] **Export downloads JSON** — tapping "Export Backup" downloads a JSON file with all `elim3-*` keys
- [ ] **Export includes error** — downloaded JSON contains `error` field with the crash message
- [ ] **Export includes timestamp** — downloaded JSON contains `exportedAt` ISO timestamp

---

## Known Limitations

1. **Photos are base64 in Firebase** — works fine at current scale (no photos currently uploaded). Auto-resize (800×600, 70% quality) keeps payloads manageable. Photo blob split is the next P3 item.
2. **No offline first-load** — if phone has never loaded the app and has no internet, it can't load. Add to Home Screen while online to cache.
3. **No auth** — anyone with the URL can access the app. Fine for a small trusted team.
4. **Firebase free tier limits** — 10GB bandwidth/month, 1GB database storage. More than enough for this team size.
5. **Data persists across deploys** — redeploying only replaces code, never touches stored data.
6. **Custom checklist tasks don't support dependsOn** — dependency warnings only work for built-in tasks.
7. **Edit mode gate is name-based** — checklist restricted to `currentUser === "Sam"`, I/O restricted to Sam or Bri. A weak gate but prevents accidental entry by other users.
8. **Concurrent note edits overwrite** — the entire `elim3-notes` object is written on every save. Last-write-wins for simultaneous edits.
9. **Firebase rules are open** — read/write rules are unrestricted. Monitor for abuse if the URL leaks beyond the team.
10. **dB Meter calibration not persisted** — calibration offset resets on page reload. Intentional — mic characteristics vary by session.
11. **dB Meter accuracy** — phone mic-based; reliable for relative comparisons and quick level checks, not a replacement for a calibrated SPL meter.

---

## References

- Open items requiring on-site investigation: **OPEN-ITEMS.md**
- Planned features and improvements: **BACKLOG.md**
- Hardware configuration and packing: **HARDWARE-RUNBOOK.md**
