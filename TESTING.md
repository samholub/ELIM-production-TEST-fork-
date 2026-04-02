# Pre-Deployment Testing Checklist

Quick validation before taking this into a real Sunday setup. Current as of **v5.11.0**.

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
- [ ] **I/O Line List loads** — 3 tabs (Stage Box, Wing, Dante) render with data; Stage Box sub-toggle switches between Inputs and Outputs
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
- [ ] **Repairs sync** — edit a repair item on Phone A, change appears on Phone B within 1-2 seconds

### Offline & Recovery
- [ ] **Offline write preserved** — airplane mode on, check a box, airplane mode off → checkbox persists (not reverted)
- [ ] **No stale overwrite** — check a box on Phone A while Phone B is offline; bring Phone B online; Phone A's change should appear on Phone B (not be overwritten)
- [ ] **Write retry** — check a box with WiFi off, reconnect within 10 seconds; write reaches Firebase after reconnect
- [ ] **Offline banner** — offline warning banner appears when device loses connectivity
- [ ] **Sync error indicator** — 🔴 appears when Firebase writes fail after retries

### Null Safety
- [ ] **Null doesn't crash** — (via Firebase console) set `elim3-checks` to `null`; app falls back to empty state, no white screen

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
- [ ] **Mid-screen swipe works** — swipe-back triggers normally from center or bottom of screen
- [ ] **Back button still reliable** — ← button in header always navigates back, never intercepted by swipe

### Scroll Preservation
- [ ] **Preserved on navigate** — scroll to middle of checklist, tap back, re-enter → same position
- [ ] **Preserved on swipe-back** — swipe gesture preserves position same as back button
- [ ] **Fresh on new view** — navigating to a view for the first time starts at top

---

## Timer & Progress

### Timer Basics
- [ ] **First box doesn't start timer** — check one box, timer does NOT auto-start
- [ ] **Second box starts timer** — check a second box, timer auto-starts
- [ ] **Uncheck doesn't stop** — uncheck one of two checked boxes, timer keeps running
- [ ] **Timer persists** — set countdown, refresh page — countdown resumes with correct remaining time

### Countdown Timer
- [ ] **No target set** — dashboard shows count-up time with "⏱ Set Timer" link
- [ ] **Timer picker** — tapping "Set Timer" opens inline picker with 30/45/60/90 presets + custom input
- [ ] **Set preset** — tap "60m", timer area shows "60:00" countdown
- [ ] **Color green** — countdown above 50% remaining shows green text
- [ ] **Color yellow** — countdown between 25-50% remaining shows yellow/amber text
- [ ] **Color red** — countdown below 25% remaining shows red text
- [ ] **Auto-pause at zero** — when countdown reaches 0:00, timer stops and shows "⏰ Time!"
- [ ] **Change target** — "change" link opens picker to set a new target
- [ ] **Remove countdown** — "Remove countdown (use count-up)" option reverts to elapsed timer
- [ ] **Custom input** — can enter a custom number (e.g., 75 minutes) and it works
- [ ] **Timer in headers** — countdown with matching colors visible in sticky header on Checklist and My Tasks views

### Timer Auto-Stop
- [ ] **Auto-stop at 100%** — when all non-post-event tasks are checked, timer automatically pauses
- [ ] **One-way gate** — after auto-stop, unchecking a task does NOT restart the timer

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
- [ ] **History line** — "Last: {date} — {pct}% in {time}" visible below timer

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
- [ ] **Detail visible** — tasks with `item.detail` show text below assignee, up to 5 lines (hidden when checked)
- [ ] **Detail doesn't overflow** — long detail text wraps within card boundaries on mobile
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

### CHECKLIST_VERSION Stamp
- [ ] **Version mismatch resets structure** — if code's `CHECKLIST_VERSION` changes, stale `elim3-checklist-data` is deleted on load and rebuilt from `CHECKLIST_DATA` constants

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

