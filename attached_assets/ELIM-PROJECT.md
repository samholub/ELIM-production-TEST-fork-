# ELIM Production App

Portable church production setup/teardown checklist app for ELIM Church (The Well / The ELIM Arrival). Progressive Web App built with React, Firebase Realtime Database for cross-device sync, offline-capable, no authentication required.

---

## Current Status: v5.8.0

**Live URL:** https://elim-production.web.app
**Test URL:** Replit preview (connected to production Firebase)
**Test repo:** https://github.com/samholub/ELIM-production-TEST-fork-

### What's Built
- **15 sections, 59 tasks** covering full setup flow from Pre-Setup through Post-Event
- **Photo documentation** — camera capture or upload per task, auto-resized, persistent
- **Troubleshooting guides** — expandable Q&A per task for common issues
- **Per-task notes** — persistent, shared across team; preview shown above the fold (hidden on checked tasks)
- **Task detail surfacing** — static reference text shown below assignee without expanding (hidden on checked tasks)
- **Custom assignment dropdown** — dark-themed dropdown replacing native select; click-outside-to-close, unassign option
- **Countdown timer** — configurable (30/45/60/90 presets or custom) with green→yellow→red transition, auto-pause at zero, syncs across devices
- **Weekly reset** — clears checkboxes/timer, preserves assignments/notes/photos; saves history (last 52 weeks); atomic with pending-reset flag
- **SVG brand mark** — geometric `ElimLogo` component (right-pointing chevrons + block E)
- **Signal Flow diagrams** — three static SVG diagrams (Audio, Video, Network/Dante) with color-coded sections
- **Crew focused views** — tap a crew member name on dashboard → focused task list
- **Unassigned task awareness** — dashboard row, dashed/italic styling, section header counts
- **Smart auto-expand** — sections with your incomplete tasks or in-progress auto-expand
- **Crew status dashboard** — per-person progress list (you sorted first) + unassigned row
- **Timer in header** — visible on Dashboard, Checklist, My Tasks, and Crew Tasks views
- **Follow-finger swipe-back gesture** — real-time content tracking, 35% threshold, header zone exclusion
- **Power sequence** — 10-step equipment power order with teardown reminder; badges on task cards
- **Editable I/O Line List** — edit channel labels, types, notes; add/remove channels; reset to defaults
- **Repairs tracker** — Firebase-persisted with full CRUD, search, priority levels
- **Checklist task editing** — search toggle (🔍 icon), edit mode (Sam only) with add/remove/edit tasks inline
- **View stack navigation** — proper breadcrumb-style back navigation with scroll position preservation
- **Default My Tasks view** — app opens to My Tasks; dashboard accessible via back navigation
- **Error boundary** — render crashes show "Tap to Reload" instead of permanent white screen
- **Dependency warnings** — prerequisite tasks must be complete (amber modal, user can always proceed)
- **Dynamic roster** — add/remove/reorder team members in Settings
- **Remember-me** — device remembers last selected user
- **Real-time cross-device sync** — Firebase Realtime Database with localStorage offline fallback
- **Resilient storage** — graceful degradation: Firebase → localStorage → in-memory
- **Firebase write retry** — failed writes tracked and retried; stale sync events ignored for pending keys
- **Error boundary** — render errors show "Tap to Reload" screen instead of permanent white screen
- **Edit mode restricted** — checklist edit mode (task add/delete) only visible to Sam
- **Task deletion confirmation** — `confirm()` dialog before removing any task
- **Atomic weekly reset** — pending-reset flag prevents partial state on interruption
- **Orphan cleanup** — task deletion and roster removal clean up all supporting data keys

---

## Design Philosophy: Warm Night

- **4 visual states:** Orange = yours, Green = done, Dashed/italic/dim = unassigned, Neutral = everything else
- **No per-person colors** in the main UI (only in Settings roster management)
- **Readable at a glance:** 16px task text, 28px checkboxes, solid triangle expand arrows — mobile thumb-sized
- **Quiet by default:** collapsed sections with separator lines, minimal badges, no timestamps
- **Your tasks stand out:** orange left border + "YOU" badge on sections with your incomplete work
- **Unassigned tasks stand out:** dashed border, lighter background, italic dimmed text
- **Crew focused views:** tap a name → see just their tasks, not the full checklist
- **Progressive disclosure:** details, photos, troubleshooting hidden until expanded
- **Calm urgency:** countdown timer uses color transition (green→yellow→red), not flashing

