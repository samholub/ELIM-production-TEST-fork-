# ELIM Production App — Zero-Principles Audit Report
**v5.6.0 · 2,377 lines · Audited 2026-03-28**

---

## Acknowledged Once

- Firebase rules are fully open: intentional for a small trusted team.
- API key in source HTML: normal for Firebase client-side apps; keys are not secret.
- No authentication: intentional — volunteers select name on launch.
- Single-file architecture: intentional — zero build step, single deploy artifact.

---

## 1. Fragile Assumptions

### 1.1 Offline Conflict: Last Write Silently Wins — and the Loser Is Local Changes

**What it is.** When a Firebase write fails (network drop, venue WiFi hiccup), `window.storage.set` logs a console warning but returns successfully. `localStorage` has the new value. Firebase has the old value. The real-time listener (`_fb.child(key).on('value', ...)`) is still active, and when Firebase next propagates any state (from a different device's write, or just on reconnect), it fires `elim-sync` with its stale version. `useStorage`'s handler calls `setVal(JSON.parse(e.detail.value))` unconditionally — replacing React state with Firebase's older value. The locally-saved change is silently discarded.

**Why it matters.** A volunteer checks off "Power on Stage Box" at a venue with spotty WiFi. Firebase write fails. Thirty seconds later, the Firebase listener fires (triggered by another device's unrelated write), and the checkbox goes back to unchecked on their screen. They don't know if this is a real state or a sync artifact. They check it again. Or they walk away thinking it's done when it isn't. Under time pressure, this confusion compounds.

**Concrete fix.** After a failed Firebase write, mark the key as "pending sync" in memory (a simple `Set`). On any future `elim-sync` event for that key, if the key is still pending, compare timestamps or ignore the incoming event until the local write succeeds (add a retry with `setTimeout`). At minimum: if a Firebase write fails, re-attempt once after 5 seconds. If that fails too, surface a toast: "Changes saved locally — syncing when connection restores."

---

### 1.2 Firebase Re-seeding on Reconnect Can Clobber Newer Data

**What it is.** In `storage.get`, the fallback path reads:
```javascript
const cached = _ls.get('elim_' + key);
if (cached !== null) {
  if (_fb) _fb.child(fbKey).set(cached).catch(() => {});
  return { key, value: cached, shared: !!shared };
}
```
This path runs when Firebase fails (the outer `try/catch`). If Device A goes offline, makes changes (written to localStorage only), then comes back online, the very next `storage.get` call for that key — triggered by a navigation or re-mount — will push localStorage's value to Firebase, potentially overwriting changes Device B made while Device A was offline.

**Why it matters.** In practice, `useStorage` only calls `get` once on mount. After mount, updates come through the `elim-sync` listener. So this re-seed only fires at mount time (app load). If Device A goes offline, closes the app, makes changes (which can't happen if app is closed), and reopens... the changes weren't made, so no issue. The real scenario: Device A has an old localStorage cache from last week, goes to a new venue with a different network, Firebase is briefly unavailable on first load, localStorage cache is pushed to Firebase — overwriting all the changes made since last week. This is a data regression risk.

**Concrete fix.** Never push localStorage to Firebase unconditionally in `get`. Remove the re-seed push (`if (_fb) _fb.child(fbKey).set(cached)`) from the error fallback path. The only place localStorage should seed Firebase is intentionally, via explicit reset. If the app needs to recover stale localStorage data, make it opt-in, not automatic.

---

### 1.3 Checked Task Counter Overcounts After Task Deletion

**What it is.** When a task is deleted via edit mode, its ID is removed from `checklistData` but its entry in `elim3-checks` (and `elim3-assign`, `elim3-notes`, etc.) is never cleaned up. `totalChecked` is computed as `Object.values(checks).filter(Boolean).length` — it counts ALL checked IDs in the Firebase store, including deleted ones. `totalItems` counts tasks in the live `checklistData`. These two numbers can diverge.

**Why it matters.** After deleting two already-checked tasks: dashboard shows "61/59 tasks" and 103%. Progress bar overflows its container visually. Volunteers see impossible state and lose trust in the app.

**Concrete fix.** Compute `totalChecked` by intersecting with live task IDs: `const liveIds = new Set(allItems.map(i => i.id)); const totalChecked = Object.values(checks).filter((v, _, arr) => v).length` won't help — use `allItems.filter(i => checks[i.id]).length` everywhere totalChecked is needed. Also, on task deletion in `removeTask`, delete the orphaned entries from `checks`, `assignments`, `taskNotes`, and `completions`.

---

### 1.4 `newWeek` Reset Is Not Atomic — Partial State Is Possible

**What it is.** `newWeek` executes four sequential state saves: `setHistory`, `setChecks`, `setCompletions`, `setTimerState`. Each calls `window.storage.set` which writes localStorage synchronously but Firebase asynchronously. If the phone sleeps, loses connection, or the user force-closes the app between the first and fourth write, the state is partially reset. Specifically: history is saved (sealing the old week's data) but `elim3-checks` still contains all the old checkmarks.

**Why it matters.** On next load, `loaded` becomes `true` and the app renders with the new history entry (implying a fresh week started) but all 59 checkboxes still checked. The crew sees "100% complete" on what looks like a new week. They proceed under false confidence.

**Concrete fix.** Either: (a) write all four keys in a single JSON transaction — create a composite key `elim3-reset-state` containing the week boundary snapshot, and have the app interpret that on next load; or (b) add a `pendingReset: true` flag to Firebase before any clearing, and on load, if `pendingReset` is set, complete the reset before rendering. The simplest viable approach: batch the writes so they go to localStorage first synchronously and Firebase second, and treat any inconsistency as "complete the reset" rather than "undo the reset."

---

### 1.5 Base64 Photos Will Eventually Brick Initial Load

**What it is.** All photos across all time are stored under a single Firebase key `elim3-photos` as one JSON object: `{ "lw1-assembly": "data:image/jpeg;base64,...", "sb1-position": "data:image/jpeg;base64,...", ... }`. Every app load reads this entire blob. The `!loaded` guard holds the full UI until all 9 storage keys resolve — including photos.

A single photo at 800×600 JPEG 70% is roughly 80–150KB. Five photos per setup × 52 weeks = 260 photos × 100KB = ~26MB of base64 strings. That is a 26MB Firebase read on every app open, from a phone on venue WiFi.

Firebase Realtime DB has an undocumented but real per-node limit around 10–32MB for a single read. Once photos accumulate beyond that, `storage.get("elim3-photos")` will timeout or fail, and the app will be stuck on the loading spinner indefinitely.

**Why it matters.** This is the only finding that predicts a hard failure with no graceful degradation. The app will simply stop loading.

**Concrete fix.** Two changes: (1) Remove photos from the `loaded` gate — render the app without photos, lazy-load them per-task when a task is expanded. (2) Shard photos by week or year (e.g., `elim3-photos-2026`) so the current week's photos are a small key. Long-term: archive photos older than 4 weeks to a separate key that is never loaded on startup.

---

### 1.6 No Migration Path for `CHECKLIST_DATA` Structural Changes

**What it is.** `elim3-checklist-data` in Firebase, once populated, is only updated by: (a) in-app edit mode, or (b) a manual reset that clears checks/completions/timer. There is no version field, no schema hash, no way to detect that the code's `CHECKLIST_DATA` has changed since the last reset.

**Why it matters.** When new tasks are added to the code constant (e.g., the three tasks in BACKLOG.md Priority 2), deploying the code does nothing to live Firebase state. The team will continue using the old checklist, missing the new tasks, until someone deliberately resets. The reset clears all progress. If it happens mid-week, that's lost data.

**Concrete fix.** Add a `version` string to `CHECKLIST_DATA` (e.g., `"v5.6.0"`). Store the version inside `elim3-checklist-data`. On app load, after reading `checklistData`, compare its version to the code constant's version. If they differ, surface a banner: "Checklist structure has been updated. Reset to apply changes — or continue with the current structure." Let the admin (Sam) choose.

---

## 2. Data Resilience

### 2.1 `elim-sync` Fire-and-Forget Creates State Thrashing Under Rapid Multi-Device Edits

**What it is.** Every Firebase listener fires `window.dispatchEvent(new CustomEvent('elim-sync', ...))` directly and synchronously in the listener callback. The `useStorage` handler calls `setVal(JSON.parse(...))` on every event with no debouncing.

When Device A checks three boxes quickly: Firebase receives three writes, fires three listener callbacks (on Device B), dispatches three `elim-sync` events, triggers three `setVal` calls on `elim3-checks`, causing three sequential re-renders of the full app tree on Device B.

**Why it matters.** In React 18, `setVal` called inside native event handlers is batched, but `setVal` called inside `window.addEventListener` callbacks (async context) may not batch in all environments — particularly the CDN Babel/React 18 setup without the full scheduler. Under Babel standalone, React 18 auto-batching is less reliable. Three back-to-back re-renders of the full 2,000-line app tree with 59 task cards is measurable on an old phone.

**Concrete fix.** Debounce the `elim-sync` handler in `useStorage` by 50ms. Collect all arriving sync events for a given key and apply only the last one. This collapses three rapid writes into a single re-render with no observable delay to the user.

---

### 2.2 `localStorage` Full = Silent Data Loss

**What it is.** `_ls.set` catches `localStorage.setItem` errors silently:
```javascript
set(k,v) { try { localStorage.setItem(k,v); } catch(e) {} }
```
On iOS, `localStorage` quota is ~5-10MB. Each photo is added to the `elim3-photos` key, which is written to localStorage on every photo change. Once localStorage fills (entirely plausible with accumulated photos), all writes fail silently. `window.storage.set` still writes to Firebase, but the localStorage cache is stale. On next load (if Firebase fails or is slow), the app reads stale data.

**Why it matters.** Silent data loss. The volunteer takes a reference photo, sees it on screen, later the localStorage write fails, the photo disappears on next load. No indication of what happened.

**Concrete fix.** In `_ls.set`, catch the error and set a flag `window._lsFull = true`. In `useStorage`, check this flag and show a one-time toast: "Storage nearly full — photos may not persist offline." Not a perfect fix, but at least surfaces the condition.

---

### 2.3 `JSON.parse("null")` Path Can Cause Crashes

**What it is.** In `useStorage`:
```javascript
window.storage.get(key, shared).then(r => {
  if (r?.value) setVal(JSON.parse(r.value));
})
```
`r.value` is whatever comes back from `localStorage.getItem`. If Firebase stored `null` (which in Firebase means "node deleted") and that somehow got written to localStorage as the string `"null"` (which is truthy), then `JSON.parse("null")` = `null`. `setVal(null)` replaces e.g. `checks` with `null`. Any subsequent `Object.values(checks)` throws `TypeError: cannot convert undefined to object`.

**Why it matters.** Low probability but zero graceful handling. If `elim3-checks` gets set to `null` in Firebase (e.g., someone runs `firebase.delete()` on the database), the app crashes with a white screen on every device.

**Concrete fix.** Guard against null: `const parsed = JSON.parse(r.value); if (parsed !== null && parsed !== undefined) setVal(parsed);` — or use a nullish coalescing fallback: `setVal(JSON.parse(r.value) ?? initial)`.

---

### 2.4 Orphaned Data in Supporting Keys Accumulates Forever

**What it is.** Deleting a task in edit mode removes it from `checklistData` but leaves entries in `elim3-assign`, `elim3-notes`, `elim3-photos`, and `elim3-comp` for that task ID. Removing a person from the roster leaves their name as the value of any `elim3-assign` entries. Over time, both forms of orphan accumulate.

The assignment orphan is the more operationally painful one: removing "Taylor" from the roster still shows "Taylor" in the assignment dropdown for all tasks assigned to them (the `<select>` value shows it but it's not in the options list), causing the dropdown to appear blank/unset, looking like an unassigned task when it isn't.

**Concrete fix.** On task deletion, delete the corresponding keys from `checks`, `assignments`, `taskNotes`, `completions`. On roster member removal, reassign (or clear) all tasks assigned to that person's name before removing them.

---

## 3. Volunteer UX Failure Modes

### 3.1 Task Deletion Has No Confirmation — and Syncs Instantly

**What it is.** In edit mode, a ✕ button appears next to every task. `onClick={() => removeTask(section.id, item.id)}` — immediate, no confirm, no undo. The deletion syncs to Firebase within ~1 second, where it overwrites all other devices.

**Why it matters.** The single most dangerous volunteer action in the app. A confused volunteer taps the pencil icon, sees the ✕ buttons, and accidentally taps one. The task is gone on all 16 devices. Recovery requires: Sam knowing the original task text from memory, re-creating it in edit mode, and re-syncing. If it was a task with notes or a reference photo, those are also gone.

**Concrete fix.** Add a `window.confirm("Delete this task? This can't be undone.")` before `removeTask`. Crude, but it adds one confirmation for a destructive action. Better: add a 5-second undo toast that defers the Firebase write. Best (but requires more code): restrict edit mode entry — move it behind Settings or a passcode, removing it from the main checklist view entirely.

---

### 3.2 Edit Mode Is Visible and Accessible to All 16 Volunteers

**What it is.** The ✏️ button is always rendered in `ChecklistView` because `setChecklistData` is always passed from `ElimProductionApp`. There is no admin check, no hidden feature, no long-press. Any volunteer can tap it.

**Why it matters.** This is a combination of the above risk multiplied by 15. The pencil icon is adjacent to the search bar — a tool that looks generically useful, not like "admin mode for the app owner." A volunteer casually tapping the screen near the search bar could enter edit mode and see ✕ delete buttons on every task.

**Concrete fix.** Condition the edit button on a check: `settings.adminMode` in localStorage (set from Settings), or simply a long-press on the section header, or just move the button entirely into Settings. The one-line fix: in `ChecklistView`, add `{setChecklistData && isAdmin && (...)}` where `isAdmin = currentUser === "Sam"` (the app owner). Acknowledge this is a weak gate but it prevents accidental entry by the other 15 users.

---

### 3.3 Assignment Dropdown Is Always Visible and Easy to Mis-Tap

**What it is.** Every `CheckItem` renders a `<select>` regardless of assignment state. On a phone in landscape or with slightly mis-aimed thumbs, a volunteer checking off a task can inadvertently open the dropdown instead of the checkbox (the checkbox is 28px; the dropdown is right below it, padding: "4px 10px").

**Why it matters.** Accidental reassignment of a task syncs instantly to all devices. Tyler sees "Setup mics" disappear from his task list because someone accidentally reassigned it. There's no notification, no undo, and the person who caused it probably doesn't know it happened.

**Concrete fix.** Show assignment as plain text (non-interactive) when the task is already assigned. Only render the `<select>` when: the task is unassigned, or the user explicitly taps an "edit assignment" link. This is backlog item already, but it's urgent enough to call out here as a data integrity risk, not just a UX nicety.

---

### 3.4 Dependency Warning Modal Is Unreadable

**What it is.** The backdrop is `rgba(0,0,0,0.6)`. The modal card background is `S.card = "rgba(255,235,210,0.03)"` — nearly transparent. Both the backdrop and the card are translucent, so checklist text from behind bleeds through the warning text.

**Why it matters.** A volunteer triggers a dependency warning. They can't read what it says. They tap "Continue" (it's the more prominent button) because they don't know what else to do. The warning serves no purpose.

**Concrete fix.** Change the modal card background to a solid color: `background: "#1C1A18"` (dark opaque). The backdrop can stay translucent. This is a one-line fix.

---

### 3.5 Populated Notes Are Invisible Until Expanded

**What it is.** `CheckItem` shows a 📌 emoji if `taskNote` exists and the task is not expanded. That's it. There is no preview, no indication of what the note says, no surface-level visibility.

**Why it matters.** Notes are the primary async communication channel for the team. "Left WiFi router at venue — it's in the storage cabinet." A volunteer sets up the WiFi router task, doesn't expand it, doesn't see the note, can't find the router. The note exists, it's just invisible.

**Concrete fix.** Render the first 60 characters of `taskNote` as a one-line preview below the assignment dropdown when `taskNote` exists and the task is not expanded. This is already in the backlog; note it's medium-severity here because it actively suppresses important crew communication.

---

### 3.6 First-Time Volunteer Can't Determine What to Do in Under 3 Seconds

**What it is.** The app opens to a dashboard showing: a percentage badge, a timer with "▶ Tap to start," a "Your Tasks" card showing 0/0 (if unassigned), and a crew status list. There is no instruction, no onboarding prompt, no indication of where to begin. The "Your Tasks" card showing 0 remaining looks like there's nothing to do.

**Why it matters.** New volunteers (turnover is explicitly noted) will open the app, see "0 remaining" on their task card, and assume they're done — or ask the team lead for help, consuming attention during the highest-pressure moment of setup.

**Concrete fix.** If `myItems.length === 0`, change the "Your Tasks" callout text from "0 remaining" to "No tasks assigned yet — See checklist to assign yourself tasks." Or better: surface the top-level onboarding step on the dashboard: a prompt visible when `myItems.length === 0` that says "Head to Settings to assign tasks, or tap the Checklist to assign yourself."

---

### 3.7 Timer Auto-Starts on First Checkbox — Undiscoverable and Sticky

**What it is.** `toggleCheck` calls `if (!running) setRunning(true)`. The first checkbox checked starts the global timer. This write syncs to Firebase immediately. The timer cannot be "un-started" short of pausing it from the dashboard. An accidental check-then-uncheck starts the timer with no way for the volunteer to know that happened.

**Why it matters.** Setup history records `elapsed` time. If the timer was started accidentally before real setup began (during assignment/planning phase), the history entry shows a longer setup time than reality. Over 52 weeks, this corrupts the historical trend data.

**Concrete fix.** Don't auto-start. Remove the `if (!running) setRunning(true)` from `toggleCheck`. Let the timer be a manual, deliberate action from the dashboard or header. The timer is synced across devices anyway — the team lead can start it when setup actually begins.

---

## 4. The "Fake Feature" Check

### 4.1 Equipment and Purchases Nav Cards Are Dead Ends

Two of the six nav cards (`inventory`, `purchases`) render `PlaceholderView` with "coming in a future update." They occupy 33% of the nav grid surface area. During a rushed setup, a volunteer who taps Equipment or Purchases and hits a dead end loses confidence in the app and may stop using the nav grid entirely.

**Fix:** Remove both cards and the `PlaceholderView` component. Reduce the grid to 2×2. When Equipment and Purchases are actually built, add them back.

---

### 4.2 Checklist Search Bar Is Always Visible for a 59-Item List

The search bar occupies the full width below the progress bar on every checklist load. With smart auto-expand showing only the sections that have your tasks (typically 3-5 sections, 15-20 tasks), search is redundant for 15 of 16 users in normal operation.

**Why it's a liability:** It takes up 42px of vertical space on a phone, pushing the first section header further below the fold. It creates visual noise at the top of the most-used screen. Its presence implies the checklist is hard to navigate — which is the wrong signal to send about a 59-item list with collapsible sections.

**Fix:** Replace with a search icon (🔍) that reveals the search input on tap. The icon can live in the same row as the edit button, so no space is wasted.

---

### 4.3 Signal Flow SVGs Will Silently Drift Out of Sync

The three SVG diagrams (~200 lines of hand-placed `<Box>`, `<Arrow>`, `<Label>` elements) are a maintenance liability, not just a static reference. Equipment changes in v5.5.2 alone required updates to all three diagrams and the I/O list and multiple checklist tasks — and OPEN-ITEMS.md still documents four unknowns (ATEM SDI port assignments, Resi encoder connection, Omar/Mitch Mac ATEM connection, full Dante routing) that mean the diagrams may be wrong right now.

**Why it matters:** A volunteer references the Audio signal chain SVG to troubleshoot a Dante routing issue. The SVG shows a path that no longer reflects the actual setup. They follow the wrong diagnostic steps. The SVG creates false confidence.

**Fix:** Add a "Last updated" date stamp to each SVG section header. When equipment changes in code, updating the SVG timestamp becomes a deliberate act that forces the author to acknowledge the diagram needs review. Longer term: replace with reference photos taken on-site, which self-certify their own accuracy.

---

### 4.4 "Last Setup" Stat on Dashboard Hero Is a Post-Mortem Feature Shown During Setup

```javascript
{history && history.length > 0 && (
  <div style={{ marginTop: 10, fontSize: 12, color: S.textMuted }}>
    Last: {history[0].date} — {history[0].pct}% in {formatTime(history[0].elapsed)}
  </div>
)}
```
This renders below the timer during the active setup window. "Last: 2026-03-21 — 100% in 1:12:34" is information that serves the team lead reviewing trends, not a volunteer setting up cables at 8am.

**Fix:** Remove this line from the dashboard timer zone. The data already lives in Settings → Setup History.

---

## 5. Performance & Render Efficiency

### 5.1 Full App Re-Renders Every Second Due to Timer Interval

**What it is.** `setElapsed(e)` is called inside `setInterval(compute, 1000)` in `ElimProductionApp`. This triggers a re-render of `ElimProductionApp` every second, which re-renders every currently-mounted child. When the checklist is visible, that means 59 `CheckItem` instances re-render every second.

Each `CheckItem` render:
- Executes the `cardStyle` IIFE (conditional logic + object construction)
- Recreates all inline style objects
- Evaluates `isMine`, `isUnassigned`, and `hasExpandable` conditions

On an iPhone 12 Pro this is unnoticeable. On a mid-range 2019 Android, rendering 59 cards per second is measurable jank.

**Concrete fix.** Move `elapsed` and `remaining` out of the prop chain for `ChecklistView`. The checklist doesn't display the timer inline — it only passes `running`/`setRunning` to start it. Only the `Header`'s `rightContent` and `DashboardView` need live timer values. Extract a `TimerDisplay` component that reads timer state directly, preventing the cascade.

Alternatively: `React.memo` on `CheckItem` and pass only stable props. The `cardStyle` should be memoized with `useMemo`.

---

### 5.2 `allItems` Recomputed Every Render With No Memoization

**What it is.** In `ElimProductionApp`:
```javascript
const allItems = checklistData.flatMap(s => s.items.map(i => ({ ...i, section: s })));
```
This runs on every render. Given the timer fires every second, `allItems` (an array of 59 objects) is recreated every second. It's passed to `DashboardView` and used as `effectiveAllItems`, which then runs crew stats computation inline:
```javascript
effectiveAllItems.forEach(i => { ... });
const sortedPeople = Object.entries(people).sort(...);
```
Both run on every second.

**Concrete fix.** `useMemo` on `allItems`:
```javascript
const allItems = useMemo(
  () => checklistData.flatMap(s => s.items.map(i => ({ ...i, section: s }))),
  [checklistData]
);
```
This makes `allItems` stable across timer re-renders, and crew stat computation only runs when the checklist structure actually changes.

---

### 5.3 Firebase Writes on Assignment Changes Propagate Entire Object

**What it is.** `setAssignments({ ...assignments, [item.id]: u })` writes the entire `assignments` object to Firebase on every assignment change. With 59 tasks all assigned, that's a JSON blob of ~59 key-value pairs written in full every time one assignment changes. Same for `checks`, `completions`, `taskNotes`.

**Why it matters.** For `taskNotes` specifically, if a volunteer types a multi-paragraph note and saves it, the entire notes object (all notes for all tasks) is written to Firebase. This is fine for the current scale but becomes the wrong pattern as note content grows.

**Concrete fix.** This is architectural and low priority for now. Flagging it because: the current pattern means that when two devices are simultaneously editing different tasks' notes, they will overwrite each other's changes (last-write-wins on the entire object). Mitigating this requires per-task keys (`elim3-notes.{taskId}`) which is a larger refactor. For now, document the known limitation.

---

### 5.4 Initial Load Is Gated on All 9 Storage Keys Including Photos

**What it is.** `const loaded = c1 && c2 && c3 && c4 && c5 && c6 && c7 && c8 && c9;` — the app renders a loading spinner until all 9 Firebase reads complete. `elim3-photos` is one of those 9. If photos have accumulated to 5MB+, the initial load is blocked until a multi-megabyte Firebase read completes, on a phone, on venue WiFi.

**Why it matters.** Setup day is the worst time to have a slow app load. A volunteer opens the app 5 minutes before setup starts and stares at a spinner for 8 seconds because of reference photos from 3 months ago.

**Concrete fix.** Remove `c5` (photos loaded flag) from the `loaded` gate. Photos can be fetched lazily when tasks are expanded. The app should be fully interactive without photos loaded — they're non-blocking reference material.

---

### 5.5 `cards` Array Recreated Every Render in `DashboardView`

**What it is.** In `DashboardView`:
```javascript
const cards = [
  { id: "power-seq", icon: "⚡", title: "Power Sequence", subtitle: "Equipment order" },
  ...
];
```
This array of 6 static objects is recreated on every render. `DashboardView` re-renders on every timer tick. The `cards` array is never used outside `DashboardView`.

**Concrete fix.** Hoist to module scope — it's a constant. Place it immediately below `POWER_SEQUENCE`. Zero runtime cost, trivial change.

---

## Zero-Principles Fix List

Ranked strictly by (1) data loss/corruption risk, (2) volunteer confusion/failure risk, (3) performance, (4) code quality.

| # | Finding | Category | Risk |
|---|---------|----------|------|
| 1 | Task deletion has no confirmation — instant sync, no undo | Data loss | **Critical** |
| 2 | Failed Firebase writes are silently discarded when listener fires | Data loss | **Critical** |
| 3 | Checked task counter overcounts after task deletion (shows 103%) | Data corruption | **High** |
| 4 | `newWeek` reset is non-atomic — partial state on interruption | Data corruption | **High** |
| 5 | Base64 photos block initial load and will eventually breach Firebase read limits | Data loss (future) | **High** |
| 6 | localStorage re-seed on reconnect can overwrite newer Firebase data | Data loss | **High** |
| 7 | `JSON.parse("null")` crash path — white screen on all devices | App crash | **High** |
| 8 | Edit mode accessible to all 16 volunteers, not just admin | Volunteer confusion + data loss | **High** |
| 9 | Dependency warning modal unreadable — volunteers ignore or misread it | Volunteer confusion | **Medium** |
| 10 | Assignment dropdown always visible — easy accidental reassignment | Volunteer confusion | **Medium** |
| 11 | Populated notes invisible without expanding — suppresses crew communication | Volunteer confusion | **Medium** |
| 12 | Timer auto-starts on first checkbox — corrupts setup time history | Data corruption (minor) | **Medium** |
| 13 | First-time volunteer sees "0 remaining" and has no next step | Volunteer confusion | **Medium** |
| 14 | Orphaned data accumulates in supporting keys after task/member deletion | Data creep | **Medium** |
| 15 | `localStorage` full → silent data loss on photo writes | Data loss (future) | **Medium** |
| 16 | Equipment and Purchases nav cards lead to dead ends | Fake feature / confusion | **Low** |
| 17 | Signal Flow SVGs will drift out of sync silently | Fake feature / false confidence | **Low** |
| 18 | "Last setup" stat on dashboard hero is post-mortem noise | Visual noise | **Low** |
| 19 | Checklist search bar always visible — wastes space, implies difficulty | Visual noise | **Low** |
| 20 | Full app re-renders every second due to timer interval | Performance | **Low** |
| 21 | `allItems` recomputed every timer tick — no `useMemo` | Performance | **Low** |
| 22 | Photos block initial load gate — should be lazy | Performance | **Low** |
| 23 | `elim-sync` events not debounced — potential rapid re-renders | Performance | **Low** |
| 24 | `cards` array in `DashboardView` recreated every render | Code quality | **Trivial** |
| 25 | `CHECKLIST_DATA` has no version field — no migration path for structural changes | Maintainability | **Low** |
| 26 | Whole-object Firebase writes mean concurrent note edits overwrite each other | Known limitation | **Low** |
