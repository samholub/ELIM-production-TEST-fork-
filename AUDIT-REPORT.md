# ELIM Production App — Zero-Principles Audit Report

**Auditor:** Principal Solutions Architect (AI-assisted)
**Date:** 2026-03-31
**Codebase version:** v5.9.0 (per in-app label, line 1282 of `index.html`)
**Scope:** Full read of `index.html` (~2,834 lines), `.cursorrules`, `PROJECT-BRIEF.md`, `BACKLOG.md`, `ELIM-PROJECT.md`, `OPEN-ITEMS.md`

---

## Acknowledged Deliberate Decisions (One Line Each)

- **Firebase rules fully open** — intentional for a small trusted team; risk documented in PROJECT-BRIEF.md.
- **API key in source HTML** — normal for Firebase client-side apps; keys are not secrets.
- **No authentication** — intentional; volunteers select name on launch.
- **Single-file architecture** — intentional; zero build step, single deploy artifact.

---

## 1. Fragile Assumptions

### 1.1 `useStorage` Sync: Last-Write-Wins with a Silent Freeze Bug

**What:** The `_pendingWrites` Set tracks keys with in-flight Firebase writes and suppresses incoming `elim-sync` events for those keys. On successful write, the key is removed from `_pendingWrites`. On final retry failure (after 4 retries), the error event is dispatched — but the key is **never removed** from `_pendingWrites`.

**Why it matters:** If a write fails all retries (spotty venue WiFi), the key stays in `_pendingWrites` permanently (until page reload). From that point forward, the Firebase listener silently ignores all remote updates for that key. The device is frozen on stale data for that key with no user-visible indication. Other devices' changes to that key are invisible.

**Concrete scenario:** Sam checks a box while WiFi drops. Write retries exhaust. Sam's device shows the box checked. WiFi comes back. Tyler checks the same box from his phone. Sam's device never sees Tyler's update for `elim3-checks` because the key is stuck in `_pendingWrites`. Sam's next write to that key will overwrite Tyler's state.

**Fix:** In the final `.catch` handler of `attemptWrite` (after all retries exhausted), add `_pendingWrites.delete(key)`. The sync-status error event already fires — the key just needs to be unfrozen so future remote updates can flow in.

### 1.2 Offline Writes: No Conflict Resolution, No Merge

**What:** When two devices edit the same storage key while one or both are offline, whichever device writes to Firebase last wins. The entire value (full checks object, full assignments object, etc.) is overwritten — not merged at the field level.

**Why it matters:** If device A checks tasks 1-5 offline and device B checks tasks 6-10 offline, when both reconnect, one device's 5 checks are silently lost. Firebase's `.set()` replaces the entire node.

**Concrete scenario:** Two volunteers work different sections while venue WiFi flickers. One reconnects first and pushes their checks. The second reconnects and overwrites the first's checks with their own copy (which lacks the first volunteer's changes).

**Fix:** For object-valued keys (checks, assignments, notes), write individual fields using Firebase's `.update()` instead of `.set()` on the full object. This way `checks/ps-1 = true` and `checks/sb-1 = true` are independent writes that don't clobber each other. This requires restructuring how `storage.set` handles object values vs. scalar values.

### 1.3 Canonical Source Pattern: I/O, Repairs, and Checklist Edits Are Ephemeral

**What:** Lines 2652–2660 run on every app load and actively **delete** `elim3-checklist-data`, `elim3-io-config`, and `elim3-repairs` from Firebase and localStorage. Meanwhile, `IOListView` and `RepairsView` use plain `useState` (not `useStorage`), and `checklistData` in the main app is `useState(CHECKLIST_DATA)`.

**Why it matters:** The in-app edit modes for I/O Line List, Repairs, and Checklist tasks give the user a fully functional editing UI — but all edits are lost on page refresh, on navigation away from the view (component unmount), and never sync to other devices. This is a fake-persistence trap. A user who carefully edits the I/O list will lose everything the moment they navigate away.

**Concrete scenario:** Sam edits the I/O line list on-site to correct a channel label. He navigates to the checklist to verify something. He comes back to the I/O list — his edits are gone. The component remounted with `DEFAULT_IO_LINE_LIST`.