---

## Data Management

### Firebase / Code Sync Strategy

The hardcoded `CHECKLIST_DATA` constant in the code is the **canonical source of truth**. Firebase is the live store.

**How it works:**
- First app load (or after reset) → seeds Firebase from code constant
- After that → Firebase is the live data store
- If Firebase has data, the code constant is ignored

**Workflow:**
1. **Structural changes go through code first** — update `CHECKLIST_DATA`, deploy, then reset to pull in new baseline
2. **In-app edit mode is for live adjustments** — Sunday morning task description fix, quick task addition
3. **After any in-app edit worth keeping** — mirror it back to the code constant

**Same pattern applies to:** `DEFAULT_REPAIRS`, `DEFAULT_IO_LINE_LIST`

### Storage Keys
| Key | Content |
|-----|---------|
| `elim3-checks` | Checkbox states |
| `elim3-assign` | Task assignments |
| `elim3-comp` | Completion metadata |
| `elim3-notes` | Per-task notes |
| `elim3-photos` | Photo data (base64) |
| `elim3-roster` | Team roster |
| `elim3-timer` | Timer state |
| `elim3-history` | Setup history (last 52 weeks) |
| `elim3-io-config` | I/O line list overrides |
| `elim3-repairs` | Repairs list |
| `elim3-checklist-data` | Checklist structure (sections + tasks) |
| `elim3-pending-reset` | Transient flag for atomic reset (deleted after reset completes) |

### What Resets Clear vs Preserve
| Cleared on reset | Preserved across resets |
|---|---|
| Checkboxes | Assignments |
| Completions | Notes |
| Timer | Photos |
| | Roster |
| | I/O config |
| | Repairs |
| | Checklist structure |

---

## File Structure

```
elim-production/                  ← C:\Users\sammh\elim-production\
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
│   └── OPEN-ITEMS.md             ← On-site investigations, open questions, unknowns
│
├── archive/                      ← Previous versions for reference
│   ├── elim-v3.jsx
│   ├── elim-v5.jsx
│   ├── proto-timeline.jsx
│   ├── proto-crew.jsx
│   ├── proto-cards.jsx (rejected)
│   └── proto-hybrid.jsx
│
└── docs/
    └── checklist-source.xlsx     ← Original checklist spreadsheet
```

---

## Deployment

**Hosted on:** Firebase Hosting (free Spark plan)