### Tab Structure
- [ ] **Three tabs** — Stage Box, Wing, Dante
- [ ] **No Outputs tab** — outputs are under Stage Box via sub-toggle
- [ ] **Sub-toggle on Stage Box** — Inputs/Outputs pill-style segmented control appears below tab bar when Stage Box is selected
- [ ] **Sub-toggle not on other tabs** — Wing and Dante tabs do not show the sub-toggle
- [ ] **Defaults to Inputs** — Stage Box opens with Inputs sub-view selected

### Stage Box — Inputs
- [ ] **6 group headers** — Drums (12), Instruments (13), Talk Backs (2), Vocals (9), Wireless (4), Crowd (4)
- [ ] **Group header format** — uppercase label, thin divider line, channel count on right
- [ ] **Channels under correct groups** — channel numbers match group ranges (e.g., ch 1-12 under Drums)
- [ ] **44 total channels** — all stage box channels render
- [ ] **Channel numbers sorted** — numerically sorted within each group
- [ ] **"Other" group** — user-added channels with out-of-range numbers appear under "Other" at bottom
- [ ] **Type badges** — input type shown (XLR, DI/XLR, Line, 1/4")

### Stage Box — Outputs
- [ ] **IEM group** — "In-Ear Monitors" header with IEM pack channels (destination contains "in-ear" or "in ear")
- [ ] **Infrastructure group** — "Infrastructure" header with subs, mains, unused channels
- [ ] **18 total outputs** — all output entries render
- [ ] **In-ear snake badge** — channels with `inEarSnake` data show 🎧 pill badge (e.g., "🎧 Michael", "🎧 Snake 7")
- [ ] **Badge styling** — accent glow background, accent border, bold text, pill shape
- [ ] **No badge when empty** — channels without `inEarSnake` data show no badge

### Wing Tab
- [ ] **2 entries** — Bianca TB and ProPresenter Dante entries render
- [ ] **Unchanged** — no group headers or sub-toggle on Wing tab

### Dante Tab
- [ ] **5 routes** — all Dante routing entries render
- [ ] **Unchanged** — no group headers on Dante tab

### Cross-Tab Search
- [ ] **🔍 icon** — search icon visible in toolbar
- [ ] **Opens input** — tapping 🔍 opens search text field
- [ ] **Cross-tab results** — searching shows matching items from all sections (Stage Box, Wing, Outputs, Dante)
- [ ] **Section headers in results** — results grouped by section with labeled headers
- [ ] **Match count** — total result count displayed
- [ ] **Hides normal view** — tab content and sub-toggle hidden during active search
- [ ] **Empty state** — "No matches" message when search finds nothing
- [ ] **Clear restores** — clearing search text returns to normal tab/sub-toggle view
- [ ] **Case-insensitive** — search matches regardless of capitalization
- [ ] **Searches all fields** — matches channel number, label, notes, destination, inEarSnake, protocol, from/to

### Edit Permissions
- [ ] **Sam sees edit** — log in as Sam, ✏️ pencil visible in toolbar
- [ ] **Bri sees edit** — log in as Bri, ✏️ pencil visible in toolbar
- [ ] **Others don't** — log in as any other name, no edit button visible
- [ ] **Compact pencil** — edit button is a small ✏️ icon (not full-width "Edit I/O")

### Draft-Based Editing
- [ ] **Enter edit mode** — tap ✏️, inline inputs appear for all fields
- [ ] **✓ Done + Cancel** — compact buttons replace pencil during editing
- [ ] **Channel number edits** — can edit channel numbers
- [ ] **Label edits** — can edit channel labels
- [ ] **Notes edits** — can edit notes field
- [ ] **Custom type dropdown** — inputs tab shows IODropdown for type (XLR, DI/XLR, Line, 1/4") with dark theme, hover highlights, checkmark on current
- [ ] **Custom protocol dropdown** — Dante tab shows IODropdown for protocol (Dante, Analog, AES50, USB) with same styling
- [ ] **No native selects** — dropdowns are custom components, not browser-native `<select>`
- [ ] **Add channel** — "+ Add Input" / "+ Add Output" / "+ Add Route" buttons in edit mode
- [ ] **Remove channel** — ✕ button on each row in edit mode
- [ ] **Confirm modal on save** — tapping "✓ Done" triggers confirm modal before writing
- [ ] **Save writes to Firebase** — confirming save persists changes to all devices
- [ ] **Discard option** — can discard draft changes without saving
- [ ] **All sections editable** — Stage Box inputs, Stage Box outputs, Wing, and Dante all support editing

### Auto-Sort & Duplicate Detection
- [ ] **Auto-sort on blur** — editing a channel number and blurring re-sorts list numerically
- [ ] **Non-numeric at end** — entries like "L/R Main" sort after numeric entries
- [ ] **Sort preserves data** — labels, notes, types stay with correct channel numbers after sort
- [ ] **Duplicate inputs highlighted** — two inputs sharing a channel number show red styling
- [ ] **Duplicate outputs highlighted** — same behavior on outputs
- [ ] **No false positives** — unique channel numbers show normal styling
- [ ] **Dante unaffected** — Dante tab has no auto-sort or duplicate checks

---

## Repairs Tracker

- [ ] **Search visible** — search input at top of Repairs view
- [ ] **Search filters** — filters by item name and issue description, case-insensitive
- [ ] **Item count updates** — count reflects filtered results (e.g. "2 items of 6")
- [ ] **Edit mode toggle** — "✏️ Edit Repairs" enters edit mode; "✓ Done Editing" exits
- [ ] **Inline edits** — item name and issue description editable in edit mode
- [ ] **Priority dropdown** — changeable in edit mode; updates badge color immediately
- [ ] **Priority colors** — Urgent = red, Soon = orange/amber, Low = green
- [ ] **Delete item** — ✕ button removes item immediately
- [ ] **Add repair** — "+ Add Repair" at bottom in edit mode; hidden during search
- [ ] **Reset button** — restores to default items with confirmation flow
- [ ] **Edits persist** — edit an item, exit edit mode, refresh → edit persists
- [ ] **Cross-device sync** — edit on Phone A, change appears on Phone B within 1-2 seconds

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

- [ ] **Error boundary renders** — if a component throws, shows "Something went wrong — Tap to Reload" instead of white screen
- [ ] **Reload button works** — tapping "Tap to Reload" refreshes the page

---

## Known Limitations

1. **Photos are base64 in Firebase** — works fine at current scale. Auto-resize (800×600, 70% quality) keeps payloads manageable. Photo blob split is a backlog item (P3).
2. **No offline first-load** — if phone has never loaded the app and has no internet, it can't load. Add to Home Screen while online to cache.
3. **No auth** — anyone with the URL can access the app. Fine for a small trusted team.
4. **Firebase free tier limits** — 10GB bandwidth/month, 1GB database storage. More than enough for this team size.
5. **Data persists across deploys** — redeploying only replaces code, never touches stored data.
6. **Custom checklist tasks don't support dependsOn** — dependency warnings only work for built-in tasks.
7. **Edit mode gates are name-based** — checklist restricted to Sam; I/O restricted to Sam and Bri. Weak gates but prevent accidental entry by other users.
8. **Concurrent note edits overwrite** — the entire `elim3-notes` object is written on every save. Last-write-wins for simultaneous edits.
9. **Firebase rules are open** — read/write rules are unrestricted. Monitor for abuse if the URL leaks beyond the team.

---

## References

- Open items requiring on-site investigation: **OPEN-ITEMS.md**
- Planned features and improvements: **BACKLOG.md**
- Hardware configuration and packing: **HARDWARE-RUNBOOK.md**