**Clarification needed:** The cleanup on lines 2652–2660 appears to be intentional (forcing the code constants to always be canonical). But it creates a contradiction: the edit UI exists and works in-session, but nothing persists. Either the edit UI should be removed (read-only reference), or the data should persist via `useStorage`.

### 1.4 No Data Migration Path

**What:** When `CHECKLIST_DATA` changes in code (tasks added, removed, IDs changed) and the app is redeployed, the live `elim3-checks` and `elim3-assign` objects in Firebase still reference the old task IDs. There is no migration function to reconcile old data with a new checklist structure.

**Why it matters:** Orphaned data accumulates silently. Checks for deleted tasks remain in Firebase forever. Assignments for renamed task IDs persist but no longer match any task. Over time, the checks/assignments objects grow with dead keys. More critically, if a reset is done to pick up the new structure, assignments, notes, and photos are preserved — but they're keyed to old task IDs that may no longer exist in the new structure.

**Fix:** The schema versioning approach in BACKLOG.md Priority 4 is correct. Add `elim3-schema-version` and a `runMigrations()` function on load that can remap old task IDs to new ones, prune orphaned keys, and handle structural changes without requiring a full reset.

### 1.5 Base64 Photos: Unbounded Growth on a Single Firebase Key

**What:** All photos are stored as a single JSON object under `elim3-photos`. Photos persist across weekly resets (by design). Each photo is ~30–80KB as a base64 string. Firebase's `.on('value')` and `.get()` download the **entire** `elim3-photos` node on every page load and every sync event.

**Why it matters:** After 52 weeks at ~10 photos/week, the photos blob is ~26MB. After 2 years, ~52MB. Firebase Realtime Database serves the full node as a single JSON payload. A 50MB JSON string requires 3–5× memory to parse, pushing old phones toward the ~150MB heap limit. Initial load time degrades linearly with photo count. The `loaded` gate (line 2750) blocks rendering until photos finish loading, so every user waits for the full photo download before seeing anything.

**Storage math:** Firebase Spark plan allows 1GB storage. 52 weeks × 10 photos × 60KB avg = ~30MB/year. Storage isn't the immediate bottleneck — download time and memory are.

**Fix:** Either (a) move photos out of the real-time sync path and into Firebase Storage (separate service, URL-based), or (b) partition photos by date range or session and only load the current session's photos eagerly, with history photos loaded on demand.

### 1.6 `newWeek` Reset Is Not Atomic

**What:** The `newWeek` function (lines 2598–2631) writes a `pending-reset` flag, then independently clears checks, completions, and timer state via separate `setChecks({})`, `setCompletions({})`, and `setTimerState({...})` calls. Each triggers an independent Firebase write via `useStorage`'s `save()`. There is no transaction or batched write.

**Why it matters:** If the device loses connectivity or the user closes the tab mid-reset, the app could end up with checks cleared but history not updated (data loss), or completions cleared but checks still present (inconsistent state). The `pending-reset` flag mechanism is a good mitigation — on next load, the flag is detected and clears are re-run. But between the crash and the next load, the data is temporarily inconsistent and visible to all devices.

**Actual risk:** Low-medium. The pending-reset flag handles the most likely failure mode (interrupted reset). But the flag itself is written to Firebase asynchronously, so a crash before the flag reaches Firebase AND localStorage would leave no trace that a reset was attempted.

### 1.7 PWA Is Not a Functional PWA

**What:** The app has a manifest (base64 data URI, line 13) with `"icons": []` and `"display": "standalone"`. There is no service worker. No `navigator.serviceWorker.register()` call anywhere in the code.

**Why it matters:** Without a service worker:
- A browser refresh in a dead zone = dinosaur screen. The app requires network connectivity to load.
- "Add to Home Screen" produces an icon-less shortcut that may not behave as a standalone app on all platforms.
- There is no offline caching of the HTML, CDN scripts (React, Babel, Firebase), or fonts.
- The manifest's `"icons": []` means iOS won't use a proper app icon, and Android won't show the install prompt.

**Concrete scenario:** The venue WiFi drops 10 minutes into setup. A volunteer accidentally closes the browser tab (or the browser clears it from memory). They reopen — blank page. They have no access to the checklist, assignments, or any reference data until connectivity returns.