**To redeploy after changes:**
1. Edit `index.html` in `C:\Users\sammh\elim-production\`
2. Open PowerShell, `cd C:\Users\sammh\elim-production`
3. Run `firebase deploy`
4. Live in ~10 seconds at the same URL

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
- **Nav Cards:** 3×2 grid — Power Sequence, I/O Line List, Signal Flow, Equipment, Repairs, Purchases

## Settings Layout
- **Reset Checklist** (top) — two-tap confirmation
- **Team Roster** — add/remove members with color assignment
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
8. 📹 Camera Setup (3 tasks)
9. 💻 ProPresenter Mac Setup (5 tasks)
10. 📡 Streaming Setup (4 tasks)
11. ⚡ Final Power-On (4 tasks)
12. 🎙️ Mic Check (2 tasks)
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
7. Glenn's ProPresenter Mac
8. Streaming Camera
9. LED Transmitter
10. Speakers

---

## Equipment Reference
- **Stage Box**: Midas DL251 (or DL252) — connects to Wing AES50 Supermac port A
- **Soundboard**: Behringer Wing (with Dante Expansion card) — Wing ethernet left port to WiFi Router; Dante Expansion to port 5 on switch
- **Network Switch**: 5-Port Switch (Dante network) — port 5 = Wing Dante Expansion
- **Tyler's Mac**: ProTools for live recording — USB-C direct to Wing (NOT on Dante network)
- **Omar/Mitch's Mac**: ProTools for stream mixing — CAT6 to 5-Port Switch (Dante)
- **Glenn's Mac**: ProPresenter for video/presentation — CAT6 to 5-Port Switch (Dante); ATEM USB in
- **Dante AVIO**: (Audinate) — connects to ATEM Mini Pro Analog Audio IN Ch 1-2
- **Video Switcher**: ATEM Mini Pro — USB to Glenn's Mac; Control Ethernet to Resi Encoder LAN 1
- **Resi Encoder**: Streaming encoder — set up alongside ATEM; SDI in from ATEM (port TBD)
- **LED Wall**: 9×5 panels (45 total) via LED Transmitter (LED Scaler removed)
- **Confidence Monitor**: Stage display from Glenn's ProPresenter Mac (USB-C)
- **Bianca's IEM**: Wing XLR Output 8
- **Streaming**: ProPresenter → Internet → Resi
- **Wireless Control**: WiFi Router → iPad

---

## Team Members
Managed dynamically via Settings. Default roster: Glenn, Sunny, Omar, Mitch, Michael, Taylor, Seth, Sam, Bianca, Bri, Leo, Austin, Tyler, Kolton, Savanah, Emily

---

## Technical Notes

- **Framework:** React 18 (via CDN, no build step)
- **Styling:** Inline styles, no CSS framework
- **Fonts:** DM Sans (body) + JetBrains Mono (data/timers) via Google Fonts
- **Storage:** Firebase Realtime Database with `window.storage` API wrapper + `_ls` safe localStorage wrapper
- **Real-time sync:** Firebase `.on('value')` listeners dispatch `elim-sync` CustomEvents, consumed by `useStorage` hook
- **Resilience:** Firebase init in try/catch, all localStorage in `_ls` safe wrapper — app degrades gracefully
- **Photo handling:** Base64 JPEG, resized to max 800×600, 70% quality. Stored in Firebase as string values.
- **Theme:** "Warm Night" — 4-state color system (orange #D4763A / green #5CB87A / dashed-dim for unassigned / neutral), warm cream text (#F0E6D8), dark warm background (#111010)
- **Brand mark:** SVG `ElimLogo` component — two right-pointing chevrons + block E, sharp corners, scaled 82% inside circle
- **Countdown timer:** `timerColor()` helper returns green/yellow/red based on remaining %. Timer state: `{startedAt, accumulated, targetMinutes}`. Auto-pauses at zero.
- **Weekly reset:** Saves history before clearing. History record: `{date, pct, elapsed, tasks, total}`. Uses live `checklistData` for accurate `total`. Keeps last 52 weeks.
- **Signal Flow:** `SignalFlowView` component with three SVG sections using helper sub-components (Box, Arrow, Label, HLine, VLine). Audio viewBox 570×660. Video 480×510. Network 480×360. Color-coded: orange/purple/green.
- **Header positioning:** `position: fixed; top: 0` with `paddingTop: calc(env(safe-area-inset-top, 0px) + 8px)`. Body has `padding-top: 0`. Background extends behind iOS status bar. `overflow-x: clip` on body. Header spacer div: 48px height.
- **Back swipe:** `useBackSwipe` hook with follow-finger drag gesture. Real-time `translateX` via direct DOM manipulation (refs, not state). 35% screen width threshold. `data-no-swipe` attribute excludes photo scroll areas. Top-left zone (top 80px, left 120px) excluded.
- **Crew focused views:** `MyTasksView` accepts `targetUser` prop. `__unassigned__` special value filters to tasks with no assignee.
- **Unassigned styling:** `CheckItem` computes `isUnassigned` from `!assignee && !checked`, applies dashed border + dim italic. Section headers compute `unassignedCount`.
- **Card styling:** Computed via `cardStyle` function to avoid CSS property conflicts (especially `border` vs `borderLeft`).
- **Dependency warnings:** `dependsOn` property on task items. CheckItem checks prerequisite completion before toggling. Amber modal with Continue/Cancel.
- **No auth:** User selects name on launch, no passwords. Remember-me via localStorage.
- **Shared state:** All data shared (visible to all users) via Firebase
- **Hosting:** Firebase Hosting (free tier: 10GB bandwidth/month, 1GB database storage)
- **Error boundary:** `ErrorBoundary` class component wraps `ElimProductionApp` at mount point. Catches render errors, shows reload prompt. Prevents white-screen crashes from propagating.
- **Firebase write resilience:** `_pendingWrites` Set tracks keys with in-flight writes. `_startListener` ignores incoming `elim-sync` events for pending keys. Failed writes retry once after 5 seconds.
- **Atomic reset:** `newWeek` writes `elim3-pending-reset` flag before clearing state. On load, `useEffect` checks for flag and completes interrupted resets.
- **Performance:** `allItems` memoized via `useMemo(, [checklistData])`. `CheckItem` wrapped in `React.memo`. Photos excluded from initial load gate. Static style objects and arrays hoisted to module scope.
