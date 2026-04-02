# ELIM Production App

Portable church production setup/teardown checklist app for ELIM Church (The Well / The ELIM Arrival). Progressive Web App built with React, Firebase Realtime Database for cross-device sync, offline-capable, no authentication required.

---

## Purpose

The app solves three problems that get worse as the team grows: (1) verification at scale — Sam can't physically watch every setup task across the venue while operating the ATEM and managing the stream, (2) knowledge transfer — as volunteers rotate in and out, the system of record for what connects to what, in what order, and why needs to live somewhere other than people's heads, and (3) consistency across venues — the checklist enforces the same standard regardless of room layout, power situation, or which crew members are present.

---

## Current Status: v5.11.0

**Live URL:** https://elim-production.web.app
**Test URL:** Replit preview (connected to production Firebase)
**Test repo:** https://github.com/samholub/ELIM-production-TEST-fork-

### What's Built
- **15 sections, 62 tasks** covering full setup flow from Pre-Setup through Post-Event
- **Photo documentation** — camera capture or upload per task, auto-resized, persistent
- **Troubleshooting guides** — expandable Q&A per task for common issues
- **Per-task notes** — persistent, shared across team; preview shown above the fold (hidden on checked tasks)
- **Task detail surfacing** — static reference text shown below assignee, up to 5 lines with word-break (hidden on checked tasks)
- **Custom assignment dropdown** — dark-themed dropdown replacing native select; click-outside-to-close, unassign option
- **Countdown timer** — configurable (30/45/60/90 presets or custom) with green→yellow→red transition, auto-pause at zero, syncs across devices
- **Timer auto-stop** — timer automatically pauses when all non-post-event tasks reach 100%; one-way gate per session (unchecking doesn't restart)
- **Manual "Setup Complete"** — button in Settings stops timer and logs current progress to history without clearing any checkboxes or assignments; two-tap confirm
- **Post-event timer exclusion** — Post-Event section tasks are excluded from dashboard progress percentage and timer calculations but remain visible and functional in the checklist. ChecklistView counts all tasks including post-event for its own progress bar (intentional asymmetry).
- **Weekly reset** — clears checkboxes/timer, preserves assignments/notes/photos; saves history (last 52 weeks); atomic with pending-reset flag
- **SVG brand mark** — geometric `ElimLogo` component (right-pointing chevrons + block E)
- **Signal Flow diagrams** — three static SVG diagrams (Audio, Video, Network/Dante) with color-coded sections
- **Crew focused views** — tap a crew member name on dashboard → focused task list
- **Unassigned task awareness** — dashboard row, dashed/italic styling, section header counts
- **Smart auto-expand** — sections with your incomplete tasks or in-progress auto-expand
- **Crew status dashboard** — per-person progress list (you sorted first) + unassigned row
- **Timer in header** — visible on Checklist, My Tasks, and Crew Tasks views
- **My Tasks collapsible sections** — completed sections auto-collapse to a single green ✓ header line; tap any header to toggle; manual override remembered per session
- **Follow-finger swipe-back gesture** — real-time content tracking, 35% threshold, header zone exclusion
- **Power sequence** — 10-step equipment power order with teardown reminder; badges on task cards
- **Persistent I/O Line List** — 3 tabs (Stage Box, Wing, Dante) organized by physical device; Stage Box has Inputs/Outputs sub-toggle; inputs grouped into 6 channel-range sections (Drums, Instruments, Talk Backs, Vocals, Wireless, Crowd) with "Other" fallback; outputs split into IEM vs Infrastructure groups; in-ear snake pill badge (🎧); cross-tab search across all sections; draft-based editing with confirm-on-save modal; edit restricted to Sam and Bri; custom IODropdown components replacing native selects; persisted via `useStorage` and synced to all devices
- **Repairs tracker** — Firebase-persisted with full CRUD, search, priority levels
- **Checklist task editing** — search toggle (🔍 icon), edit mode (Sam only) with add/remove/edit tasks inline
- **View stack navigation** — proper breadcrumb-style back navigation with scroll position preservation (retry-based restore for async-loaded views)
- **Default My Tasks view** — app opens to My Tasks; dashboard accessible via back navigation
- **Error boundary** — render crashes show "Tap to Reload" instead of permanent white screen
- **Dependency warnings** — prerequisite tasks must be complete (amber modal, user can always proceed)
- **Dynamic roster** — add/remove/reorder team members in Settings; blast-radius warnings on removal (shows affected assignments)
- **Remember-me** — device remembers last selected user
- **Real-time cross-device sync** — Firebase Realtime Database with localStorage offline fallback, smart prevRef diffing to prevent stale overwrites
- **Offline awareness** — Firebase `.info/connected` listener shows warning banner when device is offline
- **Resilient storage** — graceful degradation: Firebase → localStorage → in-memory
- **Firebase write retry** — failed writes tracked via `_pendingWrites` Set and retried with exponential backoff (4 retries, 2s→30s); incoming sync events ignored for keys with pending writes
- **Safe reconnection** — `storage.get` fallback path does not push localStorage back to Firebase, preventing stale cache from clobbering newer remote data
- **Edit mode restricted** — checklist edit mode (task add/delete) only visible to Sam; I/O edit mode visible to Sam and Bri
- **Task deletion confirmation** — `confirm()` dialog before removing any task
- **Atomic weekly reset** — pending-reset flag prevents partial state on interruption
- **Orphan cleanup** — task deletion and roster removal clean up all supporting data keys (checks, assignments, notes, completions)

---

## Design Philosophy: Warm Night

- **4 visual states:** Orange = yours, Green = done, Dashed/italic/dim = unassigned, Neutral = everything else
- **No per-person colors** in the main UI (only in Settings roster management)
- **Readable at a glance:** 16px task text, 28px checkboxes (44px tap targets), solid triangle expand arrows — mobile thumb-sized
- **Quiet by default:** collapsed sections with separator lines, minimal badges, no timestamps
- **Your tasks stand out:** orange left border + "YOU" badge on sections with your incomplete work
- **Unassigned tasks stand out:** dashed border, lighter background, italic dimmed text
- **Crew focused views:** tap a name → see just their tasks, not the full checklist
- **Progressive disclosure:** details, photos, troubleshooting hidden until expanded
- **Calm urgency:** countdown timer uses color transition (green→yellow→red), not flashing

---

## Data Management

### Storage Architecture

The app uses a three-tier storage system with Firebase Realtime Database as the primary store, localStorage as a sync cache, and in-memory state as the final fallback.

**Write path:** React state update → localStorage cache (immediate) → Firebase write (async with retry). If the Firebase write fails, the key is added to `_pendingWrites` and retried with exponential backoff (4 retries, 2s initial delay, max 30s). If all retries fail, the pending flag is removed so the listener can resume, and a sync error indicator appears.

**Read path (on mount):** Try Firebase first → fall back to localStorage cache → throw if neither exists. The localStorage fallback does **not** push its value back to Firebase — this prevents stale cache from clobbering newer remote data written by other devices while this device was offline.

**Sync path (real-time):** Firebase `.on('value')` listeners fire `elim-sync` CustomEvents consumed by the `useStorage` hook. Before applying an incoming sync event, the listener checks `_pendingWrites` — if the key has a local write in flight, the incoming event is ignored to prevent stale Firebase data from reverting the user's local change.

**Offline awareness:** A Firebase `.info/connected` listener monitors connectivity. When the device goes offline, a warning banner appears at the top of the app. All writes continue to localStorage immediately and will reach Firebase when connectivity restores.

**Smart diffing:** The `useStorage` save function compares the new value against a `prevRef` to determine whether to use `storage.merge` (field-level atomic update for partial object changes) or `storage.set` (full replacement). This minimizes Firebase writes and reduces conflict surface.

### Draft-Based Editing Pattern

Used by IOListView for the I/O Line List:
1. ✏️ pencil button (Sam and Bri only) clones live state into a local draft (`useState`)
2. All user modifications target the draft, not live state
3. "✓ Done" opens a confirm modal: "Save I/O changes? This syncs to all devices."
4. "Save" writes draft → `useStorage` → localStorage + Firebase → syncs to all devices
5. "Discard" / "Cancel" drops the draft, reverts UI to live state

This pattern prevents half-finished edits from syncing to other devices mid-edit.

### Firebase / Code Sync Strategy

The hardcoded constants in the code (`CHECKLIST_DATA`, `DEFAULT_IO_LINE_LIST`, `DEFAULT_REPAIRS`) are the **canonical baseline**. Firebase is the live store.

**How it works:**
- First app load (or after reset) → seeds Firebase from code constant
- After that → Firebase is the live data store
- If Firebase has data, the code constant is ignored

**Workflow:**
1. **Structural changes go through code first** — update the constant, deploy, then reset to pull in new baseline
2. **In-app editing is for live adjustments** — Sunday morning task description fix, quick I/O label change
3. **After any in-app edit worth keeping** — mirror it back to the code constant

**⚠️ Important:** In-app edits are stored in Firebase. If you deploy new code and reset, the code constant replaces Firebase data. Any in-app edits not mirrored to code will be lost on reset. Always mirror live edits to the code constant before the next deploy + reset cycle.

**Checklist data specifics:** The cleanup `useEffect` on app load actively deletes `elim3-checklist-data` from Firebase and localStorage, forcing re-seed from the `CHECKLIST_DATA` code constant on every load. This means in-app checklist edits are ephemeral — they persist in Firebase until the next app load or deploy. This is intentional: checklist structure changes should go through the code.

**I/O Line List specifics:** The I/O config is persistent via `useStorage("elim3-io-config", DEFAULT_IO_LINE_LIST)`, owned by `ElimProductionApp` and passed as props to `IOListView`. Unlike checklist data, I/O edits persist across loads and are NOT deleted on app load. The cleanup `useEffect` was updated in v5.10.0 to stop deleting `elim3-io-config`.

### Storage Keys
| Key | Content | Persistence |
|-----|---------|-------------|
| `elim3-checks` | Checkbox states | Cleared on reset |
| `elim3-assign` | Task assignments | Preserved |
| `elim3-comp` | Completion metadata | Cleared on reset |
| `elim3-notes` | Per-task notes | Preserved |
| `elim3-photos` | Photo data (base64) | Preserved |
| `elim3-roster` | Team roster | Preserved |
| `elim3-timer` | Timer state | Cleared on reset |
| `elim3-history` | Setup history (last 52 weeks) | Preserved |
| `elim3-io-config` | I/O line list (persistent, draft-based editing) | Preserved |
| `elim3-repairs` | Repairs list | Preserved |
| `elim3-checklist-data` | Checklist structure — deleted on load, re-seeded from code | Ephemeral |
| `elim3-pending-reset` | Transient flag for atomic reset (deleted after reset completes) | Transient |

### What Resets Clear vs Preserve
| Cleared on reset | Preserved across resets |
|---|---|
| Checkboxes | Assignments |
| Completions | Notes |
| Timer | Photos |
| | Roster |
| | I/O config |
| | Repairs |

---

## File Structure

```
workspace/                        ← Replit workspace (primary dev environment)
├── index.html                    ← Production app (single file, Firebase + React + app code)
├── firebase.json                 ← Firebase Hosting config
├── .firebaserc                   ← Firebase project link
├── database.rules.json           ← Realtime Database rules
│
├── (Claude project files)
│   ├── ELIM-PROJECT.md           ← This file — project overview, tech reference, deployment
│   ├── CHANGELOG.md              ← Version history
│   ├── TESTING.md                ← Pre-deployment testing checklist
│   ├── BACKLOG.md                ← All planned work, organized by category
│   ├── OPEN-ITEMS.md             ← On-site investigations, open questions, unknowns
│   └── HARDWARE-RUNBOOK.md       ← Physical production reference — network config, packing, cables
│
└── (GitHub backup: github.com/samholub/ELIM-production-TEST-fork-)
```

---

## Deployment

**Hosted on:** Firebase Hosting (free Spark plan)

**Development environment:** Replit (sole dev environment — live preview, terminal, Firebase CLI all in one place)
**Source backup:** GitHub (push once per version, not per edit — `git add . && git commit -m "v5.x.0" && git push`)

**To redeploy after changes:**
1. Edit `index.html` in Replit
2. Run `firebase deploy --only hosting` in Replit terminal
3. Live in ~10 seconds at the same URL

Firebase CLI is authenticated in Replit under `sammholub@gmail.com` with project `elim-production` linked.

**Data safety:** Redeploying only replaces `index.html`. All team data lives in Firebase Realtime Database and is never affected by code deployments.

**Firebase project:** `elim-production`
- Console: https://console.firebase.google.com/project/elim-production
- Database: https://elim-production-default-rtdb.firebaseio.com
- Database rules: open read/write (no auth required, permanent)

---

## Dashboard Layout
- **Header:** Logo (left) · Centered username with ▾ tap-to-switch · ⚙️ gear (right)
- **Hero:** Stacked progress zone (tappable → full checklist) + timer zone (with picker)
- **Your Tasks:** Card linking to My Tasks with remaining count
- **Crew Status:** Per-person progress rows, you sorted first, "Unassigned tasks" row at bottom
- **Nav Cards:** 1×3 grid — I/O Line List, Power Sequence, Signal Flow

## Settings Layout
- **Reset Checklist** (top) — two-tap confirmation
- **Setup Complete** — stops timer and saves progress to history without clearing checkboxes
- **Team Roster** — add/remove members with color assignment; removal shows blast-radius warning
- **Setup History** — last 52 sessions with date, progress bar, percentage, time
- **About** — app version and description

---

## Checklist Sections & Power Sequence

### Sections (in order)
1. 🙏 Pre-Setup (3 tasks)
2. 🖥️ LED Wall Setup (3 tasks)
3. 🎤 Stage Box Setup (6 tasks)
4. 🎛️ Wing Soundboard Setup (7 tasks)
5. 🌐 Dante Network Setup (6 tasks)
6. 🎵 ProTools Mac Setup (3 tasks)
7. 🖥️ Display Setup (3 tasks)
8. 📹 Camera Setup (7 tasks)
9. 💻 ProPresenter Mac Setup (5 tasks)
10. 📡 Streaming Setup (4 tasks)
11. ⚡ Final Power-On (4 tasks)
12. 🎙️ Mic Check (3 tasks)
13. 🔊 Sound Check (4 tasks)
14. ✅ Pre-Service Checks (4 tasks)
15. 📦 Post-Event (3 tasks)

### Power Sequence (on → 1-10, off → 10-1)
1. Stage Box
2. Wing Sound Board
3. WiFi Router
4. Dante Switch / Dante AVIO
5. ProTools Macs (Tyler + Omar/Mitch)
6. Confidence Monitor
7. ProPresenter Mac
8. Streaming Camera
9. LED Transmitter
10. Speakers

---

## Equipment Reference

### Audio / Stage

- **Stage Box**: Midas DL251 — connects to Wing AES50 Supermac port A; 44 input channels + output jacks for IEM packs and speakers (routed from Wing via AES50)
- **Soundboard**: Behringer Wing (with Dante Expansion card) — Wing ethernet left port to WiFi Router; Dante Expansion to port 5 on switch
- **Network Switch**: 5-Port Switch (Dante network) — port 5 = Wing Dante Expansion
- **WiFi Router**: TP-Link AX55 — ATEM/Mac control network + iPad wireless control. Router on 192.168.0.x subnet.
- **Bianca's IEM**: Wing XLR Output 8
- **Wing XLR Input 8**: Bianca Talk Back

### Video / ATEM

- **Video Switcher**: ATEM Television Studio Pro 4K — static IP 192.168.0.240; ethernet to AX55 LAN port for remote control; IEC C14 power (internal PSU)
- **ATEM SDI Inputs**: IN 1 = BlackMagic wide-angle (fixed), IN 2 = Hollyland wireless receiver (handheld), IN 3 = DJI camera, IN 4 = BlackMagic 6K, IN 8 = UltraStudio Monitor SDI out (from ProPresenter Mac). Inputs 5-7 unused/unknown.
- **ATEM SDI Outputs**: OUT 8 = Resi SDI IN. Outputs 1-7 unknown/unused.
- **UltraStudio Monitor**: Blackmagic — SDI in from ATEM SDI 8; USB-C out to ProPresenter Mac
- **Anker USB Hub**: ATEM HDMI control → hub → USB-C → ProPresenter Mac
- **Hollyland Receiver**: Wireless camera receiver — SDI out to ATEM SDI IN 2
- **SDI/HDMI Converter**: 1× at production booth, short SDI cable to ATEM

### Streaming

- **Resi Encoder**: Streaming encoder — lives in Wing case; SDI in from ATEM SDI OUT 8; ethernet direct to venue internet (NOT through router — separate network)

### Computers

- **Tyler's Mac**: ProTools for live recording — USB-C direct to Wing (NOT on Dante network)
- **Omar/Mitch's Mac**: ProTools for stream mixing — CAT6 to 5-Port Switch (Dante)
- **ProPresenter Mac** (Bianca operates; historically labeled "Glenn's Mac"): ProPresenter for video/presentation — ethernet to AX55 LAN port (192.168.0.100, static); HDMI to ATEM via Anker hub; USB-C from UltraStudio; HDMI to Stage Box 2 (LED wall transmitter); USB-C to Confidence Monitor. WiFi active simultaneously for internet, prioritized above ethernet in macOS service order.

### Display

- **LED Wall**: 9×5 panels (45 total) via LED Transmitter from Stage Box 2
- **Stage Box 2**: Production box — ProPresenter Mac HDMI in, LED Transmitter out
- **Confidence Monitor**: Stage display from ProPresenter Mac (USB-C)

### Control

- **Wireless Control**: WiFi Router (AX55) → iPad (Wing control) + phone (ATEM control via ATEM Software Control at 192.168.0.240)

---

## Network Architecture (Three Separate Networks)

The production setup uses three independent networks that do not interconnect. See **HARDWARE-RUNBOOK.md** for full configuration details, IP addresses, and revert procedures.

| Network | Devices | Purpose |
|---------|---------|---------|
| **Dante** (5-port switch) | Wing Dante Expansion, Omar/Mitch's Mac, ProPresenter Mac (Dante audio), AVIO | Audio routing |
| **Production Control** (AX55 router, 192.168.0.x) | ATEM (192.168.0.240 static), ProPresenter Mac (192.168.0.100 wired), Phone (DHCP WiFi), Wing (WiFi) | ATEM remote control, Wing iPad control |
| **Internet** (direct line) | Resi encoder | Streaming to CDN |

**Key detail:** The ProPresenter Mac is on both the Dante network (CAT6 to switch for audio) and the Production Control network (ethernet to AX55 for ATEM control) simultaneously.

**Troubleshooting:** "Why can't X see Y?" → Check which network each device is on. Dante devices can't see internet devices. ATEM control devices can't see Dante audio. The Resi is isolated by design.

---

## Team Members
Managed dynamically via Settings. Default roster: Glenn, Sunny, Omar, Mitch, Michael, Taylor, Seth, Sam, Bianca, Bri, Leo, Austin, Tyler, Kolton, Savanah, Emily

---

## Technical Notes

- **Framework:** React 18 (via CDN, no build step)
- **Styling:** Inline styles, no CSS framework
- **Fonts:** DM Sans (body) + JetBrains Mono (data/timers) via Google Fonts
- **Storage:** Firebase Realtime Database with `window.storage` API wrapper + `_ls` safe localStorage wrapper
- **Real-time sync:** Firebase `.on('value')` listeners dispatch `elim-sync` CustomEvents, consumed by `useStorage` hook. Incoming events for keys in `_pendingWrites` are ignored to prevent stale data from reverting local changes.
- **Offline protection:** `_pendingWrites` Set tracks keys with in-flight Firebase writes. Failed writes retry with exponential backoff (4 retries, 2s→30s). `storage.get` fallback path does not push localStorage back to Firebase, preventing stale cache from clobbering newer remote data on reconnect.
- **Offline awareness:** Firebase `.info/connected` listener monitors connectivity status. A warning banner appears at the top of the app when the device is offline.
- **Resilience:** Firebase init in try/catch, all localStorage in `_ls` safe wrapper — app degrades gracefully: Firebase → localStorage → in-memory
- **Photo handling:** Base64 JPEG, resized to max 800×600, 70% quality. Stored in Firebase as string values. Photos excluded from initial load gate — app renders immediately without waiting for photo data.
- **Theme:** "Warm Night" — 4-state color system (orange #D4763A / green #5CB87A / dashed-dim for unassigned / neutral), warm cream text (#F0E6D8), dark warm background (#111010)
- **Brand mark:** SVG `ElimLogo` component — two right-pointing chevrons + block E, sharp corners, scaled 82% inside circle
- **Countdown timer:** `timerColor()` helper returns green/yellow/red based on remaining %. Timer state: `{startedAt, accumulated, targetMinutes}`. Auto-pauses at zero. Auto-stops at 100% non-post-event completion via `autoStoppedRef` (one-way gate per session).
- **Weekly reset:** Saves history before clearing. History record: `{date, pct, elapsed, tasks, total}`. Uses live `checklistData` for accurate `total`. Keeps last 52 weeks.
- **Signal Flow:** `SignalFlowView` component with three SVG sections using helper sub-components (Box, Arrow, Label, HLine, VLine). Audio viewBox 570×660. Video 480×510. Network 480×360. Color-coded: orange/purple/green.
- **Header positioning:** `position: fixed; top: 0` with `paddingTop: calc(env(safe-area-inset-top, 0px) + 8px)`. Body has `padding-top: 0`. Background extends behind iOS status bar. `overflow-x: clip` on body. Header spacer div: 48px height.
- **Back swipe:** `useBackSwipe` hook with follow-finger drag gesture. Real-time `translateX` via direct DOM manipulation (refs, not state). 35% screen width threshold. `data-no-swipe` attribute excludes photo scroll areas. Top-left zone (top 80px, left 120px) excluded.
- **Scroll restoration:** Retry-based restore with up to 10 attempts and 5px tolerance. Handles async-loaded views (I/O, Repairs) where content renders after initial scroll attempt.
- **Crew focused views:** `MyTasksView` accepts `targetUser` prop. `__unassigned__` special value filters to tasks with no assignee. Sections auto-collapse when all tasks are complete; tap header to toggle.
- **Unassigned styling:** `CheckItem` computes `isUnassigned` from `!assignee && !checked`, applies dashed border + dim italic. Section headers compute `unassignedCount`.
- **Card styling:** Computed via `cardStyle` function to avoid CSS property conflicts (especially `border` vs `borderLeft`).
- **Dependency warnings:** `dependsOn` property on task items. CheckItem checks prerequisite completion before toggling. Amber modal with Continue/Cancel.
- **I/O Line List:** Persistent via `useStorage("elim3-io-config", DEFAULT_IO_LINE_LIST)` owned by `ElimProductionApp`, passed as props to `IOListView`. Three tabs (Stage Box, Wing, Dante) organized by physical device. Stage Box has Inputs/Outputs pill-style sub-toggle — inputs grouped by `STAGE_BOX_GROUPS` constant (6 channel-range sections with "Other" fallback), outputs split into IEM vs Infrastructure by `isIEM()` helper matching destination field. In-ear snake renders as 🎧 pill badge. Cross-tab search via `matchesSearch()` filters all sections simultaneously, hides normal tab view during search. Editing uses draft-based flow — edits modify a local clone, not live state. "✓ Done" triggers confirm modal; Save writes draft to `useStorage` → Firebase. Edit restricted to Sam and Bri via `currentUser` prop. Custom `IODropdown` component replaces native `<select>` for type and protocol fields. No reset-to-defaults button.
- **IODropdown component:** Lightweight custom dropdown matching `AssignDropdown` visual pattern — click to open, dark `#1C1A18` dropdown panel, hover highlights, checkmark on current selection, click-outside-to-close. Used for Stage Box input type (XLR, DI/XLR, Line, 1/4") and Dante protocol (Dante, Analog, AES50, USB).
- **No auth:** User selects name on launch, no passwords. Remember-me via localStorage.
- **Shared state:** All data shared (visible to all users) via Firebase
- **Hosting:** Firebase Hosting (free tier: 10GB bandwidth/month, 1GB database storage)
- **Error boundary:** `ErrorBoundary` class component wraps `ElimProductionApp` at mount point. Catches render errors, shows reload prompt. Prevents white-screen crashes from propagating.
- **Atomic reset:** `newWeek` writes `elim3-pending-reset` flag before clearing state. On load, `useEffect` checks for flag and completes interrupted resets.
- **Orphan cleanup:** Task deletion removes entries from `checks`, `assignments`, `taskNotes`, and `completions`. Roster removal clears affected assignments and shows a blast-radius warning listing the tasks that will be unassigned.
- **Tap targets:** Checkboxes use 44px minimum tap target size per mobile accessibility guidelines.
- **Performance:** `allItems` memoized via `useMemo(, [checklistData])`. `CheckItem` wrapped in `React.memo`. Photos excluded from initial load gate. Static style objects and arrays hoisted to module scope.

---

## What NOT to Do

- **Don't split into multiple files.** The single-file architecture is intentional — it deploys to Firebase Hosting with zero build step.
- **Don't add a build system.** No webpack, no Vite, no esbuild. The CDN approach works and eliminates toolchain complexity for a non-developer maintainer.
- **Don't add authentication.** The no-auth model is intentional for a small trusted team. Adding auth would create a barrier for volunteers.
- **Don't add TypeScript.** The Babel CDN handles JSX transformation. TypeScript would require a build step.
- **Don't add CSS frameworks.** Tailwind, styled-components, etc. would either require a build step or add CDN weight. Inline styles work fine at this scale.
- **Don't refactor the navigation to React Router.** The custom navigation is simple and works. Router adds complexity and a dependency.
- **Don't over-architect.** This is a ~2,500 line app for 16 volunteers. Context providers, state management libraries, and abstraction layers are overkill unless they solve a concrete, measurable problem.