**Fix:** Add a minimal service worker that caches `index.html` and the CDN dependencies on install. Cache-first strategy for the HTML, stale-while-revalidate for CDN scripts. This is listed in BACKLOG.md Priority 5 but should be elevated — it's a reliability issue, not a feature.

---

## 2. Data Resilience

### 2.1 Cross-Key Consistency: No Transactional Guarantees

**What:** The 11 storage keys are independently read and written. Operations that span multiple keys (reset, task deletion, roster member removal) perform sequential independent writes with no rollback mechanism.

**Why it matters:** Task deletion (lines 1544–1555) independently updates `checklistData`, then `checks`, then `assignments`, then `taskNotes`, then `completions`. If the browser tab crashes between the `assignments` and `taskNotes` writes, the task is deleted from the checklist but its note persists as an orphan in Firebase. The orphan cleanup code is correct in structure but not resilient to partial execution.

**Blast radius assessment:** Corruption of one key doesn't cascade to others — they're independent. A corrupted `elim3-checks` doesn't affect `elim3-assign`. The worst case is orphaned data (harmless but wasteful) or missing data for a key that should exist (user sees default state for that key). No key corruption can cause a white-screen crash because `useStorage` always falls back to the initial value.

### 2.2 `JSON.parse(JSON.stringify())` Deep Clone: Low Risk for This Data Shape

**What:** Used 8+ times throughout the codebase for deep cloning before mutations.

**Edge cases:**
- `undefined` object values → silently dropped (keys vanish)
- `Date` objects → become ISO strings (irreversible)
- `NaN` → becomes `null`
- Functions → silently dropped
- Circular references → throws (unrecoverable crash)

**Why it matters for THIS app:** The data structures are simple: strings, numbers, booleans, arrays of plain objects. No Date objects in the data model, no functions, no circular references. The `undefined` silently dropping is the only realistic concern — if a task field is explicitly set to `undefined` (rather than omitted), a deep clone would erase it. In practice this hasn't happened because task data comes from code constants with no undefined values.

**Fix:** Extract to `deepClone()` utility as planned in BACKLOG.md Priority 4 for readability and single point of change, but don't add complexity (e.g., structuredClone) unless a data shape change introduces Date objects or similar.

### 2.3 `_ls` localStorage Wrapper: Silent Failure Is the Wrong Default for Writes

**What:** The `_ls.set()` method catches and swallows all errors. If localStorage is full (5MB quota on most browsers), writes fail silently.

**Why it matters:** `elim3-photos` stores all photos as a JSON-serialized object in localStorage under `elim_elim3-photos`. As photos accumulate, this can exceed the 5MB localStorage quota. When that happens, `_ls.set()` silently fails. The photo appears saved (it's in React state and pushed to Firebase), but the localStorage cache is not updated. On next load, if Firebase is unreachable, the fallback to localStorage returns stale photo data. Not catastrophic, but the user's mental model ("I took a photo, it's saved") is violated.

**Fix:** `_ls.set()` should catch quota errors specifically and surface them via a `console.warn` at minimum. For the `elim3-photos` key specifically, consider not caching in localStorage at all (Firebase is the source of truth for photos, and the base64 strings are the biggest localStorage consumers).

### 2.4 Firebase Returning Null: Handled Correctly with One Gap

**What:** The `storage.get()` function checks `snap.exists()` before using Firebase data. The `.on('value')` listener checks `val !== null && val !== undefined` before dispatching. `useStorage` catches all errors and falls back to the initial value.

**The gap:** If someone (or a script) deletes a Firebase key entirely, the `.on('value')` listener fires with `val = null`, and the condition on line 92 (`if (val !== null && val !== undefined)`) prevents the sync event from dispatching. The local device retains its last known value and doesn't know the key was deleted. This is actually protective — an accidental Firebase key deletion doesn't wipe client state. But it also means there's no way to force-clear a key on all devices except by writing an explicit empty/default value.

### 2.5 `elim-sync` CustomEvent: No Debouncing, Potential State Thrashing

**What:** Every Firebase `.on('value')` callback immediately dispatches an `elim-sync` CustomEvent. The `useStorage` handler calls `setVal()` on every event. There is no debouncing or batching.

**Why it matters:** If 16 volunteers are all checking boxes simultaneously, `elim3-checks` receives rapid-fire updates. Each update triggers a full checks object replacement in every device's `useStorage` handler, causing a re-render cascade in `ElimProductionApp` and all children. React 18's automatic batching helps with synchronous updates, but Firebase callbacks are asynchronous — each one triggers a separate render cycle.

**Practical severity:** Low-medium. 16 concurrent checkbox toggles would generate 16 Firebase events within a few seconds. Each causes a full re-render of the checklist. On old phones, this could feel janky. Not data-corrupting, but a UX smoothness issue.

### 2.6 RTDB Payload Size: Photos Are the Memory Bomb

**What:** `elim3-photos` is a single Firebase node containing all photos as base64 strings. Firebase RTDB transfers the entire node on initial `.get()` and on every `.on('value')` update. The payload is a JSON-serialized string of the entire photos object.

**Why it matters:** When any device adds a photo, every other device's Firebase listener receives the ENTIRE photos object (all photos for all tasks for all time). On a phone with 2GB RAM and a browser tab memory limit of ~150MB, a 30MB+ photos blob will cause parsing delays and potential tab crashes. Worse: the `loaded` gate blocks all rendering until photos finish loading.

**Current trajectory:**
- Week 1: ~500KB (10 photos × 50KB)
- Week 26: ~13MB
- Week 52: ~26MB
- Week 104: ~52MB — probable tab crashes on budget phones

**Fix:** This is the highest-priority architectural fix. Options: (a) Firebase Storage for photos with URLs in RTDB, (b) partition photos by session ID, (c) aggressive photo pruning on reset (only keep current session + reference photos).

---

## 3. Volunteer UX Failure Modes

### 3.1 First-Time Volunteer: "What Do I Do?"

**What:** A new volunteer opens the app, selects their name, and sees the My Tasks view. If no tasks are assigned to them, they see "No tasks assigned to you yet" with a "View Full Checklist" button.

**Why it matters:** The volunteer now needs to: (1) understand they need to navigate to the full checklist, (2) find tasks relevant to them, (3) use the assignment dropdown to assign tasks to themselves. This is 3 steps of non-obvious interaction during a rushed 30-minute setup with no training. The likely outcome is the volunteer either assigns themselves random tasks or gives up and asks Sam.

**What should happen:** The team lead (Sam) assigns tasks before the setup. Assignments persist across resets. So in theory, the first-time experience only happens once per volunteer. But with volunteer turnover ("the app can't assume familiarity"), this is a recurring friction point.

**Fix:** Add an onboarding hint on the empty My Tasks state: "Ask your team lead to assign you tasks, or tap below to see all tasks and pick some." The BACKLOG Priority 1 "All" assignment option would also help — communal tasks would show up for everyone.

### 3.2 No Undo for Any Action

**What:** Checking/unchecking a box, reassigning a task, editing a note, removing a photo, deleting a history entry, removing a roster member — all changes are immediately written to Firebase and propagated to all devices. There is no undo mechanism.

**Why it matters:** A fumbled tap reassigns someone else's task. A swipe while scrolling accidentally toggles a checkbox. The change instantly propagates to 15 other devices. The only recovery is to manually reverse the action — if the user noticed it happened.

**Concrete scenario:** A volunteer scrolling through the checklist accidentally taps a checkbox. The timer starts (auto-start on 2nd check). They uncheck it, but the timer keeps running. They don't know how to stop it. The timer now shows incorrect elapsed time for the entire team.

**Fix:** For checkboxes, add a brief "undo" toast (3-second window) before writing to Firebase. For assignments, the current dropdown approach is correct (requires explicit selection). For the timer auto-start, consider raising the threshold or requiring an explicit "Start Timer" action.

### 3.3 Timer Auto-Start Behavior: Not Discoverable

**What:** The timer auto-starts when the 2nd checkbox is checked across any device (line 1530). There is no visual indication that this will happen, no confirmation, and no explanation.

**Why it matters:** A volunteer checking their first two tasks has no idea they just started a team-wide timer. If someone accidentally checks and unchecks boxes during task review, the timer starts and stays running. The timer display says "⏱ Running" but doesn't explain why it started or how to stop it.

**Fix:** Remove the auto-start behavior. Make the timer explicitly started via the timer UI. The auto-stop on 100% completion is fine — it's the auto-start that's confusing.

### 3.4 Assignment Dropdown Always Visible on Incomplete Tasks

**What:** `AssignDropdown` renders for every task regardless of state (line 1039). Even tasks that are already assigned show the dropdown with the assignee name and a ▾ indicator.

**Why it matters:** During setup, a volunteer scrolling through their tasks might accidentally tap the dropdown and reassign a task. The dropdown opens on any tap of the assignee area — there's no tap-and-hold or other protection against accidental activation.

**Concrete scenario:** Volunteer scrolling quickly, finger lands on "Tyler ▾" instead of the checkbox. Dropdown opens. They tap to dismiss but accidentally tap a name. Task is now reassigned. Tyler's task count changes on his device. Nobody knows what happened.

**Fix:** Show assignee as plain text by default. Only show the dropdown when the task is unassigned or when in an explicit "assign mode." This is already identified in BACKLOG Priority 1 ("Task Card Density").

### 3.5 Roster Removal: No Protection Against Removing Active Members

**What:** Any user can remove any roster member via Settings (line 1134–1149). The only protection is a `confirm()` dialog. Removing a member clears all their assignments.

**Why it matters:** If someone accidentally removes "Tyler" from the roster, all of Tyler's 8 task assignments are instantly cleared across all devices. Tyler's name disappears from the login screen. If Tyler is currently logged in, he's logged out (line 1146–1149). There's no way to restore the assignments — they're gone.

**Fix:** Move roster management behind a confirmation that mentions the blast radius ("Tyler has 8 assigned tasks. Removing them from the roster will clear all assignments. This cannot be undone."). Consider making roster management Sam-only (like edit mode).

### 3.6 No Offline Indicator Until Write Failure

**What:** The app shows a 🔴 sync error indicator only after a Firebase write fails all retries (lines 2540–2548). There is no proactive connection monitoring.

**Why it matters:** A volunteer could be checking boxes for 5 minutes on a dead WiFi connection, thinking everything is syncing. Their checks are saved to localStorage but not Firebase. Other devices don't see the changes. When WiFi returns, the full checks object is pushed to Firebase, potentially overwriting other devices' changes (see finding 1.2).

**Fix:** Add a Firebase `.info/connected` listener that shows an immediate "Offline — changes saved locally" banner when the connection drops. This gives the user awareness that their changes aren't syncing yet.

### 3.7 Checkbox Tap Target: Below Minimum Size

**What:** Checkboxes are 28×28px (line 1022). The minimum recommended touch target for mobile is 44×44px (Apple HIG) or 48×48dp (Material Design).

**Why it matters:** During rushed setup with sweaty/cold hands, a 28px target on a phone screen is easy to miss. The miss-tap might land on the task text (expanding details) or the assignment dropdown (opening it accidentally) instead.

**Fix:** Keep the visual checkbox at 28px but increase the tappable area to 44×44px via padding on the button element.

### 3.8 History Entry Deletion: Tiny Tap Target, No Undo

**What:** Each history entry has a ✕ button (line 1262) that is styled at `fontSize: 14` with `padding: "2px 4px"`. The effective tap area is approximately 20×20px. The only protection is a `confirm()` dialog.

**Why it matters:** A fat-finger while scrolling through setup history could delete a historical record. Historical data is used for week-over-week comparison and trend analysis. Once deleted, it's gone from all devices.

**Fix:** Either remove the delete button from individual history entries (only allow clearing all history from settings), or increase the tap target and add a more prominent confirmation.

---

## 4. The "Fake Feature" Check

### 4.1 I/O Line List and Repairs Edit Modes: Fully Functional But Ephemeral

**What:** `IOListView` (line 1840) uses `useState(DEFAULT_IO_LINE_LIST)`. `RepairsView` (line 2310) uses `useState(DEFAULT_REPAIRS)`. Neither uses `useStorage`. Additionally, lines 2652–2660 actively delete any previously stored Firebase/localStorage data for these keys on every app load.

**Why it matters:** The edit UI is fully functional — you can add channels, remove routes, change repair priorities, reset to defaults. But every edit is lost when the user navigates away from the view (component unmounts) or refreshes the page. The "Reset to Defaults" button is meaningless because the component always loads from defaults.

**Severity:** High. This directly contradicts the ELIM-PROJECT.md statement that "In-app edits to checklist structure, I/O config, or repairs are stored in Firebase." They are NOT stored in Firebase — the code actively prevents it.

**Fix:** Either (a) reconnect these views to `useStorage` so edits persist and sync (remove the deletion code on lines 2652–2660), or (b) remove the edit modes entirely and make these views read-only references from the code constants. Option (a) conflicts with the "code is canonical" pattern; option (b) is honest about the architecture.

### 4.2 Placeholder Views Still Routed

**What:** `PlaceholderView` (line 2464) still exists. Equipment and Purchases routes are still active in the main app (lines 2818, 2820). They're accessible via Settings → "Coming Soon" section (lines 1288–1306).

**Why it matters:** They've been moved out of the main dashboard nav grid (good), but they're still reachable from Settings and still have active routes. A volunteer exploring Settings sees "Equipment" and "Purchases" cards, taps them, and gets a dead end. It's a minor nuisance, but it's dead code that adds to the component count and maintenance surface.

**Fix:** Remove `PlaceholderView`, remove the routes from the main app's view switch, and remove the "Coming Soon" section from `SettingsView`. Add these features when they're built, not before.

### 4.3 Signal Flow SVGs: Hardcoded and Already Drifting

**What:** Three SVG diagrams (~160 lines total) in `SignalFlowView` (lines 2138–2298) hardcode equipment names, connections, and routing. The I/O Line List already contains the routing information that the SVGs depict.

**Why it matters:** The SVGs currently reference "ATEM TV Studio 4K" correctly, but previous versions said "ATEM Mini Pro" (per BACKLOG.md, this was a known correction). As equipment changes, the SVGs must be manually updated separately from the I/O data — a dual source of truth. The SVGs will silently drift from the actual I/O configuration.

**Current state:** The ATEM name appears corrected in the SVGs. But the video signal chain SVG shows "ProPresenter → Internet → Resi" which isn't quite right (the Resi gets direct internet, not through ProPresenter). The audio chain shows the AVIO → ATEM path correctly.

**Fix:** Accept SVGs as a manually maintained reference document (they're professional and always sharp). Add a comment at the top of `SignalFlowView` noting the last-verified date and which equipment changes would require SVG updates. The dynamic generation approach from BACKLOG Priority 5 is ambitious and probably not worth the complexity.

### 4.4 Checklist In-App Edit Mode: Works But Edits Don't Survive Reload

**What:** The checklist edit mode (Sam-only) allows adding, removing, and renaming tasks. These edits modify the `checklistData` state, which is a plain `useState(CHECKLIST_DATA)` (line 2538). Like the I/O and Repairs views, edits are lost on page refresh.

**Why it matters:** Sam edits a task description during setup to fix a typo. The fix is visible to all devices because `checklistData` flows through props. But when anyone refreshes, the edit is gone. Sam thinks he fixed it; next week it's back to the old text.

**Key difference from I/O/Repairs:** Checklist edits DO propagate to other devices in the current session because `checklistData` is lifted to the root component and passed as props. I/O and Repairs edits don't even propagate because they're local component state.

**Fix:** Same as 4.1 — either persist via `useStorage` or remove the edit mode. If persisted, the "code is canonical, Firebase is live" pattern needs a reconciliation strategy.

---

## 5. Performance & Render Efficiency

### 5.1 `React.memo` on `CheckItem` Is Defeated by Inline Closures

**What:** `CheckItem` is wrapped in `React.memo` (line 988). However, it receives props created inline in the render function:
- `onToggle={() => toggleCheck(item.id)}` — new function every render
- `onAssign={u => setAssignments({ ...assignments, [item.id]: u })}` — new function every render
- `allChecks={checks}` — new object reference on every sync event

**Why it matters:** `React.memo` does shallow prop comparison. New function references and new object references fail the shallow check every time. Every `CheckItem` re-renders on every state change in the parent, regardless of whether the item's actual data changed. With 59 tasks and a timer ticking every second, that's 59 wasted re-renders per second.

**Fix:** Wrap `onToggle` and `onAssign` in `useCallback` at the `ChecklistView` level (keyed to item ID). For `allChecks`, pass only the relevant boolean `allChecks[item.id]` as a prop, not the full object. Alternatively, accept the re-renders (React 18's concurrent features handle this reasonably on modern phones) and only optimize if profiling shows actual jank.

### 5.2 Timer Causes Full App Re-Render Every Second

**What:** The timer's `setInterval` (line 2572) calls `setElapsed(e)` every second. `elapsed` is state in `ElimProductionApp` (the root component). Every state change in the root causes the entire component tree to re-render.

**Why it matters:** Combined with the defeated `React.memo` (5.1), the timer causes all 59 `CheckItem` components, the `Header`, the `DashboardView` or `ChecklistView`, and all their children to re-render every second. On a budget phone, this is measurable — each render cycle involves creating ~60 style objects, diffing ~60 DOM subtrees, and running the GC on the old objects.

**Fix:** Extract the timer display into a self-contained component with its own `setInterval` and local state. The root `ElimProductionApp` only needs to own the `timerState` (persisted data). The computed `elapsed` and `remaining` values can be derived locally inside the timer component. This isolates the per-second re-renders to the timer display only.

### 5.3 `ALL_ITEMS` Module-Scope Constant vs. Dynamic `allItems`

**What:** Line 616 computes `ALL_ITEMS` from `CHECKLIST_DATA` at module scope. Line 2725 computes `allItems` via `useMemo` from the dynamic `checklistData` state. `CheckItem` uses the module-scope `ALL_ITEMS` for dependency lookups (line 1001: `const depTask = ALL_ITEMS.find(t => t.id === unmet[0])`).

**Why it matters:** If tasks are added or removed via the in-app edit mode, the module-scope `ALL_ITEMS` is stale. Dependency lookups (for the "Dependency Not Complete" warning) won't find dynamically added tasks. This is a correctness bug, not just a performance issue.

**Fix:** Pass the dynamic `allItems` (or the full `checklistData`) as a prop to `CheckItem` for dependency resolution. Remove or repurpose the module-scope `ALL_ITEMS` to avoid confusion.

### 5.4 Style Objects Recreated in Render Functions

**What:** Most style objects are defined inline in JSX: `style={{ display: "flex", alignItems: "center", ... }}`. Each render creates new object allocations that are immediately eligible for garbage collection.

**Why it matters:** With 59 tasks × 5-10 inline style objects per task × re-renders every second (timer), that's 300-600 object allocations per second. On old phones with aggressive GC, this causes periodic frame drops (GC pauses).

**What's already done:** `CARD_BASE_STYLE` (line 232) and the `S` theme object (lines 216–230) are hoisted to module scope. This pattern is correct but inconsistently applied.

**Fix:** Hoist style objects that don't depend on dynamic values to module scope (e.g., the header style, section header style, checkbox style). Style objects that depend on computed values (e.g., `cardStyle` which depends on checked/assigned state) can stay inline but should use memoization or conditional spreading from hoisted base objects.

### 5.5 Initial Load: Photos Block Rendering

**What:** The `loaded` gate (line 2750) requires all 8 `useStorage` hooks to resolve before rendering: `const loaded = c1 && c2 && c3 && c4 && c5 && c6 && c7 && c8`. One of these is `c5` (photos). Photos are the largest payload.

**Why it matters:** On slow WiFi, if photos take 10 seconds to download, the user sees the loading spinner for 10 seconds even though all the data they actually need (checks, assignments, roster) loaded in 1 second. Photos are only needed when a user expands a task with photo slots.

**Fix:** Exclude photos from the initial load gate. Load checks, assignments, roster, etc. eagerly. Load photos lazily — either on first expand of a task with photos, or in the background after the main UI renders. This is already noted in ELIM-PROJECT.md line 405: "Photos excluded from initial load gate" — but the current code does NOT implement this. `c5` (photos loaded flag) is still part of the gate.

### 5.6 Memory Leaks: Properly Handled

**What:** All `useEffect` hooks that add event listeners return cleanup functions. `useBackSwipe` cleans up touch listeners. `useStorage` cleans up `elim-sync` listeners. The timer cleans up its `setInterval`. `AssignDropdown` cleans up click-outside listeners.

**Why it matters:** No action needed. This is well-implemented.

---

## Zero-Principles Fix List

Ranked strictly by: (1) data loss/corruption risk, (2) volunteer confusion/failure risk, (3) performance, (4) code quality.

| # | Finding | Category | Risk | Complexity | Ref |
|---|---------|----------|------|------------|-----|
| 1 | **`_pendingWrites` never cleared on final retry failure** — silent sync freeze for the failed key on that device | Data Loss | Critical | Low | 1.1 |
| 2 | **Offline writes use full-object `.set()` instead of field-level `.update()`** — concurrent offline edits silently overwrite each other | Data Loss | Critical | High | 1.2 |
| 3 | **I/O, Repairs, and Checklist edit modes are ephemeral** — edits lost on refresh/navigate, contradicts documented behavior | Data Loss | High | Med | 1.3, 4.1, 4.4 |
| 4 | **Photos stored as unbounded base64 blob on single Firebase key** — growing toward tab crash on mobile | Data Loss | High | High | 1.5, 2.6 |
| 5 | **No service worker — app is not offline-functional** — refresh in dead zone = blank page | Volunteer UX | High | Med | 1.7 |
| 6 | **No offline connection indicator** — volunteers check boxes thinking they're syncing when they aren't | Volunteer UX | High | Low | 3.6 |
| 7 | **No undo for any action** — accidental checkbox toggle, reassignment, or roster removal propagates instantly to all devices | Volunteer UX | High | Med | 3.2 |
| 8 | **Assignment dropdown always visible** — invites accidental reassignment during rushed scrolling | Volunteer UX | Med | Med | 3.4 |
| 9 | **Timer auto-start on 2nd checkbox** — confusing, not discoverable, hard to reverse | Volunteer UX | Med | Low | 3.3 |
| 10 | **Roster removal clears all assignments with minimal warning** — one tap destroys a session's worth of assignment work | Volunteer UX | Med | Low | 3.5 |
| 11 | **No data migration path for checklist structure changes** — orphaned data accumulates, assignments lost on reset after structural change | Data Loss | Med | Med | 1.4 |
| 12 | **`ALL_ITEMS` module-scope constant used for dependency lookup instead of dynamic `allItems`** — dependency warnings fail for dynamically added tasks | Correctness | Med | Low | 5.3 |
| 13 | **Photos block initial rendering** — `loaded` gate waits for photo download before showing any UI | Performance | Med | Low | 5.5 |
| 14 | **Timer re-renders entire app every second** — 59 CheckItem re-renders/sec due to root state update + defeated React.memo | Performance | Med | Med | 5.2 |
| 15 | **`React.memo` on CheckItem defeated by inline closures** — no memoization benefit, wasted comparison overhead | Performance | Low | Med | 5.1 |
| 16 | **Checkbox tap target 28px, below 44px minimum** — miss-taps during rushed setup | Volunteer UX | Low | Low | 3.7 |
| 17 | **PlaceholderView and dead routes still exist** — minor confusion, dead code | Code Quality | Low | Low | 4.2 |
| 18 | **Signal Flow SVGs hardcoded, dual source of truth with I/O data** — will silently drift as equipment changes | Code Quality | Low | Low | 4.3 |
| 19 | **`_ls.set()` swallows quota errors silently** — localStorage full goes unnoticed | Data Resilience | Low | Low | 2.3 |
| 20 | **`elim-sync` events not debounced** — rapid-fire updates from 16 devices cause render thrashing | Performance | Low | Low | 2.5 |
| 21 | **`newWeek` reset not fully atomic** — theoretical partial state on crash during reset (mitigated by pending-reset flag) | Data Resilience | Low | Med | 1.6 |
| 22 | **Inline style objects recreated every render** — GC pressure on old phones with timer running | Performance | Low | Med | 5.4 |
| 23 | **`JSON.parse(JSON.stringify())` used 8+ times without utility extraction** — readability, not correctness | Code Quality | Low | Low | 2.2 |
| 24 | **History entry delete button: tiny tap target (20px), no undo** — accidental deletion of historical data | Volunteer UX | Low | Low | 3.8 |
| 25 | **No data validation on load** — malformed Firebase data could cause render errors (caught by ErrorBoundary, but user sees crash screen) | Data Resilience | Low | Low | BACKLOG P4 |

---

*End of audit. No code changes included. Each fix should be implemented and tested individually, in priority order.*
